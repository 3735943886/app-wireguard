# ipTIME redirect (WireGuard)

ipTIME 공유기에 연결된 특정 기기의 트래픽을 WireGuard 터널로 받아, 지정한
포트를 Home Assistant(또는 다른 주소)로 리다이렉트(DNAT)하는 앱입니다.
LG ThinQ 가전을 [rusthinq][rusthinq]에 연결하는 용도를 기본으로 합니다.

## 동작 방식

```text
LG 가전 ──▶ ipTIME ══WireGuard══▶ 이 앱 ──DNAT──▶ Home Assistant (rusthinq)
            (기기 MAC 기준 정책)        443  → HA:443
                                        8883 → HA:8883
```

1. ipTIME의 **WireGuard 클라이언트**가 이 앱에 터널을 연결합니다.
1. **클라이언트 사용 관리**에서 지정한 기기(LG 가전)의 트래픽만 터널로 들어옵니다.
1. 앱이 터널로 들어온 트래픽 중 `redirects`에 지정한 포트를 대상 주소로
   DNAT합니다. 나머지 트래픽은 Home Assistant 호스트를 거쳐 그대로 인터넷으로
   나갑니다.

기기 입장에서는 원래 서버와 통신하는 것처럼 보이므로, 기기 설정이나 DNS를
바꿀 필요가 없습니다.

## 빠른 시작

### 1. 앱 설정

대부분 기본값 그대로 쓰면 됩니다. rusthinq가 다른 포트에서 동작한다면
`redirects`의 `to_port`만 바꿉니다.

```yaml
server:
  addresses:
    - 172.27.66.1
  dns: []
peers:
  - name: iptime
    addresses:
      - 172.27.66.2
    allowed_ips: []
    client_allowed_ips: []
    client_dns: "off"
redirects:
  - port: 443
    to_port: 443 # rusthinq HTTPS 포트 (443이 사용 중이면 8443 등)
  - port: 8883
    to_port: 8883 # rusthinq MQTT 포트
```

앱을 시작하면 로그에 리다이렉트 내역이 표시됩니다.

```text
Redirecting tcp/443 from the tunnel to 192.168.0.50:443
Redirecting tcp/8883 from the tunnel to 192.168.0.50:8883
```

### 2. 설정 파일 받기

앱 정보 화면의 **웹 UI 열기**(또는 "사이드바에 표시"를 켠 뒤 사이드바의
**ipTIME redirect**)를 누르고, `iptime.conf 다운로드`를 누릅니다.

웹 UI는 Home Assistant에 로그인한 사용자만 볼 수 있습니다. 설정 파일에는
개인키가 들어 있으니 다른 곳에 공유하지 마세요.

### 3. ipTIME에 WireGuard 클라이언트 등록

1. ipTIME 관리 화면(보통 `http://192.168.0.1`)에 로그인합니다.
1. **관리도구 › VPN 설정 › WireGuard 클라이언트**에서 **추가(+)**를 누릅니다.
1. **[설정 가져오기]**를 눌러 받은 `iptime.conf`를 올립니다.
1. 이름을 입력하고, 클라이언트는 **실행**을 선택합니다.
1. **[모든 트래픽이 이 VPN을 경유하도록 규칙 생성]은 체크하지 마세요.**
   체크하면 공유기에 연결된 모든 기기의 트래픽이 Home Assistant를 거치게
   됩니다.
1. **[추가]**를 누릅니다.

### 4. 리다이렉트할 기기 지정

1. **관리도구 › VPN 설정 › 클라이언트 사용 관리**에서 **추가(+)**를 누릅니다.
1. 다음과 같이 설정하고 **[추가]**를 누릅니다.
   - 이름: 아무 이름 (예: `LG 가전`)
   - VPN: 3단계에서 등록한 항목
   - 단말: 리다이렉트할 기기 (LG 가전들)
   - 목적지: **모든 목적지**
   - 네트워크 Lock: **끄기**

