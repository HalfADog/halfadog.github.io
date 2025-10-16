---
title: Unity Navigation导航系统
date: 2024-02-07 17:57:00 +0800
categories: [Unity]
tags: [Unity,Navigation]
author: HalfDog
---

## 一、Navigation导航系统基本介绍

Navigation导航系统是unity的寻路组件，将静态或动态的复杂场景烘焙简化为简单的（NavMesh）导航网格用于AI寻路计算。值得注意的是，这个组件的NavMesh系统存在一些性能和使用场景上的缺陷，如很难在运行时修改navmesh、场景过大会造成内存上的开销，以及不支持垂直面的导航。以上问题可以被新的脚本组件式的NavMesh系统所解决，不过根据需要可以适当使用旧版的系统，接下来我们先介绍旧版的navigation系统打基础，之后再讲解新版的系统。

笼统的来讲，navigation系统包含了四个部分

NavMesh、NavMeshAgent、Off-MeshLink、NavMeshObstacle

即 导航网格、导航网格代理（寻路代理）、离散网格链接、障碍物（多用于动态）

## 二、旧版组件式Navigation系统

Navigation窗口在window->AI->Navigation处打开

![image](/assets/images/2024-02-07/1.png)

### 1.Navigation组件面板

Navigation组件是用于创建可导航的环境的工具。它包括创建和编辑NavMesh、设置导航代理和路径等功能。

![image](/assets/images/2024-02-07/2.png)

Agents：NavMesh Agent setting 导航网格代理

agent即代理，主要是为需要导航的游戏物体生成一个对应的圆柱体以用来在navmesh上进行导航计算（如上图）。

不同的Name属性代表不同的代理，比如游戏中有玩家有怪物，则所需的代理也不同，“+”按钮即可添加新代理。

radius属性、Height属性记录了代理的体积，即规定了代理能够通过的最小宽度和高度。

StepHeight属性记录代理能够通过的最高阶梯上限，MaxSlope属性记录能够通过的坡面角上限。

需要注意的是，Agents设置中的参数不参与NavMesh的生成，Agents的参数影响的是运行时agent的活动，例如躲避其他agent和动态的障碍

![image](/assets/images/2024-02-07/3.png)

Area : NavMesh Area setting 导航网格区域

代理在不同的导航区域有不同的行为，例如有Walkable区域，即可以行走UnWalkable区域即不能行走，Jump区域即是跳跃区域。

用户可以自定义导航区域类型，即可以自定义代理在某一区域的行为。

最右侧的数字即是每个区域所对应的权重，权重越接近于0优先级也就越高。优先级高的区域就会在生成导航路径时被优先考虑。

![image](/assets/images/2024-02-07/4.png)

Bake：NavMesh Bake Setting 导航网格烘焙

这一栏与Agents的设置有相似的地方，但Bake不同于Agents的地方在于，Bake中的Radius、Height等等参数是用来生成NavMesh的也就是导航网格。

最终的导航路线生成是在NavMesh的基础上计算的，所以在烘焙NavMesh时，参数的选择就要考虑到不同的agent（如果有的话），否则有可能就会出现一些agent穿模的情况。

总之NavMesh是固定的，生成NavMesh与Agents的设置无关。

Generated Off Mesh Links 是生成NavMesh时自动生成离散链接的选项，DorpHeight决定可以从一个到另一个连续或非连续的NavMesh跳跃下来的最大高度，JumpDistance决定从一个NavMesh平面跳跃到另一个等高NavMesh平面的最大距离。如果要自动生成OffMeshLink就要在Navigation的Object面板或Inspector面板的static下拉选项中当中勾选Generate OffMeshLinks。

![image](/assets/images/2024-02-07/5.png)

Object：Bake setting of currently selected object 当前选择物体的烘焙设置

这个页面显示了你当前选择物体的烘焙设置。

Navigation Static：只有勾选了此选项当前物体才参与到生成NavMesh中。

Generate OffMeshLinks：只有勾选了此选项当前物体才会生成OffmeshLink。

Navigation Area：当前导航区域的类型。

### 2.NavMesh Agent组件面板

NavMesh Agent是Unity中用于导航的组件之一，它可以使游戏对象在NavMesh上移动

![image](/assets/images/2024-02-07/6.png)

Agent Type：代理的类型，在Agents面板中定义1

Base Offset：代理的高度偏移 

Speed：代理的最大移动速度。

Angular Speed：代理的最大旋转速度。

Acceleration：代理的最大加速度。

