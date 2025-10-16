---
title: Unity Scriptable Object
date: 2024-02-09 01:09:00 +0800
categories: [Unity]
tags: [Unity,Scriptable Object]
author: HalfDog
---

## 如何理解Scriptable Object
Scriptable Object是一种数据容器（data container），通常被用来存储大量的数据，并且不依赖于类实例。换句话说，Scriptable Object本身就是一个存放数据的实例。Scriptable Object没有继承自MonoBehavior，而是继承自Scriptable Object，所以Scriptable Object不能当脚本挂载到GameObject上，也不能进行GetComponent等对GameObject的操作，同时Scriptable Object以Asset文件存储，但不论是MonoBehavior还是Scriptable Object最终都继承自Unity Object。

但要注意的是，虽然Scriptable Object没有继承自MonoBehavior，并且他是一种资源文件，但是Scriptable Object始终是以C#代码脚本的形式来编辑的，所以只要不涉及到MonoBehavior继承下来的操作，我们可以编写任何代码来实现我们的想要的功能。

那么接下来就介绍Scriptable Object的基本操作以及几种最常用的用法。

## 创建Scriptable Object Asset文件
首先像创建脚本一样创建一个C#脚本，打开后把类名后的继承的MonoBehavior改为Scriptable Object，这样我们就创建一个最简单的Scriptable Object的模板，之所以称之为模板，那是因为其以C#脚本的形式存储，我们还不能使用它。我们需要实例化一个继承自Scriptable Object的C#脚本，为了方便在Unity Editor中实例化一个Scriptable Object，我们在脚本类声明前添加CreateAssetMenu属性，这样我们就可以在Unity Asset面板中右键创建一个Scriptable Object实例了，如下图。

![image](/assets/images/2024-02-09/1369799-20240209010437503-965995927.jpg)

当我们实例化一个Scriptable Object脚本之后，我们就可以发现实例化后的Scriptable Object脚本的图标变了。

![image](/assets/images/2024-02-09/1369799-20240209010457876-1805397796.jpg)

我们点击这个实例化后的Scriptable Object Asset文件，在Inspector中我们可以看到在Scriptable Object脚本中声明的public变量，同时我们还可以在Inspect面板中对其进行数据编辑，这个时候我们编辑的数据会被直接以Asset的文件形式保存在磁盘中，同时我们在开始游戏后的对其数据的更改会被永久保存。根据这个特性，我们可以使用Scriptable Object来实现一个简单的游戏存档功能。当然，对于一些不希望被的改变的数据来说，将其设置为私有变量，再通过公有变量进行GetSet操作，或者直接设置为只读类型的变量都可以防止数据被修改。

对于一个Scriptable Object实例，我们可以在其他脚本中访问它，只需要声明一个ScriptableObject类型的变量，再在editor的inspector面板中将我们需要的ScriptableObject实例拖拽赋值就可以在脚本中访问这个ScriptableObject实例了。

![image](/assets/images/2024-02-09/1369799-20240209010521279-1315922105.jpg)
![image](/assets/images/2024-02-09/1369799-20240209010537358-1822701378.jpg)

值得注意的是，我们声明时使用的变量名称是我们ScriptableObject模板的类名。

有些时候我们希望每一个在运行时实例化的GameObject都能有一个ScriptableObject来记录自身变量的值，那么这个时候就不能依靠拖拽赋值的方法了，并且如果我们将ScriptableObject实例设置到Prefab中，那么这个Prefab的实例所访问的ScriptableObject其实都来自于同一个ScriptableObject实例。所以针对这个问题，我们可以使用Instantiate方法(或者直接使用ScriptableObject.CreateInstance)来实例化一个ScriptableObject实例，这样我们在不同的实例当中对ScriptableObject的操作都只是其实例化的副本，使用并不会对原始数据产生修改，并且每个实例都有自己的一个ScriptableObject实例了。（最后记得要有Destroy的好习惯）

![image](/assets/images/2024-02-09/1369799-20240209010608775-1109488602.jpg)

