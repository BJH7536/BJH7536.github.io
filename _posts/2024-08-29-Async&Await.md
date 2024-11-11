---
title: Async&Await
date: 2024-08-29 +0900
categories: [C#]
tags: [C#, UniTask, Task]
math: true
mermaid: true
---

# 동기 Synchronous

**동기 Synchronous 방식**은 **작업이 순차적으로 일어나는 방식**을 말한다. 하나의 작업이 완료되기 전까지 다음 작업이 시작되지 않는 것.

대체로 클라이언트 내부에 구현하는 간단한 기능들은 이 방식에 해당한다.

### 예제 - 동기적 방식으로 텍스트 파일에 접근하기

지정된 경로의 텍스트 파일을 읽는 것을 시도하고, 성공하면 그 내용을 출력하는 코드

```csharp
using System.IO;
using System.Threading;
using UnityEngine;

public class AsyncAwaitTest : MonoBehaviour
{
    private string filePath;
    private string fileContents;

    private void Start()
    {
        // Application.dataPath = Assets 파일 경로 
        filePath = Path.Combine(Application.dataPath, "Resources/File.txt");
				
        
        
        Debug.Log("메인 스레드 진행 중");
    }

    void ReadFile()
    {
        if (File.Exists(filePath))
        {
            fileContents = File.ReadAllText(filePath);
        }
    }
    
}
```

새로운 스레드를 만들고, 이 스레드가 지정된 경로의 텍스트 파일을 읽도록 지시한다.

Join 메서드를 통해 메인 스레드는 앞서 만든 스레드의 작업이 완료될 때까지 대기한다.

스레드의 작업이 모두 완료된 후, 메인 스레드는 스레드의 작업 결과를 이용해 파일 내용을 출력.

# 비동기 Asynchronous

**비동기 Asynchronous 방식**은 **프로그램이 병렬적으로 실행**되도록 하여 작업을 효율적으로 처리할 수 있도록 한다. 하나의 작업이 완료될 때까지 기다리지 않고, 다른 작업을 수행할 수 있다.

주로 I/O 작업 (파일 읽기/쓰기, 네트워크 통신)에 사용된다.

### 예제 1. BeginInvoke와 EndInvoke의 활용

`BeginInvoke`와 `EndInvoke`는.NET Framework 1.0부터 고전적 프로그래밍 방식이다.

Delegate를 비동기적으로 실행할 수 있게 해주지만, C# 5.0에서 `async`와 `await` 키워드가 도입되면서 비동기 프로그래밍을 더욱 간단하게 다룰 수 있게 되었다. 

```csharp
using System;
using System.Threading;
using UnityEngine;

public class AsyncAwaitTest : MonoBehaviour
{
    private void Start()
    {
        Action action = LongRunningOperation;
        IAsyncResult result = action.BeginInvoke(new AsyncCallback(EndOperationCallback), null);
        Debug.Log("메인 스레드 진행 중");
    }

    void LongRunningOperation()
    {
        // 오래 걸리는 작업
        Thread.Sleep(3000);
        Debug.Log("오래 걸리는 작업 완료");
    }

    void EndOperationCallback(IAsyncResult asyncResult)
    {
        Debug.Log("콜백 실행");
    }
}
```

![2024-08-28193757-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/4d961574-7d37-487c-91b2-6e6304dd49f1)

`BeginInvoke` 메서드는 
`action`을 비동기적으로 실행하기 위해 
새로운 서브 스레드를 생성한다.

그리고 그 서브 스레드가 `action`을 
실행하도록 한다.

수행이 완료되면 
`AsyncCallback` 델리게이트에 지정된 
메서드인 `EndOperationCallback`이 
호출된다.

### 예제 2. 비동기적 방식으로 텍스트 파일에 접근하기

```csharp
using System;
using System.IO;
using UnityEngine;

public class AsyncAwaitTest : MonoBehaviour
{
    private string filePath;
    private string fileContents;

    private void Start()
    {
        // Application.dataPath = Assets 파일 경로 
        filePath = Path.Combine(Application.dataPath, "Resources/File.txt");

        // 델리게이트 생성
        Func<string> readFileDelegate = new Func<string>(ReadFile);
        
        // 비동기 호출 시작
        IAsyncResult result = readFileDelegate.BeginInvoke(ReadFileCallback, readFileDelegate);
        
        Debug.Log("메인 스레드 진행 중");
    }

    string ReadFile()
    {
        Debug.Log("파일 읽기 중");
        if (File.Exists(filePath))
        {
            return File.ReadAllText(filePath);
        }
        return null;
    }

    void ReadFileCallback(IAsyncResult ar)
    {
        // 델리게이트로부터 결과를 받아온다. ar.AsyncState에 readFileDelegate 저장됨
        Func<string> readFileDelegate = (Func<string>)ar.AsyncState;
        fileContents = readFileDelegate.EndInvoke(ar);
        
        // 비동기 작업 완료 후 결과 출력
        Debug.Log($"파일 읽기 완료 : {fileContents}");
    }
}
```

위 코드에서도 `BeginInvoke` 함수를 사용해 `readFileDelegate`를 비동기적으로 실행한다. 

이때, `BeginInvoke` 함수의 두번째 인자로 `readFileDelegate`를 넘겨준다. 

이는  `ReadFileCallback`메서드에서 `IAsyncResult` 인터페이스의 `AsyncState` 속성을 통해 `BeginInvoke`에 넘겼던 델리게이트 (`readFileDelegate`)를 다시 가져올 수 있게 하기 위함이다.

`ReadFileCallback` 메서드에서는 `EndInvoke`메서드를 통해 비동기 작업이 완료될 때까지
대기하고, 그 결과를 가져온다.

### 예제 3. async와 await을 활용해 예제2를 재현

`async`와 `await` 키워드를 활용해 예제 2와 동일한 기능을 재현해보자.

```csharp
using System.IO;
using System.Threading.Tasks;
using UnityEngine;

public class AsyncAwaitTest : MonoBehaviour
{
    private string filePath;
    private string fileContents;

    private async void Start()
    {
        // Application.dataPath = Assets 파일 경로 
        filePath = Path.Combine(Application.dataPath, "Resources/File.txt");
        Debug.Log("메인 스레드 진행 중");

        // 비동기 파일 읽기 시작
        fileContents = await ReadFileAsync(filePath);
        
        // 비동기 작업 완료 후 결과 출력
        Debug.Log($"파일 읽기 완료 : {fileContents}");
    }

    async Task<string> ReadFileAsync(string path)
    {
        if (File.Exists(path))
        {
            return await Task.Run(() => File.ReadAllText(path));
        }
        return null;
    }
}
```

위 코드는 예제 2에서 비동기적으로 텍스트파일에 접근하는 기능을 동일하게 구현한다.

동일한 기능이지만, 전반적인 코드의 양도 줄고 흐름도 자연스러워져 가독성이 개선된 점들을 알 수 있다.

## async와 await의 사용법

- `using System.Threading.Tasks;` 선언 필요
- 비동기로 실행할 메서드에 `async` 키워드 사용
- 비동기로 실행할 부분 앞에 `await` 키워드 사용
    - `await` 키워드는 `async`로 선언된 비동기 메서드 내에서만 키워드로 인식된다.
- `async` 메서드의 반환타입은 일반적으로 `Task`, `Task<TResult>` 그리고 `void`

### 예제

```csharp
using System.Threading.Tasks;
using UnityEngine;

public class AsyncAwaitTest : MonoBehaviour
{
    private async void Start()
    {
        Debug.Log("메인 스레드 시작");
        await SumAsync(10, 20);     // 비동기 메서드 호출
        Debug.Log("메인 스레드 진행 중");
    }

    async Task SumAsync(int num1, int num2)
    {
        // Task.Delay를 사용하여 오래 걸리는 작업을 비동기적으로 수행
        await Task.Delay(3000);
        int num3 = num1 + num2;
        Debug.Log("오래 걸리는 작업 완료");
        Debug.Log($"비동기 작업 결과 : {num3}");
    }
}
```

`Task.Delay`는 `Thread.Sleep`의 비동기 버전으로, 메인 스레드를 차단하지 않는다.

![image (22)](https://github.com/user-attachments/assets/6348f7f1-740b-466d-afc9-38a2ee610f77)

위 코드의 실행결과는 이와 같다.

가장 먼저 “메인 스레드 시작”이 콘솔에 출력되고

3초 후에 나머지 로그들이 출력된다.

그런데 좀 이상하다. 
“메인 스레드 진행 중”이 왜 가장 마지막에 출력될까

그 이유는 메인 스레드의 진행에 있다. 

"메인 스레드 진행 중"이 마지막에 출력되는 이유는, `await` 키워드를 만나면 비동기 메서드의 작업이 끝날 때까지 `Start` 메서드의 흐름이 일시 중단되기 때문이다. 비록 메인 스레드가 차단되지 않지만, `await` 이후의 코드 실행은 비동기 작업이 완료된 후에 재개된다.

## async와 await의 장점

- **비동기 코드를 보다 간편하고 이해하기 쉽게 작성할 수 있다**
    - 복잡한 비동기 작업을 처리할 때, 콜백 지옥을 피하고 간단하게 구현할 수 있다.
- **동기적 코드와 유사한 방식으로 작성할 수 있어 코드의 가독성이 좋다**
    - 비동기 작업을 동기적인 코드처럼 작성할 수 있어, 비동기 로직을 추적하기가 쉬워진다.
- **스레드를 차단하지 않기에 사용자 경험을 저해할 여지가 줄어든다**
    - 긴 작업을 처리하는 동안에도 UI가 응답성을 유지하므로, 사용자가 느끼는 지연이나 멈춤 현상이 감소된다.