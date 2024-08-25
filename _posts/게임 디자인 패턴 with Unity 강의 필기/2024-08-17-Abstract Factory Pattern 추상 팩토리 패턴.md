---
title: Abstract Factory Pattern 추상 팩토리 패턴
date: 2024-08-17 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Abstract Factory Pattern

### 정의

> ***추상 팩토리 패턴 (Abstract Factory Pattern)*** 은 ***팩토리 메서드 패턴 (Factory Method Pattern)*** 과 매우 유사하나, 팩토리 메서드 패턴은 클래스들이 `객체를 생성하는 메서드를 구현`하고 `하나의 객체를 반환`하는 게 전부이지만 추상 팩토리 패턴은 `클라이언트에 연관된 객체들의 패밀리를 반환`하는 것.

> 관련성 있는 여러 종류의 객체를 특정 그룹으로 묶어 `한번에 일관된 방식으로 생성하고 교체할 수 있도록` 만든 디자인 패턴.


### 장점

1. **관리 용이성**
    
    클래스 이름 대신 `팩토리 메소드를 사용해 객체를 생성`하기 때문에, 추후 실제 생성되는 객체가 바뀌거나 추가되어도 문제가 없다.
    
2. **보안성**
    
    클래스의 대부분의 내용을 숨기고 싶을 때, 인터페이스나 abstract를 통해서만 객체에 접근하게 할 수 있다.
    
3. **리소스 재활용성**
    
    팩토리 메소드가 반드시 객체를 새로 생성할 필요는 없고, 상황에 따라 새로 생성될수도, 기존의 것을 리턴할수도 있다.
    
4. 위의 세가지 모두 팩토리 메서드 패턴과 같으나, 추상 팩토리의 경우 여기에 더해 `팩토리에 상속 구조`를 둠으로서 `세밀한 팩토리 관리`가 가능해진다.

- 추상 팩토리 패턴을 사용하지 않은, 일반적인 프로젝트를 먼저 탐구하고, 여기서 느끼는 불편함을 이후에 해소해보자

### 예제 1 소스코드

```csharp
public abstract class RaceCapacity
{
    public abstract void expand();
}

public class SupplyDepot : RaceCapacity
{
    public override void expand()
    {
        Debug.Log("Terran Capacity +8");
    }
}

public class Pylon : RaceCapacity
{
    public override void expand()
    {
        Debug.Log("Protoss Capacity +8");
    }
}
```

```csharp
public abstract class UnitBuilding
{
    public abstract void produce();
}

public class Barracks : UnitBuilding
{
    public override void produce()
    {
        Debug.Log("Terran Unit 생산");
    }
}

public class Gateway : UnitBuilding
{
    public override void produce()
    {
        Debug.Log("Protoss Unit 생산");
    }
}
```

```csharp
public enum Race
{
    Terran,
    Protoss,
    Zerg,
}

public class CapacityFactory
{
    public static RaceCapacity MakeBuilding(Race type)
    {
        RaceCapacity capacity = null;

        switch (type)
        {
            case Race.Terran:
                capacity = new SupplyDepot();
                break;
            case Race.Protoss:
                capacity = new Pylon();
                break;
        }

        return capacity;
    }
}

public class UnitBuildingFactory
{
    public static UnitBuilding MakeBuilding(Race type)
    {
        UnitBuilding building = null;
        switch (type)
        {
            case Race.Terran:
                building = new Barracks();
                break;
            case Race.Protoss:
                building = new Gateway();
                break;
        }

        return building;
    }
}
```

```csharp
public class FactoryUse : MonoBehaviour
{
    private void Start()
    {
        RaceCapacity capacity = CapacityFactory.MakeBuilding(Race.Protoss);
        UnitBuilding building = UnitBuildingFactory.MakeBuilding(Race.Protoss);
        
        capacity.expand();
        building.produce();
    }

}

```

- 스타크래프트의 컨셉을 가져와, `RaceCapacity` 추상 클래스는 `SupplyDepot`과 `Pylon`이 상속받아 인구수 한도를 늘리는 기능을,
- `UnitBuilding` 추상 클래스는 `Barrack`과 `Gateway`가 상속받아 유닛 생산 기능을 구현한다.
- 그리고 `CapacityFactory` 클래스는 `RaceCapacity` 추상 클래스를 구현하는 클래스 중에서 선택하여 생성하고,
- `UnitBuildingFactory` 클래스는 UnigBuilding 추상 클래스를 구현하는 클래스 중에서 선택해 생성하는 팩토리 클래스이다.

