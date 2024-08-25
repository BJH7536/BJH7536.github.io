---
title: Object Pool Pattern 오브젝트 풀 패턴
date: 2024-08-20 +0900
categories: [게임 디자인 패턴 with Unity 강의 필기]
tags: [디자인 패턴, Unity, C#]
math: true
mermaid: true
---

> 위 글은 이재환님의 게임 디자인 패턴 with Unity 인프런 강의를 듣고 남긴 필기입니다.

## Object Pool Pattern

재사용 가능한 객체들을 모아놓은 객체 풀 클래스를 정의한다.

여기에 들어가는 객체는 현재 자신이 ‘사용 중’인지 여부를 알 수 있는 방법을 제공해야 한다.

1. 풀은 초기화될 때 사용할 객체들을 미리 생성하고 (보통 같은 종류의 객체를 연속된 컬렉션에 넣는다), 이 객체들을 사용 안함 상태로 초기화 한다.
2. 새로운 객체가 필요하면 풀에 요청한다.
3. 풀은 사용가능한 객체를 찾아 ‘사용 중’으로 초기화한 뒤 반환한다.
4. 객체를 더이상 사용하지 않는다면 ‘사용 안함’ 상태로 되돌린다.

이런 식으로 메모리나 다른 자원 할당을 신경쓰지 않고 마음껏 객체를 생성, 삭제할 수 있다.

### 이 패턴을 사용할 때는

- 객체의 생성과 삭제가 빈번히 일어날 때
- 객체들의 크기가 비슷할 때
- 객체를 힙에 생성하기가 느리거나, 메모리 단편화가 우려될 때
- DB 연결이나 네트워크 연결같이 접근 비용이 비싸면서 재사용 가능한 자원을 객체가 캡슐화할 때.

***Object Pool Pattern***은 ***Flyweight Pattern***과 비슷한 점이 있다. 둘 다 재사용 가능한 객체 집합을 관리한다. 차이점은 “재사용”의 목표에 있다.

- ***Flyweight Pattern*** 은 같은 인스턴스를 여러 객체에서 공유함으로써 재사용한다. 
같은 객체를 동시에 여러 문맥에서 사용함으로써 메모리 중복 사용을 피하는 것이 그 목표.
- ***Object Pool Pattern*** 에서도 객체를 재사용하지만, 여러 곳에서 동시에 재사용하지는 않는다. 
Object Pool 에서 “재사용”이란,  이전 사용자가 사용을 완료한 다음에 객체 메모리를 회수해 가는 것을 의미한다. 
이 때, 객체는 한번에 한 곳에서만 사용된다.

### 예제1

```csharp
public class Enemy : MonoBehaviour
{
    private void OnMouseOver()
    {
        if(Input.GetMouseButtonDown(0))
            gameObject.SetActive(false);
    }
}

```

```csharp
public class EnemyManager : MonoBehaviour
{
    public GameObject enemy;
    public int poolSize = 10;
    private GameObject[] _enemyPool;

    private void Start()
    {
        _enemyPool = new GameObject[poolSize];
        for (int i = 0; i < poolSize; i++)
        {
            _enemyPool[i] = Instantiate(enemy);
            _enemyPool[i].name = "Enemy_" + i;
            _enemyPool[i].SetActive(false);
        }

        StartCoroutine(SpawnManager());
    }

    IEnumerator SpawnManager()
    {
        while (true)
        {
            // 2초 간격으로 생성
            yield return new WaitForSeconds(2.0f);

            for (int i = 0; i < poolSize; i++)
            {
                if (_enemyPool[i].activeSelf) continue;

                float x = Random.Range(-5, 6);
                float z = Random.Range(-5, 6);

                _enemyPool[i].transform.position = new Vector3(x, 0.5f, z);
                _enemyPool[i].SetActive(true);

                break;
            }
        }
    }
}

```

![TestingProject-ObjectPoolPattern-WindowsMacLinux-Unity2022 3 17f1__DX11_2024-03-2414-55-46-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/aa19af04-a790-4f77-924d-2076ccc59b5b)

- 일정한 시간마다 `EnemyManager` 가 인스턴스화 된 `Enemy` 를 무작위 위치에 놓고 활성화상태로 변화시킨다.
- `Enemy` 오브젝트를 마우스로 클릭하면 비활성화 상태로 변하고, `EnemyManager`는 자신의 코루틴에 따라 일정시간마다 이들을 활성화시킨다.