---
title:  Model View Presenter and Conclusion
date:   2024-03-31 +1000
categories: [Unity E-book]
tags: [디자인 패턴, Unity, C#, Unity E-book, LevelUpYourCodeWithGameProgrammingPatterns]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.

모델 뷰 컨트롤러 ***(Model View Controller, MVC)*** 는 UI를 개발할 때 흔히 사용되는 디자인 패턴의 한 가족이다.

MVC의 일반적인 아이디어는 소프트웨어의 논리 부분을 데이터와 프레젠테이션으로부터 분리하는 것이다. 이는 불필요한 의존성을 줄이고 [스파게티 코드](https://en.wikipedia.org/wiki/Spaghetti_code)를 줄일 수 있는 잠재력을 제공한다.

## Model View Controller (MVC) Design Pattern

이름에서 알 수 있듯이, MVC 패턴은 애플리케이션을 세 개의 계층으로 분리한다

#### 모델은 데이터를 저장한다
    모델은 엄격히 데이터 컨테이너로, 값들을 보유한다. 
    게임플레이 로직을 수행하거나 계산을 실행하지 않는다. 

#### 뷰는 인터페이스다
    뷰는 데이터의 그래픽 표현을 화면에 포맷하고 렌더링한다. 

#### 컨트롤러는 로직을 처리한다
    이것을 두뇌로 생각해보자. 
    게임 데이터를 처리하고 실행 시간에 값이 어떻게 변경되는지 계산한다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/2fa0b24b-78bc-407c-be5b-d5c56e026f66)

이러한 관계의 분리는 이 세 부분이 서로 어떻게 상호작용하는지를 구체적으로 정의한다. 
모델은 애플리케이션 데이터를 관리하며, 뷰는 그 데이터를 사용자에게 표시한다. 
컨트롤러는 입력을 처리하고 게임 데이터에 대한 모든 결정이나 계산을 수행한다. 
그런 다음 결과를 모델로 다시 보낸다.

따라서, 컨트롤러는 자체적으로 게임 데이터를 포함하지 않는다. 뷰도 마찬가지다. 
MVC 디자인은 각 계층이 수행하는 역할을 제한한다. 
한 부분은 데이터를 보유하고, 다른 부분은 데이터를 처리하며, 
마지막 부분은 그 데이터를 사용자에게 표시한다.

표면적으로, 이는 단일 책임 원칙 (***Single-Responsibility Principle***) 의 확장으로 생각할 수 있다. 
각 부분은 하나의 일을 하고, 그 일을 잘 한다. 이는 MVC 아키텍처의 한 가지 장점이다.

## Model View Presenter (MVP) and Unity

Unity 프로젝트를 MVC로 개발할 때, 기존 UI 프레임워크([UI 툴킷](https://docs.unity3d.com/Manual/UIElements.html?utm_source=demand-gen&utm_medium=pdf&utm_campaign=clean-code&utm_content=game-programming-patterns-ebook) 또는 [Unity UI](https://docs.unity3d.com/Manual/com.unity.ugui.html?utm_source=demand-gen&utm_medium=pdf&utm_campaign=clean-code&utm_content=game-programming-patterns-ebook))는 자연스럽게 View로 기능한다. 엔진이 완전한 UI 구현을 제공하기 때문에, 개별 UI 컴포넌트를 처음부터 개발할 필요가 없다.

그러나, 전통적인 MVC 패턴을 따르려면 실행 시간에 모델의 데이터 변경을 듣기 위해  View 특정 코드가 필요하다.

이는 유효한 접근 방법이지만, 많은 Unity 개발자들은 Controller 가 중재자 역할을 하는 MVC의 변형을 사용하기로 선택한다. 여기서 View 는 모델을 직접 관찰하지 않는다. 대신, 다음과 같이 된다.

![MVP : MVC의 변형](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/947ece2e-50c3-4a69-bebf-aed6c7ca1e4b){: width="972" height="589" .w-50 .left .shadow}

이러한 ***MVC*** 의 변형은 ***Model View Presenter design***, 또는 ***MVP*** 라고 불린다. ***MVP*** 는 여전히 세 개의 구분된 애플리케이션 계층을 가진 관심사의 분리를 보존하지만, 각 부분의 책임을 약간 변경한다.

***MVP*** 에서는 ***Presenter*** (***MVC***에서는 ***Controller*** 라고 함)가 다른 계층 사이의 중재자 역할을 한다. ***Presenter*** 는 ***Model*** 로부터 데이터를 검색한 다음, 그 데이터를 ***View*** 에서 표시하기 위해 포맷한다. ***MVP*** 는 입력을 처리하는 계층을 switch 한다. ***Controller*** 대신 ***View*** 가 사용자 입력을 처리하는 책임을 진다.

이 디자인에서 ***Event*** 와 ***Observer Pattern*** 이 어떻게 사용되는지 주목하자. 사용자는 Unity UI의 버튼, 토글, 슬라이더 컴포넌트와 상호작용할 수 있다. ***View*** 계층은 이 입력을 UI 이벤트를 통해 ***Presenter*** 에게 전달하고, ***Presenter*** 는 차례로 ***Model*** 을 조작한다. ***Model*** 로부터의 상태 변경 이벤트는 데이터가 업데이트되었음을 ***Presenter*** 에게 알린다. ***Presenter*** 는 수정된 데이터를 ***View*** 에 전달하고, ***View*** 는 UI를 새로고침한다.

<br>

## 예제 : 체력 인터페이스

***MVP*** 예제를 공식화하기 위해, 캐릭터나 아이템의 체력 상태를 보여주는 간단한 시스템을 상상해보자. 모든 것을 데이터와 UI를 섞은 하나의 클래스에 넣을 수는 있지만, 그것은 잘 확장되지 않을 것이다. 더 많은 기능을 추가하려면 확장해야 할 필요성이 커지면서 더 복잡해질 것이다.

대신, 체력 컴포넌트를 더 ***MVP*** 중심적인 방식으로 재작성할 수 있다. 스크립트를 `Health`와 `HealthPresenter` 로 나눈다. `Health` 컴포넌트는 다음과 같을 수 있다.

```cs

public class Health: MonoBehaviour 
{ 
    public event Action HealthChanged; 
    
    private const int minHealth = 0; 
    private const int maxHealth = 100; 
    private int currentHealth; 
    
    public int CurrentHealth 
    { 
        get => currentHealth; 
        set => currentHealth = value; 
    }
    public int MinHealth => minHealth; 
    public int MaxHealth => maxHealth; 
    
    public void Increment(int amount) 
    { 
        currentHealth += amount; 
        currentHealth = Mathf.Clamp(currentHealth, minHealth, maxHealth);
        UpdateHealth(); 
    } 
    public void Decrement(int amount) 
    { 
        currentHealth -= amount; 
        currentHealth = Mathf.Clamp(currentHealth, minHealth, maxHealth);
        UpdateHealth(); 
    } 
    public void Restore() 
    { 
        currentHealth = maxHealth; 
        UpdateHealth(); 
    } 
    public void UpdateHealth() 
    { 
        HealthChanged?.Invoke(); 
    } 
}
```

이 버전에서 `Health`는 ***Model*** 로서 역할을 한다. 실제 체력 값을 저장하고 그 값이 변경될 때마다 `HealthChanged` 이벤트를 호출한다. `Health`는 게임플레이 로직을 포함하지 않고, 데이터를 증가시키고 감소시키는 메소드만 포함한다.

그러나 대부분의 객체는 `Health` 자체를 조작하지 않는다. 그 작업은 `HealthPresenter` 에게 예약된다.

```cs
public class HealthPresenter : MonoBehaviour 
{ 
    [SerializeField] Health health; 
    [SerializeField] Slider healthSlider; 
    private void Start() 
    { 
        if (health != null) 
        { 
            health.HealthChanged += OnHealthChanged; 
        } 
        UpdateView(); 
    } 
    
    private void OnDestroy() 
    { 
        if (health != null) 
        { 
            health.HealthChanged -= OnHealthChanged; 
        } 
    } 
    
    public void Damage(int amount) 
    { 
        health?.Decrement(amount); 
    } 
    public void Heal(int amount) 
    { 
        health?.Increment(amount); 
    } 
    public void Reset() 
    { 
        health?.Restore(); 
    } 
    public void UpdateView() 
    { 
        if (health == null) return; 
        if (healthSlider !=null && health.MaxHealth != 0) 
        { 
            healthSlider.value = 
                (float) health.CurrentHealth / (float) health.MaxHealth; 
        } 
    } 
    public void OnHealthChanged() 
    { 
        UpdateView(); 
    } 
}
```

다른 GameObject는 `Damage`, `Heal`, `Reset`을 사용하여 체력 값을 수정하기 위해 `HealthPresenter`를 사용할 필요가 있다. `HealthPresenter`는 일반적으로 `Health`가 `HealthChanged` 이벤트를 발생시킬 때까지 사용자 인터페이스를 `UpdateView`로 업데이트하기를 기다린다. ***Model*** 에서 값을 설정하는 데 약간의 시간이 걸리는 경우 (예를 들어, 값을 디스크에 저장하거나 데이터베이스에 저장하는 경우) 이 방법이 유용하다.

샘플 프로젝트에서, 사용자는 대상 객체를 손상시키거나 버튼으로 체력을 재설정할 수 있다. 
이는 직접 `Health`를 변경하는 대신 `HealthPresenter`(손상이나 재설정을 호출하는)에 정보를 알린다. UI 텍스트와 UI 슬라이더는 `Health`가 이벤트를 발생시키고 그 값이 변경되었음을 `HealthPresenter`에 알릴 때 업데이트된다.

## 장단점

***MVP(and MVC)*** 는 더욱 큰 애플리케이션에 정말 빛을 발한다. 게임 개발에 상당한 규모의 팀이 필요하고 출시 후 오랜 시간 동안 유지할 것으로 예상된다면, 다음과 같은 이점을 얻을 수 있다.

#### 작업의 원활한 분담
    View 를 Presenter 로부터 분리했기 때문에, 사용자 인터페이스의 개발과 업데이트는 
    코드베이스의 나머지 부분으로부터 거의 독립적으로 진행될 수 있다. 
    이를 통해 전문 개발자들 사이에서 노동을 분담할 수 있다. 
    
    팀에 전문 프론트엔드 개발자가 있는가? 그들이 View 를 담당하게 하라. 
    그들은 다른 모든 사람으로부터 독립적으로 작업할 수 있다.

#### MVP와 MVC를 사용한 단위 테스팅의 간소화
    이러한 디자인 패턴은 게임플레이 로직을 사용자 인터페이스로부터 분리한다. 
    그 결과, 에디터에서 Play 모드로 들어가지 않고도 코드와 함께 작동하는 객체를 
    시뮬레이션할 수 있다. 이는 상당한 시간을 절약할 수 있다.

#### 유지보수 가능한 읽기 쉬운 코드
    이 디자인 패턴을 사용하면 일반적으로 더 작은 클래스를 만들게 되어, 
    이들을 읽기 더 쉬워진다. 일반적으로 의존성이 적으면 
    소프트웨어가 고장 날 장소가 줄어들고 버그가 숨어 있을 가능성이 있는 장소도 줄어든다.

***MVC*** 와 ***MVP***는 웹 개발이나 기업 소프트웨어에서 널리 사용되지만, 애플리케이션이 충분한 크기와 복잡성에 도달할 때까지 종종 이점이 명확하지 않을 수 있다. Unity 프로젝트에서 이러한 패턴을 구현하기 전에 다음 사항을 고려해야 한다.

#### 미리 계획을 세워야 한다
    이 가이드에서 설명된 다른 패턴들과 달리, MVC와 MVP는 더 큰 아키텍처 패턴이다. 
    이 중 하나를 사용하려면, 클래스를 책임에 따라 분할해야 하는데, 
    이는 일정한 조직과 사전 작업이 더 많이 필요하다.

#### Unity 프로젝트의 모든 것이 패턴에 맞지는 않는다
    "순수한" MVC 또는 MVP 구현에서, 화면에 렌더링하는 모든 것은 실제로 View의 일부다. 
    Unity 컴포넌트가 데이터, 로직, 인터페이스(e.g., MeshRenderer) 사이에서 
    쉽게 분리되는 것은 아니다. 또한, 간단한 스크립트는 
    MVC/MVP에서 많은 이점을 얻지 못할 수 있다.
    
    패턴에서 가장 큰 이점을 얻을 수 있는 곳에서 판단을 행사해야 한다. 
    보통, 단위 테스트를 통해 가이드를 얻을 수 있다. MVC/MVP가 
    테스팅을 용이하게 할 수 있다면, 애플리케이션의 그 측면을 위해 그것들을 고려해볼 수 있다. 
    그렇지 않다면, 프로젝트에 패턴을 강제로 적용하려고 하지 말아야 한다.

## 결론

소프트웨어 패턴에 익숙하지 않다면, 이 가이드가 Unity 개발에서 마주칠 수 있는 가장 일반적인 패턴들을 이해하는 데 도움이 되었기를 바란다.

Prefab을 생성하는 팩토리부터 AI를 위한 상태 패턴에 이르기까지, 필요할 때 이러한 기법들을 손쉽게 사용할 수 있도록 유지하라. 언제 그리고 어떻게 디자인 패턴을 적용할지 인식하는 것은 다음 Unity 도전과제를 해결하는 데 도움이 될 수 있다. 물론, 특정 패턴을 맞추려고 애쓰지 말아야 한다; 패턴을 사용하지 않는 것이 패턴을 사용하는 것만큼 중요하다.

디자인 패턴을 올바르게 적용하면 작업 흐름을 가속화하고 반복적인 문제에 대한 우아한 해결책을 제공할 수 있다. 그러면 중요한 것에 집중할 수 있다. 플레이어에게 재미있고 독특한 경험을 제공하는 것.

그러니 바퀴를 다시 발명할 필요는 없지만, 분명히 자신만의 스핀을 추가할 수는 있다.

### 다른 디자인 패턴들

이 가이드는 컴퓨팅 및 게임 개발에서 잘 알려진 몇 가지 디자인 패턴의 일부에 대한 작은 샘플링일 뿐이다. 그 세부 사항에 대해 자세히 다루지는 않겠지만, 여러분에게 유용할 수 있는 다른 몇 가지에 대한 간략한 개요는 다음과 같다.

#### [Adapter](https://en.wikipedia.org/wiki/Adapter_pattern)
    두 관련 없는 엔티티 사이에 인터페이스(Wrapper 라고도 함)를 제공하여 
    그들이 함께 작동할 수 있게 한다.

#### [Flyweight](https://en.wikipedia.org/wiki/Flyweight_pattern)
    대량의 객체를 가지고 있다면, 기본 클래스에서 공통 속성을 공유하고 자원을 절약한다. 
    예를 들어, 숲을 디자인할 때 나무의 보편적 속성을 모두 기본 Tree 클래스에 저장한다. 
    그러면 서브클래스(예: PineTree, MapleTree 등)에서 그것들을 반복할 필요가 없다.

#### [Double Buffer](https://gameprogrammingpatterns.com/double-buffer.html)
    계산이 완료되는 동안 두 세트의 배열 데이터를 유지할 수 있다. 
    그러면 하나의 데이터 세트를 표시하면서 다른 데이터를 처리할 수 있으며, 
    이는 절차적 시뮬레이션(예: 셀룰러 오토마타)이나 화면에 렌더링할 때 유용하다.

#### [Dirty Flag](https://gameprogrammingpatterns.com/dirty-flag.html)
    게임에서 무언가 변경되었지만 비용이 많이 드는 작업이 아직 실행되지 않았을 때 
    (예: 디스크에 저장하거나 물리 시뮬레이션을 실행하는 경우) 불리언을 설정할 수 있는 기술이다.

#### [Interpreter / Bytecode](https://gameprogrammingpatterns.com/bytecode.html)
    모딩 지원을 추가하거나 비프로그래머가 게임을 확장할 수 있게 하고 싶다면, 
    사용자가 외부 텍스트 파일에서 편집할 수 있는 간소화된 언어를 생성할 수 있다. 
    바이트코드 컴포넌트는 그 해석된 언어를 C# 게임 코드로 번역할 수 있다.

#### [Subclass Sandbox](https://gameprogrammingpatterns.com/subclass-sandbox.html)
    비슷한 객체들이 다양한 행동을 가지고 있다면, 
    그 행동들을 부모 클래스에서 protected로 정의할 수 있다. 
    그러면 자식 클래스들이 새로운 조합을 만들기 위해 혼합하고 매치할 수 있다.

#### [Type Object](https://gameprogrammingpatterns.com/type-object.html)
    GameObject의 많은 변형이 있다면, 각각에 대해 서브클래스를 만드는 대신 
    모든 가능한 행동을 단일 추상 또는 부모 클래스에서 정의한다. 
    개별 객체의 특별한 특성을 코드를 변경하지 않고도 
    커스터마이징할 수 있는 별도의 데이터 파일(예: ScriptableObject)에서 구분한다. 
    예를 들어, 이를 통해 동일한 클래스에서 파생된 것처럼 보이는 
    다양한 아이템의 인벤토리를 생성할 수 있다. 게임 디자이너는 데이터 파일을 커스터마이징하여 
    각 아이템을 독특하게 만들 수 있다(예: RPG의 무기), 이 모든 것이 
    프로그래머의 도움 없이 이루어진다.

#### [Data Locality](https://gameprogrammingpatterns.com/data-locality.html)
    데이터를 메모리에 효율적으로 저장하도록 최적화하면, 성능의 이점을 얻을 수 있다. 
    클래스를 구조체로 대체하면 데이터를 더 캐시 친화적으로 만들 수 있다. 
    Unity의 ECS 및 DOTS 아키텍처는 이 패턴을 구현한다.

#### [Spatial Partition](https://gameprogrammingpatterns.com/spatial-partition.html)
    큰 씬과 게임 월드에서는 특별한 구조를 사용하여 GameObjects를 위치에 따라 조직한다. 
    그리드, 트라이(쿼드트리, 옥트리), 이진 탐색 트리는 모두 보다 효율적으로 나누고 
    검색을 돕는 기술이다.

#### [Decorator](https://en.wikipedia.org/wiki/Decorator_pattern)
    기존 구조를 변경하지 않고 객체에 책임을 추가할 수 있다. 
    데코레이터는 특별한 능력을 부여하거나 GameObject를 수정할 수 있다. 
    예를 들어, 기본 무기 클래스를 변경할 필요 없이 무기에 특전을 추가한다.

#### [Facade](https://en.wikipedia.org/wiki/Facade_pattern)
    복잡한 시스템에 간단하고 통합된 인터페이스를 제공한다. 
    AI, 애니메이션, 사운드 컴포넌트가 분리된 GameObject가 있다면, 
    그 컴포넌트 주변에 래퍼 클래스를 추가할 수 있다 
    (플레이어 컨트롤러 클래스가 PlayerInput, PlayerAudio 등을 관리하는 것을 상상해보라). 
    이 파사드는 원래 컴포넌트의 세부 사항을 숨기고 사용을 단순화한다.

#### [Template Method](https://en.wikipedia.org/wiki/Template_method_pattern)
    알고리즘의 정확한 단계를 서브클래스로 연기하는 패턴이다. 
    예를 들어, 추상 클래스에서 알고리즘 또는 데이터 구조의 대략적인 골격을 정의할 수 있지만, 
    서브클래스가 알고리즘의 전체 구조를 변경하지 않고 특정 부분을 오버라이드할 수 있도록 
    허용한다.

#### [Stragety](https://en.wikipedia.org/wiki/Strategy_pattern)
    알고리즘군을 정의하고 각각을 클래스 내부에 캡슐화하여, 실행 시간에 
    각 알고리즘(전략)을 상호 교체할 수 있게 도와주는 행동 패턴 (정책 패턴) 이다. 
    예를 들어, 경로 찾기 시스템을 만들었다면, 전략 패턴을 사용하여 
    여러 알고리즘(A*, 다익스트라 최단 경로 등)을 정의하고 게임플레이 중에 
    컨텍스트에 따라 교체할 수 있다.

#### [Composite](https://en.wikipedia.org/wiki/Composite_pattern)
    객체를 트리 구조로 조직하고, 결과적인 구조를 개별 객체처럼 처리할 수 있게 해주는 
    구조적 디자인 패턴이다. 단순하고 복합 요소 (잎사귀와 컨테이너) 로부터 트리를 구성한다. 
    모든 요소가 동일한 인터페이스를 구현하여 전체 트리에 대해 동일한 행동을 
    재귀적으로 실행할 수 있다.


## 참고한 자료
[Unity_E-Book_LevelUpYourCodeWithGameProgrammingPatterns](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)