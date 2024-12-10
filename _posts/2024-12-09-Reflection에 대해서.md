---
title: Reflection에 대해서
date: 2024-12-09 +0900
categories: [C#]
tags: [C#, Reflection]
math: true
mermaid: true
---

## Reflection

모든 타입은 내부에 다양한 데이터가 있다.

Reflection은 이런 타입의 데이터를 런타임 중에 동적으로 볼 수 있는 강력한 기능을 제공한다.

이를 통해 해당 타입의 프로퍼티, 메서드, 필드 이벤트 등을 확인할 수 있다.

### 프로세스

![image (63)](https://github.com/user-attachments/assets/c4b22090-ebe1-4ddc-a347-2f6292ae0f60)

Reflection에 대해 알아보기 전에, 프로세스의 구조에 대해 먼저 짚고 넘어가자.

프로세스의 내부에는 App Domain이 있고,

그 안에는 어셈블리가 있다.

### AppDomain

- 프로세스 내부에 독립적인 실행환경을 제공해 서로 충돌하지 않도록 보호하는 영역.
- 이는 각각이 격리된 공간
    - 그래서 서로간에 통신할 방법을 설정하지 않으면 통신할 수 없다.
    - 서로 영역을 침범할 수 없다.
- 하나의 프로세스에 AppDomain이 여러개 있으면, 하나가 죽더라도 나머지는 영향을 받지 않는다.
- 모든 프로세스는 기본적으로 하나의 AppDomain을 갖고 있다.

### 어셈블리

![image (64)](https://github.com/user-attachments/assets/efe5a7d9-3cc2-4c73-aec5-429ef0c613b7)

- 하나 이상의 모듈로 구성된다.
- 모듈은 여러 클래스를 갖는다.
- 어셈블리를 설명하자면
    - 컴파일된 코드의 단위, 프로그램의 **배포 단위**

### 모듈

- 어셈블리 내의 코드 및 데이터 단위
- 하나의 모듈은 하나의 EXE 또는 DLL 파일을 나타낸다.

### 현재 AppDomain 내부의 어셈블리들에 직접 접근해보기

```csharp
void Start()
{
  AppDomain currentDomain = AppDomain.CurrentDomain;

  foreach (var assembly in currentDomain.GetAssemblies())
  {
      Debug.Log(assembly.FullName);
  }
}
```

위 코드를 실행한 결과 다음의 사진 속 콘솔창에 무수히 많은 어셈블리에 대한 정보들이 출력된다.

`AppDomain.CurrentDomain` 을 통해 현재 `AppDomain`에 접근할 수 있고, `GetAssemblies()` 함수를 통해 AppDomain의 내부에 있는 어셈블리들에 접근한다. 

이렇게 많은 어셈블리들이 현재 AppDomain 내부에 존재함을 알 수 있다.

![image (65)](https://github.com/user-attachments/assets/781c0cf1-3a18-47c9-ae80-086ce0878dff)

### 어셈블리 내부의 모듈까지 접근해보기

```csharp
void Start()
{
    AppDomain currentDomain = AppDomain.CurrentDomain;

    foreach (var assembly in currentDomain.GetAssemblies())
    {
        foreach (var module in assembly.GetModules())
        {
            Debug.Log(module.FullyQualifiedName);
        }
    }
}
```

위 코드는 한단계 더 들어가서 `GetModules()` 를 통해 하나의 어셈블리 안에 있는 모듈들에 접근한다.

이번에도 무수히 많은 모듈이 존재함을 알 수 있다.

![image (66)](https://github.com/user-attachments/assets/d87f5f3c-0058-419a-aa0d-f055350cf1e8)

### 모듈 내부의 클래스(타입)까지 접근해보기

```csharp
void Start()
{
    AppDomain currentDomain = AppDomain.CurrentDomain;

    foreach (var assembly in currentDomain.GetAssemblies())
    {
        foreach (var module in assembly.GetModules())
        {
            foreach (var type in module.GetTypes())
            {
                Debug.Log(type.FullName);
            }
        }
    }
}
```

위 코드는 또 한단계 더 들어가서, 모듈 내부에 속하는 클래스(타입)들까지 접근한다.

셀 수도 없이 많은 클래스(타입)들이 존재함을 알 수 있다. 

![image (67)](https://github.com/user-attachments/assets/cb7e2b2f-b32b-444f-bb8e-05cd5fa5e746)

```csharp
void Start()
{
    AppDomain currentDomain = AppDomain.CurrentDomain;

    foreach (var assembly in currentDomain.GetAssemblies())
    {
        foreach (var type in assembly.GetTypes())
        {
            Debug.Log(type.FullName);
        }
    }
}
```

이렇게 어셈블리에서도 바로 모듈 내부의 클래스(타입)들에 접근할 수 있다.

결과는 당연히 동일하다.

### 하나의 클래스(타입)에 대해 사용할 수 있는 몇가지 메서드들

지금까지 하나의 프로세스안에는 한 개 이상의 AppDomain이 있고, 

- 하나의 AppDomain 내부에는 여러 어셈블리가,
- 하나의 어셈블리 안에는 여러 모듈이,
- 하나의 모듈안에는 또 여러 클래스(타입)가 있을 수 있음에 대해 알아보았다.

그리고 이렇게 접근한 클래스(타입)에 대해서 사용할 수 있는 메서드가 몇가지 있는데, 
다음은 그 중 일부다.

- `GetConstructors()`
    - 기능 : 생성자 목록 반환
    - 반환 타입 : `ConstructorsInfo[]`
- `GetEvents()`
    - 기능 : 이벤트 목록 반환
    - 반환 타입 : `EventInfo[]`
- `GetFields()`
    - 기능 : 필드 목록 반환
    - 반환 타입 : `FieldInfo[]`
- `GetGenericArguments()`
    - 기능 : 형식 매개변수 목록 반환
    - 반환 타입 : `Type[]`
- `GetInterfaces()`
    - 기능 : 인터페이스 목록 반환
    - 반환 타입 : `Type[]`
- `GetMembers()`
    - 기능 : 멤버 목록 반환
    - 반환 타입 : `MemberInfo[]`
- `GetMethods()`
    - 기능 : 메서드 목록 반환
    - 반환 타입 : `MethodInfo[]`
- `GetNestedTypes()`
    - 기능 : 내장 형식 (중첩 클래스) 목록 반환
    - 반환 타입 : `Type[]`
- `GetProperties()`
    - 기능 : 프로퍼티 목록 반환
    - 반환 타입 : `PropertyInfo[]`

```csharp
void Start()
{
    int a = 0;
    Type type = a.GetType();
    FieldInfo[] fields = type.GetFields();

    foreach (var fieldInfo in fields)
    {
        Debug.Log($"{fieldInfo.FieldType.Name} : {fieldInfo.Name}");
    }
}
```

위 코드의 동작을 설명하자면

1. 가장 먼저 `int` 타입 변수 `a`의 타입을 추출한다. `System.Int32`에 해당한다.
2. `System.Int32` 의 필드 목록을 추출한다. 
    
    ![image (68)](https://github.com/user-attachments/assets/afd06c67-4bb2-46f3-a503-c66aa82c647b)
    
    실제 `System.Int32`의 구조는 위와 같다. 
    해당 클래스의 필드에는 `MaxValue`와 `MinValue`, 이 두 개가 존재한다. 
    
    한 필드에 대한 정보를 `FieldInfo`로, 모든 필드에 대한 정보는 `FieldInfo[]` 로 반환한다.
    
3. 각각의 필드에 대해 타입의 이름과, 필드의 이름을 출력한다.

그 결과는 다음과 같다.

![image (69)](https://github.com/user-attachments/assets/6b74414a-6c82-4b73-be26-92775ba2f663)

### 타입에 접근하는 방법

```csharp
void Start()
{
    int a = 0;

    Type t1 = typeof(int);
    Type t2 = a.GetType();
    Type t3 = Type.GetType("System.Int32");
    
    Debug.Log(t1.Name);
    Debug.Log(t2.Name);
    Debug.Log(t3.Name);
}
```

위와 같이 총 3가지 방법으로 타입에 접근할 수 있다. 

- `typeof()`
    - 컴파일 시점에 해당 타입을 가져올 때 사용한다.
    - 컴파일 시점에 이미 타입 정보가 있어야 한다.
- `GetType()`
    - 런타임에 인스턴스의 타입을 가져올 때 사용한다.
- `Type.GetType()`
    - 런타임에 문자열로 타입 정보를 가지고 온다.
    - 어셈블리 전체를 대상으로 찾는다. 문자열에 전체 네임스페이스를 포함해야 한다.

그리고 위 3가지 방법은 모두 동일한 결과를 출력한다.

![image (70)](https://github.com/user-attachments/assets/96cb8570-fffc-40cd-a5cc-c19b89146e64)

### 타입의 더 자세한 정보에 대해 접근해보자

```csharp
void Start()
{
    int a = 0;
    Type type = a.GetType();
    
    Debug.Log("============= Interfaces =============");
    Type[] interfaces = type.GetInterfaces();
    foreach (var _interface in interfaces)
    {
        Debug.Log($"{_interface.Name}");
    }
    
    Debug.Log("============= Fields =============");
    FieldInfo[] fields = type.GetFields(
        BindingFlags.Public | 
        BindingFlags.NonPublic | 
        BindingFlags.Static | 
        BindingFlags.Instance);

    foreach (var field in fields)
    {
        string accessLevel = field.IsPublic ? "public" : (field.IsPrivate ? "private" : "protected");
        
        Debug.Log($"{accessLevel} | {field.FieldType.Name} | {field.Name}");
    }
    
    Debug.Log("============= Methods =============");
    MethodInfo[] methods = type.GetMethods();
    foreach (var method in methods)
    {
        Debug.Log($"{method.ReturnType} | {method.Name}");
    }

}
```

위와 같이 하나의 클래스(타입)에 대해

- 어떤 인터페이스를 상속받는지
- 어떤 필드들을 갖고 있는지
- 어떤 메소드들을 갖고 있는지

와 같은 정보에 대해 접근하는 것 또한 가능하다.

위 코드 중 `GetFields()` 의 인자로 들어간 값들에 대해 설명하자면, 우선 이 값들은 `BindingFlag`라고 부르는 값이다. 이에 대한 정의는 `System.Reflection`의 아래에 위치한다. 클래스의 필드, 메서드, 프로퍼티와 같은 멤버를 검색할 때 검색 조건에 해당한다고 이해하면 될 것이다. 다양한 값이 존재하지만, 대체로 많이 쓰이는 값들은 다음과 같다.

- `Public` : public 으로 선언된 멤버를 검색
- `NonPublic` : private, protected 등 비공개 맴버 검색
- `Instance` : 인스턴스 멤버 검색
- `Static` : 정적 (static) 멤버 검색
- `DeclaredOnly` : 상속받은 멤버 제외, 클래스에서 선언된 멤버 검색
- `IgnoreCase` : 멤버 이름 대소문자 구분 없이 검색

그리고 다음은 위 코드의 실행 결과

![image (71)](https://github.com/user-attachments/assets/c38c6a3b-7cd9-4790-aef2-fc554dc627a3)

![image (72)](https://github.com/user-attachments/assets/fdf558d3-6f3c-4d9f-9435-9a5978a77dbf)

![image (73)](https://github.com/user-attachments/assets/ac84351c-f871-49a1-8a8e-00b679885b6b)

## Reflection을 활용한 인스턴스 생성과 메서드 호출

```csharp
public class SystemInfo
{
    private bool _is64Bit;

    public SystemInfo()
    {
        _is64Bit = Environment.Is64BitOperatingSystem;
        Debug.Log("SystemInfo created");
    }

    public void WriteInfo()
    {
        Debug.Log($"OS == {(_is64Bit == true ? "64" : "32")} bits");
    }
}

private void Start()
{
    SystemInfo systemInfo = new SystemInfo();
    systemInfo.WriteInfo();

    Type systemInfoType = typeof(SystemInfo);
    object objInstance = Activator.CreateInstance(systemInfoType);
    MethodInfo methodInfo = systemInfoType.GetMethod("WriteInfo");
    methodInfo.Invoke(objInstance, null);
}
```

위 코드의 `Start` 함수 안에는 동일한 기능을 두 번 하는 코드가 작성되어 있는데, 바로 

```csharp
SystemInfo systemInfo = new SystemInfo();
systemInfo.WriteInfo();
```

```csharp
Type systemInfoType = typeof(SystemInfo);
object objInstance = Activator.CreateInstance(systemInfoType);
MethodInfo methodInfo = systemInfoType.GetMethod("WriteInfo");
methodInfo.Invoke(objInstance, null);
```

위 두 코드가 동일한 기능을 한다. 

`SystemInfo systemInfo = new SystemInfo()`와 같이 `SystemInfo` 타입의 인스턴스를 만드는 것을 
두 번째 코드에서는 `Activator.CreateInstance()`로 대신하고, 
`systemInfo.WriteInfo()` 와 같이 해당 인스턴스의 인스턴스 메서드를 호출하는 것을 
두 번째 코드에서는 `systemInfoType.GetMethod("WriteInfo")`  로 받아온 `MethodInfo` 객체에`methodInfo.Invoke(objInstance, null)` 와 같이 `Invoke` 해줌으로써 대신한다. 

결과는 다음과 같다.

![image (74)](https://github.com/user-attachments/assets/a292a928-9650-4b5f-9e3f-1e22c64a6814)

## Reflection을 활용한 인스턴스 필드 값 수정

```csharp
public class SystemInfo
{
    private bool _is64Bit;

    public SystemInfo()
    {
        _is64Bit = Environment.Is64BitOperatingSystem;
        Debug.Log("SystemInfo created");
    }

    public void WriteInfo()
    {
        Debug.Log($"OS == {(_is64Bit == true ? "64" : "32")} bits");
    }
}

private void Start()
{
    SystemInfo systemInfo = new SystemInfo();
    systemInfo.WriteInfo();

    Type systemInfoType = typeof(SystemInfo);
    object objInstance = Activator.CreateInstance(systemInfoType);

    FieldInfo fieldInfo = systemInfoType.GetField("_is64Bit", BindingFlags.NonPublic | BindingFlags.Instance);
    fieldInfo.SetValue(objInstance, !Environment.Is64BitOperatingSystem);
    
    MethodInfo methodInfo = systemInfoType.GetMethod("WriteInfo");
    methodInfo.Invoke(objInstance, null);
}
```

위 코드는 직전에서 본 코드에서 두 줄이 추가된 코드다.

```csharp
FieldInfo fieldInfo = systemInfoType.GetField("_is64Bit", BindingFlags.NonPublic | BindingFlags.Instance);
fieldInfo.SetValue(objInstance, !Environment.Is64BitOperatingSystem);
```

이는 `objInstance` 의 필드인 `_is64Bit`의 값을 `!Environment.Is64BitOperatingSystem` 로 수정한다.

`_is64Bit` 필드는 `private`으로 제한되어있지만, 위와 같이 Reflection을 활용하면 해당 필드에 접근해 값을 바꾸는 것도 가능하다.

결과는 다음과 같다.

![image (75)](https://github.com/user-attachments/assets/bf92aedb-8572-4bb5-abf6-f47ddc0fb57b)

## 주의할 점

지금까지 Reflection에 대해 알아보면서 기존과는 다른 방식으로 클래스와 인스턴스의 정보에 접근하는 방법을 알아보았다. 그러나 그렇다고해서 Reflection을 남용하는 것은 금물인데, 그 주된 이유 중 하나는 바로 성능에 있다.

Reflection은 런타임에 메타데이터를 조회하고, 동적으로 객체 생성과 메서드 호출을 수행하기에 일반적인 코드의 실행에 비해 느릴 수 밖에 없다.

그리고 다음은 직접 얼마나 느린지를 테스트하기 위한 코드다.

```csharp
public class SpeedTestClass
{
    public void SpeedTestMethod()
    {
        for (int i = 0; i < 10; i++) { }
    }
}

private void Start()
{
    const int iterations = 1000000;
    var myObject = new SpeedTestClass();

    Stopwatch stopwatch = Stopwatch.StartNew();
    for (int i = 0; i < iterations; i++)
    {
        myObject.SpeedTestMethod();
    }
    stopwatch.Stop();
    Debug.Log($"함수를 직접 호출 : {stopwatch.ElapsedMilliseconds} ms");

    MethodInfo methodInfo = typeof(SpeedTestClass).GetMethod("SpeedTestMethod");
    stopwatch.Restart();
    for (int i = 0; i < iterations; i++)
    {
        methodInfo.Invoke(myObject, null);
    }
    stopwatch.Stop();
    Debug.Log($"Reflection을 통한 호출 : {stopwatch.ElapsedMilliseconds} ms");
}
```

동일한 함수를 직접 호출하는 방법과 Reflection을 통해 호출하는 방법, 
두 방법을 각각 백만번씩 반복했다.

그리고 그 결과는 다음과 같았다.

![image (76)](https://github.com/user-attachments/assets/6d475dfc-5940-4c80-921f-0632eb50ee46)

내 컴퓨터 환경에서는 Reflection을 통한 호출이 약 24배 느렸다.

그렇다면 Reflection은 성능이 중요한 게임 개발에는 활용방법이 없을까?

조금만 잘 생각해보면, 그래도 활용 방안이 있다. 

C#을 사용하는 Unity에서는 **에디터 확장**, **동적 로딩**, **데이터 직렬화 커스터마이징**, 
**디버깅 및 테스트 자동화** 같은 영역에 유용하게 쓰일 수 있을 것 같다.

게임 자체의 개발에 활용하기보단, 게임 개발에 도움이 되는 기능 개발에 활용도가 더 있을 것으로 보인다.

## 레퍼런스

다음은 내가 Reflection에 대해 배우고 이해할 수 있게 해준 영상이다.
항상 양질의 영상을 올려주시는 채널장분께 감사의 인사를 드린다.

[유니티 C# 고급문법 Reflection](https://www.youtube.com/watch?v=y96OaJXKM7o&list=PLO7uIclOcS0hoG3etDjye9F8jnpnL9KLj&index=19)


