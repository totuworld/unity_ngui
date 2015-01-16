### 적 캐릭터 제작까지 진행된 소스코드

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
