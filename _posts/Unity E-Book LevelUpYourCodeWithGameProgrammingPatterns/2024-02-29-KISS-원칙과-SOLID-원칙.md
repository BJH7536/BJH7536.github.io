---
title:  KISS 원칙과 SOLID 원칙
date:   2024-02-29 +0900
categories: [Unity E-book LevelUpYourCodeWithGameProgrammingPatterns]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.

## KISS 원칙
> *복잡한건 최소화하고, 가능한 한 단순하게 유지하자!*

- 'Keep it Simple, Stupid'의 줄임말
- 시스템에서 불필요하게 복잡한 코드를 줄이고, 이해하기 쉽고, 유지보수가 용이한 설계가 목적.
- `단순함`과 `쉬움`은 다른 것.
- 과정이 쉽고 빠르다고해서, 무조건 단순한 결과물이 나오지는 않는다.
- 좋아보인다고해서 무조건 쓰지 말고, 필요하다는 생각이 들 때 패턴을 사용할 것.

## SOLID 원칙
- S/W 설계에서 지켜야할 다섯가지 핵심 원칙
- 유연하고 관리하기 쉬운 객체지향 설계를 추구하기 위해 알아야 한다.

### 단일 책임 (Single Responsibility)
    - 각 모듈, 클래스, 함수는 한가지 역할에만 책임을 지며, 오직 그 부분만을 캡슐화 해야 한다.
    - 대규모의 단일 클래스를 구축하는 대신, 많고 작은 프로젝트들을 조합, 조립하라.
    - 더 짧은 클래스와 메서드가 설명, 이해, 사용이 쉽다.
    - 이러한 개념은 Unity에서 한 GameObject에 다양한 Component들이 부착가능한 형식으로 구현되었다.

이러한 개념을 반영하지 않고, 다양한 책임이 혼합된 `UnrefactoredPlayer`클래스는 다음과 같다.

```cs
public class UnrefactoredPlayer : MonoBehaviour  
{  
    [SerializeField] private string inputAxisName;   
    [SerializeField] private float positionMultiplier;   
    private float yPosition;   
    private AudioSource bounceSfx;  
  
    private void Start() { bounceSfx = GetComponent<AudioSource>(); }  
    private void Update()  
    {
        float delta = Input.GetAxis(inputAxisName) * Time.deltaTime;
        yPosition = Mathf.Clamp(yPosition + delta, -1, 1);   
        transform.position = new Vector3(transform.position.x, yPosition * positionMultiplier, transform.position.z);  
    }
    private void OnTriggerEnter(Collider other) { bounceSfx.Play(); }
}

```

이러한 클래스를 도식화하면 다음과 같다. <br>
플레이어 내부에서 존재할 수 있는 다양한 세부적 책임을 한 곳에서 중앙 집중형으로 관리한다.

![Pasted image 20240223192751](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/d0d7ce4c-6389-425a-aae9-838bc280f458)


다음과 같은 방식으로 더 작은 클래스들로 나누는 것을 고려해보아라.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/0c9948b4-079e-4b41-bcaf-81c7d909c021)
```cs
[RequireComponent(typeof(PlayerAudio), typeof(PlayerInput), typeof(PlayerMovement))]  
public class Player : MonoBehaviour  
{  
    [SerializeField] private PlayerAudio playerAudio;   
    [SerializeField] private PlayerInput playerInput;   
    [SerializeField] private PlayerMovement playerMovement;  
  
    private void Start()  
    {        
        playerAudio = GetComponent<PlayerAudio>();
        playerInput = GetComponent<PlayerInput>();  
        playerMovement = GetComponent<PlayerMovement>();  
    }
}   
public class PlayerAudio : MonoBehaviour { ... } 
public class PlayerInput : MonoBehaviour { ... } 
public class PlayerMovement : MonoBehaviour { ... }
```
이렇게 하면 `Player`클래스는 여전히 다른 스크립트된 컴포넌트들을 관리할 수 있지만, 각 클래스는 오직 하나의 일만 수행한다.
이러한 설계는 시간이 지나면서 요구사항이 변할 때 코드에 접근하고, 수정하기 더 쉽게 만든다.

