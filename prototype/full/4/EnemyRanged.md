### 새로운 적 캐릭터 추가까지 진행된 소스코드

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
        // 게임 오브젝트에 공격력을 담아 장애물 등과 충돌했을 때 데미지를 끼칠 수 있도록 한다.
        EnemyShotObj tempEnemyShot = tempObj.GetComponent<EnemyShotObj>();
        tempEnemyShot.InitShotObj( attackPower );
    }

    public void CallAttack()
    {
        Attack();
    }
}
```
