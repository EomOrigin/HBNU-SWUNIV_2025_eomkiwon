  # 한밭대학교 컴퓨터공학과 2025 포트폴리오경진대회
## 국립한밭대학교 컴퓨터공학과 20231203 엄기원

## 주제 
- 눈·코·입 영역 강조 가중치를 활용한 ViT 기반 딥페이크 탐지 기법 연구
  
## Project Background
  - ### 개요
   본 프로젝트는 Vision Transformer(ViT) 기반의 딥페이크 탐지 성능을 향상시키기 위해 눈·코·입(eyes, nose, mouth) 영역에 학습 가능한 강조 가중치(Region Weight Module, RWM)를 부여하는 기법을 제안하고 구현한 연구입니다. 얼굴 랜드마크는 MediaPipe를 통해 추출하고, 입력 이미지를 16×16 패치(총 196개)로 분할한 뒤 해당 패치들에 부위별 가중치를 적용하여 ViT의 패치 임베딩 단계에서 국소적 합성 흔적(artifact)을 더 민감하게 학습하도록 설계했습니다. 이 방법은 ViT가 갖는 전역 정보에는 강하지만 국소적 왜곡을 놓치기 쉬운 한계를 보완합니다.


<img width="519" height="217" alt="image" src="https://github.com/user-attachments/assets/99427343-8805-489f-87a7-087aa8ca904f" />

    
  - ### 필요성
   최근 딥페이크 기술의 고도화와 함께 피해를 목적으로 한 딥페이크 범죄(사칭, 금전적 사기, 명예훼손 등) 가 증가하고 있어, 단순 감시·탐지 수준을 넘어 실사용자 보호를 목표로 한 예방적·실시간 대응 기술의 도입이 시급합니다. 본 연구의 RWM 기반 딥페이크 탐지 기법은 이러한 위협에 대해 현장 적용 가능한 실시간 검증 도구로 활용될 잠재력이 높습니다.
    
## 프로젝트 내용

<img width="504" height="250" alt="image" src="https://github.com/user-attachments/assets/f9043ce0-699a-436d-aff5-6b0bc7a6c7a5" />

 - ### 구현 내용
  1.  **데이터 파이프라인 & 전처리**
      -   FaceForensics++ 등 공개 딥페이크 데이터셋을 수집하고, 학습/검증/테스트 split을 구성
      -   프레임 추출, 얼굴 검출/정렬, 패치 분할(16×16) 등 전처리 스크립트를 작성하여 일관된 입력 형식을 작성

  2.  **RWM 모듈**
      -   RegionAttention 은 입력 이미지(224×224 등)를 ViT 패치 단위(예: 16×16 → 14×14 = 196 patch)로 분할한 뒤, 눈/코/입 영역에 대한 가중치(스칼라 또는 패치별 벡터) 를 만들어 각 패치 토큰에 곱해주는 모듈
      -   image_size와 patch_size로 grid_rows/grid_cols(R, C)와 patch_h/patch_w(실패치 크기)를 계산하고, 얼굴 랜드마크 좌표를 받아, 각 landmark를 패치 인덱스 (r,c)로 변환해 region별 mask(0/1)를 생성

  3.  **최종 모델 구조**
      -   ViTWithRegionBias 클래스가 timm의 vit_small_patch16_224를 불러와 RegionAttention을 생성하고 패치 임베딩 단계 직후에 weight_map을 적용 

- ### 실험 결과 
  1.  **Baseline과 RWM 적용 후 및 가중치 모드 별 Test 성능**
     
     - 본 논문의 제안 기법인 Multiple Learnable 가중치는 Baseline 대비 최대 7.94%의 ACER 개선을 이끌어냈으며, AUC 역시 87.91%에서 90.90%로 증가하는 성능 향상이 나타났습니다. 
     
     <img width="661" height="193" alt="image" src="https://github.com/user-attachments/assets/d756ae27-f75d-4e4a-b0d1-3bb8ff5a5e17" />

  2.  **5-point 랜드마크 세트와 Full-point 랜드마크 세트의 성능 비교**
     
    - 양쪽 눈 동공, 코끝, 입술 양끝의 5개 대표 지점을 담은 5-point set와  부위의 모든 세부 지점을 포함한 Full-point 랜드마크 세트를 비교할 경우, Full-point 랜드마크 세트가 5-point 랜드마크 세트보다 더 높은 성능을 보였습니다. 이는 세분화된 포인트들이 눈·코·입 부위의 미세한 합성 흔적을 더욱 정밀하게 포착하기 때문으로 해석할 수 있습니다.

    <img width="655" height="238" alt="image" src="https://github.com/user-attachments/assets/adf2052c-e792-4edc-94de-6d93e0a6071e" />

  3.  **RWM 추론 비용 비교**
  
    - RWM의 평균 추론 시간은 CBAM 대비 0.47ms(-3.91%) 더 낮았는데, 랜드마크 조회 시간과 weight map 생성·적용이라는 추가 연산을 포함하고도 RWM은 CBAM보다 오히려 더 낮은 평균 추론 시간을 달성하여, 효율적인 연산 비용을 가졌음을 증명하였습니다. 
 
    <img width="648" height="131" alt="image" src="https://github.com/user-attachments/assets/9e436fd1-9ad0-44c3-819e-63bafd5376d3" />

     
- ### 기대 효과
  1.  **딥페이크 탐지 성능 향상**: 딥페이크는 전체 얼굴이 아닌 눈·코·입 주변에서 미세한 합성 왜곡이 주로 발생하므로, 전역적 패치 처리만 하는 기존 ViT 구조는 이러한 미세 징후를 약하게 학습합니다. 해당 기법을 통해 이러한 단점을 보완하여 더욱 강건한 딥페이크 탐지가 가능합니다. 
  2.  **연산 효율성 및 실시간 추론 적용**: 랜드마크 조회와 weight map 생성 연산을 포함하더라도 RWM의 평균 추론시간은 약 12.01 ms로, 실시간·저지연 환경에서도 적용 가능함을 보였습니다.
  3.  **설명 가능성(Explainability) 제고**: RWM이 학습하는 패치별 가중치(heatmap)를 통해 모델이 어느 얼굴 부위를 근거로 판별했는지 시각적으로 확인할 수 있습니다. 이는 딥페이크 판정의 근거를 제공해 포렌식 분석·디버깅·사용자 신뢰 확보에 도움을 주며, 규제·컴플라이언스 대응이나 결과 해석이 필요한 실제 서비스 환경에서 중요한 장점이 됩니다.
 
  <img width="594" height="249" alt="image" src="https://github.com/user-attachments/assets/115c59cf-ae2e-4aa4-8dd7-a52d53fbb396" />


