---
title: Adapter Pattern 어댑터 패턴
date: 2024-08-24 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Adapter Pattern

> 한 클래스의 인터페이스를 클라이언트에서 사용하고자 하는 다른 인터페이스로 변환한다.
***Adapter Pattern***을 이용하면 인터페이스 호환성 문제 때문에 같이 쓸 수 없는 클래스들을 연결해서 쓸 수 있다.
> 

‘이미 제공되어 있는 것’과 ‘필요한 것’ 사이의 차이를 없애주는 디자인 패턴

***Adapter Pattern* 은 *Wrapper Pattern* 으로 불리기도 한다.**

일반 상품을 예쁜 포장지로 싸서 선물용 상품으로 만드는 것처럼, 무엇인가를 한번 포장해서 다른 용도로 사용할 수 있게 교환해주는 것이 wrapper이며 adapter이다.

***Adapter Pattern*에는 두가지 종류가 있다.**

- 클래스에 의한 Adapter 패턴 (상속을 사용한 Adapter 패턴)
- 인스턴스에 의한 Adapter 패턴 (위임을 사용한 Adapter 패턴)

***Adapter Pattern***은 기존의 클래스를 개조해 필요한 클래스를 만든다. (기존에 많이 사용되는 라이브러리에서 제공되는 그러한 클래스들을 떠올려보자)

- 필요한 메서드를 발빠르게 만들 수 있다.
- 만약 버그가 발생하는 경우에도, 기존의 클래스에서 발생할 가능성은 낮다.
그렇기에 Adapter 역할의 클래스를 중점적으로 조사할 수 있고, 이로인해 프로그램 검사도 쉬워진다.

### 예제 1. 가장 일반적인 형태

```csharp
// 오리 인터페이스
public interface Duck
{
    void quack();
    void fly();
}
```

```csharp
using UnityEngine;

// 청둥오리
public class MallardDuck : Duck
{
    public void quack()
    {
        Debug.Log("오리 : 울기 (꽥꽥)");
    }

    public void fly()
    {
        Debug.Log("오리 : 날기");
    }
}

```

```csharp
// 칠면조 인터페이스
public interface Turkey
{
    void gobble();
    void fly();
}
```

```csharp
using UnityEngine;

// 야생칠면조
public class WildTurkey : Turkey
{
    public void gobble()
    {
        Debug.Log("칠면조 : 울기 (고르륵고르륵)");
    }

    public void fly()
    {
        Debug.Log("칠면조 : 날기");
    }
}

```

```csharp
// Duck 객체가 모자라서 Turkey 객체를 대신 사용해야 하는 상황
// 인터페이스가 다르기 때문에 Turkey 객체를 바로 사용할 수는 없다.
public class TurkeyAdapter : Duck
{
    private Turkey _turkey;

    public TurkeyAdapter(Turkey turkey)
    {
        this._turkey = turkey;
    }
    
    public void quack()
    {
        _turkey.gobble();
    }

    public void fly()
    {
        _turkey.fly();
    }
}

```

```csharp
using UnityEngine;

public class Client : MonoBehaviour
{
    private void Start()
    {
        MallardDuck duck = new MallardDuck();
        WildTurkey turkey = new WildTurkey();
        Duck turkeyAdapter = new TurkeyAdapter(turkey);
        
        Debug.Log("오리 사용...");
        testDuck(duck);
        
        Debug.Log("오리 부족. 칠면조로 대체...");
        testDuck(turkeyAdapter);
    }

    void testDuck(Duck duck)
    {
        // 동일한 방법으로 사용
        duck.quack();
        duck.fly();
    }
}

```

