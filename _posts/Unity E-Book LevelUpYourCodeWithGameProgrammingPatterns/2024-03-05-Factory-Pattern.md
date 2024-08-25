---
title:  Factory Pattern
date:   2024-03-05 +0900
categories: [Unity E-book LevelUpYourCodeWithGameProgrammingPatterns]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/ad255a75-820e-40d6-9b1a-c2ffa988e72c)

다른 오브젝트들을 만드는 특별한 오브젝트가 있는 것이 가끔은 도움이 될 때가 있다. 많은 게임들은 게임 플레이 과정에서 다양한 것들을 생성하고, 정말로 필요해지기 전까지는 런타임에 정말 무엇이 필요한지 종종 모를 수 있다.

 팩토리 패턴은 이 목적을 위한 특별한 객체, 즉 '팩토리'를 지정한다.  한 단계에서, 이 패턴은 '생산품'을 생성하는 데 관련된 많은 세부 사항을 캡슐화한다. 즉각적인 이점은 코드를 정리하는 것이다.

그러나, 각 제품이 공통 인터페이스나 기본 클래스를 따른다면, 이를 한 단계 더 발전시켜 그것의 생성 로직을 더 많이 포함시키고, 이를 팩토리 자체로부터 숨길 수 있다. 이렇게 함으로써 새로운 객체를 생성하는 것이 더 확장 가능해진다.

또한 팩토리를 서브 클래스화하여 특정 제품 전용의 팩토리를 만들 수 있다. 이렇게하면 런타임에 적, 장애물 또는 기타 어떤 것들도 생성하는 데 도움이 된다.

## 예제 : 단순한 팩토리
게임 레벨에 아이템을 인스턴스화하기 위한 팩토리 패턴을 만들고 싶다고 상상해 보자. 
프리팹(Prefabs)을 사용하여 `GameObject`들을 생성할 수 있지만, 각 인스턴스를 생성할 때 몇 가지 사용자 정의 동작을 실행하고 싶을 수도 있다. 

이 로직을 유지하기 위해 `if`문이나 `switch`문을 사용하는 대신, `IProduct`라는 인터페이스와 `Factory`라는 추상 클래스를 생성하라.

```cs
public interface IProduct 
{ 
    public string ProductName { get; set; } 
    public void Initialize();
} 
public abstract class Factory : MonoBehaviour 
{ 
    public abstract IProduct GetProduct(Vector3 position); 
    
    // shared method with all factories 
    ...
}
```

생산품은 그들의 메소드에 대해 특정 템플릿을 따라야 하지만, 그 외에는 어떠한 기능도 공유하지 않는다. 따라서, `IProduct` 인터페이스를 정의한다. 

팩토리는 일부 공통 기능을 공유해야 할 수도 있으므로, 이 예제는 추상 클래스를 사용한다. 
서브클래스를 사용할 때 SOLID 원칙 중 리스코프 치환 원칙을 염두에 두어라. 

다음과 같은 구조가 그 결과일 수 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/72887ac5-5d27-4c8f-b0e2-db8ee7dc8041)

`IProduct`는 생산품들 간의 공통점을 정의한다. 이 경우에는, 단순히 `ProductName` 프로퍼티와 `Initialize`에서 동작하는 로직이 있겠다. <br>
그러면 `IProduct` 인터페이스를 따르는 한 (ProductA, ProductB 등) 필요한 만큼 많은 제품을 정의할 수 있다.

기본 클래스인 `Factory`는 `IProduct`를 반환하는 `GetProduct`메소드를 가지고 있다. <br> 
이는 추상 클래스이기 때문에, `Factory`의 인스턴스를 직접 만들 수는 없다. 대신, 구체적인 서브클래스 (`ConcreteFactoryA` 와 `ConcreteFactoryB`)를 파생시켜, 실제로 다양한 생산품들을 관리할 수 있다. 

이 예제에서 `GetProduct`는 `Vector3` 위치를 인수로 받아 특정 위치에 프리팹 GameObject를 더 쉽게 인스턴스화할 수 있다. 각 구체적인 팩토리의 필드에는 해당하는 템플릿 프리팹도 저장된다. 

여기 `ProductA`와 `ConcreteFactoryA`의 예시가 있다.

