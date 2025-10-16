---
title: Unity Animation动画系统
date: 2023-02-05 14:31:00 +0800
categories: [Unity]
tags: [Unity,Animation]
author: HalfDog
---

## 一、Unity Animation动画系统基本介绍

unity提供了一套非常强大灵活且成熟的动画系统，不论是2d还是3d动画都有相应的组件和接口提供给开发者使用,不过这篇文章主要还是讲解3D部分的动画系统。我们在游戏开发时时常需要角色动起来，除了位置上的移动之外，我们还需要匹配角色的行为或玩家的输入播放不同的动画，不同的动画之间又需要平滑的衔接，有时我们还需要将两个动画混合、覆盖，动画复用等等，以上提到的种种操作在unity中都可以通过几个unity的组件实现：Animation Clip、Animator Controller、Avatar。

笼统的介绍一下这三个组件，Animation Clip顾名思义是一个个动画片段，如何理解动画片段呢，在unity中，动画片段记录了对应的模型上各个部分例如左大臂、左手小拇指的第二个指节或者一些简单的物体如门、窗的位置、旋转、缩放数据，而我们unity中的动画就是模型中各个部分的位置、旋转、缩放随时间变换组成的。

Animator Controller则像是可以自定义规则的容器，我们将某个角色要的Animation Clip放入Animator Controller中，这个时候我们就可以按照我们的需要来规定每个Animation Clip应该是什么时候以什么方式播放。一句话，Animator Controller不生产动画，它只是动画的搬运工，按照我们的规则将Animation Clip在合适的时间以合适的方式在模型上展现出来。

而这个Avatar就比较特殊了，Avatar所针对的只是人形的动画，并且Avatar起到的是一个翻译官的作用，他被用来在两个动画骨骼不相同的模型之间匹配动画以达到复用动画的作用，其原理笼统的来讲，就是unity自身提供了一套标准的蒙皮动画骨骼，Avatar将A模型的动画以unity标准的格式记录下来然后在以Avatar为骨骼操纵B模型播放A模型的动画。接下来我们就系统的讲解一下这三个unity的动画系统组件。

## 二、Animation Clip：动画切片

Animation Clip动画切片是Unity动画的原材料，任何unity动画都是一个或多个动画片段拼接组合起来的。而Animation Clip可以在Unity编辑器内使用Animation Window创建，也可以从外部导入如导入FBX。如果需要的话，从外部导入的单个动画切片也可以通过切分再次分为几个不同的动画切片。

![image](/assets/images/2023-02-05/1369799-20240205105339077-1926062350.png)

上图为从外部导入的Animation Clip的inspector视图

需要注意的是，从外部导入的动画不能在unity的animation window 中编辑，但可以在animation window中查看，也就是外部导入的animation数据是只读的（read only）。更多的关于这个界面的介绍参考官方文档

就如概述中所说，Animation Clip记录的是对应物体各个部分的位置、旋转、缩放随时间变换的信息。

![image](/assets/images/2023-02-05/1369799-20240205105631949-551729769.png)

我们拿上面这个在Unity中制作的Box Bounce动画来举例子，可以看到，这个动画片段包含了位置、旋转、缩放的信息，在Animation视图中这些信息以动画曲线的方式展现出来，当然还有以关键帧的形式展现,但最终的动画时依照动画曲线来运动的。当我们播放这个动画片段的时候，这个box就会按照我们的曲线在不同时间段的值来改变相应的数值从而让我们的Box动起来。

这个时候我们就要问：只有这个box能播放这个动画片段吗？既然我都这样说了，那么说明其它的模型也是可以播放这个动画片段的。为什么？我们再次看这一段话“Animation Clip记录了gameobject及其子物体的位置、旋转、缩放随时间变换的信息”，是的，所有拥有transform组件的gameobject都可以播放这个动画片段。当然也不是所有的动画片段都是这样，上述说明的只是其中最简单的一种情况，一些拥有子物体动画的Animation Clip就不一定能够在其他gameobject上播放了，再比如骨骼动画就不能，一个骨骼动画只能有对应骨骼的模型才能播放，这一点也涉及到了之后我们要讲的Avatar组件，后面我们在解释为什么，这里暂且按下不表，让我们先再深入一点了解animation clip文件。

Animation Clip究竟是以何种方式记录动画信息的呢？让我们看一张截图：

