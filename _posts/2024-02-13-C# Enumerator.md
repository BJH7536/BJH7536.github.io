---
title:  Unity C# Enumerator
date:   2024-02-13 +0900
categories: [C# 문법, Unity]
tags: [C# 문법, Unity]
math: true
mermaid: true
---

## Enumerator
- 데이터 요소를 하나씩 반환하는 기능
- `C#` 에서는 `IEnumerator` 인터페이스를 이용해 구현.
- 어떠한 객체를 대상으로 `foreach`문을 실행하고자 할 때, 다음의 조건을 만족해야 한다.
    `foreach (var item in (대상 객체))`
    1. 루프 대상이 `GetEnumaretor()` 메서드를 구현해야 한다.
    2. `GetEnumerator()` 메서드의 반환 타입이 `bool MoveNext()` 메서드와 `object Current { get; }` 프로퍼티를 가지고 있어야 한다.

### 예제 1
- 오류를 일으키지 않기 위한 최소한의 코드만 작성한 예제.
- 최소한의 형식이라 이해해도 괜찮을 듯 함.

```cs
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    public class MyList
    {
        // hold Somethings ...
        
        public class MyEnumerator
        {
            public object Current { get; }

            public bool MoveNext()
            {
                return true;
            }
        }
        
        public MyEnumerator GetEnumerator()
        {
            return new MyEnumerator();
        }
    }
    
    void Start()
    {
        MyList myList = new MyList();
        foreach (var item in myList)
        {
            // do Something ...
        }
    }
}
```

- 위 코드는 실제로 동작시키면 무한 루프에 걸린다.

### 예제 2
- 다음은 실제로 동작하는 코드

```csharp
using System.Collections.Generic;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    private List<int> list;
    
    public class MyList
    {
        public class MyEnumerator
        {
            private int[] _num = new int[5] { 1, 2, 3, 4, 5 };
            private int _index = -1;
            
            public object Current => _num[_index];        // 현재 요소 반환
            public bool MoveNext()
            // 다음 요소로 이동. 성공하면 true, 실패하면 false를 반환.
            {
                if (_index == _num.Length - 1)
                    return false;
                
                _index++;
                return _index < _num.Length;
            }
        }
        
        public MyEnumerator GetEnumerator()
        {
            return new MyEnumerator();
        }
    }
    
    void Start()
    {
        MyList myList = new MyList();
        foreach (var item in myList)
            Debug.Log(item);
    }
}
```
- 실행시 1, 2, 3, 4, 5가 순서대로 Console에 출력된다.

```csharp
void Start()
{
    MyList myList = new MyList();
    MyList.MyEnumerator e = myList.GetEnumerator();

    e.MoveNext();
    Debug.Log(e.Current);
    e.MoveNext();
    Debug.Log(e.Current);
    e.MoveNext();
    Debug.Log(e.Current);
    e.MoveNext();
    Debug.Log(e.Current);
    e.MoveNext();
    Debug.Log(e.Current);
}
```

```csharp
void Start()
{
    MyList myList = new MyList();
    MyList.MyEnumerator e = myList.GetEnumerator();
    
    while(e.MoveNext())
        Debug.Log(e.Current);
}
```

- 결과적으로 `foreach (var item in myList)`는 내부적으로 `Enumarator`객체의 `MoveNext()`와 `Current` 프로퍼티를 활용함을 알 수 있다.

---
## IEnumerator 인터페이스
- 컬렉션을 순회하는 동안 
	1) 컬렉션의 현재 요소에 접근하고,
	2. 다음 요소로 이동하며,
	3) 순회를 초기화할 수 있는 메커니즘을 제공.

- 다음의 인터페이스 멤버들을 구현해야 한다.
    - `object Current { get; }` 프로퍼티
    - `bool MoveNext()` 메서드
    - `void Reset()` 메서드

## IEnumerable 인터페이스

- 다음의 인터페이스 멤버를 구현해야 한다.
    - `IEnumerator GetEnumerator()` 메서드
- 인터페이스의 필수 멤버를 구현함으로써, **순회 가능하다는 것**을 나타낸다.

위 두 인터페이스는 `Enumerator`를 구현할 때 오류를 줄여주는 인터페이스
- *`Enumerator`로써 필수인 멤버들을 구현하도록 강제하기 때문*

### IEnumerator / IEnumerable 인터페이스를 적용한 예제
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    private List<int> list;
    
    public class MyList: IEnumerable
    {
        public class MyEnumerator : IEnumerator 
        {
            private int[] _num = new int[5] { 1, 2, 3, 4, 5 };
            private int _index = -1;

            public void Reset()
            {
                throw new System.NotImplementedException();
            }

            public object Current => _num[_index];        // 현재 요소 반환
            public bool MoveNext()          // 다음 요소로 이동. 성공하면 true, 실패하면 false를 반환.
            {
                if (_index == _num.Length - 1)
                    return false;
                
                _index++;
                return _index < _num.Length;
            }
        }
        
        public IEnumerator GetEnumerator()
        {
            return new MyEnumerator();
        }
    }
    
    void Start()
    {
        MyList myList = new MyList();
        foreach (var item in myList)
            Debug.Log(item);
    }
}
```

## `IEnumerable<T>` 인터페이스

- `Generic` 버전의 `IEnumerable` 인터페이스
- 타입 안정성을 제공.
- `IEnumerator<T>` 타입의 열거자를 반환하는 `GetEnumerator()` 메서드를 포함

`Enumerator` 하나를 활용하기 위해 구현해야 할 것들이 한 두 가지가 아니다.

번거로운 이런 단점을 보완하기 위해 `yield`키워드가 존재함.

---
## yield 키워드

- `C#` 컴파일러는 yield문이 사용된 메서드를 컴파일 단계에서 
	`IEnumerable / IEnumerator` 코드로 치환해서 내부적으로 구현한다.
- 반복자 블록에서 사용되며, 컬렉션을 하나씩 반환하는 데 사용된다.
- `yield return`문을 사용하여 각 요소를 반환하고, 
	`yield break`문을 사용하여 반복자를 종료할 수 있다.

### 위 예제와 동일한 기능을 하는 예제

```csharp
using System.Collections;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    public class MyList
    {
        public IEnumerator GetEnumerator()
        {
            int[] _num = new int[5] { 1, 2, 3, 4, 5 };
            int _index = -1;

            while (_index < _num.Length - 1)
            {
                _index++;
                yield return _num[_index];
            }
        }
    }

    private void Start()
    {
        MyList list = new MyList();
        foreach (var item in list)
            Debug.Log(item);
    }
}
```

- `yield return`문이 한 함수에서 사용될 때는, 사용된 그 위치에 자취를 남기고 return, 반환된다.
    - 다시 그 함수가 실행될 때는 이전에 남겼던 자취를 찾아, 그 위치부터 실행한다.

## yield break;

- `yield return`문에서 다음에 이어서 실행할 기준인 자취를 남길 수 있었다면, 
	  `yield break`는 자취를 모두 지우는 키워드.

### 예제

```csharp
using System.Collections;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{
    public class MyList
    {
        public IEnumerator GetEnumerator()
        {
            int[] _num = new int[5] { 1, 2, 3, 4, 5 };
            int _index = -1;

            while (_index < _num.Length - 1)
            {
                _index++;
                if(_index == 4)
                    yield break;
                yield return _num[_index];
            }
        }
    }

    private void Start()
    {
        MyList list = new MyList();
        foreach (var item in list)
            Debug.Log(item);
    }
}
```