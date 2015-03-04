# 게임 플레이 UI 연결


### 농장 체력, 점수, 적 웨이브 표시 연결
예제 3-52: GamePlayManager.cs
```csharp
---(전략)---
    public UISlider farmHPSlier;
    public UILabel scoreLb;
    public UILabel waveLb;
---(후략)---
```

예제 3-53: GamePlayManager.cs
```csharp
---(전략)---
	public void AddScore(int addScore)
	{
---(중략)---
		// 획득한 점수를 화면에 표시.
		scoreLb.text = score.ToString();
	}
---(후략)---
```

예제 3-54: GamePlayManager.cs
```csharp
---(전략)---
	public void Damage(float damageTaken)
	{
---(증략)---
		// 농장 체력 표시.
		farmHPSlier.value = farmCurrentHP / farmLimitHP;
	}
---(후략)---
```

예제 3-55: GamePlayManager.cs
```csharp
---(전략)---
	void SpawnEnemy(EnemyWaveData enemyData)
	{
		int positionPointer = 1;
		int shiftPosition = 0;
		// 생성할 위치 값으로 생성할 유닛 수 판단.
		enemyData.amount = positionToAmount[enemyData.spawnPosition];
		// 웨이브 표시.
		waveLb.text = enemyData.waveNo.ToString();
---(후략)---
```

* @GM 게임 오브젝트의 GamePlayManager에 아래 사항 연결

| 속성 | 값 |
| ---|:---:|
| Farm HPSlier | HPBar 게임 오브젝트 |
| Score Lb | ScoreLabel 게임 오브젝트 |
| Wave Lb | WaveLabel 게임 오브젝트 |


### 일시 정지 버튼과 배속 버튼 처리

예제 3-56: GamePlayManager.cs
```csharp
---(전략)---
public partial class GamePlayManager : MonoBehaviour, IDamageable {
---(후략)---
```


예제 3-57: GamePlayManager.Button.cs
```csharp
using UnityEngine;
using System.Collections;
// 유저 인터페이스를 통해서 호출되는 메서드를 관리.
public partial class GamePlayManager {

}
```


예제 3-58: GamePlayManager.Button.cs

```csharp
---(전략)---
	// 일시 정지 화면.
	public GameObject pauseWindow;
---(후략)---
```

예제 3-59: GamePlayManager.Button.cs
```csharp
---(전략)---
	public void ClickPauseButton()
	{
		// 게임을 일시 정지 시킨다.
		Time.timeScale = 0;
		// 일시 정지 화면을 화면에 나타나도록 한다.
		pauseWindow.SetActive(true);
	}
---(후략)---
```

예제 3-60: GamePlayManager.Button.cs
```csharp
---(전략)---
	public UILabel speedButtonTextLb;
---(중략)---
	public void ClickSpeedButton()
	{
		// 게임이 정지된 상태라면 더이상 처리하지 못하도록 예외처리.
		if( Time.timeScale == 0.0f) return;
		// 현재 배속을 참조하여 배속 변경.
		if(Time.timeScale == 1.0f)
		{
			Time.timeScale = 2.0f;
			speedButtonTextLb.text = "2x";
		}
		else
		{
			Time.timeScale = 1.0f;
			speedButtonTextLb.text = "1x";
		}
	}
---(후략)---
```

* PauseButton 게임 오브젝트의 UIButton에 아래 사항 연결

| 속성 | 값 |
| ---|:---:|
| Notify | @GM 게임 오브젝트 |
| Method | GamePlayManager/ClickPauseButton |

* SpeedButton 게임 오브젝트의 UIButton에 아래 사항 연결

| 속성 | 값 |
| ---|:---:|
| Notify | @GM 게임 오브젝트 |
| Method | GamePlayManager/ClickSpeedButton |

예제 3-61: GamePlayManager.Button.cs
```csharp
---(전략)---
	public void ClickPauseReloadButton()
	{
		Time.timeScale = 1;
		// 씬을 다시 로딩하여 새로 게임이 시작되도록 한다.
		Application.LoadLevel("PlayScene");
	}

	public void ClickPausePlayButton()
	{
		// 게임 재개.
		if( speedButtonTextLb.text == "2x" )
		{
			Time.timeScale = 2.0f;
		}
		else
		{
			Time.timeScale = 1.0f;
		}
		// 일시 정지 화면을 화면에서 사라지도록 한다.
		pauseWindow.SetActive(false);
	}

	public void ClickPauseHomeButton()
	{
		// TODO: 다른 씬으로 전환한다.
	}
---(후략)---
```

예제 3-62: GamePlayManager.cs
```csharp
---(전략)---
public enum GameState {ready, idle, gameOver, wait, loading}
---(후략)---
```