![image](/assets/images/2023-02-05/1369799-20240205105805470-1818034574.png)

上面这个截图就是记录了Box Bounce动画的Animation Clip文件，文件以anim后缀存储，用标记语言YAML编写。让我们仔细看一下这个文件的第九行，有一个字段叫m_Name，值为box_Bouncing正是unity中显示的animation clip的名称。再往下看，第16行、78行、185行，分别为m_EulerCurves、m_PositionCurves、m_ScaleCurves，显然这几个字段就记录了动画的旋转、位置、缩放信息，并且从m_EulerCurves字段下的m_Curves的六个子字段可以看出其正是对应了旋转动画的六个关键帧。值得一提的是，所有unity动画的信息都是以曲线的方式记录的。这也不难理解为什么其他的gameobject也可以播放这个动画片段。

![image](/assets/images/2023-02-05/1369799-20240205105906195-1499287003.png)

上图就是一个人类模型正在播放boxBounce的动画，模型在这一刻依照动画曲线的值被压缩了。

可以说，boxBounce这个动画片段可以用在任何gameobject上，但不是所有的Animation Clip都能够在任何gameobject上播放，正如我们前面所说的，一些拥有子物体动画的Animation Clip就不能在任意的gameobject上播放了，下面让我们看一个双开门开门的动画，这个取名为door的gameobject拥有两个子物体分别是左边的门left和右边的门right。

![image](/assets/images/2023-02-05/1369799-20240205110025392-1511566092.png)

在Animation View的曲线视图中我们可以看到两个子物体left和right都有对应的动画曲线也就是两个子物体都有动画，当我们想在其他的gameobject上比如我们之前使用的Box上使用这个Animation Clip的时候就会发现没有任何反应，问题出在哪里？很显然box这个gameobject并没有任何的子物体，也自然不能播放这个动画片段，那么现在我们在创建一个与这个双开门层级结构相同的gameobject，我们把两个box放下一个空gameobject下，一个取名CubeLeft一个取名CubeRight，然后我们再让这个gameobject播放双开门的动画试一下。

![image](/assets/images/2023-02-05/1369799-20240205110103441-1466995976.png)

可以很直观的看到，这个gameobject没有成功播放我们双开门的动画，即使它的层级结构是一样的，问题出在哪里呢，我们看Animation View中那些发黄的字段后面有”Missing“的提示，这表示unity没有找到left和right的信息，现在我们有眉目了，如果我们把两个子物体的名称改成一样的试试看呢？

![image](/assets/images/2023-02-05/1369799-20240205110203426-469458474.png)

非常好，当我们把两个box的名称改成双开门动画切片所记录的left和right后动画被播放了，现在我们可以说，只要父子级层级结构和名称相同，动画片段就可以被复用。其中的原因让我们在记录动画片段数据的anim文件中寻找。我们同时打开boxBounce和doorOpen的动画片段文件作比较。

![image](/assets/images/2023-02-05/1369799-20240205110312270-335594976.png)

可以看到在第16行记录旋转信息的m_EulerCurves字段，box有一个curves字段而door有两个，这很好理解，因为door有两个子物体，两个子物体都分别有旋转动画，并且door的position和scale都没有动画所以两个记录位置信息和缩放信息的m_PositionCurves和m_ScalerCurves字段都没有curves信息。现在让我们展开m_EulerCurves字段的第一个Curves字段再做比较。

![image](/assets/images/2023-02-05/1369799-20240205110358970-1854070025.png)

可以看到，因为两个动画的关键帧不同，m_Curves字段记录的数量也不同，对应于box动画的6个关键帧，和door动画的2个关键帧，具体的曲线数据被记录在serializedVersion字段下，这里我们不展开讨论。除了关键帧数量不同外，我们还可以看到最后一个path字段，这里正是动画能否被正确播放的关键，box动画中的path字段没有记录信息，而door动画中的path字段记录了一个left信息，这里我们就可以大胆猜测，这里的path信息就记录了当前的curves所记录的动画信息是属于哪一个子物体的，让我们来进一步验证一下。

![image](/assets/images/2023-02-05/1369799-20240205110442981-1954645849.png)

我们构造了这样一个gameobject，这个gameobject包含两层父子级结构，e是b的子物体，b是y的子物体，在动画编辑器中我们给e这个孙级添加了动画，现在让我们看看在anim的文件当中是否也记录了这样的层级关系以用来确定动画数据的归属。

