---
title: Singleton Pattern 싱글톤 패턴
date: 2024-08-13 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Singleton Pattern

- **GoF 정의** : ***“오직 한개의 클래스 인스턴스만을 갖도록 보장하고, 이에 대한 전역적인 접근점을 제공한다.”***
- 유니티에서는 `DontDestroyOnLoad()` 함수를 이용해 구현한다.
    - 유니티에서는 Scene 개념을 활용해 다양한 환경이나 상태를 구성한다. 이 때, Scene은 그 기본 단위가 되기에 Scene을 변화시키면 그 Scene에 설정된 환경도 함께 변한다.
    - 이렇게 Scene을 전환할 때, 활성화되어있던 GameObject들은 남겨지면서 비활성화되는데, GameObject가 Scene 전환의 영향을 받지 않도록 하는 것이 바로 `DontDestroyOnLoad()` 함수.
- 싱글톤 객체는 `일종의 전역변수와 유사한 성격`을 갖는다.
    - 모든 영역에서 접근 가능하기 때문에, 객체의 변경 시점과 그 주체를 알기 쉽지 않다.
    - 여러 클래스와 `Coupling`된다는 단점이 존재한다.
    - 멀티 쓰레드 환경에서는 모든 곳에서 접근 가능하기 때문에, ***Race Condition***이 발생할 수 있다.
        - ***Race Condition :*** 여러 프로세스 또는 스레드가 데이터에 동시에 접근하고 이를 변경하려 할 때, 그 최종 결과가 실행 순서에 따라 달라질 수 있는 상황.

```csharp
public class Manager : Monobehavior
{
	static Manager _instance = null;
	public static Manager Instance => _instance;

	void Awake() 
	{
		if(_instance = null)
		{
			_instance = this;
			DontDestroyOnLoad(this.gameObject);
		}
	}
}
```