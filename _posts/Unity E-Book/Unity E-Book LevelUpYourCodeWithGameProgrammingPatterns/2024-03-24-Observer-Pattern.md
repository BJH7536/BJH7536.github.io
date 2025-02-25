---
title:  Observer Pattern
date:   2024-03-24 +1000
categories: [Unity E-book]
tags: [디자인 패턴, Unity, C#, Unity E-book, LevelUpYourCodeWithGameProgrammingPatterns]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.


실행 시간에 게임에서 여러 가지 일이 발생할 수 있다. 적을 파괴했을 때는 어떤 일이 일어날까? 파워업을 수집하거나 목표를 완수했을 때는 어떠할까? 종종 일부 객체가, 불필요한 의존성이 생기게,  다른 객체에게 직접 참조하지 않고도 알릴 수 있는 메커니즘이 필요하다.

***Observer Pattern*** 은 이러한 문제에 대한 일반적인 해결책이다. 이를 통해 객체들이 서로 소통할 수 있지만 "일 대 다" 의존성을 사용하여 느슨하게 결합되도록 한다. 하나의 객체가 상태를 변경할 때, 모든 의존 객체들이 자동으로 알림을 받는다. 이는 많은 다른 수신자에게 방송하는 라디오 타워에 비유할 수 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/2feccd66-f3a5-4e76-8767-ab9a6505d8de)

방송하는 객체를 '***주체(subject)***'라고 부른다. 듣고 있는 다른 객체들을 '***옵저버(observer)***'라고 부른다.

이 패턴은 주체를 옵저버와 느슨하게 분리한다. 주체는 옵저버가 누구인지 정말로 알지 못하며, 신호를 받은 후 그들이 무엇을 하는지에 대해 신경 쓰지 않는다. 옵저버들은 주체에 대한 의존성을 가지고 있지만, 옵저버들 자신은 서로에 대해 알지 못한다.

## Events

옵저버 패턴은 매우 널리 사용되기 때문에 C# 언어에 내장되어 있다. 자신만의 주체-옵저버 클래스를 설계할 수 있지만, 보통은 불필요하다. 바퀴를 다시 발명하는 것에 대한 언급을 기억하는가? C#은 이미 이 패턴을 이벤트를 사용하여 구현하고 있다.

이벤트는 단순히 무언가 발생했다는 것을 나타내는 알림이다. 이것은 몇 가지 부분을 포함한다:

- 주체 (subject) 는 특정 함수 시그니처를 설정하는 델리게이트를 기반으로 이벤트를 생성한다. 
  이벤트는 주체가 실행 시간에 수행할 어떤 행동이다 (예: 데미지를 받음, 버튼을 클릭함 등).

- 옵저버(observer) 는 델리게이트의 시그니처와 일치해야 하는 이벤트 핸들러라고 불리는 메소드를 각각 만든다.

-  각 옵저버(observer) 의 이벤트 핸들러는 주체 (subject) 의 이벤트에 구독한다. 필요한 만큼 많은 옵저버가 구독에 참여할 수 있다. 그들 모두는 이벤트가 발동되기를 기다린다.

- 주체 (subject) 가 실행 시간에 이벤트의 발생을 신호할 때, 이를 이벤트를 발생시킨다고 한다. 이는 차례로 구독자의 이벤트 핸들러를 호출하며, 이들은 응답으로 자신의 내부 로직을 실행한다.

이러한 방식으로, 주체로부터의 단일 이벤트에 많은 컴포넌트가 반응하도록 만들 수 있다. 주체가 버튼이 클릭되었다고 표시하면, 옵저버들은 애니메이션 또는 사운드를 재생하거나, 컷신을 트리거하거나, 파일을 저장할 수 있다. 그들의 반응은 무엇이든 될 수 있으며, 이것이 바로 객체 간 메시지를 보내는 데 옵저버 패턴이 자주 사용되는 이유다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/a80bf8d0-e6ce-4f2f-b446-f5150db25702)

## 예제 : 간단한 주체와 옵저버

예를 들어, 기본적인 주체/발행자를 다음과 같이 정의할 수 있다:

```cs
using UnityEngine; 
using System; 

public class Subject: MonoBehaviour 
{ 
    public event Action ThingHappened; 
    public void DoThing() 
    { 
        ThingHappened?.Invoke(); 
    } 
}
```

여기서, `MonoBehaviour`를 상속받아 `GameObject`에 더 쉽게 부착할 수 있지만, 이것이 필수는 아니다.

자신만의 커스텀 델리게이트를 정의할 수 있지만, 대부분의 경우 System.Action이 작동한다. 이벤트와 함께 매개변수를 보내야 할 경우, `Action<T>` 델리게이트를 사용하고 꺾쇠괄호 안에 `List<T>`로 매개변수를 전달한다(최대 16개의 매개변수까지).

`ThingHappened`는 실제 이벤트로, 주체가 `DoThing` 메소드에서 호출한다.

이벤트를 듣기 위해, 예제 `Observer` 클래스를 구축할 수 있다. 여기서 편의를 위해 `MonoBehaviour`를 상속받지만, 이것도 필수는 아니다.

