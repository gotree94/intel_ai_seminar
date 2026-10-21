# ② Physical AI 선점 전략과 Physical AI Studio 실습

> 공개된 Intel 보도자료, GitHub/PyPI 문서, Intel Robotics AI Suite 해커톤 가이드를 바탕으로 정리했습니다. 패키지 버전과 옵션은 빠르게 바뀌므로 설치 전 각 저장소의 최신 README를 확인하세요. 핸즈온 세션의 실제 구성(보드, 로봇, 순서)은 일정표만으로는 알 수 없어 **현장 확인 항목**으로 남겼습니다.

---

## 1. 상황: Intel이 Physical AI에서 노리는 것

### 1.1 배경
- 모델은 이미 충분히 많아졌고, 이제 병목은 **모델을 실제 로봇에 얹어 동작시키는 것**입니다.
- LLM은 웹의 방대한 데이터로 학습하지만, 로봇은 새 작업(물건 집기, 수건 접기 등)마다 **직접 시연 데이터를 수집**하고 **미세조정**해야 합니다. Intel 측 설명에 따르면 같은 동작을 50~100번 시연해야 하는 경우도 있습니다.
- 그래서 Intel은 **데이터 수집 → 학습·미세조정 → 최적화 → 배포**의 전 과정을 묶은 도구를 제공해 로봇 개발자를 자사 플랫폼으로 끌어들이려 합니다.

### 1.2 Intel Robotics 포트폴리오 (공개 자료 기준)

| 구성 | 역할 |
|---|---|
| **Intel Core Ultra Series 3 / Core Series 3** | 로봇·엣지 AI용 프로세서 (CPU + GPU + NPU 통합). 코드명 Panther Lake 계열 |
| **Intel Robotics AI Suite** | SLAM, 로봇 비전·제어 라이브러리, ROS 2 연동, 참조 애플리케이션, VLM/VLA 모델을 묶은 오픈소스 스택 |
| **Physical AI Studio** | 시연 데이터 수집, 모델 미세조정, 최적화·양자화, 사전 검증된 VLA 모델 내보내기 (GUI/CLI/Python API) |
| **OpenVINO Physical AI** | 카메라·센서·AI 모델·로봇 동작을 하나의 API로 통합하는 배포 프레임워크. 공개 자료상 프리뷰가 GitHub에 공개되었고 정식 출시는 2026년 하반기 예정 |
| **LeRobot 연동** | Hugging Face LeRobot 데이터셋·정책과 연동해 PyTorch 또는 OpenVINO로 내보내기 |

### 1.3 "GPU 없이" 메시지 (XPU 이기종 구조)
Intel 측 설명에 따르면 로봇은 공간, 전력, 발열 제약이 커서 **별도 대형 GPU를 달면 비용·전력·크기가 늘어납니다.** 그래서 한 SoC 안에서 역할을 나눕니다.

| 연산 유닛 | 담당 (설명 예시) |
|---|---|
| iGPU | 영상 인식 등 병렬 연산 |
| NPU | 저전력 AI 추론 |
| CPU | 에이전틱 AI 연산, 작업 스케줄링, 외부 시스템 통신 |

### 1.4 일정표 세션과의 매핑 (추론)

| 세션 | 추정 내용 |
|---|---|
| Intel: Open Edge approach | 오픈 엣지 플랫폼 생태계 접근 |
| Intel: Future of Robotics | Physical AI 기회와 Intel 로보틱스 로드맵 |
| Intel: Edge Architectures for Modern Mission Systems | 미션·방산급 엣지 시스템 아키텍처 |
| Intel: Purpose Built Silicon | 용도별 특화 실리콘 활용 |
| Intel for Startups | 스타트업 생태계 프로그램 |
| Tommoro Robotics | Intel PTL + Physical AI Studio로 Habilis Brain Zero 평가 |
| Daim Technology | Intel x DAIM 무인 공장 로드맵 |
| Circlus | 서큘러스 onAIR 기반 산업 Physical AI |
| RovifyLab | AgentDrive: 멀티모달 Physical AI와 자율 로봇 제어 통합 |
| **핸즈온 (그랜드볼룸 101)** | 15:20~16:40, 4개 파트너 세션과 시간대가 겹침 |

> 핸즈온 시간대가 트랙 2 파트너 세션(15:20~16:40)과 같으므로 **둘 중 하나를 선택**해야 할 가능성이 큽니다.

---

## 2. 방향: Physical AI Studio가 해결하는 워크플로

