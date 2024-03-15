---
title:  Unity C# Action, Func, Predicate, UnityAction
date:   2024-02-13 +0900
categories: [C# 문법]
tags: [C# 문법, Unity]
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

## Action<T>

- `Generic Delegate`, 일반화 대리자
- 매개변수를 사용할 수 있는 `Action`
- `System` 내에 `Generic` 타입을 최대 15개까지 받을 수 있도록 `Action`들이 정의.

### Action<T>의 정의

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

## Predicate<T>

- 반환값이 `bool`로 고정된 `Delegate`
- 매개변수가 존재.
- 특정 조건을 충족하는지 여부를 검사하는 데 사용됨.
- 주로 컬렉션의 모든 요소에 대해 조건을 테스트하는데 사용됨.

### Predicate<T>의 정의

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