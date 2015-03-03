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

### 일시 정지 버튼과 배속 버튼 처리
280p