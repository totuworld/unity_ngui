# 게임 플레이 추가 작업

### 코인 표시
### 적 보스 캐릭터 등장 표시
### 결과창 제작

* 위 내용은 이미 작성되어있기 때문에 추가로 필요한 작업이 없습니다.
> 직접 진행하고 싶으신 분들은 책 299~316 페이지를 확인하세요.


### 코인 제작

* coins 스프라이트 편집
* 씬에 배치(name : coin, Sorting Layer : ForeGround)
* 애니메이션 제작(coin)

표 3-87: 프레임별 스프라이트

|프레임|스프라이트|
|:---:|:---:|
|0|coins_0|
|1|coins_1|
|2|coins_3|
|3|coins_2|
|4|coins_4|
|5|coins_5|
|6|coins_0|
|7|coins_1|
|8|coins_3|
|9|coins_2|
|10|coins_4|
|11|coins_5|
|12|coins_0|

표 3-88: 프레임 별 Transform 컴포넌트 Position 입력 값

|프레임|Transform 컴포넌트 Position|
|:---:|:---:|
|0|0, 0, 0|
|2|0, 0.5, 0|
|4|0, 0, 0|
|9|0, 0, 0|
|12|0, 2, 0|

* Position.y 커브 편집
* coin Controller 편집


### 코인 스크립트 제작 및 적용

예제 3-76: Coin.cs

```csharp
using UnityEngine;
using System.Collections;
public class Coin : MonoBehaviour {
	Animator animator;
	Vector3 removePosition = Vector3.right*30;

	void Awake()
	{
		animator = GetComponent<Animator>();
	}
	
	// 애니메이션을 작동하도록 한다.
	public void StartCoinAnimation(Vector3 setRemovePosition = default(Vector3))
	{
		if( setRemovePosition != Vector3.zero)
		{
			removePosition = setRemovePosition;
		}
		animator.SetTrigger("startAnimation");
	}

	// 애니메이션이 종료되면 호출된다.
	void EndCoinAnimation()
	{
		transform.parent.position = removePosition;
	}
}
```

* coin Prefab 제작

예제 3-77: GamePlayManager.cs

```csharp
---(전략)---
	// 코인 프리팹을 등록하는데 사용.
	public GameObject coinObj;
	public UILabel coinLb;
---(후략)---
```

예제 3-78: GamePlayManager.cs
```csharp
---(전략)---
	void InitGameObjectPools()
	{
---(중략)---
		// 적 캐릭터의 dead 애니메이션이 종료된 후 일정 확율로 나타나게될 코인.
		CreateGameObject(coinObj, 20, gameObjectPoolPosition);
	}
---(후략)---
```

예제 3-79: GamePlayManager.cs

```csharp
	// 코인을 생성할 포지션을 spawnPosition 매개변수로 전달하여 사용.
	public void SpawnCoin(Vector3 spawnPosition)
	{
		GameObject currentCoin;
		if (!gameObjectPools [coinObj.name]
			.NextGameObject(out currentCoin))
		{
			// 사용가능한 게임 오브젝트가 없다면 생성하여 추가한다.
			currentCoin =
			Instantiate(
				coinObj,
				gameObjectPoolPosition.transform.position,
				Quaternion.identity) as GameObject;
			currentCoin.transform.parent = enemyHPBarRoot;
			currentCoin.name =
				coinObj.name + gameObjectPools [coinObj.name].lastIndex;
			gameObjectPools [enemyHPBar.name].AddGameObject(currentCoin);
		}

		// 코인 애니메이션을 시작하도록 한다.
		Coin coinScript = currentCoin.transform.GetChild(0).GetComponent<Coin>();
		coinScript.StartCoinAnimation(gameObjectPoolPosition.position);
		// 코인 생성될 위치를 정한다.
		currentCoin.transform.position = spawnPosition;
	}
---(후략)---
```

예제 3-80: Enemy.cs
```csharp
---(전략)---
	public void DeadEnd()
	{
		// 임의의 확율로 코인을 생성한다.
		int makePossibleCoin = UnityEngine.Random.Range(0, 10);
		if(makePossibleCoin >= 7)
		{
			GameData.Instance.gamePlayManager.SpawnCoin(transform.position);
		}

		// 적 캐릭터를 초기 위치로 이동시킨다.
		currentState = EnemyState.none;
		transform.position =
			GameData.Instance.gamePlayManager.gameObjectPoolPosition.position;
		// hpbar를 반환한다.
		hpBarObj.transform.position =
			GameData.Instance.gamePlayManager.enemyHPBarRoot.position;
		hpBarObj = null;
	}
---(후략)---
```

* Pig와 Rat의 dead 애니메이션 11 프레임에 Add Event로 DeadEnd 메서드를 호출한다.

