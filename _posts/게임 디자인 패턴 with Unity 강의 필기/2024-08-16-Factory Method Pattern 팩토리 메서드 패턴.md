---
title: Factory Method Pattern 팩토리 메서드 패턴
date: 2024-08-16 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Factory Method Pattern

### 정의

> ***팩토리 메서드 패턴 (Fatory Method Pattern)*** 은 `객체를 생성하기 위한 인터페이스`를 정의한다. 
서브 클래스에서 어떤 클래스를 만들지를 결정하게 함으로써 `객체 생성을 캡슐`화한다. 
***팩토리 메서드 패턴 (Fatory Method Pattern)*** 을 이용하면 클래스의 인스턴스를 만드는 일을 서브클래스에게 맡기게 된다.

> 결국 **클라이언트**와 **유닛** 사이에 **팩토리**를 배치시켜 **추상화**를 통해 클라이언트와 유닛 간의 **결합도를 낮추고**,
**추상 클래스**를 통해 모든 유닛과 팩토리가 일정한 규칙을 따르도록 하여 **유지보수를 쉽게** 만드는 패턴!

### UML 다이어그램

![Untitled (4)](https://github.com/user-attachments/assets/77acdc68-67c8-44e8-bb9e-e21e85060222)

### 사용목적 및 용도, 장점

- 객체 생성 코드를 전부 한 객체 또는 메서드 내에서 전담하여 중복되는 코드를 줄일 수 있고, 객체의 생성을 한 곳에서 관리할 수 있다.
    - `ConcreteCreator` 클래스에만 객체 생성 코드가 구현된다.
- 동일한 인터페이스 구현으로 새로운 객체가 추가되어도, 소스의 수정을 최소화 할 수 있다.
    - 생성 부분의 수정 ⇒ `ConcreteCreator`의 `FactoryMethod()`를 수정
    - 신규 클래스의 추가 `ConcreteProduct` 추가
- 구현이 아닌, 인터페이스를 바탕으로 개발할 수 있게 되고, 결과적으로 `유연성과 확장성이 높은` 코드를 완성할 수 있다.
    - 확장성 및 소스 유지관리 측면을 고려했을 때, 사용하기 좋은 패턴.

## 예제1

### 예제1 소스코드

### 열거형과 추상 클래스들

```csharp
public enum UnitType
{
    Marine,
    Firebat,
}
```

```csharp
public abstract class Unit
{
    protected UnitType type;
    protected string name;
    protected int hp;
    protected int exp;
    public abstract void Attack();
}

```

```csharp
public abstract class UnitGenerator
{
    public List<Unit> units = new List<Unit>();

    public List<Unit> getUnits()
    {
        return units;
    }
    
    // Factory Method
    public abstract void CreateUnits();
}

```

### `Unit` 추상 클래스를 상속받아 구현하는 클래스들

```csharp
/// <summary>
/// Firebat "ConcreteProduct" class
/// </summary>
public class Firebat : Unit
{
    public Firebat()
    {
        type = UnitType.Firebat;
        name = nameof(Firebat);
        hp = 120;
        exp = 15;

        
        Debug.Log(this.name + " : 생성 !!");
    }
    
    public override void Attack()
    {
        Debug.Log(this.name + " : 공격 !!");
    }
}
```

```csharp
/// <summary>
/// Marine "ConcreteProduct" class
/// </summary>
public class Marine : Unit
{
    public Marine()
    {
        type = UnitType.Marine;
        name = nameof(Marine);
        hp = 100;
        exp = 50;
        
        Debug.Log(this.name + " : 생성!!");
    }
    
    public override void Attack()
    {
        Debug.Log(this.name + " : 공격!!");
    }
}

```

### `UnitGenerator` 추상 클래스를 상속받아 구현하는 클래스들

```csharp
/// <summary>
/// A "ConcreteCreator" class
/// </summary>
public class PatternAGenerator : UnitGenerator
{
    // Factory Method
    public override void CreateUnits()
    {
        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
    }
}
```

```csharp
/// <summary>
/// B "ConcreteCreator" class
/// </summary>
public class PatternBGenerator : UnitGenerator
{
    // Factory Method
    public override void CreateUnits()
    {
        units.Add(new Firebat());
        units.Add(new Firebat());
        units.Add(new Firebat());
        units.Add(new Firebat());

        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
        units.Add(new Marine());
    }
}

```

### `Generator`들을 사용하는 외부를 담당하는 클래스

```csharp
using System.Collections.Generic;
using UnityEngine;

public class UseFactoryMethod : MonoBehaviour
{
    private UnitGenerator[] _unitGenerators = null;

    private void Start()
    {
        _unitGenerators = new UnitGenerator[2];
        _unitGenerators[0] = new PatternAGenerator();
        _unitGenerators[1] = new PatternBGenerator();
    }
    
    // 객체의 생성에 크게 관여하지 않는다.
    // 그저 UnitGenerator가 존재함을 알고, 이 Generator에게 객체 생성을 명령할 뿐, 
    // 그리고 필요에 따라, 생성된 객체를 참조할 수 있을 뿐.
    public void DoMakeTypeA()
    {
        _unitGenerators[0].CreateUnits();

        List<Unit> units = _unitGenerators[0].getUnits();
        foreach (var unit in units)
        {
            unit.Attack();
        }
    }

    public void DoMakeTypeB()
    {
        _unitGenerators[1].CreateUnits();

        List<Unit> units = _unitGenerators[1].getUnits();
        foreach (var unit in units)
        {
            unit.Attack();
        }
    }
    
}

```

![TestingProject-FactoryMethodPattern-WindowsMacLinux-Unity2022 3 17f1_DX11_2024-03-0517-34-54-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/b8d31aba-8cda-41d8-851b-aa1a913a5af8)

- 크게 두 종류의 `추상 클래스`가 존재한다. **`Unit`** 과 **`UnitGenerator`**
- 각각의 추상 클래스를 상속받는 클래스들이 존재한다.
    - **`Unit`** 추상 클래스를 상속받는 클래스들은 유닛으로써 공통된 기능을 갖는다.
        - 예제에서는 공통된 기능 중 공격하는 기능으로 `Attack()` 함수가 예시로 등장한다.
    - **`UnitGenerator`** 추상 클래스를 상속받는 클래스들은 유닛 생성자로써 공통된 기능을 갖는다.
        - 예제에서 클래스들은 각각의 패턴대로 유닛들을 생성하는 **`CreateUnits()`** 함수가 예시로 등장한다.
- 이렇게 `추상 클래스`로 유닛과 유닛을 생성하는 클래스들의 공통 기능을 미리 선언하여, 유닛의 생성을 처음 트리거하는 부분과의 **`커플링(Coupling)`** 을 낮출 수 있다.
    - 만약에 새로운 유닛이 추가된다면, **`Unit`** 추상 클래스를 상속받도록 유닛 클래스를 만들고, **`UnitGenerator`** 에서 이를 생성하도록 추가하면 될 것이다.