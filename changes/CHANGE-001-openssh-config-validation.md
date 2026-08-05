# CHANGE-001: OpenSSH 설정 복사본 검사

## 목적

운영 OpenSSH 설정을 바꾸지 않고 정상 복사본과 의도적으로 잘못된 복사본을
`sshd` 문법 검사로 구분할 수 있는지 확인했다.

- 수행일: 2026-08-03
- 대상: `<LAB-SERVER>`
- 운영 설정 변경: 없음
- reload 또는 restart: 없음

## 작업 전 확인

- 대상 서버를 사용할 수 있다고 전달받았다.
- 다른 로컬 사용자가 작업 중이지 않음을 직접 확인했다.
- `eno4`를 주 경로, `eno1`을 보조 경로로 구분했다.
- `sshd`는 active였고 TCP 22가 Listening 중이었다.
- 현재 운영 설정의 문법 검사 결과는 종료 코드 0이었다.

## 실행과 결과

| 순서 | 확인 내용 | 결과 |
|---|---|---|
| 1 | 운영 설정 문법 검사 | 종료 코드 0 |
| 2 | `/etc/ssh/sshd_config`를 임시 `TEST_CONFIG`로 복사 | 운영 파일 무변경 |
| 3 | 정상 임시 복사본 검사 | 종료 코드 0 |
| 4 | 임시 복사본에만 잘못된 지시어 추가 | 운영 파일 무변경 |
| 5 | 잘못된 복사본 검사 | 종료 코드 255 |
| 6 | `sshd`와 TCP 22 재확인 | active, Listening |
| 7 | `<CONTROL-NODE>`에서 새 SSH 세션 확인 | 1회 연결 |
| 8 | 임시 검증 자료 제거 | 제거함 |

잘못된 복사본은 `Bad configuration option`으로 거부됐으며 실행 중인 서비스에 적용하지 않았다.

## 영향과 Rollback

운영 `/etc/ssh` 파일을 수정하지 않았다.
검사 결과를 운영 서비스에 적용하지 않았고 reload와 restart도 실행하지 않았다.

따라서 운영 설정 Rollback은 해당하지 않는다.
임시 복사본만 검사 뒤 제거했다.

## 확인 범위와 한계

- 정상 복사본: 종료 코드 0
- 잘못된 복사본: 종료 코드 255
- `sshd`: active
- TCP 22: IPv4와 IPv6 Listening 관찰
- 새 SSH 세션: 1회 연결

Gateway Ping, 외부 Ping, HTTP, 다른 응용 서비스, 손실과 지연은 확인하지 않았다.
모든 사용자, 클라이언트와 인증 방식의 영향을 확인한 것도 아니다.

임시 주 설정 파일을 사용했지만 모든 절대 경로 Include 대상까지
별도로 복제해 완전히 격리했는지는 확인하지 않았다.

공개 관찰 기록은
[`2026-08-04-openssh-change-validation.txt`](../evidence/sanitized-public/2026-08-04-openssh-change-validation.txt)에 남겼다.
