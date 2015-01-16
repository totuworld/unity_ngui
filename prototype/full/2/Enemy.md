### 적 캐릭터 제작까지 진행된 소스코드

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
