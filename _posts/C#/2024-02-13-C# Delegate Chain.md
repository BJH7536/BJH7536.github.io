---
title:  Unity C# Delegate Chain
date:   2024-02-13 +0900
categories: [C#]
tags: [C#, Unity]
math: true
mermaid: true
---

# Delegate Chain

## 하나의 델리게이트가 여러 함수를 참조할 수 있다.

- `Delegate Chain`도 이러한 점에서 코드 간의 `의존성`과 `결합도`를 낮출 수 있다.
- 코드의 가독성은 오히려 감소한다.
    - 해당 `Delegate`에 어떠한 함수가 들어가는지 번거롭게 찾아야 하기 때문.
    - `의존성`과 `결합도`에 있어서는 훨씬 편해진다는 점은 유의미함.

## 사용법

```csharp
public class DelegateTest : Monobehavior
{
    private delegate void TestDelegate();    // delegate 타입 정의

    private TestDelegate testDelegate;    // delegate 변수 선언

    // 참조할 함수들
    void Chain1() { Debug.log("Chain1"); }
    void Chain2() { Debug.log("Chain2"); }
    void Chain3() { Debug.log("Chain3"); }

    void Start() 
    {
        TestDelegate test1 = new TestDelegate(Chain1);
        TestDelegate test2 = new TestDelegate(Chain2);
        TestDelegate test3 = new TestDelegate(Chain3);

        // 다음의 방법들은 모두 동일 기능

        // 방법1
        testDelegate = Delegate.Combine(test1, test2) as TestDelegate;
        testDelegate = Delegate.Combine(testDelegate , test3) as TestDelegate;    

        // 방법2
        testDelegate = new TestDelegate(Chain1) 
        + new TestDelegate(Chain2) 
        + new TestDelegate(Chain3);

        // 방법3
        testDelegate = new TestDelegate(Chain1);
        testDelegate += new TestDelegate(Chain2);
        testDelegate += new TestDelegate(Chain3);

        // 방법4
        testDelegate = Chain1;
        testDelegate += Chain2;
        testDelegate += Chain3;

        // 반대로 빼는 방법
        testDelegate = Chain1;
        testDelegate += Chain2;
        testDelegate += Chain3;

        testDelegate -= Chain2;
        testDelegate -= Chain3;
        // 결과적으로 Chain1 함수 하나만 실행된다.

        testDelegate .Invoke();
    }
}
```