그러나 한편으로는, ***Single Responsibility Principle***을 상식적인 수준에서 균형을 맞추어야 한다.
단 하나의 메서드만을 위한 클래스를 만드는 것과 같이 극단적인 단순화는 피할 것.

다음을 염두에 둘 것.

##### **가독성**
    - 짧은 클래스는 읽기 더 쉽다.
    - 엄격한 규칙은 없지만, 200~300줄 정도를 한계로 설정해보아라
    - `짧다`의 정의를 스스로, 혹은 팀과 함께 의논해 결정해보아라.
    - 이 한계를 초과할 때, 더 작을 부분들로 리팩터링할 수 있을지 결정할 것.

##### **확장성**
    - 작은 클래스에서는 더 쉽게 상속할 수있다.
    - 의도하는 기능이 손상되는 것을 두려워하지 말고 수정하거나 교체할 것.

##### **재사용성**
    - 게임의 다른 부분에도 재사용할 수 있도록 클래스를 작고 모듈식으로 설계할 것.

---

### 개방 폐쇄 (Open-Closed)
    - 클래스는 확장에는 개방되어 있되, 수정에는 폐쇄되어야 한다.
    - 기존 코드를 수정하지 않고도 새로운 행동을 생성할 수 있도록 클래스를 구조화해야 한다.
  
다음은 도형의 면적을 계산하는 고전적인 예이다. <br>
사각형과 원의 면적을 반환하는 메서드를 가진 `AreaCalculator`라는 클래스가 있고,
면적을 계산하기 위해 `Rectangle`클래스는 너비와 높이가, `Circle`클래스는 반지름이 필요하다.

```cs
public class AreaCalculator  
{  
    public float GetRectangleArea(Rectangle rectangle)  
    {        
        return rectangle.width * rectangle.height;  
    }  
    public float GetCircleArea(Circle circle)  
    {        
        return circle.radius * circle.radius * Mathf.PI;  
    }
}

public class Rectangle 
{   
    public float width;  
    public float height;  
}  
  
public class Circle  
{  
    public float radius;  
}
```

만약에 `AreaCalculator`클래스에 도형을 몇가지 더 추가하고 싶다면, 새 도형마다 새 메서드를 만들어야 할 것이다. 도형이 매우 많아진다면, 그에 따라 `AreaCalculator`클래스도 매우 커질 것이다.

이에 대한 해결책으로 `Shape`라는 기본 클래스를 만들고 도형들을 처리하는 하나의 메서드를 생성할 수 있다. 그러나, 이러한 방법은 다수의 `if` 문이 필요하며, 확장되기 어렵다.

프로그램이 확장에는 열려 있도록 (새로운 형태를 사용할 수 있는 능력) 하면서도 원래 코드 
(`AreaCalculator`의 내부)를 수정하지 않도록 해야하나, 현재의 `AreaCalculator`는 이를 위반한다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/59d4034f-ee02-46a1-bff1-d222fbff072d)

대신, 추상 `Shape` 클래스를 고려해보아라.

```cs
public abstract class Shape 
{ 
    public abstract float CalculateArea(); 
}
```

이는 `CalculateArea()` 추상 함수를 포함한다. 이를 상속하는 `Rectangle`, `Circle`클래스를 만들면 각각의 도형은 스스로의 면적을 계산하고 다음의 결과를 반환할 것이다.

```cs
public class Rectangle : Shape   
{   
    public float width;  
    public float height;  
  
    public override float CalculateArea()  
    {        
        return width * height;  
    } 
} 
public class Circle : Shape   
{   
    public float radius;  
  
    public override float CalculateArea()  
    {        
        return radius * radius * Mathf.PI;  
    }
}
```

그러면 `AreaCalculator`는 다음과 같이 단순화될 수 있다.

