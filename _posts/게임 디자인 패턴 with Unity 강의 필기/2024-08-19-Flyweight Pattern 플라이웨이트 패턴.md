---
title: Flyweight Pattern 플라이웨이트 패턴
date: 2024-08-19 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Flyweight Pattern

먼저 여러 그루의 나무들을 배치한다고 생각해보자.

![Untitled (6)](https://github.com/user-attachments/assets/98e35147-c382-4c4f-a7f9-675762754e0b)

이렇게 배치된 나무마다 하나의 객체로 판단하고 각각의 나무들을 모두 메모리에 인스턴싱할 수 있겠다.

![Untitled (7)](https://github.com/user-attachments/assets/7cb20f60-cbfe-4986-82fd-37c8a9cc9fc8)

그런데 만약 이렇게 한다면?

어차피 나무들의 메시, 나무껍질, 잎사귀 등등 (이하 “모델”이라 칭함) 에 대한 정보는 동일하다면, 
이렇게 메모리에 한번만 올려도 동일한 기능을 할 수 있지 않을까?

- 이처럼 모든 나무가 같은 위치에 있지는 않을테니, ***서로 다른 값을 가질 수 있는 정보***만 개별 인스턴스에 구분지어놓고,
- ***값이 동일해서 공유가 가능한 데이터***는 중복되지 않게 한번만 메모리에 올리고 이를 모두가 참조하게 한다면

### 메모리를 절약할 수 있지 않을까?

### 이렇게 경량 패턴은 객체 데이터를 두 종류로 나눈다.

1. ***모든 객체의 데이터 값이 같아서 공유할 수 있는 데이터***
2. ***인스턴스마다 값이 달라질 수 있는 나머지 데이터***

특히나 이는 유니티에서, 결과적으로 동일한 메쉬면 공유하여 사용하기 때문에 DrawCall의 횟수가 줄어든다.

### 이미 유니티에는 이 Flyweight Pattern이 적용되어 있다.

공유기능을 유지하고 싶다면, 인스턴스를 요청받을 때 같은 걸 만들어놓은 게 있는지 확인하고, 있다면 그것을 반환하면 된다.

이러려면, 객체를 생성하는 기능을 구현할 때, 기존의 객체가 이미 있는지 유무를 확인하도록 하는 코드를 인터페이스 내부에 포함시켜야 한다.

이러한 맥락에서, ***Object Pool 패턴***과 ***Factory 패턴***을 함께 사용하는 것이 호환성과 사용성에서 좋겠다.

- 이전에 만들어놓은 경량 패턴 객체를 관리하고, 반환하는 데 ***Object Pool 패턴***
- 생성 내부에 추가적인 로직을 구현하고 이를 추상화하는 데 ***Factory 패턴***

### 예제 1

```csharp
public interface Unit
{
    string getName();
}

public class Marine : Unit
{
    private string name;
    private int hp;

    public Marine(string name)
    {
        this.name = name;
    }
    
    public string getName()
    {
        return name;
    }
}
```

```csharp
using System.Collections.Generic;
using UnityEngine;

public class UnitFactory : MonoBehaviour
{
    private static Dictionary<string, Marine> dic = new Dictionary<string, Marine>();

    public static Marine getPerson(string name)
    {
        if (!dic.ContainsKey(name))
        {
            Marine tmp = new Marine(name);
            dic.Add(name, tmp);
        }

        return dic[name];
    }

}

```

```csharp
using UnityEngine;

public class FlyweightUse : MonoBehaviour
{
    private void Start()
    {
        Marine p1 = UnitFactory.getPerson("홍길동");
        Marine p2 = UnitFactory.getPerson("전우치");
        Marine p3 = UnitFactory.getPerson("홍길동");

        Debug.Log(p1 == p2);
        Debug.Log(p1 == p3);
        
        Debug.Log("name : " + p1.getName());
    }
}

```

- `Unit` 인터페이스와 이를 구현하는 `Marine` 클래스가 존재한다.
    - 인터페이스에는 간단히 `getName()` 이라는 메서드가 선언되었고, 이것이 `name` 이라는 변수로 지정된 `string`을 반환하는 형태로 구현되었다.
- `UnitFactory` 에서는 `Object Pooling`을 위해서 string을 키로, Marine을 값으로 갖는 `Dictionary`를 관리한다.
    - `getPerson()` 메서드 내부에서 이 `Dictionary`를 확인하고 조건부로 객체를 생성하는 로직이 포함되어 있는 걸 볼 수 있다.

![image](https://github.com/user-attachments/assets/f7d2feba-c9da-4db5-a1f5-f15843ddf4b3){: w="250" }

- 실행 결과는 위와 같다.
- `p1` 과 `p2`는 애초에 서로 다른 이름을 `getPerson()`의 인자로 전달했기에, 서로 완전히 다른 객체고 그에 따라 False를 출력하고
- `p1` 과 `p3`는 동일한 이름을 `getPerson()`의 인자로 전달했기에 , `p3`는 `p1`과 동일한 객체를 가리킨다. 그래서 두번째로는 True가 출력된 것이다.

### 예제 2

```csharp
public interface Unit
{
    string getName();
    void setName(string name);
}
```

```csharp
using UnityEngine;

public class Marine : MonoBehaviour, Unit
{
    public string name;
    public int hp;

    public Marine(string name)
    {
        this.name = name;
    }
    
    public void setName(string name)
    {
        this.name = name;
    }
    
    public string getName()
    {
        return name;
    }

}
```

```csharp
using System.Collections.Generic;
using UnityEngine;

public class UnitFactory : MonoBehaviour
{
    private Dictionary<string, GameObject> dic = new Dictionary<string, GameObject>();
    public GameObject marine;
    
    public GameObject getMarine(string name)
    {
        if (!dic.ContainsKey(name))
        {
            float x = Random.Range(-10, 11);
            float z = Random.Range(-10, 11);
            Vector3 pos = new Vector3(x, 1, z);

            GameObject obj = Instantiate(marine, pos, Quaternion.identity);
            obj.GetComponent<Marine>().setName(name);
            dic.Add(name, obj);
        }

        return dic[name];
    }

}

```

```csharp
using UnityEngine;

public class FlyweightUse : MonoBehaviour
{
    private void Start()
    {
        UnitFactory factory = GetComponent<UnitFactory>();

        for (int i = 0; i < 10; i++)
        {
            factory.getMarine("홍길동" + i);
        }

        GameObject p1 = factory.getMarine("홍길동");
        GameObject p2 = factory.getMarine("전우치");
        GameObject p3 = factory.getMarine("홍길동");
        
        if(p1 == p3)
            Debug.Log("name : " + p1.GetComponent<Marine>().getName());

    }
}

```

- `Unit` 인터페이스는 예제1과 유사하게 간단한 인터페이스고, `Marine` 클래스는 `Unit`  인터페이스와 `MonoBehaviour` 를 모두 상속받아 GameObject에 부착될 수 있다.
    - `Unit` 인터페이스에 `setName(string name)` 메서드가 추가되었고, 이는 `Marine` 클래스에서 인자로 받은 name을 `Marine` 클래스의 name 변수에 대입하는 형태로 구현되었다.
- `UnitFactory` 는 살짝의 변화가 있었는데, `Dictionary`의 값을 `Marine` 에서 `GameObject`로 바꾸어 유니티의 방식에 맞춘 ***Object Pooling***이 가능하도록 하였다.
    - `getMarine()` 메서드는 인자로 받은 이름을 `Dictionary`에서 찾고 있다면 그것을, 없다면 무작위 위치에 새로 만들어 이를 반환한다.

### 예제 2를 직접 실행해보자

![Untitled (8)](https://github.com/user-attachments/assets/a03e928c-ad68-4d60-bef7-1a922933f99a)

***Flyweight Pattern*** 의 의미는 앞서 나무를 활용해 설명했던 것 처럼, 
중복되어 사용하는 정보는 공유시키도록 하여 성능을 향상시키는 데 있다.

그럼 유니티에서 이를 확인해보자. 먼저, 성능상의 차이를 확인하기 위해 Profiler를 활용할 필요가 있다.

Profiler는 `Windows > Analysis > Profiler`로 추가할 수 있다.

이를 추가하고, 실행해보자.

![Untitled (9)](https://github.com/user-attachments/assets/f9264719-6979-40bc-8f48-975c962f4e16)

지금 집중할 것은 바로 *Render Textures* 라는 수치다.

현재, `FlyweightUse` 스크립트의 `Start()`함수에서 getMarine()을 한번만 
호출하도록 나머지를 주석 처리했고, 그 결과는 좌측의 이미지와 같다.

현재 10 / 171.2 MB 라는 수치를 보이고 있다.

먼저, Render Texture는 Unity에서 렌더링된 이미지를 텍스처로 저장할 수 
있게 해주는 기능이다. Profiler에서 *Render Textures* 라는 수치는 메모리 사용량과 
 Render Texture의 수를 의미한다. 

즉, 위의 이미지는 현재

1. 프로파일링 시점에서사용하고 있는 렌더 텍스처가 총 ***10***개이며,
2. 10개의 렌더 텍스처가 사용하고 있는 총 메모리 용량이 ***171.2 MB***라는 것이다.

그리고 `FlyweightUse` 스크립트의 `Start()`함수에서 주석처리한 부분을 풀고 다시 실행시켜 보자.

현재 총 12개의 Marine 프리팹이 GameObject로 instantiate 되어 있지만, 
*Render Textures* 는 변하지 않았음을 알 수 있다.

이는 유니티가 자체적으로 ***Flyweight Pattern*** 을 사용함을 알 수 있는 부분이다.

오늘도 또 하나 배웠다.

![Untitled (10)](https://github.com/user-attachments/assets/982ff9e4-3d41-4563-805a-5c19f7833208)