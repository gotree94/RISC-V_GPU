# RISC-V 기반 GPU 프로젝트 현황

> 최종 업데이트: 2026년 5월

---

## 목차

1. [개요](#개요)
2. [Bolt Graphics Zeus GPU (상업용)](#1-bolt-graphics-zeus-gpu-상업용)
3. [Borg — European GPU](#2-borg--european-gpu)
4. [e-GPU (EPFL)](#3-e-gpu-epfl)
5. [Vortex (Georgia Tech)](#4-vortex-georgia-tech)
6. [VOLT — Vortex 컴파일러](#5-volt--vortex-컴파일러)
7. [Ventus (칭화대학교)](#6-ventus-칭화대학교)
8. [기타 프로젝트](#7-기타-프로젝트)
9. [비교 요약](#8-비교-요약)
10. [참고 자료](#9-참고-자료)

---

## 개요

RISC-V 생태계에서 GPU(GPGPU 포함)를 구현하려는 여러 오픈소스 및 상업 프로젝트가 활발히 진행 중이다.
RISC-V의 개방형 ISA 특성 덕분에 명령어 확장을 통한 GPU 가속, 벡터 확장(RVV) 활용,
혹은 완전한 커스텀 GPU 아키텍처 등 다양한 접근 방식이 존재한다.

---

## 1. Bolt Graphics Zeus GPU (상업용)

- **발표일**: 2025년 10월
- **GitHub**: https://github.com/BoltGraphics (조직 페이지)
- **라이선스**: 상업용 (일부 드라이버/도구는 오픈소스)
- **주요 기술 스택**:
  - 수많은 소형 **RISC-V 코어** + **RISC-V Vector Extension (RVV)**
  - 자체 개발 **Lightning ray-tracing 가속기**
  - Path tracing 전용 하드웨어 파이프라인
  - **Ubuntu Linux** 기반 온보드 스택
- **인터페이스**: DisplayPort 2.1a, HDMI 2.1b, Ethernet 출력
- **라인업**:
  - Zeus 1C: 32GB LPDDR5 + SO-DIMM 슬롯 (최대 160GB)
  - Zeus 2C: 64GB (최대 320GB) / 128GB (최대 384GB)
  - Zeus 4C: 2TB 이상 메모리 지원
- **현재 상태**: FPGA 시뮬레이션 완료, ASIC 양산 준비 중
- **특이사항**: 기존 GPU와 달리 **범용 쉐이더가 아닌 path tracing 전용** 아키텍처

---

## 2. Borg — European GPU

- **GitHub**: https://github.com/gonsolo/borg ⭐
- **라이선스**: 오픈소스
- **언어**: Scala (Chisel) 47.2%, Python 28.8%, C 16.4%
- **타겟**: iCE40 FPGA (pico-ice) + **Tiny Tapeout ASIC (IHP SG13G2 130nm)**
- **아키텍처**:
  - **TinyQV** RV32I RISC-V SoC (Chisel로 재작성)
  - **Borg FP16 shader processor** — 메모리맵드 페리페럴
  - 8 x FP16 레지스터 (r0-r7), MMU-accessible
  - 6-word 명령어 (zero-overhead looping)
- **렌더링 파이프라인** (펌웨어 구현):
  1. Vertex Shader — SPIR-B programmable
  2. Screen-Space Translation (NDC → 픽셀 좌표)
  3. Rasterization — Edge-function, FP16 cross product 하드웨어 가속
  4. Fragment Shader — Barycentric interpolation, per-vertex RGB
  5. Framebuffer Output → PSRAM → RP2040 호스트
- **마일스톤**:
  - FPU 시뮬레이터 완료 / SoC 통합 완료
  - Vertex shader FPGA 검증 완료
  - Triangle rasterization + fragment shading 완료
  - SPIR-B 런타임 셰이더 로딩 완료
  - **Tiny Tapeout TTIHP26a 제출 완료** ✅
  - 칩 테스트: ⏳ 대기 중
  - Vulkan 드라이버: 📋 계획

---

## 3. e-GPU (EPFL)

- **발표일**: 2025년 5월 (arXiv: 2505.08421)
- **GitHub**: https://github.com/esl-epfl/e-gpu
- **라이선스**: 오픈소스
- **타겟**: **TinyAI** — 초저전력 엣지 디바이스
- **공정**: TSMC 16nm SVT CMOS, 300MHz, 0.8V
- **전력**: 28mW (16-스레드 구성)
- **성능**: 최대 **15.1x 속도 향상**, 에너지 **3.1x 절감**, 면적 2.5x 오버헤드
- **주요 특징**:
  - 설정 가능한 아키텍처 (코어/warp/thread 수 조절 가능)
  - 부동소수점 유닛 제거 → 정수/고정소수점 전용 (TinyAI 특화)
  - 커스텀 `SLEEP_REQ` 명령어로 전력 효율 향상
  - **Tiny-OpenCL** 경량 프로그래밍 프레임워크 제공
  - X-HEEP 호스트 플랫폼과 통합되어 APU 구성
- **소프트웨어 스택**:
  - SIMT RISC-V extension API (warp 관리, divergence handling, barrier)
  - Startup/scheduler 함수 (정적 라이브러리, RISC-V GNU toolchain)
  - Tiny-OpenCL runtime (Newlib-capable MCU 지원)
- **벤치마크**: GeMM (행렬 곱), TinyBio (바이오신호 처리)

---

## 4. Vortex (Georgia Tech)

- **GitHub**: https://github.com/vortexgpgpu/vortex ⭐ **~2,000 stars**
- **라이선스**: Apache 2.0
- **언어**: Verilog 54.9%, C++ 17.5%, SystemVerilog 6.4%
- **컨트리뷰터**: 50명
- **최신 릴리스**: **v2.3** (2026년 4월)
- **ISA**: RISC-V RV32IMAF / RV64IMAFD + **6개 커스텀 명령어** (wspawn, tmc, split, join, bar, tex)
- **아키텍처**:
  - 설정 가능한 코어/warp/thread 수
  - 설정 가능한 ALU/FPU/LSU/SFU 유닛 수
  - 설정 가능한 파이프라인 issue width
  - Local memory / L1 / L2 / L3 캐시 옵션
- **소프트웨어 스택**: **OpenCL 1.2**, OpenGL, CUDA (VOLT 통해)
- **지원 FPGA**: Altera Arria 10 / Stratix 10, Xilinx Alveo U50/U250/U280, Versal VCK5000
- **백엔드**: C++ 시뮬레이터 (simx), RTL 시뮬레이터, FPGA
- **성능**: 최대 32코어, 25.6 GFlops @ 200MHz (Stratix 10)
- **특징**: RISC-V GPU 중 가장 성숙한 풀스택, 실제 FPGA 구동 검증, 활발한 개발

---

## 5. VOLT — Vortex 컴파일러

- **GitHub**: https://github.com/vortexgpgpu/Volt
- **라이선스**: 오픈소스
- **최신 릴리스**: **v1.0** (2025년 12월)
- **학회**: CC 2026 (2026년 1월)
- **설명**: Vortex GPU 및 변종을 위한 **엔드투엔드 오픈소스 컴파일러 프레임워크**
- **프론트엔드**: **CUDA** (CuPBoP) + **OpenCL** (PoCL)
- **IR 최적화**: LLVM 기반 SIMT-aware 최적화 (GPU 변종 간 이식성)
- **백엔드**: LLVM-RISC-V 확장으로 Vortex ISA 완전 지원
- **구성 요소**: PoCL, LLVM-Vortex, RISC-V GNU toolchain, Verilator, Yosys 등

---

## 6. Ventus (칭화대학교)

- **GitHub**: https://github.com/THU-DSP-LAB/ventus-gpgpu
- **라이선스**: 오픈소스
- **언어**: Chisel HDL
- **학회**: **IEEE TVLSI 2025** 논문 게재, ICCD 2024
- **설명**: RISC-V Vector Extension 기반 GPGPU 프로세서
- **특징**:
  - XiangShan 및 RocketChip 모듈 재사용
  - Chisel → Verilog 변환 (`make verilog`) → FPGA 배포 (`make fpga-verilog`)
  - sim-verilator 기반 RTL 시뮬레이션

---

## 7. 기타 프로젝트

| 프로젝트 | GitHub / 링크 | 설명 | 언어 | 상태 |
|---------|--------------|------|------|------|
| **Vorion** | arXiv:2511.16831 (2025.11) | RISC-V GPU + 3D Gaussian Splatting 가속 | - | 논문, 16nm 시뮬레이션 |
| **RV64X** | https://github.com/avl-bsuir/rv64x-base | RISC-V GPU ISA 확장, Vulkan 목표 | Dockerfile | 초기 |
| **VeriGPU** | https://github.com/hughperkins/VeriGPU | Verilog RISC-V GPU, ML 타겟, ASIC 목표 | Verilog/SystemVerilog | 개발 중 |
| **X-Silicon C-GPU** | https://x-silicon.com (ISA 오픈소스 예정) | RISC-V Vector CPU + GPU 통합, 세계 첫 Vulkan-on-RISC-V | - | 2024년 발표 |
| **T-HEAD TH1520** | Linux kernel mainline | Imagination GPU + Zink, 첫 RISC-V 메인라인 3D 가속 | C | Linux 6.18+ 병합 |

---

## 8. 비교 요약

| 프로젝트 | 유형 | GPU 목적 | HW 공개 | SW 공개 | 성숙도 | FPGA 검증 | ASIC |
|---------|------|---------|---------|---------|-------|----------|------|
| **Vortex** | 오픈소스 | GPGPU + 3D | ✅ Verilog | ✅ OpenCL/CUDA | ★★★★★ | ✅ | ❌ |
| **Ventus** | 오픈소스 | GPGPU | ✅ Chisel | ✅ | ★★★★ | ✅ | ❌ |
| **e-GPU** | 오픈소스 | TinyAI 가속 | ✅ Verilog | ✅ Tiny-OpenCL | ★★★★ | ❌ | ✅ 16nm |
| **Borg** | 오픈소스 | 3D 그래픽 (교육용) | ✅ Chisel | ✅ SPIR-B | ★★★ | ✅ iCE40 | ✅ IHP130nm |
| **Bolt Zeus** | 상업용 | Path tracing | ❌ | 일부 | ★★★ | ✅ FPGA | ⏳ |
| **RV64X** | 오픈소스 | 3D 그래픽 | 일부 | ❌ | ★ | ❌ | ❌ |
| **VeriGPU** | 오픈소스 | ML 가속 | ✅ Verilog | ✅ | ★★ | ❌ | 목표 |
| **X-Silicon** | 상업용 + 오픈ISA | 통합 CPU-GPU | ❌ | Vulkan 드라이버 | ★★★ | ❌ | 목표 |

---

## 9. 참고 자료

- RISC-V International: https://riscv.org
- Vortex 프로젝트: https://vortex.cc.gatech.edu
- Vortex 논문 (MICRO '21): https://doi.org/10.1145/3466752.3480128
- e-GPU 논문 (arXiv 2505.08421): https://arxiv.org/abs/2505.08421
- Vorion 논문 (arXiv 2511.16831): https://arxiv.org/abs/2511.16831
- Bolt Graphics Zeus 발표 (The Register): https://www.theregister.com/2025/10/29/bolt_graphics_zeus_gpu
- T-HEAD TH1520 3D 가속: https://mwilczynski.dev/posts/riscv-gpu-zink/
