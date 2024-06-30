---
title:  State Pattern
date:   2024-03-24 +1000
categories: [디자인 패턴]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.

플레이어블 캐릭터를 구성한다고 상상해보자. 어느 순간, 캐릭터는 땅에 서 있을 것이다. 컨트롤러를 움직이면, 그것이 달리거나 걷는 것처럼 보인다. 점프 버튼을 누르면 캐릭터는 공중으로 뛰어오른다. 몇 프레임 뒤에, 그것은 착지하여 다시 자신의 대기 중인, 서 있는 자세로 돌아간다.

## States and State Machines

게임은 상호작용적이며, 우리로 하여금 실행 시간에 변경되는 많은 시스템을 추적하게 한다. 
캐릭터의 다양한 상태를 나타내는 다이어그램을 그린다면, 다음과 같이 그릴 수 있을 것이다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/75a261b4-a5cf-4a30-9a09-7a4184416a41)

몇 가지 차이점이 있긴하지만 플로우차트와 유사하다:

- 다이어그램은 여러 상태(대기/서기, 걷기, 달리기, 점프하기 등)로 구성되며, 주어진 시간에는 오직 하나의 현재 상태만 활성화된다. 
- 각 상태는 실행 시간에 조건을 기반으로 다른 하나의 상태로의 전환을 촉발할 수 있다. 
- 전환 발생 시, 출력 상태가 새로운 활성 상태가 된다.

이 다이어그램은 [유한 상태 기계(FSM; Finite state machine)](https://en.wikipedia.org/wiki/Finite-state_machine)라고 불리는 것을 설명한다. 
게임 개발에서 하나의 전형적인 사용 사례는 게임 액터나 소품의 내부 상태를 추적하는 것이다.

기본적인 FSM을 코드로 설명하기 위해, `enum`과 `switch` 문을 사용하는 접근 방식을 사용할 수 있다.

```cs
public enum PlayerControllerState 
{ 
    Idle, 
    Walk, 
    Jump 
} 
public class UnrefactoredPlayerController : MonoBehaviour
{
    private PlayerControllerState state; 
    private void Update() 
    { 
        GetInput(); 
        switch (state)
        {
            case PlayerControllerState.Idle:
                Idle(); 
                break; 
            case PlayerControllerState.Walk:
                Walk(); 
                break; 
            case PlayerControllerState.Jump: 
                Jump(); 
                break; 
        } 
    } 
    
    private void GetInput()
    { 
        // process walk and jump controls 
    }
    private void Walk()
    {
        // walk logic
    }
    private void Idle()
    {
        // idle logic
    } 
    private void Jump() 
    { 
        // jump logic 
    }
}
```

이 방법은 작동하지만, `PlayerController` 스크립트는 금방 지저분해질 수 있다. 
더 많은 상태와 복잡성을 추가할 때, 우리는 매번 `PlayerController` 스크립트의 내부를 다시 검토해야 한다.

## 예제 : 단순한 상태 패턴

다행히도, ***State pattern*** 은 로직을 재구성하는 데 도움을 줄 수 있다. <br>
원래의 Gang of Four에 따르면, ***State pattern***은 두 가지 문제를 해결한다:

- 객체는 내부 상태가 변경될 때 그 행동을 변형시켜야 한다.
- 상태별 행동은 독립적으로 정의되며, 
  새로운 상태를 추가해도 기존 상태들의 행동에 영향을 주지 않는다.

위의 예제인 `UnrefactoredPlayerController` 클래스는 상태 변경을 추적할 수 있지만, 두 번째 문제를 만족시키지 못한다. 새로운 상태를 추가할 때 기존 상태들에 미치는 영향을 최소화하고 싶다면 대신, 상태를 객체로 캡슐화할 수 있다.

각 상태를 다음과 같이 구조화하는 것을 상상해보자:

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/15bdf141-2a81-417c-9363-53d858f4f001)

여기서는 상태에 진입하여, 제어 흐름이 종료되게 하는 조건이 발생할 때까지 매 프레임을 반복한다. 이 패턴을 구현하기 위해, `IState`라는 인터페이스를 생성한다:

```cs
public interface IState 
{ 
    public void Enter()
    {
        // code that runs when we first enter the state
    } 
    
    public void Update()
    { 
        // per-frame logic, include condition to transition to a new state
    } 
    
    public void Exit() 
    { 
        // code that runs when we exit the state 
    } 
}
```