![image (7)](https://github.com/user-attachments/assets/ab6839ab-63f1-4e1e-aca0-b9dfc51aa4bb)

Client 스크립트를 실행시키면 좌측과 같은 결과가 출력된다.

오리가 아닌 칠면조를 마치 오리처럼 사용하기 위해 TurkeyAdapter 를 활용했다.

### 예제 2.

```csharp
// PayX 회사로부터 받은 인터페이스
public interface PayX
{
    string getCreditCardNum();

    void setCreditCardNum(string creditCardNum);
}
```

```csharp
// PayY 회사로부터 받은 인터페이스
public interface PayY
{
    string getCustomerCardNum();

    void setCustomerCardNum(string customerCardNum);
}
```

### PayImpl 버전 1

```csharp
using UnityEngine;

// 우리 회사에서 PayX 인터페이스 구현
public class PayImpl : PayX
{
    private string creditCardNum;
    
    public string getCreditCardNum()
    {
        Debug.Log("PayX (Get)");

        return creditCardNum;
    }

    public void setCreditCardNum(string creditCardNum)
    {
        Debug.Log("PayX (Set)");
        this.creditCardNum = creditCardNum;
    }
}
```

### PayImpl 버전 2

```csharp
using UnityEngine;

// 우리 회사에서 PayY 인터페이스 구현
public class PayImpl : PayX, PayY
{
    // for PayY
    private string customerCardNum;
    
    public string getCustomerCardNum()
    {
        Debug.Log("PayY (Get)");
        return customerCardNum;
    }

    public void setCustomerCardNum(string customerCardNum)
    {
        Debug.Log("PayY (Set)");
        this.customerCardNum = customerCardNum;
    }
    
    // -----------------------
    
    // for PayX Method
    public string getCreditCardNum()
    {
        return getCustomerCardNum();
    }

    public void setCreditCardNum(string creditCardNum)
    {
        setCustomerCardNum(creditCardNum);
    }
}
```

```csharp
using UnityEngine;

public class MyPay : MonoBehaviour
{
    private void Start()
    {
        // for PayX : 원래 이렇게 사용중 ...

        PayImpl myPay = new PayImpl();
        myPay.setCreditCardNum("12345");
        string myCardNum = myPay.getCreditCardNum();
        // Debug.log("PayX : " + myCardNum);
        
        // for PayY : PayX 와 같은 메서드 명을 사용
        // 그러므로 코드를 바꿀 필요가 없다
        Debug.Log("PayY : " + myCardNum);
    }
}

```

본 예제는 외부 시스템 (PayX, PayY) 과 이를 실제로 사용하는 사용부 (MyPay) 를 연결하는 PayImpl 클래스에 대한 구현으로 ***Adapter Pattern***의 사용을 보여준다.

본 강의에서는 기존에는 PayX 외부 시스템을 사용함에 따라 **PayImpl 버전 1** 을 기존에 사용하고 있던 상황에서, 사용하는 외부 시스템을 PayY로 변경함에 따라

사용부 (MyPay)의 변경사항을 최소화하거나 없앨 수 있도록하는 **PayImpl 버전 2** 를 활용하여 호환성을 제공하는 방법을 보여준다.

### 예제 3.

```csharp
using UnityEngine;

/// <summary>
/// The 'Adaptee' interface
/// </summary>
public interface IUnitAction
{
    void NormalMove(Transform tr);
    void NormalStop(Transform tr);
}
```

```csharp
using UnityEngine;

/// <summary>
/// The 'Target' class
/// </summary>

// 인터페이스 : 원하는 기능
public interface INewAction
{
    void EventMove(Transform tr);
    void EventStop(Transform tr);
}
```

```csharp
using UnityEngine;

/// <summary>
/// The 'Adaptee' class
/// </summary>
public class UnitAction : MonoBehaviour, IUnitAction
{
    // 이동
    public void NormalMove(Transform tr)
    {
        tr.Translate(Vector3.forward * 1.0f);
        Debug.Log("노멀 이동");
    }

    // 정지
    public void NormalStop(Transform tr)
    {
        Debug.Log("노멀 정지");
    }
}
```

```csharp
using System.Collections;
using UnityEngine;

/// <summary>
/// The 'Adapter' class
/// </summary>
public class Adapter : MonoBehaviour, INewAction, IUnitAction
{
    public GameObject shield;

    // for INewAction
    
    public void EventMove(Transform tr)
    {
        StartCoroutine(CoEventMove(1.0f, tr));
    }

    IEnumerator CoEventMove(float tm, Transform tr)
    {
        shield.SetActive(true);
        
        // 이벤트로 두 번 이동
        tr.Translate(Vector3.forward * 1.0f);
        Debug.Log("이벤트 이동");
        yield return new WaitForSeconds(tm);
        
        tr.Translate(Vector3.forward * 1.0f);
        Debug.Log("이벤트 이동");
        yield return new WaitForSeconds(tm);

        shield.SetActive(false);
    }
    
    public void EventStop(Transform tr)
    {
        Debug.Log("이벤트 정지");
    }

    // for IUnitAction
    
    public void NormalMove(Transform tr)
    {
        EventMove(tr);
    }

    public void NormalStop(Transform tr)
    {
        EventStop(tr);
    }
}

```

```csharp
using UnityEngine;

public class Player1 : MonoBehaviour
{
    void Start()
    {
        IUnitAction act = gameObject.GetComponent<UnitAction>();
        act.NormalMove(transform);
    }
    
}

```

```csharp
using UnityEngine;

public class Player2 : MonoBehaviour
{
    void Start()
    {
        IUnitAction act = gameObject.GetComponent<Adapter>();
        act.NormalMove(transform);
    }
    
}

```
![2024-08-23213711-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/81c19243-13c4-43e3-b59b-4bb7fff49bb8)

이 예제를 동작시키면, 좌측과 같이 Player2는 Player1과는 다르게 동작한다.

Player1은 단순히 앞으로 한칸 이동하는 동작 외엔 별다른 동작이 없지만

Player2는 처음에 Shield를 켠 후 두번 이동하고, 그 후에 Shield를 끈다.

이러한 동작의 차이는 `UnitAction` 클래스와 `Adapter` 클래스의 차이에서 비롯된다. 

`Adapter` 클래스는 `IUnitAction`과 `INewAction`을 구현하여 기존의 `NormalMove`를 새로운 `EventMove`로 대체한다. 

`EventMove` 메서드에서 두 번 이동하고, 중간에 `shield`를 켜고 끄는 추가 동작을 수행한다.

이렇게 기존에 사용되던 기능을 잠시동안 대체하는 기능을 구현하는 방법으로  ***Adapter Pattern***을 사용할 수 있다.