# Rocky Linux 물리 서버 NIC 진단 Runbook

> 이 문서는 이 저장소의 `eno1`·`eno4` 실습 환경을 위한 진단 체크리스트이며, 실제 운영 환경의 범용 Runbook을 대신하지 않는다.

## 1. 작업 전 경로 구분

대상 서버와 인터페이스 이름을 먼저 확인한다.

- `eno4`: 주 접속 및 외부 Route 경로
- `eno1`: 보조 Link/Carrier 실험 경로
- 한 번에 하나의 물리 변수만 변경

점검 시 `eno4` Link·목적지 Route 또는 SSH 접속 상태에서 예상하지 못한 변화가 관찰되면 추가 물리 동작을 중단한다.

## 2. 물리 케이블

대상 포트와 케이블 양 끝, 변경할 한쪽 끝을 기록한 뒤 물리 동작마다 다음 상태를 조회한다.

```bash
cat /sys/class/net/eno1/carrier
cat /sys/class/net/eno1/operstate
```

- `carrier 1`: 커널이 물리 Link를 감지한 상태
- `carrier 0`: 물리 Link를 감지하지 못한 상태
- `operstate`: 커널이 보는 인터페이스 동작 상태

물리 동작 뒤 값이 바뀌지 않으면 장애가 발생했다고 기록하지 않는다.

## 3. Link와 Carrier

```bash
ethtool eno1
```

`Link detected`, `Speed`, `Duplex`를 함께 확인한다.

임시 테스트 기준선에서는 Carrier 1과 1Gbps Full Duplex가 관찰됐다.
분리 재시도 뒤에는 Carrier 0과 `NO-CARRIER`가 관찰됐다.
Link가 있어도 IPv4 주소나 통신이 성립했다고 단정하지 않는다.

## 4. Linux 인터페이스

```bash
ip -br link
ip -br addr
```

인터페이스 이름, 관리 상태, Link 플래그와 주소 유무를 함께 본다.

- `UP`만으로 케이블, 주소 또는 Route 정상을 판단하지 않는다.
- `eno1`은 원래 테스트 케이블과 IPv4 주소가 없는 미사용 경로였다.

## 5. IP와 Route

```bash
ip route get <EXTERNAL-IP>
```

출력에서 선택된 인터페이스를 확인한다. 이 실습의 외부 목적지 Route는 `eno4`를 선택했다.

Route 조회는 커널의 경로 선택만 보여 주며 `eno1` 직접 연결 구간의 IPv4 통신은 확인하지 않았다.

## 6. 서비스와 Listening

```bash
systemctl is-active sshd
ss -lnt
```

`sshd` 상태와 TCP 22 Listening을 함께 확인하되 두 결과를 별개의 관찰로 기록한다.

조회 중 설정 변경, reload 또는 restart를 수행하지 않는다.

## 7. 시점별 새 SSH 세션 확인

이 실습에서는 다음 두 시점에 별도의 새 SSH 세션을 각각 한 번 연결했다.

- 2026-08-03 OpenSSH 설정 복사본 검사 후 1회
- 2026-08-04 `eno1` Link/Carrier 실험 종료 후 1회

각 확인은 해당 시점의 새 연결 결과이며 모든 사용자와 인증 방식을 대표하지 않는다.

Gateway Ping, 외부 Ping, HTTP, 메일, 손실과 지연은 확인하지 않았다.

## 8. Rollback과 실험 종료

Rollback은 임시 테스트 기준선으로 되돌리는 단계로, 재연결 뒤 Carrier 1과 1Gbps Full Duplex를 확인했다.

테스트 장비 제거는 별도 종료 단계이며 최종 Carrier 0은 실험 전 미사용 상태로 돌아간 결과다.

관찰 결과는 공개 증거에 연결하고 원인 범위는 직접 확인한 물리 Link 구간보다 넓히지 않는다.
