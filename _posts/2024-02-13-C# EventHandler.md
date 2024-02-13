---
title:  Unity C# EventHandler
date:   2024-02-13 +0900
categories: [C# 문법, Unity]
tags: [C# 문법, Unity]
math: true
mermaid: true
---

## EventHandler

- `Delegate`의 일종
- 미리 선언된 `Delegate`, 이벤트 용도

## 정의

```csharp
namespace System
{
		public delegate void EventHandler(object sender, EventArgs e);

		// 혹은 
		public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);
		// 사용 예제2에서 사용
}
```

- 미리 선언 되어있기 때문에, 함수를 위 Signature와 동일하게 작성해야 한다.
- `object sender` : 이벤트를 실행시키는 객체 (보통 this)
- `EventArgs e` : 이벤트 실행 시, 전달하고자 하는 정보가 있을 때 사용하는 매개변수

## **EventHandler와 EventArgs 확장**

- **사용자 정의 EventArgs**: **`EventArgs`** 클래스는 이벤트와 관련된 데이터를 전달하는 데 사용됩니다. **`EventArgs`**를 상속받아 사용자 정의 데이터를 포함하는 클래스를 만들 수 있으며, 이를 통해 이벤트와 관련된 추가 정보를 전달할 수 있습니다.

### 사용 예제1

```csharp
using System;
using UnityEngine;

// EventArgs를 상속받는 class를 정의.
// 이 class 내에는 이벤트 시 보내고자 하는 정보를 정의한다.
public class EventTest1 : EventArgs
{
    public string _name;

    public EventTest1(string name)
    {
        _name = name;
    }
}

public class NewBehaviourScript : MonoBehaviour
{
    public event EventHandler eventHandler;
    
    void Start()
    {
        EventTest1 eventTest = new EventTest1("this is test");
				// 보내고자 하는 정보를 담아서 

        eventHandler += Test;
        eventHandler.Invoke(this, eventTest);    // 보낸다
    }

    void Test(object o, EventArgs e)
    {
        Debug.Log("Test");
        Debug.Log(((EventTest1)e)._name);    // 받을 때는 타입 캐스팅 필수
    }
    
}
```

```csharp
namespace System
{
		public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);
}
```

- **제네릭 EventHandler**: **`EventHandler<TEventArgs>`**는 제네릭 타입을 사용하여 이벤트 데이터를 보다 유연하게 관리할 수 있게 한다. 제네릭 버전을 사용하면 이벤트 핸들러에서 특정 타입의 **`EventArgs`**를 직접 처리할 수 있으며, 타입 캐스팅이 필요 없음.

### 사용 예제2

```csharp
using System;
using UnityEngine;

public class EventTest1    // 제네릭을 사용할 때는 상속 X
{
    public string _name;

    public EventTest1(string name)
    {
        _name = name;
    }
}

public class NewBehaviourScript : MonoBehaviour
{
    public event EventHandler<EventTest1> eventHandler;    // 이번에는 제네릭을 이용
    
    void Start()
    {
        EventTest1 eventTest = new EventTest1("this is test");
				// 보내고자 하는 정보를 담아서 

        eventHandler += Test;
        eventHandler.Invoke(this, eventTest);    // 보낸다
    }

    void Test(object o, EventTest1 e)    // Signature도 기존과 다르게 변한다.
    {
        Debug.Log("Test");
        Debug.Log(e._name);    // 제네릭을 이용하면 타입 캐스팅 X
    }
    
}
```

### **이벤트 핸들러의 추가적인 사항**

- **이벤트 구독 및 구독 취소 시 로직**: 이벤트의 **`add`** 및 **`remove`** 접근자를 사용하여 이벤트에 구독하거나 구독을 취소할 때 추가 로직을 구현할 수 있음. 이를 통해 이벤트 구독이나 구독 취소에 대한 추가적인 처리를 할 수 있음.
- **이벤트의 null 확인**: 이벤트를 호출하기 전에 이벤트가 null인지 확인하여, 이벤트에 구독자가 없는 경우에 발생할 수 있는 예외를 방지. `_eventHandler?.Invoke(this, EventArgs.Empty);`

### 사용 예제3

```csharp
using System;
using UnityEngine;

public class EventTest2 : EventArgs
{
    private EventHandler _eventHandler;
    public event EventHandler EventHandler
    {
        add    // 해당 이벤트 구독 시 실행
        {
            Debug.Log("ADD");
            _eventHandler += value;
        }
        remove    // 해당 이벤트 구독해지 시 실행
        {
            Debug.Log("REMOVE");
            _eventHandler -= value;
        }
    }

    public void StartEvent()
    {
        _eventHandler.Invoke(this, EventArgs.Empty);
    }
}

public class NewBehaviourScript : MonoBehaviour
{
    void Start()
    {
        EventTest2 eventTest = new EventTest2();

        eventTest.EventHandler += Test;
        eventTest.StartEvent();
    }

    void Test(object o, EventArgs e)
    {
        Debug.Log("Test");
    }
    
}
```