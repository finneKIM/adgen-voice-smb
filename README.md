# AdGen — 음성 기반 소상공인 마케팅 콘텐츠 자동 생성 서비스

소상공인이 음성 한마디로 홍보 문구·포스터 이미지·배경음악·릴스 영상을 자동 생성할 수 있는 AI 마케팅 어시스턴트입니다.

**서비스 접속**: https://adgen4.duckdns.org/ *(프로젝트 기간 종료로 현재는 운영되지 않습니다)*

**👇 클릭하면 YouTube에서 영상을 확인할 수 있습니다.**

[![AdGen 시연 영상](https://github.com/user-attachments/assets/8bf1ed27-adce-4b11-b503-0ca0fe793f1b)](https://youtu.be/dajeIKtOP-A)

---

## 핵심 기능

**1. 음성 기반 요구 분석**

사용자는 복잡한 입력 없이 말 한마디면 충분합니다. "영상으로 만들어줘", "포스터 하나 만들어줘" 같은 모호한 말도 자연스럽게 이해해 적절한 형태의 광고 콘텐츠를 제안합니다.

**2. 콘텐츠 생성 모드 제공 (3가지)**

- **이미지 생성 모드**

  원하는 분위기·스타일을 자동으로 해석해 광고용 이미지를 생성. 제품 사진을 업로드하면 별도 편집 없이 제품이 돋보이는 광고 이미지가 자동 구성됨

- **이미지 + 음악 생성 모드**

  완성된 광고 이미지에 분위기에 어울리는 짧은 배경 음악을 자동 생성. SNS 게시물·매장 홍보용 음원으로 활용 가능

- **이미지 + 음악 → 영상 생성 모드**

  생성된 이미지와 음악을 자동으로 조합해 짧은 홍보 영상 제작. 릴스/쇼츠 형태로 바로 다운로드 가능

**3. Packshot 광고 생성**

사용자가 올린 실제 제품 사진을 기반으로 제품이 돋보이도록 구성된 Packshot 스타일 광고 이미지를 자동 생성합니다.

**4. 프롬프트 없이도 사용 가능**

전문적인 프롬프트 지식 없이, 음성이나 자연스러운 대화로 원하는 분위기·콘셉트를 설명하면 됩니다. 멀티턴 챗봇이 업종·상황·선호 스타일을 대화 흐름 속에서 파악해 광고 아이디어·문구·이미지 콘셉트를 자동 제안하며, Rigid / Balanced / Creative 모드로 이미지의 창의성·변형 정도를 조절할 수 있습니다.

**5. SNS 공유 기능**

생성된 이미지·음원·영상은 즉시 다운로드 가능하며, 모바일에서는 공유 패널로 인스타그램·카카오톡 등에 바로 공유할 수 있습니다.

---

## 서비스 흐름

1. 사용자가 음성 입력
2. Whisper가 텍스트 변환
3. 사용자 선택 모드에 따라 이미지 / 이미지+음악 / 이미지+음악→영상 중 하나 생성
4. 사용자가 실제 제품 이미지 입력
5. GPT가 이미지·음악 생성에 필요한 프롬프트 및 문구 생성
6. 생성형 AI 파이프라인에서 제품과 배경이 합성된 광고 이미지 생성
7. 사용자는 GPT가 생성한 홍보 문구의 색상, 위치 등을 편집
8. 제품 + 배경 + 문구가 있는 최종 포스터 생성
9. 사용자 선택 모드에 따라 추가로 음원 및 영상 생성
10. MinIO에 저장 후 UI에 표시
11. 다운로드하여 SNS 등에서 활용

---

## 기술 스택

**AI 모델**

- Whisper API — 음성 인식
- GPT-4o-mini — 마케팅 문구 및 이미지·음악 프롬프트 생성
- Stable Diffusion 1.5 — 이미지 생성
- ControlNet / IP-Adapter — 제품 기반 포스터 강화
- SAM / MobileSAM — 누끼
- MusicGen API (Replicate) — 광고용 음악 생성

**Backend**

- FastAPI
- MinIO (이미지·음원·영상 저장)
- MoviePy (영상 합성)

**Frontend**

- React
- Zustand
- PWA

**Infra**

- GCP VM (L4 GPU)
- Nginx Reverse Proxy
- DuckDNS + HTTPS (Certbot)

---

## 담당 업무

### 1. STT (Whisper)

**기여**: self-host 실험 및 최종 API 연동 설계

- Hugging Face `transformers` 라이브러리로 `whisper-large-v3`를 자체 서버에 올려 파인튜닝을 시도함 (사투리·특화 용어 인식률 개선 목적)
- GPU 메모리 제약으로 self-host 방식을 포기하고, OpenAI Whisper API로 전환

**Trouble & Decision**

| 문제 | 판단 | 결과 |
| --- | --- | --- |
| `whisper-large-v3` self-host 시 GPU 메모리 부족(OOM) 위험 | API로 테스트해본 결과 파인튜닝 없이도 사투리 인식 성능이 충분히 확보됨을 확인 | self-host 코드 폐기, OpenAI Whisper API로 최종 확정 |

---

### 2. Diffusion (이미지 생성)

**기여**: SDXL→SD1.5 전환, ControlNet(Depth)·IP-Adapter 오케스트레이션, CompositionMode 프리셋(Rigid/Balanced/Creative) 설계

- 팀 베이스라인(자체호스팅 SD 파이프라인) 위에 ControlNet + IP-Adapter를 결합해 제품 이미지 기반 배경 합성 기능 구현
- 사용자 요청 성격에 따라 Control Weight / IP-Adapter Scale 조합을 프리셋화(rigid/balanced/creative)해 결과물 일관성 확보

**Trouble & Decision**

| 문제 | 판단 | 결과 |
| --- | --- | --- |
| SD1.5+ControlNet+IP-Adapter 조합 시 VRAM 초과(OOM), L4 24GB 환경에서 peak 16~22GB | 5가지 양자화 방식(PyTorch Dynamic/Static, bitsandbytes, ONNX Runtime, TensorRT INT8) 검토 — ControlNet이 UNet 내부 레이어에 직접 결합되는 구조라 전부 구조적으로 불가능함을 확인 | 양자화 대신 fp16 + VAE slicing + semaphore 기반 요청 직렬화로 메모리 최적화, peak VRAM 22GB→15GB로 안정화 |

---

### 3. Audio/BGM 생성

**기여**: MusicGen(Replicate API) 연동 설계 및 구현

- GPT가 생성한 BGM 프롬프트를 기반으로 Replicate의 `meta/musicgen` 모델을 호출해 배경음악(wav) 생성
- 파일 저장 방식(URL 반환)과 스트리밍 방식(즉시 재생) 두 가지 엔드포인트 제공

**Trouble & Decision**

| 문제 | 판단 | 결과 |
| --- | --- | --- |
| Stable Audio Open API를 1차 검토했으나 유료 과금 정책 확인(크레딧 기반) | 사용량 기반 과금인 Replicate MusicGen API가 비용 효율적이라 판단 | MusicGen을 Replicate API로 연동해 최종 확정 |

---

### 4. Media 합성 (이미지 + 오디오 → 영상)

**기여**: 이미지·BGM 합성 mp4 생성 파이프라인 구현

- Instagram이 로컬 오디오 업로드를 지원하지 않는 제약을 확인 → 이미지+BGM을 moviepy/ffmpeg 기반으로 mp4로 합성해 릴스/스토리에 바로 업로드 가능한 형태로 제공
- 이미지 단독 / 이미지+오디오 / 이미지+오디오+영상 세 가지 생성 모드를 옵션 기반으로 지원

**Trouble & Decision**

| 문제 | 판단 | 결과 |
| --- | --- | --- |
| MoviePy 2.x API 변경(`set_duration`→`duration`, `with_audio`)으로 기존 코드 호환 안 됨 | 라이브러리 버전 확인 후 2.x 스타일 API로 코드 수정 | 정상 동작 확인 |
| 연속 요청 시 VRAM 누적으로 OOM 발생 | `torch.cuda.empty_cache()`, `torch.cuda.ipc_collect()`를 파이프라인 최종 단계에서 호출 | OOM 재발 방지 |

---

<details>
<summary><h2>배포 구조 / Repository Structure (펼치기)</h2></summary>

### 배포 구조

AdGen은 단일 VM에서 동작하며, Nginx가 다음과 같이 라우팅합니다:

| 리소스 | 접근 방식 | Nginx 내부 라우팅 |
| --- | --- | --- |
| 백엔드(API) | https://adgen4.duckdns.org/api/... | http://localhost:8080/api/... |
| MinIO | https://adgen4.duckdns.org/media/... | http://localhost:9000/... |
| 프론트엔드 | https://adgen4.duckdns.org/ | /var/www/adgen/dist |

### Repository Structure

```
adgen-service
├── backend/                        # FastAPI Backend
│   ├── app/
│   │   ├── api/                   # API 라우터
│   │   ├── core/                  # 설정 및 공통 모듈
│   │   ├── services/              # 주요 비즈니스 로직
│   │   ├── weights/               
│   │   ├── main.py                # FastAPI 엔트리포인트
│   │   └── __init__.py
│   ├── alembic/                   # DB 마이그레이션
│   │   └── versions/              # 변경 이력
│   ├── alembic.ini                      
│   ├── data/                      # 서비스 내부 데이터
│   ├── db/                        # DB 초기화 스크립트
│   └── init_db.py                 # 초기 DB 세팅 스크립트
│
├── frontend/                       # React + TypeScript Frontend (Vite)
│   ├── public/                    # HTML 템플릿 및 정적 파일
│   ├── src/
│   │   ├── api/                   # API 연동 모듈
│   │   ├── components/            # UI 컴포넌트
│   │   ├── context/               # 전역 상태 관리(Context API)
│   │   ├── hooks/                 # 커스텀 훅
│   │   ├── pages/                 # 라우팅 페이지
│   │   ├── router/                # 라우터 설정
│   │   ├── styles/                # Tailwind 및 전역 스타일
│   │   ├── types/                 # TypeScript 타입 선언
│   │   └── utils/                 # 유틸리티 함수
│   ├── index.html
│   ├── package.json
│   └── vite.config.ts
│
├── nginx/                          # Nginx Reverse Proxy 설정
│   └── nginx.conf
││
├── docker-compose.yml              # 컨테이너 구성(ES/MinIO 등)
├── ecosystem.config.js             # PM2 프로세스관리 설정
├── pyproject.toml                  # Python 프로젝트 설정
└── README.md
```

</details>

---

## Links

- 팀 최종 병합 레포: https://github.com/godori0322/adgen-service
- 최종 보고서(PDF): [4팀_최종보고서.pdf](https://github.com/godori0322/adgen-service/blob/main/4%ED%8C%80_%EC%B5%9C%EC%A2%85%EB%B3%B4%EA%B3%A0%EC%84%9C.pdf)