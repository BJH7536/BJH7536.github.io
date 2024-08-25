---
title: Observer Pattern 옵저버 패턴
date: 2024-08-23 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Observer Pattern

### 정의

한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들에게 연락이 가고 자동으로 내용이 갱신되는 방식으로, 일대다 (**one-to-many**) 의존성을 정의한다.

**Subject Object**의 상태 (변수, …)가 변하면 **Observer**들에게 이를 알린다. **Observer**는 상태 변경에 따른 필요한 동작을 수행한다.

***Observer Pattern***은 **Subject Object**가 느슨하게 결합되어 있는 객체 디자인을 제공한다.

- **Subject**가 **Object**에 대해서 아는 것은 **Observer**가 **특정 인터페이스 (Observer Interface)** 를 구현한다는 것뿐.
- **Observer**는 언제든지 새로 추가할 수 있다.
    
    (**Subject**는 **Observer 인터페이스**를 구현하는 객체 목록에만 의존하기 때문)
    
- 새로운 형식의 **Observer**를 추가하려해도 **Subject**를 전혀 변경할 필요가 없다.
    
    (새로운 클래스에서 **Observer** 인터페이스만 구현해주면 된다)
    
- **Subject**나 **Object**가 바뀌더라도 서로에게 전혀 영향을 주지 않는다. 그래서 **Subject**와 **Observer**는 서로 독립적으로 재사용할 수 있다.

### UML Diagram

