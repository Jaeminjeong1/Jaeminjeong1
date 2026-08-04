# 두꺼비집 캐릭터 자체 리디자인 3종 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 같은 청록 지도 핀·집 프레임 안에 신체 구조가 서로 다른 두꺼비 캐릭터 3종을 제작하고 투명 PNG로 제공한다.

**Architecture:** 기존 청록 지붕형 마젠타 배경 원본을 세 방향의 공통 참조로 사용한다. 각 방향을 독립 이미지 편집 호출로 생성하고, 하드 크로마키 제거 후 알파·색상·구조를 검수한다.

**Tech Stack:** OpenAI ImageGen, PNG RGBA, `remove_chroma_key.py`, Pillow 기반 픽셀 검수

## Global Constraints

- 지도 핀 외곽, 내부 크림색 면, 청록 지붕 띠, 오른쪽 굴뚝, 연한 민트-아이보리 집 몸체는 유지한다.
- 노란색·주황색·글자를 사용하지 않는다.
- 지도 핀 바깥만 투명하고 핀 내부는 완전 불투명해야 한다.
- 세 캐릭터는 머리 실루엣·눈 구조·체형·피부 특징에서 명확히 달라야 한다.

---

### Task 1: 든든한 집지킴이 생성

**Files:**
- Reference: `work/imagegen/dukkeobijip-pin-teal-roof-confident-magenta.png`
- Create: `work/imagegen/dukkeobijip-character-guardian-magenta.png`
- Create: `outputs/dukkeobijip-character-guardian-transparent.png`

**Interfaces:**
- Consumes: 기존 청록 지도 핀·집 프레임 이미지
- Produces: 넓고 낮은 머리, 작은 눈, 두꺼운 눈두덩, 묵직한 체형의 RGBA PNG

- [ ] **Step 1: 이미지 편집 생성**

  공통 프레임과 마젠타 바깥 배경을 고정하고 두꺼비만 집지킴이 구조로 교체한다.

- [ ] **Step 2: 하드 크로마키 제거**

  Run: `python3 /Users/jeongjaemin/.codex/skills/.system/imagegen/scripts/remove_chroma_key.py --input work/imagegen/dukkeobijip-character-guardian-magenta.png --out outputs/dukkeobijip-character-guardian-transparent.png --auto-key border --tolerance 48`

- [ ] **Step 3: 시각 검수**

  지도 핀과 집 구조가 보존되고 캐릭터가 넓고 묵직한 실루엣인지 확인한다.

### Task 2: 영리한 길잡이 생성

**Files:**
- Reference: `work/imagegen/dukkeobijip-pin-teal-roof-confident-magenta.png`
- Create: `work/imagegen/dukkeobijip-character-guide-magenta.png`
- Create: `outputs/dukkeobijip-character-guide-transparent.png`

**Interfaces:**
- Consumes: 기존 청록 지도 핀·집 프레임 이미지
- Produces: 세로로 긴 배형 머리, 솟은 눈두덩, 날렵한 아래턱과 점무늬를 가진 RGBA PNG

- [ ] **Step 1: 이미지 편집 생성**

  공통 프레임과 마젠타 바깥 배경을 고정하고 두꺼비만 길잡이 구조로 교체한다.

- [ ] **Step 2: 하드 크로마키 제거**

  Run: `python3 /Users/jeongjaemin/.codex/skills/.system/imagegen/scripts/remove_chroma_key.py --input work/imagegen/dukkeobijip-character-guide-magenta.png --out outputs/dukkeobijip-character-guide-transparent.png --auto-key border --tolerance 48`

- [ ] **Step 3: 시각 검수**

  집지킴이와 다른 세로형 실루엣, 높은 눈 위치, 가벼운 하체가 명확한지 확인한다.

### Task 3: 개성 강한 진짜 두꺼비 생성

**Files:**
- Reference: `work/imagegen/dukkeobijip-pin-teal-roof-confident-magenta.png`
- Create: `work/imagegen/dukkeobijip-character-authentic-magenta.png`
- Create: `outputs/dukkeobijip-character-authentic-transparent.png`

**Interfaces:**
- Consumes: 기존 청록 지도 핀·집 프레임 이미지
- Produces: 비대칭 저상형 머리, 가로형 동공, 단순화된 피부 돌기와 반점을 가진 RGBA PNG

- [ ] **Step 1: 이미지 편집 생성**

  공통 프레임과 마젠타 바깥 배경을 고정하고 두꺼비만 실제 두꺼비 특징을 단순화한 구조로 교체한다.

- [ ] **Step 2: 하드 크로마키 제거**

  Run: `python3 /Users/jeongjaemin/.codex/skills/.system/imagegen/scripts/remove_chroma_key.py --input work/imagegen/dukkeobijip-character-authentic-magenta.png --out outputs/dukkeobijip-character-authentic-transparent.png --auto-key border --tolerance 48`

- [ ] **Step 3: 시각 검수**

  피부 특징이 로고 수준으로 단순하며 다른 두 방향보다 종 차별성이 강한지 확인한다.

### Task 4: 공통 파일 검증

**Files:**
- Test: `outputs/dukkeobijip-character-guardian-transparent.png`
- Test: `outputs/dukkeobijip-character-guide-transparent.png`
- Test: `outputs/dukkeobijip-character-authentic-transparent.png`

**Interfaces:**
- Consumes: 세 RGBA PNG
- Produces: 크기·알파·색상 검증 결과와 최종 다운로드 파일

- [ ] **Step 1: 픽셀 검증**

  Pillow로 RGBA 모드, 동일 크기, 네 모서리 알파 0, 중앙 핀 내부 알파 255, 마젠타 잔여 0을 확인한다.

- [ ] **Step 2: 축소 미리보기 검수**

  각 결과를 128px 수준으로 확인해 실루엣과 눈 구조가 구분되는지 검수한다.

- [ ] **Step 3: 최종 산출물 전달**

  `outputs/`의 PNG 3개를 각각 미리보기와 다운로드 링크로 제공한다.
