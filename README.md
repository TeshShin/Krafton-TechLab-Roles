# 크래프톤 게임테크랩 2기 — 주차별 담당 정리

**신동민 (TeshShin)** · 2025.09 – 2026.02 · C++ · DirectX 11 · 자체 게임 엔진

매주 3~4인 팀이 새로 편성되어 자체 게임 엔진을 이어서 개발했습니다.
이 저장소는 **14주 동안 제가 실제로 구현한 것**을 주차별로 정리한 인덱스입니다.

각 주차의 저장소 README 맨 위에도 같은 내용이 있고, 모든 항목은 커밋으로 확인할 수 있습니다.

> **참고** — GitHub의 contributors 숫자는 앞 주차 히스토리가 누적되어 그 주의 기여를 나타내지 않습니다.
> 아래 커밋 수는 각 주차의 날짜 범위로 필터링한 값입니다.

---

## 주차별 담당

| 주차 | 기간 | 내 커밋 | 담당한 것 | 저장소 |
| --- | --- | ---: | --- | --- |
| W01 | 09.02–09.04 | 39 | 사운드 매니저·BGM, 스테이지 1~3, 별 수집·점수, AABB 판정 | [BounceMario](https://github.com/TeshShin/Krafton_TechLab_Week01_BounceMario) |
| W02 | 09.05–09.10 | 34 | MVP 변환, 카메라 입력, **쿼터니언 회전 체계**, 뷰포트 비율 유지 | [Week02](https://github.com/TeshShin/Krafton_TechLab_Week02) |
| W03 | 09.11–09.18 | 37 | **AABB 바운딩 박스**, 빌보드, 그리드·카메라 설정 저장/로드 | [Week03](https://github.com/TeshShin/Krafton_TechLab_Week03) |
| W04 | 09.19–09.25 | 27 | **씬 JSON 직렬화**, FName 네임테이블, 레벨 생성 경로 통일 | [Week04](https://github.com/TeshShin/Krafton_TechLab_Week04) |
| W05 | 09.26–10.01 | 52 | **Frustum Culling**, **Occlusion Culling (BVH 하이브리드)** | [Week05](https://github.com/TeshShin/Krafton_TechLab_Week05) |
| W06 | 10.10–10.15 | 47 | **데칼 시스템**(페이드 인/아웃 포함), **FXAA** | [Week06](https://github.com/TeshShin/Krafton_TechLab_Week06) |
| W07 | 10.16–10.22 | 46 | **PointLight**, **Normal Mapping**, 고로 셰이딩 | [Week07](https://github.com/TeshShin/Krafton_TechLab_Week07) |
| W08 | 10.23–10.30 | 26 | **PSM (Perspective Shadow Map)** | [Week08](https://github.com/TeshShin/Krafton_TechLab_Week08) |
| W09 | 10.31–11.07 | **72** | **충돌 시스템**(옥트리 쿼리), 오브젝트 풀, HitStop/Slomo, 카메라 트랜지션 | [TopGun](https://github.com/TeshShin/Krafton_TechLab_Week09_TopGun) |
| W10 | 11.08–11.13 | 43 | **Skeletal Mesh Viewer**, **본 기즈모·본 피킹**, 스냅 | [Week10](https://github.com/TeshShin/Krafton_TechLab_Week10) |
| W11 | 11.14–11.21 | 65 | **GPU Skinning**, **GPU 타이머 기반 성능 측정**, 미니덤프 | [Week11](https://github.com/TeshShin/Krafton_TechLab_Week11) |
| W12 | 11.22–11.30 | 58 | **파티클 코어·모듈·에디터**, **파티클 콜리전(BVH)** | [Week12](https://github.com/TeshShin/Krafton_TechLab_Week12) |
| W13 | 12.01–12.04 | 34 | **Physics Asset Editor**, 컨스트레인트 축 정렬, 래그돌 | [Week13](https://github.com/TeshShin/Krafton_TechLab_Week13) |
| Final | 12.05–12.13 | 54 | 아이템 수집 파이프라인, 시민 구조 래그돌, 사운드, 게임패드 | [Let's Go Firefighter!](https://github.com/TeshShin/Krafton_TechLab_Final) |

**합계 634 커밋**

10월 2일 ~ 9일은 추석 연휴로 진행하지 않았습니다.

---

## 특히 파고들었던 것

문제를 만나 원인을 찾고 해결한 과정이 남아 있는 항목들입니다.

### Occlusion Culling — 정확도가 안 나오던 문제 (W05)
BVH + 제한적 쿼리 + 히스토리를 결합한 하이브리드 방식으로 구현했습니다.
처음에는 NDC Z를 기준으로 판정해 깊이 분포가 비선형이라 컬링이 어색했고, **뷰포트 Z를 사용해 선형화**한 뒤 초기 그리드 해상도를 절반으로 낮춰 정확도를 확보했습니다. 두 차례 롤백 후 재구현했습니다.

### PSM — W가 음수가 되는 문제 (W08)
원근 그림자 맵에서 광원 뒤쪽 정점의 W가 음수가 되며 투영이 깨졌습니다.
**음수를 양수로 바꾸는 대신 음수가 나오는 상황 자체를 회피**하도록 만들고, `W<0`인 경우에만 역투영 경로로 분기했습니다. 섀도우 아크네는 **컬링 모드를 front로** 전환해 해결했습니다.

### GPU Skinning — 성능 측정이 원인이던 메모리 누수 (W11)
CPU/GPU 스키닝을 전역 플래그로 전환하고 그림자 패스에도 적용했습니다.
성능을 재려고 붙인 **GPU 쿼리 링버퍼가 오히려 메모리 누수의 원인**이었고, GPU 타이머가 **8프레임 이전 결과를 읽는** 문제도 동기화로 해결했습니다. 측정 도구가 측정 대상을 망가뜨리고 있던 경우였습니다.

### 컨스트레인트 — PhysX와 언리얼의 축 규약 차이 (W13)
Physics Asset Editor의 스윙/트위스트 방향이 언리얼과 다르게 표시됐습니다.
언리얼 방식(자식 1 / 부모 2)에 맞춰 **부모와 자식의 축을 정렬**해 PhysX와 동일한 결과가 나오도록 정리했습니다.

---

## 만든 도구

엔진 작업과 별개로, 작업 속도를 위해 에디터·뷰어를 직접 만들었습니다.

- **Skeletal Mesh Viewer** (W10) — 본별 기즈모(월드/로컬), 본 피킹(큰 본 안의 작은 본까지 선택), 스냅
- **Particle Editor** (W12) — 모듈 삽입·삭제·복제, 배열 위젯 노출, 파티클 수 Min/Max/Avg Stat
- **Physics Asset Editor** (W13) — 바디 생성, 시뮬레이션, 컨스트레인트 시각화, 부피 비례 질량
- **성능 프로파일러** (W11) — GPUTimer 기반, CPU/GPU 스키닝 스왑 측정

---

## 관련 링크

- **[FleshRing](https://github.com/TeshShin/FleshRing)** — 언리얼 엔진 플러그인, FAB 마켓플레이스 출시
- **[무너진 성채](https://github.com/nansu0425/nan2026-game)** — NAN 2026 예선 과제, Unity WebGL
