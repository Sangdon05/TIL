Kaggle에서 제공하는 Mistral 모델을 외부(로컬 PC 또는 다른 서버)에서 Ollama를 통해 사용하려면, ==Kaggle 노트북 환경에 Ollama 서버를 띄우고 이를 외부로 터널링하여 API로 호출하는 방식==이 가장 효율적입니다. [1, 2]

Kaggle의 무료 GPU(T4 또는 P100)를 활용해 로컬 PC 터미널이나 코드에서 Mistral을 원격으로 제어하는 핵심 단계를 정리해 드립니다. [1, 3]

---

## 1단계: Kaggle 노트북 설정 (서버 가동)

Kaggle Notebook을 새로 생성하고, 우측 설정(Settings)에서 Accelerator를 GPU(T4 x2 또는 P100)로 지정합니다. 이후 노트북 셀에 아래 코드를 순서대로 입력하고 실행합니다.

## 1. Ollama 설치

```bash
!curl -fsSL https://ollama.com/install.sh | sh
```

## 2. Ollama 백그라운드 서버 실행

```python
import subprocess
import time

# Ollama 서버를 백그라운드에서 실행
process = subprocess.Popen("nohup ollama serve > ollama.log 2>&1 &", shell=True)
time.sleep(5) # 서버가 켜질 때까지 잠시 대기
```

## 3. 외부 연결을 위한 Ngrok 터널링 설정

Kaggle 내부 포트(`11434`)를 외부에서 접속할 수 있도록 주소를 만들어야 합니다. [ngrok 공식 홈페이지](https://ngrok.com/)에 가입 후 발급받은 Authtoken이 필요합니다.

```bash
!pip install pyngrok

from pyngrok import ngrok

# 본인의 ngrok Authtoken을 입력하세요
NGROK_TOKEN = "YOUR_NGROK_AUTH_TOKEN"
ngrok.set_auth_token(NGROK_TOKEN)

# 11434 포트를 외부로 개방
public_url = ngrok.connect(11434, "http")
print("외부 접속용 Ollama 주소:", public_url.public_url)
```

> 💡 실행 결과로 나오는 `http://ngrok-free.app` 형태의 주소를 복사해 둡니다. [1]

---

## 2단계: 외부(로컬 PC)에서 접속 및 모델 실행

이제 Kaggle에서 열어준 터널을 통해 로컬 PC에서 Mistral 모델을 제어할 수 있습니다. [1]

## 방법 A: 로컬 터미널(CLI)에서 바로 사용하기

로컬 PC의 터미널(Mac/Linux)이나 명령 프롬프트(Windows)를 열고 아래 명령어를 입력합니다.

```bash
# 환경 변수에 Kaggle ngrok 주소 등록
export OLLAMA_HOST="http://ngrok-free.app"  # Windows (CMD)의 경우: set OLLAMA_HOST=http://ngrok-free.app

# Mistral 모델 다운로드 및 실행 명령어 입력
ollama run mistral
```

- 최초 실행 시 Kaggle 서버가 Ollama 레지스트리로부터 `mistral` 모델을 자동으로 다운로드하여 가동합니다. [1, 4]

## 방법 B: Python 코드로 호출하기 (LangChain 등 활용)

로컬 프로젝트 코드에서 API 형태로 Kaggle에 구동 중인 Mistral을 불러올 수 있습니다. [5]

```python
import requests

# Kaggle ngrok 주소 입력
OLLAMA_URL = "http://ngrok-free.app"

response = requests.post(
    f"{OLLAMA_URL}/api/generate",
    json={
        "model": "mistral",
        "prompt": "Kaggle 외부에서 Ollama를 통해 전하는 인사말을 작성해줘.",
        "stream": False
    }
)
print(response.json()['response'])
```

---

## ⚠️ 주의 및 팁

- Kaggle 세션 제한: Kaggle 노트북은 브라우저를 닫거나 무반응 상태가 지속되면 몇 시간 내에 세션이 종료됩니다. 장시간 서빙이 필요하다면 노트북 우측 상단의 'Save Version' -> 'Save and Run All (Commit)'으로 백그라운드 실행을 걸어두는 것이 좋습니다. (최대 12시간 유지)
- Mistral 버전 선택: 일반 `mistral`(7B) 외에도 고성능인 `mistral-small` 등을 사용하고 싶다면 Ollama 명령어에 모델명을 명시하면 됩니다 (예: `ollama run mistral-small`). [6, 7]

원격 터널링 과정에서 ngrok 인증 오류가 발생하거나 다른 Mistral 파라미터 버전(예: 7B, 24B)을 올리고 싶으시다면 언제든 말씀해 주세요!

  

[1] [https://medium.com](https://medium.com/data-science-collective/create-a-remote-llm-server-using-kaggle-notebooks-and-ollama-acb299ead1e5)

[2] [https://www.reddit.com](https://www.reddit.com/r/ollama/comments/1iqqq0e/ollamaremote_make_local_ollama_run_models_on/?tl=ko)

[3] [https://www.kaggle.com](https://www.kaggle.com/code/xrouge/ollama)

[4] [https://medium.com](https://medium.com/@abdul_hadi/integrating-ollama-mistral-into-your-web-application-with-commands-code-examples-b9c665c45bc0)

[5] [https://www.kaggle.com](https://www.kaggle.com/code/samratabduljalil/local-rag-with-ollama-mistral-nomic-embedding)

[6] [https://news.hada.io](https://news.hada.io/topic?id=18989)

[7] [https://ollama.com](https://ollama.com/library/mistral-small)