```cs
public class AreaCalculator
{
    public float GetArea(Shape shape)
    {
        return shape.CalculateArea();
    }
}
```

이제 수정된 `AreaCalculator`클래스는 `Shape` 추상 클래스를 상속하는 어떤 클래스라도 그 면적을 구할 수 있다. 이를 통해 `AreaCalculator`의 기능을 원본  소스 코드를 변경하지 않고도 확장 할 수 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/07942456-a776-404a-b6bf-2524b0bd64db)

새로운 다각형이 필요할 때마다, `Shape`에서 상속받는 새로운 클래스를 정의하기만 하면 된다.
각각의 하위 클래스에서 정의된 도형은 `CalculateArea()`메서드를 오버라이딩하여 올바른 면적을 반환한다. 

이러한 설계는 디버깅을 더 쉽게 만든다. 새로운 도형이 오류를 유발하더라도, `AreaCalculator`를 수정할 필요는 없어진다. 

Unity에서 새로운 클래스를 생성할 때 인터페이스와 추상화를 이용하라. 이는 나중에 확장하기 어려운 `switch문`, `if문`의 사용을 피하는데 도움이 된다. 이러한 방법에 익숙해지면, 장기적으로 새로운 클래스를 추가하는 것이 더 간단해진다.

---

### 리스코프 치환 (Liskov Substitution)
    - 파생된 클래스는 상속 사용 시, 기본 클래스를 대체할 수 있어야 한다.
    - 그러나, 불필요한 복잡성으로 이어지지 않아야 한다.

게임에서 `Vehicle`이라는 이름의 클래스를 필요로한다면, 이는 이후에 생성할 차량 하위 클래스의 기반 클래스가 될 것이다. 예를 들어, `Car` 혹은 `Truck`과 같은 클래스가 될 수 있을 것이다.

![Pasted image 20240223224000](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/e8479dbe-a1e7-440e-bf1e-772c04d7a421)
기반 클래스(`Car`)가 사용될 수 있는 모든 곳에, 애플리케이션이 깨지지 않고 그 하위 클래스 (`Car` 혹은 `Truck`)가 사용될 수 있다.

`Vehicle`클래스는 다음과 같이 구성할 수 있다.

```cs
public class Vehicle
{
    public float speed = 100;
    public Vector3 direction;

    public void GoForward() { ... }
    public void Reverse() { ... }
    public void TurnRight() { ... }
    public void TurnLeft() { ... }
}
```

턴제 기반의, 차량이 보드 근처를 이동하는 게임을 개발중이라고 가정해보자.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/6de023e4-7697-4f2e-a0ee-e5df478a9297)

차량을 정해진 경로를 따라 조종하기 위해, `Navigator`라는 다른 클래스를 가질 수 있다.

```cs
public class Navigator
{
    public void Move(Vehicle vehicle)
    {
        vehicle.GoForward();
        vehicle.TurnLeft();
        vehicle.GoForward();
        vehicle.TurnRight();
        vehicle.GoForward();
    }
}
```

이 클래스를 사용하면, 어떤 차량이든 `Navigator`의 `Move()` 메서드로 전달할 수 있으며, 이는 `Car` 와 `Truck`에 잘 동작할 것이다. 그러나 만약에, `Train` 이라는 클래스를 구현하고자한다면 어떻게 될 까?

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/f6d47f45-2140-4747-a94a-db8d3b08e4bf)

기차는 트랙을 벗어날 수 없기 때문에, `TurnLeft()` 와 `TurnRight()` 메서드는 `Train` 클래스에 동작하지 않을 것이다. 만약 `Navigator`의 `Move()` 메서드에 `Train`을 전달한다면, 구현되지 않은 `Exception`을 던지거나, 아무것도 하지 않을 것이다.

Subtype으로 Type을 대체할 수 없다면, 이는 ***리스코프 치환 원칙 (Liskov Substitution Principle)***을 위반하게 된다.

