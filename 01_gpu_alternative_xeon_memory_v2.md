# ① GPU 중심 구도의 대안: Xeon 6 + 대용량 메모리 + 이기종 조합

> 일정표의 세션 제목에서 추론한 내용과, 공개된 Intel/Lenovo 기술 문서의 설치 절차를 정리한 문서입니다. 성능 수치는 환경에 따라 크게 달라지므로 **직접 측정**을 권장합니다.

---

## 1. 상황: 왜 CPU와 메모리를 내세우는가

### 1.1 배경
- 대규모 모델 **학습**은 고성능 GPU가 압도적이라 GPU 성능 경쟁에서 Intel이 두각을 보이기 어렵습니다.
- 반면 기업 현장의 AI는 **추론, RAG, 에이전트 운영** 비중이 커지고 있고, 여기서는 "가장 빠른 칩"보다 **비용, 전력, 기존 인프라 재활용, 데이터 보안**이 더 중요해집니다.
- 에이전트 워크로드는 LLM 호출 외에도 검색, 도구 호출, 스케줄링, 외부 시스템 연동 같은 **CPU가 잘하는 일**이 많습니다.
- 모델이 커지고 컨텍스트(KV 캐시)가 길어지면 병목은 연산보다 **메모리 용량과 대역폭**으로 이동합니다.

### 1.2 일정표 세션과의 매핑 (추론)

| 세션 | 추정 내용 |
|---|---|
| KTNF | AI 인프라·소버린 AI 밸류체인의 하드웨어 플랫폼 소개 |
| 정원엔시스 / HPE | Xeon + ProLiant 기반 엔터프라이즈 AI 인프라 구성 |
| ASUS, Supermicro, Acer/Altos | Xeon 6 기반 AI 서버, 에이전트 구동 구성 |
| Seagate MOZAIC+ | 고밀도 HDD로 AI 데이터 저장 비용 절감 |
| SambaNova | Intel CPU + SambaNova 가속기 이기종 추론 (에이전틱 AI) |
| Panmnesia | CXL 3.2 스위치 실리콘으로 메모리 풀링·확장 |
| Track 3 Intel 세션 | 에이전틱 AI 시대 CPU와 이기종 플랫폼의 중요성 |
| NH농협은행 사례 | B70급 GPU 기반 영상통화 신원확인 (GPU도 병행) |

### 1.3 핵심 기술 요소

| 요소 | 설명 |
|---|---|
| **Intel AMX** | Xeon의 행렬 연산 가속 명령어(BF16/INT8). CPU에서 LLM·임베딩 추론 속도를 끌어올림 |
| **AVX-512** | 벡터 연산 확장. 전처리·소규모 모델에 활용 |
| **다채널 DDR5 / MRDIMM** | 서버 CPU는 채널 수가 많아 메모리 용량·대역폭이 큼. LLM 디코딩은 메모리 대역폭 의존적 |
| **CXL (Compute Express Link)** | PCIe 기반 인터커넥트. 메모리를 서버 밖으로 확장하거나 여러 서버가 풀링해 공유하는 구조 |
| **OpenVINO** | 모델 압축·양자화(FP16/INT8/INT4)와 CPU 최적화 추론 툴킷 |
| **이기종(Heterogeneous)** | CPU가 오케스트레이션·전처리·일부 추론, 가속기가 무거운 연산을 분담 |

---

## 2. 방향: 어떤 워크로드에 맞는가

### 2.1 잘 맞는 경우
- 7B~14B급 모델, 또는 양자화한 모델의 **사내 챗봇, RAG, 문서 요약, 에이전트**
- 동시 사용자가 많지 않거나 지연시간 요구가 엄격하지 않은 **내부 업무용** 서비스
- 기존 서버를 재활용해 **GPU 신규 구매 없이** PoC를 하려는 경우
- 데이터를 외부로 내보내지 못하는 **폐쇄망·소버린** 환경
- 대용량 컨텍스트나 벡터 DB를 메모리에 올려야 하는 경우

### 2.2 불리한 경우
- 대규모 모델 학습·파인튜닝, 수십 B 이상 모델의 저지연 서비스
- 다수 사용자 동시 접속의 높은 토큰 처리량 요구
- 이미지·영상 생성처럼 GPU 우위가 큰 워크로드

### 2.3 도입 판단 기준
1. 목표 지연시간(첫 토큰까지의 시간, 초당 토큰 수)과 동시 사용자 수
2. 모델 크기와 양자화 허용 범위(정확도 손실 검증 필요)
3. 총소유비용(서버, 전력, 냉각, 라이선스)을 GPU 구성과 같은 조건으로 비교
4. 운영 인력과 소프트웨어 스택 성숙도 (vLLM, OpenVINO 등)

---

