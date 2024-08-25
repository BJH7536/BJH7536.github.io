---
title: Builder Pattern 빌더 패턴
date: 2024-08-21 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Builder Pattern

***Bulider Pattern*** 은 복잡한 유형의 오브젝트를 작성하는 데 사용된다.

객체를 생성할 때 

- 그 객체를 구성하는 일부분들을 먼저 생성하고,
- 이들을 조합함으로써 객체 전체를 생성한다.

생성할 객체의 종류를 손쉽게 추가할 수 있고, 손쉬운 확장이 가능한 설계.

### 추상 팩토리 패턴 (Abstract Factory Pattern)과의 차이점

***Builder Pattern***

- 복잡한 객체의 단계별 생성에 중점을 두고 있는 패턴.
- 마지막 단계에서 생성한 제품을 반환한다.

***Abstract Factory Pattern***

- 제품의 유사군들이 존재하는 경우, 유연한 설계에 중점을 두는 패턴.
- (단계마다) 만드는 즉시 제품을 반환한다.

***BuilderPattern*** 은 다음의 4가지로 구성된다.

1. Builder 인터페이스 : 객체 생성을 위한 단계를 정의한다.
2. Concrete Builder 클래스 : Builder 인터페이스를 구현하며, 구체적인 제품을 생성한다.
3. Director 클래스 : Builder 인터페이스를 사용해 객체 생성을 지시한다. 객체 생성의 순서를 정의한다. 
4. Product 클래스 : 최종적으로 생성되는 복잡한 객체다.

## 예제 1

### 최종 생산품이 될 클래스 Vehicle

```csharp
using System.Collections.Generic;

// 최종 생산품이 될 클래스
public class Vehicle
{
    /// <summary>
    /// 차량의 종류 (Car, MotorCycle)
    /// </summary>
    public VehicleType vehicleType;
    
    /// <summary>
    /// 차량의 부품들을 담는 List<string>
    /// </summary>
    private List<string> _parts = new List<string>();
    
    // 생성자
    public Vehicle(VehicleType vehicleType)
    {
        this.vehicleType = vehicleType;
    }

    /// <summary>
    /// 위 차량에 부품을 추가하는 메서드
    /// </summary>
    /// <param name="desc">부품</param>
    public void AddPart(string desc)
    {
       _parts.Add(desc); 
    }

    /// <summary>
    /// 위 차량의 모든 부품을 반환하는 메서드
    /// </summary>
    /// <returns></returns>
    public string GetPartsList()
    {
        string partsList = vehicleType.ToString() + " parts:\n\t";
        foreach (string part in _parts)
        {
            partsList += part + " ";
        }

        return partsList;
    }
    
}
```

### 차량을 만드는 Bulider를 위한 인터페이스 IVehicleBuilder

```csharp
/// <summary>
/// 차량을 만드는 Bulider를 위한 인터페이스.
/// </summary>
public interface IVehicleBuilder
{
    Vehicle getVehicle();

    void BuildFrame();
    void BuildEngine();
    void BuildWheels();
}

public enum VehicleType
{
    Car,
    MotorCycle,
}

```

### CarBuilder 클래스

```csharp
/// <summary>
/// CarBuilder 클래스는
/// 차량은 강철 프레임으로, 엔진은 100마력으로, 바퀴는 앞(좌, 우) 뒤(좌, 우) 총 4개의 바퀴로 구성되어 있다.
/// </summary>
public class CarBuilder : IVehicleBuilder
{
    private Vehicle _vehicle;

    public Vehicle getVehicle()
    {
        return _vehicle;
    }

    public CarBuilder()
    {
        // 빌더가 만들어질 때 빈 차도 하나 같이 만든다.
        _vehicle = new Vehicle(VehicleType.Car);
    }
    
    public void BuildFrame()
    {
        _vehicle.AddPart("강철");
    }

    public void BuildEngine()
    {
        _vehicle.AddPart("100마력");
    }

    public void BuildWheels()
    {
        _vehicle.AddPart("앞.왼쪽 바퀴");
        _vehicle.AddPart("앞.오른쪽 바퀴");
        _vehicle.AddPart("뒤.왼쪽 바퀴");
        _vehicle.AddPart("뒤.오른쪽 바퀴");
    }
}

```

### MotorCycleBuilder 클래스

```csharp
/// <summary>
/// MotorCycleBuilder 클래스는
/// 차량은 알루미늄 프레임으로, 엔진은 50마력으로, 바퀴는 앞, 뒤 총 2가지 바퀴로 구성되어 있다.
/// </summary>
public class MotorCycleBuilder : IVehicleBuilder
{
    public Vehicle _vehicle;

    public Vehicle getVehicle()
    {
        return _vehicle;
    }

    public MotorCycleBuilder()
    {
        // 빌더가 만들어질 때 빈 모터사이클도 하나 같이 만든다.
        _vehicle = new Vehicle(VehicleType.MotorCycle);
    }
    
    public void BuildFrame()
    {
        _vehicle.AddPart("알루미늄");
    }

    public void BuildEngine()
    {
        _vehicle.AddPart("50마력");
    }

    public void BuildWheels()
    {
        _vehicle.AddPart("앞 바퀴");
        _vehicle.AddPart("뒤 바퀴");
    }
}

```