`Train`이 `Vehicle`의 Subtype이기 때문에, `Vehicle` 클래스가 있을 수 있는 모든 곳에 대체될 수 있을 것이라 예상할 수 있다. 이와 다르게 행동하면, 코드가 예측할 수 없는 방식으로 동작할 수 있다.

다음은 리스코프 치환 법칙을 더욱 철저히 지키기 위한 몇가지 팁을 고려할 것.

#### 서브클래스를 만들때 기능의 삭제가 일어난다면, 이는 LSP를 위반할 가능성이 높다.
    `NotImplementedException`은 이 원칙을 위반했다는 분명한 신호. 
    메서드를 비워두는 것도 마찬가지. 
    서브클래스나 베이스 클래스처럼 동작하지 않는다면, LSP를 따르고 있지 않은 것. 
    명시적인 오류나 예외가 없더라도 마찬가지.

#### 추상화를 단순하게 유지할 것
      베이스 클래스에 더 많은 로직을 넣을수록, LSP를 위반할 가능성이 높아진다. 
      베이스 클래스는 파생 서브 클래스들의 공통 기능만을 표현해야 한다.

#### 서브 클래스는 베이스 클래스와 동일한 public 멤버를 가져야 한다.
      이 멤버들은 호출될 때 동일한 시그니처와 행동을 가져야 한다.

#### 클래스 계층을 확립하기 전에 API를 고려할 것
      모두 차량으로 생각할지라도, 
      `Car`와 `Train`이 별도의 부모 클래스로부터 상속받는 것이 더 자연스러울 수 있다. 
      현실의 분류가 항상 클래스 계층으로 번역되는 것은 아님.

#### 상속보다 구성을 선호하라
      Inheritance을 통해 기능을 전달하려 하기보다, 
      특정 행동을 캡슐화하기위한 인터페이스나 별도의 클래스를 생성하라. 
      그런 다음 다양한 기능의 "Composition"을 혼합하여 매칭함으로써 구축하라.

![Pasted image 20240223230242](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/17be30fc-7dad-4555-a295-94cff0bde827)


이러한 설계를 수정하기 위해, 원래의 `Vehicle`타입을 폐기하고, 대부분의 기능을 인터페이스로 이동시키자.

```cs
public Interface ITurnable
{
    public void TurnRight();
    public void TurnLeft();
}

public Interface IMovable
{
    public void GoForward();
    public void Reverse();
}

```

`RoadVehicle`과 `RailVehicle`타입을 만들어 LSP를 더 밀접하게 따를 수 있다. `Car`와 `Train`은 각각 해당하는 기반 클래스로부터 상속받게 된다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/32f6dbe1-0ab3-4ad8-ac42-c3b8b6c50292)

이를 코드로 나타내면 

```cs
public class RoadVehicle : IMovable, ITurnable  
{  
    public float speed = 100f;  
    public float turnSpeed = 5f;  
    public virtual void GoForward() { ... }   
    public virtual void Reverse() { ... }   
    public virtual void TurnLeft() { ... }   
    public virtual void TurnRight() { ... }  
} 
public class RailVehicle : IMovable   
{   
    public float speed = 100;   
    public virtual void GoForward() { ... }   
    public virtual void Reverse() { ... } 
} 
public class Car : RoadVehicle { ... } 
public class Train : RailVehicle { ... }
```

이러한 기능은 상속보다 인터페이스를 통해 제공된다. `Car`와 `Train`은 더이상 동일한 기반 클래스를 공유하지 않으며, 이는 LSP를 만족시킨다. 비록 `RoadVehicle`과 `RailVehicle`을 동일한 기반 클래스로부터 파생시킬 수는 있지만, 이 경우에는 그럴 필요가 거의 없다. 