**[문제점]**

- 지금 이 구조를 사용하는 클라이언트에서는 Protoss 종족을 위주로 사용하고 있는데,
- 단순히 Protoss 종족에서 Terran 종족만 바꾸는 정도의 수정이라면 큰 문제는 아니다.
- 하지만 프로젝트가 그럴듯하게 갖춰질수록, 각 종족마다 건물을 훨씬 다양하게 가질 수 있을 것이다.
- 각각의 건물마다 팩토리 클래스가 존재할 것이기에, 건물 수만큼 많은 부분을 수정해야 한다.
- 또한, 지금은 지원하지 않는 Zerg 종족을 추가할 경우에는 각각의 팩토리의 `switch` 문 마다 `case Zerg:`를 추가해야 할 것이다.

**[해결책]**

- 엘리베이터가 A_Company라는 회사의 엘리베이터라면, 엘리베이터를 구성하는 모든 부품 또한 동일한 회사의 제품일 것이다.
- 이렇게 `여러 종류의 객체를 생성`할 때 `객체들 사이에 관련성`이 있는 경우라면
- 각 종류별로 별도의 Factory를 사용하는대신,
- `관련 객체들을 일관성있게 생성하는 추상 팩토리 패턴`을 적용하는 것이 바람직 할 것이다.

그러면 이제 예제1을 수정해보자

### 예제 2 소스코드

```csharp
public abstract class RaceFactory
{
    public abstract RaceCapacity MakeCapacityBuilding();
    public abstract UnitBuilding MakeUnitBuilding();
}

public class TerranFactory : RaceFactory
{
    public override RaceCapacity MakeCapacityBuilding()
    {
        return new SupplyDepot();
    }

    public override UnitBuilding MakeUnitBuilding()
    {
        return new Barracks();
    }
}

public class ProtossFactory : RaceFactory
{
    public override RaceCapacity MakeCapacityBuilding()
    {
        return new Pylon();
    }

    public override UnitBuilding MakeUnitBuilding()
    {
        return new Gateway();
    }
}
```

```csharp
public class FactoryUse : MonoBehaviour
{
    private void Start()
    {
        RaceFactory factory = null;
        Race type = Race.Protoss;

        if (type == Race.Protoss)
            factory = new ProtossFactory();
        else
            factory = new TerranFactory();
        
        // 하나의 팩토리 객체로 모든 건물을 다 만들 수 있다.

        RaceCapacity capacity = factory.MakeCapacityBuilding();
        UnitBuilding building = factory.MakeUnitBuilding();
        
        capacity.expand();
        building.produce();;
    }
}

```

- 기존의 `CapacityFactory`와 `UnitBuildingFactory` 대신,
- 추상 클래스인 `RaceFactory`에서 CapacityBuilding과 UnitBuilding을 만들도록 한다.
- 그리고 이를 상속받는 `TerranFactory`와 `ProtossFactory`를 만들어 Buliding의 생성을 종족별로 구분한다.
    - 한 종족을 전담하는 하나의 Factory에서 CapacityBuilding과 UnitBuilding을 모두 만들도록 한다는 의미.
- 그리고 이에 맞춰 클라이언트의 코드도 살짝 변화한다.

SupplyDepot은 Barrack과 둘다 Terran 건물이라는 점에서 높은 관련성이 있고, Pylon은 Gateway와 둘다 Protoss 건물이라는 점에서 높은 관련성이 있기에, 
이렇게 관련성이 높은 건물끼리 각각을 묶어 이들을 생산할 수 있는 하나의 Factory를 정의한다.

같은 종족의 건물로 묶였기 때문에, Factory는 종족별로 하나씩 만들어 질 수 있겠다.

> 직전의 ***추상 팩토리 패턴 (Abstract Factory Pattern)*** 에서 본 두번째 예제에 ***팩토리 메서드 패턴 (Factory Method Pattern)*** 을 함께 사용하여,
두 패턴을 혼합하여 사용하는 경우를 탐구해보자.
> 