Stopping Distance：代理需要到达目标点时停止的距离。

Auto Braking：是否自动刹车以避免超过目标点。

Radius：代理的半径。

Height：代理的高度。

Quality：代理的避障质量

Priority：代理的优先级，用于控制多个代理之间的相对重要性。

Auto Traverse Off Mesh Link：是否允许代理自动穿过断开的NavMesh。可以选择手动控制代理通过OffMeshLink，以控制玩家通过不同offmeshlink的行为。

这些参数可以用于控制代理在NavMesh上的移动和行为，其中一些参数如半径、高度和避障类型对代理的碰撞和障碍物避免非常重要。

### 3.Off-Mesh Link组件面板

Off-meshLink用于创建穿越可行走导航网格之外的路径
表面。例如，跳过沟渠或栅栏，或在穿过一扇门之前打开一扇门，都可以使用Off-meshLink。

![image](/assets/images/2024-02-07/7.png)

Start：开始位置

End：结束位置

Cost Override：可以理解为寻路时被选中的权重值，越低越容易被选择

Activated：是否启用

Auto Update Position：是否在端点移动时自动更新

值得注意的是，手动放置OffMeshLink的时候要注意将其放置在生成的NavMesh上，如果没有放置到位那么Agent在计算路线时会将其忽略。

定义了Off-MeshLink后可以通过手动的方式控制agent通过，总的来说就是在脚本中手动控制agent从start点到end点最后在告诉agent我手动控制完了你可以接管了，具体实现看官方手册。

### 4.NavMesh Obstacle组件面板

这个组件用于移动的障碍物，在运行时agent计算路径时会将运动的障碍物考虑进去。

![image](/assets/images/2024-02-07/8.png)

Shape：障碍物的形状，可以理解为碰撞体的形状，有box和capsule两种

Center、Size：与Collider的设置相似

Carve：只有将这个选项激活这个障碍物才能参与到生成NavMesh中并根据shape的数据在NavMesh上创建孔洞。

Move Threshold：判定这个障碍物在移动的阈值（Threshold ）障碍移动超过移动阈值设置的距离时，Unity将其视为移动

Time To Stationary（静止的）：障碍物静止时间超过了设定的时间后会被视为静止。

Carve Only Stationary：当启用时，障碍物只有在静止时才会参与NavMesh的生成。

### 5.手动控制Agent通过Off-MeshLink区域

代码中一些重要的字段及方法

isOnOffMeshLink、currentOffMeshLinkData、autoTraversOffMeshLink、baseOffset、CompleteOffMeshLink()；

以上字段及方法都是NavMeshAgent组件上的。

对于不同的OffMeshLink Area，可以通过OffMeshLinkData.offMeshLink.area访问到当前的area类型，进而进行不同的通过操作。

```csharp
//在offMeshLink的climb区域手动操作agent通过
if(agent.isOnOffMeshLink){
    OffMeshLinkData linkData = agent.currentOffMeshLinkData;
    if(linkData.offMeshLink == null)return;
    //如果当前offmeshlink区域的类型是climb类型
    if(linkData.offMeshLink.area == 3){
        //就暂停agent的自动通过offmeshlink
        agent.autoTraverseOffMeshLink = false;
    }
    if(!agent.autoTraverseOffMeshLink){
        Vector3 startPos = linkData.startPos+Vector3.up*agent.baseOffset;
        Vector3 endPos = linkData.endPos+Vector3.up*agent.baseOffset;
        //先控制agent到startPos
         if(!isOnOffMeshLinkStartPos){
            transform.position = Vector3.MoveTowards(transform.position,startPos,agent.speed*Time.deltaTime);
            if(transform.position.Equals(startPos))isOnOffMeshLinkStartPos = true;
        }
        //再从startPos到endPos
        else{
            climbTimer+=Time.deltaTime;
            transform.position = Vector3.Slerp(startPos,endPos,climbTimer/climbTime);
            if(transform.position == endPos){
                //让agent做最后一点收尾工作彻底通过offmeshlink
                agent.CompleteOffMeshLink();
                //调用CompleteOffMeshLink之后isOnOffMeshLink就为false
            }
        }
    }
}
else{
    //相关变量初始化为下一次通过做准备
    climbTimer = 0.0f;
    agent.autoTraverseOffMeshLink = true;
    isOnOffMeshLinkStartPos = false;
}
```

## 三、新版脚本式Navigation导航系统

具体如何使用参考官方手册 
一般使用旧版系统就够用了
