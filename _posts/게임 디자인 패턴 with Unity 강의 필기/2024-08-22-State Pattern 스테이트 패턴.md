---
title: State Pattern 스테이트 패턴
date: 2024-08-22 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## State Pattern

강의 영상에선 슈퍼 마리오와 같은 플랫포머 게임의 캐릭터 조작을 구현하는 상황을 예시로 들었다.

- `Update()` 안에서 사용자의 입력을 받아, 입력에 맞는 동작을 구현한다. (점프, 엎드리기, 엎드려서 기모으기, 이동 등)
- 단순히 동작만 만들면 끝이아니라, 고려해야 할 것들이 생각보다 많다.
    - 이중 점프를 막는다던지, 점프 중에는 엎드리기를 막아야 한다던지 등등…
- 이를 제한하고 확인하기 위해 수많은 Flag를 사용할 수 있는데, 이렇게 계속 개발하다보면 나중엔 셀 수 없이 많은 Flag 속에서 무엇을 사용해야 할지 감도 안온다.
    - 거기다 “기를 모은다”라는 개념도 구현하려면 코드가 훨씬 복잡해지기 시작한다.

이에 대한 해법으로 **유한 상태 기계 Finite State Machine** 를 사용하는 방법이 있겠다. (FSM에 대한 자세한 개념은 패스, 전공에서 몇번이고 배웠으니)

그리고 이를 코드로 구현한 것이 ***State Pattern***이겠다.

### 정의

> GoF의 정의 : “객체의 내부 상태에 따라 스스로 행동을 변경할 수 있게 허가하는 패턴”
> 

상태를 별도의 클래스로 캡슐화한 다음 현재 상태를 나타내는 객체에게 행동을 위임하기 때문에, 내부 상태가 바뀜에 따라서 행동이 달라지게 된다는 것을 알 수 있다.

행동을 동적으로 교체할 수 있고 ***Strategy Pattern*** 과 구조가 매우 유사하나 쓰임의 용도가 다르다.

### UML Diagram

