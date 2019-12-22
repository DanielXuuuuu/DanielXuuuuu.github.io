---
title: 3D Game Programming & Design - 离散仿真引擎基础
date: 2019-09-04 09:56:32
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程的第二次作业，初次接触Unity3D.

{% asset_img title.jpg %}

<!--more-->

##### 解释游戏对象(GameObjects) 和资源(Assets)的区别与联系

+ 游戏对象：指出现在游戏场景中的每一个功能对象，例如人物、道具、场景等。游戏对象本身不做任何事情，就像一个空盒子，通过在里面装入需要的组件（Components）来实现具体的功能。类似于资源的集合体，是资源整合的具体表现。
+ 资源：指任何可以被加入到游戏中的素材资源，可以被多个对象使用。资源既可以来自unity3D外部，例如3D模型，音频，图像等，也可以来自内部，例如动画控制器，纹理渲染等。资源可用于实例化游戏中的具体对象。

##### 下载几个游戏案例，分别总结资源、对象组织的结构(指资源的目录组织结构与游戏对象树的层次结构)

+ 资源的目录组织结构：包括图片、场景、脚本、模型、素材等
+ 游戏对象树的层次结构：包括摄像机包，场景布局，文本等

##### 编写一个代码，使用debug语句来验证MonoBehaviour基本行为或事件触发的条件

​	代码如下：

```c#
public class test : MonoBehaviour
{
    void Awake()
    {
        Debug.Log("onAwake");
    }

    // Start is called before the first frame update
    void Start()
    {
        Debug.Log("onStart");
    }

    // Update is called once per frame
    void Update()
    {
        Debug.Log("onUpdate");
    }

    void FixedUpdate()
    {
        Debug.Log("onFixedUpdate");
    }

    void LateUpdate()
    {
        Debug.Log("onLateUpdate");
    }

    void OnGUI()
    {
        Debug.Log("onGUI");
    }

    void OnDisable()
    {
        Debug.Log("onDisable");
    }

    void OnEnable()
    {
        Debug.Log("onEnable");
    }
}
```

MonoBehaviour是Unity中所有行为对象的基类，其中有以下可重写函数：

- 基本行为
  - Awake()：当一个脚本实例被载入时调用，因此大多用于成员变量的初始化
  - Start()：在所有Update函数之前，在Awake之后被调用一次，可以把一些需要依赖Awake的变量放在Start里面初始化
  - Update()：当行为启动时，其Update在每一帧被调用
  - FixedUpdate()：当行为启动时，其FixedUpdate在每一时间片（每一固定帧）被调用
  - LateUpdate()：当行为启动时，在所有Update函数之后执行，例如：摄像机的跟随
- 常用事件
  - OnGUI()：绘制GUI时调用
  - OnDisable()：当对象变为不可用或非激活状态时调用；当物体销毁时调用；
  - OnEnable()：当对象变为可用或激活状态时调用
  - OnDestory()：当对象被销毁时调用



##### 查找脚本手册，了解GameObject，Transform，Component 对象

###### 分别翻译官方对三个对象的描述（Description）

- **GameObject**：GameObjects are the fundamental objects in Unity that represent characters, props and scenery. They do not accomplish much in themselves but they act as containers for Components, which implement the real functionality.

  游戏对象是Unity中的基本对象，代表了角色，道具和场景。它们本身的功能并不完整，但它们充当了组件的容器，得以实现真正的功能。

- **Transform**：Every object in a Scene has a Transform. It's used to store and manipulate the position, rotation and scale of the object. 

  场景中的每个对象都有一个变换，用于存储和操纵对象的位置，旋转和缩放。

- **Component**：Base class for everything attached to GameObjects. Note that your code will never directly create a Component. Instead, you write script code, and attach the script to a GameObject.

  绑定到游戏对象上的一组相关属性，通过编写脚本代码来创建组件。

###### 描述下图中 table 对象（实体）的属性、table 的 Transform 的属性、 table 的部件

{% asset_img 1.png %}

+ table的对象是GameObject。在第一行中：第一个选择框是activeSelf属性，第二个文本框是对象名称，第三个选择框是static属性；第二行中：从左至右为Tag属性和Layer属性；第三行为prefab（预设）属性。

- table的Transform属性包含position（位置），Rotation（旋转）以及Scale（比例）
- table的部件Component对象有Transform，Mesh Filter，Box Collider，Mesh Renderer，Default-Material。

###### 用 UML 图描述 三者的关系

{% asset_img uml.png %}

##### 整理相关学习资料，编写简单代码验证以下技术的实现：

