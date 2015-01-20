### 새로운 적 캐릭터 추가까지 진행된 소스코드

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
            // 공격 후 게임 오브젝트 제거
            AttackAndDestroy(other);
        }
    }
}
```
