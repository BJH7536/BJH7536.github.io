---
title: Strategy Pattern 스트레티지 패턴
date: 2024-08-14 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Strategy Pattern
- 여러 알고리즘을 하나의 추상적인 접근점(인터페이스)을 만들어 접근점에서 알고리즘이 서로 교환 가능하도록 하는 패턴
    - ⇒ 즉, 동일 목적 알고리즘의 선택적인 적용 문제를 말하는 것.

### 예제 1 소스코드

```csharp
using UnityEngine;

public class WeaponManager : MonoBehaviour
{
    private MyWeapon myWeapon;

    // 처음에는 총알을 들고 시작하도록
    private void Start()
    {
        myWeapon = new MyWeapon();
        myWeapon.SetWeapon(new Bullet());
    }

    public void ChangeBullet()
    {
        myWeapon.SetWeapon(new Bullet());
    }

    public void ChangeMissile()
    {
        myWeapon.SetWeapon(new Missile());
    }

    public void ChangeArrow()
    {
        myWeapon.SetWeapon(new Arrow());
    }

    public void Fire()
    {
        myWeapon.Shoot();
    }
}

```

```csharp
public class MyWeapon
{
    // 접근점
    public IWeapon weapon;

    // 무기 교환
    public void SetWeapon(IWeapon weapon)
    {
        this.weapon = weapon;
    }

    // 접근점을 활용해 공격
    public void Shoot()
    {
        weapon.Shoot();
    }

}

```

```csharp
public interface IWeapon
{
    void Shoot();
}
```

```csharp
public class Missile : IWeapon
{
    public void Shoot()
    {
        Debug.Log("Shoot Missile");
    }
}

```

```csharp
public class Bullet : IWeapon
{
    public void Shoot()
    {
        Debug.Log("Shoot Bullet");
    }
}
```

```csharp
public class Arrow : IWeapon
{
    public void Shoot()
    {
        Debug.Log("Shoot Arrow");
    }
}

```

- 예제에는 `Bullet`, `Arrow`, `Missile`이 등장했는데, 이는 모두 `Shoot()`이라는 행위가 공통적으로 가능한 개념들이다.
- 이러한 공통점을 인터페이스로 선언하고 `IWeapon`, 각각의 클래스에서 인터페이스를 상속받아 디테일한 부분은 각각의 클래스 내부에서 구현할 수 있다.
- 그리고 이를 기반으로, 클래스 외부에서도 해당 클래스가 `IWeapon` 을 상속받음을 알 수 있기 때문에, 디테일 한 부분은 고려하지 않고 `Shoot()`이라는 함수가 분명히 존재함을 알고 이를 호출할 수 있는 것이다.

### 예제 2 소스코드

```csharp
public class WeaponManager : MonoBehaviour
{
    public GameObject _arrow;
    public GameObject _bullet;
    public GameObject _missile;
    public GameObject myWeapon;
    
    private IWeapon weapon;

    // 무기 교환
    private void SetWeaponType(WeaponType weaponType)
    {
        IWeapon iweapon = gameObject.GetComponent<IWeapon>();
        
        if(iweapon != null)
            Destroy(iweapon as Component);

        switch (weaponType)
        {
            case WeaponType.Bullet:
            {
                weapon = gameObject.AddComponent<Bullet>();
                myWeapon = _bullet;
                break;
            }
            case WeaponType.Missile:
            {
                weapon = gameObject.AddComponent<Missile>();
                myWeapon = _missile;
                break;
            }
            case WeaponType.Arrow:
            {
                weapon = gameObject.AddComponent<Arrow>();
                myWeapon = _arrow;
                break;
            }
            default:
            {
                weapon = gameObject.AddComponent<Bullet>();
                myWeapon = _bullet;
                break;
            }
        }
    }
    
    void Start() 
    {
        SetWeaponType(WeaponType.Bullet);    
    }

    public void ChangeBullet()
    {
        SetWeaponType(WeaponType.Bullet);
    }
    
    public void ChangeMissile()
    {
        SetWeaponType(WeaponType.Missile);
    }
    public void ChangeArrow()
    {
        SetWeaponType(WeaponType.Arrow);
    }

    public void Fire()
    {
        weapon.Shoot(myWeapon);
    }
}

```

```csharp
public class Missile : MonoBehaviour, IWeapon
{
    public void Shoot(GameObject obj)
    {
        Vector3 initialPosition = new Vector3(transform.position.x, transform.position.y, 0);
        GameObject bullet = Instantiate(obj) as GameObject;
        bullet.transform.position = initialPosition;
    }
}

```

```csharp
public class Bullet : MonoBehaviour, IWeapon
{
    public void Shoot(GameObject obj)
    {
        Vector3 initialPosition = new Vector3(transform.position.x, transform.position.y, 0);
        GameObject bullet = Instantiate(obj) as GameObject;
        bullet.transform.position = initialPosition;
    }
}

```

```csharp
public class Arrow : MonoBehaviour, IWeapon
{
    public void Shoot(GameObject obj)
    {
        Vector3 initialPosition = new Vector3(transform.position.x, transform.position.y, 0);
        GameObject bullet = Instantiate(obj) as GameObject;
        bullet.transform.position = initialPosition;
    }
}

```

```csharp
public class BulletMove : MonoBehaviour
{
    private float speed = 10.0f;

    private void Update()
    {
        transform.Translate(Vector3.up * speed * Time.deltaTime);

        if (transform.position.y > 10.0f)
        {
            Destroy(gameObject);
        }
    }
}

```

```csharp
public enum WeaponType
{
    Bullet,
    Missile,
    Arrow,
}
```

```csharp
public interface IWeapon
{
    void Shoot(GameObject obj);
}
```

![TestingProject-StrategyPattern-WindowsMacLinux-Unity2022 3 17f1_DX11_2024-03-0315-06-30-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/67aaf73a-a83f-485b-8e7c-b328c3173fc8)

- `Missile`, `Arrow`, `Bullet`은 각각 다른 형태의 발사체를 의미하는 클래스. 모두 `IWeapon` 인터페이스를 상속받아 `Shoot(GameObject obj)` 함수를 구현한다.
- `BulletMove` 클래스는 발사체에게 모두 부착되어 움직임을 담당하는 클래스. 위 각기 다른 클래스마다 프리팹이 생성되어 각각 부착되지만, `BulletMove` 클래스는 모두에게 부착된다.
- **`WeaponManager`** 클래스는 이 발사체들을 발사시키는 클래스.
    - `Missile`, `Arrow`, `Bullet` 이 모두 `IWeapon` 인터페이스를 구현하기 때문에 각각 클래스의 인스턴스가 모두 `Shoot(GameObject obj)`함수를 구현함을 보장받고 호출할 수 있다.
    - 각각의 `Shoot(GameObject obj)` 함수들은 모두 자신의 프리팹을 인스턴스화하는 기능을 갖는다.

내 개인적인 생각으로는, 

- 이러한 형태는 코드의 중복 작성을 막고, 유사한 형태의 여러 구현체들을 만들 때, 각각이 다른 점에 집중할 수 있도록 한다.
- 예를들어 생존게임을 만든다고 상상해보면 도끼, 망치, 활과 같은 아이템은 모두 플레이어가 손에 쥐고 각각의 행동이 가능하게 만들 수 있을 것이다.
- 이 때, 이러한 아이템들은 모두 `플레이어가 손에 쥘 수 있음.` 그리고 `클릭하면 각각의 행동을 한다`. 라는 공통점을 갖기 때문에, 이러한 부분을 인터페이스화시켜 관리할 수 있을 것이다.