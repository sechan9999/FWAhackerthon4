# 🛡️ FWA Detection Intelligence v4.0

### AI-Powered Healthcare Fraud, Waste & Abuse Detection System

> **7대 개선사항 통합** — LangGraph 파이프라인, Provider 네트워크, 시계열 탐지,  
> Slack/Teams 알림, SageMaker 연동, 다국어 지원
>
> MVP dashboard: https://sechan9999.github.io/FWAdetection/


---

## 🚀 Quick Start

```bash
git clone https://github.com/sechan9999/FWAhackerthon.git
cd FWAhackerthon
pip install -r requirements.txt
streamlit run app/fwa_dashboard.py
```

---

## 🏗️ 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Streamlit Dashboard (8 pages)                       │
│  🔍 Real-time │ 📊 Batch │ 🕵️ Chat │ 🕸️ Network │ 📅 Temporal │ ...│
└───────────────────────────┬─────────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│            LangGraph 5-Stage Pipeline (langgraph_pipeline.py)        │
│  [Parse] → [Rule Engine] → [AI Analysis] → [Scoring] → [Action]     │
│                                                    ↓                 │
│                                            Slack/Teams Alert         │
└───────────────────────────┬─────────────────────────────────────────┘
                            ▼
┌──────────────────┐  ┌──────────────────┐  ┌─────────────────────────┐
│   Rule Engine     │  │  OpenAI GPT      │  │  Analytics Engines      │
│   (rules.py)      │  │  (ai_analyzer.py)│  │                         │
│                   │  │                  │  │  • Provider Network     │
│  • ICD-NDC Match  │  │  • Function Call │  │    (NetworkX Graph)     │
│  • ICD Conflicts  │  │  • Medical AI    │  │  • Temporal Anomaly     │
│  • GLP-1 Rules    │  │  • Pattern Det.  │  │    (Z-score + MA)       │
│  • HCC Upcoding   │  │  • Investigator  │  │  • SageMaker Batch      │
└──────────────────┘  └──────────────────┘  └─────────────────────────┘
```

---

## ✨ v4.0 개선사항 (7개)

### 1️⃣ LangGraph 다단계 에이전트 파이프라인

5단계 StateGraph 워크플로우로 청구를 자동 검증합니다.

```
Parse → Rule Engine → AI Analysis → Risk Scoring → Action/Escalation
```

- 조건부 라우팅: 파싱 실패 시 바로 에스컬레이션
- 종합 리스크 스코어: 룰(최대 40점) + AI(최대 20점) = 100점 척도
- LangGraph 미설치 시 순차 실행 자동 전환

| Score | Level | Action |
|-------|-------|--------|
| 25+ | HIGH | BLOCK |
| 15-24 | MEDIUM | REVIEW |
| 7-14 | LOW | MONITOR |
| 0-6 | MINIMAL | APPROVE |

### 2️⃣ GitHub Actions CI/CD

Push 시 자동으로 린트, 테스트, 보안 스캔, Streamlit Cloud 배포가 실행됩니다.

```yaml
# .github/workflows/ci-cd.yml
Jobs: test (3 Python versions) → security → deploy
```

### 3️⃣ Provider 네트워크 분석

NetworkX 그래프 기반으로 의료기관 간 연결을 분석합니다.

- **환자 공유 그래프**: Provider 간 공유 환자 수로 엣지 구성
- **커뮤니티 탐지**: Greedy Modularity로 의심 클러스터 발견
- **Hub Provider**: 비정상적 연결수 (평균 + 2σ 초과)
- **의심 의뢰**: 양쪽 모두 높은 위반율 + 환자 공유
- **Betweenness Centrality**: 네트워크 매개 역할 Provider

### 4️⃣ 시계열 이상 탐지

이동 평균 + Z-score로 시간 패턴 변화를 탐지합니다.

| 탐지 유형 | 설명 |
|----------|------|
| VOLUME_SPIKE | 월별 청구 건수/금액 급증 (z ≥ 2.0) |
| EOY_UPCODING | Q4 위반율이 나머지 기간 대비 1.3배 이상 |
| WEEKEND_BILLING | 주말 위반율이 평일 대비 1.5배 이상 |
| PATTERN_SHIFT | Provider별 월간 청구 급변 (z ≥ 2.5) |

### 5️⃣ Slack/Teams 실시간 알림

Webhook으로 FWA 탐지 결과를 실시간 전송합니다.

- **Slack**: Block Kit 구조화 메시지 (Risk Badge, Claims, Actions)
- **Teams**: Adaptive Card (MessageCard 포맷)
- **배치 요약**: 전체 검증 결과 자동 알림
- **심각도 필터**: CRITICAL/WARNING/INFO 필터링

### 6️⃣ AWS SageMaker 대용량 배치

- **청크 처리**: 메모리 효율적 대용량 CSV 검증
- **진행 콜백**: Streamlit progress bar 연동
- **S3 입출력**: `upload_to_s3()`, `download_from_s3()`
- **SageMaker Job**: IAM Role 설정 시 AWS Processing Job 실행

### 7️⃣ 다국어 지원 (한국어/영어)

모든 UI 텍스트가 `engine/i18n.py`에서 중앙 관리됩니다.

```python
from engine.i18n import t, set_language
set_language("en")  # Switch to English
print(t("nav.realtime_scan"))  # "🔍 AI Real-time Scan"
```

---

## 📁 프로젝트 구조

```
FWAhackerthon/
├── .github/workflows/
│   └── ci-cd.yml                  # 🆕 GitHub Actions CI/CD
├── .streamlit/
│   └── config.toml                # 🆕 Streamlit Cloud 설정
├── engine/
│   ├── rules.py                   # 룰 기반 검증 엔진
│   ├── ai_analyzer.py             # OpenAI GPT AI 분석기
│   ├── langgraph_pipeline.py      # 🆕 LangGraph 5단계 파이프라인
│   ├── provider_network.py        # 🆕 Provider 네트워크 분석
│   ├── temporal_detector.py       # 🆕 시계열 이상 탐지
│   ├── alerts.py                  # 🆕 Slack/Teams 알림
│   ├── i18n.py                    # 🆕 다국어 지원
│   └── sagemaker_replication.py   # ⬆️ SageMaker 청크 처리 강화
├── app/
│   └── fwa_dashboard.py           # ⬆️ 8페이지 통합 대시보드
├── data/
│   └── sample_claims.csv          # 샘플 1,002건
├── tests/
│   └── test_all.py                # ⬆️ 34개 종합 테스트
├── requirements.txt
└── README.md

