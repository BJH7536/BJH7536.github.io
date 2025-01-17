---
title:  Unity C# Action, Func, Predicate, UnityAction
date:   2024-02-13 +0900
categories: [C#]
tags: [C#, Unity]
math: true
mermaid: true
---

## Action

- `using System` 필요
- 반환 타입이 `void`인 메소드를 참조하기 위해 사용되는 `Delegate`

### Action의 정의

```csharp
namespace System
{ 
    public delegate void Action(); 
}
```

### 오류가 발생하는 코드

```csharp
using System;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    private Action _action;

    private void Start()
    {
        _action += () => Debug.Log("asdf");
        _action += Test1;
        
        _action.Invoke();
    }
    
    void Test1(int num1) {}
}
```

- `_action += Test1;` 에서 오류가 발생.
    - `Action`은 매개변수가 없는 `Delegate`이기 때문에, 매개변수를 받아 넘기는 것이 불가능.

## Action&#60;T&#62;

- `Generic Delegate`, 일반화 대리자
- 매개변수를 사용할 수 있는 `Action`
- `System` 내에 `Generic` 타입을 최대 15개까지 받을 수 있도록 `Action`들이 정의.

### Action&#60;T&#62;의 정의

```csharp
namespace System
{
    public delegate void Action<in T>(T obj);
    public delegate void Action<in T1, in T2>(T1 arg1, T2 arg2, T3 arg3);
    public delegate void Action<in T1, in T2, in T3>(T1 arg1, T2 arg2, T3 arg3);
    ...
    public delegate void Action<in T1, in T2, ..., in T15>(T1 arg1, T2 arg2, ... , T15 arg15);
}
```

### 예제 코드1

```csharp
using System;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    private Action<int> action1;
    private Action<int, int> action2;
    private Action<string, string, string> action3;

    private void Start()
    {
        action1 = Test1;
        action2 = Test2;
        action3 = Test3;
        
        action1.Invoke(3);
        action2.Invoke(3, 4);
        action3.Invoke("abc", "def", "ghi");
    }
    
    void Test1(int num1) { Debug.Log(num1); }
    void Test2(int num1, int num2) {Debug.Log(num1 + num2);}
    void Test3(string str1, string str2, string str3) {Debug.Log(str1 + str2+ str3);}
    
}
```

---

## Func<T, TResult>

- `Generic Delegate`, 일반화 대리자
- `Generic` 매개변수가 없고, 반환타입만 있는 경우 `Func<Result>`
- `Generic` 매개변수 한개와, 반환타입이 있는 경우 `Func<T, Result>`
- `Generic` 매개변수는 최대 16개까지 사용 가능.
- `Func Delegate Chain`에서 여러 함수를 연결한 경우
    - 마지막으로 추가한 함수의 반환 값만을 받음.

### Func<T, TResult>의 정의

```csharp
namespace System
{
    public delegate TResult Func<out TResult>();
    public delegate TResult Func<in T, out TResult>(T arg);
    public delegate TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2);
    ...
    public delegate TResult Func<in T1,in T2, ...,in T16,out TResult>(T1 arg1, T2 arg2, ..., T16 arg16);
}
```

### 예제 코드1

```csharp
using System;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    private Func<int> func;

    private void Start()
    {
        func = () =>
        {
            int num2 = 3;
            Debug.Log(num2);
            return num2;
        };

        func += () =>
        {
            Debug.Log(5);
            return 5;
        };

        int num1 = func.Invoke();
        Debug.Log(num1);
    }
}
```

- 위 코드의 실행결과, Console에는 Log로 `3`, `5`, `5`만이 출력된다.
- 이는 `int num1 = func.Invoke();` 로 받아온 `num1`은 `5`이고, 앞서 `func`에 추가한 일회용함수의 `return num2;` 은 사라짐.

### 예제 코드2

```csharp
using System;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    // 제네릭 타입의 갯수가 늘면서, 매개변수가 늘어난다.
    private Func<int> func1;
    private Func<int, int> func2;
    private Func<int, int, int> func3;

    private void Start()
    {
        // 람다식으로 간소화
        func1 = () => 1;
        func2 = (num1) => num1;
        func3 = (num1, num2) => num1 + num2;

        int r1 = func1.Invoke();
        int r2 = func2.Invoke(3);
        int r3 = func3.Invoke(4, 5);
        
        Debug.Log(r1);
        Debug.Log(r2);
        Debug.Log(r3);
    }
}
```

---

## Predicate&#60;T&#62;

- 반환값이 `bool`로 고정된 `Delegate`
- 매개변수가 존재.
- 특정 조건을 충족하는지 여부를 검사하는 데 사용됨.
- 주로 컬렉션의 모든 요소에 대해 조건을 테스트하는데 사용됨.

### Predicate&#60;T&#62;의 정의

```csharp
namespace System
{
  public delegate bool Predicate<in T>(T obj);
}
```

---

## UnityAction

- `Unity Event System`에서 사용되는 `Delegate`
- `using UnityEngine.Events;` 필요
- 매개변수 없음.
- 주로 **`UnityEvent<T>`**와 함께 사용됨.

### UnityAction의 정의

```csharp
namespace UnityEngine.Events
{
  /// <summary>
  ///   <para>Zero argument delegate used by UnityEvents.</para>
  /// </summary>
  public delegate void UnityAction();
}
```

### UnityAction&#60;T&#62;의 정의

```csharp
namespace UnityEngine.Events
{
    public delegate void UnityAction<T0>(T0 arg0);
    public delegate void UnityAction<T0, T1>(T0 arg0, T1 arg1);
    ...
    public delegate void UnityAction<T0, T1, T2, T3>(T0 arg0, T1 arg1, T2 arg2, T3 arg3);
}
```

## 정리


| 구분            | 반환값                 | 매개변수(인수) 범위             | 용도                                                                                                                                                        | 장점                                                                                                                       | 단점                                                                                                                           |
| --------------- | ---------------------- | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Action**      | 없음(`void`)           | 0개 이상 (일반적으로 최대 16개) | 1. 반환값 없이 실행할 메서드(람다 표현식 등)를 담는 델리게이트                                 <br>2. `Action<T1, T2, ...>` 형태로 매개변수를 받음          | 1. 반환값이 필요 없는 콜백을 간단히 처리 가능 <br>2. 람다나 메서드를 일관된 방식으로 사용할 수 있음                        | 1. 반환값이 필요한 경우에는 사용 불가 <br>2. 매개변수 개수가 많아지면 가독성이 떨어질 수 있음                                  |
| **Func**        | 제네릭으로 지정된 타입 | 0개 이상 (일반적으로 최대 16개) | 1. 반환값이 있는 메서드(람다 표현식 등)를 담는 델리게이트                                  <br>2. `Func<T1, T2, ... , TResult>` 형태로 마지막 타입이 반환값 | 1. 반환값을 반환받아 로직에서 유연하게 활용 가능 <br>2. 매개변수와 반환값을 명시적으로 표현하여 코드 가독성 향상           | 1. 반환값이 필요 없는 경우 `Func` 대신 `Action`을 쓰는 것이 더 명확 <br>2. 매개변수 개수가 많아지면 복잡도가 증가              |
| **Predicate**   | `bool` (고정)          | 일반적으로 1개                  | 1. 조건 판별 목적의 델리게이트(주로 필터링, 검색 등) <br>2. 매개변수 하나를 받고 `bool` 결과 반환                                                           | 1. `bool`만을 반환하므로 의도가 명확해 가독성 우수 <br>2. 컬렉션 메서드(`Find`, `Exists` 등)와 함께 사용 시 직관적         | 1. `bool` 외 반환값을 처리할 수 없음 <br>2. 범용도가 `Action`이나 `Func`에 비해 제한적                                         |
| **UnityAction** | 없음(`void`)           | 0개 이상 (일반적으로 최대 16개) | 1. Unity 이벤트 시스템 전용으로 제공되는 델리게이트 <br>2. 이벤트 등록 시 `AddListener`/`RemoveListener` 등에 사용                                          | 1. Unity의 이벤트 시스템(UISystem, uGUI 등)에 최적화 <br>2. 시리얼라이즈(Serialize) 가능해서 에디터에서 이벤트 등록이 편리 | 1. Unity 의존성이 있기 때문에 .NET 일반 프로젝트에서는 별 이점이 없음 <br>2. `Action`에 비해 특정 환경(유니티)에서만 사용 가능 |


- Action: 반환값이 필요 없을 때 사용하는 일반적인 델리게이트.
- Func: 반환값이 필요할 때 사용하는 델리게이트. 마지막 타입이 반드시 반환형을 의미함.
- Predicate: bool 반환에 특화된 델리게이트로, 조건 검사 또는 필터링 용도로 주로 사용.
- UnityAction: Unity 이벤트 시스템 전용 델리게이트로, UnityEvent와 함께 쓰이면 직관적이고 편리함.