## 3. 실습: Xeon CPU에서 LLM 추론 서버 구축

> 아래는 공개 문서(Lenovo Press의 Xeon 6 생성형 AI 가이드, Intel 커뮤니티 OpenVINO 블로그)에 나온 방식을 요약한 것입니다. 버전과 옵션은 자주 바뀌므로 실행 전 각 저장소의 최신 문서를 확인하세요.

### 3.0 준비물
- **서버/PC**: AMX를 지원하는 Intel Xeon(4세대 Sapphire Rapids 이상, 6세대 권장). 개인 PC로도 OpenVINO 실습은 가능하지만 AMX 효과는 볼 수 없음
- **OS**: Ubuntu 22.04 / 24.04
- **메모리**: 8B 모델 BF16 기준 모델 가중치만 약 16GB. 여유 있게 64GB 이상 권장
- **저장공간**: 100GB 이상 (모델, Docker 이미지)
- **네트워크**: Hugging Face 모델 다운로드용

### 3.1 CPU 기능 확인
```bash
lscpu | grep -E "Model name|Flags" | grep -o -E "Model name.*|amx[a-z_0-9]*|avx512[a-z_0-9]*" | sort -u
# amx_bf16, amx_int8, amx_tile 이 보이면 AMX 지원
```

NUMA 구성도 확인합니다. 멀티 소켓·SNC 환경에서는 성능에 큰 영향을 줍니다.
```bash
lscpu | grep -i numa
numactl --hardware
```

### 3.2 방법 A: OpenVINO GenAI (가장 간단, 단일 프로세스)
```bash
sudo apt update && sudo apt install -y python3-venv git
python3 -m venv ~/ov-env && source ~/ov-env/bin/activate
pip install -U pip
pip install "optimum[openvino]" openvino-tokenizers openvino-genai huggingface_hub
```

모델을 OpenVINO IR로 변환하며 INT4로 양자화합니다. (모델 이름은 예시이며, 라이선스 동의가 필요한 모델은 `huggingface-cli login` 필요)
```bash
optimum-cli export openvino \
  --model meta-llama/Llama-3.1-8B-Instruct \
  --weight-format int4 \
  ./llama31-8b-int4-ov
```

추론 테스트:
```python
import openvino_genai as ov_genai

pipe = ov_genai.LLMPipeline("./llama31-8b-int4-ov", "CPU")
print(pipe.generate("Xeon CPU에서 LLM을 돌릴 때의 장점을 3줄로 설명해줘", max_new_tokens=200))
```

> 이미 변환된 모델은 Hugging Face의 `OpenVINO` 조직에서 받을 수 있어 변환 단계를 건너뛸 수 있습니다.

### 3.3 방법 B: OpenVINO Model Server (OVMS, 서비스형)
Docker로 REST API 서버를 띄우는 방식입니다. Intel 블로그에서 안내하는 구조는 변환된 모델 디렉터리를 마운트하고 `--target_device CPU`로 서빙하는 형태입니다. 정확한 옵션은 OVMS 공식 퀵스타트를 따르세요.
```bash
docker pull openvino/model_server:latest
# 이후 모델 디렉터리 마운트 및 설정은 OVMS 문서의 LLM 서빙 가이드 참고
```

### 3.4 방법 C: vLLM CPU 백엔드 (다중 사용자, OpenAI 호환 API)
Lenovo 문서의 방식을 요약하면, vLLM 소스를 받아 AMX/AVX 옵션을 켜고 CPU용 이미지를 빌드합니다.
```bash
# Docker 설치 (공식 문서 참고: https://docs.docker.com/engine/install/)
git clone https://github.com/vllm-project/vllm.git
cd vllm

sudo docker buildx build --platform=linux/amd64 \
  --build-arg VLLM_CPU_AMXBF16=1 \
  --build-arg VLLM_CPU_AVX512=1 \
  --build-arg VLLM_CPU_AVX512BF16=1 \
  --build-arg VLLM_CPU_AVX512VNNI=1 \
  --target vllm-openai -t vllm-cpu-amx \
  -f docker/Dockerfile.cpu --load .
```

실행 (포트 8000, 모델은 예시):
```bash
docker run -it --rm --name vllm-amx -p 8000:8000 \
  -e HF_TOKEN=<본인 토큰> \
  vllm-cpu-amx meta-llama/Llama-3.1-8B-Instruct
```
> 실행 옵션(KV 캐시 크기, 코어 바인딩, dtype 등)은 vLLM CPU 문서를 확인하세요. 소스 빌드 옵션은 vLLM 버전에 따라 바뀔 수 있습니다.

호출 테스트:
```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"meta-llama/Llama-3.1-8B-Instruct",
       "messages":[{"role":"user","content":"안녕, 자기소개 해줘"}],
       "max_tokens":100}'
```

