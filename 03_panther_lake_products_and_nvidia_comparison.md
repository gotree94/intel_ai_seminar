# Panther Lake 제품 현황과 NVIDIA 대비 위치

> 공개된 기사, Intel 보도자료·성능 문서, NVIDIA 문서를 바탕으로 정리했습니다. 가격은 대부분 **출시 초기 기준**이고 이후 달라졌을 수 있습니다. Intel의 성능 비교 수치는 **Intel이 직접 측정한 값**이므로 참고용으로 보세요.
>
> 관련 문서: `00_intel_ai_seminar_summary.md`, `01_gpu_alternative_xeon_memory.md`, `02_physical_ai_studio.md`

---

## 1. Panther Lake란

- 정식 명칭은 **Intel Core Ultra Series 3**이며 Intel 18A 공정 기반입니다.
- CPU + iGPU(Xe3) + NPU가 한 SoC에 들어 있는 이기종 구조입니다.
- Intel 자료 기준 최대 180 TOPS(플랫폼 합계), 최대 154GB/s 메모리 대역폭을 내세웁니다.

### 1.1 칩 라인업 (노트북 기준)

| 구분 | 모델 | 내장 GPU | 대상 |
|---|---|---|---|
| 상위 | Core Ultra X9 388H, X7 368H/358H | Arc B390 (Xe3 12코어) | dGPU 없는 고성능 울트라포터블 |
| 중상위 | Core Ultra 9 386H, 7 366H/356H, 5 338H/336H | 일부 구성 | 단독 또는 dGPU 결합 |
| 일반 | Core Ultra 7 365/355, 5 335/325 | 4코어 수준 | 울트라북 |
| 보급 | **Core Series 3** | 4코어 수준 | 가성비 PC, 상업용, "필수 엣지" |

> Core Series 3는 Panther Lake 구조를 바탕으로 만든 **보급형 라인**입니다. 일반 PC뿐 아니라 로보틱스, 스마트 빌딩, POS, 스마트 미터링 같은 엣지도 겨냥합니다.

---

## 2. 노트북

### 2.1 국내 판매 제품

| 제조사 | 제품 | 비고 |
|---|---|---|
| 삼성 | 갤럭시 북6 울트라(16형), 갤럭시 북6 프로(14/16형) | 1월 27일 국내 출시, NPU 최대 50 TOPS |
| 삼성 | 갤럭시 북6 엔터프라이즈 에디션 | vPro 탑재, 5월 출시 예고 |
| LG | 그램 프로 AI(16/17형), 그램 프로 360 AI, 그램 AI(14/15형), 그램북 AI | 총 7종 순차 출시, 16형 그램 프로 AI 최대 혜택가 208만 원대 |
| 기타 | 기가바이트, 델, 레노버, 에이서, 에이수스, HP, MSI | 인텔 쇼케이스에 30여 종 전시 (국내 판매 여부는 모델별 확인) |

- 갤럭시 북6 울트라 사용기 구성: Core Ultra 7 356H + **엔비디아 RTX 5070** + 32GB + 2TB. 외장 GPU가 있어 인텔 NPU·iGPU 실습 목적과는 결이 다릅니다.

### 2.2 해외 중심 제품 (출시 초기 가격)

| 제품 | 사양 | 가격 |
|---|---|---|
| MSI Prestige 14 Flip AI+ Evo | Ultra X7 358H, 14형 OLED, 32GB, 1TB | 약 1,299달러 |
| HP OmniBook Ultra | Ultra 7 356H, 32GB | 약 1,549달러 |
| Dell XPS 14 / 16 | Series 3 + Arc 내장 그래픽 선택 가능 | 구성별 |
| ASUS Zenbook S 14 | Ultra 7 355, 16GB, 512GB, 1.2kg | 구성별 |
| Lenovo | Ultra 7 355, 32GB LPDDR5X-7467, 1TB | 구성별 |

---

## 3. 미니 PC

| 제품 | 칩 / 메모리 | 특징 |
|---|---|---|
| GMKtec EVO-T2 | Core Ultra X7 358H, Arc B390 | Core Ultra 300 최초 판매 미니 PC, 독일 1,649유로부터 |
| Khadas Mind Pro | X7 358H (65W), 최대 64GB LPDDR5X, 최대 4TB | 휴대성 좋은 소형 본체, 전용 그래픽 독 있음 |
| AceMagic F2A | X7 358H, 32GB, 1TB SSD | 약 0.6L, USB4·2.5G 이더넷, 약 1,100달러 |
| ASRock Industrial NUC Ultra 300 BOX | Ultra 5 325 또는 X7 358H 베어본 4종 | 메모리·SSD 직접 장착, 산업용 라인 |
| Asus NUC 16 Pro | Core Ultra 300 기반 | 초기 Panther Lake 미니 PC로 발표 |
| Gigabyte Brix GB-BRU9-386H | Core Ultra 9 386H | 출시 준비 보도 (현재 판매 여부 미확인) |

