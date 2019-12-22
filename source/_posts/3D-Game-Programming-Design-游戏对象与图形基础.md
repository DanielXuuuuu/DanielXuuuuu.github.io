---
title: 3D-Game-Programming-Design-游戏对象与图形基础
date: 2019-09-22 10:38:02
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程的第四次作业：
>
> + 牧师与恶魔 - 动作分离版
>
> [课程主页](https://pmlpml.github.io/unity3d-learning/04-gameobject-and-graphics)

<!-- more -->

##### 基本操作演练

> 下载 Fantasy Skybox FREE， 构建自己的游戏场景

从Asset Store中搜索并下载Fantasy Skybox FREE后，其中包含了好几个skybox，用于设置游戏界面的背景。同时还有一些树的预制Prefabs，地面图片等等，可以用于制作地形Terrain。但由于这个资源是免费的，个人感觉并不是很好看。

> 写一个简单的总结，总结游戏对象的使用

目前为止，我对于游戏对象的使用还只是停留在最基本的操作上：移动，缩放，旋转等等，并可以通过编写脚本来对于游戏对象进行加载和控制，来达到一些简单的游戏功能。

此外还因为从Asset Store中下载的模型自带动作，我简单的尝试了对于游戏对象动作的控制，关键在于设定具体的状态，将动作与状态绑定，并设置状态之间的转移。

##### 编程实践

> 牧师与魔鬼 动作分离版
>
> + 【2019新要求】：设计一个裁判类，当游戏达到结束条件时，通知场景控制器游戏结束

[项目地址](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Priests_and_Devils_Pro)

由于只是变更了代码结构，实际游戏效果并没有改变，所以视频沿用上一次的：[演示视频地址](https://www.bilibili.com/video/av68331555) 

在[上一次的作业]([https://danielxuuuuu.github.io/2019/09/16/3D-Game-Programming-Design-%E7%A9%BA%E9%97%B4%E4%B8%8E%E8%BF%90%E5%8A%A8/#more](https://danielxuuuuu.github.io/2019/09/16/3D-Game-Programming-Design-空间与运动/#more))中，对于牧师与魔鬼这个游戏，我们采用了MVC的设计架构，使得程序在一定程度上解耦，每一层可以专注于自己的业务逻辑。但是，在实践中发现，即使对于这样一个简单的小游戏，在控制器层所要做的事情还是太多了，既要负责加载资源，同时要处理模型间很多动作的执行与判断，导致代码比较冗长。

因此，本次作业中我们将写一个动作管理器，负责单独管理动作。除此之外，还将游戏结束判断的代码也提取了出来，写了一个裁判类。

以下是动作管理器的UML类图：

{% asset_img actionManageUML.png %}

基于上述类，修改游戏的脚本结构如下：

{% asset_img scripts.png %}

下面简单介绍一些这些类的关系及其职责：

###### SSAction

SSAction是所有动作的基类。包含了一些所有动作都会用到的属性。

```c#
public class SSAction : ScriptableObject
{
    public bool enable = true;
    public bool destory = false;

    public GameObject gameObject { get; set; }
    public Transform transform { get; set; }
    public ISSActionCallback callback { get; set; }

    protected SSAction() { }

    public virtual void Start()
    {
        throw new System.NotImplementedException();
    }
    public virtual void Update()
    {
        throw new System.NotImplementedException();
    }
}
```

###### SSActionCallback

类似于回调函数，作为动作和动作管理器交流的一种方式。我们可以看到，在动作基类中，存在`ISSActionCallbcak`属性，而动作管理器类会继承这个接口，并实现其方法。每当动作管理器执行某个动作时，会把该动作的这个属性设为自己。因此，当动作完成后，就可以通过`callback.SSActionEvent`这个函数通知管理器。

```c#
public enum SSActionEventType : int { Started, Compeleted }

public interface ISSActionCallback
{
    void SSActionEvent(SSAction source,
        SSActionEventType events = SSActionEventType.Compeleted,
        int intParam = 0,
        string strParam = null,
        Object objectParam = null);
}
```

###### CCMoveToAction

简单的移动，从上次作业的代码中提取出的

```c#
public class CCMoveToAction : SSAction
{
    public Vector3 target;
    public float speed;

    public static CCMoveToAction GetSSAction(Vector3 target, float speed)
    {
        CCMoveToAction action = ScriptableObject.CreateInstance<CCMoveToAction>();
        action.target = target;
        action.speed = speed;
        return action;
    }

    // Use this for initialization
    public override void Start()
    {
        
    }

    // Update is called once per frame
    public override void Update()
    {
        this.transform.position = Vector3.MoveTowards(this.transform.position, target, speed * Time.deltaTime);
        if(this.transform.position == target)
        {
            this.destory = true;
            this.callback.SSActionEvent(this);
        }
    }
}
```

###### CCSequenceAction

这个类用于实现一系列连续的动作，通过`List`保存所要执行的一系列动作，采用了**门面设计模式**，将连续的动作封装。因此，它有着动作管理器的效果，所以也实现了ISSActionCallback接口。当一系列动作完成后，先通知这个类，然后这个类通知更高一级的管理器。

```c#
public class CCSequenceAction : SSAction, ISSActionCallback
{
    public List<SSAction> sequence;
    public int repeat = -1;
    public int currentIndex = 0;

    public static CCSequenceAction GetSSAction(int repeat, int currentIndex, List<SSAction> sequence)
    {
        CCSequenceAction action = ScriptableObject.CreateInstance<CCSequenceAction>();
        action.repeat = repeat;
        action.currentIndex = currentIndex;
        action.sequence = sequence;
        return action;
    }

    // 执行动作前，为每个动作注入当前动作游戏对象，并将自己作为动作事件的接收者
    public override void Start()
    {
        foreach(SSAction action in sequence)
        {
            action.gameObject = this.gameObject;
            action.transform = this.transform;
            action.callback = this;
            action.Start();
        }
    }

    // Update is called once per frame
    public override void Update()
    {
        if (sequence.Count == 0)
            return;
        if(currentIndex < sequence.Count)
        {
            sequence[currentIndex].Update();
        }
    }

    public void SSActionEvent(SSAction source, SSActionEventType events = SSActionEventType.Compeleted, int intParam = 0, string strParam = null, Object objectParam = null)
    { 
        source.destory = false;
        this.currentIndex++;
        if(this.currentIndex >= sequence.Count)
        {
            this.currentIndex = 0;
            if (repeat > 0)
                repeat--;
            if(repeat == 0)
            {
                this.destory = true;
                this.callback.SSActionEvent(this);
            }
        }
    }

    private void OnDestroy()
    {
        foreach(SSAction action in sequence)
        {
            Object.Destroy(action);
        }
    }
}
```

###### SSActionManager

动作管理器基类，实现了动作管理器的核心功能。在每次Update时，检查所有动作，通过两个队列和一个字典数据结构完成当前所有动作的添加、执行、销毁操作。

```c#
public class SSActionManager : MonoBehaviour
{
    private Dictionary<int, SSAction> actions = new Dictionary<int, SSAction>();
    private List<SSAction> waitingAdd = new List<SSAction>();
    private List<int> waitingDelete = new List<int>();
    
    
    // Use this for initialization
    protected void Start()
    {

    }

    // Update is called once per frame
    protected void Update()
    {
        foreach (SSAction ac in waitingAdd)
        {
            actions[ac.GetInstanceID()] = ac;
        }
        waitingAdd.Clear();

        foreach(KeyValuePair<int, SSAction> kv in actions)
        {
            SSAction ac = kv.Value;
            if (ac.destory)
            {
                waitingDelete.Add(ac.GetInstanceID());
            }else if (ac.enable)
            {
                ac.Update();
            }
        }

        foreach(int key in waitingDelete)
        {
            SSAction ac = actions[key];
            actions.Remove(key);
            Object.Destroy(ac);
        }
        waitingDelete.Clear();
    }

    public void RunAction(GameObject gameobject, SSAction action, ISSActionCallback manager)
    {
        action.gameObject = gameobject;
        action.transform = gameobject.transform;
        action.callback = manager;
        waitingAdd.Add(action);
        action.Start();
    }
}
```

###### CCActionManager

具体的动作管理器，其中Update函数沿用父类的，并定义了具体的动作和回调函数。通过父类的`RunAction`为动作设置对象等信息，并将动作加入等待队列。

```c#
public class CCActionManager : SSActionManager, ISSActionCallback
{
    public FirstController sceneController;
    public CCMoveToAction moveBoat; 
    public CCSequenceAction moveRole; //移动角色是一个组合动作
    // Use this for initialization
    protected new void Start()
    {
        sceneController = SSDirector.GetInstance().CurrentSceneController as FirstController;
        sceneController.actionManager = this;
    }

    // Update is called once per frame
    protected new void Update()
    {
        base.Update();
    }

    public void MoveBoat(GameObject boat, Vector3 target, float speed)
    {
        moveBoat = CCMoveToAction.GetSSAction(target, speed);
        this.RunAction(boat, moveBoat, this);
        sceneController.moving = true;
    }

    public void MoveRole(GameObject role, Vector3 middlePosition, Vector3 endPosition, float speed)
    {
        SSAction step1 = CCMoveToAction.GetSSAction(middlePosition, speed);
        SSAction step2 = CCMoveToAction.GetSSAction(endPosition, speed);
        moveRole = CCSequenceAction.GetSSAction(1, 0, new List<SSAction> { step1, step2 });
        this.RunAction(role, moveRole, this);
        sceneController.moving = true;
    }

    #region ISSActionCallback implementation
    public void SSActionEvent(SSAction source, 
        SSActionEventType events = SSActionEventType.Compeleted,
        int intParam = 0,
        string strParam = null,
        Object objectParam = null)
    {
        //回调函数,动作执行完后调用
        sceneController.moving = false;
        
    }
    #endregion
}
```

以上就是动作管理器的全部类，在完成这些类后，需要对于之前的代码进行修改，最主要的是以下函数，其余细节部分此处不再赘述，具体可以见项目地址。

```c#
 public void MoveRole(RoleModel role)
    {
        if (!gaming || moving)
            return;
        Vector3 endPosition;

        if (role.IsOnBoat())        //上岸
        {
            //这里，为了和原游戏一致，使牧师和魔鬼在两边的排列顺序一致
            role = Boat.DeletePassenger(role);
            int id = role.GetName()[role.GetName().Length - 1] - '0';
            if (role.IsGood())
            {
                endPosition = PrisetsOriginPositions[id];
            }
            else
            {
                endPosition = DevilsOriginPositions[id];
            }
            if(role.GetSide() == -1)
            {
                endPosition.x = 0 - endPosition.x;
            }
        
            Vector3 middlePosition = new Vector3(role.GetRole().transform.position.x, endPosition.y, endPosition.z);
            actionManager.MoveRole(role.GetRole(), middlePosition, endPosition, speed);
            role.GoLand();
        }
        else                        //上船
        {
            if (Boat.IsFull() || Boat.GetSide() != role.GetSide())
            {
                return;

            }
            endPosition = Boat.getEmptyPosition();
            Vector3 middlePosition = new Vector3(endPosition.x, role.GetRole().transform.position.y, endPosition.z);
            actionManager.MoveRole(role.GetRole(), middlePosition, endPosition, speed);
            role.GoBoat(Boat);
            Boat.AddPassenger(role);
        }
    }
    public void MoveBoat()
    {
        //当船为空，或者船或人物在运动时，不允许移动船
        if (Boat.IsEmpty() || moving)
            return;
        actionManager.MoveBoat(Boat.GetBoat(), Boat.GetMoveDirection(), speed);
    }
}
```

可以看到，真正动作的执行代码已经很少，全部交由`actionManager`完成。

###### 裁判类

裁判类比较简单，只是把之前用于判断结果的代码单独写成一个类即可。我采用了让其在每次Update的时候进行判断，似乎频率有点过高，应该有更好的办法，但对于这个小游戏还是没有影响的。以下是裁判类代码：

```c#
public class Judger : MonoBehaviour
{
    public FirstController sceneController;
    // Use this for initialization
    public void Start()
    {
        sceneController = SSDirector.GetInstance().CurrentSceneController as FirstController;
    }

    // Update is called once per frame
    public void Update()
    {
        if((sceneController.currentState = Judge()) != 0)
        {
            sceneController.gaming = false;
            
        }
    }

    public int Judge()
    {
        if (sceneController.moving)
            return 0;
        //计算两边的牧师和恶魔数量
        int rightPriestNum = 0, leftPriestNum = 0, rightDevilNum = 0, leftDevilNum = 0;
        for (int i = 0; i < 3; i++)
        {
            if (sceneController.Priests[i].GetSide() == 1)
            {
                rightPriestNum++;
            }
            else
            {
                leftPriestNum++;
            }

            if (sceneController.Devils[i].GetSide() == 1)
            {
                rightDevilNum++;
            }
            else
            {
                leftDevilNum++;
            }
        }
        if (leftPriestNum + leftDevilNum == 6)
        {
            for (int i = 0; i < 3; i++)
            {
                sceneController.Devils[i].Lose();
            }
            return 1; //win
        }
        else if ((leftPriestNum > 0 && leftDevilNum > leftPriestNum) || (rightPriestNum > 0 && rightDevilNum > rightPriestNum))
        {
            int attackSide;
            if (leftDevilNum > leftPriestNum)
                attackSide = -1;
            else
                attackSide = 1;
            for (int i = 0; i < 3; i++)
            {
                if (sceneController.Devils[i].GetSide() == attackSide)
                    sceneController.Devils[i].Attack();
            }
            return -1; //lose
        }
        else
        {
            return 0;
        }
    }
}
```

##### 游戏界面

游戏界面也与上次相同：

{% asset_img p&d.png %}

##### 感悟

本次的作业，主要是对于上一次作业的代码结构进行了调整，使得各部分代码各司其职，分工合作。但在初看这些类关系的时候，确实让我费了一番功夫才看明白。但也真的学习到了不少，让我对于游戏框架设计有了更加深入的认识，特别是回调函数`callback`的使用，在一些特定的场景十分有效。

遗憾的是本来想对游戏画面进行进一步完善的，但是近期作业较多，没有抽出时间，希望以后可以有机会。