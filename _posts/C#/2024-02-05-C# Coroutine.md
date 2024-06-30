---
title:  Unity C# Coroutine
date:   2024-02-05 +0900
categories: [C#]
tags: [C#]
math: true
mermaid: true
---

## 코루틴 Coroutine

- `Enumerator`를 이용해서 여러 프레임에 걸쳐 실행된다.

> 업데이트 메서드와 달리 메서드 제어권을 유니티에 반환하고, <br> 특정 조건이 되면 다시 제어하는 기능을 가진다.

### 코루틴 사용조건

1. `MonoBehaviour` 를 상속하는 클래스 안에서만 사용가능.
2. `IEnumerator` 를 반환하는 함수여야 한다.
3. `yield return` 을 사용한다.
    1. 코드가 실행되는 시점을 다른 프레임으로 넘긴다.
4. `yield break;` 는 코루틴을 종료시킨다.
5. 코루틴을 실행하는 게임오브젝트가 비활성화되거나 파괴되면 코루틴 종료.

### 관련 메서드

- `StartCoroutine()` : 코루틴 실행 메서드
    - *`Coroutine StartCoroutine(string methodName)`*
    - *`Coroutine StartCoroutine(string methodName, [DefaultValue("null")] object value)`*
    - *`Coroutine StartCoroutine(IEnumerator routine)` - 권장*
        - 특별한 경우가 아니라면, *`methodName`*을 `string`으로 받는 버전이 아닌, <br> *`IEnumerator`*를 매개변수로 받는 버전을 사용할 것을 권장.
        - **가비지 Garbage**를 생성시킨다.
- `StopCoroutine()` : 코루틴 종료 메서드
    - *`void StopCoroutine(string methodName)`*
    - *`void StopCoroutine(IEnumerator routine)`*
    - *`void StopCoroutine(Coroutine routine)` - 권장*
    - 가급적이면 종료하고자 하는 코루틴을 정확하게 식별하기 위해, <br>
	    다음과 같은 방식으로 사용하는 것을 권장.
    ```csharp
    Coroutine co = StartCoroutine(Coroutine1());
    StopCoroutine(co);
    ```
- `StopAllCoroutines()` : 해당 스크립트의 모든 코루틴을 종료
    

## 작동 방식

- `C#` `Enumerator`를 활용해서 만들어짐
- 열거할 때 마다 특정 프레임에서 실행되도록 작동함.
- **코루틴을 사용할 때 가비지 컬렉션(Garbage Collection, GC)에 주의해야 함.**
    - **`new WaitForSeconds(1.0f)`**와 같이 매번 새로운 인스턴스를 생성하는 대신, 가능한 한 캐싱하여 재사용하는 것이 좋음. 이는 메모리 사용량을 줄이고, 가비지 컬렉션으로 인한 프레임 드랍을 방지할 수 있음.

### `YieldInstruction` 클래스

- 코루틴이 대기하고 실행하는 방식을 정의하는 클래스

```csharp
// 다음 프레임의 시작까지 대기 후 실행.
yield return null;

// 다음 FixedUpdate() 호출 시 까지 대기 후 실행.
yield return new WaitForFixedUpdate();

// 정한 시간만큼 지난 후 첫 프레임까지 대기 후 실행.
yield return new WaitForSeconds(1.0f);

// 정한 시간만큼 지난 후 첫 프레임까지 대기 후 실행.
// TimeScale의 영향을 받지 않음.
yield return new WaitForSecondsRealtime(1.0f);

// 현재 프레임의 렌더링 작업이 모두 완료된 후까지 대기 후 실행.
yield return new WaitForEndOfFrame();

// 해당 조건이 참이 될 때까지 대기 후 실행.
yield return new WaitUntil(()=>true);

// 위 모든 방식은 new를 사용하기 때문에, 이러한 방식은 가비지를 생성시킨다.
// 동일한 YieldInstruction을 여러번 사용할 때에는, 캐싱해서 사용하는 것도 방법.
```

- `yield return new {WaitForSeconds(1.0f), WaitForSecondsRealtime(1.0f)}`
    
    - **테스트 코드 & 결과**
        
        ```csharp
        using System.Collections;
        using UnityEngine;
        
        public class NewBehaviourScript : MonoBehaviour
        {
            private void Start()
            {
                StartCoroutine(Coroutine1());
                StartCoroutine(Coroutine2());
            }
        
            IEnumerator Coroutine1()
            {
                float time1, time2;
        
                time1 = Time.time;
                yield return new WaitForSecondsRealtime(1.0f);
                time2 = Time.time;
                Debug.Log("WaitForSecondsRealtime : " + (time2 - time1));
                
                StartCoroutine(Coroutine1());
            }
            
            IEnumerator Coroutine2()
            {
                float time1, time2;
        
                time1 = Time.time;
                yield return new WaitForSeconds(1.0f);
                time2 = Time.time;
                Debug.Log("WaitForSeconds : " + (time2 - time1));
                
                StartCoroutine(Coroutine2());
            }
            
        }
        ```

        ![스크린샷 2024-02-05 234407](https://github.com/BJH7536/BJH7536.github.io/assets/114412598/03faf88b-f793-428b-9cab-c2c8a9a74a5b){: width="433" height="573" }
        _위 코드를 실행한 후, 에디터의 콘솔부분 스크린샷_
    - 위 두 `CustomYieldInstruction`은 1초를 정확히 측정할 수는 없다.
        
#### `WaitForSeconds`로 인한 가비지 생성을 최소화하기 위한 방법
```csharp
public class WaitForSecondsDictionary : MonoBehaviour
{
	public static readonly WaitForFixedUpdate WaitForFixedUpdate = new WaitForFixedUpdate();
	public static readonly WaitForEndOfFrame WaitForEndOfFrame = new WaitForEndOfFrame();

	private static readonly Dictionary<float, WaitForSeconds> WaitForSecondsDict = new Dictionary<float, WaitForSeconds>();

	public static WaitForSeconds WaitForSeconds(float waitTime)
	{
		WaitForSeconds wfs;

		if (WaitForSecondsDict.TryGetValue(waitTime, out wfs))
			return wfs;
		else
		{
			wfs = new WaitForSeconds(waitTime);
			WaitForSecondsDict.Add(waitTime,wfs);
			return wfs;
		}
	}
}
```
    
- 매개변수를 필요로 하는 `YieldInstruction`은 하나의 객체로 생성해서 캐싱,
-  그리고 `WaitForSeconds`는 실수를 키로 갖는 `Dictionary`를 만들어, <br> 해당 시간만큼의 `WaitForSeconds`가 만들어져있지 않다면, 새로 만들어 사용하도록하는 클래스.

## 이벤트 함수와 코루틴

```csharp
private IEnumerator Start()
{
    yield return null;
}

private IEnumerator OnCollisionEnter(Collision other)
{
    yield return null;
}

private IEnumerator OnMouseEnter()
{
    yield return null;
}

private IEnumerator OnBecameInvisible()
{
    yield return null;
}
```

- 위와 같이, 유니티의 이벤트 함수를 코루틴 버전으로 동작시키는 것도 가능하다.

## 가비지 컬렉션을 최소화하는 CustomYieldInstruction 상속 클래스

```csharp
using System;
using UnityEngine;

public class WaitForCustomCondition : CustomYieldInstruction
{
    private Func<bool> _condition;  // 대기를 계속할 조건을 체크하는 함수
    
    // keepWaiting 프로퍼티 오버라이드. _condition 함수의 반환값이 false가 될 때까지 대기.
    public override bool keepWaiting => !_condition();
    
    // 생성자에서 대기 조건 함수를 받음
    public WaitForCustomCondition(Func<bool> condition)
    {
        _condition = condition;
    }
}

public class CoroutineExample : MonoBehaviour
{
    // 코루틴 예제
    IEnumerator ExampleCoroutine()
    {
        Debug.Log("코루틴 시작, 조건을 만족할 때까지 대기");
        
        // CustomYieldInstruction을 사용하여 특정 조건이 참이 될 때까지 대기
        yield return new WaitForCustomCondition(() => Time.time > 5); 
        // 예를 들어, 시간이 5초를 초과할 때까지 대기
        
        Debug.Log("조건 만족, 코루틴 계속 진행");
    }

    void Start()
    {
        StartCoroutine(ExampleCoroutine()); // 코루틴 실행
    }
}
```