### 5. 기기 다시 연결

기기는 이미 맺어 둔 연결을 계속 쓰기 때문에, 기기를 재부팅하거나 ipTIME의
클라이언트 관리 화면에서 Wi-Fi 연결을 끊었다가 다시 연결합니다. 이후 새로
맺는 연결부터 터널을 거쳐 리다이렉트됩니다.

### 6. 확인

- ipTIME의 WireGuard 클라이언트 목록에서 연결 상태를 확인합니다.
- 앱 로그에서 시작 30초 뒤 출력되는 `wg show` 결과에 `latest handshake`가
  있으면 터널이 연결된 것입니다.
- rusthinq를 쓰는 경우, 기기가 접속하면 Home Assistant에 기기와 엔티티가
  자동으로 생깁니다.

## 설정 옵션

### `redirects`

터널로 들어온 트래픽 중 리다이렉트할 포트 목록입니다. 원래 목적지와 상관없이
`port`로 가는 연결을 `to_host:to_port`로 보냅니다.

| 항목 | 설명 | 기본값 |
| --- | --- | --- |
| `port` | 가로챌 목적지 포트 | (필수) |
| `protocol` | `tcp` 또는 `udp` | `tcp` |
| `to_host` | 보낼 주소 | Home Assistant의 LAN 주소 |
| `to_port` | 보낼 포트 | `port`와 같음 |

```yaml
redirects:
  - port: 443
    to_port: 8443
  - port: 8883
    to_port: 18883
```

### `server.host`

ipTIME이 접속할 이 앱의 주소입니다. 비워두면 Home Assistant의 LAN 주소를
자동으로 사용합니다. ipTIME은 사설 IP 엔드포인트를 허용하므로 보통 비워두면
됩니다. 포트는 넣지 말고, 바꾸려면 앱 설정의 "네트워크" 섹션을 사용합니다.

### `server.addresses`

터널 안에서 쓰는 서버 주소입니다. 기본값 `172.27.66.1`을 그대로 쓰면 되고,
집 LAN 대역과 겹치지 않게만 하면 됩니다.

### `server.dns`

서버 쪽 DNS입니다. 비워두면 Home Assistant의 DNS를 사용합니다.

### `peers`

터널에 접속할 클라이언트 목록입니다. ipTIME 하나면 기본값 `iptime` 하나로
충분합니다.

| 항목 | 설명 |
| --- | --- |
| `name` | 이름. 설정 파일 이름(`이름.conf`)으로도 쓰입니다. |
| `addresses` | 터널 안에서 쓸 주소 (예: `172.27.66.2`) |
| `allowed_ips` | 이 피어에서 받을 출발지 주소. 비워두면 `addresses`를 사용합니다. |
| `client_allowed_ips` | 클라이언트가 터널로 보낼 대상. 비워두면 `0.0.0.0/0` |
| `client_dns` | 클라이언트 설정에 넣을 DNS(쉼표로 구분). `off`면 넣지 않습니다. |
| `private_key` / `public_key` | 직접 지정할 키. 비워두면 자동 생성합니다. |
| `pre_shared_key` | 사전 공유 키(PSK) |
| `persistent_keep_alive` | Keepalive 간격(초). 기본값 `25` |

ipTIME에서는 `client_dns: "off"`를 권장합니다. 기기의 DNS 조회를 Home
Assistant에 의존하지 않게 됩니다.

### `log_client_config`

`true`로 하면 각 피어의 클라이언트 설정을 로그에 출력합니다. 웹 UI를 쓸 수
없을 때만 사용하세요. **로그에 개인키가 남으니** 확인 후 다시 끄세요.

### 고급 옵션

원본 WireGuard 앱의 옵션도 그대로 쓸 수 있습니다.

- `server.interface`, `server.private_key`, `server.public_key`,
  `server.fwmark`, `server.table`, `server.mtu`
- `server.pre_up`, `server.pre_down`, `server.post_up`, `server.post_down`
- `peers.endpoint`, `peers.fwmark`
- `log_level`