![image (3)](https://github.com/user-attachments/assets/fa129574-6bf4-4d8d-8510-0ad42b84d7eb)

### 예제 1

```csharp
// 옵저버 추상 클래스
// : 옵저버들이 구현해야 할 인터페이스 메서드
public abstract class Observer
{
    // 상태 update 메서드
    public abstract void OnNotify();
}
```

```csharp
using UnityEngine;

// 옵저버 구현 클래스
public class ConcreteObserver1 : Observer
{
    // 대상 타입의 클래스에서 이 메서드를 실행시킨다
    public override void OnNotify()
    {
        Debug.Log("옵저버 클래스의 메서드 실행 #1");
    }
}

```

```csharp
using UnityEngine;

// 옵저버 구현 클래스
public class ConcreteObserver2 : Observer
{
    // 대상 타입의 클래스에서 이 메서드를 실행시킨다
    public override void OnNotify()
    {
        Debug.Log("옵저버 클래스의 메서드 실행 #2");
    }
}
```

```csharp
// 대상 인터페이스
// : 옵저버 관리, 활용에 관한 타입 정의
public interface ISubject
{
    void AddObserver(Observer o);
    void RemoveObserver(Observer o);
    void Notify();
}
```

```csharp
using System.Collections.Generic;
using UnityEngine;

// 대상 클래스
// : 대상 인터페이스를 구현한 클래스
public class ConcreteSubject : MonoBehaviour, ISubject
{
    private List<Observer> observers = new List<Observer>();        // 옵저버를 관리하는 List
    
    // 관리할 옵저버를 등록
    public void AddObserver(Observer observer)
    {
        observers.Add(observer);
    }

    // 관리 중인 옵저버를 삭제
    public void RemoveObserver(Observer observer)
    {
        if (observers.IndexOf(observer) > 0) observers.Remove(observer);
    }

    // 관리 중인 옵저버에게 연락
    public void Notify()
    {
        foreach (Observer observer in observers)
        {
            observer.OnNotify();
        }
    }

    private void Start()
    {
        Observer obj1 = new ConcreteObserver1();
        Observer obj2 = new ConcreteObserver2();
        
        AddObserver(obj1);
        AddObserver(obj2);
    }
}

```

![image (4)](https://github.com/user-attachments/assets/a57abc7d-1de8-49dd-b449-1454ddbab4b4)

ConcreteSubject의 Notify 함수를 실행한  결과

### 예제 2

```csharp
// 대상 인터페이스
// : 옵저버 관리, 활용에 관한 타입 정의
public interface ISubject
{
    void AddObserver(Observer o);
    void RemoveObserver(Observer o);
    void Notify();
}
```

```csharp
// 옵저버 추상 클래스
// : 옵저버들이 구현해야 할 인터페이스 메서드
public abstract class Observer
{
    // 상태 update 메서드
    public abstract void OnNotify(int num);
}
```

```csharp
using System.Collections.Generic;
using UnityEngine;

// 대상 클래스
// : 대상 인터페이스를 구현한 클래스
public class ConcreteSubject : MonoBehaviour, ISubject
{
    private List<Observer> observers = new List<Observer>();        // 옵저버를 관리하는 List
    private int myNum;
    
    // 관리할 옵저버를 등록
    public void AddObserver(Observer observer)
    {
        observers.Add(observer);
    }

    // 관리 중인 옵저버를 삭제
    public void RemoveObserver(Observer observer)
    {
        if (observers.IndexOf(observer) > 0) observers.Remove(observer);
    }

    // 관리 중인 옵저버에게 연락
    public void Notify()
    {
        foreach (Observer observer in observers)
        {
            observer.OnNotify(myNum);
        }
    }

    private void Start()
    {
        myNum = 10;
        
        Observer obj1 = new ConcreteObserver1(gameObject);
        Observer obj2 = new ConcreteObserver2(gameObject);
        
        AddObserver(obj1);
        AddObserver(obj2);
    }

    public int getNum()
    {
        return myNum;
    }
}

```

```csharp
using UnityEngine;

// 옵저버 구현 클래스
public class ConcreteObserver1 : Observer
{
    private GameObject obj;
    
    // 생성자를 통해 객체 전달
    public ConcreteObserver1(GameObject obj)
    {
        this.obj = obj;
    }
    
    // 대상 타입의 클래스에서 이 메서드를 실행시킨다
    public override void OnNotify(int num)
    {
        int num2 = obj.gameObject.GetComponent<ConcreteSubject>().getNum();
        
        Debug.Log("옵저버 클래스의 메서드 실행 #1");
        Debug.Log("메서드의 파라미터 : " + num);
        Debug.Log("객체 변수를 통한 접근 : " + num2);
    }
}

```

```csharp
using UnityEngine;

// 옵저버 구현 클래스
public class ConcreteObserver2 : Observer
{
    private GameObject obj;
    
    // 생성자를 통해 객체 전달
    public ConcreteObserver2(GameObject obj)
    {
        this.obj = obj;
    }
    
    // 대상 타입의 클래스에서 이 메서드를 실행시킨다
    public override void OnNotify(int num)
    {
        int num2 = obj.gameObject.GetComponent<ConcreteSubject>().getNum();
        
        Debug.Log("옵저버 클래스의 메서드 실행 #2");
        Debug.Log("메서드의 파라미터 : " + num);
        Debug.Log("객체 변수를 통한 접근 : " + num2);
    }
}
```

![image (5)](https://github.com/user-attachments/assets/d8e6d5e9-c35e-4e71-90f6-e412bab7a9fb)

ConcreteSubject의 Notify 함수를 실행한  결과

### 예제 3

```csharp
using UnityEngine;

public abstract class Observer : MonoBehaviour
{
    public abstract void OnNotify(float time);
}
```

```csharp
using UnityEngine;

public class ConcreteObserver : Observer
{
    private float accTime = 0.0f;
    private float limitTime = 0.0f;
    private bool bRotate = false;

    public override void OnNotify(float time)
    {
        accTime = 0.0f;
        limitTime = time;
        bRotate = true;
    }

    private void Update()
    {
        if (accTime > limitTime)
        {
            bRotate = false;
        }

        if (bRotate)
        {
            accTime += Time.deltaTime;
            
            transform.Rotate(100.0f * Time.deltaTime * Vector3.up);
        }
    }
}

```

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ConcreteSubject : MonoBehaviour
{
    public GameObject sphere;
    public GameObject box1;
    public GameObject box2;
    public GameObject box3;

    private List<Observer> observers = new List<Observer>();        // 옵저버를 관리하는 List
    private float time;

    private void Start()
    {
        Observer obj1 = box1.GetComponent<ConcreteObserver>();
        Observer obj2 = box2.GetComponent<ConcreteObserver>();
        Observer obj3 = box3.GetComponent<ConcreteObserver>();

        observers.Add(obj1);
        observers.Add(obj2);
        observers.Add(obj3);

        time = 2.0f;
    }

    public void MovePosition()
    {
        // 업적 달성
        sphere.transform.position = new Vector3(2, 0.5f, -3);

        foreach (Observer o in observers)
        {
            o.OnNotify(time);
        }

        StartCoroutine(ResetPosition(time));
    }

    IEnumerator ResetPosition(float time)
    {
        yield return new WaitForSeconds(time);
        sphere.transform.position = new Vector3(0, 0.5f, -3);
    }
}
```

![2024-08-22002212-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/cc644053-be1a-4424-96d4-c38f7816d1d5)

버튼에 연결된 `MovePosition()` 함수가 버튼이 클릭됨에 따라 호출되고, 
이에 따라 GameManager를 Subject로 하는 Observer인 Blue Cube들이 2초간 회전하게 된다.

### 예제 4

```csharp
// 옵저버 추상 클래스
// : 옵저버들이 구현해야 할 인터페이스 메서드
public abstract class Observer
{
    // 상태 update 메서드
    public abstract void OnNotify(int num);
}
```

```csharp
// 대상 클래스

using System;
using UnityEngine;

public class ConcreteSubject : MonoBehaviour
{
    private int myNum;

    delegate void NotiHandler(int num);
    private NotiHandler _notiHandler;

    // 관리 중인 옵저버에게 연락
    public void Notify()
    {
        _notiHandler(myNum);
    }

    private void Start()
    {
        myNum = 10;
        
        Observer obj1 = new ConcreteObserver1(gameObject);
        Observer obj2 = new ConcreteObserver2(gameObject);

        _notiHandler += new NotiHandler(obj1.OnNotify);
        _notiHandler += new NotiHandler(obj2.OnNotify);
    }

    public int getNum()
    {
        return myNum;
    }
}
```

```csharp
using UnityEngine;

// 옵저버 구현 클래스
public class ConcreteObserver1 : Observer
{
    private GameObject obj;
    
    // 생성자를 통해 객체 전달
    public ConcreteObserver1(GameObject obj)
    {
        this.obj = obj;
    }
    
    // 대상 타입의 클래스에서 이 메서드를 실행시킨다
    public override void OnNotify(int num)
    {
        int num2 = obj.gameObject.GetComponent<ConcreteSubject>().getNum();
        
        Debug.Log("옵저버 클래스의 메서드 실행 #1");
        Debug.Log("메서드의 파라미터 : " + num);
        Debug.Log("객체 변수를 통한 접근 : " + num2);
    }
}
```

```csharp
using UnityEngine;

// 옵저버 구현 클래스
public class ConcreteObserver2 : Observer
{
    private GameObject obj;
    
    // 생성자를 통해 객체 전달
    public ConcreteObserver2(GameObject obj)
    {
        this.obj = obj;
    }
    
    // 대상 타입의 클래스에서 이 메서드를 실행시킨다
    public override void OnNotify(int num)
    {
        int num2 = obj.gameObject.GetComponent<ConcreteSubject>().getNum();
        
        Debug.Log("옵저버 클래스의 메서드 실행 #2");
        Debug.Log("메서드의 파라미터 : " + num);
        Debug.Log("객체 변수를 통한 접근 : " + num2);
    }
}
```

![image (6)](https://github.com/user-attachments/assets/0c6d50b3-a982-4555-9c57-240a7f241cbb)

`ConcreteSubject` 의 `Notify()` 함수를 호출하여 왼쪽과 같은 결과를 출력한다.

이는 예제2와 동일한 기능을, 내부적으로는 C#의 delegate를 활용해 재구현한 것이다.

delegate의 사용으로 Observer들을 담을 List와, 이를 순회하는 foreach문이 사라지고, 코드가 더욱 간결해짐을 알 수 있다.

이렇게 Observer Pattern은 delegate를 활용해 구현되기도 한다.

### 예제 5

```csharp
using UnityEngine;

public abstract class Observer : MonoBehaviour
{
    public abstract void OnNotify(float time);
}
```

```csharp
using UnityEngine;

public class ConcreteObserver : Observer
{
    private float accTime = 0.0f;
    private float limitTime = 0.0f;
    private bool bRotate = false;

    public override void OnNotify(float time)
    {
        accTime = 0.0f;
        limitTime = time;
        bRotate = true;
    }

    private void Update()
    {
        if (accTime > limitTime)
        {
            bRotate = false;
        }

        if (bRotate)
        {
            accTime += Time.deltaTime;
            
            transform.Rotate(100.0f * Time.deltaTime * Vector3.up);
        }
    }
}
```

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ConcreteSubject : MonoBehaviour
{
    public GameObject sphere;
    public GameObject box1;
    public GameObject box2;
    public GameObject box3;

    delegate void NotiHandler(float rot);
    private NotiHandler _notihandler;
    private float time;

    private void Start()
    {
        Observer obj1 = box1.GetComponent<ConcreteObserver>();
        Observer obj2 = box2.GetComponent<ConcreteObserver>();
        Observer obj3 = box3.GetComponent<ConcreteObserver>();

        _notihandler += new NotiHandler(obj1.OnNotify);
        _notihandler += new NotiHandler(obj2.OnNotify);
        _notihandler += new NotiHandler(obj3.OnNotify);

        time = 2.0f;
    }

    public void MovePosition()
    {
        // 업적 달성
        sphere.transform.position = new Vector3(2, 0.5f, -3);

        _notihandler(time);

        StartCoroutine(ResetPosition(time));
    }

    IEnumerator ResetPosition(float time)
    {
        yield return new WaitForSeconds(time);
        sphere.transform.position = new Vector3(0, 0.5f, -3);
    }
}
```

이는 예제 3과 동일한 기능을, 내부적으로는 C#의 delegate를 활용해 재구현한 것이다.

delegate의 사용으로 Observer들을 담을 List와, 이를 순회하는 foreach문이 사라지고, 코드가 더욱 간결해짐을 알 수 있다.