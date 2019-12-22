---
title: 3D-Game-Programming-Design-UI系统
date: 2019-11-17 15:07:11
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程的第九次作业：
>
> + 血条（Health Bar）的预制设计
>
> [课程主页]( https://pmlpml.github.io/unity3d-learning/09-ui.html )

<!-- more -->

### 要求

- 分别使用 IMGUI 和 UGUI 实现
- 使用 UGUI，血条是游戏对象的一个子元素，任何时候需要面对主摄像机
- 分析两种实现的优缺点
- 给出预制的使用方法

[项目地址]( https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/UI_System )

[演示视频]( https://www.bilibili.com/video/av76047817/ )

### 实现过程

#### IMGUI

IMGUI主要通过`HorizontalScrollbar`实现，将其宽度作为血条的显示值。

具体代码如下，主要就是创建了一个矩形的`HorizontalScrollbar`作为血条，以及两个按钮，使得我们对于血量可以进行直观的增减，按钮触发的事件直接改变血量。

其中，参考以前师兄们的博客，使用了线性插值函数`Lerp()`，该函数有很多用法，这里的`Mathf.Lerp(from : float, to : float, t : float) ` 函数基于浮点数`t`返回`a`到`b`之间的插值，`t`限制在0～1之间。当`t` = 0返回from，当`t` = 1 返回to。当`t` = 0.5 返回from和to的平均值。 因此，通过这个函数可以实现血量的平滑改变。

```c#
public class IMGUI : MonoBehaviour
{
    public float blood;
    private float resultBlood;

    private Rect bloodBar;      // 血条
    private Rect addButton;     // 加血按钮
    private Rect reduceButton;  // 扣血按钮

    // Start is called before the first frame update
    void Start()
    {
        // 初始值
        blood = 50;
        resultBlood = 50;

        // 固定位置
        bloodBar = new Rect(Screen.width / 2 -100, Screen.height / 4 - 10, 200, 20);
        reduceButton = new Rect(Screen.width / 2 - 160, Screen.height / 4 - 10, 40, 20);
        addButton = new Rect(Screen.width / 2 + 120, Screen.height / 4 - 10, 40, 20);
    }

    // Update is called once per frame
    void Update()
    {
    
    }

    private void OnGUI()
    {
        if(GUI.Button(addButton, "+"))
        {
            resultBlood = resultBlood + 10.0f > 100.0f ? 100.0f : resultBlood + 10.0f;
        }
        if(GUI.Button(reduceButton, "-"))
        {
            resultBlood = resultBlood - 10.0f < 0.1f ? 0.0f : resultBlood - 10.0f;
        }

        GUI.color = new Color(255f / 255f, 0f / 255f, 0f / 255f, 1f);

        //插值计算HP值
        blood = Mathf.Lerp(blood, resultBlood, 0.05f);

        GUI.HorizontalScrollbar(bloodBar, 0.0f, blood, 0.0f, 100.0f);
    }
}
```

#### UGUI

通过UGUI实现血条的方式参考老师的课程页面指导，步骤如下：

- 菜单 Assets -> Import Package -> Characters 导入人物资源
- 在层次视图，Context 菜单 -> 3D Object -> Plane 添加 Plane 对象
- 资源视图展开 Standard Assets :: Charactors :: ThirdPersonCharater :: Prefab
- 将 ThirdPersonController 预制拖放放入场景，改名为 Ethan
- 检查以下属性
  - Plane 的 Transform 的 Position = (0,0,0)
  - Ethan 的 Transform 的 Position = (0,0,0)
  - Main Camera 的 Transform 的 Position = (0,1,-10)
- 运行检查效果
- 选择 Ethan 用上下文菜单 -> UI -> Canvas, 添加画布子对象
- 选择 Ethan 的 Canvas，用上下文菜单 -> UI -> Slider 添加滑条作为血条子对象
- 选择 Ethan 的 Canvas，在 Inspector 视图
  - 设置 Canvas 组件 Render Mode 为 World Space
  - 设置 Rect Transform 组件 (PosX，PosY，Width， Height) 为 (0,2,160,20)
  - 设置 Rect Transform 组件 Scale （x,y） 为 (0.01,0.01)

- 展开 Slider
  - 选择 Handle Slider Area，禁灰（disable）该元素
  - 选择 Background，禁灰（disable）该元素
  - 选择 Fill Area 的 Fill，修改 Image 组件的 Color 为 红色
- 选择 Slider 的 Slider 组件
  - 设置 MaxValue 为 100
  - 设置 Value 为 75

经过上述步骤运行检查效果，发现血条会随着人物旋转。所以需要给 Canvas 添加以下脚本使其一直朝着相机方向。

```c#
public class LookAtCamera : MonoBehaviour {
	void Update () {
		this.transform.LookAt (Camera.main.transform.position);
	}
}
```

到此，基础的血条就完成了，但为了显示其血量的变化，我在游戏场景增加了一个障碍物，并使得人物在撞到障碍物后血量会下降。

障碍物通过Cube实现，添加如下脚本：

```c#
private void OnCollisionEnter(Collision collision)
{
    if(collision.gameObject.name == "Ethan")
    {
        collision.gameObject.GetComponentInChildren<Controller>().collision();
    }
}
```

同时，给人物Ethan的子对象Canvas添加如下控制器代码，扣血的原理和IMGUI的原理相同，当Cube检测到碰撞后使得flag变量变为true，通过控制Slider的值实现血量的改变。

```c#
public class Controller : MonoBehaviour
{
    // UGUI的Slider对象
    public Slider mainSlider;

    // 血量
    private float currentBlood;
    private float resultBlood;

    // 碰撞检测
    private bool flag;

    // Start is called before the first frame update
    void Start()
    {
        // 初始血量
        mainSlider.value = 80;
        currentBlood = mainSlider.value;
        resultBlood = currentBlood;
    }

    // Update is called once per frame
    void Update()
    {
        // 如果发生碰撞，扣血
        if (flag)
        {
            resultBlood = mainSlider.value - 20.0f < 0.1f ? 0.0f : mainSlider.value - 20.0f;
            flag = false;
        }
        // 平滑减少血量
        currentBlood = Mathf.Lerp(currentBlood, resultBlood, 0.1f);
        mainSlider.value = currentBlood;
    }

    public void collision()
    {
        flag = true;
    }
}
```

### 实现效果

#### IMGUI

{% asset_img IMGUI.png %}

#### UGUI

{% asset_img UGUI.png %}

### 优缺点分析

#### IMGUI

个人认为IMGUI比较符合传统的UI写法，即通过代码的方式生成一个UI对象，并且定义与该对象进行交互时会发生的事件，这和我这个学期正在学习的IOS开发比较类似，这样的编程使得一切都在程序员的掌控之中，对于程序员来说应该是十分友好的。但缺点就是一切都是由代码生成，那么当需要的UI界面十分复杂时，就需要大量的代码支持，此时效率就会比较低下，调试起来也很困难，同时对于非程序员来说使用就比较困难，例如设计师们。

#### UGUI

UGUI的出现就是为了解决IMGUI对于设计师们不够友好的困境，无需编码，而只需要创建UI组件并设置相应属性即可。这种所见即所得的设计方式让设计师们也能够参与程序的开发，脱离程序员的帮助。而其缺点，目前我使用的还比较简单，暂时还不太清楚有何缺点，可能对于作为程序员的我来说，创建的方式相对比较复杂，有很多属性的设置需要查看相应的文档才能清楚，不像程序中的方法调用那样直观。

### 预制的使用方法

本项目中有`IMGUI`和`UGUI`两个预制，分别对应了各自的血条实现。使用方法如下：

+ 当需要使用由`IMGUI`实现的血条时，只需要直接将该预制拖入场景即可。

+ 当需要使用由`UGUI`实现的血条时，首先按照上文所说的搭建好基本场景，例如导入人物资源、设置 Plane 对象。然后将`UGUI`预制拖入到 Ethan对象下，成为其子对象，然后将`UGUI`预制对象下的 子对象Slider拖入对象的Controller.cs组件中的MainSlider属性即可。如需实现碰撞效果，需要再创建一个Cube对象，并为其添加脚本中的Collider.cs。