---

## 4. 실습용 선택 가이드 (Physical AI Studio 기준)

| 항목 | 권장 | 이유 |
|---|---|---|
| 칩 | **X7 358H / X9 388H** (Arc B390, Xe3 12코어) | iGPU 성능이 확연히 높음. Ultra 5 325급은 GPU 4코어로 약함 |
| 메모리 | **32GB 이상** | CPU·iGPU·NPU가 공유, 모델 변환·데이터셋 처리에 16GB는 빠듯. 온보드(납땜)가 많아 사후 증설 불가 |
| 저장공간 | 1TB 이상 | 로봇 시연 영상 데이터셋 |
| 외장 GPU | 없는 쪽 | 인텔 NPU·iGPU 체험이 목적일 때 |
| OS | Ubuntu 24.04 (HWE 커널) | Intel 해커톤 가이드의 대상 환경 |
| 폼팩터 | 이동 개발은 노트북, 로봇 옆 고정은 미니 PC | 용도에 따라 선택 |

### 구매 전 확인할 것
- [ ] 같은 모델의 **Ubuntu 24.04 설치 후기**와 NPU 인식 여부 (모델별로 확인하지 못함)
- [ ] 메모리 구성(온보드 여부), SSD 교체 가능 여부
- [ ] 국내 정식 유통, AS, 전원 어댑터 규격 (직구 시)
- [ ] 세미나 핸즈온에서 쓰는 보드 사양 (현장 확인 후 결정해도 늦지 않음)

### 한계
- 노트북·미니 PC에서는 **추론과 소규모 정책 학습**까지가 현실적입니다. 큰 VLA 미세조정은 별도 GPU 서버가 필요합니다.

---

## 5. Intel의 전략: Jetson과 직접 경쟁하려는가

**결론(추정)**: 겨냥하는 상대는 Raspberry Pi가 아니라 **Jetson(AGX Orin, Thor)**이고, 칩도 최하위가 아니라 **중상위(X7/X9 H)** 입니다.

### 근거
- Intel은 X7 358H가 로보틱스 VLA에서 Jetson AGX Orin보다 1.5배 빠르다고 주장합니다.
- Jetson AGX Orin과 Jetson Thor T5000에 대해 중간 크기 VLA 기준 비용·성능·가치 우위를 보여주는 비교 차트를 공개했습니다.
- π0.5 VLA 최적화 문서에서 Panther Lake를 다른 임베디드 AI 플랫폼과 최대 TDP 및 40W 동일 전력 조건으로 비교했습니다.
- "크고 비싸고 뜨거운 외장 GPU를 대체한다"는 메시지를 반복합니다.
- Raspberry Pi급은 전력·가격 구간이 달라 직접 경쟁으로 보기 어렵습니다. 다만 최하위 Core Series 3의 "필수 엣지" 겨냥은 저가 엣지와 일부 겹칠 수 있습니다. (Intel이 Pi를 직접 거론한 자료는 찾지 못함)

### Intel이 내세우는 차별점
| 항목 | 내용 |
|---|---|
| x86 호환성 | Windows/Linux, 기존 소프트웨어 자산 |
| 산업용 사양 | 확장 온도, ECC 메모리, 최대 10년 공급, 장기 지원 OS |
| 실시간성 | TCC, TSN 기반 결정적 실행 |
| 추론 스택 | OpenVINO로 CPU·GPU·NPU 분담 |
| 개발 도구 | Physical AI Studio, OpenVINO Physical AI, Robotics AI Suite |

### 주의할 점
- 비교 조건이 다릅니다. Intel 측정은 2026년 3월 노트북(HP OmniBook X 16, Ubuntu 24.04, OpenVINO 2026.0), Jetson 측은 2025년 11월 TensorRT 기준이며 각각 최대 전력입니다.
- VLA 추론은 메모리 대역폭의 영향이 크므로 Intel의 154GB/s를 Jetson 모델별 사양과 직접 비교해야 합니다.
- Jetson은 CUDA/TensorRT 생태계, 레퍼런스, 커뮤니티에서 앞섭니다. 현실적인 포지션은 "x86 자산이 있거나 장기 공급·산업 인증이 중요한 쪽의 새 선택지"입니다.

