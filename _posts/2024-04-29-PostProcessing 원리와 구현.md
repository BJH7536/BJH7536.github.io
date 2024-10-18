---
title:  포스트 프로세싱✨ 원리와 구현 정리본
date:   2024-04-29 +0900
categories: [Unity]
tags: [Unity, PostProcessing]
math: true
mermaid: true
---


> [포스트 프로세싱✨ 원리와 구현](https://www.youtube.com/live/9_QwGdL_gvA?si=SMlaHL1xhO7gdl4z)

> 본 글은 위 영상을 참고하여 만든 정리본임.

---

# PostProcessing

## PostProcessing 원리

### UI 역시 메시

- Vertex들, Edge들, Polygon들이 모여 만들어진 메시.
- PostProcessing도 마찬가지

**PostProcessing도 결국 화면크기의 사각형 (삼각형 두개로 이루어진)에 Shader를 적용하는 과정.**

- 이 사각형도 결국 Mesh.
- 말한 것처럼, Shader를 활용한다.
- 다른 Shader와는 Rendering Pipeline 상의 위치가 다르다.

---

카메라로 촬영한 게임 공간 내의 한 현상을 화면에 렌더링할 때, 현재 프레임에서 렌더링하는 버퍼를 ***Back Buffer***라고 한다.

***Back Buffer***, 화면 앞 (Front)이 아닌 ***Back Buffer***라고 하는 가상의 버퍼에 카메라가 촬영한 이미지의 정보가 매 프레임마다 써진다.

그렇게 한 이미지의 정보가 버퍼에 꽉 차면, ***Flip***, ***Front Buffer***와 뒤집어서 화면에 시각화한다.

***Back Buffer***가 한 프레임에 이미지 정보를 쓰는동안, ***Front Buffer***는 이미 다 그려진 이미지를 우리에게 보여주고 있는 것이다.

> ***PostProcessing***은 이 ***Back Buffer***에 그려진 이미지 정보를 ***Off Screen Render Target (혹은 Off Screen Render Texture)*** 라는 대상에 그린 후, Texture와 Shader를 이용해 추가적인 이미지 처리 효과를 추가하는 과정이다.
> 
- ***Off Screen Render Target (혹은 Off Screen Render Texture) 이란?***
    - 화면에 직접 그려지기 전에 텍스처, 쉐도우 맵, 또는 다른 시각적 효과를 미리 계산하고 준비하는 데 사용되는 대상.

---

당연히, 이는 **프레임 디버거(Framge Debugger)**로 확인하는 것도 가능하다.

- ***Renderer Pipeline***의 과정을 모두 들여다 볼 수 있는 것 같다.

### 셋업 실습

- **[ShaderGraph & Meterial 셋업하기]**
    
    Project Window 에서 `Create > Shader Graph > URP > Full Screen Shader Graph` 를 통해 셰이더 그래프를 만듦.
    
    이를 활용할 Meterial도 필요함.
    
    ![스크린샷 2024-03-12 205752](https://github.com/user-attachments/assets/a1aba921-2876-448a-a93f-8585dc1e7ce8)
    
    ![Untitled (11)](https://github.com/user-attachments/assets/e2e863f4-9dba-464c-bd0d-19d8da05bc7e)
    
    만든 셰이더 그래프를 Meterial로 드래그하여 Material에 Shader를 적용해준다.
    
    ![Untitled (12)](https://github.com/user-attachments/assets/ed764d94-2081-4e79-ba7d-7229616edffb)
    
    ShaderGraph를 선택하면 위와 같은 ShaderGraphWindow가 열린다.
    
- **[Shader Material 적용하기]**
    
    ![Untitled (13)](https://github.com/user-attachments/assets/daa90e76-2ed0-4e39-acb1-79cada2870cc)
    
    현재 설정된 Renderer Asset에 왼쪽과 같이 **Full Screen Pass Renderer Feature** 를 추가.
    
    ![Untitled (14)](https://github.com/user-attachments/assets/965b0383-fd0e-4d13-8b4b-81cbd5817229)
    
    여기 Pass Material 에 직접 만든 Material을 넣어주면 된다.
    
- **[Full Screen Pass Renderer Feature 에 대해서]**
    
    ![Untitled (15)](https://github.com/user-attachments/assets/d708fbbb-9c8d-4a0f-bf76-80bf85d1c165)
    
    - **Injection Point**
        
        이 효과가 언제 적용될지를 선택하는 옵션
        
        - **Before Rendering Transparents**: skybox pass 의 이후에, 그리고 transparents pass 직전에 적용.
        - **Before Rendering Post Processing**: transparents pass 이후에, 그리고 post-processing pass 직전에 적용.
        - **After Rendering Post Processing**: post-processing pass 이후에, 그리고 AfterRendering pass 직전에 적용.
        
        각각의 옵션이 어떤 시각적 변화를 만드는지는 직접 변화를 보면서 적용하는게 좋을 듯.
        
- **[Shader Node]**
    
    ![Untitled (16)](https://github.com/user-attachments/assets/9a1278f2-9a10-4581-90a2-09c2317ba2a5)
    
    - ***Scene Color 노드***와 ***URP Sample Buffer 노드***
    - 둘 모두 동일한 기능을 하지만,
        
        후처리에 적합한 건 ***URP Sample Buffer 노드.***
        
    - ***URP Sample Buffer***는 URP에서만 쓸 수 있다.
        - *Source Buffer* 선택이 가능한데,
            - *NormalWorldSpace* : Scene에 존재하는 표면의 모든 픽셀에서 법선벡터를 가져온다
            - *MotionVectors* : 주어진 객체가 프레임 사이에서 얼마나 멀리 이동하는지 나타내는 벡터를 제공한다.
            - *BlitSource* : 모든 객체의 색상을 제공한다.
                
                
        
    

## GrayScale

말그대로 그냥 흑백사진으로 만들어내는 것.

### 실습

![Untitled (17)](https://github.com/user-attachments/assets/df296d9e-6925-4144-a53f-466c1d68af4c)

GrayScale을 적용하기 위한 ShaderGraph. 내적을 활용한다.

- 여기서는 RGB의 가중치가 모두 0.33인데, 
실제 사람의 눈은 B에 대한 민감도가 낮다.
- 그래서 이 가중치를 실제로는 살짝식 변화를 주어 사용한다.
    - 대체로 B에 대한 가중치가 낮아지는 편으로.

![Untitled (18)](https://github.com/user-attachments/assets/0bf14210-80e9-426a-a4c0-cda7c83f6e6b)

GrayScale이 적용된 화면

![Untitled (19)](https://github.com/user-attachments/assets/15b331fb-4027-4bdd-8939-105650a48ab2)

가중치에 변화를 준 GrayScale

좌측의 화면은 GrayScale의 RGB 가중치가 각각 0.21, 0.71, 0.07로 설정된 화면.

![Untitled (20)](https://github.com/user-attachments/assets/eb3be27b-cd8b-4ebf-aa22-6628a95e7007)


### 컬러 그레이딩

- 이미지나 비디오의 색조, 채도, 밝기 등을 조정하여, 특정한 분위기나 스타일을 만들어내는 고급 색상 조정 과정.

## Posterization

- 색상 범위를 의도적으로 줄이는 이미지 처리 기술
- 원본의 많은 색상을 더 적은 수의 색상으로 단순화 ⇒ 결과적으로 이미지가 마치 포스터처럼 단순화된 색상 구성을 가지게 된다.
- 사람의 눈은 색마다 다른 민감도를 가지고 있고, 채도보다 명도에 민감하다.
    - 그러니 명도를 기준으로, 포스터라이제이션을 적용해보자.
- URP는 기본적으로 LinearSpace 선형공간
- 채도, 휘도, 명도 ⇒ HSV 공간으로 바꿔줄건데, 감마공간에 있는걸 가져다가…

### HSV 색 공간

![Untitled (21)](https://github.com/user-attachments/assets/c1a73e50-00b4-48c9-b008-e4b14b0141c0)

위키백과의 HSV 색 공간 원뿔 모형

> 색을 표현하는 하나의 방법이자, 그 방법에 따라 색을 배치하는 방식. 
색상 (Hue), 채도 (Saturation), 명도 (Value)의 좌표를 써서 특정한 색을 지정한다.
> 

### Gamma Color Space vs **Linear Color space**

- 베버의 법칙과 관련이 있는데,
- 인간의 감각 인식이 선형적이지 않다는 점에서 비롯된다. 인간의 시각은 특히 저조도에서 더 민감하여, 높은 밝기 수준에서는 더 많은 밝기 변화를 필요로 한다.
- 이 원리가 감마 보정 곡선을 설계할 때 고려된다.
- 감마 보정은 밝기의 선형적 증가와는 달리, **인간의 시각이 감지하는 방식을 반영하여 비선형적으로 밝기를 조정**

**Gamma Color Space**

- **비선형 밝기 조정**: 감마 값(γ)을 사용하여 이미지의 밝기를 비선형적으로 조정한다. 감마 값이 1보다 클 경우, 중간 밝기의 세부 정보가 강조되며, 1보다 작을 경우에는 어두운 영역의 세부 정보가 더 잘 보이게 된다.
- **인간의 시각에 최적화**: 인간의 눈은 밝은 영역보다 어두운 영역의 변화에 더 민감하다. 감마 보정을 통해 이러한 인간의 시각적 특성을 반영하여, 제한된 비트 수로도 더 효과적으로 밝기 정보를 표현할 수 있다.

**Linear Color space**

- **선형적 밝기 표현**: 색상의 밝기가 선형적으로 증가하며, 이는 광원의 실제 특성과 일치한다.
- **물리적 기반 렌더링(PBR)에 적합**: Linear Color Space는 물리적으로 정확한 렌더링을 위한 기반이 된다. 3D 그래픽스와 시각 효과에서, 광원, 재질, 카메라 등의 상호 작용을 더 정확하게 계산할 수 있게 한다.

### 실습

![Untitled (21)](https://github.com/user-attachments/assets/c1a73e50-00b4-48c9-b008-e4b14b0141c0)

![Untitled (22)](https://github.com/user-attachments/assets/b61fad89-3989-4b2d-a8ca-eb491a31ffd3)

poster count라는 float는 5로 설정되어있다. 

- 과정은 생각보다 복잡하지 않다.
- 처음에 Scene의 색(RGB)을 감마보정해주고, 색 공간을 RGB에서 HSV로 바꿔준다. 명도를 기준으로 포스터라이제이션하기 때문에.
- 그리고 그렇게 추출한 H, S, V 중 V인 Value, 명도를 임의로 정한 상수와 Multiply, Round, Divide 연산을 통해 연속적인 변화를 계단형 변화로 바꾸어준다.
- 그리고 다시 성분들을 재조립하고, RGB 색 공간으로 바꾸고, 감마보정을 풀어 적용한다.

## Blur

![Untitled (23)](https://github.com/user-attachments/assets/8ff5b187-b08d-4a7f-80e1-b2db3a7d50d8)

- 가장 널리 쓰이는게 **가우시안 블러**
    - 이미지에 부드러운 흐림 효과를 주어 디테일을 줄이고 전체적인 시각적 노이즈를 감소시키는 데 사용
    - 가우스 분포(정규 분포)를 기반으로 하여 이미지의 각 픽셀에 가중치를 부여하여 흐림 효과를 적용한다.
    - 실시간 (Real-Time)으로 이를 적용하는 데 있어서는 성능을 고려해 대상화소를 기준으로 3x3 샘플링을 하는 편이 일반적.

### 실습

- 서브 그래프를 만들기 위해서, ***URP Sample Buffer*** 대신 ***Scene Color*** 노드를 사용
- 효과가 시각적으로 잘 보이게 하기 위해서 단위를 1이 아닌 5로 수정
- 퀄리티를 높이려면, 샘플링하는 영역을 3x3에서 더 키울 것.
    - 그러나 이를 키우면, 대역폭이 늘어나면서 성능상의 이슈가 될 수 있으므로 주의할 것.

![Untitled (24)](https://github.com/user-attachments/assets/e23fbcc0-2eb8-4e8e-827d-a518966077de)

Blur효과가 적용된 화면

![Untitled (25)](https://github.com/user-attachments/assets/effd6fd5-7a9a-4185-a740-fbdc1eddd76b)

- SubGraph를 활용해 Blur 효과를 구현하는 Shader Graph

![Untitled (26)](https://github.com/user-attachments/assets/9bbace81-0ffe-4281-ad44-8e174657d92a)

- Blur효과 구현을 위해 사용된 SubGraph의 Graph
- 하나의 UV좌표를 선택할 때 지정된 offset만큼 조정된 위치의 좌표를 선택하고, 지정된 weight 만큼의 가중치를 곱하여 결과로 출력한다.
- 이러한 SubGraph들을 3x3 총 9개의 픽셀에 대해 Blur 효과를 적용한 결과가 좌측의 이미지에 나타나 있다.

## Depth of Field 피사계심도

- 배경과 전경의 흐림 정도를 조절하여 시각적 주목도를 조정하는 데 사용
- 피사계심도가 좁은 경우, 즉 깊이가 얕은 경우, 사진의 일부분만 초점이 맞고 나머지 부분은 흐려지게 된다. 이는 주제를 강조하고 배경의 방해 요소를 줄이는 효과를 준다.
- 반면, 피사계심도가 넓은 경우, 사진의 대부분 또는 전체가 선명하게 나타난다.
- 성능상의 비용이 큰 효과. 모바일 기기에서 사용은 비추천한다고 한다.

### 실습

- 직전에 만든 Blur 그래프를 활용한다.
- 대상 거리, 혹은 대상 물체를 제외한 부분을 흐리게 하기 때문에 Blur효과가 사용된다.

![Untitled (27)](https://github.com/user-attachments/assets/7d3ea872-a424-44ff-bdb0-b6e45a166e09)

![Untitled (28)](https://github.com/user-attachments/assets/ed656f8d-b9bd-43a9-b72b-871eeee2ff24)

카메라의 Clipping Planes - Far 의 설정을 기본 3000에서 100으로 줄이고 효과를 적용한 화면

- 이전에 만든 Blur 그래프과, 기본적인 Scene의 모습을 Lerp한다.
- Lerp의 세번째 인자는, 피사계 심도이기 때문에 Scene Depth 즉 거리를 활용한다.
- ***Scene Depth*** 노드
    - 카메라로부터의 깊이 정보를 가져오는 데 사용
    - Sampling 옵션
        - **Linear01**
            - 깊이 값을 선형화하여 0과 1 사이의 값으로 정규화한다.
            - 이때, 카메라가 닿을 수 있는 가장 먼 거리가 1로 반환된다.
            - 카메라의 Clipping Planes - Far 의 설정이 매우 멀리 설정되어있으면 
            효과가 미미하게 적용되니, 이를 줄이는것을 추천
        - **Raw**
            - 카메라의 깊이 버퍼에서 직접 읽은 원시 깊이 값
            - 렌더링 시스템의 낮은 수준의 기능을 직접 제어하거나, 
            깊이 값에 대한 더 세밀한 조정이 필요한 경우 사용할 수 있겠다.
        - **Eye**
            - 객체가 카메라로부터 실제 거리를 선형 단위로 반환한다. 
            이 값은 선형화된 값으로, 실제 세계의 거리 측정과 관련된 계산에 직접 사용될 수 있다.
        

### Fog 실습

- 환경에 대기나 안개 효과를 추가하여 시각적 깊이감과 분위기를 만드는 기술
- 시야 거리를 제한하여 먼 배경이 점차적으로 사라지게 함으로써, 실제 세계에서 관측할 수 있는 대기 산란 현상을 모방
- 렌더링 성능 최적화와 시각적 몰입감 향상에도 기여

![Untitled (29)](https://github.com/user-attachments/assets/0e622adf-3627-425b-8065-9c0a074dc6a1)

start는 10, end는 20으로 설정된다.

- 좌측의 그래프는 Fog효과를 만들기 위한 Shader Graph.
- Lerp의 두번째 인자는 이전 실습에서 만든 Blur
- 이는 영상에서 설명한 공식을 구현한 것인데,
    
    ![Untitled (30)](https://github.com/user-attachments/assets/a627cca5-e844-4c04-a079-108360e7f14e)
    
    S는 시작점 start, E는 end 끝점. 
    각각이 안개효과가 시작되는 지점과 효과가 최대에 도달한 지점을 의미한다.  그리고 처리하고자 하는 대상의 위치를 D라고 하고, 
    위의 공식이 성립한다.
    

![Untitled (31)](https://github.com/user-attachments/assets/87f19f55-c9a0-414b-b737-614013757e97)

- 안개로 Blur를 사용하여 외관상으로는 앞에서 한 Depth of Field와 크게 다르지 않아 보인다.
- 이전의 실습과 유의미한 차이점이라면, 
**Scene Depth**의 설정이 Eye로 바뀌어 카메라로부터의 물리적 거리를 기반으로 
효과가 적용된다는 점이다.

---

# PostProcessing의 비용

- 모든 포스트 프로세싱은 각각의 비용이 다르다.
- 효과에 따라, 옵션에 따라 모바일에서도 무리없이 적용가능한 효과가 있기도하고, 그렇지 않은 것도 있으니 잘 알아보고 사용할 것.
- 성능에 관한 부분은 유니티 매뉴얼말고도 다양한 정보를 찾아서 활용해야 할 듯.