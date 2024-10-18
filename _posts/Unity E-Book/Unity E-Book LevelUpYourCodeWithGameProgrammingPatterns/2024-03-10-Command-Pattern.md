---
title:  Command Pattern
date:   2024-03-10 +1000
categories: [Unity E-book]
tags: [디자인 패턴, Unity, C#, Unity E-book, LevelUpYourCodeWithGameProgrammingPatterns]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.

Gang of Four 패턴 중 하나인 ***커맨드 패턴 (Command Pattern)*** 은 특정한 일련의 행동을 추적하고자 할 때 유용하다. 만약 실행 취소/재실행 기능을 사용하거나 입력 이력을 리스트에 보관하는 게임을 플레이해 본 적이 있다면, ***커맨드 패턴 (Command Pattern)*** 을 이미 경험한 것이다. 사용자가 실제로 실행하기 전에 여러 턴을 계획할 수 있는 전략 게임을 상상해 보세요. 그것이 바로 ***커맨드 패턴 (Command Pattern)*** 이다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/b4f9f438-760f-4996-a80e-4ebf399b0822)

이러한 커맨드 객체들을 큐나 스택 같은 컬렉션에 저장하면, 그 실행 타이밍을 제어할 수 있다. 이것은 작은 버퍼로써 기능한다. 그러면 나중에 재생하거나 실행 취소하기 위해 일련의 행동을 지연시킬 수 있다.

커맨드 패턴을 구현하려면, 행동을 포함할 일반 객체가 필요하다. 이 커맨드 객체는 수행할 로직과 그것을 어떻게 실행 취소할지를 보유한다.

## 커맨드 객체와 커맨드 호출자 <br> The Command Object and Command Invoker

이를 구현하는 방법은 여러 방법이 있겠다. 인터페이스를 사용하는 방법을 보자.

```cs
public interface ICommand
{
    void Execute();
    void Undo();
}
```

이 경우, 모든 게임플레이 동작은 `ICommand` 인터페이스를 적용할 것이다. 
(추상 클래스로 이를 구현할 수도 있다).

각 커맨드 객체는 자신의 `Execute`와 `Undo` 메소드에 대해 책임을 진다. 
따라서, 게임에 더 많은 커맨드를 추가해도 기존의 커맨드에는 영향을 미치지 않는다.

커맨드를 실행하고 실행 취소하는 또 다른 클래스가 필요하다. `CommandInvoker` 클래스를 생성하라. 이는 `ExecuteCommand`와 `UndoCommand` 메소드 외에도, 커맨드 객체의 순서를 보유할 실행 취소 스택을 갖는다.

```cs
public class CommandInvoker 
{ 
    private static Stack<ICommand> undoStack = new Stack<ICommand>(); 
    public static void ExecuteCommand(ICommand command) 
    { 
        command.Execute(); 
        undoStack.Push(command); 
    } 
    
    public static void UndoCommand() 
    { 
        if (undoStack.Count > 0) 
        { 
            ICommand activeCommand = undoStack.Pop(); 
            activeCommand.Undo(); 
        } 
    } 
}
```

## 예제 : 실행취소 가능한 이동

애플리케이션에서 플레이어를 미로를 통해 이동시키고 싶다고 상상해보자. 플레이어의 위치를 변경하는 책임을 가진 `PlayerMover`를 생성할 수 있습니다.

```cs
public class PlayerMover : MonoBehaviour 
{ 
    [SerializeField] private LayerMask obstacleLayer; 
    
    private const float boardSpacing = 1f; 
    public void Move(Vector3 movement) 
    { 
        transform.position = transform.position + movement;
    } 
    public bool IsValidMove(Vector3 movement) 
    { 
        return !Physics.Raycast(transform.position, movement, boardSpacing, obstacleLayer); 
    } 
}
```

`Move` 메소드에 `Vector3`을 전달하여 플레이어를 네 개의 방위(북, 남, 동, 서)로 안내할 수 있다. 
또한, 적절한 `LayerMask`에서 벽을 감지하기 위해 레이캐스트를 사용할 수도 있다. 
물론, 커맨드 패턴에 적용하고 싶은 것을 구현하는 것은 패턴 자체와 별개.

커맨드 패턴을 따르기 위해, `PlayerMover`의 `Move` 메서드를 객체로 포착하라. 
`Move`를 직접 호출하는 대신, `ICommand` 인터페이스를 구현하는 새로운 클래스인 `MoveCommand`를 생성한다. 

```cs
public class MoveCommand : ICommand 
{ 
    PlayerMover playerMover; 
    Vector3 movement; 
    
    public MoveCommand(PlayerMover player, Vector3 moveVector) 
    { 
        this.playerMover = player; 
        this.movement = moveVector; 
    } 
    public void Execute() 
    { 
        playerMover.Move(movement); 
    } 
    public void Undo() 
    { 
        playerMover.Move(-movement); 
    } 
}
```

`ICommand` 인터페이스는 당신이 달성하고자 하는 것을 저장하기 위한 `Execute` 메소드를 요구한다. 여기에는 달성하고자 하는 로직이 들어가므로, 이동 벡터를 사용하여 `Move`를 호출하면 된다.