* @GM 게임 오브젝트의 GamePlayManager에 아래 사항 연결

| 속성 | 값 |
| ---|:---:|
| Coin Obj | coin 프리팹 |
| Coin Lb | CoinLabel 게임 오브젝트 |


### 결과창 연결

예제 3-81: GamePlayManager.cs

```csharp
---(전략)---
	// 획득한 점수 저장.
	int score = 0;
	// 처치한 적 숫자 기록.
	int deadEnemys = 0;
	// 획득한 코인 숫자 저장.
	int getCoins = 0;
---(후략)---
```

예제 3-82: GamePlayManager.cs
```csharp
	// 처치한 적과 획득한 코인 저장.
	public void AddDeadEnemyAndCoin(int getCoin = 0)
	{
		// 처치한 적 숫자 증가.
		deadEnemys++;
		// 획득한 코인 숫자 증가.
		getCoins += getCoin;
		if(getCoin > 0)
		{
			// 유저 인터페이스에 반영.
			coinLb.text = getCoins.ToString();
		}
	}
---(후략)---
```

예제 3-83: Enemy.cs
```csharp
---(전략)---
	public void DeadEnd()
	{
		// 임의의 확율로 코인을 생성한다.
		int makePossibleCoin = UnityEngine.Random.Range(0, 10);
		if(makePossibleCoin >= 7)
		{
			GameData.Instance.gamePlayManager.SpawnCoin(transform.position);
			// 처치한 적 숫자 및 획득한 코인 숫자 전달.
			GameData.Instance.gamePlayManager.AddDeadEnemyAndCoin(10);
		}
		else
		{
			// 처치한 적 숫자 전달.
			GameData.Instance.gamePlayManager.AddDeadEnemyAndCoin();
		}
---(후략)---
```

예제 3-84: GamePlayManager.cs
```csharp
---(전략)---
	// 결과창 게임 오브젝트.
	public GameObject resultWindow;
	// 결과창에 사용되는 UILabel
	public UILabel resultHighScoreLb, resultNowScoreLb,
		resultWaveLb, resultDeadEnemysLb, resultGetCoinsLb;
---(중략)---
	// 결과창을 나타나게 한다.
	public void OpenResult()
	{
		// 재 생성 방지.
		if(resultWindow.activeInHierarchy) return;

		// 적 생성 구문 해제.
		if (IsInvoking("CheckSpawnEnemy"))
		{
			CancelInvoke("CheckSpawnEnemy");
		}
		// TODO: 최고 점수를 나타낼 수 있도록 해야한다.
		resultHighScoreLb.text = "0";

		resultNowScoreLb.text = score.ToString();
		resultWaveLb.text = waveLb.text;
		resultDeadEnemysLb.text = deadEnemys.ToString();
		resultGetCoinsLb.text = getCoins.ToString();
		// 결과창을 나타나게 한다.
		resultWindow.SetActive(true);
	}
---(후략)---
```

예제 3-85: GamePlayManager.cs
```csharp
---(전략)---
	public void Damage(float damageTaken)
	{
---(중략)---
		if (farmCurrentHP <= 0)
		{
			nowGameState = GameState.gameOver;
			// 결과창 표시.
			OpenResult();
		}
---(중략)---
	void CheckSpawnEnemy()
	{
---(중략)---
	// 적 생성 데이터 전체가 소모되었다면 게임을 종료하도록 한다.
	if (currentEnemyWaveDataIndexNo >= enemyWaveDatas.Count)
	{
		nowGameState = GameState.gameOver;
		CancelInvoke("CheckSpawnEnemy");
		// 결과창 표시.
		OpenResult();
		return;
	}
---(후략)---
```

예제 3-86: Enemy.cs
```csharp
---(전략)---
	// idle 애니메이션만 재생하는 상태로 변경.
	public void FreezeEnemy()
	{
		currentState = EnemyState.none;
		if( IsInvoking("ChangeStateToMove") )
		{
			CancelInvoke("ChangeStateToMove");
		}
	}
---(후략)---
```

예제 3-87: GamePlayManager.cs
```csharp
---(전략)---
	// 결과창이 나타날 때 적 캐릭터를 정지하는 목적으로 사용된다.
	public event System.Action OnFreeze;
---(중략)---
	// 모든 적 캐릭터의 FreezeEnemy 메서드를 OnFreeze에 등록.
	void SetupAllEnemyFreeze()
	{
		int j=0;
		GameObject tempObj;
		Enemy tempEnemyScript;
		for(int i=0; i<spawnEnemyObjs.Count; ++i)
		{
			j=0;
			while(j< gameObjectPools[ spawnEnemyObjs[i].name].lastIndex)
			{
				if(gameObjectPools[ spawnEnemyObjs[i].name].GetObject(j, out tempObj))
				{
					tempEnemyScript = tempObj.GetComponent<Enemy>();
					// enemy 스크립트의 FreezeEnemy 메서드를 등록.
					OnFreeze += tempEnemyScript.FreezeEnemy;
				}
				++j;
			}
		}
	}
---(후략)---
```