### 3.5 RAG 에이전트까지 확장
Lenovo 문서에서는 같은 CPU 서버에서 **임베딩 생성**과 **LLM 서빙**을 함께 돌려 RAG 에이전트를 구성했고, LangChain 클라이언트로 로컬 vLLM 엔드포인트에 연결하는 방식을 보여줍니다.
```python
# pip install langchain-openai
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    base_url="http://localhost:8000/v1",
    api_key="dummy",
    model="meta-llama/Llama-3.1-8B-Instruct",
)
print(llm.invoke("사내 규정 문서를 요약하는 에이전트를 설계하는 방법은?").content)
```

### 3.6 벤치마크 방법 (직접 측정)
| 지표 | 의미 | 측정 방법 |
|---|---|---|
| TTFT | 첫 토큰까지 시간 | 요청 후 첫 스트림 토큰 수신 시간 |
| TPOT / tokens/s | 토큰당 시간, 초당 토큰 | 스트리밍 응답 길이 / 시간 |
| 동시 처리량 | 동시 N명일 때 총 tokens/s | 부하 도구(예: vLLM 벤치마크 스크립트) |
| 전력 | 부하 시 소비 전력 | 서버 BMC/PDU 또는 `turbostat` |

비교는 **같은 모델, 같은 양자화 수준, 같은 프롬프트 길이, 같은 동시 사용자 수**로 GPU 구성과 해야 공정합니다.

---

## 4. CXL 메모리 확장 이해 (Panmnesia 세션 대비)

> 이 절은 공개된 일반 지식 수준의 정리이며, Panmnesia 세션의 구체적 내용은 아닙니다.

- **문제**: GPU/서버의 메모리 용량은 슬롯과 비용에 묶여 있고, 대형 모델·긴 컨텍스트·대형 벡터 DB는 메모리가 부족해지기 쉽습니다.
- **CXL 접근**: 메모리를 PCIe 계열 링크로 **확장·풀링**하여 여러 호스트가 필요한 만큼 쓰게 하는 구조. 스위치 칩이 중간에서 연결을 맡습니다.
- **트레이드오프**: 로컬 DRAM보다 **지연시간이 길고** 대역폭도 다릅니다. 그래서 "뜨거운 데이터는 로컬 DRAM, 차가운 데이터는 CXL 메모리"처럼 **계층화**하는 소프트웨어 지원이 중요합니다.
- **AI와의 연결**: 대형 KV 캐시, 임베딩 테이블, 벡터 DB 같은 메모리 집약 데이터를 확장하는 용도로 거론됩니다.

### 현장에서 확인할 질문
- CXL 3.2 스위치 실리콘의 현재 상용화 단계와 지원 호스트 플랫폼(Xeon 6 등)
- 로컬 DRAM 대비 실측 지연시간·대역폭
- 소프트웨어 계층화(OS, 런타임, 프레임워크) 지원 수준
- 실제 AI 워크로드(LLM 서빙, 벡터 DB)에서의 효과 수치와 측정 조건

---

## 5. 이기종 조합 (SambaNova 세션 대비)

- 구조 추정: 에이전트 오케스트레이션·전처리·도구 호출은 **Xeon CPU**, 모델의 무거운 추론은 **SambaNova 가속기**가 분담
- 확인할 점: 모델 분할 방식, 대상 모델 크기, 소프트웨어 스택(API 호환성), 비교 대상(GPU 서버) 및 조건

---

## 6. 정리: 실무 관점 결론

| 관점 | 평가 |
|---|---|
| 비용 | 기존 서버 재활용 시 PoC 비용 낮음. 대규모 서비스는 TCO 비교 필수 |
| 성능 | 소·중형 모델, 중소 규모 동시 사용자는 가능. 대형 모델·고처리량은 GPU 우위 |
| 보안 | 폐쇄망·소버린 운영에 유리 |
| 운영 | OpenVINO, vLLM 등 스택이 빠르게 변하므로 버전 고정과 검증 필요 |

**권장 순서**: ① 보유 서버에서 `lscpu`로 AMX 확인 → ② OpenVINO GenAI로 소형 모델 테스트 → ③ vLLM CPU로 동시 사용자 부하 테스트 → ④ 업무 시나리오(RAG)에서 품질·지연 확인 → ⑤ GPU 구성과 TCO 비교.

---

## 7. 참고 자료
- Lenovo Press, *Implementing Generative AI Using Intel Xeon 6 CPUs* (lp2249)
- Lenovo Press, *Implementing AI Agents without GPUs: High-Performance Inference on Intel Xeon 6* (lp2406)
- Intel Community Blog, *Efficient AI Inference on CPUs with OpenVINO*
- OpenVINO: https://github.com/openvinotoolkit/openvino
- vLLM: https://github.com/vllm-project/vllm