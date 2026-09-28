# Macro → Stock Forecast Engine

한국/미국 거시환경 지표를 바탕으로 시장 국면과 3·6·12개월 방향성, 종목별 상승/횡보/하락 확률을 추정하는 Streamlit 연구용 앱입니다.

> **주의:** 통계·머신러닝 기반 연구 도구입니다. 예측 확률은 보장된 결과가 아니며, 투자 의사결정의 유일한 근거로 사용하지 마세요.

## 기능

- **시장:** 한국(KOSPI/KOSDAQ) · 미국(S&P 500 / Nasdaq 100)
- **매크로 데이터:** FRED (미국) · ECOS (한국) · yfinance (가격)
- **예측:** HistGradientBoosting + (옵션) 확률 Calibration
- **검증:** Walk-forward(엠바고) 백테스트 및 OOS 지표
- **종목:** 시장별 대표 종목 상승/횡보/하락 확률

## 프로젝트 구조

```
app.py                 # Streamlit 엔트리포인트
requirements.txt
core/
  __init__.py
  pipeline.py          # 데이터 수집 → feature → 학습 → 예측
  features.py          # 매크로/가격 feature & 라벨
  backtest.py          # Walk-forward 백테스트
  settings.py          # API 키 (Secrets / 환경변수 / 입력)
models/
  __init__.py
  classifier.py        # 분류 모델
providers/
  __init__.py
  fred.py              # FRED API
  ecos.py              # ECOS API (테이블 자동 검색)
  market.py            # yfinance 가격 & 시장 설정
```

**중요:** `core/`, `models/`, `providers/` 각 폴더에 `__init__.py`가 있어야 `from core.pipeline import ...` 가 동작합니다.  
Streamlit Cloud / GitHub에 올릴 때 이 폴더 구조를 그대로 유지하세요. 파일을 루트에 평평하게 올리면 `ModuleNotFoundError`가 발생합니다.

## 로컬 실행

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
streamlit run app.py
```

## API 키

| 용도 | 발급처 | 설정 방법 |
|------|--------|-----------|
| 미국 매크로 | [FRED API Key](https://fred.stlouisfed.org/docs/api/api_key.html) | 사이드바 입력 또는 Secrets |
| 한국 매크로 | [ECOS Open API](https://ecos.bok.or.kr/api/#/) | 사이드바 입력 또는 Secrets |

### Streamlit Cloud Secrets 예시

앱 설정 → Secrets에 다음을 추가합니다.

```toml
FRED_API_KEY = "your_fred_key"
ECOS_API_KEY = "your_ecos_key"
```

환경변수 `FRED_API_KEY`, `ECOS_API_KEY`도 지원합니다. 코드에 키를 하드코딩하지 마세요.

## Streamlit Cloud 배포

1. 이 저장소를 GitHub에 push
2. [share.streamlit.io](https://share.streamlit.io) 에서 해당 repo 연결
3. Main file path: `app.py`
4. Secrets에 API 키 등록 후 Deploy

배포 후 `ModuleNotFoundError: core.pipeline` 가 나오면, GitHub에 `core/__init__.py` 등 패키지 폴더가 제대로 올라갔는지 확인하세요.

## 데이터 구성

- **미국:** FRED + yfinance  
- **한국:** ECOS(통계표 자동 검색) + yfinance  

ECOS는 테이블 코드 하드코딩 대신 키워드 검색을 사용합니다. 실제 운용 전에 각 항목이 의도한 지표인지 검증하세요.  
경제지표는 발표·수정 시차가 있으므로, 완전한 Point-in-Time 연구에서는 vintage 데이터를 쓰는 것이 이상적입니다.

## 면책

본 시스템은 통계적/머신러닝 모델의 불확실성을 포함합니다. 과거 성과는 미래 성과를 보장하지 않습니다. 모델 오류, 데이터 지연·수정, 구조적 변화(regime shift), 거래비용 및 유동성은 별도로 고려해야 합니다.