게임의 각 구체적인 상태는 `IState` 인터페이스를 구현할 것이다:

 - `Enter` : 이 로직은 상태에 처음 진입할 때 실행된다.
 - `Update` : 이 로직은 매 프레임마다 실행된다(Execute 또는 Tick이라고도 함). 
   MonoBehaviour가 하는 것처럼 Update 메소드를 더 세분화할 수 있으며, 
   물리 연산을 위한 FixedUpdate, LateUpdate 등을 사용할 수 있다.
   
   업데이트 내의 모든 기능은 상태 변경을 유발하는 조건이 감지될 때까지 매 프레임마다 실행된다.

-  `Exit` : 이 코드는 상태를 떠나고 새로운 상태로 전환하기 전에 실행된다.

`IState`를 구현하는 각 상태에 대한 클래스를 생성해야 한다. 
샘플 프로젝트에서는 `WalkState`, `IdleState`, `JumpState`에 대해 별도의 클래스가 설정되어 있다.

다른 클래스인 `StateMachine`은 제어 흐름이 상태에 어떻게 진입하고 퇴장하는지를 관리할 것이다. 세 가지 예제 상태를 가진다면, `StateMachine`은 다음과 같이 보일 수 있다.

```cs
[Serializable]
public class StateMachine
{
    public IState CurrentState { get; private set; } 
    
    public WalkState walkState; 
    public JumpState jumpState; 
    public IdleState idleState; 
    
    public void Initialize(IState startingState)
    {
        CurrentState = startingState; 
        startingState.Enter(); 
    } 
    
    public void TransitionTo(IState nextState) 
    { 
        CurrentState.Exit(); 
        CurrentState = nextState; 
        nextState.Enter(); 
    } 
    
    public void Update() 
    { 
        if (CurrentState != null) 
        { 
            CurrentState.Update();
        } 
    } 
}
```

패턴을 따르기 위해, `StateMachine`은 관리하에 있는 각 상태에 대한 public 객체를 참조한다 (이 경우, `walkState`, `jumpState`, `idleState`). `StateMachine`이 MonoBehaviour를 상속받지 않기 때문에, 생성자를 사용하여 각 인스턴스를 설정한다:

```cs
public StateMachine(PlayerController player) 
{ 
    this.walkState = new WalkState(player); 
    this.jumpState = new JumpState(player); 
    this.idleState = new IdleState(player); 
}
```

생성자에 필요한 모든 매개변수를 전달할 수 있다. 샘플 프로젝트에서는 각 상태에 `PlayerController`가 참조된다. 그런 다음 그것을 사용하여 매 프레임마다 각 상태를 업데이트한다(아래의 `IdleState` 예제 참조).

`StateMachine`에 대해 다음 사항을 참고한다:

-  Serializable Attribute를 사용하면 `StateMachine`(및 그 public 필드)을 인스펙터에 표시할 수 있다. 그런 다음 다른 MonoBehaviour(예: PlayerController 또는 EnemyController)가 `StateMachine`을 필드로 사용할 수 있다.

-  CurrentState 프로퍼티는 읽기 전용이다. StateMachine 자체가 이 필드를 명시적으로 설정하지 않는다. 외부 객체(예: PlayerController)가 `Initialize` 메소드를 호출하여 기본 상태를 설정할 수 있다.

- 각 상태 객체는 현재 활성 상태를 변경하기 위해 `TransitionTo` 메소드를 호출하는 자체 조건을 결정한다. StateMachine 인스턴스를 설정할 때 각 상태에 필요한 모든 의존성(StateMachine 자체 포함)을 전달할 수 있다.

예제 프로젝트에서, PlayerController는 이미 StateMachine에 대한 참조를 포함하고 있으므로, player 매개변수 하나만 전달한다.

각 상태 객체는 자신의 내부 로직을 관리하며, GameObject 또는 컴포넌트를 설명하는 데 필요한 만큼 많은 상태를 만들 수 있다. 각각은 `IState`를 구현하는 자신만의 클래스를 갖는다. SOLID 원칙을 준수하여, 더 많은 상태를 추가해도 이전에 생성된 상태에 미치는 영향이 최소화된다.

다음은 `IdleState`의 예시다.

```cs
public class IdleState : IState
{
    private PlayerController player; 
    
    public IdleState(PlayerController player) 
    { 
        this.player = player; 
    } 
    
    public void Enter() 
    { 
        // code that runs when we first enter the state 
    } 
    
    public void Update()
    { 
        // Here we add logic to detect if the conditions exist to 
        // transition to another state 
        ... 
    } 
    public void Exit() 
    { 
        // code that runs when we exit the state 
    } 
}
```

