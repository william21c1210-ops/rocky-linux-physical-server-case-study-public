# 별도 유휴 HPE DL360 Gen9 섀시 구조 관찰

## 목적

이 문서는 Change·Incident·구축 결과가 아니라 보조 관찰 기록이다.
기존 프로젝트의 장애 진단이나 복구 범위를 확장하지 않고,
별도 유휴 서버의 상판과 섀시 내부 구조를 비교해 관찰한 내용만 정리한다.

## 장비 구분

이번에 관찰한 장비는 관찰 당시 사용 중이지 않은 별도 유휴 서버다.
상판 서비스 가이드에서 기존 NIC 실습 서버와 같은
HPE ProLiant DL360 Gen9 모델명을 확인했지만,
서로 다른 물리 장비이며 SKU와 부품 옵션의 동일성은
확인하지 않았다.
또한 CHANGE-003에서 3TB HDD를 장착한 모델 미확정 유휴 서버와도 다른 장비다.

정확한 Firmware와 장비 상태도 확인하지 않았다.

## 상판 Service Map 관찰

상판 안쪽의 Service Map을 보며 System Board, DIMM, Fan과
전·후면 Port 위치를 실제 섀시 내부와 대조했다.
DIMM configuration과 population order, Fan numbering,
전·후면 구성 및 LED 안내가 한곳에 정리된 구조도 확인했다.

이 안내는 부품 위치와 정비 순서를 이해하는 기준으로 사용했지만,
사진만으로 장착된 개별 부품의 정확한 모델이나 동작 상태를 확정하지 않았다.

## 내부 구조 관찰

상판을 연 상태에서 두 개의 대형 Heatsink, 다수의 DIMM 슬롯,
Fan 모듈 열과 확장·Controller 영역의 형태를 관찰했다.
1U 섀시 안에 냉각과 확장을 위한 구성 요소가 조밀하게 배치돼 있었다.

사진에서 Heatsink 두 개가 보이더라도 실제 CPU 모델,
OS가 인식한 CPU 개수와 DIMM 총 용량은 확인되지 않았다.
Smart Array의 정확한 모델과 NIC 옵션도 확정하지 않았다.

## 섀시 작업 방식에서 느낀 차이

같은 랙 서버라도 모델과 옵션에 따라 상판의 고정·분리 방식과
내부 부품 배치가 달라질 수 있음을 관찰했다.
팀원·강사와 섀시 부품의 분리·고정 방식을 살펴봤지만,
사진만으로 해당 부품의 정확한 명칭을 단정하지 않았다.

이번 관찰을 모든 HPE 서버에 적용되는 일반 규칙으로 확대하지 않는다.
제조사 문서와 해당 장비의 정확한 구성을 별도로 확인해야 한다.

## 공개·안전 메모

섀시 내부와 서비스 라벨에는 자격증명이나 자산정보가 포함될 수 있어
촬영 자료를 공개하기 전에 세부 표식을 확인해야 한다는 점을 배웠다.
공개 사진에서는 개별 장비 식별정보, QR과 바코드를 제거했다.

섀시를 열거나 부품을 다루기 전에는 전원 상태, ESD 대책과
제조사 작업 절차를 먼저 확인해야 한다.
이번 기록은 공식 ESD·PPE 절차를 완료했다는 주장이 아니다.

## 확인하지 않은 범위

- 부팅과 운영체제 상태
- Network 연결과 NIC 동작
- Storage 인식, RAID와 Filesystem 구성
- iLO 접속과 동작
- 실제 CPU 모델과 시스템의 CPU 인식 결과
- DIMM 총 용량과 개별 메모리 상태
- Smart Array 모델과 확장 옵션
- 정확한 SKU, Firmware와 기존 사용 서버와의 구성 동일성
- 실제 장애, 수리 또는 break/fix 결과

관찰하지 않은 항목은 완료된 검증이나 장비 상태로 확대하지 않는다.

## 사진 기록

공개 사진은 장비 식별정보·QR·바코드를 검토한 2장으로 선별했다.

<table>
  <tr>
    <td width="50%">
      <img src="../evidence/approved-photos/chassis-comparison/01-dl360-gen9-access-panel-service-map.jpg" alt="DL360 Gen9 상판 내부 서비스 가이드" width="480"><br>
      <strong>01</strong><br>
      상판 내부의 DL360 Gen9 서비스 가이드에서 보드·DIMM·팬·전후면 구성 안내를 확인했다. 개별 장비 식별정보와 QR·바코드는 제거했다.
    </td>
    <td width="50%">
      <img src="../evidence/approved-photos/chassis-comparison/02-dl360-gen9-internal-layout.jpg" alt="DL360 Gen9 내부 구성 요소 배치" width="480"><br>
      <strong>02</strong><br>
      두 개의 대형 히트싱크, DIMM 슬롯, 팬 모듈과 확장 영역이 1U 섀시에 조밀하게 배치된 모습을 관찰했다. 실제 CPU 모델과 시스템 인식 결과는 확인하지 않았다.
    </td>
  </tr>
</table>