### 우리의 Director 클래스 Engineer

```csharp
/// <summary>
/// 우리의 Director 클래스
/// </summary>
public class Engineer
{
    /// <summary>
    /// 인자로 들어온 Builder가 단계벌로 차량을 만들도록 시키는 함수
    /// </summary>
    /// <param name="vehicleBuilder"></param>
    public void Construct(IVehicleBuilder vehicleBuilder)
    {
        vehicleBuilder.BuildFrame();
        vehicleBuilder.BuildEngine();
        vehicleBuilder.BuildWheels();
    }
}

```

### 유니티에서 직접 사용

```csharp
using UnityEngine;

public class BuilderUse : MonoBehaviour
{
    void Start()
    {
        // Director와 Builder들을 인스턴스화
        Engineer engineer = new Engineer();
        CarBuilder carBuilder = new CarBuilder();
        MotorCycleBuilder motorCycleBuilder = new MotorCycleBuilder();
        
        // 빌더를 통해 구성해야 할 제품을 구성한다.
        engineer.Construct(carBuilder);
        engineer.Construct(motorCycleBuilder);

        // 최종 생산된 제품을 반환받는다.
        Vehicle car = carBuilder.getVehicle();
        Debug.Log(car.GetPartsList());

        Vehicle motorCycle = motorCycleBuilder.getVehicle();
        Debug.Log(motorCycle.GetPartsList());
    }
}

```

## 예제 2

### 최종 생산품이 될 클래스 Vehicle

```csharp
using UnityEngine;

// 최종 생산품이 될 클래스
public class Vehicle : MonoBehaviour
{
    /// <summary>
    /// 차량의 종류 (Car, MotorCycle)
    /// </summary>
    public VehicleType vehicleType;

    public void setVehicleType(VehicleType vehicleType)
    {
        this.vehicleType = vehicleType;
    }
    
    public void AddPart(GameObject part, Vector3 localPosition)
    {
        GameObject obj = GameObject.Instantiate(part, transform, true);
        obj.transform.localPosition = localPosition;
    }

    /// <summary>
    /// 위 차량의 모든 부품을 반환하는 메서드
    /// </summary>
    /// <returns></returns>
    public string GetPartsList()
    {
        string partsList = vehicleType.ToString() + " parts:\n\t";

        for (int i = 0; i < transform.childCount; i++)
        {
            partsList += transform.GetChild(i).gameObject.name + " ";
        }

        return partsList;
    }
    
}

```

### 차량을 만드는 Builder를 위한 인터페이스

```csharp
/// <summary>
/// 차량을 만드는 Bulider를 위한 인터페이스.
/// </summary>
public interface IVehicleBuilder
{
    Vehicle getVehicle();

    void BuildFrame();
    void BuildEngine();
    void BuildWheels();
}

public enum VehicleType
{
    Car,
    MotorCycle,
}

```

### CarBuilder 클래스

```csharp
using UnityEngine;

/// <summary>
/// CarBuilder 클래스는
/// 차량은 강철 프레임으로, 엔진은 100마력으로, 바퀴는 앞(좌, 우) 뒤(좌, 우) 총 4개의 바퀴로 구성되어 있다.
/// </summary>
public class CarBuilder : MonoBehaviour, IVehicleBuilder
{
    public GameObject ParentOfVehicle;
    public GameObject frame;
    public GameObject engine;
    public GameObject[] Wheels;
    
    private Vehicle _vehicle;

    public Vehicle getVehicle()
    {
        return _vehicle;
    }

    public void makeVehicle()
    {
        // _vehicle = new Vehicle(VehicleType.Car);
        GameObject obj = Instantiate(ParentOfVehicle);
        _vehicle = obj.GetComponent<Vehicle>();
        _vehicle.setVehicleType(VehicleType.Car);
    }
    
    public void BuildFrame()
    {
        _vehicle.AddPart(frame, Vector3.zero);
    }

    public void BuildEngine()
    {
        _vehicle.AddPart(engine, new Vector3(0, 0.5f, 0));
    }

    public void BuildWheels()
    {
        _vehicle.AddPart(Wheels[0], new Vector3(0.75f, -0.5f, 0.5f));
        _vehicle.AddPart(Wheels[1], new Vector3(-0.75f, -0.5f, 0.5f));
        _vehicle.AddPart(Wheels[2], new Vector3(0.75f, -0.5f, -0.5f));
        _vehicle.AddPart(Wheels[3], new Vector3(-0.75f, -0.5f, -0.5f));
    }
}

```

### MotorCycle 클래스

