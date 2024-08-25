---
title:  Singleton Pattern
date:   2024-03-05 +1000
categories: [Unity E-book LevelUpYourCodeWithGameProgrammingPatterns]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.

***싱글톤 (Singleton)*** 은 종종 나쁜 평판을 받는다. Unity 개발에 입문하는 경우, 싱글톤은 아마도 처음으로 인식할 수 있는 패턴 중 하나일 것이다. 또한 가장 비난받는 패턴 중 하나이다.

원래의 ***Gane of Four***에 따르면, 싱글톤 패턴은

- 클래스가 자기 자신의 인스턴스를 단 하나만 인스턴스화할 수 있도록 보장한다.
- 단일 인스턴스에 쉽게 전역적으로 접근할 수 있게 한다.

전체 Scene에서 동작을 조정해야하는 정확히 하나의 객체가 필요한 경우 이는 유용하다. 예를 들어, 주요 게임 루프를 지시하는 Scene내에서 정확히 하나의 게임 매니저가 필요할 수있다. 또한, 한 번에 하나의 파일 매니저만이 파일 시스템에 쓰기를 원할 것이다. 이와 같은 중앙, 매니저 수준의 객체들은 ***싱글톤 패턴 (Singleton Pattern)*** 에 좋은 후보가 될 수 있다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/c9b18904-421c-4cc5-b6de-d1df550306af)

*"게임 프로그래밍 패턴(Game Programming Patterns)"* 에서는 싱글톤이 해가 더 많다고 하며, 이를 안티 패턴으로 분류한다. 이러한 나쁜 평판은 패턴의 사용 용이성이 남용으로 이어지기 때문이다. 개발자들은 부적절한 상황에서 싱글톤을 적용하며, 불필요한 전역상태나 의존성을 도입한다.

Unity에서 싱글톤을 구축하는 방법을 살펴보고, 그 장단점을 평가해보자. 그러면 이를 애플리케이션에 통합할 가치가 있는지 여부를 결정할 수있다.

## 예제 : 단순한 싱글톤

가장 단순한 싱글톤들 중 하나는 이렇게 생길 수 있다.

```cs
using UnityEngine; 

public class SimpleSingleton : MonoBehaviour 
{ 
    public static SimpleSingleton Instance; 
    private void Awake() 
    { 
        if (Instance == null)
        { 
            Instance = this;
        } 
        else 
        { 
            Destroy(gameObject);
        }
    }
}
```

`public static Instance`는 Scene에서 싱글톤의 유일한 인스턴스를 보유하게 된다.

`Awake()`메소드에서는 이미 설정되어있는지 확인한다. `Instance`가 현재 `null`이라면,  `Instance`는 이 특정 객체로 설정된다. 이 객체는 Scene에서 첫번째 싱글톤이어야한다.

그렇지 않다면 이 인스턴스는 중복된 것이며, Scene에서 이런 컴포넌트가 단 하나만 존재하도록 보장하기 위해 `Destroy(gameObject)`를 호출한다.

런타임에 hierarchy 에서 하나 이상의 `GameObject`에게 이 스크립트를 부착하면, `Awake()`의 로직은 첫번째 객체를 유지하고 나머지를 모두 버릴 것이다.

`Instance` 필드는 `public`이며 `static`이다. 어떤 컴포넌트도 Scene 내 어디서나 유일한 싱글톤에 전역적으로 접근할 수 있다.

## Persistence and Lazy instantiation 지속성과 지연 초기화

`SimpleSingleton`은 작성된 대로 작동합니다. 그러나 두 가지 문제점이 있다:

- 새 Scene을 로딩하면 `GameObject`가 파괴된다.
- 사용하기 전에 싱글톤을 hierarchy 에서 설정해야 한다.

싱글톤이 종종 어디에나 존재하는 매니저 스크립트로 작용하기 때문에, `DontDestroyOnLoad`를 사용하여 지속성을 부여하는 것이 유리할 수 있다.