- 다음의 두 코드를 추가하자. 이는 기존의 `FactoryUse` 클래스를 대체한다.
    
    
    ```csharp
    public class FactoryMethod
    {
        public Race type = Race.Terran;
    
        public RaceFactory GetFactory()
        {
            RaceFactory factory = null;
    
            switch (type)
            {
                case Race.Terran:
                    factory = new TerranFactory();
                    break;
                case Race.Protoss:
                    factory = new ProtossFactory();
                    break;
            }
    
            return factory;
        }
    }
    
    ```
    
    ```csharp
    public class FactoryMethodUse : MonoBehaviour
    {
        private void Start()
        {
            FactoryMethod factoryMethod = new FactoryMethod();
    
            RaceFactory factory = factoryMethod.GetFactory();
            
            // 하나의 팩토리 객체로 모든 건물을 다 만들 수 있다.
            RaceCapacity capacity = factory.MakeCapacityBuilding();
            UnitBuilding building = factory.MakeUnitBuilding();
    
            capacity.expand();
            building.produce();
        }
    }
    ```
    
- 기존의 `FactoryUse` 클래스를 `FactoryMethod` 클래스와 `FactoryMethodUse` 클래스가 대체한다.
- `FactoryMethodUse` 클래스는 클라이언트나 다름 없다. 그러면 `FactoryMethod` 클래스가 추가됨은 무엇을 의미할까?
- 잘 보면 클라이언트에 해당하는`FactoryMethodUse` 클래스의 사용부에서, Factory의 종류는 특정하지 않았다. 이는 Factory의 종류와 무관하게 동작하는 코드라는 의미다.
- 어쩌면 이는 무책임하게 사용하는 것일지도 모르겠다. 클라이언트에서 무책임하게 사용하지만, 이는 충분히 추상화되었다는 말과 같은 말이 아닐까.

### 예제 3 소스코드

```csharp
public abstract class RaceCapacity : MonoBehaviour
{
    public abstract void Expand();
}
public class SupplyDepot : RaceCapacity
{
    public override void Expand()
    {
        Debug.Log("Terran Capacity +8");
    }
}
public class Pylon : RaceCapacity
{
    public override void Expand()
    {
        Debug.Log("Protoss Capacity +8");
    }
}
```

```csharp
public abstract class UnitBuilding : MonoBehaviour
{
    public abstract void produce();
}
public class Barracks : UnitBuilding
{
    public override void produce()
    {
        Debug.Log("Terran Unit 생산");
    }
}
public class Gateway : UnitBuilding
{
    public override void produce()
    {
        Debug.Log("Protoss Unit 생산");
    }
}
```

```csharp
public abstract class RaceFactory : MonoBehaviour
{
    public abstract GameObject MakeCapacityBuilding();
    public abstract GameObject MakeUnitBuilding();
}
```

```csharp
public class TerranFactory : RaceFactory
{
    public GameObject supply;
    public GameObject barracks;
    
    public override GameObject MakeCapacityBuilding()
    {
        return Instantiate(supply, new Vector3(-1.0f, 1.0f, 0.0f), Quaternion.identity);
    }

    public override GameObject MakeUnitBuilding()
    {
        return Instantiate(barracks, new Vector3(1.0f, 0.5f, 0.0f), Quaternion.identity);
    }
}
```

```csharp
public class ProtossFactory : RaceFactory
{
    public GameObject pylon;
    public GameObject gateway;
    
    public override GameObject MakeCapacityBuilding()
    {
        return Instantiate(pylon, new Vector3(-1.0f, 1.0f, 0.0f), Quaternion.identity);
    }

    public override GameObject MakeUnitBuilding()
    {
        return Instantiate(gateway, new Vector3(1.0f, 0.5f, 0.0f), Quaternion.identity);
    }
}
```

```csharp
public class FactoryMethod : MonoBehaviour
{
    public Race type = Race.Terran;

    public RaceFactory GetFactory()
    {
        RaceFactory factory = null;

        switch (type)
        {
            case Race.Terran:
                factory = GetComponent<TerranFactory>();
                break;
            case Race.Protoss:
                factory = GetComponent<ProtossFactory>();
                break;
        }

        return factory;
    }
}
```

![TestingProject-AbstractFactoryPattern-WindowsMacLinux-Unity2022 3 17f1__DX11_2024-03-0701-15-28-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/cbc5d8d5-9ee0-4bcd-9ada-b6c54c62e650)

- 이는 지금까지 본 팩토리 패턴을 유니티 내에서 그럴듯하게 구현한 것.
- 전반적인 개념은 지금까지 배운것과 동일하고, 유니티에서 구현함에 따라 객체를 new 하고 만들던 것을 프리팹을 instantiate하는 것과 GetComponent로 대체.