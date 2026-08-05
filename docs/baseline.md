# Rocky Linux 물리 서버 기준선

이 문서는 Link/Carrier 실험 전에 실제 서버에서 확인한 네트워크와 원격 접속 기준선이다.
정확한 주소와 장비 식별자는 자리표시자로 바꿨고 원본 증거는 공개 문서와 분리했다.

## 기록 범위

- 기준선 수집일: 2026-08-03
- 실험 전 재확인일: 2026-08-04
- 공개 서버 이름: `<LAB-SERVER>`
- 서버: HPE ProLiant DL360 Gen9
- 운영체제: Rocky Linux 10.2
- 커널: Linux 6.12 계열
- 확인 대상: NIC, Link/Carrier, IP·Route, OpenSSH와 Listening 포트
- 공개 증거: [`2026-08-04-baseline-summary.txt`](../evidence/sanitized-public/2026-08-04-baseline-summary.txt)

실험 전부터 있던 일반 하드웨어 상태 경고는 네트워크 실습과 분리했다.
그 경고의 원인 분석과 수리는 수행하지 않았다.

## 인터페이스 기준선

| 인터페이스 | Link 상태 | Carrier | operstate | 주소·Route | 관찰된 역할 |
|---|---|---:|---|---|---|
| `eno4` | `UP`, `LOWER_UP` | `1` | `up` | `<SERVER-IP>/<SERVER-PREFIX>`, 기본 Route | 외부 SSH 주 경로 |
| `eno1` | `DOWN`, `NO-CARRIER` | `0` | `down` | IPv4와 Gateway 없음 | 평소 미사용, 보조 실험 경로 |
| `eno2` | `DOWN`, `NO-CARRIER` | `0` | `down` | 미확인 | 미사용 |
| `eno3` | `DOWN`, `NO-CARRIER` | `0` | `down` | 미확인 | 미사용 |

표의 `eno1` 값은 테스트 케이블을 연결하기 전 원래 상태다.
직접 연결한 임시 테스트 기준선의 Carrier `1` 상태와 구분한다.

`eno2`와 `eno3`은 인터페이스 존재와 Carrier 없음까지만 확인했다.
상세 물리 포트 위치는 확정하지 않았다.
Link LED는 독립된 공개 기록이 없어 확인 결과로 사용하지 않았다.

## 주 경로 eno4

- 온보드 NIC 4와 Linux `eno4`의 관계를 섀시 포트 관찰과 udev 정보로 확인했다.
- 드라이버는 `tg3`였다.
- Carrier는 `1`, operstate는 `up`이었다.
- 협상된 Link는 1000Mb/s Full Duplex였고 자동 협상이 켜져 있었다.
- `<SERVER-IP>/<SERVER-PREFIX>`가 관찰됐다.
- 기본 Route와 `<EXTERNAL-IP>` Route 조회는 모두 `eno4`를 선택했다.

Route 결과는 확인한 목적지에 대한 당시 커널의 경로 선택만 보여 준다.
전체 외부 목적지, 패킷 손실이나 지연을 보장하지 않는다.

## 보조 경로 eno1

- 온보드 NIC 1과 Linux `eno1`의 관계를 직접 연결과 Carrier 변화로 확인했다.
- 원래 상태에서는 테스트 케이블이 없었다.
- Carrier는 `0`, operstate는 `down`이었다.
- IPv4 주소와 Gateway는 관찰되지 않았다.
- 외부 SSH 주 경로로 사용하지 않았다.

노트북을 연결한 임시 기준선에서는 Carrier `1`과 1Gbps Full Duplex를 관찰했다.
`eno1`용 NetworkManager 프로파일은 존재했지만 inactive 상태였다.
active IPv4 연결은 성립하지 않았고, 직접 연결 구간에서 DHCP 임대를 받지 못했다.
직접 연결 구간의 IPv4 통신은 확인하지 않았다.

## 서비스와 Listening 포트

| 항목 | 관찰 결과 | 해석 범위 |
|---|---|---|
| OpenSSH 서버 | loaded, active, running | 서비스 상태 조회 |
| TCP 22 | IPv4와 IPv6에서 Listening | 포트 조회 |
| TCP 9090 | Listening | 소유 프로세스와 소켓 상태 미확인 |
| firewalld | active | 당시 서비스 상태 조회 |
| public zone | `eno4` 연결, SSH와 Cockpit 허용 항목 관찰 | 당시 규칙 조회 |

TCP 9090의 Listening만으로 특정 프로세스나 `cockpit.socket` 상태를 소급해 확정하지 않는다.

## Link/Carrier 실험 종료 후 외부 SSH 경로 확인

실험 종료 뒤 점검에서 `eno4` Link를 up으로 관찰했고,
확인한 외부 목적지 Route는 `eno4`를 선택했다.
같은 점검에서 `sshd`는 active였고 TCP 22는 Listening 상태였다.
테스트 케이블 제거 뒤 별도의 새 SSH 세션 한 번이 연결됐다.

한 번의 연결은 모든 사용자나 인증 방식의 무영향을 증명하지 않는다.

## 확인하지 않은 항목

- Gateway 또는 외부 목적지 ICMP 응답
- HTTP와 메일 서비스
- 패킷 손실과 지연
- `eno1`의 IPv4 통신
- 모든 사용자와 인증 방식의 영향
- 스위치, VLAN과 전체 물리 토폴로지
- TCP 9090의 소유 프로세스와 소켓 상태
- 실제 장애에서 iLO를 사용한 복구

이 기준선은 기록한 시점과 확인한 경로에만 적용한다.