또한, 처음 필요할 때 자동으로 싱글톤을 구축하기 위해 [지연 초기화(Lazy Instantiation)](https://en.wikipedia.org/wiki/Lazy_initialization) 를 사용할 수 있다. `GameObject`를 생성하고 적절한 싱글톤 컴포넌트를 추가하는 일부 로직만 필요해진다.

개선된 싱글톤은 이렇게 생길 수 있다.

```cs
public class Singleton : MonoBehaviour 
{ 
    private static Singleton instance;
    public static Singleton Instance 
    { 
        get 
        { 
            if (instance == null) 
            { 
                SetupInstance();
            } 
            return instance; 
        } 
    } 
    
    private void Awake() 
    { 
        if (instance == null) 
        { 
            instance = this; 
            DontDestroyOnLoad(this.gameObject); 
        } 
        else 
        { 
            Destroy(gameObject); 
        } 
    } 
    
    private static void SetupInstance() 
    { 
        instance = FindObjectOfType<Singleton>(); 
        if (instance == null) 
        { 
            GameObject gameObj = new GameObject(); 
            gameObj.name = "Singleton"; 
            instance = gameObj.AddComponent<Singleton>(); 
            DontDestroyOnLoad(gameObj); 
        } 
    }
```

`Instance`는 이제 private `instance`  백업 필드를 참조하는 public 프로퍼티가 되었다. 싱글톤을 처음 참조할 때, `getter`에서 `instance`의 존재 여부를 확인한다. 존재하지 않는 경우, `SetupInstance()`메소드가 적절한 컴포넌트와 함께 `GameObject`를 생성한다.

`DontDestroyOnLoad(gameObject)`는 Scene 로드가 hierarchy 에서 싱글톤을 지우는 것을 방지한다. 이제, 싱글톤 인스턴스는 지속적(persistence)이며, 게임에서 Scene을 변경하더라도 활성상태를 유지한다.

## 제네릭의 사용

현재 스크립트 버전은 동일한 씬 내에서 서로 다른 싱글톤을 생성하는 방법을 다루지 않습니다. 예를 들어, `AudioManager`로 동작하는 싱글톤과 `GameManager`로 동작하는 또 다른 싱글톤을 원한다면, 이들이 현재 함께 공존할 수 없습니다. 관련 코드를 복사하여 각 클래스에 로직을 붙여넣어야 한다.

대신, 다음과 같이 스크립트의 제네릭 버전을 만들 수 있다.

```cs
public class Singleton<T> : MonoBehaviour where T : Component 
{ 
    private static T instance;
    public static T Instance 
    { 
        get 
        { 
            if (instance == null) 
            { 
                instance = (T)FindObjectOfType(typeof(T)); 
                if (instance == null) 
                { 
                    SetupInstance(); 
                } 
            } 
            return instance; 
        } 
    } 
    
    public virtual void Awake() 
    { 
        RemoveDuplicates(); 
    } 
    
    private static void SetupInstance() 
    { 
        instance = (T)FindObjectOfType(typeof(T)); 
        if (instance == null) 
        { 
            GameObject gameObj = new GameObject(); 
            gameObj.name = typeof(T).Name; 
            instance = gameObj.AddComponent<T>(); 
            DontDestroyOnLoad(gameObj); 
        } 
    } 
    
    private void RemoveDuplicates() 
    { 
        if (instance == null) 
        { 
            instance = this as T; 
            DontDestroyOnLoad(gameObject); 
        } 
        else 
        { 
            Destroy(gameObject); 
        } 
    } 
}
```

이는 어떠한 클래스라도 싱글톤으로 변환할 수 있도록 해준다. 클래스를 선언할 때, 단순히 제네릭 싱글톤을 상속받으면 된다. 예를 들어, `GameManager`라는 `Monobehavior`를 싱글톤으로 만들고자 한다면 다음과 같이 선언할 수 있다.

```cs
public class GameManager : Singleton<GameManager>
{
    // ...
}
```

그리고 나선, 필요할 때마다 항상 `public static GameManager.Instance`를 참조할 수 있다. 

## 장단점

싱글톤은 이 가이드의 다른 패턴들과는 달리 여러 면에서 SOLID 원칙과 어긋난다. 많은 개발자들은 다양한 이유로 그들을 싫어한다.

#### 싱글톤은 전역 접근을 요구한다.
    전역 인스턴스로 사용하기 때문에, 많은 의존성을 숨길 수 있으며, 
    이는 버그를 해결하기 더 어렵게 만든다.

#### 싱글톤은 테스트를 어렵게 만든다.
    단위 테스트는 서로 독립적이어야 한다. 
    싱글톤은 Scene 전체의 많은 GameObject들의 상태를 변경할 수 있기 때문에, 
    테스트를 방해할 수 있다.

#### 싱글톤은 밀접한 결합을 지향한다.
    이 가이드의 대부분의 패턴은 의존성을 분리하려고 시도한다. 이는 싱글톤과 정 반대.
    밀접한 결합은 리팩토링을 어렵게 만든다. 한 컴포넌트를 변경하면, 
    그것과 연결된 모든 컴포넌트에 영향을 줄 수 있으며, 이는 깨끗하지 않은 코드로 이어진다.

싱글톤에 대한 부정적인 의견은 상당하다. 만약 앞으로 몇 년 동안 유지보수를 기대하는 엔터프라이즈 레벨의 게임을 만들고 있다면, 싱글톤을 피하는 것이 좋을 수 있다.

그러나 많은 게임들은 엔터프라이즈 레벨의 애플리케이션이 아니다. 비즈니스 소프트웨어를 지속적으로 확장해야 하는 것과 같은 방식으로 그것들을 계속 확장할 필요가 없다.

실제로, 싱글톤은 확장성이 필요하지 않은 작은 게임을 만드는 경우 매력적인 몇 가지 이점을 제공한다

#### 싱글톤은 배우기가 비교적 빠르다
    핵심 패턴 자체가 다소 직관적.
    
#### 싱글톤은 사용자 친화적
    다른 컴포넌트에서 싱글톤을 사용하려면, 단순히 공개적이고 정적인 인스턴스를 참조하면 된다.
     싱글톤 인스턴스는 씬의 어떤 객체에서든 요구할 때마다 항상 사용할 수 있다.

#### 싱글톤은 성능에 좋다
    항상 정적 싱글톤 인스턴스에 전역적으로 접근할 수 있기 때문에, 
    `GetComponent`나 `Find` 작업의 결과를 캐싱할 필요가 없어지며, 
    이 작업들은 일반적으로 느리다.

이런 방식으로, 게임 흐름 매니저나 오디오 매니저와 같은 매니저 객체를 만들 수 있으며, 씬 내의 모든 다른 `GameObject`에서 항상 접근할 수 있다. 또한, 오브젝트 풀을 구현했다면, 풀링 시스템을 싱글톤으로 설계하여 풀링된 객체를 가져오기 쉽게 만들 수 있다.

프로젝트에서 싱글톤을 사용하기로 결정한 경우, 싱글톤 사용을 최소화하라. 싱글톤을 함부로 사용하지 말 것. 전역 접근에서 이익을 볼 수 있는 소수의 스크립트에 대해 싱글톤을 예약하라.

## 참고한 자료
[Unity_E-Book_LevelUpYourCodeWithGameProgrammingPatterns](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)