###### 查找对象

- 按名字查找 `public static GameObject Find(string name)`
- 按标签查找单个对象 `public static GameObject FindWithTag(string tag)`
- 按标签查找多个对象：`public static GameObject[] findGameObjectsWithTag(string tag)`

###### 添加子对象

- `public static GameObject CreatePrimitive(PrimitiveType type)`

###### 遍历对象树

```c#
foreach (Transform child in transform){
	Debug.Log(child.gameObject.name);
}
```

###### 清除所有子对象

```c#
foreach (Transform child in transform){
	Destory(child.gameObject);
}
```

##### 资源预设(Prefabs)与对象克隆(clone)

###### 预设（Prefabs）有什么好处？

- 预设是Unity中的一种特殊资源，通常就是一个或者一系列组件的集合体，类似于一个模板。预设创建后，可以对其进行实例化出一些具有相同属性的对象，这些对象与预设关联。若预设发生改变，所有关联的对象都会发生相应变化。
- 好处：
  - 批量处理方便
  - 使对象和资源能重复利用，提高效率

###### 预设与对象克隆 (clone or copy or Instantiate of Unity Object) 关系？

- 预设是从无到有的创建，供以后的需要；而对象克隆需要场景中实现存在被克隆对象
- 预设创建出来的对象会随着与之相关联的预设的变化而变化，而克隆出的相当于一个独立的新对象，不会因被克隆对象的改变而改变

###### 制作 table 预制，写一段代码将 table 预制资源实例化成游戏对象

```c#
GameObject anotherTable = (GameObject)Instantiate(table.gameObject);
```



##### 编程实践—小游戏

游戏内容： 简单计算器 

技术限制： 仅允许使用 **IMGUI** 构建 UI

作业目的：
- 提升 debug 能力
- 提升阅读 API 文档能力

项目地址：[Tic Tac Toe](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Tic-Tac-Toe)

运行后结果如下：

- 初始界面：

  {% asset_img gameStart.png %}
  
- 胜出界面：

  {% asset_img gameOver.png %}



##### 思考题【选作】

###### 微软 XNA 引擎的 Game 对象屏蔽了游戏循环的细节，并使用一组虚方法让继承者完成它们，我们称这种设计为“模板方法模式”。

- 为什么是“模板方法”模式而不是“策略模式”呢？
- **策略模式**定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。
- **模板方法模式**在一个方法中定义一个算法骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。
- 结合以上定义，我们可以看到这两个模式虽然同样都用到了继承和虚函数的技术，但应用场景有着较大的区别，策略模式主要是定义一系列相同接口的算法，通过委托（delegate）来达到动态改变算法；而模板方法模式用于定义一个大框架，子类根据实际情况具体实现这个框架。
- 对于微软XNA引擎的Game基类，其真实的目的是为游戏提供一个基础框架，给用户一个编程模板，用户去实现具体的函数即可。因此是模板方法模式。
- 这启发我们，思考和辨别设计模式的时候，应该看问题的背景和上下文，而不是技术的解决方法。

###### 将游戏对象组成树型结构，每个节点都是游戏对象（或数）

- 尝试解释组合模式（Composite Pattern / 一种设计模式）

  - 组合模式，又叫部分-整体模式，将对象组合成**树形结构**以表示“部分-整体”的层次结构，把一组相似的对象当作一个单一的对象，用户对单个对象和组合对象的使用具有一致性。其关键点在于简单对象和复合对象必须实现相同的接口

- 使用 BroadcastMessage() 方法，向子对象发送消息。你能写出 BroadcastMessage() 的伪代码吗?

  ```c#
  //父对象
  void test(){
  	Debug.Log("我是父对象");
  }
  
  void Start(){
  	this.BroadcastMessage("test");
  }
  
  
  //子对象
  void test(){
  	Debug.Log("我是子对象");
  }
  ```

###### 一个游戏对象用许多部件描述不同方面的特征。我们设计坦克（Tank）游戏对象不是继承于GameObject对象，而是 GameObject 添加一组行为部件（Component）。

- 这是什么设计模式？
  - **Decorator** 装饰器模式，通过在原有类的基础上通过包装组合增加功能，以实现更多的功能
- 为什么不用继承设计特殊的游戏对象？
  - 使用继承来设计各种特殊的游戏对象，会导致产生很多父类和子类，而且产生的继承关系十分复杂，增加编程人员的编程难度。
  - 同时，一旦需要增加新的行为，可能会需要去修改很多相关类，耦合性强，使得编程效率变得很低。