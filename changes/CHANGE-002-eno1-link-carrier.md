# CHANGE-002: eno1 Link/Carrier 실험 계획과 결과

## 목적

주 경로 `eno4`를 변경하지 않고 보조 경로 `eno1`의 테스트 케이블 분리와 재연결이
Carrier와 Link 상태에 어떻게 나타나는지 확인했다.

- 수행일: 2026-08-04
- 대상: `<LAB-SERVER>`의 `eno1`
- 주 접속 경로: `eno4`
- 변경 변수: 노트북 쪽 테스트 케이블 연결 상태 한 가지

## 실행 조건

- 대상 서버를 사용할 수 있다고 전달받았다.
- 다른 로컬 사용자가 작업 중이지 않음을 직접 확인했다.
- iLO Remote Console과 로컬 콘솔 로그인이 가능했다.
- `eno4`와 `eno1`의 역할을 구분했다.
- `eno4` Link, Route 또는 SSH 상태에 변화가 생기면 중단하기로 했다.
- 물리 동작마다 상태를 다시 조회하고 결과를 추정하지 않기로 했다.

콘솔 접근 가능 여부는 실제 SSH 장애 복구를 수행했다는 뜻이 아니다.

## 두 기준 상태

### 운영 원래 상태

- `eno1` 테스트 케이블 없음
- Carrier 0, Link 미감지
- IPv4 주소와 Gateway 없음
- `eno4`가 주 경로와 기본 Route 담당

### 임시 테스트 기준선

- 노트북과 `eno1`을 테스트 케이블로 직접 연결
- Carrier 1
- 1Gbps Full Duplex
- `eno1`용 NetworkManager 프로파일은 존재했지만 inactive 상태
- active IPv4 연결 미성립, 직접 연결 구간에서 DHCP 임대 미수신

## 계획한 순서

1. 운영 원래 상태를 기록한다.
2. 테스트 케이블을 연결해 임시 기준선을 만든다.
3. 노트북 쪽 케이블만 분리한다.
4. Carrier와 Link 변화를 확인한다.
5. 변화가 없으면 성공으로 기록하지 않고 한 번 재시도한다.
6. 같은 케이블을 재연결해 임시 기준선으로 Rollback한다.
7. 테스트 케이블을 제거해 운영 원래 상태로 종료한다.
8. 실험 종료 뒤 주 경로 상태와 새 SSH 세션을 확인한다.

## Rollback과 실험 종료

Rollback 목표는 Carrier 1과 1Gbps Full Duplex인 임시 테스트 기준선이다.
같은 테스트 케이블을 재연결한 뒤 이 두 상태를 확인한다.

실험 종료는 Rollback 확인 뒤 케이블을 제거하는 별도 단계다.
종료 목표는 실험 전 미사용 상태인 Carrier 0이다.

## 결과 요약

```text
운영 원래 상태 Carrier 0
→ 임시 테스트 기준선 Carrier 1
→ 첫 분리 확인에서 변화 없음
→ 분리 재시도 뒤 Carrier 0과 NO-CARRIER
→ 재연결 Rollback 뒤 Carrier 1
→ 테스트 케이블 제거 뒤 Carrier 0
```

첫 분리에서 변화가 없었으므로 성공으로 기록하지 않았고 원인은 확인하지 않았다.
재시도 뒤 확인한 직접 원인 범위는 노트북 쪽 케이블 분리에 따른 물리 Link 손실이다.

Rollback에서는 Carrier 1과 1Gbps Full Duplex를 다시 관찰했다.

실험 종료 뒤 `eno4` Link, 외부 목적지 Route의 `eno4` 선택,
`sshd` active와 TCP 22 Listening을 확인했으며, 별도의 새 SSH 세션 1회가 연결됐다.

`eno1` IPv4 통신, Gateway·외부 Ping, HTTP, 메일, 손실·지연,
모든 사용자 영향과 전체 물리 토폴로지는 확인하지 않았다.

상세 타임라인은
[`INCIDENT-001`](../incidents/observed/INCIDENT-001-eno1-link-carrier.md)에 남겼다.
