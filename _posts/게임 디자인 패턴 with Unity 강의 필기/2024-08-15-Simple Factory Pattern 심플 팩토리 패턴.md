---
title: Simple Factory Pattern 심플 팩토리 패턴
date: 2024-08-15 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Simple Factory Pattern
- 일반적인 팩토리 패턴은 무언가 객체를 생성하고자 할 때 사용하는 패턴.
- 객체의 생성을 쉽게 하기 위해서, 이를 위한 클래스를 만들어 관련된 역할과 기능을 몰아주는 것.
- 그 중 가장 기본이 되는 것이 바로 **심플 팩토리 패턴 (Simple Factory Pattern).**
- 이는 이후에 설명할 **팩토리 메서드 패턴 (Factory Method Pattern)** 이나, **추상 팩토리 패턴 (Abstract Factory Pattern)** 의 기본이 된다.

- 간단한 심플 팩토리 패턴은, 주어진 입력을 기반으로 다른 유형의 객체를 반환하는 메서드를 갖는 팩토리 클래스를 활용하는 것.

### 예제1 소스코드

```csharp
public abstract class Unit
{
    public abstract void Move();
}
```

```csharp
public class Marine : Unit
{
    public Marine()
    {
        Debug.Log("Marine 생성");
    }
    
    public override void Move()
    {
        Debug.Log("Marine 이동");
    }
}
```

```csharp
public class Firebat : Unit
{
    public Firebat()
    {
        Debug.Log("Firebat 생성");
    }
    
    public override void Move()
    {
        Debug.Log("Firebat 이동");
    }
}

```

```csharp
public class Medic : Unit
{
    public Medic()
    {
        Debug.Log("Medic 생성");
    }
    
    public override void Move()
    {
        Debug.Log("Medic 이동");
    }
}
```

```csharp
public enum UnitType
{
    Marine,
    Firebat,
    Medic,
}
```

```csharp
public class UnitFactory
{
    public static Unit CreateUnit(UnitType type)
    {
        Unit unit = null;

        switch (type)
        {
            case UnitType.Marine:
                unit = new Marine();
                break;
            case UnitType.Firebat:
                unit = new Firebat();
                break;
            case UnitType.Medic:
                unit = new Medic();
                break;
        }
        return unit;
    }
}
```

```csharp
public class FactoryUse : MonoBehaviour
{
    private void Start()
    {
        Unit unit1 = UnitFactory.CreateUnit(UnitType.Marine);
        Unit unit2 = UnitFactory.CreateUnit(UnitType.Firebat);
        Unit unit3 = UnitFactory.CreateUnit(UnitType.Medic);
        
        unit1.Move();
        unit2.Move();
        unit3.Move();
    }
}

```

- `Unit` 추상 클래스는 `Move()` 라는 추상 메서드를 하나 가지며, 이를 `Marine`, `Firebat`, `Medic`이 상속받는다. 이 세 서브 클래스는 모두 `Move()` 메소드를 구현한다.
- `FactoryUse` 스크립트를 Scene내의 GameObject에 부착시키고 Play하면 Console Window에 다음과 같은 결과를 출력한다.
    
    ![Untitled (3)](https://github.com/user-attachments/assets/3ff4f9a1-6e26-4df5-abac-7a86cee770c2)
    

### 예제2 소스코드

```csharp
public abstract class Unit : MonoBehaviour
{
    public abstract void Move();
}
```

Marine과 Firebat 클래스는 예제1과 동일하다.

```csharp
public enum UnitType
{
    Marine,
    Firebat,
}
```

```csharp
using UnityEngine;
using Random = UnityEngine.Random;

public class FactoryUnit : MonoBehaviour
{
    public GameObject marine = null;
    public GameObject firebat = null;

    public GameObject createUnit(UnitType type)
    {
        GameObject unit = null;

        float x = Random.Range(0, 6);
        float z = Random.Range(0, 6);
        
        switch (type)
        {
            case UnitType.Marine:
                unit = Instantiate(marine, new Vector3(x, 1.0f, z), Quaternion.identity);
                break;
            case UnitType.Firebat:
                unit = Instantiate(firebat, new Vector3(x, 0.5f, z), Quaternion.identity);
                break;
        }
        return unit;
    }
}
```

```csharp
public class FactoryUse : MonoBehaviour
{
    private FactoryUnit factory = null;
    public GameObject unit1 = null;
    public GameObject unit2 = null;
    public GameObject unit3 = null;
    
    private void Start()
    {
        factory = GetComponent<FactoryUnit>();

        unit1 = factory.createUnit(UnitType.Marine);
        unit2 = factory.createUnit(UnitType.Firebat);
        unit3 = factory.createUnit(UnitType.Firebat);

        StartCoroutine(nameof(UnitAction));
    }

    IEnumerator UnitAction()
    {
        yield return new WaitForSeconds(0.2f);
        
        unit1.GetComponent<Unit>().Move();
        unit2.GetComponent<Unit>().Move();
    }
}
```

- **`FactoryUnit`** 클래스가 바로 이 패턴에서 팩토리가 된다.
- **`FactoryUse`** 클래스에서 **`FactoryUnit`** 클래스 인스턴스를 활용해 `createUnit(UnitType type)` 메서드로 새로운 `GameObject`를 만들도록 지시하고, 이렇게 생성된 `GameObject`들에게 명령을 내린다.
- 생성의 빈도가 잦은 특정한 종류, 카테고리의 객체들을 어느 한 곳에서 집중적으로 관리할 때에 매우 적합한 패턴으로 보인다.
    
    ![TestingProject-SimpleFactoryPattern-WindowsMacLinux-Unity2022 3 17f1_DX11_2024-03-0417-45-36-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/61c6e105-1dd0-45d3-863c-8c3d8081000d)

    
    play직후에, 각각 Marine과 Firebat의 생성과 이동이 Console Window에 텍스트로 출력된다.