```
[시연 수집] → [데이터셋] → [정책 학습/미세조정] → [최적화·내보내기] → [로봇 배포·추론]
 텔레오퍼레이션   LeRobot 형식      ACT, Pi0, SmolVLA,        OpenVINO / ONNX /      physicalai 런타임
 (리더-팔로워)                    GR00T, Pi0.5 등           Torch                  (카메라·로봇 API)
```

### 2.1 핵심 개념
| 용어 | 설명 |
|---|---|
| **모방학습 (Imitation Learning)** | 사람의 시연을 보고 로봇이 같은 작업을 하도록 학습 |
| **VLA (Vision-Language-Action)** | 카메라 영상과 언어 지시를 입력받아 로봇 동작을 출력하는 모델 |
| **정책 (Policy)** | 관측(영상, 관절 상태)을 입력받아 행동을 내는 모델. ACT, SmolVLA, Pi0 등 |
| **텔레오퍼레이션** | 사람이 리더 팔을 움직이면 팔로워 팔이 따라 하며 데이터를 기록하는 방식 |

### 2.2 저장소 구성 (공개 저장소 기준)
| 패키지 | 용도 | 설치 |
|---|---|---|
| Physical AI Studio (앱) | GUI 기반 수집·학습·평가 (백엔드 + 웹 UI) | 소스 클론 후 `uv sync` |
| `physicalai-train` | 학습·평가·내보내기 Python SDK | `pip install physicalai-train` |
| `physicalai` | 배포용 런타임 (카메라·로봇 API, 추론 엔진) | `pip install physicalai` |
| `physicalai-so101-plugin` | SO-101 로봇 암 카탈로그·드라이버·URDF | `uv add physicalai-so101-plugin` |

---

## 3. 실습: 환경 설정 → 설치 → 활용

### 3.0 준비물

**최소 (소프트웨어 체험)**
- Ubuntu 24.04 PC (CPU만으로 가능하나 학습은 느림)
- 시뮬레이션(MuJoCo SO-101 플러그인)으로 로봇 없이 파이프라인 체험 가능

**권장 (하드웨어 포함)**
- Intel Core Ultra 계열 PC (iGPU + NPU). 공개 가이드의 대상 환경은 **Ubuntu 24.04 LTS(HWE 커널) + Intel Core Ultra/Arc**
- SO-101 로봇 암 세트 (리더 1 + 팔로워 1), USB 카메라 1~2대 또는 Intel RealSense
- 디스크 100GB 이상, 인터넷 연결(데이터셋·모델 다운로드)

**학습 GPU 대안**
- 학습은 Intel XPU, NVIDIA CUDA, CPU 중 선택 (`--extra xpu | cuda | cpu`)
- 소형 정책(ACT 등)은 가능하지만 큰 VLA 미세조정은 GPU 메모리가 중요

### 3.1 시스템 드라이버 설치 (Intel NPU / iGPU)
Intel Robotics AI Suite 해커톤 가이드는 스크립트 기반 설치를 안내합니다. 구성 순서는 다음과 같습니다.

1. **NPU 드라이버**: `github.com/intel/linux-npu-driver` 릴리스에서 최신 `.deb` 패키지를 받아 설치, Level Zero 로더(`libze1`) 확인, 사용자를 `render` 그룹에 추가
2. **iGPU 드라이버**: Intel graphics PPA(`ppa:kobuk-team/intel-graphics`)로 컴퓨트 런타임(OpenCL, Level Zero)과 미디어(VA-API) 설치, `render`·`video` 그룹 추가
3. 재부팅 후 확인

```bash
# 그룹 반영을 위해 재로그인 또는 재부팅
clinfo | grep "Device Name"      # Intel GPU가 보이면 정상
ls /dev/accel*                   # NPU 장치 노드 (환경에 따라 다름)
groups                           # render, video 포함 여부
```
> 드라이버 버전 변수(예: `NPU_DRIVER_VERSION`)는 가이드의 스크립트 상단에서 최신 릴리스로 갱신하라고 안내합니다. 가이드: Intel Robotics AI Suite 문서의 "Hack-a-thon Resources" 페이지.

### 3.2 Physical AI Studio 설치 (앱, 권장 경로)
필요 도구: `git`, `uv`(Python 패키지 관리), `nvm`과 Node v24.