```csharp
using UnityEngine;

/// <summary>
/// MotorCycleBuilder 클래스는
/// 차량은 알루미늄 프레임으로, 엔진은 50마력으로, 바퀴는 앞, 뒤 총 2가지 바퀴로 구성되어 있다.
/// </summary>
public class MotorCycleBuilder : MonoBehaviour, IVehicleBuilder
{
    public GameObject ParentOfVehicle;
    public GameObject frame;
    public GameObject engine;
    public GameObject[] Wheels;
    
    public Vehicle _vehicle;

    public Vehicle getVehicle()
    {
        return _vehicle;
    }

    public void makeVehicle()
    {
        // _vehicle = new Vehicle(VehicleType.MotorCycle);
        GameObject obj = Instantiate(ParentOfVehicle);
        _vehicle = obj.GetComponent<Vehicle>();
        _vehicle.setVehicleType(VehicleType.MotorCycle);
    }
    
    public void BuildFrame()
    {
        _vehicle.AddPart(frame, Vector3.zero);
    }

    public void BuildEngine()
    {
        _vehicle.AddPart(engine, new Vector3(0, 0.5f, 0));
    }

    public void BuildWheels()
    {
        _vehicle.AddPart(Wheels[0], new Vector3(1.5f, 0, 0));
        _vehicle.AddPart(Wheels[0], new Vector3(-1.5f, 0, 0));
    }
}

```

### Director 역할을 할 클래스

```csharp
/// <summary>
/// 우리의 Director 클래스
/// </summary>
public class Engineer
{
    /// <summary>
    /// 인자로 들어온 Builder가 단계벌로 차량을 만들도록 시키는 함수
    /// </summary>
    /// <param name="vehicleBuilder"></param>
    public void Construct(IVehicleBuilder vehicleBuilder)
    {
        vehicleBuilder.BuildFrame();
        vehicleBuilder.BuildEngine();
        vehicleBuilder.BuildWheels();
    }
}

```

### 유니티에서 사용할 BuilderUse

```csharp
using UnityEngine;

public class BuilderUse : MonoBehaviour
{
    void Start()
    {
        // Engineer와 Builder들의 인스턴스들을 받아온다.
        // 그리고 Builder들은 각자의 차량을 만들도록 한다. (예제 1의 생성자에 해당)
        Engineer engineer = new Engineer();
        CarBuilder carBuilder = GetComponent<CarBuilder>();
        carBuilder.makeVehicle();
        MotorCycleBuilder motorCycleBuilder = GetComponent<MotorCycleBuilder>();
        motorCycleBuilder.makeVehicle();
        
        // 빌더를 통해 제품을 구성한다.
        engineer.Construct(carBuilder);
        engineer.Construct(motorCycleBuilder);

        // 최종 생산된 제품을 받는다. 
        Vehicle car = carBuilder.getVehicle();
        Debug.Log(car.GetPartsList());

        Vehicle motorCycle = motorCycleBuilder.getVehicle();
        Debug.Log(motorCycle.GetPartsList());

        // 최종 생산된 제품의 위치 지정
        car.transform.position = new Vector3(-3, 0, 0);
        motorCycle.transform.position = new Vector3(3, 0, 0);
    }
}

```

![TestingProject-ObjectPoolPattern-WindowsMacLinux-Unity2022 3 17f1__DX11_2024-03-2414-55-46-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/0c08619a-f2ef-4ff7-896c-468c9ce35408)

좌측은 Car, 우측은 MotorCycle.

노란 구는 두 차량에서 모두 사용되는 Engine이며,

각각의 몸체는 각각의 Frame.

두 차량은 모두 같은 Wheel을 사용한다.

두 차량은 GameManager 라는 이름을 가진 GameObject에 부착된 
CarBulider와 MotorCycleBuilder 그리고 BuilderUse 스크리트에 의해 만들어짐.

***BuilderPattern*** 은 복잡한 최종 객체를 만들기 위해 여러 부분을 단계적으로 만들어 나갈 수 있다는 점에서, 캐릭터를 만드는 데 사용될 수 있다고 한다.

RPG 게임을 처음 시작해, 첫 캐릭터를 만들 때를 상기해보면 
캐릭터의 헤어스타일이나, 얼굴, 신체의 외형, 등등, 캐릭터의 다양한 부분을 모두 지정하고나면 하나의 복잡한 캐릭터가 만들어진다는 점에서 이러한 패턴이 용이하게 사용될 여지가 충분해보인다.

어쩌면 조금 복잡한 아이템을 생성할 때 사용될수도 있겠다.

게임 디아블로를 생각해보면,
아이템의 이름, 타입, 공격력, 방어력, 희귀도, 특수효과, 등등 동일한 종류의 아이템일지라도 개체마다 상세한 옵션이 다르며 
종류와 정도가 무작위로 지정되는 특수효과 옵션까지 존재하는 복잡한 아이템 시스템을 가지고 있다.

이렇게 복잡한 아이템을 생성하고 관리하는데도 ***BuilderPattern***이 용이하게 사용될 여지가 있어보인다.