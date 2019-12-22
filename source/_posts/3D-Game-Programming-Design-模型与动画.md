---
title: 3D-Game-Programming-Design-模型与动画
date: 2019-10-24 21:10:24
tags:
	- 课程作业
	- Unity
categories:
	- 3D游戏编程
---

> 3D游戏编程课程的第七次作业：
>
> +  智能巡逻兵 
>
> [课程主页]( https://pmlpml.github.io/unity3d-learning/07-model-and-animation )

<!-- more -->

## 智能巡逻兵

[项目地址]( https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/AI_Patrol )

[演示视频]( https://www.bilibili.com/video/av73372763/ )

### 设计要求

- 游戏设计要求：
  - 创建一个地图和若干巡逻兵(使用动画)；
  - 每个巡逻兵走一个3~5个边的凸多边型，位置数据是相对地址。即每次确定下一个目标位置，用自己当前位置为原点计算；
  - 巡逻兵碰撞到障碍物，则会自动选下一个点为目标；
  - 巡逻兵在设定范围内感知到玩家，会自动追击玩家；
  - 失去玩家目标后，继续巡逻；
  - 计分：玩家每次甩掉一个巡逻兵计一分，与巡逻兵碰撞游戏结束；
- 程序设计要求：
  - 必须使用订阅与发布模式传消息
  - 工厂模式生产巡逻兵

### 设计实现

#### 巡逻兵

##### 巡逻兵数据

```
public class PatrolData : MonoBehaviour
{
    public int manageFloor;       // 管理的房间
    public int plyerFloor;        // 玩家当前所在房间
    public Vector3 initPosition;  // 初始位置
}
```

##### 巡逻兵工厂

巡逻兵工厂负责产生一定数量、预定位置的巡逻兵，并记录每个巡逻兵相应的管辖范围。

```c#
public class PatrolFactory : MonoBehaviour
{
    private GameObject patrolPrefab = null;
    private Vector3[] position = new Vector3[6]; //记录巡逻兵的位置
    private List<GameObject> patrols = new List<GameObject>();

    public List<GameObject> GetPatrols()
    {
        int[] pos_x = { -10, 3, 18, -11, 3, 15 };
        int[] pos_z = { -18, -15, -15, 12, 10, 13 };

        for(int i = 0; i < 6; i++)
        {
            position[i] = new Vector3(pos_x[i], 0, pos_z[i]);
            patrolPrefab = Object.Instantiate(Resources.Load<GameObject>("Prefabs/patrol2"), position[i], Quaternion.identity);
            patrolPrefab.name = "patrol" + i;
            patrolPrefab.AddComponent<PatrolData>();

            patrolPrefab.GetComponent<PatrolData>().manageFloor = i + 1;
            patrolPrefab.GetComponent<PatrolData>().initPosition = position[i];
            patrols.Add(patrolPrefab);
        }
        return patrols;
    }

    public void Reset()
    {
        for(int i = 0; i < patrols.Count; i++)
        {
            patrols[i].transform.position = position[i];
            patrols[i].GetComponent<Animator>().SetBool("shoot", false);
        }
    }

}
```

##### 巡逻兵动作

巡逻兵动作一共有两种：普通巡逻和追踪敌人，动作的切换通过回调函数实现。

###### 巡逻动作

每个巡逻兵都按照简单的矩形路线不断重复运动，每当到达指定的顶点就换方向。在每次巡逻的同时，判断玩家所在区域是不是自己管辖的区域。如果是，则停止巡逻，开始追击玩家（销毁当前动作，执行回调函数）。

```c#
public class PatrolMoveAction : SSAction
{
    private float posX, posZ;
    private float rectLength;                           // 边长
    private enum Dirction { EAST, NORTH, WEST, SOUTH }; // 巡逻的四个方向
    private float speed = 2f;                           // 巡逻速度
    private bool reach = true;                          // 是否到达目的地
    private Dirction dirction = Dirction.EAST;          // 移动的方向
    private PatrolData data;                            // 巡逻兵的数据

    public static PatrolMoveAction GetSSAction(Vector3 location)
    {
        PatrolMoveAction action = CreateInstance<PatrolMoveAction>();
        action.posX = location.x;
        action.posZ = location.z;

        action.rectLength = Random.Range(5, 8);
        return action;
    }

    public override void Start()
    {
        data = this.gameObject.GetComponent<PatrolData>();
    }

    public override void Update()
    {
        //防止碰撞发生后的旋转
        if (transform.localEulerAngles.x != 0 || transform.localEulerAngles.z != 0)
        {
            transform.localEulerAngles = new Vector3(0, transform.localEulerAngles.y, 0);
        }
        if (transform.position.y != 0.5f)
        {
            transform.position = new Vector3(transform.position.x, 0.5f, transform.position.z);
        }
        // 移动
        Move();
         
        // 如果所在房间相同，摧毁当前动作并回调
        if (data.manageFloor == data.plyerFloor)
        {
            this.destory = true;
            this.callback.SSActionEvent(this, SSActionEventType.Compeleted, 0 ,"follow player", this.gameObject);
        }
    }

    public void Move()
    {
        if (reach)
        {
            // 如果已到达，就换方向
            switch (dirction)
            {
                case Dirction.EAST:
                    posX -= rectLength;
                    break;
                case Dirction.NORTH:
                    posZ += rectLength;
                    break;
                case Dirction.WEST:
                    posX += rectLength;
                    break;
                case Dirction.SOUTH:
                    posZ -= rectLength;
                    break;
            }
            reach = false;
        }

        //面朝目的地
        this.transform.LookAt(new Vector3(posX, 0.5f, posZ));

        //计算距离
        float distance = Vector3.Distance(transform.position, new Vector3(posX, 0.5f, posZ));
        if (distance > 1)
        {
            transform.position = Vector3.MoveTowards(this.transform.position, new Vector3(posX, 0.5f, posZ), speed * Time.deltaTime);
        }
        else
        {
            dirction = dirction + 1;
            if (dirction > Dirction.SOUTH)
            {
                dirction = Dirction.EAST;
            }
            reach = true;
        }
    }
}
```

###### 追踪玩家动作

追踪动作比较简单，创建该动作时会传入玩家对象，之后都朝着玩家的位置移动即可。在动作执行的同时判断玩家是否离开管辖区域，如果离开了，就重新开始巡逻（销毁当前动作，执行回调函数）。

```c#
public class PatrolFollowAction : SSAction
{     
    private GameObject player;        // 创建动作时传入玩家对象，以便对玩家进行追踪
    private float speed = 3f;         // 追踪玩家的速度
    private PatrolData data;          // 巡逻兵数据

    public static PatrolFollowAction GetSSAction(GameObject player)
    {
        PatrolFollowAction action = CreateInstance<PatrolFollowAction>();
        action.player = player;

        return action;
    }

    public override void Start()
    {
        data = this.gameObject.GetComponent<PatrolData>();
    }

    public override void Update()
    {
        if (transform.localEulerAngles.x != 0 || transform.localEulerAngles.z != 0)
        {
            transform.localEulerAngles = new Vector3(0, transform.localEulerAngles.y, 0);
        }
        if (transform.position.y != 0.5f)
        {
            transform.position = new Vector3(transform.position.x, 0.5f, transform.position.z);
        }

        transform.position = Vector3.MoveTowards(this.transform.position, player.transform.position, speed * Time.deltaTime);
        //追踪时面朝玩家
        this.transform.LookAt(player.transform.position);

        //丢失目标，停止追踪
        //如果侦察兵没有跟随对象，或者需要跟随的玩家不在侦查兵的区域内
        if (data.manageFloor != data.plyerFloor)
        {
            this.destory = true;
            this.callback.SSActionEvent(this, SSActionEventType.Compeleted, 1, "stop follow", this.gameObject);
        }
    }
}
```

##### 巡逻兵动作管理器

动作管理器主要负责的是在游戏的开始让巡逻兵开始巡逻，并通过回调函数处理不同的情况：根据回调的参数不同，进行巡逻动作和追踪动作的切换。同时，在由追踪切换为巡逻时，需要发布玩家逃脱的消息，进行后续的处理。

```c#
public class PatrolActionManager : SSActionManager, ISSActionCallback
{
    //巡逻动作
    private PatrolMoveAction move;
    private SceneController sceneController;

    protected new void Start()
    {
        sceneController = SSDirector.GetInstance().CurrentSceneController as SceneController;
    }

    public void PatrolMove(GameObject patrol)
    {
        move = PatrolMoveAction.GetSSAction(patrol.transform.position);
        this.RunAction(patrol, move, this);
    }

    #region ISSActionCallback implementation
    public void SSActionEvent(SSAction source,
        SSActionEventType events = SSActionEventType.Compeleted,
        int intParam = 0,
        string strParam = null,
        GameObject objectParam = null)
    {
        //回调函数,动作执行完后调用
        if (intParam == 0)
        {
            //开始跟随玩家
            PatrolFollowAction follow = PatrolFollowAction.GetSSAction(sceneController.player);
            this.RunAction(objectParam, follow, this);
        }
        else
        {
            //丢失目标，继续巡逻
            PatrolMoveAction move = PatrolMoveAction.GetSSAction(objectParam.gameObject.GetComponent<PatrolData>().initPosition);
            this.RunAction(objectParam, move, this);
            //玩家逃脱
            Singleton<GameEventManager>.Instance.PlayerEscape();
        }
    }
    #endregion
}
```

#### 玩家

##### 玩家移动

玩家移动的实现通过获取键盘输入并执行相应的函数完成。这里实现的移动可以通过方向键来控制，上下表示前进和后退，左右表示转向，类似于汽车游戏。

```c#
// UserGUI.cs
void Update()
{
    float transitionX = Input.GetAxis("Horizontal");
    float transitionZ = Input.GetAxis("Vertical");
    //移动玩家
    action.MovePlayer(transitionX, transitionZ);
    timeCounter = action.GetTime();
}


// SceneController.cs
public void MovePlayer(float x, float z)
{
    if (!gameOver)
    {
        //移动和旋转
        player.transform.Translate(0, 0, z * playerSpeed * Time.deltaTime);
        player.transform.Rotate(0, x * 135f * Time.deltaTime, 0);
        //防止碰撞带来的移动
        if (player.transform.localEulerAngles.x != 0 || player.transform.localEulerAngles.z != 0)
        {
            player.transform.localEulerAngles = new Vector3(0, player.transform.localEulerAngles.y, 0);
        }
        if (player.transform.position.y != 0.5f)
        {
            player.transform.position = new Vector3(player.transform.position.x, 0.5f, player.transform.position.z);
        }
    }
}
```

#### 碰撞事件

##### 玩家与地图碰撞

为不同的房间添加以下脚本，每个房间的`sign`值不同，通过玩家与不同房间地面的碰撞更新玩家的位置。

```c#
public class AreaCollide : MonoBehaviour
{
    public int sign = 0;
    SceneController sceneController;
    private void Start()
    {
        sceneController = SSDirector.GetInstance().CurrentSceneController as SceneController;
    }
    void OnTriggerEnter(Collider collider)
    {
        //标记玩家进入自己的区域
        if (collider.gameObject.name == "player")
        {
            Debug.Log("player enter floor " + sign);
            sceneController.floorNumber = sign;
        }
    }
}
```

##### 玩家与金币碰撞

为金币添加以下脚本，与玩家碰撞后设为不可见，并发布金币减少的消息。

```c#
public class CoinCollide : MonoBehaviour
{
    void OnTriggerEnter(Collider collider)
    {
        //玩家吃到金币事件触发
        if (collider.gameObject.name == "player")
        {
            this.gameObject.SetActive(false);
            Singleton<GameEventManager>.Instance.RecudeCoinNum();
        }
    }
}
```

##### 玩家与巡逻兵碰撞

为巡逻兵添加以下脚本，当巡逻兵与玩家相撞后，设置两者的动作，并发布玩家被捕的消息。

```c#
public class PatrolCollide : MonoBehaviour
{
    void OnCollisionStay(Collision other)
    {
        //当侦察兵与玩家相撞
        if (other.gameObject.name == "player")
        {
            other.gameObject.GetComponent<Animator>().SetBool("death", true);
            this.GetComponent<Animator>().SetBool("shoot", true);
            Singleton<GameEventManager>.Instance.PlayerArrested();
        }
    }
}
```

#### 订阅与发布模式实现

##### 发布者

发布者类共定义了三种消息类型：分数改变、游戏结束和硬币被玩家拾取。

```c#
public class GameEventManager : MonoBehaviour
{
    public delegate void ScoreEvent();
    public static event ScoreEvent ScoreChange;

    public delegate void GameOverEvent();
    public static event GameOverEvent GameOver;

    public delegate void CoinEvent();
    public static event CoinEvent CoinNumberChange;

    //玩家逃脱
    public void PlayerEscape()
    {
        if (ScoreChange != null)
        {
            ScoreChange();
        }
    }
    //玩家被捕
    public void PlayerArrested()
    {
        if (GameOver != null)
        {
            GameOver();
        }
    }

    public void RecudeCoinNum()
    {
        if(CoinNumberChange != null)
        {
            CoinNumberChange();
        }
    }

    public void TimeOut()
    {
        if (GameOver != null)
        {
            GameOver();
        }
    }
}
```

##### 订阅者

订阅者为场景控制器，实现的相关函数如下：

```c#
void OnEnable()
{
    //注册事件
    GameEventManager.ScoreChange += AddScore;
    GameEventManager.GameOver += GameOver;
    GameEventManager.CoinNumberChange += ReduceCoinNumber;
}
void OnDisable()
{
    //取消注册事件
    GameEventManager.ScoreChange -= AddScore;
    GameEventManager.GameOver -= GameOver;
    GameEventManager.CoinNumberChange -= ReduceCoinNumber;
}

void AddScore()
{
    scoreRecorder.AddScore();
}

void GameOver()
{
    gameOver = true;
    actionManager.DestroyAll();
}

void ReduceCoinNumber()
{
    coinNumberGet += 1;
}
```
#### 附加功能

##### 金币

金币是玩家获得游戏胜利的条件，需要拾起地上所有的金币才能结束游戏。

###### 金币工厂

金币也通过工厂模式生成，按照预定位置生成固定数量的金币。玩家每次与金币发生碰撞就把金币设为不可见即可，同时在重新开始游戏时全部设置成可见，这样金币就得到了复用，不需要重新生成新的。

```c#
public class CoinFactory : MonoBehaviour
{
    private GameObject coinPrefab = null;
    private Vector3[] position = new Vector3[6]; //记录金币的位置
    private List<GameObject> coins = new List<GameObject>();

    public List<GameObject> GetCoins()
    {
        int[] pos_x = {-15, 3, 13, -7, 9};
        int[] pos_z = {-15, -3, -15, 15, 10};

        for (int i = 0; i < 5; i++)
        {
            position[i] = new Vector3(pos_x[i], 1, pos_z[i]);
            coinPrefab = Object.Instantiate(Resources.Load<GameObject>("Prefabs/Coin"), position[i], Quaternion.identity);
            coinPrefab.name = "coin" + i;
            coins.Add(coinPrefab);
        }
        return coins;
    }

    public void Reset()
    {
        for (int i = 0; i < coins.Count; i++)
        {
            coins[i].SetActive(true);
        }
    }
}
```

##### 倒计时

为了增加游戏的趣味性，增加了倒计时功能，由`TimeManager`类实现。当倒计时为0后，游戏结束。

```c#
public class TimeManager : MonoBehaviour
{
    private float gameTime = 90f;
    private float timer = 0;
    private string timeCounter;

    // Use this for initialization
    void Start()
    {

    }

    public void Reset()
    {
        gameTime = 90f;
    }

    public string GetTimeText()
    {
        return timeCounter;
    }

    // Update is called once per frame
    void Update()
    {
        int M = (int)(gameTime / 60);
        float S = gameTime % 60;

        timer += Time.deltaTime;
        if (timer >= 1f)
        {
            timer = 0;
            gameTime--;
            timeCounter = M.ToString() + ":" + string.Format("{0:00}", S);
        }

        if (gameTime == 0)
        {
            Singleton<GameEventManager>.Instance.TimeOut();
        }
    }
}
```

##### 相机跟随

相机跟随的实现方式就是让相机在上空与玩家始终保持固定距离，产生相机跟随着玩家移动的效果，增强游戏的真实感。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraFollow : MonoBehaviour
{
    public GameObject player;
    public float smothing = 5f;
    Vector3 offset;    

    void Start()
    {
        offset = new Vector3(0, 25, -20);
    }

    void FixedUpdate()
    {
        Vector3 target = player.transform.position + offset;
        transform.position = Vector3.Lerp(transform.position, target, smothing * Time.deltaTime);
    }
}
```

#### 实现结果

游戏画面如下：

{% asset_img game.png %}