![image](/assets/images/2023-02-05/1369799-20240205110553894-267731510.png)

正如我们所想的那样，path这个字段就记录了这样的层级关系，这告诉unity“上述动画数据是属于子物体b下的子物体e的”，这样动画就能被正确播放了。同理，任何一个gameobject只要有符合这样的层级关系并且名称也相同，那么这个gameobject就可以正确播放这个Animation Clip了。

现在我们反过来想一想path字段为空的情况意味着什么，意味着当前动画数据就是属于这个物体本身的，拿boxbounce的动画来说，为什么其能够在所有的gameobject上被播放？因为不论是有子级还是没有子级或者本身就是子级等等情况都不重要，path字段为空就意味着，当前播放的这个动画只会影响到自身，而其子物体也会随着父级物体变换而变换。

那么到这里我们就对Animation Clip从表面到底层有了一个粗浅的了解，还有一些内容我们放到之后讲Animator Controller和Avatar的时候在讲，接下来我们进入unity动画系统的重中之重，Animator Controller动画控制器。

## 三、Animator Controller：动画控制器

Animator Controller的作用是将一堆Animation Clip用动画状态机（Animation State Machine）的方式组织起来，按照我们设定的条件、方式去选择、处理并播放动画切片。Animation Controller的核心之处就在于动画状态机，其通过类似于流程图的方式管理动画在各个状态之间切换。同时Animation Controller文件以controller为后缀同样是用标记语言YAML所编写。

![image](/assets/images/2023-02-05/1369799-20240205111055800-451987304.png)

上图就是Unity的Animator Controller的界面。

Animator Controller的界面分为了两个主要部分，一个是左边的图层（Layers）和参数（Parameters），一个是右边的可视化的动画状态机编辑界面。我们先从动画状态机讲起。

动画状态机也是State Machine（状态机）的一种，状态机的一些概念我们暂且不提，这里我们就了解一下unity动画状态机的构成部分，包括一个状态集合（Animation Clip集合），当前状态（当前播放的Animation Clip），和各个状态之间的转换关系和转换条件的集合（Transitions and Parameters）。

Animation Parameters动画参数，这个是控制Animation State Machine从一个State转移到另一个State的关键，几乎所有对Animation State Machine 的当前状态任何的修改都要通过改变Animation Parameter的值来达到目的。Animation Parameter可以在Animator 的Parameters面板中定义，一共有四种参数类型，int、float、bool、trigger，并且可以通过脚本在代码中访问并修改Parameter的值。

![image](/assets/images/2023-02-05/1369799-20240205111711303-1755577122.png)

实现状态之间的转换除了转换的条件之外unity还需要知道转换应该从哪里开始从哪里结束，这个时候就可以通过State Machine Transitions（状态机过渡）来表示状态之间的转换关系，在Animator Controller的可视化状态机视图中，这种关系以单箭头的方式表现出来。

![image](/assets/images/2023-02-05/1369799-20240205111742595-1546861935.png)

由状态机过渡链接的两个状态在满足转换条件时会依照箭头的朝向从一个状态转移到另一个相连接的状态，如果没有任何的转换条件，当上一个动画播放完成时会自动向所链接的下一个状态转换，并且，转换可以不是瞬时发生的，在Transition的设置中我们可以设置两个动画转换时平滑的混合过渡和转化发生的时间等等。

![image](/assets/images/2023-02-05/1369799-20240205111821964-54426879.png)

上图就是Transition的Inspector面板，在最下方我们可以看到添加转换条件的选线Conditions。

现在让我们回到动画状态机的“状态”上，在动画状态机中，状态一词指代的是一个动画片段，但其实在unity的动画状态机中，状态可以为空，也就是这个状态上没有动画片段，说到底，状态和最后呈现的动画没有必然联系，相同的动画状态机可以有不同的动画表现，这完全取决于开发者的设置，这样状态机相同，但表现的动画不同的设计可以实现一种动作的不同状态，例如正常的奔跑和受伤后的奔跑，都是处于奔跑的状态但是表现出的动画确实不同的，unity中也提供了让这一设想实现的功能，Animation Layers。

![image](/assets/images/2023-02-05/1369799-20240205111915664-404590677.png)

我们可以在Animator Controller的Layers面板中新增Animation Layer。