```bash
# uv 설치
curl -LsSf https://astral.sh/uv/install.sh | sh

# nvm + Node 24 (nvm 설치 후)
nvm install 24 && nvm use 24

# 소스 받기
git clone https://github.com/open-edge-platform/physical-ai-studio.git
cd physical-ai-studio/application/backend

# 백엔드 의존성: 하드웨어에 맞는 extra 하나 선택
uv sync --extra xpu     # Intel GPU(XPU)
# uv sync --extra cuda  # NVIDIA GPU
# uv sync --extra cpu   # CPU only
```

실행은 터미널 2개로 합니다. (해커톤 가이드 기준 경로)
```bash
# 터미널 1: 백엔드
cd ~/physical-ai-studio/application/backend && ./run.sh

# 터미널 2: UI
cd ~/physical-ai-studio/application/ui && nvm use && npm install && npm run start
```
브라우저에서 `http://localhost:3000` 접속.

> **Docker 경로**: 저장소의 Docker README에 Intel XPU / NVIDIA CUDA 장치 설정이 안내되어 있으니 컨테이너 방식을 선호하면 그쪽을 따르세요.

### 3.3 Python SDK만 쓰는 경우 (GUI 없이)
```bash
sudo apt-get install -y ffmpeg      # LeRobot 의존성
uv venv && source .venv/bin/activate
pip install physicalai-train
pip install openvino                # OpenVINO 내보내기 시
```
소스에서 설치하려면 `library/` 디렉터리에서 `uv sync --extra xpu --extra all`처럼 하드웨어 extra를 지정합니다.

학습 예시는 공개 문서의 데이터 모듈 구성처럼 LeRobot 데이터셋을 지정해 시작합니다.
```python
# 문서 예시: 시뮬레이션 ALOHA 데이터셋 (정확한 API는 최신 문서 확인)
# datamodule = LeRobotDataModule(repo_id="lerobot/aloha_sim_transfer_cube_human")
```
지원 정책은 ACT, Pi0, Pi0.5, SmolVLA, GR00T 및 LeRobot 정책군이며, 내보내기 백엔드는 **OpenVINO(Intel CPU/GPU/NPU), ONNX, Torch**입니다.

### 3.4 로봇 연결 (SO-101 기준)
```bash
pip install physicalai                      # 런타임
pip install "physicalai[so101]"             # SO-101 암
pip install "physicalai[realsense]"         # RealSense 카메라 (선택)
pip install "physicalai[plugin-mujoco]"     # 시뮬레이션 SO-101 (로봇 없이 체험)
pip install "physicalai[plugin-bimanual-so101]"  # 양팔 구성 (선택)
```

Linux에서 시리얼 포트 권한을 확인합니다.
```bash
ls /dev/ttyUSB* /dev/ttyACM*
sudo usermod -aG dialout $USER     # 재로그인 필요
```

SO-101 플러그인 사용 예 (공개 문서의 형태, 포트·캘리브레이션 파일은 본인 환경에 맞게):
```python
from physicalai_so101_plugin import SO101
arm = SO101(port="/dev/ttyUSB0", calibration="calibration.json")
```
> 처음 연결할 때는 **캘리브레이션**이 필요합니다. 리더/팔로워 팔 모두 같은 기준 자세로 보정해야 시연 데이터가 올바르게 기록됩니다.

### 3.5 활용 워크플로 (GUI 기준)

| 단계 | 작업 | 팁 |
|---|---|---|
| ① 프로젝트 생성 | 새 프로젝트에 로봇, 카메라, 센서 등록 | 카메라 해상도·FPS를 학습 때와 배포 때 동일하게 |
| ② 시연 수집 | 리더 팔로 같은 작업을 반복 시연, 에피소드 기록 | 작업당 50회 안팎에서 시작, 시작 자세·물체 위치를 약간씩 변화 |
| ③ 데이터 검토 | 실패 에피소드 제거, 영상·관절 데이터 동기 확인 | 품질이 양보다 중요 |
| ④ 학습 | 정책 선택(소형은 ACT, VLA는 SmolVLA 등) 후 학습·미세조정 | GPU 메모리에 맞춰 배치 크기 조절 |
| ⑤ 평가 | 시뮬레이션 또는 실제 로봇에서 성공률 측정 | 시도 횟수를 고정해 비교 |
| ⑥ 내보내기 | OpenVINO IR로 내보내고 양자화 | Intel NPU/iGPU 타깃 지정 |
| ⑦ 배포 | `physicalai` 런타임으로 로봇에서 추론 실행 | 안전 정지(E-stop)와 속도 제한부터 설정 |

