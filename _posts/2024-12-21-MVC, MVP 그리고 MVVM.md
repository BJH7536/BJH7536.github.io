---
title: MVC, MVP 그리고 MVVM
date: 2024-12-21 +0900
categories: [C#]
tags: [C#, MVC, MVP, MVVM, 디자인 패턴, 아키텍처 패턴]
math: true
mermaid: true
---

> [**UI에 걸맞는 MVC, MVP, MVVM 패턴**](https://youtu.be/fxlYxhhf83s?si=t_kaKGxbIwD2Puql)

> 본 글은 위 영상을 참고하여 만든 정리본임.

## MVC, MVP 그리고 MVVM

- S/W의 UI 개발에 사용되는 패턴
- 모두 UI와 로직의 분리가 목적인 패턴들
- 불필요한 종속 관계를 줄인다.
- SoC(Separation of Concerns, 관심사 분리) 측면
- 아키텍처 패턴 Architecture Pattern 으로도 분류된다.
- 잠재적인 스파게티 코드의 발생을 방지
- 신입 프로그래머 혹은 지망생들이 근시일 내 쓰게 될 패턴
- UI/UX 프레임워크에 의존적

## MVC (Model-View-Controller)

![image (9)](https://github.com/user-attachments/assets/cd3d0f30-9a63-4229-940c-1f63d6122ca8)

- S/W의 UI 개발에 일반적으로 사용되는 디자인 패턴
- S/W의 논리적 부분을 데이터와 프레젠테이션에서 분리
- 불필요한 종속성과 스파게티 코드를 줄인다.
- 각 레이어의 기능을 제한한다.
    
    ⇒ 단일 책임 원칙 Single Responsibility Principle (SRP) in SOLID
    

### 모델 Model

- 어플리케이션 데이터를 관리한다.
- 데이터 컨테이너의 역할
- 게임플레이 로직의 수행이나 계산을 담당하지 않는다.

### 뷰 View

- 데이터를 사용자에게 표시한다.
- 화면에 데이터의 그래픽을 표현하는 인터페이스
    
    ⇒ UI의 역할
    

### 컨트롤러 Controller

- 사용자의 입력과 어플리케이션의 로직을 처리한다.
- 게임 데이터를 처리하고 런타임에 값이 어떻게 변하는지 계산한다.
- 입력을 처리하고 그 결과를 모델에게 다시 전송한다.
- 그 자체로 게임 데이터를 포함하지 않는다.

유니티에서, 특히나 UGUI를 통해 UI 개발을 진행하면서 MVC 패턴을 적용할 때, 모델 데이터의 변경사항을 수신하기 위한 뷰별코드가 필요하게 된다.

그래서 주로 MVC 보다는 MVP 패턴을 채용한다.

## MVP (Model-View-Presenter)

![image (10)](https://github.com/user-attachments/assets/605f7c5d-6a15-4aac-ad74-1d3b7c5f4846)

- Model과 View의 분리가 MVP의 주된 목표다.
- MVC에 비해서 더 나은 확장성, 모듈성, 재사용성 및 유지 관리성을 갖는다.

### 모델 Model

- MVC의 모델과 유사.
- 데이터와 데이터의 관리 규칙이 포함된다.
- View에 대한 정보를 갖지 않는다.

### 프레젠터 Presenter

- Model과 View 를 중재한다.
- View에서 **사용자 입력 이벤트를 처리**한다.
- 게임의 조건이 변경되면 Model을 업데이트하고, View를 업데이트한다.

### 뷰 View

- 사용자에게 데이터를 표시하는 **애플리케이션의 UI 역할**을 한다.
- 버튼의 클릭과 같은 **사용자 상호작용을 Presenter로 전송**한다.
- Presenter로부터 받은 **수정된 데이터에 맞춰 UI를 갱신**한다.
- MVP 패턴의 View는 Model과 직접 상호작용하지 않는다.
- UI 로직에 대한 별도의 스크립트가 있을 수 있지만, 게임의 **비즈니스 로직**을 처리하지는 않는다.

## MVVM (Model-View-ViewModel)

![image (11)](https://github.com/user-attachments/assets/bb6b39dc-07ef-46ed-be39-90b2d6547f1f)

- 기존의 MVP에서 View와 Controller/Presenter 의존성을 약화한 형태.
- **데이터 바인딩**
    - MVP의 View와 Presenter는 서로를 직접 참조하지만, MVVM의 View와 ViewModel은 바인딩 시스템을 통해 서로 간의 의존성을 약화한다.
    - 데이터 바인딩을 통해 **ViewModel 속성 변경이 자동으로 View에 반영**된다.
- 커맨드
    - ViewModel에서 UI 상호작용을 처리하는 커맨드 패턴을 사용한다.
- 관심사 분리 (SoC)
    - ViewModel은 View에 대한 참조를 가지지 않으므로 테스트가 용이해진다.
- 마크업파일, 데이터바인딩

### 모델 Model

- 비즈니스 로직에서 사용되는 데이터를 포함하는 데이터 구조
- 어떤 객체든 될 수 있다. (ScriptableObject, MonoBehavior)

### 뷰 View

- 사용자에게 데이터를 표시하고 상호작용하는 UI
- 일반적으로 UI Element로 구성
- UI Toolkit에서는 USS 스타일 시트와 UXML 파일로 구성
- View에 포함된 로직은 예를 들어 UI Element의 스타일과 관련

### 뷰모델 ViewModel

- View와 Model 사이의 중재자 역할
- 사용자가 View와 상호작용할 때 Model을 업데이트하는 역할
- Model이 변경될 때 View를 업데이트하는 로직도 포함된다.
- ViewModel과 View는 **데이터 바인딩**을 통해 연결된다.

## Data Binding

- UI가 아닌 개체의 속성과 UI 요소 간의 동기화를 보장한다.
- 바인딩은 본질적으로 UI가 아닌 속성과 이를 수정하는 UI 요소 간의 링크
- 바인딩을 설정하면 기본 데이터와 해당 시각적 요소 간의 변경 사항이 자동으로 동기화된다. 이렇게 하면 각 UI 업데이트에 대해 수동으로 이벤트 핸들러를 작성할 필요가 없다.
- Unity 6의 UI Toolkit은 이제 런타임 데이터 바인딩을 지원한다. 이 기능을 사용하면 런타임 UI 작업 중에 C# 개체의 속성을 UI 컨트롤 속성에 바인딩할 수 있다. 직렬화된 데이터용이 아닌 이상 편집기 UI에서도 사용할 수 있다.

## 표를 통한 비교

| 패턴    | 장점                           | 단점 | Unity 적용 시 유의점 |
| ------- | ------------------------------ | ---- | -------------------- |
| **MVC** | - 구조가 단순하고 이해가 쉽다. <br> - 빠른 프로토타이핑에 유리하다. <br> - 컨트롤러 중심으로 흐름 파악이 간단 | - 규모가 커질수록 **Controller**가 비대해질 수 있다. <br> - View와 Controller 간 결합도가 올라갈 수 있다. | - UGUI 사용 시, Model 변경사항을 View에 반영하려면 Observer/이벤트 코드가 많아질 수 있음. <br> - 소규모 프로젝트나 간단한 UI에 적합 |
| **MVP** | - **View**와 **Model** 완전 분리가 쉬움. <br> - 테스트와 유지보수에 용이. <br> - 재사용성과 모듈성이 개선됨 | - **Presenter**가 비대해지면 또 다른 거대 클래스가 될 수 있다. <br> - 인터페이스나 이벤트 처리 코드가 다소 많아진다. | - Unity에서 UGUI, UI Toolkit 등과 함께 쓸 때, **View** 인터페이스 혹은 베이스 클래스를 정의하면 편리. <br> - **Presenter**가 Model-View 간 모든 상호작용을 중재 |
| **MVVM** | - **데이터 바인딩**으로 View와 ViewModel 간 의존성을 크게 줄임. <br> - UI 업데이트 로직이 자동화되어 깔끔. <br> - 테스트가 용이 | - 러닝 커브가 존재(특히 C# `INotifyPropertyChanged` 등 개념 필요). <br> - 바인딩 과다 사용 시 성능 문제가 생길 수도 있음. | - Unity 2021+ / 2023+의 **UI Toolkit**에서 런타임 데이터 바인딩 지원이 향상. <br> - **ScriptableObject**나 MonoBehaviour를 직접 ViewModel로 쓰진 않도록 주의. <br> - 프로젝트 규모가 크거나 동적 UI가 많은 경우 추천 |