Total: 4,145 lines of Python code
```

---

## 🧪 테스트

```bash
pytest tests/test_all.py -v
```

```
34 passed in 4.71s

TestRuleEngine              (4 tests)  ✅
TestAIAnalyzerFallback      (3 tests)  ✅
TestSyntheticData           (3 tests)  ✅
TestLangGraphPipeline       (3 tests)  ✅
TestProviderNetwork         (3 tests)  ✅
TestTemporalDetector        (4 tests)  ✅
TestAlerts                  (5 tests)  ✅
TestI18n                    (5 tests)  ✅
TestSageMakerProcessor      (2 tests)  ✅
TestIntegration             (2 tests)  ✅
```

---

## 🔧 기술 스택

| 영역 | 기술 |
|------|------|
| AI Engine | OpenAI GPT-4o (Function Calling) |
| Pipeline | LangGraph StateGraph |
| Rule Engine | Custom Python (ICD-10, NDC, HCC) |
| Network | NetworkX (Graph + Community) |
| Temporal | Pandas + Z-score + Moving Average |
| Alerts | Slack/Teams Webhooks |
| Batch | AWS SageMaker + Chunked Pandas |
| Dashboard | Streamlit (8 pages) |
| CI/CD | GitHub Actions |
| i18n | Custom Translation Module |
| Testing | pytest (34 tests) |

---

## 📜 의존성

```
streamlit>=1.28.0    # Dashboard
pandas>=2.0.0        # Data processing
numpy>=1.24.0        # Numerical
openai>=1.12.0       # AI Analysis
langgraph>=0.2.0     # Pipeline
networkx>=3.1        # Graph Analysis
requests>=2.31.0     # Webhooks
pytest>=7.4.0        # Testing
```

---

## 👨‍💻 Author

**Gyver Chan** — Senior Data Scientist, CDC  
GitHub: [@sechan9999](https://github.com/sechan9999)
