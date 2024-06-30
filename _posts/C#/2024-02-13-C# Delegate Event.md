---
title:  Unity C# Delegate Event
date:   2024-02-13 +0900
categories: [C#]
tags: [C#, Unity]
math: true
mermaid: true
---

## Delegate (델리게이트)

- **델리게이트 (Delegate)**
    - 메소드에 대한 참조를 보유하는 타입.
    - 특정 시그니처(반환 유형과 매개 변수)를    가진 메소드를 참조할 수 있음.
    - 다중 델리게이트를 지원하여 한 델리게이트가 여러 메소드를 참조할 수 있음.

## Event (이벤트)

- **이벤트 (Event)**
    - 클래스 또는 객체의 중요한 행동이나 상태 변경을 다른 클래스나 객체에 알리는 데 사용.
    - 이벤트는 델리게이트를 기반으로 하지만, 외부 클래스에서 직접 호출(**`Invoke()`**)할 수 없음.
    - 외부 클래스는 이벤트에 대한 구독(**`+=`**)과 구독 취소(**`-=`**)만 할 수 있음.

### 사용 예제

```csharp
public class TestDele
{
    public delegate void TestEvent();
    public event TestEvent testEvent;

    public void StartEvent()
    {
        testEvent?.Invoke(); // 이벤트가 null이 아닐 때만 호출
        // 이벤트에 구독자가 없을 때 오류를 방지
    }
}

public class TestDelegate : MonoBehaviour
{
    private void Start()
    {
        TestDele testDele = new TestDele();
        testDele.testEvent += Test1;
        testDele.testEvent += Test2;
        testDele.testEvent -= Test2; // 이벤트 구독 취소 예시
        testDele.testEvent += Test3;

        //testDele.testEvent?.Invoke(); // 이벤트 외부 호출은 불가능
        testDele.StartEvent();
    }

    public void Test1() { Debug.Log("Test1"); }
    public void Test2() { Debug.Log("Test2"); }
    public void Test3() { Debug.Log("Test3"); }
}
```