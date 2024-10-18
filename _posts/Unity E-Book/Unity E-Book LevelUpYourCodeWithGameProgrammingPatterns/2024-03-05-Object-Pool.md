---
title:  Object Pool
date:   2024-03-05 +1100
categories: [Unity E-book]
tags: [디자인 패턴, Unity, C#, Unity E-book, LevelUpYourCodeWithGameProgrammingPatterns]
math: true
mermaid: true
---

> 위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.

오브젝트 풀링은 많은 `GameObject`를 생성하고 파괴할 때 CPU에 부담을 줄이기 위한 최적화 기법이다.

오브젝트 풀 패턴은 비활성화된 "Pool"에서 준비된 채 대기하는 초기화된 객체들을 사용하는 패턴이다. 객체가 필요할 때, 애플리케이션은 그것을 인스턴스화하지 않는다. 대신 풀로부터 `GameObject`를 요청하고 활성화한다.

사용이 끝나면, 객체를 파괴하는 대신 비활성화하여 풀로 반환한다.

오브젝트 풀은 가비지 컬렉션(GC) 스파이크로 인해 발생할 수 있는 끊김 현상을 줄일 수 있다. 
메모리 할당으로 인해 많은 수의 객체를 생성하거나 파괴할 때 종종 GC 스파이크가 동반된다. 이 때 GC 스파이크란 가비지 컬렉터가 동작할 때 처리해야할 메모리 양에 따라, 많게는 게임을 수백밀리초 동안 멈춰있게끔 할 수 있는데, 이러한 현상을 GC 스파이크라고 한다. 사용자가 끊김 현상을 눈치채지 못하는 시기, 예를 들어 로딩 화면 동안에 오브젝트 풀을 사전에 인스턴스화할 수 있다.

## 예제 : 간단한 풀 시스템
두개의 정의된 MonoBehaviour들을 활용해 간단한 풀링 시스템을 고려해보아라.

- 꺼내어 사용할 `GameObject`들의 컬렉션을 보유하는 `ObjectPool`
- 프리팹에 추가된 `PooledObject` 컴포넌트. 
  각 복제된 항목이 풀에 대한 참조를 유지하도록 돕는다.

`ObjectPool`에서는 풀의 크기, 저장하고자 하는 `PooledObject` 프리팹, 그리고 풀 자체를 형성할 컬렉션(이 예제에서는 스택)에 대해 설명하는 필드를 설정한다.

```cs
public class ObjectPool : MonoBehaviour 
{ 
    [SerializeField] private uint initPoolSize; 
    [SerializeField] private PooledObject objectToPool; 
    // store the pooled objects in a collection 
    private Stack<PooledObject> stack; 
    
    private void Start() 
    { 
        SetupPool(); 
    }
    
    // creates the pool (invoke when the lag is not noticeable) 
    private void SetupPool() 
    { 
        stack = new Stack<PooledObject>(); 
        PooledObject instance = null; 
        for (int i = 0; i < initPoolSize; i++) 
        { 
            instance = Instantiate(objectToPool); 
            instance.Pool = this; 
            instance.gameObject.SetActive(false); 
            stack.Push(instance); 
        }
    }
```

`SetupPool` 메소드는 오브젝트 풀을 채운다. `PooledObject`의 스택을 생성하고, 그것을 `initPoolSize` 요소로 채우기 위해 `objectToPool`의 복사본을 인스턴스화한다. 
게임 플레이 중 한 번 실행되도록 하기 위해 `Start`에서 `SetupPool`을 호출한다.

풀링된 아이템을 검색하는 메소드(`GetPooledObject`)와 풀로 아이템을 반환하는 메소드(`ReturnToPool`)도 필요할것이다.

```cs
    // returns the first active GameObject from the pool 
    public PooledObject GetPooledObject() 
    { 
        // if the pool is not large enough, instantiate a new PooledObjects 
        if (stack.Count == 0) 
        { 
            PooledObject newInstance = Instantiate(objectToPool); 
            newInstance.Pool = this; 
            return newInstance; 
        } 
        // otherwise, just grab the next one from the list 
        PooledObject nextInstance = stack.Pop();
        nextInstance.gameObject.SetActive(true); 
        return nextInstance; 
    } 
    
    public void ReturnToPool(PooledObject pooledObject) 
    { 
        stack.Push(pooledObject); 
        pooledObject.gameObject.SetActive(false); 
    } 
}

```

`GetPooledObject`는 풀이 비어 있을 경우에만 새로운 `PooledObject`를 생성한다. 
그렇지 않으면, 단순히 그 다음의 유효한 객체를 반환한다. 
풀 크기가 충분하다면, 대부분의 경우 기존 `GameObject`에 대한 참조만 가져오면 된다.

`GetPooledObject`를 호출하는 클라이언트는 그 후에 풀링된 객체를 적절한 위치로 이동/회전시켜야 합니다.

각 풀링된 객체는 `ObjectPool`을 참조하기 위한 작은 `PooledObject` 컴포넌트를 가지게 됩니다.

```cs
public class PooledObject : MonoBehaviour 
{ 
    private ObjectPool pool; 
    public ObjectPool Pool { get => pool; set => pool = value; } 
    public void Release() 
    { 
        pool.ReturnToPool(this); 
    } 
}
```

`Release`를 호출하면 `GameObject`가 비활성화되고 풀 큐로 반환된다.

동반된 프로젝트에서는 기본적인 사용 예를 보여준다. 여기서, `ExampleGun` 스크립트가 `GameObject`에 부착되어 있으며 오브젝트 풀에 대한 참조를 저장한다. 사용자가 발사할 때, 무기 스크립트는 `Object.Instantiate`를 호출하는 대신 자신의 `GetPooledObject` 메소드를 호출한다.

투사체 자체에는 `ExampleProjectile` 스크립트와 `PooledObject` 스크립트가 있다. `ExampleProjectile`에는 몇 초 후에 발사된 각 총알 `GameObject`를 비활성화하는 `Deactivate` 메소드가 있어, 사용 가능한 풀로 반환된다.

![image](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/9b5baa01-9c76-4b3c-9656-ef3abe23b351)

이 방식을 사용함으로써, 실제로는 수백 발의 총알을 화면 밖으로 발사하는 것처럼 보이게 할 수 있지만, 실제로는 그것들을 비활성화하고 재활용한다. 단지 동시에 활성화되어 보여야 하는 객체들을 표시하기에 충분히 큰 풀 크기를 확보해야 한다.

풀 크기를 초과해야 할 필요가 있다면, 풀은 추가 객체들을 인스턴스화할 수 있다. 그러나 대부분의 경우, 기존의 비활성 객체들에서 끌어온다.

Unity의 `ParticleSystem`을 사용해본 적이 있다면, 오브젝트 풀을 직접 경험한 것이다. `ParticleSystem` 컴포넌트에는 입자의 최대 수를 위한 설정이 포함되어 있다. 이는 사용 가능한 입자들을 단순히 재활용하여, 효과가 최대 수를 초과하는 것을 방지한다. 오브젝트 풀은 비슷하게 작동하지만, 선택한 어떤 `GameObject`와도 함께 사용할 수 있다.

## 개선 사항
위의 예제는 간단한 경우이고, 실제 프로젝트에 오브젝트 풀을 배포할 때는 다음과 같은 업그레이드를 고려하라.

### Static 혹은 Singleton으로 만들기
    다양한 출처에서 풀링된 객체를 생성해야 하는 경우, 
    오브젝트 풀을 static으로 만드는 것을 고려하라. 
    이렇게 하면 애플리케이션 어디에서나 접근할 수 있지만 인스펙터의 사용은 제외된다. 
    대안으로, 오브젝트 풀 패턴을 싱글톤 패턴과 결합하여 사용하기 쉽게 
    전역적으로 접근할 수 있게 만들 수 있다.

### 다수의 풀을 관리하기 위해 딕셔너리 사용하기
    풀링하고 싶은 다양한 프리팹이 있는 경우, 
    그것들을 별도의 풀에 저장하고 키-값 쌍을 저장하여 어떤 풀을 조회해야 하는지 알 수 있게 하라 
    (프리팹의 InstanceID는 고유 키로 작동할 수 있다).

### 미사용 GameObjects를 창의적으로 제거하기
    오브젝트 풀을 효과적으로 활용하는 방법 중 하나로는 미사용 객체를 숨기고 풀로 반환하는 것이 있다. 
    풀링된 객체를 비활성화할 모든 기회를 활용하라 (예: 화면 밖, 폭발에 의해 숨겨짐 등).

### 오류 확인하기
    이미 풀에 있는 객체를 해제하는 것을 피하라. 
    그렇지 않으면 런타임에 오류가 발생할 수 있다.

## 최대 크기 추가하기
    많은 수의 풀링된 객체는 메모리를 소비한다. 
    풀이 너무 많은 리소스를 사용하지 않도록 특정 한도를 초과하는 객체를 제거해야 할 수도 있다.

오브젝트 풀의 사용 방법은 애플리케이션마다 다를 것이다. 이 패턴은 총이나 무기가 다수의 발사체를 발사해야 하는 경우, 예를 들어 불렛 헬(bullet hell) 슈터 게임에서 흔히 나타난다.

대량의 객체를 인스턴스화할 때마다 가비지 컬렉션 스파이크로 인한 작은 일시 정지의 위험이 있다. 오브젝트 풀은 이 문제를 완화하여 게임 플레이를 부드럽게 유지한다.

2021년 이상의 버전의 Unity를 사용하고 있다면, 내장된 오브젝트 풀링 시스템이 포함되어 있으므로, 이전 예제처럼 자체 `PooledObject` 또는 `ObjectPool` 클래스를 생성할 필요가 없다. 

## UnityEngine.Pool

오브젝트 풀 패턴은 매우 일반적이어서 Unity 2021부터는 자체적인 `UnityEngine.Pool` API를 지원한다. 이를 통해 스택 기반 `ObjectPool`을 사용하여 오브젝트 풀 패턴으로 객체를 추적할 수 있습니다. 필요에 따라 `CollectionPool`(`List`, `HashSet`, `Dictionary` 등)을 사용할 수도 있다.

샘플 프로젝트(씬 참조)에서는 더 이상 사용자 정의 풀 컴포넌트가 필요하지 않다. 대신, 스크립트 상단에 `using UnityEngine.Pool;` 줄을 추가하여 총 스크립트를 업데이트하라. 이를 통해 내장된 `ObjectPool`을 사용하여 프로젝타일 풀을 생성할 수 있다

```cs
using UnityEngine.Pool; 

public class RevisedGun : MonoBehaviour 
{ 
    ...
    // stack-based ObjectPool available with Unity 2021 and above private
    IObjectPool<RevisedProjectile> objectPool; 
    
    // throw an exception if we try to return an existing item, already in the pool
    [SerializeField] private bool collectionCheck = true;
    // extra options to control the pool capacity and maximum size 
    [SerializeField] private int defaultCapacity = 20; 
    [SerializeField] private int maxSize = 100; 
    
    private void Awake()
    { 
        objectPool = new ObjectPool<RevisedProjectile>(CreateProjectile, OnGetFromPool, OnReleaseToPool, OnDestroyPooledObject, collectionCheck, defaultCapacity, maxSize);
    } 
    
    // invoked when creating an item to populate the object pool 
    private RevisedProjectile CreateProjectile() 
    { 
        RevisedProjectile projectileInstance = Instantiate(projectilePrefab);
        projectileInstance.ObjectPool = objectPool; 
        return projectileInstance; 
    } 
    
    // invoked when returning an item to the object pool
    private void OnReleaseToPool(RevisedProjectile pooledObject)
    { 
        pooledObject.gameObject.SetActive(false); 
    } 
    
    // invoked when retrieving the next item from the object pool 
    private void OnGetFromPool(RevisedProjectile pooledObject)
    { 
        pooledObject.gameObject.SetActive(true); 
    } 
    
    // invoked when we exceed the maximum number of pooled items (i.e. destroy the pooled object) 
    private void OnDestroyPooledObject(RevisedProjectile pooledObject)
    {
        Destroy(pooledObject.gameObject); 
    } 
    
    private void FixedUpdate() { ... } 
}
```

원래 `ExampleGun` 스크립트에 대해 많은 부분이 작동하지만, `ObjectPool` 생성자는 이제 다음과 같은 경우에 일부 로직을 설정할 수 있는 유용한 기능을 포함한다:

- 풀을 채우기 위해 풀링된 아이템을 처음 생성할 때
- 풀에서 아이템을 가져올 때
- 아이템을 풀로 반환할 때
- 풀링된 객체를 파괴할 때 (예: 최대 한도에 도달한 경우)

그런 다음 생성자에 전달할 몇 가지 해당 메소드를 정의해야 한다.

내장된 `ObjectPool`에는 기본 풀 크기와 최대 풀 크기에 대한 옵션도 포함되어 있다. 
최대 풀 크기를 초과하는 아이템은 메모리 사용을 체크하고, 자체 파괴하는 작업을 트리거한다.

projectile 스크립트는 `ObjectPool`에 대한 참조를 유지하도록 약간 수정된다. 
이는 객체를 풀로 다시 반환하는 것을 조금 더 편리하게 만든다.

```cs
public class RevisedProjectile : MonoBehaviour 
{
    ...
    private IObjectPool<RevisedProjectile> objectPool; 
    
    // public property to give the projectile a reference to its ObjectPool 
    public IObjectPool<RevisedProjectile> ObjectPool { set => object- Pool = value; } 
    ...
}
```

[`UnityEngine.Pool`](https://docs.unity3d.com/2021.1/Documentation/ScriptReference/Pool.ObjectPool_1.html?utm_source=demand-gen&utm_medium=pdf&utm_campaign=clean-code&utm_content=game-programming-patterns-ebook) API는 이제 처음부터 패턴을 다시 구축할 필요가 없기 때문에, 오브젝트 풀을 설정하는 것을 더 빠르게 만든다. 새로운 바퀴를 발명할 필요가 하나 줄어든 셈이다.


## 참고한 자료
[Unity_E-Book_LevelUpYourCodeWithGameProgrammingPatterns](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)