```cs
public class Observer : MonoBehaviour 
{ 
    [SerializeField] private Subject subjectToObserve; 
    
    private void OnThingHappened() 
    { 
        // any logic that responds to event goes here 
        Debug.Log("Observer responds"); 
    } 
    
    private void Awake() 
    { 
        if (subjectToObserve != null) 
        { 
            subjectToObserve.ThingHappened += OnThingHappened; 
        } 
    } 
    
    private void OnDestroy()
    { 
        if (subjectToObserve != null) 
        { 
            subjectToObserve.ThingHappened -= OnThingHappened; 
        } 
    } 
}
```

이 컴포넌트를 `GameObject`에 부착하고 인스펙터에서 `subjectToObserver`를 참조하여 `ThingHappened` 이벤트를 듣도록 설정한다.

`OnThingHappened` 메소드는 이벤트에 대한 옵저버의 응답으로 실행되는 모든 로직을 포함할 수 있다. 개발자들은 종종 이벤트 핸들러를 나타내기 위해 "On" 접두사를 추가한다 (스타일 가이드에서 사용하는 명명 규칙을 따르면 된다).

`Awake`나 `Start`에서 += 연산자를 사용하여 이벤트를 구독할 수 있다. 이는 옵저버의 `OnThingHappened` 메소드를 주체의 `ThingHappened`와 결합한다.

무언가가 주체의 `DoThing` 메소드를 실행하면, 그것이 이벤트를 발생시킨다. 그런 다음, 옵저버의 `OnThingHappened` 이벤트 핸들러가 자동으로 호출되어 디버그 문을 출력한다.

- **주의** : 실행 시간 동안 옵저버를 삭제하거나 제거하는데, 여전히 `ThingHappened`에 구독된 상태라면, 그 이벤트를 호출하면 에러가 발생할 수 있다. 따라서, -= 연산자를 사용하여 `MonoBehaviour`의 `OnDestroy` 메소드에서 이벤트 구독을 취소하는 것이 중요하다.

게임 플레이 동안 발생하는 거의 모든 것에 옵저버 패턴을 적용할 수 있다. 예를 들어, 플레이어가 적을 파괴하거나 아이템을 수집할 때마다 게임이 이벤트를 발생시킬 수 있다. 점수나 업적을 추적하는 통계 시스템이 필요한 경우, 옵저버 패턴을 사용하면 원래 게임 플레이 코드에 영향을 주지 않고 하나를 만들 수 있다.

많은 유니티 애플리케이션에서 이벤트를 다음에 적용한다: 

- 목표 또는 골 
- 승/패 조건
- PlayerDeath, EnemyDeath, 또는 Damage 
- 아이템 픽업
- 사용자 인터페이스

주체는 적절한 시간에 이벤트를 발생시키기만 하면 되며, 그러면 어떤 수의 옵저버든지 구독할 수 있다.

샘플 프로젝트에서, `ButtonSubject`는 사용자가 마우스 버튼으로 `Clicked` 이벤트를 호출할 수 있게 한다. 그런 다음 `AudioObserver`와 `ParticleSystemObserver` 컴포넌트를 가진 여러 다른 `GameObjects`가 자신만의 방식으로 이벤트에 반응할 수 있다.

어떤 객체가 "주체"이고 어떤 것이 "옵저버"인지는 사용 방식에 따라 달라진다. 이벤트를 발생시키는 모든 것이 주체로 작용하고, 이벤트에 반응하는 모든 것이 옵저버이다. 같은 GameObject의 다른 컴포넌트는 주체 또는 옵저버일 수 있다. 심지어 같은 컴포넌트도 하나의 맥락에서는 주체가 되고 다른 맥락에서는 옵저버가 될 수 있다.

예를 들어, 예제의 `AnimObserver`는 클릭될 때 버튼에 약간의 움직임을 추가한다. 그것은 `ButtonSubject GameObject`의 일부임에도 불구하고 옵저버로 작용한다.

## UnityEvents 와 UnityActions

유니티는 또한 UnityEngine.Events API에서 UnityAction 델리게이트를 사용하는 UnityEvents라는 별도의 시스템을 포함하고 있다.

UnityEvents는 옵저버 패턴을 위한 그래픽 인터페이스를 제공한다. 만약 당신이 유니티의 UI 시스템을 사용해 본 적이 있다면(예: UI 버튼의 OnClick 이벤트 생성), 이미 그것에 대한 일부 경험을 가지고 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/2410435b-d016-42e6-8751-ebaa5fbddb28)

이 예에서, 버튼의 `OnClick` 이벤트가 호출되고 두 `AudioObservers`의 `OnThingHappened` 메서드로부터 반응을 유발한다. 따라서 코드 없이 주체의 이벤트를 설정할 수 있다.

UnityEvents는 디자이너나 비프로그래머들이 게임플레이 이벤트를 생성하도록 하고 싶을 때 유용하다. 하지만, 그들이 System 네임스페이스의 동등한 이벤트나 액션보다 느릴 수 있다는 점을 인지해야 한다.

