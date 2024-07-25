- 로컬에서 대형 언어 모델(LLM)을 쉽게 사용할 수 있도록 지원하는 플랫폼 [ollma](https://github.com/ollama/ollama/blob/main/README.md#quickstart)
- 설치 환경: MAC, Linux, WSL2, Windows (preview)
- 실행 환경: 7Bmodel (8G ram 이상)
	- 7B model (8G ram 이상)
	- 13B model ((16G ram 이상)
	- 33B model(32G ram 이상)

## 설치방법
1. 리눅스 설치
```
	# 리눅스 쉘에서
	curl https://ollama.ai/install.sh | sh
	# 올라마 설치 됐는지 확인
	ollama
	# 올라마 서버 실행
	ollama serve
	# 에러발생시( address already in use)
	 systemctl stop ollama .
	# 다른쉘에서 LLM 실행
	ollama run mistral # mistral LLM실행시
	# 사용중인 LLM 모델 나열
	ollama list
```
올라마 key
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGizpfm6tf2VuVCzaZU25nBbXCQlROwd5tXkMlKp7PVI
```
2. MAC 설치
```
	# MAC에
	brew install ollama
	# 올라마 설치 됐는지 확인
	ollama
	# 올라마 서버 실행
	ollama serve
	# 다른쉘에서 LLM 실행
	ollama run mistral # mistral LLM실행시
	ollama run llama2:13b  # Meta의 LLM모델 
	ollama run codellama:34b # 코드 생성용 LLM
	# 사용중인 LLM 모델 나열
	ollama list
```

올라마 모델 확인
	ollama show 모델명 --modelfile
## Ollama 웹UI  적용 -> 안
1. chatbot-ollama 이용
```
# chatbot-ollama 설치
git clone https://github.com/ivanfioravanti/chatbot-ollama.git
cd chatbot-ollama   
npm install
```

```
# 올라마 서버 실행
ollama serve
# chatbot-ollama 웹서버 실행
npm run dev
```
웹브라우저 주소창에 http://localhost:3000를 입력

## Ollama 로컬에서 허깅페이스 모델 사용

- 허깅페이스에서 모델 다운 ( gguf 확장자 이어야  함)
	예로 ggml-model-Q4_K_M.gguf  다운받음
	
- Model File 을 생성
	참고: [문서참고](https://github.com/ollama/ollama/blob/69f392c9b7ea7c5cc3d46c29774e37fdef51abd8/docs/modelfile.md)
	참고 Modelfile (보통은 모델명과 같은 이름으로 만듬)
```
FROM /home/moonpc/CODE/ollama_pt/ollama_pt/ggml-model-Q4_K_M.gguf

TEMPLATE """{{- if .System }}
<s>{{ .System }}</s>
{{- end }}
<s>Human:
{{ .Prompt }}</s>
<s>Assistant:
"""

SYSTEM """A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions."""

# PARAMETER TEMPERATURE 0
PARAMETER stop <s>
PARAMETER stop </s>
```

- 모델을 ollama 에 적재 하기
  예) `ollama create ggml-model-Q4_K_M -f ggml-model-Q4_K_M`
- 모델을 실행하기
  ollama run ggml-model-Q4_K_M
