---
생성날짜:
- 2026-02-01 12:50
마지막수정날짜:
- 2026-02-01-일요일 12:49
tags:
별칭:
type:
- 자료수집
Area/Reasource:
Project:
---
```python
# 변경 전  
def load_plm(plm_path: str = "./phishing_project/fine-tuned-phishing-model")  
def load_llm(adapter_path: str = "./phishing_project/gemma-phishing-adapter")  
  
# 변경 후  
def load_plm(plm_path: str = None):  
if plm_path is None:  
plm_path = str(MODELS_DIR / "koelectra_finetuned")  
...  
  
def load_llm(base_model: str = "google/gemma-2b-it", adapter_path: str = None):  
if adapter_path is None:  
adapter_path = str(MODELS_DIR / "gemma_finetuned")  
...
```

= None (기본값, Default Value):

- **가장 중요한 부분**입니다. 사용자가 함수를 호출할 때 **아무런 값도 넣지 않으면, 자동으로 `None`(비어있음)이 된다**는 뜻입니다.
- 즉, 이 값은 **필수가 아닌 선택 사항(Optional)**입니다.

이 변경은 **"코드의 안정성"**과 **"유지보수 편의성"**을 높이기 위한 아주 모범적인 리팩토링(Refactoring)입니다.

구체적으로 왜 이렇게 바꾸는지, 3가지 핵심 이유로 정리해 드립니다.

---

### 1. 실행 위치에 따른 오류 방지 (가장 큰 이유)

- **변경 전 (`"./phishing_project/..."`)**:
    
    - `./`는 **"현재 명령어를 입력하고 있는 폴더"**를 기준으로 합니다.
        
    - 만약 `C:\1.Project`에서 실행하면 잘 되지만, 실수로 `C:\`에서 실행하면 경로를 찾지 못해 에러가 발생합니다. (상대 경로의 위험성)
        
- **변경 후 (`MODELS_DIR / "..."`)**:
    
    - `MODELS_DIR` 변수는 보통 프로젝트의 **절대 경로(변하지 않는 실제 주소)**를 담고 있도록 설정됩니다.
        
    - 따라서 내가 어느 폴더에서 명령어를 입력하든, 항상 정확하게 모델 파일의 위치를 찾아갈 수 있습니다.
        

### 2. 운영체제 호환성 해결 (Windows vs Mac/Linux)

- **변경 전**:
    
    - 경로를 문자열(`str`)로 하드코딩하면 운영체제 간의 차이로 문제가 생길 수 있습니다. (예: 윈도우는 `\` 역슬래시를 쓰고, 리눅스는 `/` 슬래시를 씀)
        
- **변경 후**:
    
    - `MODELS_DIR / "koelectra_finetuned"` 형태는 파이썬의 `pathlib` 라이브러리 문법입니다.
        
    - 이 코드는 윈도우에서 실행하면 자동으로 `\`를, 맥/리눅스에서는 `/`를 사용하여 **운영체제에 맞는 경로를 자동으로 생성**해 줍니다.
        

### 3. 설정 관리의 중앙화 (유지보수)

- **변경 전**:
    
    - 모델 경로가 바뀌면 `load_plm`, `load_llm` 등 모든 함수 선언부를 일일이 찾아다니며 수정해야 합니다.
        
- **변경 후**:
    
    - `MODELS_DIR`이라는 변수 하나만 수정하면, 이를 사용하는 모든 함수(`load_plm`, `load_llm` 등)의 경로가 한꺼번에 업데이트됩니다.
        

---

### 요약

**"어디서 실행하든 에러가 나지 않게 하고(절대 경로), 윈도우/맥 상관없이 잘 돌아가게 하며(pathlib), 나중에 경로 수정도 쉽게 하기 위해(변수 사용)"** 코드를 변경한 것입니다.

이 방식이 훨씬 더 **전문적이고 안전한 코드 스타일**입니다. `MODELS_DIR`이 정의된 부분(아마 `config.py`나 상단 상수 선언부)을 보시면 경로 관리가 어떻게 되고 있는지 확인하실 수 있을 거예요.

혹시 `MODELS_DIR`이 어떻게 정의되어 있는지 확인해 드릴까요?