---
title: 3D-Game-Programming-Design | AR小游戏
date: 2019-12-16 16:58:28
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程的第十一次作业：
>
> + AR图片识别与虚拟按键小游戏
>
> [课程主页](https://pmlpml.github.io/unity3d-learning/12-AR-and-MR)

<!-- more -->

[项目地址](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/AR_Game)

[演示视频](https://www.bilibili.com/video/av79896695/) 

## 游戏简介

这是一款极简的AR跑酷小游戏，通过虚拟按键进行控制，实现了类似Chrome浏览器在断网下可以玩的小恐龙游戏。

### Vuforia的使用

新版本的Vuforia使用和课程指导上已经有比较大的不同了，在这里简单说一下我在本次项目中关于Vuforia的使用。

#### 官网注册

这一步和老师上课说的还是一致的，我们需要现在Vuforia官网上申请license，并创建目标数据库。具体步骤就不再赘述了。

#### 开启Vuforia支持

首先需要启动Vuforia支持，勾选`Edit`->`Project Settings`->`Player`->`XR Settings`中的`Vuforia Augmented Reality Supported`选项。

{% asset_img image-20191216204400533.png %}

#### 添加AR Camera

删除原来的相机，加入`AR Camera`：

{% asset_img image-20191216204400533.png style="zoom:50%;" %}

在`AR Camera`的`Inspector`面板中，点击`Open Vuforia Engine configuration`选项，进入Vuforia设置面板，添加之前在官网注册好的License以及Databases，在该面板中还可以设置使用笔记本的前置还是后置摄像头。

{% asset_img image-20191216204518813.png %}

#### 添加ImageTarget

加入`ImageTarget`对象，并选择好相应的Database。

{% asset_img image-20191216204546229.png %}

此时，启动游戏便可以识别图片了，但是我们要想出现AR效果，即在图片上出现游戏人物，就需要在`ImageTarget`下添加一个子对象，并设置好相应的大小关系，如下图所示：

{% asset_img image-20191216205806338.png %}

启动游戏后，已经可以成功识别，并且可以显示出我们在上一步添加的游戏对象。效果如下：

{% asset_img image-20191216210000806.png %}

## 游戏设计

本游戏中要实现的主要有两个方面：小飞龙的跳跃以及鸟的飞行，其中，鸟的飞行通过脚本控制鸟的预制体移动实现，而小飞龙的跳跃则通过虚拟按键实现。

### 鸟的飞行

鸟的飞行脚本实现的比较简单，首先得到鸟的预制体对象，然后在每次`Update`函数中控制其位置按照既定的路线移动即可，一旦超出某个范围就让其回到原点，产生不断有鸟飞过的假象。需要注意的是，这里的位置都使用了相对位置，若使用绝对位置，在进行游戏时，如果相机移动了，鸟的位置也会移动。

```c#
using UnityEngine;
using System.Collections;

public class birdGenerator : MonoBehaviour
{
    private GameObject bird;
    // Use this for initialization
    void Start()
    {
        bird = Object.Instantiate(Resources.Load<GameObject>("bird"),
                    Vector3.zero, Quaternion.identity, null);
        bird.transform.parent = this.transform;
        bird.transform.localPosition = new Vector3(-0.2f, 0.1f, 0.5f);
        bird.transform.localScale *= 0.07f;
        bird.transform.Rotate(0, 180, 0, Space.Self); 
    }

    // Update is called once per frame
    void Update()
    {
        bird.transform.localPosition -= new Vector3(0, 0, 0.015f);
        if (bird.transform.localPosition.z < -0.4)
        {
            bird.transform.localPosition = new Vector3(-0.2f, 0, 0.5f);
        }   
    }
}
```

### 虚拟按键

虚拟按键的使用方式和课件中有所不同，不再是一个预制体了，而是通过下图中`Add Virtual Button`的方式进行添加。在添加了虚拟按键之后，在游戏画面中是看不到的，为了使虚拟按钮可见，可以在按钮下添加3D物体以显示其位置。

{% asset_img image-20191216211324458.png %}

虚拟按键的使用方式比较简单，主要是让类实现`IVirtualButtonEventHandler`中的`OnButtonPressed`和`OnButtonReleased`方法，并在虚拟按键中进行注册。在以下脚本中实现了，如果虚拟按键被按下，`dragon`游戏对象就进行原地跳跃（先上升后下降）。同时，通过`pressed`变量防止作弊现象（即一直按着虚拟按键，导致小火龙不下落的情况），只有当离开虚拟按键后再次按下，才会再次跳跃。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Vuforia;

public class VBController : MonoBehaviour, IVirtualButtonEventHandler
{
    public GameObject dragon;
    public Vector3 target = new Vector3(-0.2f, 0, -0.2f);
    public float speed = 0.1f;
    public bool pressed = false;

    void Start()
    {
        //在所有子物体类中找到所有VirtualButtonBehaviour组件
        VirtualButtonBehaviour[] vbs = GetComponentsInChildren<VirtualButtonBehaviour>();
        //在虚拟按钮中注册TrackableBehaviour事件
        for (int i = 0; i < vbs.Length; ++i)
        {
            vbs[i].RegisterEventHandler(this);
        }

        dragon = transform.Find("dragon").gameObject;
    }
    
    void Update()
    {
        dragon.transform.localPosition = Vector3.MoveTowards(dragon.transform.localPosition, target, speed * Time.deltaTime);
        if (dragon.transform.localPosition == target)
        {
            target = new Vector3(-0.2f, 0, -0.2f);
        }
    }

    public void OnButtonPressed(VirtualButtonBehaviour vb)
    {
        Debug.Log(vb.VirtualButtonName + " btn pressed");
        if(pressed == false)
        {
            pressed = true;
            target = new Vector3(-0.2f, 0.4f, -0.2f);
        }
        
    }

    public void OnButtonReleased(VirtualButtonBehaviour vb)
    {
        Debug.Log(vb.VirtualButtonName + " btn released");
        pressed = false;
    }
}
```

将以上两个脚本均挂载到`ImageTarget`上即可。



## 游戏画面

游戏的画面如下，下图中的白色按钮即为可视化的虚拟按键，点击它，正义的小火龙就会起跳躲避邪恶的乌鸦！实际操作中，由于虚拟按键的原因，可能会出现控制不灵敏或者误触的现象，但总体上实现了预期效果。

{% asset_img image-20191220090139193.png %}