# PROMPTS.md — DK-DETR Demo Inference 구현 프롬프트 로그

**과제:** ICCV 2023 논문 "Distilling DETR with Visual-Linguistic Knowledge for Open-Vocabulary Object Detection (DK-DETR)" 코드 구현  
**레포지토리:** https://github.com/hikvision-research/opera  
**사용 AI 도구:** Claude (claude.ai), Cursor IDE (claude-sonnet-4.6)

---

## Prompt 1: 초기 레포 구조 파악 요청 (Cursor Agent)

```
이 레포는 OPERA (https://github.com/hikvision-research/opera)이고,
ICCV 2023 논문 DK-DETR (Distilling DETR with Visual-Linguistic Knowledge for
Open-Vocabulary Object Detection)의 demo inference 실행이 목표야.

다음 순서로 파악하고 실행 계획을 세워줘:

1. configs/dk-detr/ 안의 config 파일 목록과 모델 variant 확인
2. DK-DETR 모델 구현 코드 위치 파악 (models/ 등)
3. demo/inference 실행 스크립트 존재 여부 확인
4. requirements.txt / setup.py 기반 환경 의존성 정리

환경: WSL2 Ubuntu, RTX 5070 Ti, CUDA 12.x
mmdetection 기반이면 mmcv 버전 호환성도 같이 체크해줘.

파악 후 pretrained weight 다운로드 → 환경 설치 → demo 실행까지
단계별 명령어로 정리해줘.
```

---

## Prompt 2: 환경 구성 방향 결정 (Claude)

```
분석 결과를 보면 mmcv 1.5.3이 필요한데,
RTX 5070 Ti는 Blackwell 아키텍처 (sm_120, CUDA 12.8)라
CUDA 호환 이슈가 예상돼.

가장 빠르게 demo 결과를 확인할 수 있는 방법은?
CPU inference로 동작 확인하는 것도 괜찮은지 알려줘.
```

---

## Prompt 3: mmcv 설치 트러블슈팅 (Claude)

```
pip install mmcv-full==1.5.3 시도 시 아래 에러 발생:
ModuleNotFoundError: No module named 'pkg_resources'

setuptools 버전 문제인 것 같은데, 어떻게 해결하면 돼?
현재 환경: conda opera-dkd, python 3.9, setuptools 82.x
```

---

## Prompt 4: CUDA 버전 불일치 해결 (Claude)

```
mmcv-full 소스 빌드 시 아래 에러 발생:
RuntimeError: The detected CUDA version (12.8) mismatches the version
that was used to compile PyTorch (11.7).

RTX 5070 Ti 환경에서 CPU demo를 실행하려면 어떻게 해야 해?
CUDA 충돌 없이 mmcv를 설치하는 방법 알려줘.
```

---

## Prompt 5: SyncBatchNorm CPU 에러 해결 (Claude)

```
demo 실행 시 아래 에러 발생:
ValueError: SyncBatchNorm expected input tensor to be on GPU

config에서 SyncBN을 BN으로 바꿔야 할 것 같은데,
가장 간단한 방법이 뭐야?
```
---

## 주요 트러블슈팅 요약

| 이슈 | 원인 | 해결 |
|---|---|---|
| pkg_resources not found | setuptools 82.x 격리 빌드 환경 이슈 | setuptools==59.5.0 + PIP_CONSTRAINT |
| CUDA version mismatch | RTX 5070 Ti (CUDA 12.8) vs torch 컴파일 CUDA 불일치 | CPU 전용 PyTorch 전환 |
| mmcv._ext not found | pure Python wheel 설치로 C++ ops 누락 | MMCV_WITH_OPS=1 소스 빌드 |
| CLIPProcessor import 실패 | transformers 4.57이 torch 1.13 미지원 | transformers==4.28.1 다운그레이드 |
| SyncBatchNorm CPU 에러 | SyncBN은 GPU 전용 모듈 | config 파일 sed로 BN 일괄 치환 |
| NumPy 2.x ABI 불일치 | mmcv가 NumPy 1.x 기준 컴파일 | numpy<2 다운그레이드 |
