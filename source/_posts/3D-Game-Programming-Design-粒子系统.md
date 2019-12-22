---
title: 3D-Game-Programming-Design-粒子系统
date: 2019-10-29 13:49:22
categories: 
	- 3D游戏编程
tags: 
	- 课程作业	
	- Unity
---

> 3D游戏编程课程的第八次作业：
>
> +  参考 http://i-remember.fr/en 网站，实现“粒子光环” 
>
> [课程主页](https://pmlpml.github.io/unity3d-learning/08-particle-system)

<!-- more -->

## 粒子光环

[项目地址](https://github.com/DanielXuuuuu/Unity3D-Learning/tree/master/Particle_Systems )

[演示视频](https://www.bilibili.com/video/av74442218/ )

#### 原网站效果图

{% asset_img originsitepng %}

鼠标悬停到中间的加号时，粒子光环会收缩；鼠标离开后，恢复原状。

#### 实现步骤

以下步骤是对于上图粒子光环的模仿，通过粒子流编程控制来实现。 

[参考博客](https://16sixteen.github.io/unity3d/unity3d_particle_ring)

##### 创建粒子系统

经过仔细观察，可以发现，该网站的粒子光环实际上是由两个环叠加而成，其中，外环做顺时针运动，而内环做逆时针运动，且外环比内环更粗一些，粒子也更加稀疏一点。

因此，基于上述观察，我们创建两个环，项目结构如下：

{% asset_img programStructure.png%}

##### 外光环

光环的实现还是非常简单的，编程控制粒子简单来说就是创建一个很大的粒子数组，并对于这个数组中的每个粒子，设定好其初始属性以及后续的运动轨迹：在下面这段脚本代码中，我们一共创建了4500个粒子，然后设定每个粒子的运动速度。随后，在一个范围内随机生成粒子的半径信息，在一个圆周范围内随机生成粒子相对于圆心的角度信息，并且保存，使得粒子的分布呈现圆环状。

在`Update`函数中，根据之前设定的速度值，随机改变角度信息并重新生成位置，然后通过`SetParticles`函数实现所有粒子位置的更新，形成粒子绕着圆周运动的视觉效果。

此外，`Update`函数中用于收缩变换的部分在后文会讲到，我们在`nonCollectRadius`和`collectRadius`两个数组中记录了粒子收缩前后的位置信息。

```c#
public class OuterRing : MonoBehaviour
{
    public ParticleSystem particleSystem;
    private ParticleSystem.Particle[] particles;
    private int particleNum = 4500;
    private float[] particleAngle;
    private float[] particleRadius;
    private float[] nonCollectRadius;
    private float[] collectRadius;
    private float minRadius = 3.0f, maxRadius = 4.8f;

    public float speed = 0.4f;
    public float collectSpeed = 2.5f;
    public bool isCollected = false;
    
    // Start is called before the first frame update
    void Start()
    {
        particleSystem = this.GetComponent<ParticleSystem>();
        particles = new ParticleSystem.Particle[particleNum];
        particleAngle = new float[particleNum];
        particleRadius = new float[particleNum];
        nonCollectRadius = new float[particleNum];
        collectRadius = new float[particleNum];
        particleSystem.maxParticles = particleNum;
        particleSystem.Emit(particleNum);
        particleSystem.GetParticles(particles);
        for(int i = 0; i < particleNum; i++)
        {
            float radius = Random.Range(minRadius, maxRadius);
            float angle = Random.Range(0.0f, 360.0f);
            float rad = angle / 180 * Mathf.PI;
            particles[i].position = new Vector3(radius * Mathf.Cos(rad), radius * Mathf.Sin(rad), 0.0f);

            particleAngle[i] = angle;
            particleRadius[i] = radius;

            nonCollectRadius[i] = radius;
            collectRadius[i] = radius - 1.5f * (radius / minRadius);
        }
        particleSystem.SetParticles(particles, particleNum);
    }

    // Update is called once per frame
    void Update()
    {
        for (int i = 0; i < particleNum; i++)
        {
            if (isCollected)
            {
                if (particleRadius[i] > collectRadius[i])
                {
                    particleRadius[i] -= collectSpeed * (particleRadius[i] / collectRadius[i]) * Time.deltaTime;
                }
            }
            else
            {
                if (particleRadius[i] < nonCollectRadius[i])
                {
                    particleRadius[i] += collectSpeed * (nonCollectRadius[i] / particleRadius[i]) * Time.deltaTime;
                }
                else if (particleRadius[i] > nonCollectRadius[i])
                {
                    particleRadius[i] = particleRadius[i];
                }
            }
            particleAngle[i] -= Random.Range(0, speed);
            float rad = particleAngle[i] / 180 * Mathf.PI;
            particles[i].position = new Vector3(particleRadius[i] * Mathf.Cos(rad), particleRadius[i] * Mathf.Sin(rad), 0.0f);
        }
        particleSystem.SetParticles(particles, particleNum);
    }
}
```

##### 内光环

内光环与外光环类似，改动的地方主要是粒子半径、数量的减少，粒子运动速度的变化。

```c#
public class InnerRing : MonoBehaviour
{
    public ParticleSystem particleSystem;
    private ParticleSystem.Particle[] particles;
    private int particleNum = 3000;
    private float[] particleAngle;
    private float[] particleRadius;
    private float[] nonCollectRadius;
    private float[] collectRadius;
    private float minRadius = 3.3f, maxRadius = 4.2f;

    public float speed = 0.2f;
    public float collectSpeed = 3f;
    public bool isCollected = false;

    // Start is called before the first frame update
    void Start()
    {
        particleSystem = this.GetComponent<ParticleSystem>();
        particles = new ParticleSystem.Particle[particleNum];
        particleAngle = new float[particleNum];
        particleRadius = new float[particleNum];
        nonCollectRadius = new float[particleNum];
        collectRadius = new float[particleNum];
        particleSystem.maxParticles = particleNum;
        particleSystem.Emit(particleNum);
        particleSystem.GetParticles(particles);
        for (int i = 0; i < particleNum; i++)
        {
            float radius = Random.Range(minRadius, maxRadius);
            float shiftMinAngle = Random.Range(0, 135);
            float shiftMaxAngle = Random.Range(135, 180);
            float angle = Random.Range(shiftMinAngle, shiftMaxAngle);
            if (Random.Range(0, 100) < 50)
                angle += 180.0f;

            float rad = angle / 180 * Mathf.PI;
            particles[i].position = new Vector3(radius * Mathf.Cos(rad), radius * Mathf.Sin(rad), 0.0f);
            particleRadius[i] = radius;
            particleAngle[i] = angle;

            nonCollectRadius[i] = radius;
            collectRadius[i] = radius - 1.5f * (radius / minRadius);
        }
        particleSystem.SetParticles(particles, particleNum);
    }

    // Update is called once per frame
    void Update()
    {
        for (int i = 0; i < particleNum; i++)
        {
            if (isCollected)
            {
                if (particleRadius[i] > collectRadius[i])
                {
                    particleRadius[i] -= collectSpeed * (particleRadius[i] / collectRadius[i]) * Time.deltaTime;
                }
            }
            else
            {
                if (particleRadius[i] < nonCollectRadius[i])
                {
                    particleRadius[i] += collectSpeed * (nonCollectRadius[i] / particleRadius[i]) * Time.deltaTime;
                }
                else if (particleRadius[i] > nonCollectRadius[i])
                {
                    particleRadius[i] = particleRadius[i];
                }
            }
            particleAngle[i] += Random.Range(0, speed);
            float rad = particleAngle[i] / 180 * Mathf.PI;
            particles[i].position = new Vector3(particleRadius[i] * Mathf.Cos(rad), particleRadius[i] * Mathf.Sin(rad), 0.0f);
        }
        particleSystem.SetParticles(particles, particleNum);
    }
}
```

##### 优化

基于上述步骤，得到的结果如下：

{% asset_img game1.png %}

可以看到，由于我们通过随机数生成两个圆的半径，所以粒子在圆环中的分布很均匀，导致内外粒子光环的边界比较分明，视觉效果并不是很好，因此，参考师兄的博客，对于两个光环中粒子的半径设定都做以下修改：

```c#
float midR = (maxRadius + minRadius) / 2;
//最小半径随机扩大
float rate1 = Random.Range(1.0f, midR / minRadius);
//最大半径随机缩小
float rate2 = Random.Range(midR / maxRadius, 1.0f);
float radius = Random.Range(minRadius * rate1, maxRadius * rate2);
```

##### 聚集效果

原网站的效果是鼠标悬浮到圆心时整个光环就会向内聚集，离开后恢复。这里通过两个按钮实现类似的效果。

###### 两个光环

两个光环的实现方式类似，通过增加一个`isCollected`属性，控制其粒子的半径即可，在之前就已经提到，我们已经设定好了收缩和恢复前后的半径大小（我们使用固定值），那么直接根据`isCollected`属性判断当前所使用的半径即可。具体的`Update`函数如下：

```
void Update()
	{
        for (int i = 0; i < particleNum; i++)
        {
            if (isCollected)
            {
                if (particleRadius[i] > collectRadius[i])
                {
                    particleRadius[i] -= collectSpeed * (particleRadius[i] / collectRadius[i]) * Time.deltaTime;
                }
            }
            else
            {
                if (particleRadius[i] < nonCollectRadius[i])
                {
                    particleRadius[i] += collectSpeed * (nonCollectRadius[i] / particleRadius[i]) * Time.deltaTime;
                }
                else if (particleRadius[i] > nonCollectRadius[i])
                {
                    particleRadius[i] = particleRadius[i];
                }
            }
            particleAngle[i] -= Random.Range(0, speed);
            float rad = particleAngle[i] / 180 * Mathf.PI;
            particles[i].position = new Vector3(particleRadius[i] * Mathf.Cos(rad), particleRadius[i] * Mathf.Sin(rad), 0.0f);
        }
        particleSystem.SetParticles(particles, particleNum);
    }
```

###### UserGUI

和之前类似，使用UserGUI实现“收”和“散”两个按钮负责与用户进行交互。由于本次项目比较简单，没有控制器来传递消息，这里就直接通过函数`GetComponentsInChildren<OuterRing>()`，并且将以下脚本挂载到`Ring`这个空对象上，就可以得到它两个子对象——内外两个光环，从而控制之前介绍的`isCollected`属性。

```c#
public class UserGUI : MonoBehaviour
{
    private OuterRing[] outerRing;
    private InnerRing[] innerRing;

    private void Start()
    {
        outerRing = GetComponentsInChildren<OuterRing>() as OuterRing[];
        innerRing = GetComponentsInChildren<InnerRing>() as InnerRing[];
    }
    private void OnGUI()
    {
        GUIStyle button_style;
        button_style = new GUIStyle("button")
        {
            fontSize = 15
        };
        if(GUI.Button(new Rect(Screen.width - 150, Screen.height - 100, 100, 30), "收", button_style))
        {
            outerRing[0].isCollected = true;
            innerRing[0].isCollected = true;
        }
        if(GUI.Button(new Rect(Screen.width - 150, Screen.height - 50, 100, 30), "散", button_style))
        {
            outerRing[0].isCollected = false;
            innerRing[0].isCollected = false;
        }
    }
}
```

#### 最终效果

{% asset_img game2.png %}