이러한 사고방식은 실제 세계에 대한 특정한 가정을 갖고 있기 때문에, 직관적이지 않을 수 있다. S/W 개발에서 이는 ***원-타원 문제 (Circle-Ellipse Problem)*** 라고 불린다. 
실제의 모든 "is a" 관계가 상속으로 구현되지는 않는다. S/W 설계는 실제 세계와 동일하지 않을 수 있다. 

 LSP를 따라 상속의 사용방식을 제한하여, 코드 베이스를 확장 가능하고 유연하게 유지하라.

---

### 인터페이스 분리 (Interface Segregation)
    - 클라이언트가 사용되지 않는 메서드에 의존하도록 하면 안된다. (=큰 인터페이스를 지양하라.)
    - 클라이언트는 오직 필요한 메서드만 구현해야 한다.

***단일 책임 원칙 (Single-Responsibility Principle)*** 과 동일한 idea를 따라, 클래스와 메서드를 짧게 유지하라. 이는 인터페이스들을 컴팩트하고 집중적으로 유지함으로써 최대한의 유연성을 제공한다.

다양한 플레이어 유닛이 있는 전략 게임을 만들고 있다고 가정하자. 각 유닛은 서로다른 체력, 속도와 같은 스텟을 갖고있다. 모든 유닛이 비슷한 기능을 구현하도록 보장하기 위한 인터페이스를 만들고 싶을 수 있다. 

```cs
public interface IUnitStats  
{  
    public float Health { get; set; }   
    public int Defense { get; set; }   
      
    public void Die();   
    public void TakeDamage();   
    public void RestoreHealth();
      
    public float MoveSpeed { get; set; }   
    public float Acceleration { get; set; }   
      
    public void GoForward();   
    public void Reverse();   
    public void TurnLeft();   
    public void TurnRight();   
      
    public int Strength { get; set; }   
    public int Dexterity { get; set; }   
    public int Endurance { get; set; }  
}
```

파괴 가능한 배럴, 상자와 같이 파괴 간으한 소품을 만들고 싶다고 가정해보자. 이 소품은 움직이지 않음에도 불구하고 체력의 개념이 필요할 것이다. 상자나 배럴은 게임 내 다른 유닛들과 연관된 많은 능력을 가지고 있지 않을 것이다. 

하나의 인터페이스를 만들어 파괴 가능한 소품에 너무 많은 메소드를 부여하기보다는, 여러개의 작은 인터페이스들로 분할하는 것이 좋다. 그러면 이를 구현하는 클래스는 필요한 것들만 선택하여 조합할 수 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/fab7e310-5d48-4f33-8e09-19aedc99bbfd)

```cs
public interface IMovable  
{  
    public float MoveSpeed { get; set; }   
    public float Acceleration { get; set; } 
      
    public void GoForward();  
    public void Reverse();  
    public void TurnLeft();  
    public void TurnRight();  
    }  
  
public interface IDamageable  
{  
    public float Health { get; set; }  
    public int Defense { get; set; }  
    
    public void Die();  
    public void TakeDamage();  
    public void RestoreHealth();  
}   

public interface IUnitStats {   
    public int Strength { get; set; }  
    public int Dexterity { get; set; }  
    public int Endurance { get; set; }  
}
```

폭발하는 배럴에는 `IExplodable` 인터페이스를 추가할수 있다.

```cs
public interface IExplodable 
{ 
    public float Mass { get; set; }
    public float ExplosiveForce { get; set; }
    public float FuseDelay { get; set; }
    public void Explode();
}
```

하나의 클래스는 하나 이상의 인터페이스들을 구현할 수 있기 때문에, 적 유닛을 `IDamageable`, `IMoveable`, `IUnitStates`로부터 구성할 수 있다.

폭발하는 배럴은 다른 인터페이스의 불필요한 오버헤드 없이 `IDamageable`과 `IExplodable`을 사용할 수 있다.

```cs
public class ExplodingBarrel : MonoBehaviour, IDamageable, IExplodable 
{ ... } 
public class EnemyUnit : MonoBehaviour, IDamageable, IMovable, IUnit- Stats 
{ ... }
```

이 접근법은 LSP의 예제와 유사하게 [Composition over Inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)을 선호한다. 