예제 3-63: GamePlayManager.Button.cs
```csharp
---(전략)---
	public void ClickPauseReloadButton()
	{
		// 중복으로 로딩되지 못하도록 한다.
		if( nowGameState == GameState.loading ) return;
		nowGameState = GameState.loading;
		Time.timeScale = 1;
		// 씬을 다시 로딩하여 새로 게임이 시작되도록 한다.
		Application.LoadLevel("PlayScene");
	}
---(후략)---
```

* ReloadButton 게임 오브젝트의 UIButton에 아래 사항 연결

| 속성 | 값 |
| ---|:---:|
| Notify | @GM 게임 오브젝트 |
| Method | GamePlayManager/ClickPauseReloadButton |

* @GM 게임 오브젝트의 GamePlayManager에 아래 사항 연결

| 속성 | 값 |
| ---|:---:|
| Pause Window | PauseWindow 게임 오브젝트 |
| Speed Button Text Lb | SpeedLabel 게임 오브젝트 |

### 적 캐릭터 체력 표시 연동

예제 3-64: Enemy.cs
```csharp
---(전략)---
	UISlider hpBarSlider;
	GameObject hpBarObj;
	Camera uiCam;
	UIPanel hpBarPanel;
	Vector3 hpBarCalVec3;
---(후략)---
```

예제 3-65: Enemy.cs
```csharp
---(전략)---
	public void InitHPBar(UISlider targetHPBar,
						UIPanel targetPanel,
						Camera targetCam)
	{
		// 멤버 필드 할당.
		hpBarSlider = targetHPBar;
		hpBarObj = hpBarSlider.gameObject;
		hpBarPanel = targetPanel;
		uiCam = targetCam;
		// 오브젝트 풀에서 제외되도록 초기값 임시 수정.
		hpBarObj.transform.localPosition = Vector3.left * 1000;
		// hpbar를 켠다.
		TurnOnOffHPBar(true);
	}

	protected void RepositionHPBar()
	{
		// 적 위치가 카메라 상에서 어느 위치인지 계산.
		hpBarCalVec3 = uiCam.WorldToScreenPoint(transform.position);
		hpBarCalVec3.z = 0;
		// 위치 조정.
		hpBarObj.transform.localPosition = hpBarCalVec3;
	}

	public void TurnOnOffHPBar(bool isTurnOn = false)
	{
		// hpbar를 끄고 켠다.
		hpBarObj.SetActive(isTurnOn);
	}
---(후략)---
```

예제 3-66: Enemy.cs
```csharp
---(전략)---
	void FixedUpdate()
	{
		switch (currentState)
		{
---(중략)---
		case EnemyState.move:
---(중략)---
			if (uiCam != null)
			{
				RepositionHPBar();
			}
			break;
---(후략)---
```

예제 3-67: GameData.cs
```csharp
---(전략)---
	public float targetWidth = 0, targetHeight = 640f;
---(후략)---
```

예제 3-68: Enemy.cs
```csharp
---(전략)---
	protected void RepositionHPBar()
	{
		hpBarCalVec3 = uiCam.WorldToScreenPoint(transform.position);
		hpBarCalVec3.z = 0;

		if(GameData.Instance.targetWidth == 0)
		{
			GameData.Instance.targetWidth = hpBarPanel.width*
				(GameData.Instance.targetHeight/hpBarPanel.height);
			Debug.Log(GameData.Instance.targetWidth);
		}

		// UIPanel의 크기를 고려하여 상대적인 위치를 적용.
		hpBarCalVec3.x =
			(hpBarCalVec3.x / Screen.width) * GameData.Instance.targetWidth;
		hpBarCalVec3.y =
			(hpBarCalVec3.y / Screen.height) * GameData.Instance.targetHeight;
		hpBarObj.transform.localPosition = hpBarCalVec3;
	}
---(후략)---
```

예제 3-69: GamePlayManager.cs
```csharp
---(전략)---
	void CreateGameObject(GameObject targetObj,
						int amount,
						Transform parent,
						Vector3 localScale=default(Vector3))
	{
		// 게임 오브젝트 풀 생성.
		GameObjectPool tempGameObjectPool =
			new GameObjectPool(gameObjectPoolPosition.transform.position.x,
				targetObj);
		for(int j=0;j<amount;j++)
		{
			// 게임 오브젝트 생성.
			GameObject tempObj =
				Instantiate(
					targetObj,
					gameObjectPoolPosition.position,
					Quaternion.identity
					) as GameObject;
			tempObj.name = targetObj.name + j;
			tempObj.transform.parent = parent;
			if (localScale != Vector3.zero)
			{
				tempObj.transform.localScale = localScale;
			}
			// 게임 오브젝트를 게임 오브젝트 풀에 등록.
			tempGameObjectPool.AddGameObject(tempObj);
		}
		gameObjectPools.Add(targetObj.name, tempGameObjectPool);
	}
---(후략)---
```