当然，Animation Layers不止有这一个功能，其更多被用来实现动画之间的组合，具体来讲就是两个不同的Animation Layer上的动画进行添加（additive）或覆盖（override），具体操作的部分可以通过Mask来确定。

![image](/assets/images/2023-02-05/1369799-20240205112003790-1468953940.png)

![image](/assets/images/2023-02-05/1369799-20240205112011339-853657734.png)

如上图，Mask接受一个Avatar，由Avatar给出要添加或覆盖的部分，使用了Avatar也就默认只能混合人形动画了。unity会根据Avatar给出的部分去覆盖优先级较低的layer上的动画，而layer的优先级由在视图中的高低排列顺序决定。

![image](/assets/images/2023-02-05/1369799-20240205120836652-14170546.png)

Layer越靠后优先级越高，如上图所示，Layer1的优先级大于Layer0。

需要注意的是，当有多个layer存在时，优先级高的layer的动画会始终依照我们的设置覆盖或叠加低优先级layer的动画，如果我们想控制动画覆盖或叠加的时机，我们可以通过在高优先的layer的enter状态和实际动画状态之间添加一个没有任何动画片段的空状态来避免对低优先级layer动画的叠加或覆盖，这样我们就可以手动控制只在我们需要的时候让动画叠加或覆盖。

![image](/assets/images/2023-02-05/1369799-20240205121135576-48110102.png)

如上图，Layer1的empty状态的动画片段为空，当Layer1的动画状态处于empty时，是不会对Layer0的动画产生任何影响的，只有layer1处于animation状态时才会对layer0产生影响，但这个过程是受我们控制的。

Animation Layer已经讲了大部分了，其中还有一些例如在开头提到的状态机相同但动画不同的效果我们可以通过Animation Layer syncing来实现。更具体的信息可以参考unity官方文档和观看[这个视频](https://b23.tv/p19YgwE "这个视频")

接下来我们要讲一讲子状态机（Sub-State Machine）和混合树（Blend Tree）。

子状态机很好理解，可以想象成将几个状态折叠成一个状态。在transition连接到或从子状态机连接出去的时候也是要确定要连接到子状态机当中的哪一个状态，或由子状态机中哪个状态连接出来。

![image](/assets/images/2023-02-05/1369799-20240205121737766-1308866841.png)

![image](/assets/images/2023-02-05/1369799-20240205121744580-220603352.png)

子状态机中的“（up）Base Layer”就是上一层的动画状态机的抽象表示，在子状态机连接出去时，也会让我们选择要连接的上一层动画状态机的状态。如下图所示。

![image](/assets/images/2023-02-05/1369799-20240205121804515-1175355880.png)

接下来我们来聊一聊Blend Tree混合树(内容太多了 直接看视频和官方文档吧)
不过总的来说就是在不同动画之间做过渡混合。

## 四、Animator组件

如果一个gameobject想要播放相应的动画，就要挂载Animator组件来执行我们已经准备好的Animator Controller和Avatar的动画和功能。Animator组件相较于Animation Clip和Animator Controller这些偏理论的部分，Animator组件就偏实际应用方面，在这里我们就一个个过一遍Animator组件上的各个属性。

![image](/assets/images/2023-02-05/1369799-20240205121859842-575653465.png)

Controller：这里放置需要的Animator Controller。

Avatar：这里放置已经设置好的对应模型的Avatar，如果骨骼动画是通过Avatar进行传递的话，这里缺失Avatar模型动画将不能被正常播放。
Apply Root Motion：是否应用Root Motion根运动。

Updata Mode：动画更新的模式。也就是计算动画变换的模式。

Normal：更新频率与updata同步，也就是帧率有多少帧，动画就计算那么多次。

Animate Physics：更新频率与FixedUpdata同步，即与Unity的物理引擎同步，没做一次碰撞检测动画就计算一次更新。

Unscaled Time：与Normal模式相同，但不同的地方在于，这个模式下会忽略Time Scale时间缩放。

Culling Mode：剔除模式，类似于渲染当中的剔除，当动画处于摄像机视图外时进行剔除处理。

Always Animate：不参与剔除，无论是否在摄像机视图内。

Cull Updata Transform：同样是动画不参与剔除，但是ik会在动画处于摄像机视图外时被禁用。

Cull Completely：完全参与剔除，即动画在摄像机视图外时就停止。

除上述属性外，我们还可以看到在最下方还有一些关于当前Animator组件的信息，具体信息可以参看官方文档。
