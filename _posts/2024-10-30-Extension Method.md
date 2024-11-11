---
title: Extension Method
date: 2024-10-30 +0900
categories: [C#]
tags: [C#, Extension Method]
math: true
mermaid: true
---
기존에 존재하는 클래스의 기능을 확장할 때는 상속을 이용한다. 하지만, 일부 특별한 조건에서는 상속이 좋지 못한 선택일 수 있다. 그럴 때 선택 가능한 것이 바로 **확장 메서드(Extension Method)**이다.

## 사용되는 경우

1. 상속으로 기능 확장이 어려운 경우에 사용된다.
    - **상속하고자 하는 클래스의 정의에 `sealed` 가 사용된 경우.**
        
        **`sealed` 키워드** 
        해당 클래스를 더 이상 상속할 수 없도록 만드는 데 사용되는 키워드
        `public sealed class ... { ... }`  
        
        ![image (23)](https://github.com/user-attachments/assets/710791d1-8c5e-42f6-b454-fcd99c6645cc)
        
        sealed class인 string을 상속받고자 할 때 나타나는 메세지
        
2. 상속으로 기능을 확장하면 코드 수정이 필요하다.
    - **이를 설명하는 예제**
        
        ```csharp
        public class Player
        {
            public void Move()
            {
                Debug.Log("Player is moving");
            }
        }
        
        public class AdvancedPlayer : Player
        {
            public void Jump()
            {
                Debug.Log("Player is jumping");
            }
        }
        
        private void Start()
        {
            // 기본 메서드는 Player 클래스를 사용
            Player player = new Player();
            player.Move();
            
            // 새로운 기능인 Jump를 사용하려면 AdvancedPlayer로 코드 변경이 필요
            AdvancedPlayer advancedPlayer = new AdvancedPlayer();
            advancedPlayer.Move();
            advancedPlayer.Jump();
        }
        ```
        
        AdvancedPlayer 클래스는 Player 클래스를 상속받고 Jump 함수가 추가적으로 구현한다.
        
        이렇게 추가적으로 구현된 기능을 사용하려면, 기존에 Player 클래스로 작성된 코드를 모두 AdvancedPlayer로 고쳐야 한다. 
        

## 사용법

![image (24)](https://github.com/user-attachments/assets/54551acb-2e70-4425-b7e3-89e98ee2f798)

`static class` 내의 `static method`로 정의한다. method의 첫 번째 매개변수가 중요하다.

`this 확장할타입 식별자` 와 같은 인자가 method의 signature에 포함되어 있어야 한다.

## 사용 예시 1

```csharp
public class Player
{
    public void Move()
    {
        Debug.Log("Player is moving");
    }
}

public static class AdvancedPlayer
{
    public static void Jump(this Player player)
    {
        Debug.Log("Player is jumping");
    }
}

public class ExtensionMethodTest : MonoBehaviour
{
    private void Start()
    {
        Player player = new Player();
        player.Move();
        player.Jump();
    }
}
```

`Player` 클래스에서 `Move` 함수만 정의하고, `Jump` 함수는 정의하지 않았다.

그러나 위처럼 확장 메서드를 활용하면 `Player` 인스턴스가 `Jump` 함수를 자신의 메서드처럼 Call 할 수 있다. 

- 하지만 실제로는 정적 메서드로 컴파일되는 것을 알아두자.
    - `player.Jump();`가 실제 내부적으로는 `PlayerExtensions.Jump(player);` 와 같이 컴파일된다.
    - 이렇게 확장 메서드는 단순히 매개변수로 클래스의 인스턴스를 전달하는 방식으로 동작하며, 클래스의 메서드를 변경하는 것이 아니다.

## 사용 예시 2

```csharp
public static class TransformExtension
{
    // Transform의 위치, 회전, 스케일을 초기화하는 확장 메서드
    public static void ResetTransform(this Transform transform)
    {
        transform.position = Vector3.zero;
        transform.rotation = Quaternion.identity;
        transform.localScale = Vector3.one;
    }
}
public class ExtensionMethodTest : MonoBehaviour
{
    private void Start()
    {
        Transform myTransform = transform;
        
        // myTransform.position = Vector3.zero;
        // myTransform.rotation = Quaternion.identity;
        // myTransform.localScale = Vector3.one;
        
        myTransform.ResetTransform();
    }
}
```

위와 같이 유니티 엔진에서 제공하는 `Transform`의 편리한 활용을 돕는 확장 메서드를 정의할 수도 있다. 

![2024-10-06210544-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/4ea19f4f-0931-4cb1-ba5e-ec00dd30f738)