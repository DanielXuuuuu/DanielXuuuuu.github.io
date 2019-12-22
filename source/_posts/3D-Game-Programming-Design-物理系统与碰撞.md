---
title: 3D-Game-Programming-Design - 物理系统与碰撞
date: 2019-10-17 09:26:06
tags:
	- 课程作业
	- Unity
categories:
	- 3D游戏编程
---

> 3D游戏编程课程的第六次作业：
>
> + 改进上一次的[飞碟（Hit UFO）游戏](https://danielxuuuuu.github.io/2019/10/06/3D-Game-Programming-Design-%E4%B8%8E%E6%B8%B8%E6%88%8F%E4%B8%96%E7%95%8C%E4%BA%A4%E4%BA%92/)
>   + 按 *adapter模式* 设计图修改飞碟游戏
>   + 使它同时支持物理运动与运动学（变换）运动
>
> [课程主页](https://pmlpml.github.io/unity3d-learning/06-physics-and-collision)

<!-- more -->

### 飞碟游戏改进

[项目地址](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Hit_UFO_Pro)

[演示视频](https://www.bilibili.com/video/av71508540/)

#### 设计图分析

根据题目要求，我们先来看看老师在课件中给出的adapter模式设计图：

{% asset_img UML.png %}

分析上述UML图，可以发现：我们要实现的目标就是增加一个`PhysisActionManager`，并设计一个统一的抽象接口，使得新的动作管理器和之前的`CCActionManager`都实现它。这样一来，我们就可以选择其中的一个了。

#### 适配器模式简介

+ **意图：**将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

+ **主要解决：**主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。

+ **何时使用：** 1、系统需要使用现有的类，而此类的接口不符合系统的需要。 2、想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。 3、通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。）

+ **如何解决：**继承或依赖（推荐）。

+ **关键代码：**适配器继承或依赖已有的对象，实现想要的目标接口。

适配器模式通常由两种实现方式：**类适配器模式**和**对象适配器模式**，区别就在于前者通过继承实现，后者通过委托（delegate）实现。具体在这里就不展开赘述了，具体可以见这个[网站](https://blog.csdn.net/qq_36982160/article/details/79965027)，就老师给出的UML图而言，我们使用的是**类适配器**，即通过继承方式实现，但似乎老师的UML图给出的不太符合真正的适配器模式，只是抽象出了一个接口而已，不过按照老师的设计图做就可以了。

#### 实现流程

##### 新增接口`IActionManager`

```c#
public interface IActionManager
{
    void playDisk(GameObject disk, float angle, float power, bool isPhysis);
}
```

该接口是场景控制器和动作管理器的桥梁，负责与场景控制器进行交流，从而隐藏了具体动作管理器的实现细节。

##### 适配器`ActionManagerAdapter`

`ActionManagerAdapter`类实现上述接口，实现对于不同运动管理器的管理和调用。

```c#
public class ActionManagerAdapter : MonoBehaviour, IActionManager
{
    public CCActionManager CCAction;
    public PhysisActionManager PhysisAction;

    public void playDisk(GameObject disk, float angle, float power, bool isPhysis)
    {
        if (!isPhysis)
        {
            CCAction.playDisk(disk, angle, power);
        }
        else
        {
            PhysisAction.playDisk(disk, angle, power);
        }
    }

    void Start()
    {
        CCAction = gameObject.AddComponent<CCActionManager>() as CCActionManager;
        PhysisAction = gameObject.AddComponent<PhysisActionManager>() as PhysisActionManager;
    }
}
```

##### `PhysisAction`和`PhysisActionManager`

物理运动的具体实现。实现方法可以参考之前的运动学管理器`CCActionManager`。不同的地方在于，物理运动是通过对物体添加重力和一个初速度后（使用`Rigidbody`刚体组件实现），物体就会自动按照物理学规律运动，无需进行额外的管理。

需要注意的是，这里我们在使用物理运动管理器时，设置`gameObject.GetComponent<Rigidbody>().useGravity = true;`，并添加了初速度。那么，如果需要做到随意切换，那么在运动学管理器中，我们需要做相反的操作，即取消重力和`Rigidbody`组件的初速度。

```c#
public class PhysisFlyAction : SSAction
{
    private Vector3 startVector;
    public float power;

    public static PhysisFlyAction GetSSAction(Vector3 direction, float angle, float power)
    {
        PhysisFlyAction action = CreateInstance<PhysisFlyAction>();
        if (direction.x == -1)
        {
            action.startVector = Quaternion.Euler(new Vector3(0, 1, -angle)) * Vector3.left * power;
        }
        else
        {
            action.startVector = Quaternion.Euler(new Vector3(0, 1, angle)) * Vector3.right * power;
        }
        action.power = power;
        return action;
    }

    public override void Start()
    {
        gameObject.GetComponent<Rigidbody>().velocity = power / 15 * startVector;
        gameObject.GetComponent<Rigidbody>().useGravity = true;
    }

    public override void Update()
    {
        if (this.transform.position.y < -10)
        {
            this.destory = true;
            this.callback.SSActionEvent(this);
        }
    }
}


public class PhysisActionManager : SSActionManager, ISSActionCallback
{
    public PhysisFlyAction fly;

    protected new void Start(){ }

    //飞碟飞行
    public void playDisk(GameObject disk, float angle, float power)
    {
        fly = PhysisFlyAction.GetSSAction(disk.GetComponent<DiskData>().direction, angle, power);
        this.RunAction(disk, fly, this);
    }

    #region ISSActionCallback implementation
    public void SSActionEvent(SSAction source,
        SSActionEventType events = SSActionEventType.Compeleted,
        int intParam = 0,
        string strParam = null,
        Object objectParam = null)
    {
        //回调函数,动作执行完后调用
    }
    #endregion
}
```

##### 修改场景控制器

对于场景控制器，做以下一些修改：

```c#
//添加一个公共的BOOL属性，用于在游戏进行过程中随时切换运动管理器
public bool isPhysis = false;

//将场景的运动管理器由原先的 CCActionManager 改为 IActionManager
actionManager = gameObject.AddComponent<ActionManagerAdapter>() as IActionManager;

//为原先的函数调用增加一个参数
actionManager.playDisk(disk, angle, power, isPhysis);
```

##### 修改飞碟Prefab

在完成了上述代码后，由于物理运动的实现通过`Rigidbody`刚体组件，我们需要给飞碟预制添加`Rigidbody`组件，并不勾选`Use Gravity`，交给我们的代码控制。

完成上述改动后，就基本上实现了使用 Adapter 模式实现两种运动的要求。

#### 实现结果

启动游戏后，我们可以通过右边面板中场景管理器的`isPhysis`属性在游戏中随时切换不同的运动管理器。

{% asset_img controller.png %}

游戏界面如下：

{% asset_img game.png %}