다시 말하지만, 생성자를 사용하여 `PlayerController` 객체를 전달한다. 예시에서, 이 `player`는 `StateMachine`과 `Update` 로직에 필요한 모든 것을 참조한다. `idleState`는 캐릭터 컨트롤러의 속도 또는 점프 상태를 모니터링한 다음, 적절하게 `StateMachine`의 `TransitionTo` 메소드를 호출한다.

`WalkState`와 `JumpState` 구현도 샘플 프로젝트에서 확인해보자. 하나의 큰 클래스가 행동을 전환하는 대신, 각 상태는 자신만의 업데이트 로직을 가진다. 이 방법으로, 상태들은 서로 독립적으로 기능할 수 있다.

## 장단점

상태 패턴은 객체의 내부 로직을 설정할 때 SOLID 원칙을 준수하는 데 도움을 줄 수 있다. 각 상태는 비교적 크기가 작고, 다른 상태로 전환하기 위한 조건만을 추적한다. ***개방 폐쇄 (Open-Closed) 원칙***을 준수하여, 새로운 상태를 기존 것들에 영향을 주지 않고 추가할 수 있으며, 번거로운 switch 또는 if 문장을 피할 수 있다.

한편, 추적할 상태가 몇 개뿐이라면, 추가적인 구조는 과도할 수 있다. 상태가 특정 복잡성에 이를 것으로 예상될 때만 이 패턴이 의미가 있을 수 있다.

## 개선사항

샘플 프로젝트에서 캡슐은 색상이 변경되고, UI는 플레이어의 내부 상태에 따라 업데이트된다. 
실제 예에서는 상태 변경을 동반하는 훨씬 더 복잡한 효과를 가질 수 있다.

#### State Pattern을 애니메이션과 결합하라 
    상태 패턴의 일반적인 용도는 애니메이션이다. 
    플레이어나 적 캐릭터는 종종 매크로 수준에서 기본 형태(캡슐 등)로 표현된다. 
    그러면 내부 상태 변경에 반응하여 애니메이션된 지오메트리를 가질 수 있어 
    게임 액터가 달리기, 점프하기, 수영하기, 등반하기 등을 하는 것처럼 보일 수 있다.

Unity의 Animator 창을 사용해 본 적이 있다면, 그것의 작업 흐름이 상태 패턴과 잘 어울린다는 것을 알게 될 것이다. 각 애니메이션 클립이 하나의 상태를 차지하며, 한 번에 하나의 상태만 활성화된다.

#### 이벤트 추가하기
    외부 객체에 상태 변경을 알리기 위해서 이벤트를 추가할 수 있다(Observer Pattern 참조). 
    상태 진입이나 퇴장 시 이벤트가 발생하면 관련 리스너에게 알림을 주어 
    실행 시간에 반응하도록 할 수 있다.

#### 계층 추가하기
    State Pattern으로 더 복잡한 엔티티를 설명하기 시작할 때, 
    계층적 상태 기계를 구현하고 싶을 수 있다. 
    불가피하게 일부 상태는 비슷할 것이다. 
    예를 들어, 플레이어나 게임 액터가 지면에 있을 때, 
    WalkingState나 RunningState에 있든 간에 웅크리거나 점프할 수 있다.

`SuperState`를 구현하면, 공통 행동을 함께 유지할 수 있다. 그런 다음 상속을 사용하여 하위 상태에서 특정 사항을 오버라이드할 수 있다. 예를 들어, 먼저 `GroundedState`를 선언할 수 있다. 그 다음에는 그것으로부터 `RunningState`나 `WalkingState`를 상속받을 수 있다.

#### 간단한 AI 구현하기
    유한 상태 기계는 기본적인 적 AI 생성에도 유용할 수 있다. 
    NPC 두뇌를 구축하기 위한 FSM 접근 방식은 다음과 같을 수 있다:

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/7edb2c71-c510-40fc-8e1f-984b88fdf14c)

여기 ***State Pattern***이 완전히 다른 맥락에서 다시 작동하는 모습이다. 모든 상태는 공격, 도주, 순찰과 같은 행동을 대표한다. 한 번에 하나의 상태만 활성화되며, 각 상태가 다음 상태로의 전환을 결정한다.

## 참고한 자료
[유니티 E-Book](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)