`server.post_up`/`server.post_down`을 직접 지정하면 기본 FORWARD와
MASQUERADE 규칙이 빠집니다. 리다이렉트 응답이 돌아오려면 이 두 규칙이 꼭
필요하니 직접 지정할 때 함께 넣으세요. `redirects` 규칙은 이와 상관없이
따로 추가됩니다.

## 포트

| 포트 | 용도 |
| --- | --- |
| `51830/udp` | WireGuard. ipTIME이 이 포트로 접속합니다. 원본 WireGuard 앱(`51820`)과 겹치지 않게 기본값을 바꿨습니다. |
| `80/tcp` | 피어 상태 API (기본값 비활성) |

같은 LAN 안의 ipTIME이 접속하므로 포트 포워딩은 필요 없습니다.

## 내부 동작

앱은 `wg-quick`으로 터널을 올린 뒤 다음 iptables 규칙을 적용합니다.

```text
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -t nat -A PREROUTING -i wg0 -p tcp --dport 443 -j DNAT --to-destination <HA_IP>:443
iptables -t nat -A PREROUTING -i wg0 -p tcp --dport 8883 -j DNAT --to-destination <HA_IP>:8883
```

앱이 멈추면 같은 규칙을 지웁니다. Home Assistant 터미널에서 다음 명령으로
적용된 규칙을 볼 수 있습니다(슬러그 앞부분은 설치 환경마다 다릅니다).

```bash
docker exec addon_<저장소ID>_wireguard iptables -t nat -S
```

생성된 파일은 `/ssl/wireguard`에 저장됩니다. 서버 개인키와 피어별 폴더
(`client.conf`, `qrcode.png`, 개인키)가 들어 있습니다.

## 문제 해결

### ipTIME이 연결되지 않음

- 앱이 실행 중인지, 로그에 오류가 없는지 확인합니다.
- 설정 파일의 `Endpoint`가 Home Assistant의 LAN 주소와 `51830` 포트인지
  확인합니다. 다르면 `server.host`를 지정하고 설정 파일을 다시 받습니다.
- 앱 설정에서 키나 주소를 바꿨다면 설정 파일을 다시 받아 ipTIME에 다시
  등록합니다.

### 터널은 연결됐는데 기기가 리다이렉트되지 않음

- 기기를 재부팅하거나 Wi-Fi를 다시 연결했는지 확인합니다. 기존 연결은
  터널을 거치지 않습니다.
- 클라이언트 사용 관리에 해당 기기가 지정되어 있는지 확인합니다.
- `to_port`가 실제로 서비스가 동작하는 포트인지 확인합니다.
- ipTIME이 기기 IP를 그대로 터널에 보내는 경우, 서버가 그 패킷을 버릴 수
  있습니다. 이때는 `peers.allowed_ips`에 집 LAN 대역(예: `192.168.0.0/24`)을
  추가해 보세요.

### "IP forwarding is disabled on the host system!"

호스트에서 IP 포워딩이 꺼져 있으면 기본 규칙이 적용되지 않아 리다이렉트도
동작하지 않습니다. Home Assistant OS에서는 보통 켜져 있습니다.

## 업데이트

앱을 고치면 `config.yaml`의 버전을 올려 배포합니다. Home Assistant에서
**앱 스토어 → ⋮ → 업데이트 확인**을 누르면 새 버전이 표시됩니다. 변경 내역은
[CHANGELOG](CHANGELOG.md)를 참고하세요.

## 출처와 라이선스

Franck Nijhof의 [Home Assistant Community App: WireGuard][upstream]를 포크해
ipTIME 용도에 맞게 고쳤습니다. 원본과 같이 MIT 라이선스를 따릅니다.
WireGuard®는 Jason A. Donenfeld의 등록 상표입니다.

[rusthinq]: https://github.com/3735943886/rusthinq-local
[upstream]: https://github.com/hassio-addons/app-wireguard
