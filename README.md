## 내가 맡은 부분 — 신동민

**기간** 2025.11.14 – 11.21 · **내 커밋** 65건

| 구현 | 내용 |
| --- | --- |
| **GPU Skinning** | 구현 후 **전역 플래그로 CPU/GPU 전환**이 가능하도록. 본 최대 개수를 늘리고 실제 본 개수만큼 버퍼·데이터를 넘기다가, 경고를 피하기 위해 최대 크기 고정 생성으로 변경. **그림자 패스에도 GPU 스키닝 적용** |
| **성능 측정(Stat)** | `GPUTimer` 기반 프로파일 구현. **GPU 타이머가 8프레임 이전 결과를 읽던 문제**를 동기화로 해결하고, 모든 뷰의 스키닝 성능을 측정해 CPU/GPU 모드를 스왑 표시 |
| **메모리 누수** | 스키닝 성능 측정용 **GPU 쿼리 링버퍼**가 원인이던 누수 해결 |
| **뷰어 다중 창** | 뷰어 간 z-order와 레이어 순서 정리, 다중 뷰어에서의 마우스·피킹, 뷰포트를 `ImGui::Image` 방식으로 전환, 좌우 패널 크기 조절 |
| **크래시 대응** | `main`을 감싸 모든 C++ 예외를 캐치, **미니덤프** 작성, 랜덤 크래시 재현 기능 |
| **기타** | 버텍스 셰이더에 **그람슈미트 직교화** 추가, 한글 경로 처리, `DrawTextBlock`이 매 호출마다 생성/파괴되던 문제 수정 |

**주요 파일** `Mundi/Source/Runtime/Engine/Components/SkinnedMeshComponent.cpp` · `Mundi/Source/Runtime/RHI/GPUTimer.cpp/h` · `Mundi/Source/Slate/Windows/SViewerWindow.cpp` · `Mundi/Source/Slate/StatsOverlayD2D.cpp`

→ **[내 커밋 65건 보기](https://github.com/TeshShin/Krafton_TechLab_Week11/commits?author=TeshShin)** · [14주 전체 정리](https://github.com/TeshShin/Krafton-TechLab-Roles)

---

## 11주차(2025.11.14 – 11.21)에 추가된 것 — 애니메이션·스키닝 (팀 전체)

- **애니메이션 데이터** — `AnimDataModel`, `AnimTypes`, `FAnimationCurveData`, FBX 애니메이션 로딩, 다중 AnimStack 지원, `UAnimSequence` 바이너리 베이킹, ResourceManager 통합
- **애니메이션 재생** — `SkeletalMeshComponent` 애니메이션 재생, 애님 스테이트 머신, Blend Space 2D
- **애니메이션 뷰어** — 에셋 브라우저, AnimNotify 편집 UI, 재생 컨트롤(To Front / Previous / Reverse / Next / End), 오디오 노티파이
- **GPU Skinning** — CPU/GPU 전환, 그림자 패스 지원
- **게임 프레임워크** — `Character`, `Pawn` + InputComponent, `CharacterMovementComponent`, `SpringArmComponent`, GameMode/GameState, Lua 기반 WASD 이동
- **프로파일링** — GPUTimer 기반 Stat 오버레이

---

# Mundi 엔진

> 🚫 **경고: 이 내용은 Mundi 엔진 렌더링 기준의 근본입니다.**  
> 삭제하거나 수정하면 엔진 전반의 좌표계 및 버텍스 연산이 깨집니다.  
> **반드시 유지하십시오.**

## 📘 기본 좌표계

* **좌표계:** Z-Up, **왼손 좌표계 (Left-Handed)**
* **버텍스 시계 방향 (CW)** 이 **앞면(Face Front)** 으로 간주됩니다.
  > → **DirectX의 기본 설정**을 그대로 따릅니다.

---

## 🔄 OBJ 파일 Import 규칙

* OBJ 포맷은 **오른손 좌표계 + CCW(반시계)** 버텍스 순서를 사용한다고 가정합니다.
  > → 블렌더에서 OBJ 포맷으로 Export 시 기본적으로 이렇게 저장되기 때문입니다.
* 따라서 OBJ를 로드할 때, 엔진 내부 좌표계와 일치하도록 자동 변환을 수행합니다.

```cpp
FObjImporter::LoadObjModel(... , bIsRightHanded = true) // 기본값
```

즉, OBJ를 **Right-Handed → Left-Handed**,  
**CCW → CW** 방향으로 변환하여 엔진의 렌더링 방식과 동일하게 맞춥니다.

---

## 🧭 블렌더(Blender) Export 설정

* 블렌더에서 모델을 **Z-Up, X-Forward** 설정으로 Export하여  
  Mundi 엔진에 Import 시 **동일한 방향을 바라보게** 됩니다.

> 💡 참고:
> 블렌더에서 축 설정을 변경해도 **좌표계나 버텍스 순서 자체는 변하지 않습니다.**  
> 단지 **기본 회전 방향만 바뀌므로**, Mundi 엔진에서는 항상 같은 방식으로 Import하면 됩니다.

---

## ✅ 정리

| 구분     | Mundi 엔진 내부 표현      | Mundi 엔진이 해석하는 OBJ   | OBJ Import 결과 |
| ------ | ----------------- | ------------------ | ----------------- |
| 좌표계    | Z-Up, Left-Handed | Z-Up, Right-Handed | Z-Up, Left-Handed |
| 버텍스 순서 | CW (시계 방향)        | CCW (반시계 방향)       | CW |

### [데모 씬용 FBX 파일](https://drive.google.com/file/d/14UviD0dfo2LsvJltEeCxywRB8f0m156E/view?usp=sharing)
