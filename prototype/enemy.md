# 적 캐릭터 제작

심화과정3 - 적 캐릭터 제작 1

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/dnZpH8BlxO4/0.jpg)](http://www.youtube.com/watch?v=dnZpH8BlxO4)

### 멤버 필드 추가
예제 3-12: Enemy.cs

```csharp
--(전략)--
public enum EnemyState
{
    none,
    move,
    attack,
    damaged,
    dead
}

public class Enemy : MonoBehaviour {
    // 적 상태.
    public EnemyState currentState = EnemyState.none;
    // LineCast에 사용될 위치.
    public Transform frontPosition;
    protected RaycastHit2D isObstacle;
    // 이동 속도.
    public float moveSpeed = 1.0f;
    // 체력.
    protected float currentHP;
    protected float maxHP;
    // 공격 가능여부 저장.
    protected bool enableAttack = true;
    protected float attackPower = 10;
    protected float damagedPower;
    protected Animator animator;

    void Awake()
    {
        animator = GetComponent<Animator>();
    }
--(후략)--
```

### 이동처리

예제 3-13: Enemy.cs
```csharp
--(전략)--
    void FixedUpdate ()
    {
        rigidbody2D.velocity = new Vector2(moveSpeed, rigidbody2D.velocity.y);
    }
--(후략)--
```


예제 3-14: Enemy.cs

```csharp
    void FixedUpdate ()
    {
        switch(currentState)
        {
        case EnemyState.none:
            // 이동 중지.
            rigidbody2D.velocity = Vector2.zero;
            break;
        case EnemyState.move:
            // 장애물이 있는지 Linecast로 검출.
            isObstacle = Physics2D.Linecast(
                transform.position, frontPosition.position,
                1 << LayerMask.NameToLayer("Obstacle") );
            if( isObstacle )
            {
                // TODO: 장애물을 만나면 공격 애니메이션으로 전환.
            }
            else
            {
                // 장애물이 없다면 이동.
                rigidbody2D.velocity = new Vector2(-moveSpeed,
                rigidbody2D.velocity.y);
            }
            break;
        case EnemyState.attack:
            rigidbody2D.velocity = Vector2.zero;
            break;
        case EnemyState.damaged:
            rigidbody2D.velocity = Vector2.zero;
            break;
        case EnemyState.dead:
            rigidbody2D.velocity = Vector2.zero;
            break;
        }
    }
--(후략)--
```

### 공격 애니메이션 처리
예제 3-15: Enemy.cs
```csharp
--(전략)--
            if( isObstacle )
            {
                // 장애물을 만나면 공격 애니메이션으로 전환.
                if( enableAttack )
                {
                    currentState = EnemyState.attack;
                    // Animator에 등록한 attack Trigger를 작동.
                    animator.SetTrigger("attack");
                }
            }
--(후략)--
```
예제 3-16: Enemy.cs
```csharp
--(전략)--
    void AttackAnimationEnd()
    {
        if( currentState == EnemyState.attack)
        {
            currentState = EnemyState.move;
        }
    }
--(후략)--
```

예제 3-17: Enemy.cs
```csharp
--(전략)--
    public void Attack()
    {
        // TODO: 앞에 위치한 장애물 등에 공격을 가한다.
    }
--(후략)--
```

### 충돌 처리
예제 3-18: ShotObj.cs
```csharp
using UnityEngine;
using System.Collections;

public class ShotObj : MonoBehaviour {
    protected float attackPower = 1;

    public void InitShotObj(float setupAttackPower)
    {
    attackPower = setupAttackPower;
    }
}
```

예제 3-19: FarmerTouchControl.cs
```csharp
--(전략)--
    // 발사되는 게임 오브젝트의 ShotObj 스크립트 처리.
    ShotObj shotObjScript;
    --(중략)--
    void Fire(Vector3 inputPosition)
    {
    --(중략)—
        // 속도 적용.
        tempObj.rigidbody2D.velocity = tempVector2;
        // 공격력을 전달한다.
        shotObjScript = tempObj.GetComponent<ShotObj>();
        shotObjScript.InitShotObj(1);
    }
--(후략)--
```

예제 3-20: IDamageable.cs
```csharp
using System;

public interface IDamageable
{
    void Damage(float damageTaken);
}
```

예제 3-21: ShotObj.cs
```csharp
--(전략)--
    void OnTriggerEnter2D(Collider2D other)
    {
        // 적 캐릭터 인 경우, 공격하여 피해를 가한다.
        if(other.CompareTag("enemy") || other.CompareTag("boss") )
        {
            IDamageable damageTarget = (IDamageable)other.GetComponent(typeof(IDamageable));
            damageTarget.Damage(attackPower);
            // 공격 후 제거.
            Destroy(gameObject);
        }
    }
--(후략)--
```

예제 3-22: Enemy.cs
```csharp
--(전략)--
public class Enemy : MonoBehaviour, IDamageable {
--(중략)--
    public void Damage(float damageTaken)
    {
	    // dead나 none 상태일 때 진행되지 않도록 한다.
	    if (currentState == EnemyState.dead || currentState == EnemyState.none)
	    {
	        if( IsInvoking("ChangeStateToMove") )
	        {
	            CancelInvoke("ChangeStateToMove");
	        }
	        return;
    	}

	    // 충돌 후 일정 시간 동안 이동 정지.
	    currentState = EnemyState.damaged;
	    if( IsInvoking("ChangeStateToMove") )
	    {
	        CancelInvoke("ChangeStateToMove");
	    }
	    Invoke("ChangeStateToMove", 0.3f);
	    // currentHP를 소진한다.
	    currentHP -= damageTaken;
	    // 현재 체력이 0과 같거나 작다면
	    if(currentHP <= 0)
	    {
	        currentHP = 0;
	        enableAttack = false;
	        currentState = EnemyState.dead;
	        // dead 애니메이션 재생
	        animator.SetTrigger("isDead");
	        if( IsInvoking("ChangeStateToMove") )
	        {
	            CancelInvoke("ChangeStateToMove");
	        }
            // TODO: 점수 증가.
        }
        else
        {
            animator.SetTrigger("damaged");
        }

    }

    void ChangeStateToMove()
    {
        // 충돌에 의한 경직 상태에서 이동 상태로 변경.
        currentState = EnemyState.move;
    }

    public void Attack()
    {
        //농장에 피해를 가한다.
        RaycastHit2D findObstacle = Physics2D.Linecast(
            transform.position, frontPosition.position,
            1 << LayerMask.NameToLayer("Obstacle"));
        if (findObstacle)
        {
            IDamageable damageTarget =
                (IDamageable)findObstacle.transform.GetComponent(typeof(IDamageable));
            damageTarget.Damage(attackPower);
        }
    }
--(후략)--
```

예제 3-23: Enemy.cs
```csharp
--(전략)--
    void OnEnable()
    {
        #if UNITY_EDITOR
        currentHP = 2;
        #endif
    }
--(후략)--
```

---

### 적 캐릭터 제작까지 진행된 소스코드
FarmerTouchControl.cs
```csharp
using UnityEngine;
using System.Collections;

public class FarmerTouchControl : MonoBehaviour {

	// 마우스 클릭으로 입력된 좌표를 공간 좌표로 변환하는데 사용.
	public Camera mainCamera;
	// 발사할 게임 오브젝트.
	public GameObject fireObj;
	// 새총을 발사할 지점.
	public Transform firePoint;
	// 새총을 발사할 방향.
	Vector3 fireDirection;
	// 발사 속도.
	public float fireSpeed = 3;
	// 발사 가능 여부 판단.
	bool enableAttack = true;
	// 마지막 사용자 입력 위치 저장.
	Vector3 lastInputPosition;
	// Vector3 계산에 사용.
	Vector3 tempVector3;
	// Vector2 계산에 사용.
	Vector2 tempVector2 = new Vector2();
	// 새총 발사에 사용되는 오브젝트 처리.
	GameObject tempObj;
	Animator animator;
	// 발사되는 게임 오브젝트의 ShotObj 스크립트 처리.
    ShotObj shotObjScript;

	void Awake()
	{
		animator = GetComponent<Animator>();
	}

    void Update()
    {
        // 마우스 왼쪽 버튼 입력이 발생했을 때.
        if( Input.GetMouseButton(0) )
        {
            // 마우스 입력 위치를 저장.
            lastInputPosition = Input.mousePosition;
            // 공격 가능 여부를 판단.
            if( enableAttack )
            {
                // 공격 애니메이션으로 전환.
                animator.SetTrigger("fire");
            }
        }
    }
    void FireTrigger()
    {
        // 발사 애니메이션이 진행되어 새총 발사를 하게 될 때 발사를 처리한다.
        Fire(lastInputPosition);
    }
    void FireEnd()
    {
        // 발사 애니메이션이 종료될 때, 공격 가능하도록 변경.
        enableAttack = true;
    }

	void Fire(Vector3 inputPosition)
    {
        // 입력 위치(inputPosition)를 카메라가 바라보는 영역 안의 월드 좌표(절대 좌표)로 변환.
        tempVector3 = mainCamera.ScreenToWorldPoint(inputPosition);
        tempVector3.z = 0;
        // 벡터의 뺄셈 후 방향만 지닌 단위 벡터로 변경.
        fireDirection = tempVector3 - firePoint.position;
        fireDirection = fireDirection.normalized;
        // 발사.
        tempObj = Instantiate(fireObj, firePoint.position,
        Quaternion.LookRotation(fireDirection)) as GameObject;
        // 발사한 오브젝트 속도 계산.
        tempVector2.Set(fireDirection.x, fireDirection.y);
        tempVector2 = tempVector2 * fireSpeed;
        // 속도 적용.
        tempObj.rigidbody2D.velocity = tempVector2;
        // 공격력을 전달한다.
        shotObjScript = tempObj.GetComponent<ShotObj>();
        shotObjScript.InitShotObj(1);
    }

}
```

Enemy.cs
```csharp
using UnityEngine;
using System.Collections;

public enum EnemyState
{
    none,
    move,
    attack,
    damaged,
    dead
}

public class Enemy : MonoBehaviour, IDamageable {
    // 적 상태.
    public EnemyState currentState = EnemyState.none;
    // LineCast에 사용될 위치.
    public Transform frontPosition;
    protected RaycastHit2D isObstacle;
    // 이동 속도.
    public float moveSpeed = 1.0f;
    // 체력.
    protected float currentHP;
    protected float maxHP;
    // 공격 가능여부 저장.
    protected bool enableAttack = true;
    protected float attackPower = 10;
    protected float damagedPower;
    protected Animator animator;

    void Awake()
    {
    	animator = GetComponent<Animator>();
    }

    void OnEnable()
    {
        #if UNITY_EDITOR
        currentHP = 2;
        #endif
    }

    void FixedUpdate ()
    {
        switch(currentState)
        {
        case EnemyState.none:
            // 이동 중지.
            rigidbody2D.velocity = Vector2.zero;
            break;
        case EnemyState.move:
            // 장애물이 있는지 Linecast로 검출.
            isObstacle = Physics2D.Linecast(
                transform.position, frontPosition.position,
                1 << LayerMask.NameToLayer("Obstacle") );
            if( isObstacle )
            {
                // 장애물을 만나면 공격 애니메이션으로 전환.
                if( enableAttack )
                {
                    currentState = EnemyState.attack;
                    // Animator에 등록한 attack Trigger를 작동.
                    animator.SetTrigger("attack");
                }
            }
            else
            {
                // 장애물이 없다면 이동.
                rigidbody2D.velocity = new Vector2(-moveSpeed,
                rigidbody2D.velocity.y);
            }
            break;
        case EnemyState.attack:
            rigidbody2D.velocity = Vector2.zero;
            break;
        case EnemyState.damaged:
            rigidbody2D.velocity = Vector2.zero;
            break;
        case EnemyState.dead:
            rigidbody2D.velocity = Vector2.zero;
            break;
        }
    }

    void AttackAnimationEnd()
    {
        if( currentState == EnemyState.attack)
        {
            currentState = EnemyState.move;
        }
    }

    public void Damage(float damageTaken)
    {
        // dead나 none 상태일 때 진행되지 않도록 한다.
        if (currentState == EnemyState.dead || currentState == EnemyState.none)
        {
            if( IsInvoking("ChangeStateToMove") )
            {
                CancelInvoke("ChangeStateToMove");
            }
            return;
        }

        // 충돌 후 일정 시간 동안 이동 정지.
        currentState = EnemyState.damaged;
        if( IsInvoking("ChangeStateToMove") )
        {
            CancelInvoke("ChangeStateToMove");
        }
        Invoke("ChangeStateToMove", 0.3f);
        // currentHP를 소진한다.
        currentHP -= damageTaken;
        // 현재 체력이 0과 같거나 작다면
        if(currentHP <= 0)
        {
            currentHP = 0;
            enableAttack = false;
            currentState = EnemyState.dead;
            // dead 애니메이션 재생
            animator.SetTrigger("isDead");
            if( IsInvoking("ChangeStateToMove") )
            {
                CancelInvoke("ChangeStateToMove");
            }
            // TODO: 점수 증가.
        }
        else
        {
            animator.SetTrigger("damaged");
        }

    }

    void ChangeStateToMove()
    {
        // 충돌에 의한 경직 상태에서 이동 상태로 변경.
        currentState = EnemyState.move;
    }

    public void Attack()
    {
        //농장에 피해를 가한다.
        RaycastHit2D findObstacle = Physics2D.Linecast(
            transform.position, frontPosition.position,
            1 << LayerMask.NameToLayer("Obstacle"));
        if (findObstacle)
        {
            IDamageable damageTarget =
                (IDamageable)findObstacle.transform.GetComponent(typeof(IDamageable));
            damageTarget.Damage(attackPower);
        }
    }
}
```

IDamageable.cs
```csharp
using System;

public interface IDamageable
{
    void Damage(float damageTaken);
}
```

ShotObj.cs
```csharp
using UnityEngine;
using System.Collections;

public class ShotObj : MonoBehaviour {
    protected float attackPower = 1;

    public void InitShotObj(float setupAttackPower)
    {
    	attackPower = setupAttackPower;
    }

    void OnTriggerEnter2D(Collider2D other)
    {
        // 적 캐릭터 인 경우, 공격하여 피해를 가한다.
        if(other.CompareTag("enemy") || other.CompareTag("boss") )
        {
            IDamageable damageTarget = (IDamageable)other.GetComponent(typeof(IDamageable));
            damageTarget.Damage(attackPower);
            // 공격 후 제거.
            Destroy(gameObject);
        }
    }
}
```
