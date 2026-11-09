# ipTIME redirect (WireGuard)

ipTIME 공유기에 연결된 특정 기기(예: LG ThinQ 가전)의 트래픽을 WireGuard
터널로 받아, 지정한 포트를 Home Assistant로 리다이렉트(DNAT)합니다.

```text
LG 가전 ──▶ ipTIME ══WireGuard══▶ 이 앱 ──DNAT──▶ Home Assistant (rusthinq)
```

- 리다이렉트할 포트와 대상 포트를 설정에서 지정 (기본값 `443`, `8883`)
- 웹 UI에서 ipTIME [설정 가져오기]에 올릴 `.conf` 파일을 바로 다운로드
- 기본 포트 `51830/udp`로 원본 WireGuard 앱과 함께 설치 가능

설정과 ipTIME 등록 방법은 문서 탭을 참고하세요.
