---
title: 3D Game Programming & Design - 空间与运动
date: 2019-09-16 16:48:24
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程的第三次作业：
>
> + 使用Unity模拟出了太阳系中的行星运动
> + 采用MVC设计模式编写了一个小游戏：牧师与恶魔
>
> [课程主页](https://pmlpml.github.io/unity3d-learning/03-space-and-motion)

<!-- more -->

##### 简答并用程序验证

> 游戏对象运动的本质是什么？

游戏对象运动的本质就是使用矩阵变换（平移、旋转、缩放）改变游戏对象的空间属性。

> 请用三种方法以上方法，实现物体的抛物线运动。

+ 方法一：运动分解。将抛物线运动分解为横向和纵向的运动叠加，直接修改物体的位置

  ```c#
  public class method1 : MonoBehaviour
  {
      public float speed = 200F;   //初速度
      public float angle = 60;    //起始方向与地面夹角    
  
      private float G = 9.8F;     //重力加速度
      private float xSpeed;       //x方向速度分量
      private float ySpeed;       //y方向速度分量
  
      // Start is called before the first frame update
      void Start()
      {
          xSpeed = speed * Mathf.Cos(angle / 180 * Mathf.PI);
          ySpeed = speed * Mathf.Sin(angle / 180 * Mathf.PI);
      }
  
      // Update is called once per frame
      void Update()
      {
          this.transform.position += Vector3.right * Time.deltaTime * xSpeed;
          this.transform.position += Vector3.up * Time.deltaTime * ySpeed;
          ySpeed -= G * Time.deltaTime;
      }
  }
  ```

+ 方法二：使用`Transform` 的方法 `Translate`

  ```c#
  public class method2 : MonoBehaviour
  {
      public float speed = 200F;   //初速度
      public float angle = 60;    //起始方向与地面夹角    
  
      private float G = 9.8F;     //重力加速度
      private float xSpeed;       //x方向速度分量
      private float ySpeed;       //y方向速度分量
  
      // Start is called before the first frame update
      void Start()
      {
          xSpeed = speed * Mathf.Cos(angle / 180 * Mathf.PI);
          ySpeed = speed * Mathf.Sin(angle / 180 * Mathf.PI);
      }
  
      // Update is called once per frame
      void Update()
      {
          Vector3 direction = (Vector3.right * xSpeed + Vector3.up * ySpeed) * Time.deltaTime;
          this.transform.Translate(direction);
          ySpeed -= G * Time.deltaTime;
      }
  }
  ```

+ 方法三：使用Vector3 的方法`MoveTowards`

  ```c#
  public class method3 : MonoBehaviour
  {
      public float speed = 200F;   //初速度
      public float angle = 60;    //起始方向与地面夹角    
  
      private float G = 9.8F;     //重力加速度
      private float xSpeed;       //x方向速度分量
      private float ySpeed;       //y方向速度分量
  
      // Start is called before the first frame update
      void Start()
      {
          xSpeed = speed * Mathf.Cos(angle / 180 * Mathf.PI);
          ySpeed = speed * Mathf.Sin(angle / 180 * Mathf.PI);
      }
  
      // Update is called once per frame
      void Update()
      {
          Vector3 destation= this.transform.position + (Vector3.right * xSpeed + Vector3.up * ySpeed) * Time.deltaTime;
          this.transform.position = Vector3.MoveTowards(this.transform.position, destation, 1);
          ySpeed -= G * Time.deltaTime;
      }
  }
  ```

> 写一个程序，实现一个完整的太阳系， 其他星球围绕太阳的转速必须不一样，且不在一个法平面上。

[项目地址](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Simulated_Solar_System)

[完整演示视频](https://www.bilibili.com/video/av68313777)

以下简述实现过程：

+ 首先创建10个球对象：一个太阳，八个行星，以及一个月亮。八个行星为太阳的子对象，月亮为地球的子对象。

  {% asset_img cenci.png %}

+ 按照真实的太阳系设置各个星球的大小，位置，并用网上收集到的图片更改球的外观。

  {% asset_img init.png %}

+ 编写两个脚本，实现公转和自转:

  公转：
  
  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class RotateAroundOther : MonoBehaviour
  {
      public Transform sun;
      public int speed = 50;
      float ry,rz;
      // Start is called before the first frame update
      void Start()
      {
          ry = Random.Range(30, 60);
          rz = Random.Range(-20, 20);
      }
      // Update is called once per frame
      void Update()
      {
          Vector3 axis = new Vector3(0, ry, rz);
          this.transform.RotateAround(sun.position, axis, speed * Time.deltaTime);
      }
  }
  ```
  
  + 自转：
  
  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class RotateAroundSelf : MonoBehaviour
  {
      // Start is called before the first frame update
     void Start()
      {
      }
    
    // Update is called once per frame
    void Update()
    {
        this.transform.RotateAround(this.transform.position, Vector3.up, 1 * Time.deltaTime);
    }
  }
  ```

将上述公转和自转代码赋给各个星球，设置好公转的转速和中心即可。至于为了使公转的法平面不同而设置的`ry = Random.Range(30, 60); rz = Random.Range(-20, 20);`，具体数值可以自己调整。到此，太阳系的基本功能就以实现了

+ 此外，我还增加了背景；实现了暂停和开始功能；为行星的运动添加了轨道，使得转动更加直观。

+ 最终效果图：

  {% asset_img sun.png %}

##### 编程实践

###### 要求

阅读以下游戏脚本

> Priests and Devils
>
> Priests and Devils is a puzzle game in which you will help the Priests and Devils to cross the river within the time limit. There are 3 priests and 3 devils at one side of the river. They all want to get to the other side of this river, but there is only one boat and this boat can only carry two persons each time. And there must be one person steering the boat from one side to the other side. In the flash game, you can click on them to move them and click the go button to move the boat to the other direction. If the priests are out numbered by the devils on either side of the river, they get killed and the game is over. You can try it in many > ways. Keep all priests alive! Good luck!

[示例地址]( http://www.flash-game.net/game/2535/priests-and-devils.html )

> 程序需要满足的要求：
>
> - 列出游戏中提及的事物（Objects）
> - 用表格列出玩家动作表（规则表），注意，动作越少越好
> - 请将游戏中对象做成预制
> - 在 GenGameObjects 中创建长方形、正方形、球及其色彩代表游戏中的对象。
> - 使用C#集合类型有效组织对象
> - 整个游戏仅主摄像机和一个 Empty 对象，**其他对象必须代码动态生成！！！** 。 整个游戏不许出现 Find 游戏对象， SendMessage 这类突破程序结构的通讯耦合语句。 **违背本条准则，不给分**
> - 请使用课件架构图编程，**不接受非 MVC 结构程序**
> - 注意细节，例如：船未靠岸，牧师与魔鬼上下船运动中，均不能接受用户事件！

游戏中提及的事物

+ 魔鬼
+ 牧师
+ 船
+ 河流
+ 河两岸

玩家动作表（规则表）

|           条件           |              动作               |         结果         |
| :----------------------: | :-----------------------------: | :------------------: |
|      船停靠在岸边时      | 点击与船同侧的人物（牧师/魔鬼） | 人物跳进船或跳回岸边 |
| 船停靠在岸边且船上有人时 |           点击GO按钮            |      船驶向对岸      |
|     游戏胜利或失败后     |         点击Restart按钮         |     重新开始游戏     |

###### 游戏分析与前期准备

+ 按照题中要求，游戏的设计采用MVC（Model - View - Controller）结构模式，并遵循课程中老师给出的设计框架，如下图所示：

  {% asset_img MVC.png %}

+ 由于要求游戏对象要动态生成，所以要预先把各种游戏对象制作成预制，放在`Assets/Resources/Perfabs`文件夹中，当游戏启动时通过场景控制器执行`LoadResources`函数实例化。
+ 从Assert Store中下载(白嫖)游戏模型，还是有很多免费但精美的模型的，有些还自带了动作。
+ 在模拟好整个场景，预先排练并记下每个游戏对象的合适位置后，我们就可以用代码来动态生成他们了，并为它们赋予各自的逻辑。
+ 因为牧师与恶魔的行为基本一致，所以我们可以归为一类考虑，并且用数组存储其对象，便于组织和管理。

###### 代码实例

由于源代码过长，就不全部贴出来了，这里只展示一些关键函数。详细的可以见我的Github(在下面给出了)。

+ 首先是导演类，比较简单，就是单例…不说了。

然后我先完成了Model层的设计，因为Model较为底层，个人认为类似于建筑的地基。在Model层中，需要完成对每一类游戏对象属性、行为的定义。对于此游戏，我一共定义了role、boat、land、river四个模型，各自封装到一个类中。

+ role类

```c#
public class RoleModel
{
    GameObject role;
    bool good;
    bool onBoat;
    int side;
    Click click;      
    Move move;

    public RoleModel(string roleType)
    {
        onBoat = false;
        side = 1;
        if (roleType == "priest")
        {
            role = Object.Instantiate(
                               Resources.Load<GameObject>("Prefabs/Priest"),
                               Vector3.zero, Quaternion.Euler(0, 270, 0));
            good = true;
        }
        else if (roleType == "devil")
        {
            role = Object.Instantiate(
                                Resources.Load<GameObject>("Prefabs/Devil"),
                                Vector3.zero, Quaternion.Euler(0, 270, 0));
            good = false;
        }
        move = role.AddComponent(typeof(Move)) as Move;
        click = role.AddComponent(typeof(Click)) as Click;
        click.SetRole(this);
    }

    public void Reset()
    {
        side = 1;
        role.transform.parent = null;
        onBoat = false;
    }

    //下面Attack、Idle、Lose三个函数为恶魔专属，用于设置恶魔动画，似乎有些不符合role的设定，用继承可能好一点
    public void Attack()
    {
        Animator anim = role.GetComponent<Animator>();
        anim.SetBool("attack", true);
    }
    public void Idle()
    {
        Animator anim = role.GetComponent<Animator>();
        anim.SetBool("attack", false);
        anim.SetBool("lose", false);
    }
    public void Lose()
    {
        Animator anim = role.GetComponent<Animator>();
        anim.SetBool("lose", true);
    }
    
    public bool IsGood() { return good; }
    public bool IsMoving()
    {
        return move.IsMoving();
    }
    public string GetName() { return role.name; }
    public void SetName(string name) { role.name = name; }
    public int GetSide() { return side; }
    public void SetSide(int s = 0)
    {
        if(s != 0)
        {
            side = s;
        }
        else 
            side = 0 - side;
    }
    public void ChangeDirction()
    {
        if (side == 1)
        {
            role.transform.rotation = Quaternion.Euler(0, 270, 0);
        }
        else
        {
            role.transform.rotation = Quaternion.Euler(0, 90, 0);
        }
    }
    public bool IsOnBoat() { return onBoat; }
    public void SetPosition(Vector3 pos) { role.transform.position = pos; }
    public Vector3 getPosition() { return role.transform.position; }
    public void Move(Vector3 position)
    {
        move.setPosition(position);
    }

    public void GoLand()
    {
        role.transform.parent = null;
        onBoat = false;
    }

    public void GoBoat(BoatModel boat)
    {
        role.transform.parent = boat.GetBoat().transform;
        onBoat = true;
    }
}
```

+ boat类：

```c#
public class BoatModel
{
    GameObject boat;
    //bool moving; //船是否在运动
    int side; //1表示在右边，-1表示在左边
    Vector3 rightPosition = new Vector3(3, -1, 0);
    Vector3 leftPosition = new Vector3(-3, -1, 0);
    Vector3[] rightPositions = new Vector3[] { new Vector3(2.5F, -0.8F, 0), new Vector3(3.5F, -0.8F, 0) };
    Vector3[] leftPositions = new Vector3[] { new Vector3(-2.5F, -0.8F, 0), new Vector3(-3.5F, -0.8F, 0) };
    RoleModel[] passengers = new RoleModel[2];
    Move move;
    public BoatModel()
    {
        //初始船在右边
        side = 1;
        boat = Object.Instantiate(
                                Resources.Load<GameObject>("Prefabs/Boat"),
                                new Vector3(3, -1, 0), Quaternion.Euler(0, 270, 0));
        boat.name = "boat";
        move = boat.AddComponent(typeof(Move)) as Move;
        for (int i = 0; i < 2; i++)
            passengers[i] = null;
    }

    public int GetSide()
    {
        return side;
    }
    public void ChangeDirction()
    {
        if(side == 1)
        {
            boat.transform.rotation = Quaternion.Euler(0, 270, 0);
        }
        else
        {
            boat.transform.rotation = Quaternion.Euler(0, 90, 0);
        }
    }
    public bool IsEmpty()
    {
        for(int i = 0; i < 2; i++)
            if (passengers[i] != null)
                return false;
        return true;
    }

    public bool IsFull()
    {
        for (int i = 0; i < 2; i++)
            if (passengers[i] == null)
                return false;
        return true;
    }
    public bool IsMoving()
    {
        return move.IsMoving();
    }

    public void Reset()
    {
        side = 1;
        boat.transform.position = new Vector3(3, -1, 0);
        move.setPosition(leftPosition);
        move.SetMoveSate(0);
        passengers = new RoleModel[2];
    }

    public void Move()
    {
        if(side == 1)
        {
            move.setPosition(leftPosition);
            side = -1;
        }
        else
        {
            move.setPosition(rightPosition);
            side = 1;
        }
        //同时改变role的side
        for (int i = 0; i < 2; i++)
        {
            if (passengers[i] != null)
            {
                passengers[i].SetSide();
            }
        }
    }

    public int GetEmptyIndex()
    {
        for (int i = 0; i < passengers.Length; i++)
        {
            if (passengers[i] == null)
            {
                return i;
            }
        }
        return -1;
    }

    public Vector3 getEmptyPosition()
    {
        Vector3 pos;
        int emptyIndex = GetEmptyIndex();
        if (side == 1)
        {
            pos = rightPositions[emptyIndex];
        }
        else
        {
            pos = leftPositions[emptyIndex];
        }
        return pos;
    }

    public GameObject GetBoat()
    {
        return boat;
    }

    public void AddPassenger(RoleModel passenger)
    {
        if (!IsFull()) 
            passengers[GetEmptyIndex()] = passenger;
    }

    public RoleModel DeletePassenger(RoleModel passenger)
    {
        for (int i = 0; i < 2; i++)
            if (passengers[i] != null && passengers[i].GetName() == passenger.GetName())
            {
                RoleModel role = passengers[i];
                passengers[i] = null;
                return role;
            }
        return null;
    }

    public RoleModel[] GetPassengers()
    {
        return passengers;
    }
}
```

河岸land类以及河river类都没什么特别的，因为它们都没有什么行为，只需要简单的初始化即可，这里就不展示了。

但光有对象不行，我们还需要给它们附加行为，才可以让他们动起来，同时还需要使role响应玩家鼠标的点击。就有了下面的Move和Click类。观察之前的role类，我们可以发现在初始化boat和role时，我们就实例化了这两个类，并作为组件(Component)附加到了游戏对象上，类似之前直接创建的脚本。

+ Move类

Move类实现游戏对象的平滑移动，主要依靠的是内置的`MoveToward`函数，在每次``Update`时都会执行，使得对象逐渐朝目的地移动。

```c#
public class Move : MonoBehaviour
{
    float speed = 4;  //移动速度
    Vector3 endPosition, middlePosition;
    int moveState = 0;  //0表示不移动, 1表示水平移动，2表示竖直移动
    public void SetMoveSate(int state)
    {
        moveState = state;
    }
    public bool IsMoving()
    {
        return moveState != 0;
    }
    public void setPosition(Vector3 position)
    {
        endPosition = position;
        if (position.y == transform.position.y)         //y值不变：船的移动
        {
            SetMoveSate(2);
        }
        else
        {
            if (position.y < transform.position.y)      //y值减小：角色从陆地到船
            {
                middlePosition = new Vector3(position.x, transform.position.y, position.z);

            }
            else                                          //y值增大：角色从船到陆地
            {
                middlePosition = new Vector3(transform.position.x, position.y, position.z);
            }
            SetMoveSate(1);
        }
    }
    void Update()
    {
        if (moveState == 1)
        {
            transform.position = Vector3.MoveTowards(transform.position, middlePosition, speed * Time.deltaTime);
            if (transform.position == middlePosition)
                SetMoveSate(2);
        }
        else if (moveState == 2)
        {
            transform.position = Vector3.MoveTowards(transform.position, endPosition, speed * Time.deltaTime);
            if (transform.position == endPosition)
                SetMoveSate(0);
        }
    }
}
```

+ Click类

Click类绑定一个相关的role对象，通过`OnMouseDown`函数实现响应鼠标点击，执行相关动作。

值得注意的是，要想OnMouseDown函数工作，还需要给游戏对象添加碰撞体(Collider)才行。

```c#
public class Click : MonoBehaviour
{
    IUserAction action;
    RoleModel role;
    public void SetRole(RoleModel r)
    {
        role = r;
    }
    void Start()
    {
        action = SSDirector.GetInstance().CurrentSceneController as IUserAction;
    }
    void OnMouseDown()
    {
        if (role == null)
        {
            return;
        }
        action.MoveRole(role);
    }
}
```

现在Model层就基本完成了。我们再来考虑View层，即游戏与玩家的交互，我们需要为玩家提供什么交互功能。

+ IUserAction

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public interface IUserAction
{
    void MoveBoat();
    void MoveRole(RoleModel role);
    int Check();
    void Restart();
}
```

+ UserGUI:

由于我们的FirstSceneController也实现了IUserACtion接口，相当于控制器帮助玩家实现了指定的工作。所以我们在UserGUI中，只需要得到这个控制器，当玩家进行某个操作时，调用控制器的接口函数即可。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class UserGUI : MonoBehaviour
{
    private IUserAction action;
    int sign = 0; //0：游戏进行中， 1：游戏胜利  -1：游戏失败

    // Start is called before the first frame update
    void Start()
    {
        action = SSDirector.GetInstance().CurrentSceneController as IUserAction;
    }
    
    void OnGUI()
    {
        GUIStyle text_style;
        GUIStyle button_style;
        text_style = new GUIStyle()
        {
            fontSize = 30
        };
        button_style = new GUIStyle("button")
        {
            fontSize = 15
        };
        sign = action.Check();
        if (sign == 0) {
            if (GUI.Button(new Rect(Screen.width / 2 - 30, 80, 60, 60), "Go!"))
            {
                action.MoveBoat();
            }
        }
        else if(sign == -1)
        {
            GUI.Label(new Rect(Screen.width / 2 - 75, 100, 120, 50), "You Failed!", text_style);
            if (GUI.Button(new Rect(Screen.width / 2 - 50, 150, 100, 50), "Try Agian", button_style))
            {
                action.Restart();
                sign = 0;
            }
        }
        else if(sign == 1)
        {
            GUI.Label(new Rect(Screen.width / 2 - 60, 100, 120, 50), "You Win!", text_style);
            if (GUI.Button(new Rect(Screen.width / 2 - 50, 150, 100, 50), "Restart", button_style))
            {
                action.Restart();
                sign = 0;
            }
        }
    }
}

```

最后我们完成Controller层，将之前的两层衔接起来。这里实现的是FirstController类，主要完成了加载资源，移动人物和船，判断游戏是否结束等功能。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FirstController : MonoBehaviour, ISceneController, IUserAction
{
    public RoleModel[] Priests = new RoleModel[3];
    public RoleModel[] Devils = new RoleModel[3];
    public RiverModel River;
    public LandModel RightLand;
    public LandModel LeftLand;
    public BoatModel Boat;
    Vector3[] PrisetsOriginPositions = new Vector3[] { new Vector3(5F, 0, 0), new Vector3(5.5F, 0, 0), new Vector3(6F, 0, 0) };
    Vector3[] DevilsOriginPositions = new Vector3[] { new Vector3(7F, 0, 0), new Vector3(7.5F, 0, 0), new Vector3(8F, 0, 0) };

    bool gaming = true; //用于判断游戏是否正在进行，由于目前还不存在起始界面，所以一开始游戏就开始了

    void Awake()
    {
        SSDirector director = SSDirector.GetInstance();
        //director.setFPS(60);
        director.CurrentSceneController = this;
        director.CurrentSceneController.LoadResource();
    }

    public void LoadResource()
    {
        for(int i = 0; i < 3; i++)
        {
            RoleModel priest = new RoleModel("priest");
            priest.SetName("priest" + i);
            priest.SetPosition(new Vector3(5 + i * 0.5F, 0, 0));
            Priests[i] = priest;

            RoleModel devil = new RoleModel("devil");
            devil.SetName("devil" + i);
            devil.SetPosition(new Vector3(7 + i * 0.5F, -0.1F, 0));
            devil.Idle();
            Devils[i] = devil;
        }

        RightLand = new LandModel("right");
        LeftLand = new LandModel("left");
        River = new RiverModel();
        Boat = new BoatModel();
    }

    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    public void Restart()
    {
        for (int i = 0; i < 3; i++)
        {
            Priests[i].Reset();
            Priests[i].SetPosition(PrisetsOriginPositions[i]);
            Devils[i].Reset();
            Devils[i].SetPosition(DevilsOriginPositions[i]);
            Devils[i].Idle();
        }
        Boat.Reset();
        gaming = true;
    }
    public int Check()
    {
        //等船停下来再check
        if (Boat.IsMoving())
            return 0;
        Boat.ChangeDirction();

        //计算两边的牧师和恶魔数量
        int rightPriestNum = 0, leftPriestNum = 0, rightDevilNum = 0, leftDevilNum = 0;
        for(int i = 0; i < 3; i++)
        {
            Priests[i].ChangeDirction();
            Devils[i].ChangeDirction();
            if(Priests[i].GetSide() == 1)
            {
                rightPriestNum++;
            }
            else
            {
                leftPriestNum++;
            }

            if (Devils[i].GetSide() == 1)
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
                Devils[i].Lose();
            }
            gaming = false;
            return 1; //win
        }
        else if ((leftPriestNum > 0 && leftDevilNum > leftPriestNum) || (rightPriestNum > 0 && rightDevilNum > rightPriestNum))
        {
            int attackSide;
            if (leftDevilNum > leftPriestNum)
                attackSide = -1;
            else
                attackSide = 1;
            for(int i = 0; i < 3; i++)
            {
                if(Devils[i].GetSide() == attackSide)
                    Devils[i].Attack();
            }
            gaming = false;
            return -1; //lose
        }
        else
        {
            return 0; //continue
        }
    }

    public void MoveRole(RoleModel role)
    {
        if (!gaming)
            return;
        //当其他角色还在移动时，不允许移动
        for (int i = 0; i < 3; i++)
        {
            if (Priests[i].IsMoving() || Devils[i].IsMoving())
                return;
        }
        //如果还在前进，return
        if (Boat.IsMoving())
            return;

        if (role.IsOnBoat())        //上岸
        {
            //这里，为了和原游戏一致，使牧师和魔鬼在两边的排列顺序一致
            role = Boat.DeletePassenger(role);
            Vector3 direction;
            int id = role.GetName()[role.GetName().Length - 1] - '0';
            if (role.IsGood())
            {
                direction = PrisetsOriginPositions[id];
            }
            else
            {
                direction = DevilsOriginPositions[id];
            }
            if(role.GetSide() == -1)
            {
                direction.x = 0 - direction.x;
            }
            role.Move(direction);
            role.GoLand();
        }
        else                        //上船
        {
            if (Boat.IsFull() || Boat.GetSide() != role.GetSide())
            {
                return;
            }
            role.Move(Boat.getEmptyPosition());
            role.GoBoat(Boat);
            Boat.AddPassenger(role);
        }
    }

    public void MoveBoat()
    {
        //当船为空，或者船已经在动时，不允许移动船
        if (!Boat.IsEmpty() && !Boat.IsMoving())
        {
            //当角色还在移动时，不允许移动船
            for (int i = 0; i < 3; i++)
            {
                if (Priests[i].IsMoving() || Devils[i].IsMoving())
                    return;
            }

            Boat.Move();
        }
    }
}
```

完成上述代码后，游戏的逻辑也基本完成了。

###### 细节部分

+ 船未靠岸，牧师与魔鬼上下船运动中，均不能接受用户事件：只需要在UserAction的函数中判断Move类中的state即可(具体代码见上面)，如果state表明正在移动，则直接返回，不进行任何操作。
+ 同样的，当游戏胜利或者失败后，同样不能接受除了Restart以外的任何用户事件。

###### 模型动作

在下载模型时，发现了模型好像自带有动画(Animator组件)，于是就上网查了以下如何给人物附加动画。大致就是创建一个Animator Controller，通过给模型创建状态，并为不同状态绑定不同的动画效果，状态之间的转换需要一些特定的条件，在代码中触发这些条件实现动画切换。

上述可能解释的不是很清楚，具体可以参考这个[博客](https://blog.csdn.net/ChinarCSDN/article/details/81437311)

###### 游戏主界面

{% asset_img p&d.png %}

[完整游戏视频](https://www.bilibili.com/video/av68331555)

[项目地址](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Priests_and_Devils)

###### 游戏优化

+ 人物的移动做的还是比较粗糙，特别是河岸和船之间的路，我参考了往届同学的思路，使用了一个中间点用于转折，但应该还可以有更好的实现。
+ 可以模仿原游戏，增加计时功能；也可以在起始时增加选项，根据玩家选择的难度增加牧师与恶魔的数量，以增加游戏难度与趣味。
+ 在开始和结束游戏时，制作额外的场景用于和玩家交互

下次的作业还是这个游戏，希望可以完成一些上述提到的想法，欢迎关注后续文章。

###### 感想

这应该算是一个比较完整的游戏了，整体做下来以后，感觉按这样的思路设计游戏的话，还是十分复杂的。虽然现在对于Unity的使用还只是停留在移动、旋转对象的层面，但即使这样一个逻辑十分简单的游戏，仍有很多细节需要去判断，需要制定好每个对象的行为和位置，否则很容易出现Bug，就更不用说那些复杂的游戏了，简直不敢想象。我自身对于MVC模式掌握的也不够熟练，很多函数写的也比较繁琐。

但整个游戏的制作过程还是很愉快的，特别是在Asset Store下载了一些模型后，对于把游戏做精美的欲望更加强烈了。模型的使用、模型动画机制的运用让我对于接下来的学习3D游戏设计以及Unity的使用更加期待了。