`ICommand`는 또한 장면을 이전 상태로 복원하기 위한 `Undo` 메소드가 필요하다. 이 경우, `Undo` 로직은 이동 벡터를 빼는 것으로, 본질적으로 플레이어를 반대 방향으로 밀어내는 효과를 낸다.

`MoveCommand`는 실행에 필요한 모든 매개변수를 저장한다. 이를 생성자를 통해 설정한다. 
이 경우, 적절한 `PlayerMover` 컴포넌트와 이동 벡터를 저장한다.

커맨드 객체를 생성하고 필요한 매개변수를 저장한 후, `CommandInvoker`의 static `ExecuteCommand` 및 `UndoCommand` 메소드를 사용하여 `MoveCommand`를 전달한다. 이렇게 하면 `MoveCommand`의 `Execute` 또는 `Undo`가 실행되고, 커맨드 객체가 실행 취소 스택에 추적된다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/14af568b-d869-40bb-8ee7-ced1d68d6745)

`InputManager`는 `PlayerMover`의 `Move` 메소드를 직접 호출하지 않는다. 
대신, 새로운 `MoveCommand`를 생성하고 이를 `CommandInvoker`에 전송하는 추가 메소드인 `RunPlayerCommand`를 추가한다. 

```cs
private void RunPlayerCommand(PlayerMover playerMover, Vector3 movement)
{ 
    if (playerMover == null) { return; } 
    
    if (playerMover.IsValidMove(movement))
    { 
        ICommand command = new MoveCommand(playerMover, movement);
        CommandInvoker.ExecuteCommand(command); 
    } 
}
```

그리고 UI 버튼의 다양한 `onClick` 이벤트를 설정하여 네 가지 이동 벡터를 사용해 `RunPlayerCommand`를 호출하도록 한다.

## 장단점

재생 가능성(replayability) 이나 실행 취소 기능을 구현하는 것은 커맨드 객체의 컬렉션을 생성하는 것만큼 간단하다. 또한, 커맨드 버퍼를 사용하여 특정 컨트롤과 함께 동작을 순서대로 재생할 수도 있다.

예를 들어, 일련의 특정 버튼 클릭이 콤보 움직임이나 공격을 트리거하는 격투 게임을 생각해 보자. 플레이어의 행동을 커맨드 패턴으로 저장하면 이러한 콤보를 설정하는 것이 훨씬 더 간단해진다.

반면에, 커맨드 패턴은 다른 디자인 패턴처럼 더 많은 구조를 도입한다. 이 추가적인 클래스와 인터페이스가 애플리케이션에서 커맨드 객체를 사용하는 데 충분한 이점을 제공하는지 결정해야 할 것이다.

## 개선사항
기본을 익히고 나면, 명령의 타이밍을 조정하고, 컨텍스트에 따라 명령을 연속적으로 또는 역순으로 재생할 수 있다.

커맨드 패턴을 적용할 때 다음 사항을 고려하라

### 더 많은 명령 생성
    샘플 프로젝트에는 `MoveCommand`라는 한 종류의 명령 객체만 포함되어 있다. 
    `ICommand`를 구현하는 어떤 수의 명령 객체도 생성하고 
    `CommandInvoker`를 사용해 추적할 수 있다.

### 재실행 기능 추가는 또 다른 스택을 추가하는 문제
    명령 객체를 실행 취소할 때, 재실행 작업을 추적하는 별도의 스택에 푸시하라. 
    이 방법을 통해 실행 취소 기록을 빠르게 순환하거나 그 행동들을 재실행할 수 있다. 
    사용자가 완전히 새로운 움직임을 호출할 때 재실행 스택을 비우자
    (해당 구현은 동반된 샘플 프로젝트에서 찾을 수 있다).

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/f581a252-ee7c-4b2f-8233-c599b4510e00)

### 명령 객체의 버퍼를 위해 다른 컬렉션을 사용하라
    먼저 들어온 것이 먼저 나가는(FIFO) 행동을 원한다면 큐가 더 편리할 수 있다.
    리스트를 사용한다면 현재 활성 인덱스(Curent Active Index)를 추적하자.
    활성 인덱스 이전의 명령은 실행 취소가 가능하고 인덱스 이후의 명령은 재실행이 가능하다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/768a6cdb-aec8-4108-af27-6c98c2a2c440)

### 스택의 크기 제한
    실행 취소와 재실행 작업은 빠르게 제어할 수 없을 정도로 커질 수 있다. 
    스택을 마지막 명령의 수로 제한하라.

### 필요한 모든 매개변수를 생성자로 전달하라
    이는 `MoveCommand` 예제에서 볼 수 있듯이 로직을 캡슐화하는 데 도움이 된다.

`CommandInvoker`와 같은 외부 객체는 명령 객체의 내부 작동을 보지 않고, 단지 `Execute`나 `Undo`를 호출한다. 생성자를 호출할 때 명령 객체가 작동하는 데 필요한 모든 데이터를 제공하라.

## 참고한 자료
[Unity_E-Book_LevelUpYourCodeWithGameProgrammingPatterns](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)