***인터페이스 분리 원칙 (Interface Segregation Principle)*** 은 시스템을 분리하여 수정 및 재배포하기 쉽게 만든다.

---
### 의존성 역전 (Dependency Inversion)
    - 상위 수준의 모듈은 하위 수준 모듈로부터 어떤것도 직접 임포트하면 안된다.
    - 두 모듈 모두 추상화에 종속되어야 한다.

한 클래스가 다른 클래스와 관계를 가질 때, 이는 **의존성 혹은 커플링**을 갖는다. S/W 설계에서 각 의존성은 어느정도의 위험성을 수반한다.

한 클래스가 다른 클래스가 어떻게 작동하는지에 대해 너무 많이 알게되면, 한 클래스를 수정하는것은 두번째 클래스에게 손상을 줄 수 있다, 혹은 그 반대거나. 높은 수준의 커플링은 깨끗하지 못한 코드 관행으로 여겨진다. 애플리케이션 한 부분의 오류는 많은 부분으로 눈덩이처럼 커질 수 있다.

이상적으로는, 클래스간의 의존성을 최소화하는것을 목표로한다. 각 클래스는 외부와의 연결에 의존하기보다는, 내부 부품이 조화롭게 함께 작동할 필요가 있다. 객체가 내부적이거나 비공개 로직에 따라 기능할 때, 그 객체는 결합도가 높다고 여겨진다.

최선의 시나리오에서는, ***느슨한 결합과 높은 응집력 (loose coupling and high cohesion)*** 을 목표로한다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/fe80d88c-e2c8-498f-b1ce-06a077e0cd6d)

게임 애플리케이션을 수정하고 확장할 수 있어야 한다. 만약 그것이 취약하고 수정에 저항적이라면, 현재 어떻게 구조화되어 있는지 조사해야 한다. 

***의존성 역전 원칙 (Dependency Inversion Principle)*** 은 클래스 간의 이러한 긴밀한 결합을 줄이는 데 도움이 될 수 있다. 애플리케이션에서 클래스와 시스템을 구축할 때, 일부는 자연스럽게 **고수준(High-Level)** 이고 일부는 **저수준 (Low-Level)** 이다. 고수준 클래스는 무언가를 완성하기 위해 저수준 클래스에 의존한다. SOLID 원칙은 이러한 상황을 바꾸도록 우리에게 권장한다.

캐릭터가 레벨을 탐험하고 문을 열도록 트리거하는 게임을 만들고 있다고 가정해보자. 이런 상화에서 `Switch`와`Door`라는 클래스를 만들 수 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/6a24cf97-3c3a-4ee3-88ed-96c3268afadf)

고수준에서, 캐릭터가 특정 위치로 이동하고 무언가 일어나길 원할 수 있다. `Switch` 클래스가 그 일을 담당할 것이다.

저수준에서, 문을 실제로 열고 닫는 방법의 구현을 포함하는 다른 클래스인 `Door`이 있다. 단순화를 위해, 문을 열고 닫는 로직을 대신하는 `Debug.Log`문을 활용함.

```cs
public class Switch : MonoBehaviour 
{ 
    public Door door; 
    public bool isActivated; 
    public void Toggle()
    { 
        if (isActivated) 
        { 
            isActivated = false; 
            door.Close(); 
        } 
        else 
        { 
            isActivated = true; 
            door.Open(); 
        } 
    } 
} 
public class Door : MonoBehaviour 
{ 
    public void Open() 
    { 
        Debug.Log("The door is open."); 
    } 
    public void Close() 
    { 
        Debug.Log("The door is closed.");
    } 
}
```

`Switch`는 문을 열고 닫기 위해 `Toggle` 메소드를 호출할 수 있다. 이는 작동하지만, 문제는 `Door`로부터 `Switch`로 직접 의존성이 설정된다는 점이다. 예를 들어, `Switch`로직이 문 이외에도 빛을 활성화하거나 거대 로봇을 작동시켜야 하는 경우라면 어떨까?

