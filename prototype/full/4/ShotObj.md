### 새로운 적 캐릭터 추가까지 진행된 소스코드

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
}
```