```cs
public class ProductA : MonoBehaviour, IProduct 
{ 
    [SerializeField] private string productName = "ProductA";
    public string ProductName { get => productName; set => productName = value ; } 
    private ParticleSystem particleSystem; 
    
    public void Initialize() 
    { 
        // any unique logic to this product 
        gameObject.name = productName; 
        particleSystem = GetComponentInChildren<ParticleSystem>();
        particleSystem?.Stop(); 
        particleSystem?.Play(); 
    } 
} 
public class ConcreteFactoryA : Factory 
{ 
    [SerializeField] private ProductA productPrefab; 
    
    public override IProduct GetProduct(Vector3 position) 
    { 
        // create a Prefab instance and get the product component 
        GameObject instance = Instantiate(productPrefab.gameObject, position, Quaternion.identity); 
        ProductA newProduct = instance.GetComponent<ProductA>(); 
        
        // each product contains its own logic 
        newProduct.Initialize(); 
        
        return newProduct; 
    } 
}

```

여기, 생산품 클래스들이 `MonoBehaviour`를 상속받고 `IProduct` 인터페이스를 구현하게 함으로써, 팩토리 내에서 프리팹의 장점을 살릴 수 있다.

각 생산품이 `Initialize` 메소드의 자체 버전을 가질 수 있다는 점에 주목하라. <br>
예제 `ProductA` 프리팹은 `ParticleSystem`을 포함하고 있으며, `ConcreteFactoryA`가 복사본을 인스턴스화할 때 재생된다. 팩토리 자체는 Particle을 트리거하는 구체적인 로직을 포함하고 있지 않다. 단지 모든 생산품에 공통인 `Initialize` 메소드만을 호출한다.

샘플 프로젝트를 탐색하여 `ClickToCreate` 컴포넌트가 팩토리 간에 전환하여 서로 다른 행동을 가진 `ProductA`와 `ProductB`를 생성하는 방법을 확인하라. `ProductB`는 생성될 때 소리를 재생하는 반면, `ProductA`는 입자 효과를 발생시킨다.

## 장단점

팩토리 패턴은 많은 제품을 설정할 때 가장 큰 이점을 제공한다. 애플리케이션에서 새로운 제품 유형을 정의해도 기존 제품을 변경하거나 이전 코드를 수정할 필요가 없다.

각 제품의 내부 로직을 자체 클래스로 분리함으로써 팩토리 코드를 상대적으로 짧게 유지할 수 있다. 각 팩토리는 기본적인 세부 사항을 알 필요 없이 각 제품에 대해 `Initialize`를 호출하는 방법만 알고 있다.

단점은 패턴을 구현하기 위해 다수의 클래스와 서브클래스를 생성한다는 것. 다른 패턴들처럼, 이것은 약간의 오버헤드를 도입하는데, 이는 제품의 다양성이 크지 않은 경우 불필요할 수 있다.

## 개선 사항

팩토리의 구현은 여기에서 보여진 것과는 크게 다를 수 있다. 
자신만의 팩토리 패턴을 구축할 때 다음과 같은 조정을 고려하라:

### 생산품 검색을 위한 딕셔너리 사용:
	생산품을 딕셔너리의 키-값 쌍으로 저장하고자 할 수 있다. 
	고유한 문자열 식별자(예: 이름 또는 일부 ID)를 키로 사용하고 타입을 값으로 사용하라. 
	이는 제품 및/또는 해당 팩토리를 검색하는 것을 더 편리하게 만들 수 있다.

### 팩토리(또는 팩토리 매니저)를 정적(static)으로 만들기:
	이는 사용하기 쉽게 만들지만 추가 설정이 필요하다. 
	정적 클래스는 인스펙터에 나타나지 않으므로, 제품 컬렉션도 정적으로 만들어야 한다.

### GameObject와 MonoBehaviour가 아닌 것에 적용하기:
	프리팹이나 다른 Unity 특정 컴포넌트에만 제한되지 말 것. 
	팩토리 패턴은 어떤 C# 객체와도 작동할 수 있다.

### 오브젝트 풀 패턴과 결합하기:
	팩토리는 반드시 새 객체를 인스턴스화하거나 생성할 필요는 없다. 
	계층구조에서 기존 객체를 검색할 수도 있다. 
	한 번에 많은 객체를 인스턴스화해야 하는 경우(예: 무기에서 발사체), 
	더 최적화된 메모리 관리를 위해 오브젝트 풀 패턴을 사용하라.

팩토리는 필요에 따라 어떤 게임플레이 요소든 생성할 수 있다. 그러나 제품 생성이 종종 그들의 유일한 목적은 아니라는 점에 유의하라. 팩토리 패턴을 더 큰 작업의 일부로 사용할 수도 있다(예: 대화 상자의 UI 요소 설정 또는 게임 레벨의 부분 설정).


## 참고한 자료
[Unity_E-Book_LevelUpYourCodeWithGameProgrammingPatterns](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)