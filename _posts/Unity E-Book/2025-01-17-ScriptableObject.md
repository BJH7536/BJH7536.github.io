---
title: ScriptableObjects
date:  2025-01-17 +0900
categories: [Unity E-book]
tags: [Unity, C#, Unity E-book, CreateModularGameArchitectureInUnityWithScriptableObjects, ScriptableObjects]
math: true
mermaid: true
---

[9월 11일 오후 4시 알쓸유잡 : 유니티 스크립터블 오브젝트☁️](https://www.youtube.com/watch?v=j9LUdJx0Agg&t=28s&ab_channel=UnityKorea)

> *위 글은 유니티에서 공식으로 제공하는 E Book을 기반으로 제가 번역, 공부하며 정리한 자료를 글로 남긴 것입니다.*

## Scriptable Object란 무엇인가?

- **기본적으로는 데이터 컨테이너**
    - 동일한 데이터를 다양한 객체가 복제해서 가지는 대신, 
    ScriptableObject를 참조하게 하여 불필요한 메모리 사용을 최소화할 수 있다.
        
        (Flyweight Pattern 패턴의 구현에 자주 사용된다)
        
    - MonoBehavior보다 적은 오버헤드.
        - MonoBehavior는 자기 자신과 자신이 부착되어 있는 GameObject, Transform, 그리고 GameObject의 다른 컴포넌트들까지 저장하기에 직렬화할 때에도 파일의 크기가 커진다.
- **모듈성**
    - 데이터와 로직의 분리에 유용
- **관심사 분리**
    - 게임 플레이 데이터를 동작 로직에서 분리
    - 각 독립적인 부분 테스트 및 유지 관리에 수월
    - 변경 시 오류를 줄임
- **Asset으로 저장**
    - 유니티 프로젝트에 하나의 에셋으로 저장된다. (3D Model이나 material 처럼)
    - 게임 모드 외부에서도 유지
    - 런타임에 동적으로 변경되는 정적 구성을 로드 가능
        
        (Cinemachine의 save during play 옵션의 구현에 사용됨)
        

## Scriptable Object vs MonoBehavior

### 공통점

- 둘 모두 UnityEngine의 Object를 상속받는다.

### 차이점

- MonoBehavior는 Unity로부터 다양한 Callback들을 수신한다. (Start, Awake, Update, …) 하지만 ScriptableObject는 런타임에 **Awake**, **OnEnable**, **OnDestroy**, **OnDisable**와 같은 제한된 이벤트 함수만 지원한다다. 에디터에서는 **OnValidate**와 **Reset**이 인스펙터에서 호출된다.
- MonoBehavior는 반드시 런타임에 GameObject에 부착되어있어야 하지만, ScriptableObject는 그렇지 않다. ScriptableObject는 프로젝트 내의 에셋 파일로서 존재하기 때문에, MonoBehavior와 같은 다른 스크립트에서 이를 참조해 사용할 수 있다.
- MonoBehavior는 Scene, 혹은 Prefab에만 저장될 수 있지만 ScriptableObject는 그렇지 않다.
- 에디터에서 MonoBehavior 필드의 변경은 Play Mode를 종료할 때 초기화되지만, ScriptableObject의 필드는 초기화되지 않는다.
    - 그러나 standalone 빌드에서는 런타임 도중 ScriptableObject 값의 변경은 저장되지 않는다.

그럼 지금까지 살펴본 스크립터블 오브젝트에 대해 다시 정리해보자면…

## ScriptableObject

- 런타임에 변경할 필요가 없는 모든 항목에 유용함을 가져다준다.
- 인수에 저장 / 인수로 저장
- 런타임에 동적으로 생성 / 삭제 가능
    
    ```csharp
    // 런타임에 동적으로 생성
    ScriptableObject.CreateInstance<MyScriptableObjectClass>();
    ```
    
- 메서드에서 반환
- 데이터 구조에 포함
- 런타임에 ScriptableObject의 값을 변경하면 게임 애플리케이션이 즉시 업데이트된다.
    - 애플리케이션이 실행되는 동안 변경사항을 저장할 수 있다.
    - 재생모드를 종료하면 해당 값이 그대로 유지됨.
- MonoBehavior만 사용하는 것보다 팀 협업의 측면에서 더 많은 이점을 제공한다.


> ⚠️**Destroying of ScriptableObjects**
> 다른 UnityEnigne의 Object처럼, ScriptableObject는 **C++ 네이티브 부분**과 **C# 관리 부분**으로 구성된다. 
> 네이티브 C++ 부분은 직접 제거할 수 있지만, C# 관리 부분은 **Garbage Collector**가 정리할 때까지 메모리에 남는다. 
> **GC** 정리는 씬을 변경하거나 `Resources.UnloadUnusedAssets`를 호출할 때 발생한다.
{: .prompt-info }

ScriptableObject 에셋에 대한 참조를 명시적으로 **null**로 설정해야 **Garbage Collection**이 지연되지 않는다. 특히, **`Destroy`** 또는 `DestroyImmediate`를 호출하기 전에 이 작업을 수행하는 것이 중요하다.  그렇지 않으면 에디터에서 객체가 표면적으로는 "null"로 표시되더라도 실제로는 여전히 메모리에 남아 있을 수 있다. GC 정리는 ScriptableObject에 대한 **모든 참조가 사라진 후에만** 발생한다.



## CustomEditor의 사용

MonoBehavior의 데이터를 ScriptableObject로 분리할 때, 데이터가 각기 다른 두 곳에 분리되어 저장된다. 이렇게 할 때의 장점은 지금까지 앞에서 여러번 언급했지만, 이럴 때의 단점도 분명히 존재한다. 데이터를 한 곳에서 볼 수 없어진다. 한 곳에서 모아서 볼 수는 없을까? 당연히 가능하다. CustomEditor를 사용해보자.

우선 먼저 다음과 같은 MonoBehavior가 먼저 있었다고 가정해보자.

```csharp
public class NPCHealthUnrefactored : MonoBehaviour
{
	public int maxHealth;
	public int healthThreshold;
	public NPCAIStateEnum gooldHealthAi;
	public NPCAIStateEnum lowHealthAi;

	public int currentHealth;
}
```

여기서 4개의 필드는 런타임에 변할 필요가 없는 정적인 필드다. 그래서 다음과 같이 저 필드들을 대체할 ScriptableObject를 정의하고

```csharp
[CreateAssetMenu(fileName="NPCConfig")] 
public class NPCConfigSO : ScriptableObject 
{    
	public int maxHealth;
	public int healthThreshold; 
	public NPCAIStateEnum gooldHealthAi;
	public NPCAIStateEnum lowHealthAi;
}
```

원래의 MonoBehavior를 다음과 같이 수정했다.

```csharp
public class NPCHealth: MonoBehaviour 
{    
		public NPCConfigSO config; 
    public int currentHealth;
}
```

그러나, 이렇게 단순히 기존의 필드를 ScriptableObject로 대체하기만 하면  

![Image](https://github.com/user-attachments/assets/a04c6823-e7ba-472b-b755-0e3455c0366f)

위와 같이 Unity의 InspectorView에서 각기 따로 보아야하는 불편함이 발생한다. 
기존보다 불편해지는 지점이다. 우리는 이를 CustomEditor로 보완해보자.

다음과 같이 `Editor` 클래스를 상속받는 클래스를 정의하자. `CustomEditor` Attribute를 사용해 어떤 MonoBehavior의 커스텀 에디터인지 지정해야 한다. 그리고 이 스크립트는 꼭 프로젝트의 Editor 라는 폴더에 저장되어 있어야 한다. 

`OnInspectorGUI()` 메서드에서 MonoBehaviour의 기본 인스펙터와 ScriptableObject의 인스펙터를 함께 그리도록 한다.

```csharp
using UnityEditor; 

[CustomEditor(typeof(NPCHealth))] 
public class NPCHealthEditor : Editor 
{
    private Editor editorInstance;    
    
    private void OnEnable()    
    { 
        // 에디터 인스턴스 초기화 
        editorInstance = null;
        
    }    
    public override void OnInspectorGUI()    
    { 
        // 인스펙터에 보일 컴포넌트
        NPCHealth npcHealth = (NPCHealth)target; 
        
        if (editorInstance == null) 
            editorInstance = Editor.CreateEditor(npcHealth.config); 

        // MonoBehaviour의 필드가 보이도록
        base.OnInspectorGUI(); 

        // ScriptableObjects의 인스펙터를 함께 그린다.
        editorInstance.DrawDefaultInspector();
    }
}
```

그러면 다음과 같이 MonoBehavior 스크립트의 InspectorView에서 연결된 스크립터블 오브젝트의 필드도 함께 볼 수 있다. 물론 여기서 변경한 값은 원본에 영향을 미친다.

![Image](https://github.com/user-attachments/assets/5c145b8c-bfb3-4360-84d2-3c50752666fd)

## Dual Serialization

- 유니티에서 Serialization(직렬화)/Deserialization(역직렬화)
    - Inspector에 표시됨.
    - 해당 필드는 Editor에서 쉽게 읽고 수정이 가능함.
    - vs XML, JSON
- Unity 내에서 데이터를 직렬화하는 방법을 혼합해 각 형식의 장점을 활용하자.
    - Editor에서는 ScriptableObject로 작업하면서
    - 해당 데이터를 JSON/XML로 저장하는 방식 등으로 혼합해보자
- **XML 및 JSON**
    - 에디터 내에서 작업하기는 어렵지만, Unity 외부에서 텍스트 편집기를 사용해 쉽게 수정할 수 있다.
- **스크립터블 오브젝트**
    - 에디터에서 매우 유용하며, 드래그 앤 드롭으로 빠르게 교체할 수 있지만,
    - Unity 외부에서 수정하거나 플레이어 커뮤니티와 공유하는 데는 적합하지 않다.
- 직렬화된 형식을 혼합하면 레벨 편집이나 모딩과 같은 게임에 새로운 가능성을 열어줄 수 있다.
    - 빌드 및 런타임 시 다른 파일을 ScriptableObject로 변환

위에서 설명한 XML, JSON과 ScriptableObject를 함께 사용하는 방법을 코드로 나타내면 다음과 같다.

```csharp
using System.IO; 
public class LevelManager : MonoBehaviour 
{    
	public ScriptableObject levelLayout;    
	
	public void LoadLevelFromJson(string jsonFile)    
	{ 
		if (levelLayout == null) 
		{ 
			levelLayout = ScriptableObject.CreateInstance<LevelLayout>(); 
		}    
		
		var importedFile = File.ReadAllText(jsonFile); 
		JsonUtility.FromJsonOverwrite(importedFile, levelLayout);
	}
}
```

- `ScriptableObject.CreateInstance<T>()`: 런타임에 ScriptableObject를 동적으로 생성한다/
- `File.ReadAllText()`: JSON 파일에서 텍스트 데이터를 읽어온다.
- `JsonUtility.FromJsonOverwrite()`: JSON 데이터를 읽어 ScriptableObject의 필드를 채운다.

![Image](https://github.com/user-attachments/assets/ecb54b57-edbd-40a4-bd33-bde149695ccd)

이렇게 빠르고 쾌적한 게임 플레이를 위해 유니티에서 ScriptableObject 형태로 읽어 게임에 로드하고, 이를 Export할 때는 JSON과 같은 형태를 이용하는 방식을 상상해볼 수 있겠다.

## Pattern : Extendable Enums

앞서 ScriptableObject가 데이터 컨테이너로 작동하는 방식을 보았지만, 
단순히 값이나 설정을 저장하는 것 이상의 역할을 할 수 있다. 

ScriptableObject는 Enum처럼 사용할 수 있다. 

```csharp
[CreateAssetMenu(fileName=”GameItem”)] 
public class GameItemSO : ScriptableObject { }
```

위처럼 내부가 비어있는 ScriptableObject를 활용해 마치 Enum처럼 사용하는 것이다.

![Image](https://github.com/user-attachments/assets/946a4cbc-598e-439d-9fcb-3d09fb90a2a4)

![Image](https://github.com/user-attachments/assets/66e6dee7-904a-4ba1-8da7-15af9b2feb45)

위와 같이 ScriptableObject에 대한 참조를 활용해 동등성을 확인할 수 있으며, 
이를 기반으로 특별한 데미지 효과나 가위바위보와 같은 속성을 정의할 수 있다.

ScriptableObject이므로, 당연히 다음과 같이 어떠한 기능을 추가할 수 도 있다.

```csharp
public class GameItemSO : ScriptableObject 
{    
	public GameItemSO weakness;
	public bool IsWinner(GameItemSO other)    
	{ 
		return other.weakness == this;    
	}
}
```

이렇게 하면 자신의 약점(weakness) 필드를 포함하게 된다. 
이런 식으로 가위바위보의 상성관계를 유연하게 구현할 수 있을 것이다.

```csharp
public class RockPaperScissorController : MonoBehaviour
{
	public GameItemSO MyGesture;
	
	private void JudgeWinner(GameItemSO opposite) {
		if(MyGesture.IsWinner(opposite)) {
			Debug.log("I Won!");
		}
		else {
			Debug.log("I lost...");
		}
	}
}
```

이렇게 일반적인 Enum과 달리, ScriptableObject는 Enum의 역할을 하면서도 더 많은 기능을 가질 수 있다. 

## Pattern : Delegate Objects

ScriptableObject는 지금까지 본 것처럼 단순히 데이터를 저장하는 용도로만 사용되지 않고, 로직이나 동작을 포함하는 데에도 유용하다. 그렇기에 ScriptableObject를 Delegate Object로 사용할 수 있다.

다시말해, GoF의 **Strategy Pattern**처럼 특정한 목표를 달성하기 위한 알고리즘을 캡슐화하는 도구로 사용될 수 있다는 것이다.

![Image](https://github.com/user-attachments/assets/39e97bd5-d3d6-41b3-af91-814ced65d41d)

위와 같이 EnemyUnit이라는 MonoBehaviour가 EnemyAI라는 ScriptableObject를 참조함으로써, 그 안의 메서드와 로직을 사용할 수 있다.

![Image](https://github.com/user-attachments/assets/1b262e83-5037-4de4-a7b3-fa5254c61e7c)

위와 같이 ScriptableObject를 상속받는 추상 클래스를 만들고, 이를 상속받는 ScriptableObject를 활용해 **Strategy Pattern**을 구현할 수 있다.  마치 플러그를 간편하게 꽂기도, 뽑기도 하는 것처럼 ScriptableObject를 교체함으로써 MonoBehaviour의 행동을 다양하게 분화시킬 수 있다.

이러한 방식은 SOLID의 **개방-폐쇄 원칙(Open-Closed Principle)**을 지키면서, 
코드베이스를 더욱 **확장 가능하게 유지**하는 데 도움이 된다.

이 주제에 대한 예시는 유니티에서 제공하는 데모와 Unity Korea의 영상에서 더욱 자세히 설명해주고 있으니, 필요하다면 해당 페이지를 확인해보는 것이 좋을 것 같다.

https://github.com/UnityTechnologies/PaddleGameSO

https://youtu.be/j9LUdJx0Agg?t=2857

## Pattern : Observer

싱글톤(Singleton) 패턴에 대한 논쟁은 자주 일어난다. 규모가 작은 프로젝트나 프로토타입에서는 효과적일 수도 있으나, 프로젝트의 규모가 커질수록 싱글톤의 단점이 이점을 넘어서는 경우가 많아진다. 그래서 싱글톤은 많은 개발자들에게 **안티패턴**으로 간주되기도 한다.

ScriptableObject는 이 싱글톤에 의존하지 않을 수 있는 대안을 제시해준다.

- 공유 데이터에 쉽게 접근하고자 한다면, **ScriptableObject 기반 Runtime Set**을 고려해보자
- 객체 간 메세지 전달이 필요하다면, **ScriptableObject 기반 이벤트 채널**을 사용해보자.

싱글톤에서 벗어난 구조로 전체 게임을 설계하면 높은 **확장성**과 **테스트 가능성**을 기대할 수 있다.

![Image](https://github.com/user-attachments/assets/9e9509ce-5fcf-4d5d-9301-04ed74c33a35)

기존에 Observer Pattern을 구현한다면 위와 같은 방식으로 MonoBehaviour들이 서로를 참조하게끔 할 수 있다. 이런 방식으로 개발했을 때의 단점은 비개발자 직군의 팀원과 협업하기 어렵다는 것이다. 

기존의 Subject에 대한 Observer를 추가할 때나, 반대로 Subject를 추가할 때, Observer의 Response를 변경하거나 새로 지정하는 등의 모든 기능을 코드로 작업해야 하기 때문에, 비개발자에게 불친절한 방식이다.

![Image](https://github.com/user-attachments/assets/10c40b9e-28dc-4a41-b043-de96501ba124)

그에 대한 대안으로 위와 같이 Subject와 Observer 사이에 ScriptableObject가 EventChannel로써 중계하게 만드는 방법이 있다. 이는 Subject와 Observer간의 결합도를 낮추는 방법이 되어주며, UnityEngine 자체에서 GUI를 통해 Subject와 Observer의 관계를 관리할 수 있어 더욱 직관적인 접근 방식을 제공한다.

ScriptableObject는 프로젝트 내에서 에셋으로 존재하기 때문에, 프로젝트의 어느 부분에서든 지속적인 접근이 가능하며, 싱글톤을 사용할 때 발생하는 불필요한 의존성을 방지할 수 있다. 

ScriptableObject를 활용해 Observer Pattern을 구현한다면 다음과 같이 구현할 수 있겠다.

```csharp
[CreateAssetMenu(menuName = "Events/Void Event Channel")] 
public class VoidEventChannelSO : ScriptableObject 
{
	public event UnityAction OnEventRaised;
	public void RaiseEvent() 
	{ 
		if (OnEventRaised != null) 
			OnEventRaised.Invoke(); 
	} 
}
```

우선 ScriptableObject는 이렇게 간단하게 구현한다. 

그리고 Observer들은 Response에 해당하는 자신의 메서드를 `OnEventRaised`에 연결하고, Subject는 `RaiseEvent()` 를 Call해주면 된다.

그러면 기존의 방법에 비해 Observer들과 Subject는 서로에 대한 참조 대신, 이 ScriptableObject 에셋을 참조하게 된다. 그만큼 서로간의 결합도를 낮추는 것이다.

이런 방식으로 ScriptableObject는 게임 내에서 오디오 관리, 씬 관리, UI 관리, 데이터 저장 관리 등 과 같은 다양한 event의 중계에 사용될 수 있다. 

![Image](https://github.com/user-attachments/assets/312b5e0a-15fe-41a9-8b3d-c018af8ae740)

컨트롤러 입력 이벤트를 ScriptableObject로 관리하는 것도 가능하다. Unity의 **Input System**은 Started, Performed, Canceled와 같은 종류의 InputAction들을 관리하는데, 우리는 이러한 InputAction 바인딩을 해석하기 위해, 특화된 InputReader ScriptableObject를 활용할 수 있다.

![Image](https://github.com/user-attachments/assets/5b489900-c8d0-400d-becb-8650d382fa0f)

Unity의 Input System이 Subject 또는 브로드캐스터(broadcaster) 역할을 하고, 우리는 여기에 특별한 기능을 연결할 때, 그 중계를 위한 ScriptableObject(InputReader)를 활용할 수 있다는 것이다.

![Image](https://github.com/user-attachments/assets/794198ac-0785-4e75-b8b2-ab6fc89014d5)

당연하게도, 이러한 구조는 간단한 게임보다는 프로젝트의 규모가 커질수록 빛을 발한다.

이러한 구조는 **입력을 GameObject와 분리**함으로써 **유연성**과 **재사용성**을 향상시킬 수 있으며,

개발 중에 **InputActions**를 수정해야 할 경우, **InputReader**만 유지 관리하면 되는 장점이 있다.

이벤트가 변경되지 않는 한, 입력을 청취하는 객체들은 영향을 받지 않아 입력과 옵저버 간의 연결을 유지하는 작업이 훨씬 줄어들며, 특히 많은 옵저버가 있을 때 유지보수가 훨씬 용이해진다.

## Pattern : Command

Command Pattern은 요청을 객체로 캡슐화함으로써 명령을 실행하는 객체(Receiver)와 명령을 내리는 객체(Invoker)를 분리하는 패턴이다. 단일 책임 원칙(Single Responsibility Principle)과 개방/폐쇄 원칙(Open/Closed Principle)을 만족하기 좋은 패턴이다. 다양한 게임에서 활용되기 좋은 패턴이다. RTS, 퍼즐, 등등…

Command Pattern 또한 ScriptableObject를 활용해서 구현할 수 있다.

![Image](https://github.com/user-attachments/assets/493b65c5-c883-480f-b2a3-d52b28cd2738)

Command가 필수적으로 가져야 할 동작을 인터페이스로 정의하고, 이를 ScriptableObject와 함께 상속받아 새로운 Command를 정의하면 된다. 

이렇게 구현하면 앞서 언급한 ScriptableObject를 활용한 이벤트 채널과 함께 사용할 수도 있겠다.

## Pattern : Runtime Sets

유니티로 게임을 개발하다보면, 런타임 중에 Scene에 있는 GameObject나 컴포넌트 목록을 추적해야 할 때가 생긴다. ScriptableObject를 활용하면, 앞서 언급한 것처럼 싱글톤의 대안으로 활용할 수도 있으면서 `Object.FindObjectOfType` 또는 `GameObject.FindWithTag` 와 같이 비용이 큰 Find 계열 연산의 사용을 피할 수 있다.

![Image](https://github.com/user-attachments/assets/a1aa0131-5236-45cd-96b5-0f1a49b44eb3)

다음과 같이 ScriptableObject로 GameObject들을 추적하는 간단한 Runtime Set을 구현할 수 있다.

```csharp
using System.Collections.Generic; 
using UnityEngine; 

[CreateAssetMenu(menuName = "GameObject Runtime Set", fileName = "GORuntimeSet")] 
public class GameObjectRuntimeSetSO : ScriptableObject 
{    
	private List<GameObject> items = new List<GameObject>();    
	public List<GameObject> Items => items; 
	   
	public void Add(GameObject thingToAdd)    
	{ 
		if (!items.Contains(thingToAdd)) 
			items.Add(thingToAdd);    
	} 

	public void Remove(GameObject thingToRemove)    
	{ 
		if (items.Contains(thingToRemove)) 
			items.Remove(thingToRemove);    
	}
}
```

기존에는 MonoBehaviour이 무언갈 직접 참조했다면, ScriptableObject에게 이 역할을 떠넘기는 방법이라고 이해할 수 있겠다. 

![Image](https://github.com/user-attachments/assets/f26f0dd1-bdfe-416b-aeb6-ff5ea61092b0)

ScriptableObject이 추적하는 대상에 대한 참조를 가지고 있으므로, 해당 ScriptableObject를 이벤트 채널로 사용하는 것 또한 가능해진다. 이렇게 함으로써 기존의 의존성을 약화시키고 유지보수와 디버깅을 쉽게 만든다. 

제네릭을 활용해 특정 타입의 객체에 대한 RuntimeSet 을 만들 수 있게, Base 클래스 또한 다음처럼 구현할 수 있겠다.

```csharp
public abstract class RuntimeSetSO<T> : ScriptableObject    
{ 
	[HideInInspector] public List<T> Items = new List<T>(); 
	public void Add(T thing) 
	{ 
		if (!Items.Contains(thing)) 
			Items.Add(thing); 
	} 
	
	public void Remove(T thing) 
	{ 
		if (Items.Contains(thing)) 
			Items.Remove(thing); 
	}    
}
```

## Conclusion

내가 참조한 E-북, 그리고 Unity Korea의 영상에서도 말하는 것처럼, ScriptableObject가 만능은 아니다. 디자인패턴처럼, 이를 적용하는 것이 적용하지 않는 것만큼이나 중요하다. 진행하는 프로젝트의 규모, 혹은 팀의 특징 등등 다양한 점들을 고려해야 한다. 그렇기 때문에, 이것을 사용하고 말고는 본인이 잘 판단해 사용해야 한다. 

지금까지의 글은 나 스스로의 공부를 위해 쓴 글이다보니, 혹시 이 글을 읽는 분이 계시다면 꼭 원본도 확인해보시는 걸 추천한다. 그리고 공식 데모도.