예제 3-88: Enemy.cs
```csharp
---(전략)---
	public void OpenResult()
	{
---(중략)---
		// 결과창을 나타나게 한다.
		resultWindow.SetActive(true);

		// 이벤트에 등록된 메서드가 있는지 체크.
		if(OnFreeze !=null)
		{
			OnFreeze();
		}
---(후략)---
```


### 발사 게임 오브젝트 수정


예제 3-89: ShotObj.cs

```csharp
---(전략)---
	protected Vector3 initPos;
	protected bool isWork = false;
---(후략)---
```

예제 3-90: ShotObj.cs

```csharp
---(전략)---
	// 사용중이지 않을 때 돌아갈 위치 저장.
	public void InitReturnPosition(Vector3 setupInitPos)
	{
		initPos = setupInitPos;
	}

	// 충돌 처리를 허가할 때 사용.
	public void TurnOnTrigger()
	{
		isWork = true;
	}
	
	// 발사 게임 오브젝트를 초기화한다.
	public void ResetShotObj()
	{
		transform.position = initPos;
		isWork = false;
		rigidbody2D.velocity = Vector3.zero;
	}
---(후략)---
```

예제 3-91: ShotObj.cs

```csharp
---(전략)---
	void OnTriggerEnter2D(Collider2D other)
	{
		// 사용중이지 않을 때 충돌 처리를 막는다.
		if(!isWork) return;

		// 적 캐릭터 인 경우, 공격하여 피해를 가한다.
		if (other.CompareTag("enemy") || other.CompareTag("boss"))
		{
			// 공격 후 게임 오브젝트 제거.
			AttackAndRemove(other);
		}
		// 게임 플레이 화면 외부로 진입했을 때 초기 위치로 돌아가도록 한다.
		else if (other.CompareTag("invisibleArea"))
		{
			ResetShotObj();
		}
	}
	protected void AttackAndRemove(Collider2D other)
	{
		IDamageable damageTarget =
			(IDamageable)other.GetComponent(typeof(IDamageable));
		damageTarget.Damage(attackPower);
		// 공격 후 초기화.
		ResetShotObj();
	}
---(후략)---
```

예제 3-92: EnemyShotObj.cs
```csharp
---(전략)---
	void OnTriggerEnter2D(Collider2D other)
	{
		// 장애물과 충돌 시, 공격하여 피해를 가한다.
		if( other.CompareTag("obstacle"))
		{
			// 공격 후 게임 오브젝트 제거.
			AttackAndRemove(other);
		}
		// 게임 플레이 화면 외부로 진입했을 때 초기 위치로 돌아가도록 한다.
		else if (other.CompareTag("invisibleArea"))
		{
			ResetShotObj();
		}
	}
---(후략)---
```

예제 3-93: FamerTouchControl.cs
```csharp
---(전략)---
	// shot gameobject pool
	GameObjectPool objPool;
	Vector3 spawnPos = new Vector3(0, 50, 0);
---(후략)---
```

예제 3-94: FarmerTouchControl.cs
```csharp
---(전략)---
	void OnEnable()
	{
		InitGameObjectPool();
	}
---(중략)---
	// 게임 오브젝트 풀 초기화.
	void InitGameObjectPool()
	{
		spawnPos += GameData.Instance.gamePlayManager
			.gameObjectPoolPosition.transform.position;
		// 원거리에서 공격할 수 있도록 구현.
		objPool = new GameObjectPool(
						spawnPos.x,
						tempObj);
		// 발사 오브젝트 생성.
		for(int i=0;i<20;++i)
		{
			shotObjScript = null;
			GameObject makeObj =
				Instantiate(
					fireObj,
					spawnPos,
					Quaternion.Euler(Vector3.up * 90)
					) as GameObject;
			makeObj.name = makeObj.name + i;
			objPool.AddGameObject(makeObj);
			shotObjScript = makeObj.GetComponent<ShotObj>();
			shotObjScript.InitReturnPosition(spawnPos);
		}
	}
---(후략)---
```