`Switch` 클래스에 메소드를 추가할 수 있겠지만, 이는 ***개방-폐쇄 원칙 (Open-Closed Principle)*** 을 위반하게 된다. 기능을 확장하고 싶을 때 마다, 원본 코드를 수정해야 한다. 

다시한번 추상화가 구원의 손길을 제공한다. 
클래스 사이에 `ISwitchable`이라는 인터페이스를 끼워넣을 수 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/6c6f3c67-f3a4-4c89-9cac-3ee37bb0d722)

`ISwitchable` 인터페이스는 해당 객체가 활성화 상태인지 아닌지를 알 수 있는 public property와 함께, 그것을 활성화하고 비활성화하는 몇가지 메소드만을 필요로 한다.

```cs
public interface ISwitchable
{
    public bool IsActivate { get; }

    public void Activate();
    public void Deactivate();
}

```

그리고 `Switch`는 `Door`에 직접적으로 의존하는 대신, `ISwitchable` 클라이언트에게 의존하게 된다.

```cs
public class Switch : MonoBehaviour 
{ 
    public ISwitchable client; 
    public void Toggle() 
    { 
        if (client.IsActive) 
        { 
            client.Deactivate(); 
        } 
        else 
        { 
            client.Activate(); 
        } 
    } 
}
```

반면에, `Door` 클래스는 `ISwitchable`을 구현하기 위해 수정해야 한다

```cs
public class Door : MonoBehaviour, ISwitchable 
{ 
    private bool isActive; 
    public bool IsActive => isActive; 
    public void Activate() 
    { 
        isActive = true; 
        Debug.Log("The door is open."); 
    } 
    public void Deactivate() 
    { 
        isActive = false; 
        Debug.Log("The door is closed."); 
    } 
}
```

이제, 의존성을 역전시켰다. 인터페이스는 스위치를 문에만 직접 연결하는 대신, 그 사이에 추상화를 생성한다. `Switch`는 더이상 문에 특정된 메소드에 직접적으로 의존하지 않는다. 대신, 이는 `ISwitchable`의 `Activate`와 `Deactivate`를 사용한다.

이 작지만 중요한 변경은 재사용성을 촉진한다. `Switch`가 `Door`에 대해서만 작동했다면, 이제는 `ISwitchable`을 구현하는 모든 것과 작동한다.

이는 `Switch`가 활성화 할 수 있는 더 많은 클래스를 만들 수 있게 한다. 고수준의 `Switch`는 다락문이든, 레이저빔이든 작동할 것이다. 단지 `ISwitchable`을 구현하는 호환 가능한 클라이언트만 필요하다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/c9906840-5268-4ec8-ac8f-bdafb31cd9e5)

SOLID의 다른 원칙들처럼, ***의존성 역전 원칙(Dependency Inversion Principle)*** 도 여러분이 일반적으로 클래스 간의 관계를 어떻게 설정하는지 검토하도록 요구한다. 느슨한 결합을 통해 프로젝트를 편리하게 확장할 수 있다. 

---

## Interfaces vs Abstract Classes

***"Composition over Inheritance"*** 의 철학을 따르며, 이 가이드에서 많은 예제들이 인터페이스를 사용한다. <br>
하지만, 추상 클래스를 사용해도 많은 디자인 원칙과 패턴을 따를 수 있다. 둘다 C#에서 추상화를 달성하는 유효한 방법이다. 어떤 것을 사용할지는 상황에 따른 필요성에 달려있다.

### 추상 클래스 Abstract Class

`abstract` 키워드를 사용해 기본 클래스를 정의할 수 있으므로, 상속을 통해 하위클래스에 공통 기능을 전달할 수 있다.

추상 클래스는 직접 인스턴스화할 수 없다. 대신 구체적인 클래스를 파생시켜야 한다.

