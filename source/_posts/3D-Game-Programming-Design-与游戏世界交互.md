---
title: 3D-Game-Programming-Design-与游戏世界交互
date: 2019-10-6 23:23:45
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程的第五次作业：
>
> - 简单的鼠标打飞碟（Hit UFO）游戏
>
> [课程主页](https://pmlpml.github.io/unity3d-learning/05-interaction-with-gameworld)

<!-- more -->

### 鼠标打飞碟游戏

#### 游戏要求

> 游戏内容要求：
>
> + 游戏有 n 个 round，每个 round 都包括10 次 trial；
> + 每个 trial 的飞碟的色彩、大小、发射位置、速度、角度、同时出现的个数都可能不同。它们由该 round 的 ruler 控制；
> + 每个 trial 的飞碟有随机性，总体难度随 round 上升；
> + 鼠标点中得分，得分规则按色彩、大小、速度不同计算，规则可自由设定
>
> 游戏要求：
>
> + 使用带缓存的工厂模式管理不同飞碟的生产与回收，该工厂必须是场景单实例的！具体实现见参考资源 Singleton 模板类
> + 尽可能使用 MVC 结构实现人机交互与游戏模型分离
>

#### 游戏实现

[项目地址](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Hit_UFO)

[演示视频地址](https://www.bilibili.com/video/av70636531/)

游戏实现了：

+ 共3个Round，每回合10次Trial；
+ 每次Trial中飞碟的颜色、大小、位置，个数等随Round的提升而改变，随机产生但总体难度呈上升趋势；Round 1固定每次只有一个飞碟，后续Round会有几率增加，考虑到同时出现太多点不过来的情况，没有设计同时出现三个以上的情况。
+ 不同颜色飞碟的击中得分不同，游戏会记录玩家历次游戏的最高得分；
+ 玩家共有10次MISS的机会，超过十次则游戏失败。

游戏设计模式沿用之前的MVC模式，并按照要求使用工厂模式生产飞碟。

3个Round，每回合10次Trial，每次Trial中飞碟的颜色、大小、位置等随Round的提升而改变，总体难度呈上升趋势，不同颜色飞碟不同。

#### 实现过程分析

在本次的游戏设计要求中，对于之前的游戏设计思路又进行了扩展，新增加了工厂模式。具体要求是使用带缓存的工厂模式管理不同飞碟的生产与回收，且工厂必须是场景单实例的。

飞碟工厂用于单独管理预制飞碟的创建和回收，而不再像之前一样由场景控制器管理，使得代码进一步解耦。以下是飞碟工厂的UML图：

{% asset_img UML.png %}

下面介绍一些主要的类，一些基类和接口和之前相同，在这里就不再重复展示了。

##### 飞碟数据类DiskData

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DiskData : MonoBehaviour
{
    //射击此飞碟得分
    public int score = 1;
    //颜色
    public Color color = Color.red;
    //速度
    public float speed = 20;
    //方向
    public Vector3 direction;
}
```

##### 飞碟工厂类DiskFactory

带缓存的单实例飞碟工厂主要实现了对于飞碟的创建、管理和回收。他的使用有以下好处：

+ 包装了复杂的Disk生产与回收逻辑，易于使用；
+ 带缓存的工厂来避免频繁的创建与销毁操作，提高效率：通过两个List实现；
+ 包含了Disk产生规则（控制每个round的难度），可以积极应对未来游戏规则的变化，减少维护成本。

```c#
public class DiskFactory : MonoBehaviour
{
    public GameObject diskPrefab = null;
    private List<DiskData> used = new List<DiskData>();
    private List<DiskData> free = new List<DiskData>();

    public GameObject GetDisk(int round)
    {
        if (free.Count > 0)
        {
            diskPrefab = free[0].gameObject;
            free.Remove(free[0]);
        }
        else
        {
            diskPrefab = Instantiate<GameObject>(Resources.Load<GameObject>("Prefabs/disk"), Vector3.zero, Quaternion.identity);
            diskPrefab.AddComponent<DiskData>();
        }

        int start = 0, end = 0, diskType = 0;
        if(round == 1)
        {
            start = 0;
            end = 300;
        }
        if (round == 2) {
            start = 200;
            end = 400;
        }
        else if (round == 3)
        {
            start = 300;
            end = 500;
        }

        int temp = Random.Range(start, end);

        if (temp > 400)
        {
            diskType = 5;
        }
        else if (temp > 300)
        {
            diskType = 4;
        }
        else if (temp > 200)
        {
            diskType = 3;
        }
        else if (temp > 100)
        {
            diskType = 2;
        }
        else
        {
            diskType = 1;
        }

        //生成不同的飞碟
        switch (diskType)
        {
            case 1:
                {
                    diskPrefab.GetComponent<DiskData>().color = Color.white;
                    diskPrefab.GetComponent<DiskData>().speed = 4.0f;
                    float RanX = UnityEngine.Random.Range(-1f, 1f) < 0 ? -1 : 1;
                    diskPrefab.GetComponent<DiskData>().direction = new Vector3(RanX, 1, 0);
                    diskPrefab.GetComponent<Renderer>().material.color = Color.white;
                    diskPrefab.GetComponent<DiskData>().score = 1;
                    diskPrefab.transform.localScale = new Vector3(2f, 0.2f, 2f);
                    break;
                }
            case 2:
                {
                    diskPrefab.GetComponent<DiskData>().color = Color.green;
                    diskPrefab.GetComponent<DiskData>().speed = 6.0f;
                    float RanX = UnityEngine.Random.Range(-1f, 1f) < 0 ? -1 : 1;
                    diskPrefab.GetComponent<DiskData>().direction = new Vector3(RanX, 1, 0);
                    diskPrefab.GetComponent<Renderer>().material.color = Color.green;
                    diskPrefab.GetComponent<DiskData>().score = 2;
                    diskPrefab.transform.localScale = new Vector3(1.6f, 0.16f, 1.6f);
                    break;
                }
            case 3:
                {
                    diskPrefab.GetComponent<DiskData>().color = Color.blue;
                    diskPrefab.GetComponent<DiskData>().speed = 8.0f;
                    float RanX = UnityEngine.Random.Range(-1f, 1f) < 0 ? -1 : 1;
                    diskPrefab.GetComponent<DiskData>().direction = new Vector3(RanX, 1, 0);
                    diskPrefab.GetComponent<Renderer>().material.color = Color.blue;
                    diskPrefab.GetComponent<DiskData>().score = 3;
                    diskPrefab.transform.localScale = new Vector3(1.4f, 0.14f, 1.4f);
                    break;
                }
            case 4:
                {
                    diskPrefab.GetComponent<DiskData>().color = Color.red;
                    diskPrefab.GetComponent<DiskData>().speed = 6.0f;
                    float RanX = UnityEngine.Random.Range(-1f, 1f) < 0 ? -1 : 1;
                    diskPrefab.GetComponent<DiskData>().direction = new Vector3(RanX, 1, 0);
                    diskPrefab.GetComponent<Renderer>().material.color = Color.red;
                    diskPrefab.GetComponent<DiskData>().score = 4;
                    diskPrefab.transform.localScale = new Vector3(1.0f, 0.1f, 1.0f);
                    break;
                }
            case 5:
                {
                    diskPrefab.GetComponent<DiskData>().color = Color.black;
                    diskPrefab.GetComponent<DiskData>().speed = 8.0f;
                    float RanX = UnityEngine.Random.Range(-1f, 1f) < 0 ? -1 : 1;
                    diskPrefab.GetComponent<DiskData>().direction = new Vector3(RanX, 1, 0);
                    diskPrefab.GetComponent<Renderer>().material.color = Color.black;
                    diskPrefab.GetComponent<DiskData>().score = 5;
                    diskPrefab.transform.localScale = new Vector3(0.8f, 0.08f ,0.8f);
                    break;
                }
            default:break;

        }

        used.Add(diskPrefab.GetComponent<DiskData>());
        diskPrefab.SetActive(false);
        return diskPrefab;
    }

    public void FreeDisk(GameObject disk)
    {
        foreach (DiskData dd in used)
        {
            if(disk.GetInstanceID() == dd.gameObject.GetInstanceID())
            {
                dd.gameObject.SetActive(false);
                used.Remove(dd);
                free.Add(dd);
                break;
            }
        }
    }
}
```

##### 场景控制器 RoundController

场景控制器统筹管理各个组件，管理游戏的整体进程。`LoadResource`函数会将当前的回合情况传递给飞碟工厂，以产生相应难度的飞碟。`Update`函数每隔一段时间触发一次Trial，即发送飞碟，同时判断是否继续还是结束游戏，并对当前未被击中的飞碟进行回收。此外，还负责处理用户射击事件，判断是否击中并执行后续的行为，如用户扣血，或者分数增加。

```c#
public class RoundController : MonoBehaviour, ISceneController, IUserAction
{
    public DiskFactory diskFactory;
    public CCActionManager actionManager;
    public ScoreRecorder scoreRecorder;
    public UserGUI userGui;

    private Queue<GameObject> diskQueue = new Queue<GameObject>();
    private List<GameObject> diskMissed = new List<GameObject>();

    private int totalRound = 3;
    private int trialNumPerRound = 10;
    private int currentRound = -1;
    private int currentTrial = -1;
    private float throwSpeed = 2f;
    private int gameState = 0; //-1：失败 0：初始状态 1：进行中 2：胜利
    private float throwInterval = 0;
    private int userBlood = 10;

    void Awake()
    {
        SSDirector director = SSDirector.GetInstance();
        director.CurrentSceneController = this;
        
        diskFactory = Singleton<DiskFactory>.Instance;
        userGui = gameObject.AddComponent<UserGUI>() as UserGUI;
        actionManager = gameObject.AddComponent<CCActionManager>() as CCActionManager;

        scoreRecorder = new ScoreRecorder();
    }

    public void LoadResource()
    {
        diskQueue.Enqueue(diskFactory.GetDisk(currentRound));
    }

    public void ThrowDisk(int count)
    {
        while(diskQueue.Count <= count)
        {
            LoadResource();
        }
        for(int i = 0; i < count; i++)
        {
                    float position_x = 16;
        GameObject disk = diskQueue.Dequeue();
        diskMissed.Add(disk);
        disk.SetActive(true);
        //设置飞碟位置
        float ran_y = Random.Range(-3f, 3f);
        float ran_x = Random.Range(-1f, 1f) < 0 ? -1 : 1;
        disk.GetComponent<DiskData>().direction = new Vector3(ran_x, ran_y, 0);
        Vector3 position = new Vector3(-disk.GetComponent<DiskData>().direction.x * position_x, ran_y, 0);
        disk.transform.position = position;
        //设置飞碟初始所受的力和角度
        float power = Random.Range(10f, 15f);
        float angle = Random.Range(15f, 28f);
        actionManager.diskFly(disk, angle, power);
        }
    }

    void levelUp()
    {
        currentRound += 1;
        throwSpeed -= 0.5f;
        currentTrial = 1;
    }

    void Update()
    {
        if(gameState == 1)
        {
            if(userBlood <= 0 || (currentRound == totalRound && currentTrial == trialNumPerRound))
            {
                GameOver();
                return;
            }
            else
            {
                if (currentTrial > trialNumPerRound)
                {
                    levelUp();
                }
                if (throwInterval > throwSpeed)
                {
                    int throwCount = generateCount(currentRound);
                    ThrowDisk(throwCount);
                    throwInterval = 0;
                    currentTrial += 1;
                }
                else
                {
                    throwInterval += Time.deltaTime;
                }
            }
        }
        for (int i = 0; i < diskMissed.Count; i++)
        {
            GameObject temp = diskMissed[i];
            //飞碟飞出摄像机视野且未被打中
            if (temp.transform.position.y < -8 && temp.gameObject.activeSelf == true)
            {
                diskFactory.FreeDisk(diskMissed[i]);
                diskMissed.Remove(diskMissed[i]);
                userBlood -= 1;
            }
        }
    }

    public int generateCount(int currentRound)
    {
        if(currentRound == 1)
        {
            return 1;
        }
        else if(currentRound == 2)
        {
            return Random.Range(1, 2);
        }
        else
        {
            return Random.Range(1, 3);
        }
    }


    public void StartGame()
    {
        gameState = 1;
        currentRound = 1;
        currentTrial = 1;
        userBlood = 10;
        throwSpeed = 2f;
        throwInterval = 0;
}

    public void GameOver()
    {
        if(userBlood <= 0)
        {
            gameState = -1;//失败
        }
        else
        {
            gameState = 2;//胜利
        }
    }

    public void Restart()
    {
        scoreRecorder.Reset();
        StartGame();
    }

    public void Hit(Vector3 pos)
    {
        Ray ray = Camera.main.ScreenPointToRay(pos);
        RaycastHit[] hits;
        hits = Physics.RaycastAll(ray);
        bool notHit = false;
        foreach (RaycastHit hit in hits)
            //射线打中物体
            if (hit.collider.gameObject.GetComponent<DiskData>() != null)
            {
                //射中的物体要在没有打中的飞碟列表中
                for (int j = 0; j < diskMissed.Count; j++)
                {
                    if (hit.collider.gameObject.GetInstanceID() == diskMissed[j].gameObject.GetInstanceID())
                    {
                        notHit = true;
                    }
                }
                if (!notHit)
                {
                    return;
                }
                diskMissed.Remove(hit.collider.gameObject);
                //记录分数
                scoreRecorder.Record(hit.collider.gameObject);
                diskFactory.FreeDisk(hit.collider.gameObject);
                break;
            }
    }

    public int GetScore()
    {
        return scoreRecorder.GetScore();
    }

    public int GetCurrentRound()
    {
        return currentRound;
    }
    public int GetBlood()
    {
        return userBlood;
    }
    public int GetGameState()
    {
        return gameState;
    }
}
```

##### 飞碟动作类CCFlyAction

该类负责实现飞碟的飞行效果，并在下降到一定高度后结束该动作。

```c#
public class CCFlyAction : SSAction
{
    public float gravity = -5;                                 //向下加速度
    private Vector3 startVector;                               //初速度向量
    private Vector3 gravityVector = Vector3.zero;              //加速度的向量，初始时为0
    private float time;                                        //已过去时间
    private Vector3 currentAngle = Vector3.zero;              //当前时间的欧拉角


    public static CCFlyAction GetSSAction(Vector3 direction, float angle, float power)
    {
        CCFlyAction action = CreateInstance<CCFlyAction>();
        if (direction.x == -1)
        {
            action.startVector = Quaternion.Euler(new Vector3(0, 1, -angle)) * Vector3.left * power;
        }
        else
        {
            action.startVector = Quaternion.Euler(new Vector3(0, 1, angle)) * Vector3.right * power;
        }
        return action;
    }

    public override void Start()
    {
    }

    public override void Update()
    {
        time += Time.fixedDeltaTime;
        gravityVector.y = gravity * time;

        transform.position += (startVector + gravityVector) * Time.fixedDeltaTime;
        currentAngle.z = Mathf.Atan((startVector.y + gravityVector.y) / startVector.x) * Mathf.Rad2Deg;
        transform.eulerAngles = currentAngle;

        if (this.transform.position.y < -10)
        {
            this.destory = true;
            this.callback.SSActionEvent(this);
        }
    }
}
```

##### 用户交互界面

交互界面比较简单，就是简单的根据游戏的进行状态显示不同的信息，如开始前显示开始游戏，进行时显示当前回合、得分、玩家血量，结束后显示重新开始等字样。

```c#
public class UserGUI : MonoBehaviour
{
    private IUserAction action;
    private string score, round;
    int blood, gameState, HighestScore;

    // Start is called before the first frame update
    void Start()
    {
        action = SSDirector.GetInstance().CurrentSceneController as IUserAction;
    }

    // Update is called once per frame
    void Update()
    {
        gameState = action.GetGameState();
    }

    void OnGUI()
    {
        GUIStyle text_style;
        GUIStyle button_style;
        text_style = new GUIStyle()
        {
            fontSize = 20
        };
        button_style = new GUIStyle("button")
        {
            fontSize = 15
        };

        if (gameState == 0)
        {
            //初始界面
            if (GUI.Button(new Rect(Screen.width / 2 - 50, 80, 100, 60), "Start Game", button_style))
            {
                action.StartGame();
            }
        }
        else if(gameState == 1)
        {
            //游戏进行中
            //用户射击
            if (Input.GetButtonDown("Fire1"))
            {
                Vector3 mousePos = Input.mousePosition;
                action.Hit(mousePos);
            }

            score = "Score: " + action.GetScore().ToString();
            GUI.Label(new Rect(200, 5, 100, 100), score, text_style);
            round = "Round: " + action.GetCurrentRound().ToString();
            GUI.Label(new Rect(400, 5, 100, 100), round, text_style);

            blood = action.GetBlood();
            string bloodStr = "Blood: " + blood.ToString();
            GUI.Label(new Rect(600, 5, 50, 50), bloodStr, text_style);
        }
        else
        {
            //游戏结束，有两种情况
            if (gameState == 2)
            {

                if (action.GetScore() > HighestScore) {
                    HighestScore = action.GetScore();
                }       
                GUI.Label(new Rect(Screen.width / 2 - 50, Screen.height / 2 - 250, 100, 60), "Game Over", text_style);
                string record = "Highest Score: " + HighestScore.ToString();  
                GUI.Label(new Rect(Screen.width / 2 - 70, Screen.height / 2 - 150, 150, 60), record, text_style);
            }
            else
            {
                GUI.Label(new Rect(Screen.width / 2 - 50, Screen.height / 2 - 150, 100, 70), "You Lost!", text_style);
            }

            if (GUI.Button(new Rect(Screen.width / 2 - 50, Screen.height / 2 - 30, 100, 60), "Restart", button_style))
            {
                action.Restart();
            }
        }
    }     
}
```

#### 游戏截图

{% asset_img Game.png %}

#### 改进空间

+ 飞碟重用机制的实现比较粗糙。由于我判断工厂是否需要回收飞碟的方式是：飞碟落回到一定高度下后就回收。这就导致了由于飞碟速度的不同，有时候飞碟回落过慢，导致还未回收，就又需要另外一个飞碟，就会多创建一些新的飞碟出来。
+ 用户界面比较粗糙，不够精美。
+ 用户射中飞碟后可以添加一些爆炸效果。