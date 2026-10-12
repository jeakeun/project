# 냠냠플래닛 (YummyPlanet)

신체 정보와 목표를 입력하면 **AI가 하루 식단을 구성**하고, 웹캠으로 **운동 자세를 실시간 분석**해
피드백하는 식단·운동 관리 웹 서비스입니다.

> 이 저장소는 팀 원본([YH-KIKI/YummyPlanet](https://github.com/YH-KIKI/YummyPlanet))의 **포크**입니다.
> 아래 「제가 맡은 부분」은 **심재근**이 담당한 작업이고, 나머지는 팀 공동 작업입니다.
>
> 국비 교육 「클라우드환경 자바(JAVA) 풀스택 AI융합 웹개발자」 과정의 팀 프로젝트
> · 기간 2026.04.17 – 2026.05.31

---

## 구성

React 프런트엔드 ↔ Spring Boot 백엔드 ↔ FastAPI AI 서버의 3계층 구조입니다.

```
project_react/    React 프런트엔드 (화면, API 호출)
project_java/     Spring Boot 백엔드 (kr.hi.project — controller / service / dao / dto, MyBatis)
project_python/   FastAPI AI 서버 (식단 추천, 자세 분석, 이미지 인식)
design/           화면 설계·캐릭터 등 디자인 산출물
*.sql             데이터베이스 스키마와 한식 음식 데이터
```

| 구분 | 기술 |
|---|---|
| Frontend | React, JavaScript, Axios |
| Backend | Java, Spring Boot, MyBatis, MySQL |
| AI | Python, FastAPI, scikit-learn(NearestNeighbors), MediaPipe, PyTorch / YOLO |

---

## 제가 맡은 부분 (심재근)

### 1. AI 식단 추천 알고리즘 설계·고도화

사용자의 신체 정보와 목표 데이터를 수집·가공해 추천 로직에 전달하고, **5가지 맞춤 식단**을 제공합니다.
구현은 `project_python/ai_service.py` 의 `NearestNeighbors` 기반 추천과
`main.py` 의 `POST /api/ai/recommend` 엔드포인트에 있습니다.

| 단계 | 내용 |
|---|---|
| 문제 | KNN으로 목표 칼로리에 맞춰 추천했더니 **소스나 조리되지 않은 재료가 한 끼로 반환** |
| 원인 | 추천 기준이 **칼로리 수치 하나뿐**이라 음식의 역할(밥/국/반찬)과 조리 여부가 미반영 |
| 해결 | 데이터의 **'분류명' 기반 카테고리 필터링** 추가 + 목표 칼로리에 맞춰 **메뉴별 섭취량(g)을 자동 계산·분배**하고 DB에 동적 매핑하는 하이브리드 구조로 재설계 |
| 결과 | **밥 · 국 · 메인 메뉴 · 반찬**이 갖춰진 정식 형태로 출력 |

### 2. 실시간 자세 분석 (바디첵)

웹캠 영상에서 **MediaPipe로 관절 좌표를 추출**해 자세를 분석하고 화면에 피드백합니다.
`POST /api/ai/bodycam/pose`, `POST /api/ai/bodycam/outline` 과
프런트엔드의 `src/SidebarMenu/api/bodyCheckApi.js` 가 해당 부분입니다.

| 단계 | 내용 |
|---|---|
| 문제 | 초당 수십 프레임의 분석 결과를 그대로 반영하면 **리렌더링이 연쇄 발생**해 렌더링 지연·브라우저 과부하 위험 |
| 해결 | **프레임 단위 상태 관리를 최적화**해 불필요한 리렌더링을 차단하고 데이터 처리 흐름을 정리 |
| 결과 | 지연 없는 자세 피드백 구현, 분석 결과를 백엔드로 안정적으로 동기화 |

### 3. 프런트–백엔드 API 연동

React와 Spring Boot 사이에서 **Axios로 RESTful API 엔드포인트 3개를 설계·연동**해
사용자 입력 → 가공 → AI 서버 호출 → 결과 반환의 데이터 흐름을 구성했습니다.

---

## 실행

```bash
# AI 서버
cd project_python && pip install -r requirements.txt && uvicorn main:app --reload

# 백엔드
cd project_java && ./gradlew bootRun

# 프런트엔드
cd project_react && npm install && npm start
```

> DB 접속 정보와 모델 파일 경로는 각 폴더의 설정 파일을 확인해 주세요.
> 스키마는 루트의 `냠냠(제12판).sql` 이 최신입니다.

---

**심재근** · [GitHub](https://github.com/jeakeun)