앞서 언급된 예제에서 추상 클래스를 사용하면 동일한 의존성 역전을 달성할 수 있지만, 접근 방식이 다르다. 따라서 인터페이스를 사용하는 대신, `Switchable`이라는 추상 클래스에서 구체적인 클래스를 파생시킨다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/d665a656-99ce-4aaa-86fd-69cacf43c368)

상속은 "is a" 관계를 정의한다. 위에서 언급된 다이어그램에 표시된 모든 "switchable"한 것들은 한번만 켜고 끌 수 있다.

추상 클래스의 장점은 필드와 상수뿐만 아니라 정적 멤버도 가질 수 있다. 또한 `protected`와 `private`과 같은 더 제한적인 접근 제어자를 적용할 수 있다. 인터페이스와 달리, 추상 클래스는 구체적인 클래스들 사이에 핵심기능을 공유할 수 있도록 하는 로직을 구현할 수 있게 한다.

상속은 두 개의 다른 기본 클래스의 특성을 가진 파생 클래스를 생성하고 싶을 때까지 잘 작동한다. C#에서는 두 개 이상의 기본 클래스로부터 상속받을 수 없다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/018ba326-dfd0-4fa8-8e02-a779becf21b3)

게임 내 모든 로봇들을 위한 또 다른 추상 클래스가 있다면, 어떤 기본 클래스로부터 파생시킬 지 결정하기 어려워질 수 있다. Robot 기본 클래스를 사용할지, 아니면 Switchable 기본 클래스를 사용할지 선택해야 하는 상황이 발생한다.

### 인터페이스 Interface

***인터페이스 분리 원칙(Interface Segregation Principle)*** 에서 볼 수 있듯, 인터페이스는 상속의 패러다임에 깔끔하게 맞지 않는 것이 있을 때 더 많은 유연성을 제공한다. "has a" 관계를 통해 더 쉽게 선택하고 결정할 수 있다.

하지만, 인터페이스는 그 멤버들의 선언만을 포함한다. 인터페이스를 실제로 구현하는 클래스는 특정 로직을 구체화하는 책임이 있다. 

코드를 공유하고 싶은 기본 기능을 정의할 때는 추상 클래스를 사용하고, 유연성이 필요한 주변 능력을 정의할 때는 인터페이스를 사용하라.

| 추상 클래스                  | 인터페이스                                   |
| ----------------------- | --------------------------------------- |
| 완전히, 혹은 부분적으로 메소드들을 구현. | 메소드들을 선언만할 뿐, 구현은 불가.                   |
| 변수와 필드를 선언하고 사용할 수 있다.  | 메소드와 프로퍼티만 선언 가능. (필드는 불가)              |
| 정적 멤버를 가질 수 있다.         | 정적 멤버의 선언/사용 불가                         |
| 생성자를 사용할 수 있다.          | 생성자 사용 불가                               |
| 모든 접근 제어자를 사용할 수 있다.    | 접근 제어자 사용 불가. <br>(모든 멤버는 암시적으로 public) |

---
## 요약

- **단일 책임 원칙(Single Responsibility)**: 클래스가 하나의 일만 하도록 하고 변경해야 할 이유가 하나만 있도록 한다.
- **개방-폐쇄 원칙(Open-Closed)**: 클래스의 기능을 확장할 수 있어야 하지만, 이미 작동하는 방식을 변경하지는 않아야 한다.
- **리스코프 치환 원칙(Liskov Substitution)**: 하위 클래스는 그들의 기본 클래스에 대해 대체 가능해야 한다.
- **인터페이스 분리 원칙(Interface Segregation)**: 인터페이스를 간결하게 유지하고, 몇 개의 메소드만 포함시켜라. 클라이언트는 필요한 것만 구현해야 한다.
- **의존성 역전 원칙(Dependency Inversion)**: 추상화에 의존해라. 구체적인 클래스에서 다른 구체적인 클래스로 직접 의존하지 마라.


## 참고한 자료
[Unity_E-Book_LevelUpYourCodeWithGameProgrammingPatterns](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)