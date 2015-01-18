### 게임 플레이 매니저까지 진행된 소스코드

```csharp
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
// XML 사용을 위해서 추가.
using System.Xml;
using System.Xml.Serialization;

public enum GameState {ready, idle, gameOver, wait}

public class GamePlayManager : MonoBehaviour, IDamageable {
    // 게임 상황 판단.
    public GameState nowGameState = GameState.ready;
    // 생성할 Enemy 게임 오브젝트 리스트
    public List<GameObject> spawnEnemyObjs = new List<GameObject>();
    // 적 생성할 위치 저장.
    List<Vector3> spawnPositions = new List<Vector3>{
        new Vector3(12, 2.7f, 0), new Vector3(12, 0.26f, 0),
        new Vector3(12, -2.2f, 0), new Vector3(12, -4.7f, 0)};
    // 농장 HP
    float farmCurrentHP = 300;
    float farmLimitHP = 300;
    // 게임 시작 후 경과 시간
    float timeElapsed = 0;
    // 획득한 점수 저장.
    int score = 0;

    // 게임 오브젝트 풀에 들어가는 게임 오브젝트의 최초로 생성되는 위치.
    public Transform gameObjectPoolPosition;
    // 게임 오브젝트 풀 딕셔너리.
    Dictionary<string, GameObjectPool> gameObjectPools =
        new Dictionary<string, GameObjectPool>();

    // 적 생성 데이터 저장.
    List<EnemyWaveData> enemyWaveDatas = new List<EnemyWaveData>();
    int currentEnemyWaveDataIndexNo = 0;

    // 생성할 위치값을 생성할 유닛 수로 치환.
    Dictionary<int, int> positionToAmount = new Dictionary<int, int> {
        { 1, 1}, { 2, 1}, { 4, 1}, { 8, 1},
        { 3, 2}, { 5, 2}, { 6, 2}, { 9, 2}, {10, 2}, {12, 2},
        { 7, 3}, {11, 3}, {13, 3}, {14, 3},
        {15, 4}
    };


    void Awake()
    {
        // 스크립트 연결.
        GameData.Instance.gamePlayManager = this;
    }

    void OnDestroy()
    {
        // 스크립트 연결 해제.
        GameData.Instance.gamePlayManager = null;
    }

    void OnEnable()
    {
        InitGameObjectPools();
        LoadEnemyWaveDataFromXML();
    }

    // 적 캐릭터 별로 20개씩 게임 오브젝트를 생성하여 게임 오브젝트 풀에 등록한다.
    void InitGameObjectPools()
    {
        for(int i=0;i<spawnEnemyObjs.Count;i++)
        {
            // 게임 오브젝트 풀 생성.
            GameObjectPool tempGameObjectPool =
                new GameObjectPool(gameObjectPoolPosition.transform.position.x, spawnEnemyObjs[i]);
            for(int j=0;j<20;j++)
            {
                // 게임 오브젝트 생성.
                GameObject tempEnemyObj =
                    Instantiate(
                                spawnEnemyObjs[i],
                                gameObjectPoolPosition.position,
                                Quaternion.identity
                                ) as GameObject;
                tempEnemyObj.name = spawnEnemyObjs[i].name + j;
                tempEnemyObj.transform.parent = gameObjectPoolPosition;
                // 게임 오브젝트를 게임 오브젝트 풀에 등록.
                tempGameObjectPool.AddGameObject(tempEnemyObj);
            }
            gameObjectPools.Add(spawnEnemyObjs[i].name, tempGameObjectPool);
        }
    }

    // XML을 읽어서 enemyWaveDatas에 저장한다.
    void LoadEnemyWaveDataFromXML()
    {
        // 이미 데이터를 로딩했다면 다시 로딩하지 못하도록 예외처리.
        if( enemyWaveDatas != null && enemyWaveDatas.Count > 0) return;

        // XML파일을 읽는다.
        TextAsset xmlText = Resources.Load("EnemyWaveData") as TextAsset;
        // XML 파일을 문서 객체 모델(DOM)로 전환한다.
        XmlDocument xDoc = new XmlDocument();
        xDoc.LoadXml(xmlText.text);
        // XML 파일 안에 EnemyWaveData란 XmlNode를 모두 읽어들인다.
        XmlNodeList nodeList = xDoc.DocumentElement.SelectNodes("EnemyWaveData");

        XmlSerializer serializer = new XmlSerializer(typeof(EnemyWaveData));
        // 역질렬화를 통해 EnemyWaveData 구조체로 변경하여 enemyWaveDatas 멤버 필드에 저장한다.
        for(int i=0;i<nodeList.Count;i++)
        {
            EnemyWaveData enemyWaveData =
                (EnemyWaveData)serializer.Deserialize(new XmlNodeReader(nodeList[i]));
            enemyWaveDatas.Add(enemyWaveData);
        }
    }

    void Update()
    {
        switch(nowGameState)
        {
        case GameState.ready:
            // 게임이 시작되면 3초간 사용자가에게 준비시간을 제공.
            timeElapsed += Time.deltaTime;
            if(timeElapsed >= 3.0f)
            {
                timeElapsed = 0;
                SetupGameStateToIdle();
            }
            break;
        case GameState.wait:
        case GameState.idle:
            // 두 상태 모두 게임이 진행중이므로 경과 시간을 증가시킨다.
            timeElapsed += Time.deltaTime;
            break;
        }
    }

    public void SetupGameStateToIdle()
    {
        // 게임 스테이트를 idle로 변경.
        nowGameState = GameState.idle;

        // 해제되지 못한 Invoke를 해제하고 새롭게 설정.
        if( IsInvoking("CheckSpawnEnemy") )
        {
            CancelInvoke("CheckSpawnEnemy");
        }
        InvokeRepeating("CheckSpawnEnemy", 0.5f, 2.0f);
    }

    void CheckSpawnEnemy()
    {
        // idle 상태가 아니라면 더이상 진행되지 못하도록 에러처리.
        if(nowGameState != GameState.idle) return;
        // 적 생성 데이터 전체가 소모되었다면 게임을 종료하도록 한다.
        if( currentEnemyWaveDataIndexNo >= enemyWaveDatas.Count)
        {
            nowGameState = GameState.gameOver;
            CancelInvoke("CheckSpawnEnemy");
            // TODO: 결과창 표시.
            return;
        }
        // 적을 생성한다.
        SpawnEnemy( enemyWaveDatas[currentEnemyWaveDataIndexNo] );
        // 생성된 적이 boss인 경우 적 생성을 멈춘다.
        if( enemyWaveDatas[currentEnemyWaveDataIndexNo].tagName == "boss")
        {
            nowGameState = GameState.wait;
            CancelInvoke("CheckSpawnEnemy");
        }
        currentEnemyWaveDataIndexNo++;
    }

    void SpawnEnemy(EnemyWaveData enemyData)
    {
        int positionPointer = 1;
        int shiftPosition = 0;
        // 생성할 위치 값으로 생성할 유닛 수 판단.
        enemyData.amount = positionToAmount[enemyData.spawnPosition];
        // 생성해야하는 숫자만큼 loop
        for(int i=0; i< enemyData.amount; i++)
        {
            // 생성할 위치 선택.
            while( (positionPointer & enemyData.spawnPosition) < 1 )
            {
                shiftPosition++;
                positionPointer = 1 << shiftPosition;
            }
            // 오브젝트 풀에 사용가능한 게임 오브젝트가 있는지 점검.

            GameObject currentSpawnGameObject;
            if( !gameObjectPools[enemyData.type]
            .NextGameObject(out currentSpawnGameObject) )
            {
                // 사용가능한 게임 오브젝트가 없다면 생성하여 추가한다.
                currentSpawnGameObject =
                    Instantiate(
                    gameObjectPools[enemyData.type].spawnObj,
                    gameObjectPoolPosition.transform.position,
                    Quaternion.identity) as GameObject;

                currentSpawnGameObject.transform.parent = gameObjectPoolPosition;
                currentSpawnGameObject.name =
                    enemyData.type + gameObjectPools[enemyData.type].lastIndex;
                gameObjectPools[enemyData.type].AddGameObject(currentSpawnGameObject);
            }
            currentSpawnGameObject.transform.position =
                spawnPositions[shiftPosition];
                        // 선택된 적 캐릭터를 초기화하여 작동시킨다.
            currentSpawnGameObject.tag = enemyData.tagName;
            Enemy currentEnemy = currentSpawnGameObject.GetComponent<Enemy>();
            currentEnemy.InitEnemy(enemyData.HP, enemyData.AD, enemyData.MS);
            shiftPosition++;

            if(enemyData.tagName == "boss")
            {
                // TODO: 적 보스 캐릭터가 등장했다는 표시를 띄운다.
            }
        }
    }


    public void Damage(float damageTaken)
    {
        if(nowGameState == GameState.gameOver) return;
        farmCurrentHP -= damageTaken;

        #if UNITY_EDITOR
        Debug.Log(farmCurrentHP);
        #endif

        if(farmCurrentHP <= 0)
        {
            nowGameState = GameState.gameOver;
            // TODO: 결과창 표시.
        }
    }

    public void AddScore(int addScore)
    {
        if(nowGameState == GameState.ready
            || nowGameState == GameState.gameOver) return;
        score += addScore;

        #if UNITY_EDITOR
        Debug.Log(score);
        #endif

        // TODO: 획득한 점수를 화면에 표시.
    }
}

public class GameObjectPool {

    int poolNowIndex = 0;
    int count = 0;
    float spawnPositionX = 0;
    public GameObject spawnObj;

    List<GameObject> pool = new List<GameObject>();

    // 생성자.
    public GameObjectPool(float positionX, GameObject initSpawnObj)
    {
        spawnPositionX = positionX;
        spawnObj = initSpawnObj;
    }

    // 게임 오브젝트를 풀에 추가한다.
    public void AddGameObject(GameObject addGameObject)
    {
        pool.Add(addGameObject);
        count++;
    }

    // 사용중이지 않은 게임 오브젝트를 선택한다.
    public bool NextGameObject(out GameObject returnObject)
    {
        int startIndexNo = poolNowIndex;
        if( lastIndex == 0 )
        {
            returnObject = default(GameObject);
            return false;
        }
        while( pool[poolNowIndex].transform.position.x < spawnPositionX )
        {
            poolNowIndex++;
            poolNowIndex = (poolNowIndex >= count) ? 0 : poolNowIndex;
            // 사용가능한 게임 오브젝트가 없을 때.
            if( startIndexNo == poolNowIndex )
            {
                returnObject = default(GameObject);
                return false;
            }
        }
        returnObject = pool[poolNowIndex];
        return true;
    }

    public int lastIndex
    {
        get
        {
            return pool.Count;
        }
    }

    // 해당 인덱스의 게임 오브젝트가 존재하는 경우 반환.
    public bool GetObject(int index, out GameObject obj)
    {
        if( lastIndex < index || pool[index] == null)
        {
            obj = default(GameObject);
            return false;
        }
        obj = pool[index];
        return true;
    }
}

[XmlRoot]
public struct EnemyWaveData
{
    [XmlAttribute("waveNo")]
    public int waveNo;
    [XmlElement]
    public string type;
    [XmlElement]
    public int amount;
    [XmlElement]
    public int spawnPosition;

    [XmlElement]
    public string tagName;

    [XmlElement]
    public float MS;
    [XmlElement]
    public float AD;
    [XmlElement]
    public float HP;
}
```