예제 3-70: GamePlayManager.cs
```csharp
---(전략)---
	void InitGameObjectPools()
	{
		for(int i=0;i<spawnEnemyObjs.Count;i++)
		{
			CreateGameObject(spawnEnemyObjs[i], 20, gameObjectPoolPosition);
		}
	}
---(후략)---
```

예제 3-71: GamePlayManager.cs
```csharp
---(전략)---
	// 적 체력 표시 유저 인터페이스 생성에 사용한다.
	public GameObject enemyHPBar;
	public Transform enemyHPBarRoot;
---(후략)---
```

예제 3-72: GamePlayManager.cs
```csharp
---(전략)---
	void InitGameObjectPools()
	{
		for(int i=0;i<spawnEnemyObjs.Count;i++)
		{
			CreateGameObject( spawnEnemyObjs[i], 20, gameObjectPoolPosition );
		}

		// 적 체력 표시 유저 인터페이스 생성 및 등록.
		CreateGameObject(enemyHPBar, 20, enemyHPBarRoot, Vector3.one);
	}
---(후략)---
```

예제 3-73: GamePlayManager.cs
```csharp
---(전략)---
	// 적 체력 표시 유저 인터페이스 할당에 사용한다.
	public UIPanel enemyHPBarPanel;
	public Camera enemyHPBarCam;
---(후략)---
```

예제 3-74: GamePlayManager.cs
```csharp
--(전략)--
	void SpawnEnemy(EnemyWaveData enemyData)
	{
--(중략)--
		// 선택된 적 캐릭터를 초기화하여 작동시킨다.
		currentSpawnGameObject.tag = enemyData.tagName;
		Enemy currentEnemy = currentSpawnGameObject.GetComponent<Enemy>();
		currentEnemy.InitEnemy(enemyData.HP, enemyData.AD, enemyData.MS);
		shiftPosition++;

		// 게임오브젝트 풀에서 사용가능한 적 체력 표시 인터페이스가 있는지 체크.
		GameObject currentEnemyHPBar;
		if (!gameObjectPools [enemyHPBar.name]
			.NextGameObject(out currentEnemyHPBar))
		{
			// 사용가능한 게임 오브젝트가 없다면 생성하여 추가한다.
			currentEnemyHPBar =
				Instantiate(
					enemyHPBar,
					gameObjectPoolPosition.transform.position,
					Quaternion.identity) as GameObject;

			currentEnemyHPBar.transform.parent = enemyHPBarRoot;
			currentEnemyHPBar.transform.localScale = Vector3.one;
			currentEnemyHPBar.name =
				enemyHPBar.name + gameObjectPools [enemyHPBar.name].lastIndex;
			gameObjectPools [enemyHPBar.name].AddGameObject(currentEnemyHPBar);
		}
		// 적 체력 표시 인터페이스 할당.
		UISlider tempEnemyHPBarSlider =
			currentEnemyHPBar.GetComponent<UISlider>();
		currentEnemy.InitHPBar(
			tempEnemyHPBarSlider,
			enemyHPBarPanel,
			enemyHPBarCam);

		if(enemyData.tagName == "boss")
		{
			// TODO: 적 보스 캐릭터가 등장했다는 표시를 띄운다.
		}
--(후략)--
```

예제 3-75: Enemy.cs
```csharp
--(전략)--
	public void Damage(float damageTaken)
	{
--(중략)--
		// currentHP를 소진한다.
		currentHP -= damageTaken;
		// 체력 표시를 감소시킨다.
		hpBarSlider.value = (float)currentHP/(float)maxHP;
		
		// 현재 체력이 0과 같거나 작다면
		if (currentHP <= 0)
		{
			currentHP = 0;
			// 체력 표시를 모두 제거한다.
			hpBarSlider.value = 0;
            hpBarObj.transform.localPosition = Vector3.right * 1000;
            // hpbar를 끈다.
            TurnOnOffHPBar(false);
			enableAttack = false;
--(후략)--
```

* @GM 게임 오브젝트의 GamePlayManager에 아래 사항 연결

| 속성 | 값 |
| ---|:---:|
| Enemy HPBar | EnemyHPBar 게임 오브젝트 |
| Enemy HPBar Root | EnemyHPBarRoot 게임 오브젝트 |
| Enemy HPBar Panel | Panel 게임 오브젝트 |
| Enemy HPBar Cam | Main Camera 게임 오브젝트 |

* Assets/Prefabs/Pig 프리팹의 Current State가 move라면 none으로 변경합니다.