---

## 6. Intel 도구 ↔ NVIDIA 도구 대응

| 워크플로 단계 | Intel | NVIDIA |
|---|---|---|
| 시뮬레이션·환경 | (MuJoCo SO-101 플러그인 정도) | Isaac Lab-Arena, Isaac Lab |
| 시연 데이터 수집 | Physical AI Studio | Isaac Teleop |
| 데이터 형식 | LeRobot | LeRobot |
| 모델·미세조정 | Physical AI Studio (ACT, Pi0, SmolVLA, GR00T, Pi0.5) | Isaac GR00T (N 시리즈) |
| 최적화·내보내기 | OpenVINO, ONNX, Torch | ONNX, TensorRT, LEAPP 번들 |
| 배포·실행 | OpenVINO Physical AI | Isaac ROS + Jetson Thor |

### NVIDIA 레퍼런스 파이프라인 (Unitree G1 예시)
1. XR 헤드셋으로 텔레오퍼레이션
2. 시연 에피소드를 MCAP 로그로 기록
3. MCAP → LeRobot 데이터셋 변환
4. GR00T N 미세조정
5. LEAPP으로 정책 내보내기
6. Jetson AGX Thor 등에서 Isaac ROS로 배포

### GPU 요구사항 (GR00T 문서 기준)
- 추론: VRAM 16GB 이상 (Jetson AGX Thor/Orin, DGX Spark 포함)
- 미세조정: VRAM 40GB 이상 GPU 권장

### 비교 포인트
| 항목 | Intel | NVIDIA |
|---|---|---|
| 구성 방식 | 수집·학습·내보내기를 하나의 GUI/CLI/API로 통합 | 단계별 도구를 조합 (유연하지만 설정이 많음) |
| 로봇 SW 기반 | 자체 통합 API + ROS 2 연동 | ROS 2 중심 (Isaac ROS) |
| 시뮬레이션·합성 데이터 | 공개 자료상 약함 | Isaac Lab, Cosmos 등 강점 |
| 모델 계층 | GR00T 등 외부 모델 지원 | GR00T 자체 제공 |
| 성숙도 | OpenVINO Physical AI는 프리뷰 (정식 2026 하반기 예정) | 튜토리얼·레퍼런스·커뮤니티가 풍부 |

**핵심 관찰**: 모델과 데이터 형식(LeRobot)은 양쪽이 열려 있어서, 차별화는 **하드웨어와 배포 런타임** 단계에서 일어납니다. 학습은 NVIDIA GPU, 배포는 Intel(OpenVINO)로 섞는 구성도 가능해 보이지만, 모델별 내보내기 지원은 직접 확인이 필요합니다.

---

## 7. 세미나에서 물어볼 질문

1. 같은 정책(SmolVLA, GR00T 등)을 **같은 전력 조건(예: 40W)** 에서 Jetson과 비교했을 때 지연시간은? 비교한 Jetson 모델은?
2. NPU에서 실제로 돌아가는 모델과 연산의 범위는? (iGPU, CPU로 넘어가는 부분)
3. 시뮬레이션·합성 데이터 도구는 어떻게 보완할 계획인가?
4. OpenVINO Physical AI의 정식 출시 일정, 라이선스, 지원 로봇·센서 목록
5. 한국 내 개발 키트·파트너 프로그램 구매 경로

---

## 8. 참고 자료
- Intel 뉴스룸: "130+ Customers Choose Intel Series 3 Processors for Edge Devices", "Intel Core Ultra Series 3: The New Standard for Edge AI Robotics Compute"
- Intel Performance Index (Core Ultra Series 3), Intel Core Ultra Series 3 for Edge Overview
- Intel Open Edge Platform: "Optimizing π0.5 VLA Model on Intel Core Ultra Series 3 Processor"
- TechSpot: Intel Core Series 3 targets affordable PCs
- 한국경제, 뉴시스, 아시아경제, ITWorld Korea: 국내 출시 및 갤럭시 북6 울트라 사용기
- Phoronix: Panther Lake 노트북 사전 주문 현황
- UltrabookReview: Panther Lake 노트북 목록
- Notebookcheck: Khadas Mind Pro, AceMagic F2A 기사, ComputerBase: Panther Lake 미니 PC 기사
- NVIDIA: Isaac GR00T GitHub, Isaac ROS 레퍼런스 워크플로, 개발자 블로그(GR00T 1.7 end-to-end)
