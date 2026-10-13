# 검증 근거와 확인 범위

2026.10.13 기준으로 팀에 보관된 자료를 정리했습니다. 이번 포트폴리오 작성에서 시험을 새로 실행한 결과가 아닙니다.

## 1. 2026.10.12 배포 출처 확인

| 항목 | 확인 결과 |
| --- | --- |
| Jenkins | CQC-CICD #139 · SUCCESS |
| checkout SHA | `b5f1995a7e4a494ef116d62819b169fa9d5a788e` |
| 출처 대조 | Backend·Inference·Simulator·웹, 실행 이미지와 빌드 digest 4/4 일치 |
| 기동 상태 | Compose 서비스 7개 healthy 관찰 |

배포 버전·기동 상태 확인이며 정상 100건·장애·복구·모델 품질을 모두 승인한 결과가 아닙니다.

원본: [회차 README](../wbs/results/qa-mo-20261012-stage2/README.md), [출처 JSON](../wbs/results/qa-mo-20261012-stage2/deployment-provenance.json).

## 2. 2026.10.12 정상 100건

같은 #139 배포에서 저장 이력 100건을 읽기 전용으로 관찰했습니다. 입력이 모두 저장됐음을 독립적으로 증명하는 시험은 아닙니다.

| 항목 | 실측 | 기준 | 결과 |
| --- | --- | --- | --- |
| 완료 간격 중앙값 | 1.995초 | 2.0±0.1초 | 충족 |
| 완료 간격 p90 | 2.5258초 | 2.2초 이하 | 미충족 |
| 최대 완료 간격 | 5.755초 | 4초 미만 | 미충족 |
| 처리량 | 0.472362건/초 | 0.5±0.05건/초 | 충족 |
| 재검사 | 9/100건 · 9% | 20% 이하 | 충족 |
| 시간 초과·추론 오류 | 0건 | 0건 | 충족 |

측정 전후 컨테이너·이미지는 동일하고 healthy였습니다. 빌드 감시 12회 모두 #139를 유지했습니다. 최대 지연 구간은 09:49:30.139~09:49:35.894 KST이며 원인은 미확정입니다. QA-SIM-02·13 기준 미충족 결과를 유지합니다.

원본: [결과 설명](../wbs/results/qa-mo-20261012-stage3/README.md), [100건 JSON](../wbs/results/qa-mo-20261012-stage3/normal-100-run/normal-100.json), [기준 대조](../wbs/results/qa-mo-20261012-stage3/normal-100-run/acceptance-review.json).

## 3. 2026.10.13 SIM-04·05 격리 실환경 시험

Jenkins #147 / SHA `91e0cd0b191935d0a3359b08285421b9dfe847c1`의 Backend·Inference·Simulator 이미지와 빌드 digest 3/3을 대조했습니다. 전용 Compose 프로젝트·DB·볼륨·internal network에서 실행하고 데이터셋만 읽기 전용으로 공유했습니다.

| 케이스 | 실행 결과 |
| --- | --- |
| SIM-04 정상 revision 수정 | HTTP 200, revision 1→2, concurrency 2 |
| SIM-04 이전 revision 요청 | HTTP 409 REVISION_CONFLICT, revision·설정 유지 |
| SIM-04 현재 revision 복원 | HTTP 200, revision 2→3, concurrency 1 |
| SIM-05 잘못된 설정 10종 | 전부 HTTP 422 INVALID_SETTINGS, 설정·revision·검사 수 불변 |

Backend snapshot과 Simulator 상태를 대조했고 시험 중 jobs 0·검사 수 11을 유지했습니다. **MO 실행 기대값 충족이며 공식 QA 판정·기록표 반영은 BE 검수 후 진행합니다.**

종료 후 전용 컨테이너 4개를 중지하고 증거 볼륨을 보존했습니다. Backend는 종료 유예 후 exit 137, OOMKilled=false로 기록돼 종료 지연 원인은 미확정입니다. 운영 환경의 전후 ID·상태가 같았지만 같은 호스트 자원 공유로 인한 지연 영향의 부재까지 증명하지는 않습니다.

원본: [시험 README](../wbs/results/qa-mo-sim04-sim05-20261013/README.md), [SIM-04 대조](../wbs/results/qa-mo-sim04-sim05-20261013/sim04-checks.json), [SIM-05 대조](../wbs/results/qa-mo-sim04-sim05-20261013/sim05-checks.json).

## 4. 모델·성능 수치를 읽는 기준

모델은 DM 담당 결과이며 서비스 모델은 `cqc-apple-separate12-focal-v2-cal-20260930`입니다.

| 평가 | 품질 Macro F1 | 범위 |
| --- | --- | --- |
| 개발 5-fold Group CV | 0.8844 | 모델 선정용 개발 평가 |
| 개발 source holdout | 0.7932 | 보수적인 개발 추정치 |
| 기존 Test 27그룹 회귀 | 1.0000 | v1 실패 분석 뒤 같은 Test 재사용, 독립 승인 아님 |

모델 카드는 승인 상태를 `unverified_candidate`로 기록합니다. 회귀 1.0000을 운영 정확도 100%로 표시하지 않습니다. 추론 서버 내부 p95 309ms는 전송·Backend 처리·저장을 포함한 전체 완료 시간과 다릅니다. 초당 2건 목표는 현재 구조에서 미달이며 시연 간격은 2000ms입니다.

원본: [v2 모델 카드](../wbs/reference/DM/model-card-v2.md).

## 5. 포트폴리오 정리 시 확인 범위

- 기준 커밋의 소스·모델·문서·화면과 커밋 이력 보존
- 새 README·출처·담당·문제 해결·검증 문서의 로컬 링크·이미지 확인
- 팀 전체 기능과 홍유나 개인 담당 구분
- 과거 성공과 최신 실패, 회귀와 실환경 시험, 실행 기대값과 공식 승인 구분

전체 실행·재학습·100건 재측정·Jenkins 재배포는 이번 정리의 검증 범위에 포함하지 않습니다. 전체 ALL-04 판정과 동결 후 발표 버전 확정은 팀 작업 기록을 따라야 합니다.
