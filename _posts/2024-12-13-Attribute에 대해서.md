---
title: Attribute에 대해서
date: 2024-12-13 +0900
categories: [C#]
tags: [C#, Attribute]
math: true
mermaid: true
---

## Attribute

C# 코드에 추가할 수 있는 메타 데이터

### 메타 데이터

- 코드 자체에 대한 정보
- 데이터 안의 데이터로 Attribute, Reflection을 통해 얻는 정보

![image (86)](https://github.com/user-attachments/assets/695ac4cc-a0bf-4c60-a509-2e2816f4bdf3)

## [Obsolete]

더 이상 사용하지 않는 코드에 대해 경고를 남길 때 사용한다.

![image (77)](https://github.com/user-attachments/assets/2fefa175-3f8d-4b42-b022-8bb85a7c81a7)

![image (78)](https://github.com/user-attachments/assets/7ecac7e9-aeed-484e-aca1-c6cc47ba13f0)


위와 같이 IDE에서 경고를 인식할 수 있도록 도와주기도 하며,

```csharp
[Obsolete("이 메서드는 더 이상 사용하지 않는다.", true)]
```

위와 같이 두번째 인자로 true를 집어넣으면, 이 요소를 사용한 부분에 컴파일 오류로 처리하게 된다.

![image (79)](https://github.com/user-attachments/assets/1e28e0ed-a321-4c73-99cd-4156dd667088)

## [SerializeField]

Unity에서 비공개(private) 필드에 직렬화(Serialization)를 허용하여 Inspector View**에서 노출**하고 싶을 때 사용한다. 이에 대해서는 잘 알려주는 곳이 많으니 패스.

## 그 밖에 유니티에서 자주 사용되는 Attribute들

### [AddComponentMenu]

- 이 스크립트를 GameObject에 컴포넌트로 추가할 때 메뉴에서 이 스크립트를 찾는 경로를 지정한다. *string 타입의 매개변수의 사용이 필수적이다.*
- /로 디렉토리 계층을 만들 수 있다.
- 이 매개변수의 값이 GameObject에 컴포넌트로 붙을 때의 이름에 영향을 미친다.
    
    ![image (9)](https://github.com/user-attachments/assets/6c795ef2-b1af-49cc-933a-56139ca2e540)
    
    이렇게 [AddComponentMenu] 를 작성해주면
    
    ![image (12)](https://github.com/user-attachments/assets/5a091b6a-4f0d-4161-818e-f95e36694d3a)
    
    ![image (13)](https://github.com/user-attachments/assets/5dace638-fb40-4211-88bc-df5b5ef98c06)
    
    지정한 경로를 통해 GameObject에 컴포넌트를 붙일 수 있다.
    

### [UnityEditor.MenuItem]

- 유니티 에디터의 상단에 메뉴를 만들고, 해당 메뉴를 메서드와 연결한다. *string 타입의 매개변수의 사용이 필수적이다.*
- static 메서드만 연결 가능하다.
    
    ![image (14)](https://github.com/user-attachments/assets/641b6b15-058b-454f-924e-c3de39afd840)
    
    위와 같이 작성해주면 
    
    ![image (15)](https://github.com/user-attachments/assets/1ba3d64c-ad5a-462f-a879-f702da2af8fd)
    
    이렇게 상단 메뉴에서 해당 메서드를 실행시킬 수 있다.
    

### [ContextMenu]

- GameObject에 스크립트가 컴포넌트로 부착되어있을 때, InspectorView에서 해당 함수를 호출할 수 있는 방법을 제공한다. *string 타입의 매개변수의 사용이 필수적이다.*
    
    ![image (16)](https://github.com/user-attachments/assets/4c0fc717-45e2-43c4-a746-a1a71de0871f)
    
    위와 같이 작성하면
    
    ![image (17)](https://github.com/user-attachments/assets/2545492a-b9f5-4cf4-882f-40239f2c61ac)
    
    이렇게 InspectorView에서 해당 함수를 바로 실행시킬 수 있다.
    

### [ContextMenuItem()]

- InspectorView에 드러나는 필드에서 바로 특정한 함수를 실행할 수 있도록 연결하는 Attribute
- 필드의 바로 위에서 선언하며, 두 개의 string 매개변수를 필요로한다.
    - 첫 번째 string으로 필드에서 함수를 연결할 이름을 지정하고
    - 두 번째 string으로 연결할 함수의 이름을 지정한다.
    
    ![image (18)](https://github.com/user-attachments/assets/9cf13631-176b-4d8c-aa15-3e4edc3ecc0d)
    
    위와 같이 Value라는 필드에 ResetValue 함수와 RandomValue 함수를 연결했다.
    
    ![image (19)](https://github.com/user-attachments/assets/f6aa0df7-f770-4b68-baa0-21458a053a16)
    
    InspectorView에서 해당 필드에 마우스 커서를 올려놓고 우클릭하면 위와 같이 해당 함수를 실행시킬 수 있다.
    

### [Tooltip()]

- InspectorView 에서 해당 필드에 마우스 커서를 올려놓으면 나타날 툴팁의 내용을 지정하는 Attribute
- string 타입의 인자로 툴팁의 내용을 지정한다.
    
    ![image (20)](https://github.com/user-attachments/assets/1f713d66-746f-45b8-bd0c-8769d82a6843)
    

### [HelpURL()]

![image (21)](https://github.com/user-attachments/assets/9d3084fa-9622-4f56-a63a-2bd5ca8c9012)

- 스크립트가 컴포넌트로 부착되어있는 상태에서 물음표 버튼을 클릭해 특정한 웹페이지를 열어주는 Attribute
- 해당 스크립트에 대한 설명이 있는 웹페이지를 연결해주는 용도로 사용할 수 있겠다.
- string 타입의 인자에는 연결하고자 하는 웹페이지의 주소를 넣어주면 된다.

### [ColorUsage()]

- UnityEngine.Color 타입 필드가 InspectorView에 보일 때, Color의 alpha 값 사용 여부를 조절할 수 있는 Attribute.
- 첫번째 인자로 false를 사용하면 Color 필드의 값을 에디터에서 조절할 때 Alpha 값을 나타나지 않는다.
- 두번째 인자는 HDR 표시 여부를 나타낸다.
    
    ![image (22)](https://github.com/user-attachments/assets/3eca1e67-4925-4ae9-92f6-73f11e936f78)

    위는 [ColorUsage(false, true)] 로 선언한 결과.
    

### [Header()]

- InspectorView에 보이는 필드들을 묶는 데 사용되는 Attribute
- Header는 Bold로 표시된다.
    
    ![image (23)](https://github.com/user-attachments/assets/586dba34-e7bb-4ba9-950f-b2c24b2fd1c9)

### [Space()]

- InspectorView에 보이는 요소들 사이의 수직 여백을 추가하는 Attribute
- 인자로 int 값을 넣어 수직 여백의 길이를 조절할 수 있다.
    
    ![image (24)](https://github.com/user-attachments/assets/e4a92f6d-ff48-4a12-a936-78ec136d54cc)
    
    크기 20의 수직 여백을 넣은 모습 
    

### [MultiLine()]

- InspectorView에 string 타입 필드의 입력칸을 위 아래로 늘리는 Attribute
    
    ![image (25)](https://github.com/user-attachments/assets/fef9060b-2e72-46ec-80f7-c9bbdd372c07)
    
    [Multiline(5)] 로 설정한 모습
    

### [TextArea()]

- string 타입의 필드에 쓰인다.
- 두 개의 int 타입 인자를 받는다. 각각 최소, 최대 열의 수를 의미한다.

### [ExecuteInEditMode]

- Play 모드에 진입하지 않고도 해당 스크립트가 동작하게 만드는 Attribute
- 클래스의 선언 직전에 쓰인다.

### [RequireComponent()]

- 이 선언이 사용된 스크립트의 컴포넌트를 붙일 때, 여기서 지정한 컴포넌트가 함께 붙도록 만든다.
- 클래스의 선언 직전에 쓰인다.
- 예를 들어 RigidBody를 지정할 때는 `[RequireComponent(typeof(Rigidbody))]` 와 같이 지정하면 된다.
- 스크립트의 동작에 반드시 필요한 다른 컴포넌트가 동일한 GameObject에 있는 것을 보장할 수 있는 Attribute.

### [DisallowMultipleComponent]

- 한 GameObject에 동일한 스크립트의 컴포넌트를 여러 개 붙이지 않도록 해주는 Attribute.
- 클래스의 선언 직전에 쓰인다.

### [System.NonSerialized]

- 필드의 직렬화를 푸는 Attribute
- 직렬화가 풀려, 해당 필드가 InspectorView에도 보이지 않게 된다.

### [HideInInspector]

- 필드가 InspectorView에서 보이지 않도록 감추는 Attribute.
- 직렬화는 유지한다.


## 사용자 정의 Attribute

```csharp
using System;
using System.Reflection;
using UnityEngine;

public class MyCustomAttribute : Attribute
{
    public string Desc { get; }

    public MyCustomAttribute(string desc)
    {
        Desc = desc;
    }
}

public class MyTestClass
{
    [MyCustomAttribute("테스트용 메서드")]
    public void TestMethod()
    {
        Debug.Log("테스트 메서드 실행");
    }
}

public class AboutAttribute : MonoBehaviour
{
    private void Start()
    {
        TestMethod();
    }

    public void TestMethod()
    {
        Type myTestClass = typeof(MyTestClass);
        foreach (var methodInfo in myTestClass.GetMethods())
        {
            var attribute = (MyCustomAttribute)methodInfo.GetCustomAttribute(typeof(MyCustomAttribute));

            if (attribute != null)
            {
                Debug.Log(attribute.Desc);
                methodInfo.Invoke(Activator.CreateInstance(typeof(MyTestClass)), null);
            }
        }
    }
}
```

위 코드는 Reflection을 이용하는 코드다. 위 코드의 동작을 설명하자면 

`MyTestClass`의 타입을 추출해 해당 클래스의 메서드 목록에 대해 순회한다. `MyCustomAttribute` 애트리뷰트가 있는 메서드라면, 애트리뷰트의 `Desc`를 콘솔에 출력하고 `MyTestClass` 인스턴스를 만들어 해당 메서드를 실행시킨다.

- `GetCustomAttribute()`
    - 지정한 타입의 애트리뷰트를 해당 멤버(메서드, 클래스 등)에서 가져오는 메서드

그 결과는 다음과 같다.

![image (80)](https://github.com/user-attachments/assets/ff3296b0-f346-4613-b20d-086c963059e7)

## Attribute의 활용방안

```csharp
public class AboutAttribute : MonoBehaviour
{
    public Rigidbody rigid;
    public BoxCollider coll;
    public AudioSource audi;

    private void Start()
    {
        Transform tr1 = Util.FindChild("Target1", transform);
        rigid = tr1.GetComponent<Rigidbody>();
        coll = tr1.GetComponent<BoxCollider>();

        Transform tr2 = Util.FindChild("Target2", transform);
        audi = tr2.GetComponent<AudioSource>();
    }
}
```

위와 같은 스크립트를 지금 작성 중에 있다고 하자. 컴포넌트에 대한 레퍼런스가 비어있는 상태에서, 코드로 레퍼런스를 채워주는 기능을 지금 작성 중에 있다. 참고로 `Util.FindChild` 함수에 대한 정의는 다음과 같다.

```csharp
public class Util
{
    /// <summary>
    /// 게임 오브젝트의 Transform을 찾는 재귀 함수
    /// </summary>
    /// <param name="name">타겟 이름</param>
    /// <param name="tr">시작 위치</param>
    /// <returns>찾은 Transform</returns>
    public static Transform FindChild(string name, Transform tr)
    {
        if (tr.name == name)
            return tr;

        for (int i = 0; i < tr.childCount; i++)
        {
            Transform findTr = FindChild(name, tr.GetChild(i));
            if (findTr != null)
                return findTr;
        }
        return null;
    }
    
    // ...
    
}
```

Hierarchy는 다음과 같은 상황이다. 

![image (81)](https://github.com/user-attachments/assets/480c9e2c-5a4c-4fd4-8e89-f0e27faa4333)

Root에는 위 스크립트가, Target1에는 BoxCollider와 Rigidbody가, 그리고 Target2에는 AudioSource가 있다. 

![image (82)](https://github.com/user-attachments/assets/ce834c62-5853-4116-a197-cc7a7ff9b8bf)

Root의 스크립트에는 이렇게 필드의 값이 비어있는 상태다.

이제 Play 모드에 진입하면 스크립트의 필드가 알아서 채워질 것이다. 그런데 스크립트에서 이렇게 참조해야 할 컴포넌트가 늘어나면 어떻게 될까? 참조할 컴포넌트의 수가 늘어나면 늘어날수록, `AboutAttribute.Start`메서드는 덩치가 점점 커질 것이다. 그렇게 주구장창 길어지는 걸 어떻게 해결할 수 없을까?

Attribute를 통해, 몇몇 부분을 자동화해보자.

Attribute에 특정한 값을 저장할 수 있다는 점에 집중해보자.

```csharp
[AttributeUsage(AttributeTargets.Field)]
public class FindComponentAttribute : Attribute
{
    public string _gameObjectName { get; }

    public FindComponentAttribute(string gameObjectName)
    {
        _gameObjectName = gameObjectName;
    }
}
```

위 코드에서 `AttributeUsage`를 볼 수 있는데, 그 자세한 정보는 다음과 같다.

### AttributeUsage

- Custom Attribute를 정의할 때 Attribute를 적용할 수 있는 대상 (메서드, 클래스) 과 사용 규칙을 정의한다.
- **주요 속성**
    - **`AttributeTargets`**
        
        Attribute 를 적용할 수 있는 코드 요소를 지정한다. 
        
        `AttributeTargets` 열거형의 플래그 조합 가능
        
        ![image (83)](https://github.com/user-attachments/assets/3d598879-99f5-490e-bb90-0cbf8fa77e33)
        
        해당 어트리뷰트가 상속되는지 여부를 설정
        
        - `true`: 자식 클래스에 상속됨.
        - `false`: 상속되지 않음.
    - **`AllowMultiple`**
        
        동일한 요소에 어트리뷰트를 여러 번 적용할 수 있는지 여부를 설정
        
        - `true`: 여러 번 적용 가능.
        - `false`: 한 번만 적용 가능.

그리고 `Util` 클래스에 다음의 함수가 더 있다고 가정해보자

```csharp
public class Util
{
	
	// ...
	
	public static void InjectComponents(object o)
	{
	    Type type = o.GetType();
	    MonoBehaviour script = o as MonoBehaviour;
	
	    FieldInfo[] fields = type.GetFields(BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance);
	
	    foreach (var field in fields)
	    {
	        var attribute = (FindComponentAttribute)field.GetCustomAttribute(typeof(FindComponentAttribute));
	
	        Type fieldType = field.FieldType;
	        Transform tr = FindChild(attribute._gameObjectName, script.transform);
	
	        Component component = tr.GetComponent(fieldType);
	        
	        field.SetValue(script, component);
	    }
	}

	// ....
	
}
```

그러면, 앞서 작성했던 AboutAttribute 컴포넌트는 다음과 같은 방식으로 수정할 수 있다. 물론 이렇게 해도 동작은 기존과 동일하다.

```csharp
public class AboutAttribute : MonoBehaviour
{
    [FindComponent("Target1")] public Rigidbody rigid;
    [FindComponent("Target1")] public BoxCollider coll;
    [FindComponent("Target2")] public AudioSource audi;

    private void Start()
    {
        Util.InjectComponents(this);
    }
}
```

`Util.InjectComponents` 함수는 C#의 Reflection을 이해한다면 이해할 수 있는 코드다. 

이렇게 수정한다면, `AboutAttribute`의 필드가 늘어난다해도 그에 맞춰 코드를 추가적으로 짤 필요가 사라지고 `Start`에서는 `Util.InjectComponents(this);` 한줄로 모든 필드에 대해 대응할 수 있게 된다. 대신 필드마다 `FindComponentAttribute`가 붙긴 해야 한다.

위 코드를 한번 업그레이드 해보자. `params` 키워드를 이용해 배열타입의 필드에 대해서도 적용할 수 있도록 개선해보자.

- **`params`**
    - 메서드가 정해지지 않은 개수의 인수를 받을 수 있게 하는 키워드.
    - 배열 타입 앞에 사용되며, 이를 통해 여러개의 값을 배열로 처리할 수 있다.

`FindComponentsAttribute` 를 다음과 같이 새롭게 정의해보자. 기존의 `FindComponentAttribute` 를 대신할 수 있다.

```csharp

[AttributeUsage(AttributeTargets.Field)]
public class FindComponentsAttribute : Attribute
{
    public string[] _gameObjectNames { get; }

    public FindComponentsAttribute(params string[] gameObjectNames)
    {
        _gameObjectNames = gameObjectNames;
    }
}
```

그리고 Util 클래스의 `InjectComponents` 메서드를 수정해주자. 

```csharp
public static void InjectComponents(object o)
{
    Type type = o.GetType();
    MonoBehaviour script = o as MonoBehaviour;

    FieldInfo[] fields = type.GetFields(BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance);

    foreach (var field in fields)
    {
        var attribute = (FindComponentsAttribute)field.GetCustomAttribute(typeof(FindComponentsAttribute));

        Type fieldType = field.FieldType;

        if (fieldType.IsArray)      // 배열인 경우
        {
            Type elementType = fieldType.GetElementType();

            List<Component> componentsList = new List<Component>();

            foreach (var gameObjectName in attribute._gameObjectNames)
            {
                Transform tr = FindChild(gameObjectName, script.transform);

                Component component = tr.GetComponent(elementType);
                
                componentsList.Add(component);
            }

            Array componentArray = Array.CreateInstance(elementType, componentsList.Count);
            for (int i = 0; i < componentsList.Count; i++)
            {
                componentArray.SetValue(componentsList[i], i);
            }
            
            field.SetValue(script, componentArray);
        }
        else                        // 배열이 아닌 경우
        {
            Transform tr = FindChild(attribute._gameObjectNames[0], script.transform);

            Component component = tr.GetComponent(fieldType);
        
            field.SetValue(script, component);
        }
    }
}
```

그러면 다음과 같이 필드가 배열 타입인 경우에도 사용할 수 있게 된다.

```csharp
[FindComponents("Target1")] public Rigidbody rigid;
[FindComponents("Target1")] public BoxCollider coll;
[FindComponents("Target2")] public AudioSource audi;
[FindComponents("Target3", "Target4")] public AudioSource[] audiArray = new AudioSource[2];

private void Start()
{
    Util.InjectComponents(this);
}
```

![image (84)](https://github.com/user-attachments/assets/45744dc9-d271-4f07-8df0-ce216afec78e)

위와 같이 Target3와 Target4에 있는 AudioSource 컴포넌트를 찾아 필드에 할당해준다.

필드가 배열 타입일 때를 고려했으니, `List`타입일 때도 고려해 볼 수 있겠다. 

`List`일때도 동작하게 해보자.

```csharp
public static void InjectComponents(object o)
{
    Type type = o.GetType();
    MonoBehaviour script = o as MonoBehaviour;

    FieldInfo[] fields = type.GetFields(BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance);

    foreach (var field in fields)
    {
        var attribute = (FindComponentsAttribute)field.GetCustomAttribute(typeof(FindComponentsAttribute));

        Type fieldType = field.FieldType;

        // 배열인 경우
        if (fieldType.IsArray)
        {
            Type elementType = fieldType.GetElementType();

            List<Component> componentsList = new List<Component>();

            foreach (var gameObjectName in attribute._gameObjectNames)
            {
                Transform tr = FindChild(gameObjectName, script.transform);

                Component component = tr.GetComponent(elementType);
                
                componentsList.Add(component);
            }

            Array componentArray = Array.CreateInstance(elementType, componentsList.Count);
            for (int i = 0; i < componentsList.Count; i++)
            {
                componentArray.SetValue(componentsList[i], i);
            }
            
            field.SetValue(script, componentArray);
        }
        // List인 경우
        else if (fieldType.IsGenericType && fieldType.GetGenericTypeDefinition() == typeof(List<>))
        {
            Type elementType = fieldType.GetGenericArguments()[0];

            IList componentsList = (IList)Activator.CreateInstance(fieldType);

            foreach (var gameObjectName in attribute._gameObjectNames)
            {
                Transform tr = FindChild(gameObjectName, script.transform);

                Component component = tr.GetComponent(elementType);
                
                componentsList.Add(component);
            }
            
            field.SetValue(script, componentsList);
        }
        // 배열이 아닌 경우
        else
        {
            Transform tr = FindChild(attribute._gameObjectNames[0], script.transform);

            Component component = tr.GetComponent(fieldType);
        
            field.SetValue(script, component);
        }
    }
}
```

`Util` 클래스의 `InjectComponents` 메서드만 위와 같이 수정해주자. 기존에 배열 타입, 단일 타입만 고려했던 것에 List타입에 대해서도 동작하게 경우의 수를 추가했다.

```csharp
[FindComponents("Target1")] public Rigidbody rigid;
[FindComponents("Target1")] public BoxCollider coll;
[FindComponents("Target2")] public AudioSource audi;
[FindComponents("Target3", "Target4")] public AudioSource[] audiArray = new AudioSource[2];
[FindComponents("Target5", "Target6")] public List<Rigidbody> rigidList = new List<Rigidbody>();

private void Start()
{
    Util.InjectComponents(this);
}
```

그러면 위와 같이 List타입의 필드에 대해서도 적용되는 것을 볼 수 있다.

![image (85)](https://github.com/user-attachments/assets/c5ed9ffd-9ac0-4501-8609-1f0041132d3c)

## 레퍼런스

다음은 내가 Attribute에 대해 배우고 이해할 수 있게 해준 영상이다. 항상 양질의 영상을 올려주시는 채널장분께 감사의 인사를 드린다.

[유니티 C# 고급문법 Attribute](https://youtu.be/x0CayxlgKPE?si=R0P0I4rVSpASEjKp)