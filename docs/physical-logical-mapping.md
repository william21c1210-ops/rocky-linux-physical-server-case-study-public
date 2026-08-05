# 물리 NIC와 Linux 인터페이스 매핑

이 문서는 물리 포트, Linux 인터페이스와 관찰된 역할의 관계를 정리한다.
주소, MAC, 연결 식별자와 장비 위치 정보는 공개본에서 제외했다.

## 확인된 경로

```text
온보드 NIC 4 → eno4 → 기본 Route → 외부 SSH
온보드 NIC 1 → eno1 → 노트북 직접 연결 → Link/Carrier 실험
```

`eno4`와 `eno1`의 역할은 분리돼 있었다.
실험은 주 경로인 `eno4`를 변경하지 않고 평소 미사용인 `eno1`에서 수행했다.

## 매핑 표

| 물리 NIC | Linux 인터페이스 | 관찰된 Link | IP·Route | 역할 |
|---|---|---|---|---|
| 온보드 NIC 4 | `eno4` | Carrier `1`, 1000Mb/s Full Duplex | 주 IPv4, 기본 Route | 외부 SSH 주 경로 |
| 온보드 NIC 1 | `eno1` | 케이블 없음 `0`, 직접 연결 `1` | 원래 상태에서 IPv4 없음 | 보조 Link 실험 |
| 세부 포트 미확정 | `eno2` | Carrier `0` | 미확인 | 미사용 |
| 세부 포트 미확정 | `eno3` | Carrier `0` | 미확인 | 미사용 |

`eno2`와 `eno3`은 존재와 Carrier 없음까지만 확인했다.
공개 기록으로 두 인터페이스의 물리 포트 매핑을 확정하지 않는다.

## 매핑 근거

| 관계 | 확인 방법 | 관찰 결과 | 판단 범위 |
|---|---|---|---|
| 온보드 NIC 4 ↔ `eno4` | 섀시 포트 관찰, `udevadm` 정보 | 온보드 이름 `eno4` | 해당 포트와 인터페이스 |
| `eno4` ↔ 물리 Link | Carrier, operstate, `ethtool` | `1`, `up`, 1000Mb/s Full Duplex | Layer 1 |
| `eno4` ↔ IP·Route | 주소, Route와 목적지 Route 조회 | 기본 Route와 확인한 목적지가 `eno4` 선택 | 기록한 주소와 목적지 |
| 온보드 NIC 1 ↔ `eno1` | 직접 연결과 분리, Carrier 조회 | 연결 상태에 따라 `1`과 `0` 관찰 | 직접 연결 구간 |
| `eno1` ↔ 물리 Link | Carrier, operstate, `ethtool` | 연결 때 1000Mb/s Full Duplex | Layer 1 |

Link LED는 독립된 증거로 기록하지 않았다.
인터페이스 이름, Carrier, udev 정보와 직접 연결 변화를 함께 근거로 사용했다.

## eno4 주 경로

- 물리 역할: 온보드 NIC 4
- Linux 인터페이스: `eno4`
- 드라이버: `tg3`
- Carrier: `1`
- operstate: `up`
- Link: 1000Mb/s Full Duplex
- IPv4: `<SERVER-IP>/<SERVER-PREFIX>`
- 기본 Gateway: `<DEFAULT-GATEWAY>`
- Route: 기본 Route와 `<EXTERNAL-IP>` 조회가 `eno4` 선택
- 사용자 경로: Link/Carrier 실험 종료 뒤 별도의 새 SSH 세션 한 번 연결

확인 범위는 기록한 목적지와 한 번의 SSH 연결이다.
Gateway Ping, 외부 Ping, HTTP, 메일, 패킷 손실과 지연은 확인하지 않았다.

## eno1 보조 경로

원래 상태에서는 테스트 케이블이 없고 Carrier는 `0`이었다.
노트북을 연결한 임시 테스트 기준선에서는 Carrier `1`과 1Gbps Full Duplex를 관찰했다.

첫 분리 확인에서는 Carrier 변화가 없어 성공으로 기록하지 않았다.
원인은 추정하지 않고 케이블 위치와 관찰 절차를 확인한 뒤 재시도했다.

재시도에서는 Carrier `0`과 `NO-CARRIER`를 관찰했다.
같은 케이블을 재연결한 Rollback에서는 Carrier `1`과 1Gbps Full Duplex로 돌아왔다.

검증 뒤 케이블을 제거한 실험 종료에서는 원래 미사용 상태인 Carrier `0`으로 돌아왔다.
Rollback의 목표는 임시 테스트 기준선 복구였다.
실험 종료의 목표는 테스트 장비 제거와 원래 상태 복귀였다.

두 단계의 목적과 완료 조건은 서로 다르다.
`eno1`용 NetworkManager 프로파일은 존재했지만 inactive 상태였다.
active IPv4 연결은 성립하지 않았고, 직접 연결 구간에서 DHCP 임대를 받지 못했다.
직접 연결 구간의 IPv4 통신은 확인하지 않았다.

## 해석 한계

- `eno1` 결과는 직접 연결 구간의 물리 Link 변화에 한정한다.
- Carrier `1`은 IP나 응용 서비스의 정상 상태를 뜻하지 않는다.
- `eno4` Route 결과는 확인한 목적지에 한정한다.
- 모든 사용자와 인증 방식의 무영향은 확인하지 않았다.
- 스위치 포트, VLAN과 전체 물리 토폴로지는 확인하지 않았다.
- 실제 장애에서 iLO를 사용한 복구는 수행하지 않았다.
- 공개본은 실제 주소, MAC, 연결 식별자와 위치를 제공하지 않는다.

정확한 관찰 시간과 첫 무변화 기록은 Incident 문서가 담당한다.