### 3.6 OpenVINO로 장치 확인 및 배포 점검
```python
import openvino as ov
core = ov.Core()
print(core.available_devices)    # ['CPU', 'GPU', 'NPU'] 등이 보이면 정상
```
정책 모델을 내보낸 뒤 장치를 바꿔 가며(`CPU`, `GPU`, `NPU`) 지연시간과 전력을 비교해 보면 "이기종 분담" 메시지를 직접 검증할 수 있습니다.

### 3.7 검증 스크립트
해커톤 가이드는 설치 후 `conda activate intel_dev_env` 환경에서 `verify_stack.py`로 스택(드라이버, OpenVINO, Anomalib, LeRobot 등)을 점검하는 흐름을 제공합니다. 본인이 uv 기반으로 구성했다면 아래 항목을 수동으로 확인하세요.

- [ ] `clinfo`에서 Intel GPU 인식
- [ ] OpenVINO `available_devices`에 NPU 표시
- [ ] 백엔드 실행 후 `http://localhost:3000` 접속
- [ ] 시뮬레이션(MuJoCo) 로봇이 UI에서 보임
- [ ] 샘플 데이터셋으로 소규모 학습이 1 epoch 이상 진행

---

## 4. 소규모 로봇 프로젝트에 적용하는 아이디어

- **텔레오퍼레이션 데이터 수집 → 모방학습**: 리더-팔로워 구조로 시연을 모으는 방식은 소형 교육용 로봇 암에도 응용할 수 있습니다. 로봇 암이 SO-101과 다르면 플러그인 방식(카탈로그 정의 + 드라이버)으로 직접 로봇을 추가하는 경로가 문서에 안내되어 있습니다.
- **카메라 기반 동작 인식으로 시연 수집**: 웨어러블 카메라나 마커 기반 팔 동작 인식으로 만든 시연을 **LeRobot 데이터셋 형식**으로 변환하면 같은 학습 파이프라인을 쓸 수 있는지 검토해 볼 가치가 있습니다. (가능 여부는 데이터 스키마 확인 필요)
- **교육 과정화**: "시연 수집 → 학습 → OpenVINO 내보내기 → NPU 배포"는 하루~이틀 실습 단위로 나누기 좋은 흐름입니다.

---

## 5. 주의할 점 / 한계

- **프리뷰 단계 요소**: OpenVINO Physical AI는 공개 자료상 프리뷰이며 정식 출시는 2026년 하반기 예정이라 API가 바뀔 수 있습니다. 버전을 고정하세요.
- **성능 기대치**: 큰 VLA 모델의 학습·미세조정은 GPU 메모리가 중요합니다. NPU는 주로 **추론 전용**으로 보는 것이 안전합니다.
- **안전**: 실제 로봇은 사람·환경과 직접 상호작용합니다. 속도·토크 제한, 작업 영역 확보, 비상정지를 먼저 갖추세요. Intel 측도 개발 후 실환경 테스트에 상당한 기간이 필요하다고 설명합니다.
- **드라이버 호환**: 커널, NPU 드라이버, OpenVINO 버전 조합에 민감합니다. 가이드가 지정한 Ubuntu/커널 조합을 따르는 것이 안전합니다.

---

## 6. 핸즈온 현장 확인 질문

1. 제공되는 하드웨어: 프로세서 세대(Core Ultra Series 3?), 메모리, NPU 성능, 로봇 암 종류
2. 실습 범위: 수집부터 배포까지인가, 사전 학습된 모델 추론만인가
3. 지원 정책·모델 목록과 NPU에서 실제로 돌아가는 범위
4. 자료 배포: 실습 저장소, 이미지, 스크립트를 가져갈 수 있는가
5. OpenVINO Physical AI의 정식 출시 일정과 라이선스
6. 한국 내 교육·파트너 프로그램 및 개발 키트(파트너사 제공) 구매 경로

---

## 7. 참고 자료
- Physical AI Studio: https://github.com/open-edge-platform/physical-ai-studio
- Intel Robotics AI Suite 문서(해커톤 설치 가이드 포함): https://docs.openedgeplatform.intel.com/dev/ai-suite-robotics.html
- OpenVINO Physical AI 소개: https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/physical-ai.html
- Intel 뉴스룸, "130+ Customers Choose Intel Series 3 Processors for Edge Devices"
- PyPI: `physicalai`, `physicalai-train`, `physicalai-so101-plugin`
- Intel NPU 드라이버: https://github.com/intel/linux-npu-driver
- LeRobot: https://github.com/huggingface/lerobot
- 자유재경(自由財經) 2026/09/16, 인텔 Physical AI 관련 기사
