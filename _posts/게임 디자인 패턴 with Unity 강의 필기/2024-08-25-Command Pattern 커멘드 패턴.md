---
title: Command Pattern 커멘드 패턴
date: 2024-08-25 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Command Pattern

### GoF 정의

- 요청 자체를 캡슐화하는 것
- 이를 통해 요청이 서로 다른 사용자를 매개변수로 만들고, 요청을 대기시키거나 로깅하며, 되돌릴 수 있는 연산을 지원한다.

명령 패턴은 메서드 호출을 실체화, 즉 객체로 감싼 것이다.

함수 호출을 객체로 만든 이유는

- 디커플링으로 코드가 유연해진다.

이는 게임에서 주로 입력 키 변경 혹은 실행취소 및 재실행의 구현에 사용된다.

### 예제 1. CommandPattern이 적용되지 않은 경우

플레이어의 입력에 따라 캐릭터가 총을 발사하여 공격 하거나, 쉴드를 활성화하는 간단한 기능을 만들어보자

```csharp
using System.Collections;
using UnityEngine;

public class csCannon : MonoBehaviour
{
    private float power = 900.0f;
    private Vector3 velocity;

    private void Start()
    {
        velocity = transform.forward * power;
        
        GetComponent<Rigidbody>().AddForce(velocity);
        StartCoroutine(DeleteCannon());
    }

    IEnumerator DeleteCannon()
    {
        yield return new WaitForSeconds(0.5f);
        Destroy(gameObject);
    }
}

```

```csharp
using System;
using System.Collections;
using UnityEngine;

public class csPlayerNormal : MonoBehaviour
{
    public GameObject shield;
    public GameObject cannon;
    public Transform firePos;

    private void Update()
    {
        if(Input.GetKeyDown(KeyCode.A))
            Attack();
        else if(Input.GetKeyDown(KeyCode.B))
            Defense();
    }

    void Attack()
    {
        Debug.Log("Attack");

        Instantiate(cannon, firePos.position, firePos.rotation);
    }

    void Defense()
    {
        Debug.Log("Defense");
        shield.SetActive(true);
        StartCoroutine(Defense(0.5f));
    }

    IEnumerator Defense(float tm)
    {
        yield return new WaitForSeconds(tm);
        shield.SetActive(false);
    }
}

```

![2024-08-24163542-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/1ef665c2-6477-418c-9ec7-99bd5059f406)

csPlayerNormal 스크립트로 사용자의 키 입력을 인식하여

A 키를 누르면 총에서 총알이 발사되도록하고, B 키를 누르면 Shield가 생기는 간단한 예제이다.

### 예제 2. 예제1에 CommandPattern을 적용한 경우 (동작을 텍스트 디버그로 대체)

```csharp
using UnityEngine;

// 인터페이스 : Execute() 메서드만 있는 추상 클래스
public abstract class CommandKey
{
    public virtual void Execute() {}
}

// Concrete Command 객체 : 직접적으로 동작하는 객체
public class CommandAttack : CommandKey
{
    public override void Execute()
    {
        Attack();
    }

    void Attack()
    {
        Debug.Log("Attack");
    }
}

public class CommandDefense : CommandKey
{
    public override void Execute()
    {
        Defense();
    }

    void Defense()
    {
        Debug.Log("Defense");
    }
}
```

```csharp
using TMPro;
using UnityEngine;

public class csPlayerCommand : MonoBehaviour
{
    private bool bCmd;
    private TMP_Text txt1;
    private TMP_Text txt2;

    private CommandKey btnA, btnB;

    private void Start()
    {
        bCmd = true;
        txt1 = GameObject.Find("Text1").GetComponent<TMP_Text>();
        txt2 = GameObject.Find("Text2").GetComponent<TMP_Text>();

        SetCommand();
    }

    // SetCommand() 메소드를 통해 버튼을 누르면 어떤 동작을 수행할지를 각 버튼에 등록
    public void SetCommand()
    {
        if (bCmd)
        {
            btnA = new CommandAttack();
            btnB = new CommandDefense();

            bCmd = false;
            txt1.text = "A - Attack";
            txt2.text = "B - Defense";
        }
        else
        {
            btnA = new CommandDefense();
            btnB = new CommandAttack();

            bCmd = true;
            txt1.text = "A - Defense";
            txt2.text = "B - Attack";
        }
    }

    private void Update()
    {
        if(Input.GetKeyDown(KeyCode.A))
            btnA.Execute();
        else if(Input.GetKeyDown(KeyCode.B))
            btnB.Execute();
    }
}

```