![image (1)](https://github.com/user-attachments/assets/ff4d78ec-c500-4e6f-b6fb-d897c8bdaa78)

UML 다이어그램으로 표현하 위와 같다. 

상태를 나타내는 Abstract 클래스와, 이를 구현한 Concrete 클래스가 존재한다.

그리고 Context에서는 Concrete 클래스를 다루기 위해서 Abstract 클래스를 활용한다.

### 핵심 정리

- ***State Pattern***을 이용하면 내부 상태를 바탕으로 여러가지 서로 다른 행동을 사용할 수 있다.
- ***State Pattern***을 사용하면 프로시저형 상태 기계를 쓸 때와는 달리 각 상태를 클래스를 이용해 표현하게 된다.
- Context 객체에서는 현재 상태에게 행동을 위임한다.
- 각 상태를 클래스로 캡슐화함으로써 나중에 변경시켜야 하는 내용을 국지화할 수 있다.
- ***State Pattern***과 **Strategy Pattern**의 클래스 다이어그램은 동일하지만, 그 용도는 서로 다르다.
    - **Strategy Pattern**에서는 행동 또는 알고리즘을 Context 클래스를 만들 때 설정한다.
    - ***State Pattern***을 이용하면 Context의 내부 상태가 바뀜에 따라 알아서 행동을 바꿀 수 있도록 한다.
- 상태 전환은 State 클래스에 의해서 제어할수도 있고, Context 클래스에 의해서 제어할수도 있다.
- ***State Pattern***을 이용하면 보돈 디자인에 필요한 클래스의 갯수가 늘어난다.
- State 클래스를 여러 Context 객체의 인스턴스에서 공유하도록 디자인할 수도 있다.

### 예제 1

```csharp
// State 추상 클래스
public abstract class State
{
    // public virtual void Handle() {}
    public abstract void Handle();
}

// State 인터페이스 (인터페이스로 구현할 수도 있다)
// public interface State
// {
//     void Handle();
// }
```

```csharp
// ConcreteState1 클래스
public class ConcreteStateA : State
{
    public override void Handle()
    {
        Debug.Log("Concrete State A");
    }
}

```

```csharp
// ConcreteState2 클래스
public class ConcreteStatB : State
{
    public override void Handle()
    {
        Debug.Log("Concrete State B");
    }
}
```

```csharp
public class Context
{
    private State state;
    
    // Constructor
    public Context(State state)
    {
        this.state = state;
    }
    
    // setter
    public void SetState(State state)
    {
        this.state = state;
    }

    public void Request()
    {
        state.Handle();
    }
} 
```

```csharp
public class StateManger : MonoBehaviour
{
    void Start()
    {
        // Cont 초기 설정
        Context c = new Context(new ConcreteStateA());
        c.Request();
        
        c.SetState(new ConcreteStatB());
        c.Request();
        
        c.SetState(new ConcreteStateA());
        c.Request();
    }
}

```

### 결과

![image (2)](https://github.com/user-attachments/assets/703ba0f7-7a65-4cca-b842-d1b5f5a9aada){: w="250" }

### 예제 2

```csharp
// State 추상 클래스
public abstract class State : MonoBehaviour
{
    public abstract void DoAction(MyState state);
}

// State 인터페이스 (인터페이스로 구현할 수도 있다)
// public interface State
// {
//     void DoAction();
// }
```

```csharp
// ConcreteState 클래스 : 서기
public class ConcreteStateStand : State
{
    public override void DoAction(MyState state)
    {
        Debug.Log("Stand !!");
        StartCoroutine(HandleStand(state));
    }

    IEnumerator HandleStand(MyState state)
    {
        transform.eulerAngles = new Vector3(0, 90, 0);
        transform.position = new Vector3(0, 1.0f, 0);
        yield return null;
    }
}

```

```csharp
// ConcreteState 클래스 : 점프
public class ConcreteStateJump : State
{
    private float gravity = 0.0f;       // 중력의 값
    private Vector3 velocity;           // 캐릭터의 현재 높이 저장
    private const int MAX_CHARGE = 20;
    
    public override void DoAction(MyState state)
    {
        Debug.Log("Jump !!");
        velocity = transform.position;
        StartCoroutine(HandleJump(state));
    }

    IEnumerator HandleJump(MyState state)
    {
        gravity = 0.7f;

        while (true)
        {
            if (state == MyState.STATE_DIVING) break;
            
            velocity.y = velocity.y + gravity;

            transform.position = velocity;

            if (velocity.y < 1.0f) break;

            gravity -= 0.1f;

            yield return new WaitForSeconds(0.05f);
        }

        gravity = 0.0f;
        velocity.y = 1.0f;
        transform.position = velocity;
        GetComponent<MyAction8>().setActionType(MyState.STATE_STANDING);
        // 위 코드 대신 점프 후 서있는 상태 하나를 더 만들수도 있다.
        // state = STATE.STATE_STANDING2
        
        yield return null;
    }
}

```

```csharp
// ConcreteState 클래스 : 엎드리기
public class ConcreteStateDown : State
{
    private int chargeTime = 0;
    private const int MAX_CHARGE = 10;

    public override void DoAction(MyState state)
    {
        Debug.Log("Down !!");
        // 엎드리기
        StartCoroutine(HandleDown(state));
        // 기모으기
        StartCoroutine(HandleSkill());
    }

    IEnumerator HandleDown(MyState state)
    {
        transform.Rotate(Vector3.right * 90.0f);
        transform.position = new Vector3(0, 0.5f, 0);
        yield return null;
    }

    IEnumerator HandleSkill()
    {
        chargeTime = 0;
        while (true)
        {
            chargeTime++;
            if (chargeTime > MAX_CHARGE)
            {
                chargeTime = 0;
                // 스킬 발동
                GetComponent<MyAction8>().setActionType(MyState.STATE_SKILL);
                yield return null;
            }

            yield return new WaitForSeconds(0.1f);
        }
    }
}

```

```csharp
// Concrete 클래스 : 내려찍기
public class ConcreteStateAttack : State
{
    public override void DoAction(MyState state)
    {
        Debug.Log("Down Attack !!!");
        StartCoroutine(HandleAtack(state));
    }

    IEnumerator HandleAtack(MyState state)
    {
        transform.position = new Vector3(0, 0.2f, 0);
        yield return new WaitForSeconds(0.1f);
        
        transform.position = new Vector3(0, 1.2f, 0);
        yield return new WaitForSeconds(0.1f);
        
        transform.position = new Vector3(0, 0.2f, 0);
        yield return new WaitForSeconds(0.1f);

        transform.position = new Vector3(0, 1.0f, 0);
        
        GetComponent<MyAction8>().setActionType(MyState.STATE_STANDING);
        // state = STATE.STATE_STANDING;
    }
}

```

```csharp
using System.Collections;
using UnityEngine;

// ConcreteState 클래스 : 기 모은 후 공격
public class ConcreteStateSkill : State
{
    public override void DoAction(MyState state)
    {
        Debug.Log("Forward Attack !!!");
        StartCoroutine(HandleSkill());
    }

    IEnumerator HandleSkill()
    {
        transform.eulerAngles = new Vector3(0, 90, 0);
        transform.position = new Vector3(0, 1.0f, 0);
        
        transform.Translate(Vector3.forward * 3);
        yield return new WaitForSeconds(0.1f);
        
        transform.Translate(Vector3.forward * 3);
        yield return new WaitForSeconds(0.1f);
        
        transform.Translate(Vector3.forward * 3);
        yield return new WaitForSeconds(0.1f);

        transform.position = new Vector3(0, 1.0f, 0);
        
        GetComponent<MyAction8>().setActionType(MyState.STATE_STANDING);
    }
}

```

```csharp
using System;
using Unity.VisualScripting;
using UnityEngine;

public enum MyState
{
    STATE_STANDING,
    STATE_JUMPING,
    STATE_DUCKING,
    STATE_DIVING,
    STATE_SKILL,
}

public class MyAction8 : MonoBehaviour
{
    private MyState state;
    
    // Concrete 클래스들의 접근점
    private State myState;

    // 상태 클래스 교환
    public void setActionType(MyState state)
    {
        // 현재 상태 저장
        this.state = state;
        
        // 다양한 상태의 대표인 추상 클래스를 가져온다.
        Component c = GetComponent<State>();

        if (c != null) Destroy(c);
        
        switch (state)
        {
            case MyState.STATE_STANDING:
                myState = gameObject.AddComponent<ConcreteStateStand>();
                myState.DoAction(state);
                break;
            case MyState.STATE_JUMPING:
                myState = gameObject.AddComponent<ConcreteStateJump>();
                myState.DoAction(state);
                break;
            case MyState.STATE_DUCKING:
                myState = gameObject.AddComponent<ConcreteStateDown>();
                myState.DoAction(state);
                break;
            case MyState.STATE_DIVING:
                myState = gameObject.AddComponent<ConcreteStateAttack>();
                myState.DoAction(state);
                break;
            case MyState.STATE_SKILL:
                myState = gameObject.AddComponent<ConcreteStateSkill>();
                myState.DoAction(state);
                break;
            default:
                break;
        }
    }

    private void Start()
    {
        setActionType(MyState.STATE_STANDING);
    }

    private void Update()
    {
        switch (state)
        {
            case MyState.STATE_STANDING:
                if (Input.GetKeyDown(KeyCode.Space))
                    setActionType(MyState.STATE_JUMPING);
                else if (Input.GetKeyDown(KeyCode.DownArrow))
                    setActionType(MyState.STATE_DUCKING);
                break;
            case MyState.STATE_JUMPING:
                if (Input.GetKeyDown(KeyCode.DownArrow))
                    setActionType(MyState.STATE_DIVING);
                break;
            case MyState.STATE_DUCKING:
                if (Input.GetKeyDown(KeyCode.DownArrow))
                    setActionType(MyState.STATE_STANDING);
                break;
            default:
                break;
        }
    }
}

```