---
title: Prototype Pattern 프로토타입 패턴
date: 2024-08-18 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Prototype Pattern

### UML

![Untitled (5)](https://github.com/user-attachments/assets/12424d23-0146-4c55-9b40-ce4217b34bad)

- Prototype을 상속받아 `clone()` 메소드를 구현하는 ConcretePrototype으로 구성되어 있음.
- 원형을 하나 가지고 있고, 원형의 Prototype을 상속받아 `clone()` 메소드를 구현한 상태라면 Client에서 원할 때 원형으로 클론을 만들어 사용할 수 있다.

### 사용하는 이유

- Prototype pattern은 비슷한 오브젝트를 지속적으로 생성해야 할 때 유용하게 사용할 수 있다.
    - 오브젝트를 직접 생성할 때에는 높은 비용이 들기 때문에, 이를 대체할 수 있는 유용한 패턴.
- 원본 오브젝트로부터 새로운 오브젝트를 만들어내며, 각 객체에 따라 데이터 수정이 가능한 메커니즘을 함께 제공한다.
- ***동적 클래스 생성***
    - 이 패턴의 중요한 키워드가 된다.
    - 약간의 설정 변경으로 비슷하지만 다른 클래스로의 확장이 가능한 경우, 
    세부 클래스를 미리 명세하지 않고 런타임에 원형을 복제하고, 그 복사본을 수정함으로써 **‘동적 클래스 생성’**을 가능케 하는 패턴이 Prototype 패턴.

### 예제 1 소스코드

```csharp
public interface IUnit
{
    IUnit Clone();
}

public class Marine : IUnit
{
    public int Hp { get; set; }
    public int AttackPower { get; set; }
    
    public IUnit Clone()
    {
        // 얕은 복사
        return this.MemberwiseClone() as IUnit;
    }
}

public class Firebat : IUnit
{
    public int Age { get; set; }
    public int AttackPower { get; set; }
    
    public IUnit Clone()
    {
        return this.MemberwiseClone() as IUnit;
    }
}
```

```csharp
using System;
using UnityEngine;

public class UnitManager : MonoBehaviour
{
    private void Start()
    {
        Marine marine = new Marine();
        marine.Hp = 25;
        marine.AttackPower = 5;
        
        // marine을 clone 메소드로 복사하자
        
        Marine marineClone = marine.Clone() as Marine;
        marineClone.Hp = 30;
        marineClone.AttackPower = 6;
        
        Debug.Log("Marine의 스텟");
        Debug.Log($"HP : {marine.Hp} / AttackPower : {marine.AttackPower}");
        
        Debug.Log("복제된 Marine의 스텟");
        Debug.Log($"HP : {marineClone.Hp} / AttackPower : {marineClone.AttackPower}");

    }
}

```

- `IUnit`이라는 인터페이스로, 이를 상속하는 클래스에게 스스로를 복제하는 메소드를 구현하게 하고,
- 위 예제에서는 얕은 복사를 이용해 새로운 객체를 만들어낸다.
- 하나의 객체를 생성할 때, `new` 키워드를 사용하는 것과 같은 방식으로 직접 생성하는 것 보다, 이미 생성된 객체를 원본으로 삼아 복제를통해 만들어낸다는 것이 중요 아이디어이다.
- 이러한 방식은 생성, 초기화 과정이 복잡한 객체를 생성할 때 도움이 되며 객체의 생성과 파괴의 빈도를 줄일 수 있어 GC의 부담을 줄이고 성능을 개선할 수 있다.

이러한 아이디어는 유니티에 이미 `프리팹 (Prefab)`이라는 형태로 구현되어있다.

 프리팹의 형태로 오브젝트의 원본을 저장하고, 이를 `instantiate` 함으로써 복제하여 새로운 객체를 생성해낸다.

### 예제 2 소스코드

```csharp
public class Marine : MonoBehaviour
{
    private void Start()
    {
        float r = Random.Range(0.0f, 1.0f);
        float g = Random.Range(0.0f, 1.0f);
        float b = Random.Range(0.0f, 1.0f);

        Renderer[] rends = GetComponentsInChildren<Renderer>();

        foreach (var rend in rends)
            rend.material.color = new Color(r, g, b, 0f);
    }

    private void OnCollisionEnter(Collision other)
    {
        if (other.gameObject.name == "Plane")
        {
            Debug.Log("OnCollisionEnter");
            Destroy(gameObject);
        }
    }
}
```

```csharp
public class UnitManager : MonoBehaviour
{
    public GameObject unit;
    public Transform tr;

    public void CreateUnit()
    {
        Vector3 pos = tr.position + Random.insideUnitSphere * 3;
        pos.y = tr.position.y;
            
        GameObject obj = Instantiate(unit, pos, quaternion.identity);
    }
    
}

```

![TestingProject-PrototypePattern-WindowsMacLinux-Unity2022 3 17f1__DX11_2024-03-1018-30-23-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/362f22de-142f-4f0f-b138-14adf6dc40af)

- 이 패턴의 개념은 이미 유니티에 너무 잘 구현되어있기 때문에, 코드는 굉장히 단순하다.
- 프리팹을 기준으로 `instantiate` 의 반복. 그 뿐이다.
- `Marine`은 초기값으로 매번 다른 색을 갖기 때문에, 각각의 `Marine`은 다른 색을 갖게 됨을 알 수 있다.