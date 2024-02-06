---
title:  유니티 C# 델리게이트 Delegate
date:   2024-02-06 +0900
categories: [C# 문법]
tags: [C# 문법]
math: true
mermaid: true
---

## 델리게이트 Delegate

#### **타입 안전성을 제공하는 함수 포인터**

*C와 C++과 같은 언어에서는 함수 포인터를 사용하여 함수를 변수처럼 취급. <br>
 하지만, 이러한 함수 포인터는 타입 안전성이 부족하다는 단점을 가짐.*

**C#에서 `delegate`는 타입 안전한 함수 포인터의 역할.**

> `delegate`를 사용하면 컴파일 시간에 함수의 시그니처가 확인되어, 
> 런타임 오류의 가능성을 줄일 수 있음.

1. **이벤트 드리븐 프로그래밍 지원** <br>
    `delegate`는 이벤트 기반 프로그래밍을 구현하는 데 필수. 이벤트는 `delegate`를 통해 구독자(subscriber)에게 알림을 보낼 수 있으며, 이를 통해 객체 간의 느슨한 결합(loose coupling)을 실현할 수 있음. 이는 특히 GUI 애플리케이션과 게임 개발에서 중요한 부분임.

2. **콜백 함수 구현 용이** <br>
    비동기 프로그래밍이나 이벤트 핸들링과 같은 상황에서 콜백 함수를 쉽게 구현할 수 있게 해줌. `delegate`를 사용하면 메소드를 인자로 넘겨 다른 메소드가 끝난 후에 실행될 작업을 지정할 수 있음.
    
4. **Delegate Chain 지원** <br>
    C#의 `delegate`는 하나 이상의 메소드를 참조할 수 있는 기능을 제공. 이를 통해 여러 메소드를 동시에 호출할 수 있으며, 이는 이벤트 핸들링이나 콜백 메커니즘 구현에 매우 유용.
		
1. **익명 메소드 및 람다 표현식과의 연동** <br>
    `delegate`는 익명 메소드나 람다 표현식과 연동되어 사용될 수 있음. 이를 통해 코드를 더 간결하고 읽기 쉽게 만들 수 있으며, 특정 상황에 맞는 동작을 빠르게 구현할 수 있음.

### 예제 코드1

```csharp
public class DelegateExample : Monobehavior
{
	class Player
	{
		private delegate void Buffdelegate(); // 타입 정의

		private Buffdelegate _buffdelegate;   // 객체 (변수) 선언

		public enum Buff
		{
			None,
			Buff1,
			Buff2,
		}

		private Buff _buff;
		public Buff _Buff
		{
			get { return _buff; }
			set 
			{
				if(_buff == value)
					return;
				
				_buff = value;
				if(_buff == Buff.Buff1)
					_buffdelegate = Buff1;
				if(_buff == Buff.Buff2)
					_buffdelegate = Buff2;
				if(_buff == Buff.None)
					_buffdelegate = NoneBuff;
			}
		}

		public void Attack()
		{
			_buffdelegate.Invoke();
			Debug.log("Attack");    // 적을 공격하는 코드
		}

		void Buff1() { Debug.log("Buff1"); }  // 버프식 계산 함수 1

		void Buff2() { Debug.log("Buff2"); }  // 버프식 계산 함수 2

		void NoneBuff() {}  // 버프 없을 때

	}

	void Start()
	{
		Player player = new Player();
		player._Buff = Player.Buff.Buff1;
		player.Attack();
	}

}
```

### 예제 코드2

```csharp
public class GameEvents
{
    // 이벤트에 대한 delegate 선언
    public delegate void ActionTriggered(string action);

    // 이벤트 발생 시 호출될 delegate 인스턴스
    public ActionTriggered onActionTriggered;

    // 플레이어의 특정 행동을 시뮬레이션하는 메소드
    public void PlayerAction(string action)
    {
        Console.WriteLine("Player did: " + action);

        // delegate를 통해 이벤트 알림
        if (onActionTriggered != null)
        {
            onActionTriggered(action);
        }
    }
}

public class Program
{
    public static void Main()
    {
        GameEvents gameEvents = new GameEvents();

        // 이벤트 리스너 추가
        gameEvents.onActionTriggered += ActionHandler;

        // 플레이어 행동 시뮬레이션
        gameEvents.PlayerAction("Jump");
        gameEvents.PlayerAction("Run");
    }

    // 이벤트 핸들러
    static void ActionHandler(string action)
    {
        switch (action)
        {
            case "Jump":
                Console.WriteLine("Player jumped!");
                break;
            case "Run":
                Console.WriteLine("Player is running!");
                break;
            default:
                Console.WriteLine("Unknown action performed.");
                break;
        }
    }
}
```