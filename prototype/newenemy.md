# 새로운 적 캐릭터 추가

### 새로운 적 캐릭터 설정

예제 3-44: Enemy.cs

```csharp
---(전략)---
	public virtual void Attack()
---(후략)---
```

예제 3-45: EnemyRanged.cs
```csharp
using UnityEngine;
using System.Collections;

// Enemy 스크립트를 상속 받는다.
public class EnemyRanged : Enemy {

	// 발사할 게임 오브젝트.
	public GameObject shotObj;
	// 발사 위치.
	public Transform firePosition;
	// 발사 속도.
	public float fireSpeed = 3;

    // shot gameobject pool
    GameObjectPool objPool;
    Vector3 spawnPos = new Vector3(0, 50, 0);

	// 발사할 게임 오브젝트 생성에 사용.
	GameObject tempObj;
	Vector2 tempVector2 = new Vector2();

	public override void Attack()
	{
		// TODO: 원거리에서 공격할 수 있도록 구현.
	}
}
```

예제 3-46: EnemyRanged.cs
```csharp
---(전략)---
	public override void Attack()
	{
		// 원거리에서 공격할 수 있도록 구현.
		if(objPool == null)
		{
			objPool = new GameObjectPool(
				GameData.Instance.gamePlayManager
					.gameObjectPoolPosition.transform.position.x,
				shotObj);
		}
		if( !objPool.NextGameObject(out tempObj) )
		{
			tempObj = Instantiate(
				shotObj,
				GameData.Instance.gamePlayManager
					.gameObjectPoolPosition.transform.position,
				Quaternion.identity
				) as GameObject;
			tempObj.name = shotObj.name + objPool.lastIndex;
			objPool.AddGameObject(tempObj);
		}
		// position move
		tempObj.transform.position = firePosition.position;
		// 속도 지정.
		tempVector2 = Vector2.right* -1 * fireSpeed;
		tempObj.rigidbody2D.velocity = tempVector2;
		// TODO: 게임 오브젝트에 공격력을 담아 장애물 등과 충돌했을 때
		// 데미지를 끼칠 수 있도록 한다.
	}
---(후략)---
```

예제 3-47: EnemyShotObj.cs
```csharp
using UnityEngine;
using System.Collections;
public class EnemyShotObj : ShotObj
{
	void OnTriggerEnter2D(Collider2D other)
	{
		// 장애물과 충돌 시, 공격하여 피해를 가한다.
		if( other.CompareTag("obstacle"))
		{
			IDamageable damegeTarget = (IDamageable)other.GetComponent(typeof(IDamageable));
			damegeTarget.Damage(attackPower);
			// 공격 후 제거.
			Destroy(gameObject);
		}
	}
}
```

예제 3-48: ShotObj.cs
```csharp
---(전략)---
	void OnTriggerEnter2D(Collider2D other)
	{
		// 적 캐릭터 인 경우, 공격하여 피해를 가한다.
		if(other.CompareTag("enemy") || other.CompareTag("boss") )
		{
			// 공격 후 게임 오브젝트 제거.
			AttackAndDestroy(other);
		}
	}

	protected void AttackAndDestroy(Collider2D other)
	{
		IDamageable damageTarget =
			(IDamageable)other.GetComponent(typeof(IDamageable));
		damageTarget.Damage(attackPower);
		// 공격 후 제거.
        transform.position = (Vector3.right * 30) + (Vector3.up * 10);
        rigidbody2D.velocity = Vector2.zero;
	}
---(후략)---
```

예제 3-49: EnemyShotObj.cs
```csharp
---(전략)---
	void OnTriggerEnter2D(Collider2D other)
	{
		// 장애물과 충돌 시, 공격하여 피해를 가한다.
		if( other.CompareTag("obstacle"))
		{
			// 공격 후 게임 오브젝트 제거
			AttackAndDestroy(other);
		}
	}
---(후략)---
```

예제 3-50: EnemyRanged.cs
```csharp
---(전략)---
	public override void Attack()
	{
---(중략)---
		// 게임 오브젝트에 공격력을 담아 장애물 등과 충돌했을 때 데미지를 끼칠 수 있도록 한다.
		EnemyShotObj tempEnemyShot = tempObj.GetComponent<EnemyShotObj>();
		tempEnemyShot.InitShotObj( attackPower );
	}
---(후략)---
```


# 애니메이션 수정

예제 3-51: EnemyRanged.cs
```csharp
---(전략)---
	public void CallAttack()
	{
		Attack();
	}
---(후략)---
```