![2024-08-24173005-ezgif com-video-to-gif-converter (1) (1)](https://github.com/user-attachments/assets/25db43ba-23e7-4949-b37b-4075f1f489f3)

처음 키보드의 A키를 눌렀을 때 콘솔창에 Attack이 출력되었고, 

뒤이어 B 키를 눌렀을 때 콘솔창에 Defense가 출력되었다.

csPlayerCommand 의 SetCommand 함수에 연결되어있는 버튼을 클릭한 후에 동일하게 키보드의 A키와 B키를 순서대로 눌렀고,

이번에는 콘솔창에 Defense가 먼저 출력되었고, 이후에 Attack이 출력되는 것을 볼 수 있다.

 이는 추상 클래스 `CommandKey`를 상속받는 두 클래스 (`CommandAttack` , `CommandDefense`) 로 각 기능을  캡슐화하여 구현할 수 있는 기능이다.

### 예제 3. 예제 2의 동작을 실제 유니티 상의 동작으로 연결한 경우

```csharp
using System.Collections;
using UnityEngine;

public class csCannon : MonoBehaviour
{
    private float power = 900.0f;
    private Vector3 velocity;

    private void Start()
    {
        velocity = transform.forward * power;
        
        GetComponent<Rigidbody>().AddForce(velocity);
        StartCoroutine(DeleteCannon());
    }

    IEnumerator DeleteCannon()
    {
        yield return new WaitForSeconds(0.5f);
        Destroy(gameObject);
    }
}

```

```csharp
using System.Collections;
using UnityEngine;

// 인터페이스 : Execute() 메서드만 있는 추상 클래스
public abstract class CommandKey
{
    public GameObject shield;
    public GameObject cannon;
    public Transform firePos;

    public MonoBehaviour mono;
    
    public virtual void Execute() {}
}

// Concrete Command 객체 : 직접적으로 동작하는 객체
public class CommandAttack : CommandKey
{
    public CommandAttack(MonoBehaviour _mono, GameObject _shield, GameObject _cannon, Transform _firePos)
    {
        this.mono = _mono;
        this.shield = _shield;
        this.cannon = _cannon;
        this.firePos = _firePos;
    }
    
    public override void Execute()
    {
        Attack();
    }

    void Attack()
    {
        Debug.Log("Attack");
        GameObject.Instantiate(cannon, firePos.position, firePos.rotation);
    }
}

public class CommandDefense : CommandKey
{
    public CommandDefense(MonoBehaviour _mono, GameObject _shield, GameObject _cannon, Transform _firePos)
    {
        this.mono = _mono;
        this.shield = _shield;
        this.cannon = _cannon;
        this.firePos = _firePos;
    }
    
    public override void Execute()
    {
        Defense();
    }

    void Defense()
    {
        Debug.Log("Defense");
        shield.SetActive(false);
        mono.StartCoroutine(Defense(0.5f));
    }

    IEnumerator Defense(float tm)
    {
        yield return new WaitForSeconds(tm);
        shield.SetActive(false);
    }
    
}
```

```csharp
using TMPro;
using UnityEngine;

public class csPlayerCommand : MonoBehaviour
{
    public GameObject shield;
    public GameObject cannon;
    public Transform firePos;
    
    private bool bCmd;
    private TMP_Text txt1;
    private TMP_Text txt2;

    private CommandKey btnA, btnB;

    private void Start()
    {
        bCmd = true;
        txt1 = GameObject.Find("Text1").GetComponent<TMP_Text>();
        txt2 = GameObject.Find("Text2").GetComponent<TMP_Text>();

        SetCommand();
    }

    // SetCommand() 메소드를 통해 버튼을 누르면 어떤 동작을 수행할지를 각 버튼에 등록
    public void SetCommand()
    {
        if (bCmd)
        {
            btnA = new CommandAttack(this, shield, cannon, firePos);
            btnB = new CommandDefense(this, shield, cannon, firePos);

            bCmd = false;
            txt1.text = "A - Attack";
            txt2.text = "B - Defense";
        }
        else
        {
            btnA = new CommandDefense(this, shield, cannon, firePos);
            btnB = new CommandAttack(this, shield, cannon, firePos);

            bCmd = true;
            txt1.text = "A - Defense";
            txt2.text = "B - Attack";
        }
    }

    private void Update()
    {
        if(Input.GetKeyDown(KeyCode.A))
            btnA.Execute();
        else if(Input.GetKeyDown(KeyCode.B))
            btnB.Execute();
    }
}

```

이는 예제1의 동작을 유지한 채, 

구조에 Command Pattern을 적용한 버전이다.

동작은 예제 1과 예제 2의 기능을 모두 구현한다.

키보드 A키에 공격과 B키에 쉴드 기능이 연결되어있으며

csPlayerCommand 의 SetCommand 함수를 호출하면

키보드 B키에 공격, A키에 공격 기능이 연결되도록 변경된다.

### 예제 4. “Actor에게 지시하기”

```csharp
using UnityEngine;

// 인터페이스 : Execute() 메서드만 있는 추상 클래스
public abstract class CommandKey
{
    public virtual void Execute(GameObject obj) {}
}

// Concrete Command 객체 : 직접적으로 동작하는 객체
public class CommandAttack : CommandKey
{
    // 객체를 파라미터로 받아 어떤 객체라도 메서드를 호출하여 사용할 수 있도록 한다.
    public override void Execute(GameObject obj)
    {
        // 객체와 메서드는 decoupling 관계
        Attack(obj);
    }

    void Attack(GameObject obj)
    {
        Debug.Log(obj.name + "Attack");
        obj.transform.Translate(Vector3.forward);
    }
}

public class CommandDefense : CommandKey
{
    public override void Execute(GameObject obj)
    {
        Defense(obj);
    }

    void Defense(GameObject obj)
    {
        Debug.Log(obj.name + "Defense");
        obj.transform.Translate(Vector3.back);
    }
}
```

```csharp
using UnityEngine;

public class csPlayerCommand : MonoBehaviour
{
    private CommandKey btnA, btnB;

    private void Start()
    {
        SetCommand();
    }

    public void SetCommand()
    {
        btnA = new CommandAttack();
        btnB = new CommandDefense();
    }

    public void BtnCommandA()
    {
        btnA.Execute(gameObject);
    }
    
    public void BtnCommandB()
    {
        btnB.Execute(gameObject);
    }
}

```

![2024-08-24184103-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/64f25fa1-c142-46cd-816b-da5d436aa79c)

Command 객체에 Actor, 즉 Player GameObject 객체를 인자로 넘겨주어, Command 객체가 Actor 객체에 대해 특정 명령을 수행하도록 설계되었다.

Command Pattern의 ‘Actor에게 지시하기’란,

Command 객체가 단순히 명령을 실행하는 것이 아니라, 
특정 Actor에게 특정 행동을 수행하도록 명령을 전달하는 것을 의미한다.

### 예제 5. Undo 기능의 구현

```csharp
using UnityEngine;

// 인터페이스 : Execute() 메서드만 있는 추상 클래스
public abstract class CommandKey
{
    public virtual void Execute(Transform tr, Vector3 newPos) {}
    public virtual void Undo(Transform tr) {}
}

// Concrete Command 객체 : 직접적으로 동작하는 객체
public class CommandMoveRight : CommandKey
{
    private Vector3 prevPos;

    public override void Execute(Transform tr, Vector3 newPos)
    {
        prevPos = tr.position;
        tr.Translate(newPos);
    }

    public override void Undo(Transform tr)
    {
        tr.position = prevPos;
    }
}

public class CommandMoveForward : CommandKey
{
    private Vector3 prevPos;

    public override void Execute(Transform tr, Vector3 newPos)
    {
        prevPos = tr.position;
        tr.Translate(newPos);
    }

    public override void Undo(Transform tr)
    {
        tr.position = prevPos;
    }
}
```

```csharp
using System.Collections.Generic;
using UnityEngine;

public class csPlayerCommand : MonoBehaviour
{
    private Stack<CommandKey> stack = new Stack<CommandKey>();

    public void MoveForward()
    {
        CommandKey command = new CommandMoveForward();
        stack.Push(command);
        command.Execute(transform, new Vector3(0,0,1));
    }

    public void MoveRight()
    {
        CommandKey command = new CommandMoveRight();
        stack.Push(command);
        command.Execute(transform, new Vector3(1,0,0));
    }

    public void MoveUndo()
    {
        if (stack.Count > 0)
        {
            CommandKey command = stack.Pop();
            command.Undo(transform);
        }
    }
}

```

![2024-08-24191347-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/eb04ffdb-3029-4fb0-922c-573f93c0d228)

플레이어 캐릭터를 조작하는 Command 객체들 내부에 Undo 기능을 구현하고, 이를 Stack으로 관리하여 위와 같은 기능을 구현한다.