## Scriptable Object常用用法之enum
我们在使用enum的时候通常会遇到的问题是，对于enum的修改往往牵扯到很多部分的代码也需要修改，并且enum不支持记录更多的数据。而ScriptableObject则可以在当做是enum使用的情况下记录额外的信息。更多的信息请看[Scriptable Object 1 | Enum | Unity3D开放项目](https://www.bilibili.com/video/BV1nh411W7vw/?spm_id_from=333.788&vd_source=32fcf9c880efa4159dc50a351d50a253 "Scriptable Object 1 | Enum | Unity3D开放项目")

## Scriptable Object常用用法之Data
使用ScriptableObject来分离存储数据再适合不过了，假如一个player有几个字段记录了当前血量、最大血量、攻击力等等数据，一般来讲我们都会把这几个字段都写到MonoBehavior里，如果游戏当中我们想让UI反映玩家的血量情况我们应该怎么做？我们也许会选择让玩家保有一个UI的引用，在玩家血量发生修改时通知UI进行修改，或者在UI当中保有玩家的引用，当检测到玩家的血量发生修改的时候就更新UI的状态，又或者我们不希望玩家和UI有这种耦合关系，我们会使用一个Manager脚本来管理二者之间的行为，但是这样会徒增工作量并且加大了项目的复杂程度，项目小的时候这种缺点还不明显，但是一旦项目体积开始变大后这种缺点就很明显了。

所以，这个时候我们可以合理使用ScriptableObject来将一些需要在不同脚本之间传递的数据进行统一管理，这个时候我们就抽象出了数据层，再让玩家和UI保有这个ScriptableObject的引用，玩家修改ScriptableObject中的值后，UI也能够知道值被修改了从而做出修改，但此时玩家和UI双方都不知道对方的存在，就实现了解耦。并且，此时写在ScriptableObject中的数据是可以在Inspector面板中修改的，相比于在场景Hierarchy面板中找到对象再修改的方式，这无疑要直观方便的多，特别是项目复杂了过后。

当然，上述情景是一个最简单的应用场景，对于一些稍微复杂一点的项目来说，ScriptableObject也有一定的局限性，所以要充分分析项目的需求，在合适的地方使用ScriptableObject。更多的信息请看[Scriptable Object 2 | Data | Unity3D开放项目](https://www.bilibili.com/video/BV1A34y1Q7Zp/?spm_id_from=333.788&vd_source=32fcf9c880efa4159dc50a351d50a253 "Scriptable Object 2 | Data | Unity3D开放项目") 

## Scriptable Object常用用法之Runtime Set
RuntimeSet也就是运行时集，比如在游戏运行时游戏内的所有敌人就是一个运行时集，有些情况下我们需要得到这个RuntimeSet来对所有集合内的object做统一的处理，那么ScriptableObject就是运来存放运行时集RuntimeSet的最佳选项，我们只需要让需要的object在被生成时将自己注册进存放进ScriptableObject所做的RuntimeSet中，就可以在任何地方访问到这个RuntimeSet，当然前提就是我们需要在代码中获得这个ScriptableObject的引用即可。更多信息请看[Scriptable Object 5 | Runtime Set | Unity开放项目](https://www.bilibili.com/video/BV1mf4y1w7BY/?spm_id_from=333.788&vd_source=32fcf9c880efa4159dc50a351d50a253 "Scriptable Object 5 | Runtime Set | Unity开放项目")

## Scriptable Object常用用法之Editor拓展
[Scriptable Object 3 | Editor扩展 | Unity3D开放项目](https://www.bilibili.com/video/BV16q4y1N7Ur/?spm_id_from=333.788&vd_source=32fcf9c880efa4159dc50a351d50a253 "Scriptable Object 3 | Editor扩展 | Unity3D开放项目")

## END
最后再说一点，ScriptableObject最终最合适的用法还是作为一个数据容器来使用，我们不能指望ScriptableObject能够做到更多。ScriptableObject能够帮助我们解决一些代码之间的耦合，独立出一些数据和逻辑存放在ScriptableObject中，就可以变成可拔插式的SO文件，随便组合，就能组装出较为灵活配置的AI系统。

ScriptableObject不能用做数据文件最终存储的载体