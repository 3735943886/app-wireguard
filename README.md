# ipTIME redirect (WireGuard)

ipTIME 공유기에 연결된 특정 기기(예: LG ThinQ 가전)의 트래픽을 WireGuard
터널로 Home Assistant에 끌어와, 원하는 포트로 리다이렉트(DNAT)하는
Home Assistant 앱입니다.

ipTIME 공유기는 관리 화면에서 외부로 나가는 트래픽을 DNAT할 수 없습니다.
그 대신 **WireGuard 클라이언트**와 **클라이언트 사용 관리** 기능으로 원하는
기기의 트래픽만 터널로 보낼 수 있습니다. 이 앱은 그 터널의 서버가 되어, 받은
트래픽을 Home Assistant의 [rusthinq][rusthinq] 같은 서비스로 넘겨줍니다.

```text
LG 가전 ──▶ ipTIME ══WireGuard══▶ 이 앱 ──DNAT──▶ Home Assistant (rusthinq)
            (기기 MAC 기준 정책)        443  → HA:443
                                        8883 → HA:8883
```

## 특징

- 리다이렉트할 포트와 대상 포트를 설정에서 바로 지정 (기본값 `443`, `8883`)
- Home Assistant의 LAN 주소를 자동으로 찾아 서버 주소와 리다이렉트 대상으로 사용
- 사이드바 웹 UI에서 ipTIME에 올릴 `.conf` 파일을 바로 다운로드
- 기본 포트 `51830/udp`: 원본 WireGuard 앱과 함께 설치해도 충돌하지 않음

## 설치

1. Home Assistant에서 **설정 → 앱 → 앱 스토어 → ⋮ → 저장소**로 이동합니다.
1. `https://github.com/3735943886/app-wireguard`를 추가합니다.
1. 목록에 나타난 **ipTIME redirect (WireGuard)**를 설치합니다. Home Assistant가
   직접 이미지를 빌드하므로 처음 설치는 몇 분 걸립니다.

설정과 ipTIME 등록 방법은 [문서](wireguard/DOCS.md)를 참고하세요.

## 출처와 라이선스

이 앱은 Franck Nijhof의 [Home Assistant Community App: WireGuard][upstream]를
포크해 ipTIME 용도에 맞게 고친 것입니다. 원본과 같이 [MIT 라이선스](LICENSE.md)를
따릅니다. WireGuard®는 Jason A. Donenfeld의 등록 상표입니다.

[rusthinq]: https://github.com/3735943886/rusthinq-local
[upstream]: https://github.com/hassio-addons/app-wireguard