UnityEvents와 UnityActions를 고려할 때 성능 대 사용성을 저울질해야 한다. Unity Learn의 "Create a Simple Messaging System with Events" 모듈에서 예를 볼 수 있다.

## 장단점

이벤트를 구현하면 약간의 추가 작업이 필요하지만 다음과 같은 장점이 있다:

#### 옵저버 패턴은 객체들을 분리하는 데 도움이 된다
    이벤트 발행자는 이벤트 구독자들에 대해 아무것도 알 필요가 없다. 
    한 클래스와 다른 클래스 사이에 직접적인 의존성을 생성하는 대신, 
    주체와 옵저버는 일정 정도의 분리를 유지하면서 소통한다.

#### 직접 구축할 필요가 없다
    C#은 이미 확립된 이벤트 시스템을 포함하고 있으며, 
    자신만의 델리게이트를 정의하는 대신 System Action 델리게이트를 사용할 수 있다. 
    또한, 유니티는 UnityEvents와 UnityActions를 포함하고 있다.

#### 각 옵저버는 자신만의 이벤트 처리 로직을 구현한다
    이 방식으로, 각 관찰하는 객체는 반응하는 데 필요한 로직을 유지한다. 
    이것은 디버깅과 단위 테스트를 더 쉽게 만든다.

#### 사용자 인터페이스에 잘 맞는다
    핵심 게임플레이 코드는 UI 로직과 별도로 존재할 수 있다. 
    그러면 UI 요소들은 특정 게임 이벤트나 조건을 듣고 적절히 반응한다. 
    MVP와 MVC 패턴은 이 목적을 위해 옵저버 패턴을 사용한다.


옵저버 패턴에 대해 알아야 할 몇 가지 주의사항이 있다:

#### 추가적인 복잡성을 도입한다
    다른 패턴들처럼, 이벤트 주도 아키텍처를 생성하는 것은 처음에 더 많은 설정을 요구한다.
    또한, 주체나 옵저버를 삭제할 때 주의해야 한다. 
    OnDestroy에서 옵저버의 등록을 해제해야 한다.

#### 옵저버는 이벤트를 정의한 클래스에 대한 참조가 필요하다
    옵저버는 여전히 이벤트를 발행하는 클래스에 대한 의존성을 가지고 있다. 
    모든 이벤트를 처리하는 정적 EventManager(아래)를 사용하면 
    객체들 사이를 분리하는 데 도움이 될 수 있다.

#### 성능이 문제가 될 수 있다
    이벤트 주도 아키텍처는 추가적인 오버헤드를 도입한다. 
    큰 Scene과 많은 GameObjects는 성능을 저하시킬 수 있다.

## 개선사항

여기서 소개된 옵저버 패턴은 기본적인 버전에 불과하지만, 게임 애플리케이션의 모든 필요사항을 처리할 수 있도록 확장할 수 있다.

옵저버 패턴을 설정할 때 이러한 제안사항을 고려해보자:

####  ObservableCollection 클래스 사용하기
    C#은 특정 변경사항을 추적할 수 있는 동적 ObservableCollection을 제공한다. 
    항목이 추가되거나 제거될 때, 또는 리스트가 새로고침될 때 옵저버에게 알림을 줄 수 있다.
    
#### 고유 인스턴스 ID를 인수로 전달하기
    계층 구조 내의 각 GameObject는 고유한 인스턴스 ID를 가진다. 
    하나 이상의 옵저버에 적용될 수 있는 이벤트를 트리거할 때, 
    이벤트에 고유 ID를 전달한다(Action<int> 타입 사용). 
    그런 다음 GameObject가 고유 ID와 일치하는 경우에만 이벤트 핸들러의 로직을 실행한다.
    
####  static 이벤트 매니저 생성하기
        이벤트가 게임플레이의 많은 부분을 주도할 수 있기 때문에, 
        많은 유니티 애플리케이션들은 정적이거나 싱글톤 EventManager를 사용한다. 
        이 방법으로, 옵저버들은 중앙의 게임 이벤트 소스를 주체로 참조하여 
        설정을 더 쉽게 할 수 있다.
		
#### 이벤트 큐 생성하기
    씬에 많은 객체들이 있을 때, 모든 이벤트를 한 번에 발생시키고 싶지 않을 수 있다. 
    단일 이벤트를 호출할 때 수천 개의 객체가 소리를 재생하는 소음을 상상해 보라.

    옵저버 패턴을 커맨드 패턴과 결합하면 이벤트를 이벤트 큐에 캡슐화할 수 있다.
    그런 다음 커맨드 버퍼를 사용하여 이벤트를 한 번에 하나씩 재생하거나 필요에 따라 
    선택적으로 무시할 수 있다 
    (예를 들어, 한 번에 소리를 낼 수 있는 객체의 최대 수가 있을 경우).

옵저버 패턴은 모델 뷰 프레젠터(MVP) 아키텍처 패턴에 깊이 관련되어 있으며, 이는 다음 장에서 더 자세히 다룬다.

## 참고한 자료
[Unity_E-Book_LevelUpYourCodeWithGameProgrammingPatterns](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)