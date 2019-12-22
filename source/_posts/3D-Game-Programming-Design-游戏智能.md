---
title: 3D Game Programming & Design - 游戏智能
date: 2019-12-01 14:45:16
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程的第十次作业：
>
> + 牧师与恶魔 - 智能帮助版
>
> [课程主页]( https://pmlpml.github.io/unity3d-learning/10-intelligent.html )

<!-- more -->

[项目地址]( https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Priests_and_Devils_v3 )

[演示视频]( https://www.bilibili.com/video/av77791428/ ) 

### 简介

在上一次的[牧师与恶魔]( [https://danielxuuuuu.github.io/2019/09/22/3D-Game-Programming-Design-%E6%B8%B8%E6%88%8F%E5%AF%B9%E8%B1%A1%E4%B8%8E%E5%9B%BE%E5%BD%A2%E5%9F%BA%E7%A1%80/](https://danielxuuuuu.github.io/2019/09/22/3D-Game-Programming-Design-游戏对象与图形基础/) )游戏中，我们实现了游戏对象的动作分离，即使用单独的动作管理器来管理动作，使得代码得以解耦，可扩展性更高。但这个游戏的难度对于初次接触这类游戏的人来说，还是有点难度的，因此，在学习了游戏智能后，本次实践中将在游戏中增加提示功能，来引导玩家完成这个游戏，使得游戏的体验更好。

### 要求

- 实现状态图的自动生成
- 讲解图数据在程序中的表示方法
- 利用算法实现下一步的计算

### 实现过程

#### 游戏状态图

相信接触过算法知识的同学都应该清楚，在有的时候，当已知一个初始状态和一个结束状态，要求如何从这个初始状态开始，一步一步按照一个既定的规则转变成结束状态的问题，实际上就是一个对于过程中可能出现的状态的搜索过程，而我们要做的就是在整个状态空间中找到我们要的转换顺序。

同样的，对于我们这个游戏中的每一个阶段，我们都可以用一个状态$(P_{num}D_{num}B)$来表示。由于牧师和恶魔的数量固定的，同时为了表示方便，状态$(P_{num}D_{num}B)$仅表示当前游戏画面中右岸的情况，左岸的情况可以十分快速的求得。其中下标`num`表示当前右岸的牧师（恶魔）数量，B表示当前船是否在右岸。例如：游戏一开始时的状态为$(P_{3}D_{3}B)$，并且我们期望达到的目标状态为$(P_{0}D_{0})$。

有了状态，当然需要有状态转移，对于本游戏来说，状态的转移就是船的移动导致的右岸牧师和恶魔数量的变化，即状态的变化。状态转移条件表示为每移动一次船，船上的人员情况（总人数不超过2人）。

在明确了以上表示方法后，就可以画出整个游戏的状态转移图了，为了不重复造轮子，这里借用一下[师兄]( https://blog.csdn.net/kiloveyousmile/article/details/71727667 )画好的图：

{% asset_img state.jpg %}

上图中双箭头表示两个状态之间可以相互转换。其中两个`*`的状态表示游戏失败状态。可以看到，上图中左上角的状态即为我们的初始状态，从这个状态进行转移可以到达4个状态，但只有两个状态是可以最终通过变换达到目标（左下角）状态的。所以，当我们在游戏设计中给玩家增加游戏帮助功能时，就是从上图中玩家所处的当前状态入手，找到最佳的选择，并根据下一个状态自动执行的上下船操作。

#### 状态图的表示与生成

有了可视化的图，当然就需要把这个图用代码表示出来，而且根据要求，状态图需要自动生成。这里选择使用程序中表示图比较通用的方式—邻接表，邻接表的表示法需要两个类，一个类表示整个图，其中记录了图中所有的节点，另一个类表示单个节点，其中包含了与其相邻的节点集合。

具体的实现中，我使用了三个类来实现完整的状态图表示，包括节点类、图类，以及一个用于指示下一步如何走的move类。

##### move类

move类简单的记录了船在一次移动中承载的牧师和恶魔数量，在后续的建图和获得下一步的过程中会使用到。

```c#
public class move
{
    public int priest_num;
    public int devil_num;
    public move(int pnum, int dnum)
    {
        this.priest_num = pnum;
        this.devil_num = dnum;
    }
}
```

##### 状态节点stateNode类

如前文所说，状态图中的状态表示的都是右岸中的人员以及船是否停靠情况。按照这个思路，我们可以定义以下类，这个类中除去状态节点所要包含的信息外，还包含了一个`List`数据结构，用于保存与其相邻的节点。类中有一个`move()`成员函数，可以得到当前状态在经过对应的移动后的新状态。

```c#
public class stateNode
{
    public int priest_num;  // 右岸牧师数量
    public int devil_num;   // 右岸魔鬼数量
    public bool boat;       // 船是否停靠在右岸
    public List<stateNode> nextStates; // 相邻节点

    public stateNode start; // 在搜索时用到，用于记录最短路径对应的起始步

    public stateNode(int pnum, int dnum, bool boat)
    {
        this.priest_num = pnum;
        this.devil_num = dnum;
        this.boat = boat;
        nextStates = new List<stateNode>();
    }

    // 加入相邻节点
    public bool addNextState(stateNode nextState)
    {
        foreach (stateNode node in nextStates)
        {
            if (nextState == node)
            {
                return false;
            }
        }

        nextStates.Add(nextState);
        return true;
    }

    // 获得当前可选择的走法
    public move[] getAvailableMove()
    {
        move[] res = new move[nextStates.Count];
        for(int i = 0; i < nextStates.Count; i++)
        {
            res[i] = new move(Mathf.Abs(nextStates[i].priest_num - this.priest_num), Mathf.Abs(nextStates[i].devil_num - this.devil_num));
        }
        return res;
    }

    public void move(move mv)
    {
        if (boat)
        {
            this.priest_num -= mv.priest_num;
            this.devil_num -= mv.devil_num;
        }
        else
        {
            this.priest_num += mv.priest_num;
            this.devil_num += mv.devil_num;
        }
        this.boat = !this.boat;
    }

    // 船行驶向左岸，相当于右岸少人了
    public static stateNode operator - (stateNode from, move mv)
    {
        stateNode to = new stateNode(from.priest_num - mv.priest_num, from.devil_num - mv.devil_num, false);
        return to;
    }

    // 船行驶回右岸，相当于右岸多人了
    public static stateNode operator +(stateNode from, move mv)
    {
        stateNode to = new stateNode(from.priest_num + mv.priest_num, from.devil_num + mv.devil_num, true);
        return to;
    }
    // 重载==和!=运算符
    public static bool operator ==(stateNode state1, stateNode state2)
    {
        return state1.priest_num == state2.priest_num && state1.devil_num == state2.devil_num && state1.boat == state2.boat;
    }

    public static bool operator !=(stateNode state1, stateNode state2)
    {
        return state1.priest_num != state2.priest_num || state1.devil_num != state2.devil_num || state1.boat != state2.boat;
    }
}
```

##### 状态图stateGraph类

状态图类包含了上述状态节点类，由于每个节点的相邻节点已经保存在自身的成员变量中了，状态图类只需要保存图中的节点即可，同样使用。

图生成函数`generateGraph()`的原理比较简单：先把起始节点加入我们的图中，然后对于`List`中的每个节点（即每一个状态），都会执行`prosibleMove`数组中的五种移动，并对得到的结果进行合法性判断（即会不会导致游戏失败），如果是合法的就加入图中并把节点相连（加入图中和加入节点的相邻节点列表都需要进行是否已存在的判断）。

**获取下一步走法的函数`getNextMove()`**也包含在这个图类中，该函数接收一个当前状态，返回最优的下一步走法。`getNextMove()`中又调用了`getNextState()`函数，该函数通过广度优先搜索（BFS）的方式查找到目标状态的最短路径。具体的实现见代码。

```c#
// 状态图类
public class stateGraph
{
    public List<stateNode> nodes;
    public move[] prosibleMove =
    {
        new move(0, 1),
        new move(1, 0),
        new move(1, 1),
        new move(2, 0),
        new move(0, 2),
    };
    public stateNode startState;
    public stateNode endState;

    public stateGraph()
    {
        nodes = new List<stateNode>();
        startState = new stateNode(3, 3, true);
        endState = new stateNode(0, 0, false);
        generateGraph();
    }

    // 图生成函数
    private void generateGraph()
    {
        // 首先加入初始状态节点
        nodes.Add(startState);

        for(int i = 0; i < nodes.Count; i++)
        {
            stateNode currentState = nodes[i];
            foreach(move mv in prosibleMove)
            {
                stateNode nextState;
                if (currentState.boat)
                {
                    nextState = currentState - mv;
                }
                else
                {
                    nextState = currentState + mv;
                }

                if (isLegalState(nextState))
                {
                    nextState = addNewStateToGraph(nextState);
                    currentState.addNextState(nextState);
                    nextState.addNextState(currentState);
                }

            }
        }
    }

    // 将节点加入图中
    private stateNode addNewStateToGraph(stateNode newState)
    {
        // 先判断列表里当前有没有
        foreach (stateNode state in nodes)
        {
            if (state == newState)
            {
                return state;
            }
        }
    
        nodes.Add(newState);
        return newState;
    }

    private stateNode getGraphNode(stateNode newState)
    {
        foreach (stateNode state in nodes)
        {
            if (state == newState)
            {
                return state;
            }
        }
        return null;
    }

    // 验证状态是否合法
    private bool isLegalState(stateNode state)
    {
        return (state.priest_num >= state.devil_num || state.priest_num == 0)
            && ((3 - state.priest_num) >= (3 - state.devil_num) || (3 - state.priest_num) == 0)
            && (state.priest_num <= 3 && state.devil_num <= 3)
            && ((3 - state.priest_num) <= 3 && (3 - state.devil_num) <= 3);
    }

    public move getNextMove(stateNode currentState)
    {
        // 找到图中真正的节点
        currentState = getGraphNode(currentState);
        stateNode next = getNextState(currentState);
        return new move(Mathf.Abs(currentState.priest_num - next.priest_num), Mathf.Abs(currentState.devil_num - next.devil_num));
    }

    // 找到最短路径中当前状态的下一步
    private stateNode getNextState(stateNode from)
    { 
        List<stateNode> alreadSearchState = new List<stateNode>();
        alreadSearchState.Add(from);

        foreach (stateNode state in from.nextStates)
        {
            // 判断是不是终点
            if(state == endState)
            {
                return state; 
            }
            else
            {
                state.start = state;
                alreadSearchState.Add(state);
            }
        }

        for(int i = 1; i < alreadSearchState.Count; i++)
        {
            foreach(stateNode state in alreadSearchState[i].nextStates)
            {
                // 判断是不是终点
                if (state == endState)
                {
                    state.start = alreadSearchState[i].start;
                    return state.start;
                }
                else
                {
                    bool flag = false;
                    // 检查是否已经搜索过
                    foreach (stateNode searchedState in alreadSearchState)
                    {
                        if (state == searchedState)
                        {
                            flag = true;
                            break;
                        }
                    }
                    if (!flag)
                    {
                        state.start = alreadSearchState[i].start;
                        alreadSearchState.Add(state);
                    }
                }
            }
        }
        return null;
    }
}
```

以上就完成了程序中对于状态图的自动生成，并可以根据当前状态得到最优的下一步。接下来就是需要对于原代码的一些部分进行调整即可。

#### 调整原代码

##### 场记调整

首先在场记中增加以下两个变量，并在初始化时对其进行初始化，由上文可知，此时整个图已经生成了。

```c#
public stateGraph graph;        // 状态图
public stateNode currState;     // 当前游戏状态对应的状态节点，这个状态节点只是单纯的记录，和图中真正的记录节点不同

graph = new stateGraph();
currState = new stateNode(3, 3, true);
```

在每次我们移动船的时候（不管是手动移动还是自动移动），都需要改变当前的游戏状态节点。先统计移动时船上的人员情况，再通过我们的move类实现状态转移。

```c#
public void MoveBoat()
{
    //当船为空，或者船或人物在运动时，不允许移动船
    if (Boat.IsEmpty() || moving)
        return;
    RoleModel[] passengers = Boat.GetPassengers();
    int priest_num = 0, devil_num = 0;
    for (int i = 0; i < 2; i++)
    {
        if (passengers[i] != null)
        {
            if (passengers[i].IsGood())
                priest_num++;
            else
                devil_num++;
        }
    }
    currState.move(new move(priest_num, devil_num));
    actionManager.MoveBoat(Boat.GetBoat(), Boat.GetMoveDirection(), speed);
}
```

##### 增加AI函数

AI函数实现自动获取最佳的下一步，并自动执行上下船和开船操作。

由于在之前的设计中，我设定了人物和船在移动的时候都是有一段时间间隔的，在这个事件间隔中，玩家无法进行其他点击事件，但是这里如果在一个函数中，一下子把上下船和开船操作同时执行了，就会导致错误，所以我使用了协程的方式，`yield return new WaitForSeconds(2f);`使得每一个动作分阶段执行。但由于这样又变成了人为设定的时间间隔，<u>有时候间隔会显得有点长</u>，具体的效果可以看演示视频。

```c#
public void AI()
{
    if (!gaming || moving)
        return;
    StartCoroutine(AIroutine());
}

IEnumerator AIroutine()
{
    // 搜索图得到最佳的下一步
    move mv = graph.getNextMove(currState);
    Debug.Log(mv.priest_num);
    Debug.Log(mv.devil_num);

    // 自动执行上船下船操作

    // 先统计当前船上的人员情况
    int p_num = 0, d_num = 0;
    RoleModel[] passengers = Boat.GetPassengers();
    for (int i = 0; i < 2; i++)
    {
        if (passengers[i] != null)
        {
            if (passengers[i].IsGood())
                p_num++;
            else
                d_num++;
        }
    }

    // 正数表示上岸，复数表示上船
    p_num -= mv.priest_num;
    d_num -= mv.devil_num;
    int side = Boat.GetSide();

    if (p_num == 0 && d_num == 0)
    {
        MoveBoat();
    }
    else
    {
        // 先执行上岸操作，给船留出位置
        if (p_num > 0)
        {
            int temp = 0;
            for (int i = 0; i < 2; i++)
            {
                if (passengers[i] != null)
                {
                    if (passengers[i].IsGood())
                    {
                        MoveRole(passengers[i]);
                        yield return new WaitForSeconds(2f);
                        temp++;
                        if (temp == p_num)
                            break;
                    }
                }
            }
        }
        if (d_num > 0)
        {
            int temp = 0;
            for (int i = 0; i < 2; i++)
            {
                if (passengers[i] != null)
                {
                    if (!passengers[i].IsGood())
                    {
                        MoveRole(passengers[i]);
                        yield return new WaitForSeconds(2f);
                        temp++;
                        if (temp == d_num)
                            break;
                    }
                }
            }
        }
        if (p_num < 0)
        {
            int temp = 0;
            for (int i = 0; i < 3; i++)
            {
                if (Priests[i].GetSide() == side && !Priests[i].IsOnBoat())
                {
                    MoveRole(Priests[i]);
                    yield return new WaitForSeconds(2f);
                    temp--;
                    if (temp == p_num)
                        break;
                }
            }
        }
        if (d_num < 0)
        {
            int temp = 0;
            for (int i = 0; i < 3; i++)
            {
                if (Devils[i].GetSide() == side && !Devils[i].IsOnBoat())
                {
                    MoveRole(Devils[i]);
                    yield return new WaitForSeconds(2f);
                    temp--;
                    if (temp == d_num)
                        break;
                }
            }
        }
        MoveBoat();
    }
}
```

##### GUI调整

增加按钮，调用`AI()`函数。

```c#
if (GUI.Button(new Rect(Screen.width / 2 + 40, 80, 60, 60), "Help!"))
{
 	action.AI();
}
```

同时注意，在`Restart`按钮的点击事件中，需要重置游戏状态到最开始状态。

### 游戏画面

{% asset_img game.png %}