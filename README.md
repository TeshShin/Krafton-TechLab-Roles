# 크래프톤 정글 게임테크랩 2기 — 주차별 담당 정리

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
| W05 | 09.26–10.01 | 52 | **Frustum Culling**, **CPU HZB Occlusion Culling** | [Week05](https://github.com/TeshShin/Krafton_TechLab_Week05) |
| W06 | 10.10–10.15 | 47 | **데칼 시스템**(페이드 인/아웃 포함), **FXAA** | [Week06](https://github.com/TeshShin/Krafton_TechLab_Week06) |
| W07 | 10.16–10.22 | 46 | **PointLight**, **Normal Mapping**, 고로 셰이딩 | [Week07](https://github.com/TeshShin/Krafton_TechLab_Week07) |
| W08 | 10.23–10.30 | 26 | **PSM (Perspective Shadow Map)** | [Week08](https://github.com/TeshShin/Krafton_TechLab_Week08) |
| W09 | 10.31–11.07 | **72** | **충돌 시스템**(옥트리 브로드페이즈 + 구vs구 내로우페이즈), 오브젝트 풀, HitStop/Slomo, 카메라 트랜지션 | [TopGun](https://github.com/TeshShin/Krafton_TechLab_Week09_TopGun) |
| W10 | 11.08–11.13 | 43 | **Skeletal Mesh Viewer**, **본 기즈모·본 피킹**, 스냅 | [Week10](https://github.com/TeshShin/Krafton_TechLab_Week10) |
| W11 | 11.14–11.21 | 65 | **GPU Skinning**, **GPU 타이머 기반 성능 측정**, 미니덤프 | [Week11](https://github.com/TeshShin/Krafton_TechLab_Week11) |
| W12 | 11.22–11.30 | 58 | **파티클 코어·모듈·에디터**, 파티클 콜리전을 BVH에 연결 | [Week12](https://github.com/TeshShin/Krafton_TechLab_Week12) |
| W13 | 12.01–12.04 | 34 | **Physics Asset Editor**, 컨스트레인트 축 시각화 정렬, 래그돌 | [Week13](https://github.com/TeshShin/Krafton_TechLab_Week13) |
| Final | 12.05–12.13 | 54 | 아이템 수집 파이프라인, 시민 구조 래그돌, 사운드, 게임패드 | [Let's Go Firefighter!](https://github.com/TeshShin/Krafton_TechLab_Final) |

**합계 634 커밋**

10월 2일 ~ 9일은 추석 연휴로 진행하지 않았습니다.

---

## 특히 파고들었던 것

문제를 만나 원인을 찾고 해결한 과정이 남아 있는 항목들입니다.

### Occlusion Culling — 정확도가 안 나오던 문제 (W05)
저해상도 깊이 맵 위에 밉 피라미드(HZB)를 쌓고, 사각형 영역의 최대 깊이를 적응적으로 샘플링해 가림을 판정합니다. BVH와 이전 프레임 히스토리를 함께 쓰며, 전부 CPU에서 돕니다.
처음에는 NDC Z로 판정했는데 원근 나눗셈 때문에 깊이 분포가 비선형이라 컬링이 어색했습니다. **NDC Z를 버리고 뷰 공간 Z를 [0..1]로 선형화**해 썼습니다.

남은 문제는 **오클루더를 바운딩 박스 그대로 래스터화**한 것이었습니다. 박스는 실제 실루엣보다 크고 모서리가 비어 있어서, 가려지지 않은 물체까지 가려졌다고 판정됐습니다. 세 가지로 해결했습니다.

- 오클루더의 화면 사각형을 **중심 기준 50%로 축소**(erosion). 단 8px 미만이거나 화면의 60% 이상을 덮으면 건너뜁니다
- 깊이를 박스의 가까운 면이 아니라 **먼 면(MaxZ)으로 기록** — 박스 전체보다 확실히 뒤에 있는 것만 가려집니다
- 그리드 해상도를 **화면의 1/4에서 1/2로 상향**

앞의 둘은 판정을 보수적으로 만들어 잘못된 가림을 없애고, 마지막은 해상도를 올린 것입니다. 두 차례 롤백 후 재구현했습니다.

### PSM — W가 음수가 되는 문제 (W08)
PSM은 **카메라의 원근 변환 공간**에서 그림자 맵을 만듭니다. 그래서 광원이 정면으로 보고 있는 정점이라도, **카메라 위치에 따라** 변환 후 W가 음수가 되면서 투영이 깨졌습니다.
**음수를 양수로 바꾸는 대신 음수가 나오는 상황 자체를 회피**하도록 만들고, `W<0`인 경우에만 역투영 경로로 분기했습니다. 섀도우 아크네는 **컬링 모드를 front로** 전환해 해결했습니다.

### GPU Skinning — 성능 측정이 원인이던 메모리 누수 (W11)
CPU/GPU 스키닝을 전역 플래그로 전환하고 그림자 패스에도 적용했습니다.
성능을 재려고 붙인 **GPU 쿼리 링버퍼가 오히려 메모리 누수의 원인**이었습니다.
GPU 타이머는 스톨을 피하려고 쿼리 8개를 돌려쓰며 N-7 프레임 결과를 읽습니다. 이건 의도된 설계인데, 그 지연 때문에 **스키닝 모드를 전환하면 이전 모드의 측정값이 그대로 보였습니다.** 전환 시점을 지연에 맞춰 동기화해 해결했습니다.

### 컨스트레인트 — 엔진 시각화와 PhysX의 축 규약 차이 (W13)
Physics Asset Editor가 그리는 스윙 콘과 트위스트 부채꼴이 **PhysX 디버거에서 보이는 축과 다른 방향**을 향했습니다.
엔진의 조인트 프레임은 **Y축이 트위스트 방향**인데 PhysX는 **X축이 트위스트 축**이어서, 시각화 쪽에 `Y→X` 회전과 트위스트 축 기준 +90° 회전을 더해 맞췄습니다. 시뮬레이션이 아니라 그리기 좌표계를 맞춘 작업입니다.
본 규약도 함께 정리해 언리얼과 같이 `Bone1 = Child`, `Bone2 = Parent`가 되도록 하고, `Rotation1`은 자식 본 로컬, `Rotation2`는 부모 본 로컬 기준으로 잡았습니다.

---

## 만든 도구

엔진 작업과 별개로, 작업 속도를 위해 에디터·뷰어를 직접 만들었습니다.

- **Skeletal Mesh Viewer** (W10) — 본별 기즈모(월드/로컬), 본 피킹(큰 본 안의 작은 본까지 선택), 스냅
- **Particle Editor** (W12) — 모듈 삽입·삭제·복제, 배열 위젯 노출, 파티클 수 Min/Max/Avg Stat
- **Physics Asset Editor** (W13) — 바디 생성, 시뮬레이션, 컨스트레인트 시각화, 부피 비례 질량
- **성능 프로파일러** (W11) — GPUTimer 기반, CPU/GPU 스키닝 스왑 측정