예제 3-95: FarmerTouchControl.cs
```csharp
---(전략)---
	void Fire(Vector3 inputPosition)
	{
		// 입력 위치(inputPosition)를 카메라가 바라보는 영역 안의 월드 좌표(절대 좌표)로 변환.
		tempVector3 = mainCamera.ScreenToWorldPoint(inputPosition);
		tempVector3.z = 0;
		// 벡터의 뺄셈 후 방향만 지닌 단위 벡터로 변경.
		fireDirection = tempVector3 - firePoint.position;
		fireDirection = fireDirection.normalized;
		
		// 발사할 오브젝트.
		shotObjScript = null;
		if( !objPool.NextGameObject(out tempObj) )
		{
			tempObj = Instantiate(
				fireObj,
				spawnPos,
				Quaternion.Euler(Vector3.up * 90)
				) as GameObject;
			tempObj.name = tempObj.name + objPool.lastIndex;
			objPool.AddGameObject(tempObj);
			shotObjScript = tempObj.GetComponent<ShotObj>();
			shotObjScript.InitReturnPosition(spawnPos);
		}
		
		if(shotObjScript == null)
		{
			shotObjScript = tempObj.GetComponent<ShotObj>();
		}

		tempObj.transform.position = firePoint.position;
		// 발사한 오브젝트 속도 계산.
		tempVector2.Set(fireDirection.x, fireDirection.y);
		tempVector2 = tempVector2 * fireSpeed;
		// 속도 적용.
		tempObj.rigidbody2D.velocity = tempVector2;
		// 공격력을 전달한다.
		shotObjScript.InitShotObj(1);
		shotObjScript.TurnOnTrigger();
	}
---(후략)---
```

예제 3-96: EnemyRanged.cs
```csharp
---(전략)---
	// shot gameobject pool
	GameObjectPool objPool;
	Vector3 spawnPos = new Vector3(0, 50, 0);
---(후략)---
```

예제 3-97: EnemyRanged.cs
```csharp
---(전략)---
public override void Attack()
	{
		if( spawnPos.x == 0)
		{
			spawnPos += GameData.Instance.gamePlayManager
				.gameObjectPoolPosition.transform.position;
		}
		// 원거리에서 공격할 수 있도록 구현.
		if(objPool == null)
		{
			objPool = new GameObjectPool(
						spawnPos.x,
						shotObj);
		}
		EnemyShotObj tempEnemyShot = null;
		if( !objPool.NextGameObject(out tempObj) )
		{
			tempObj = Instantiate(
				shotObj,
				spawnPos,
				Quaternion.identity
				) as GameObject;
			tempObj.name = shotObj.name + objPool.lastIndex;
			objPool.AddGameObject(tempObj);
			tempEnemyShot = tempObj.GetComponent<EnemyShotObj>();
			tempEnemyShot.InitReturnPosition(spawnPos);
		}
		// position move
		tempObj.transform.position = firePosition.position;
		// 속도 지정.
		tempVector2 = Vector2.right* -1 * fireSpeed;
		tempObj.rigidbody2D.velocity = tempVector2;

		// 게임 오브젝트에 공격력을 담아 장애물 등과 충돌했을 때 데미지를 끼칠 수 있도록 한다.
		if(tempEnemyShot == null)
		{
			tempEnemyShot = tempObj.GetComponent<EnemyShotObj>();
		}
		tempEnemyShot.InitShotObj( attackPower );
		tempEnemyShot.TurnOnTrigger();
	}
---(후략)---
```


### 게임 준비와 홈 이동 버튼 연동

예제 3-98: GamePlayManager.Button.cs
```csharp
---(전략)---
	// 로비씬으로 전환할 때 사용한다.
	void LoadReadyScene(bool isPrepareGame)
	{
		// TODO: isPrepareGame를 활용하여 로비씬의 준비상태를 분기할 수 있도록 한다.
		// LobbyScene으로 전환한다.
		Application.LoadLevelAsync("LobbyScene");
	}

	public void ClickResultHomeButton()
	{
		LoadReadyScene(false);
	}

	public void ClickReGameButton ()
	{
		LoadReadyScene(true);
	}
---(후략)---
```

예제 3-99: GameData.cs
```csharp
---(전략)---
	public bool isPrepareGame = false;
---(후략)---
```

예제 3-100: GamePlayManager.Button.cs
```csharp
---(전략)---
	// 로비씬으로 전환할 때 사용한다.
	void LoadReadyScene(bool isPrepareGame)
	{
		// isPrepareGame를 활용하여 로비씬의 준비상태를 분기할 수 있도록 한다.
		GameData.Instance.isPrepareGame = isPrepareGame;
---(후략)---
```

표 3-89: UIButton 컴포넌트 설정

|게임 오브젝트|Target|Notify|Method|
|---|:---:|:---:|:---:|
|ReGameButton|ReGameButton|@GM|GamePlayManager.ClickResGameButton|
|ResultHomeButton|ResultHomeButton|@GM|GamePlayManager.ClickResultHomeButton|