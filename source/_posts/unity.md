---
title: Unity笔记从0-1
date: 2024-12-08 11:48:47
tags:
---
# 画面优化

## 摄像机渲染原理

Unity摄像机渲染原理主要包括以下几个关键步骤：

1. **摄像机空间到世界空间的转换**：摄像机相对渲染的工作原理是在任何其他几何变换影响游戏对象和光源之前，通过取反世界空间摄像机位置来移动游戏对象和光源。

2. **几何变换**：这一步骤涉及到对游戏世界中的所有物体进行变换，包括平移、旋转和缩放等，以便将其放置在摄像机坐标系中。

3. **裁剪和投影**：**在这一步中，场景中不在摄像机视锥体内的物体会被裁剪掉，剩下的物体会被投影到一个投影面上。**（所以并不是你以前想的那样，整个场景都要渲染出来）

4. **光栅化**：将二维图像或者3D的投影图转换为屏幕上的像素点。

5. **渲染遍历**：在这个阶段，对每一个物体执行更详细的渲染工作，包括材质应用、光照计算和阴影处理。

6. **后期处理**：可能包括模糊、色彩校正等，最后生成最终的图像。



## 如何优化Unity中的渲染性能？

在Unity中优化渲染性能可以通过多个方面进行改进，以下是一些主要的优化策略：

### 1. **减少绘制调用 (Draw Calls)**

   - **合并网格**：将多个小网格合并为一个大网格，以减少绘制调用的数量。
   - **使用合批 (Static Batching / Dynamic Batching)**：**Static Batching 是在场景加载时合并静态物体的网格**，**Dynamic Batching 则是在运行时合并小的动态物体。**

### 2. **使用LOD（Level of Detail）**

   - 为不同距离的物体创建不同的细节层次，远处使用低多边形模型，以减少渲染开销。

### 3. **优化材质和纹理**

   - **纹理压缩**：使用合适的纹理压缩格式，减少显存的占用。
   - **减少贴图数量**：尽量使用单张大纹理（比如图集）替代多张小纹理，减少纹理切换。

### 4. **利用遮挡剔除**

   - 使用遮挡剔除技术，仅渲染在摄像机视野内的物体，未被视野遮挡的物体将不会被渲染。

#### 前言

unity在渲染时，默认只是对模型进行视椎体剔除，也就是在相机显示范围内的物体进行剔除，而遮挡剔除则是，渲染物体被整个遮挡住后，将不参与此帧的渲染，unity虽然内置，但是不默认启用，需要我们进行一些操作，才能够实现当前的操作。

##### 一、遮挡剔除是什么？

Unity 中的遮挡剔除（Occlusion Culling）是一种[性能优化](https://so.csdn.net/so/search?q=性能优化&spm=1001.2101.3001.7020)技术，它可以帮助开发者减少需要渲染的场景物体数量，从而提高游戏的帧率和流畅度。
遮挡剔除的基本思路是在运行时计算场景中哪些物体被遮挡而不需要被渲染，哪些物体是可见的需要被渲染。这样可以减少渲染所需的时间和开销，提高游戏性能。

Unity 中的遮挡剔除主要有两种方式：静态遮挡剔除和动态遮挡剔除。
**静态遮挡剔除**（Static Occlusion Culling）是在场景构建时进行的，主要是通过 Unity 自带的预处理工具将场景物体分成一些区域，然后计算这些区域之间的遮挡关系。这种方式适用于静态场景和场景中的大部分物体都是静态的情况。静态遮挡剔除的优点是计算量小，不会对游戏运行时的性能造成太大影响。
**动态遮挡剔除**（Dynamic Occlusion Culling）则是在游戏运行时进行的，主要是通过摄像机视野和场景中物体之间的遮挡关系来计算需要渲染的物体。这种方式适用于动态场景和场景中有大量动态物体的情况。**动态遮挡剔除的优点是可以适应动态变化的场景，但需要计算量较大，可能会对游戏运行时的性能造成一定影响。**

##### 二、静态遮挡剔除的使用步骤

###### 1.标记为遮挡剔除对象

同时勾选Occluder Static和Occludee Static 。
Occluder Static 属于静态遮挡物体，设置后，可以遮挡其它物体。
Occludee Static 属于静态被遮挡物体，设置后，可以被其它遮挡物体遮挡。
![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\55a69193c003c99559ad75282b203fbc-172870343660423.png)

###### 2.创建Occlusion Area组件

1.Window --Rendering–Occlusion Culling 打开遮挡剔除面板
![![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c38fe3dc8c3b4e319374eb1eae6c8fb3.png](D:\DeskTop\FileSum\UnityClassResources\笔记\823ffbc3497006fafd4af5fe530072c1-172870343528921.png)
2.创建Occlusion Area组件
选择到Object，再选择Occlusion Areas，最后点击最下面的Occlusion Areas，创建Occlusion Areas。步骤如下图：
![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\ddf9d688fecfe591fd45b374bd88668f.png)
3.创建成功
![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\ee20a319334bc726c8f50bde882be469.png)
4.当然也可以创建一个空物体，添加组件Occlusion Area，结果和上面步骤一样。
![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\90af7274a0f119daa41d35f806f3beb5.png)

###### 3.烘焙

选择Bake，点击下方Bake。
![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\d721d5659b77ed3f714f13ea0b84cbc3.png)

###### 4.Occlusion窗口Bake的参数

![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\96a4e774cc8f6b8fee6bb6fcbdd1211c.png)

**Smallest Occluder**

场景内最小遮挡物的尺寸，设得过大会导致剔除成功率下降，过小会导致性能问题。一般默认就好。

**Smallest Hole**

如果场景中有带孔的物体需要能被视线穿透(例如墙上的洞, 房间的门)，那么需要将Smallest Hole设置为小于孔的直径
一般默认即可。

**Backface threshold**

本参数的引入是为了减少剔除数据大小，另一方面，设置不当会导致剔除错误（可见的物体被剔除了）。因此，暂时请保持默认值100不变。
工作机制是如果PVS产生的某个cell中观察到的阻挡面是backface的比例大于设定值，那么生成的剔除数据中将不会包含这个cell相关内容，从而降低了数据大小。如果运行时camera移动到该cell内，那么剔除查询结果将会是“Undefined”。

###### 5.遮挡剔除前后的效果对比

**没有开启遮挡剔除前：**

![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\82eca806009cbed1b6fca009514f52da.png)

**开启遮挡剔除后**

![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\20f8bd825ad3c6dd4d1056fb46a4ea85.png)
可以很明显看见 三个Cube没有被渲染了，相机发射的绿色范围就是渲染到的范围。

##### 三、动态遮挡剔除的使用步骤

###### 2.设置动态遮挡剔除

**1.开启Dynamic Occlusion**

对于动态或者可移动的物体，如果需要被遮挡，那么需要在其Mesh Renderer 或者 Skinned Mesh Renderer上面设置即可
![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\3cf0cf1f170ae5ae0dd546ac98f72975.png)

**2.挂载Occlusion Portal脚**

动态加载的物体，要能够遮挡其它物体，需要挂载Occlusion Portal脚本进行实现，添加这个组件的物体必须取消Occluder Static和Occludee Static。
Open勾选则是不启用遮挡剔除，不勾选则是启用遮挡剔除，可以通过代码控制。
![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\cc3813b7ea2f9622035121ca4a4903f7.png)

**3.烘培**

选择Bake，点击下方Bake。
烘培和静态剔除的步骤一样。

##### 四、注意点

1.静态剔除的物体，无法移动
2.如果修改了场景，需要Clear，然后重新Bake，才会生效

##### 总结

好记性不如烂笔头！

### 5. **简化光照计算**

   - **使用实时光照的最小数量**：尽量减少实时光源的数量，多使用烘焙光照。
   - **使用光照探针和反射探针**：可以减少需要实时计算的光照效果。

#### 光照探针

##### 优化原理

**预计算全局光照‌：使用光照贴图和[光照探针](https://www.baidu.com/s?sa=re_dqa_generate&wd=光照探针&rsv_pq=bbe397c2000627a7&oq=unity不在摄像机范围内的物体会被渲染吗&rsv_t=58a2PO2H/u+VfhosV0wpv2ce28lWDK2VVTq6MqbvhNFArShvlW37LrnSvvkdDFc1YFdDXJORSv9B&tn=39042058_26_oem_dg&ie=utf-8)等技术，优化动态物体的光照计算‌**

> **[Unity光照探针](https://www.baidu.com/s?sa=re_dqa_generate&wd=Unity光照探针&rsv_pq=a8e9edcf000ce6b9&oq=unity光照探针的优化原理&rsv_t=f87bcoJ+Oavetwdt9/qdhF4G3iIKrpw3bDinGd6vsSvktBLsHQrNpDdrCVuzRHJ+TmjxSUCE5KD5&tn=39042058_26_oem_dg&ie=utf-8)的优化原理主要是通过预计算和编码技术，利用[球谐函数](https://www.baidu.com/s?sa=re_dqa_generate&wd=球谐函数&rsv_pq=a8e9edcf000ce6b9&oq=unity光照探针的优化原理&rsv_t=f87bcoJ+Oavetwdt9/qdhF4G3iIKrpw3bDinGd6vsSvktBLsHQrNpDdrCVuzRHJ+TmjxSUCE5KD5&tn=39042058_26_oem_dg&ie=utf-8)来记录和还原场景中的光照信息，从而在运行时高效地模拟物体的光照效果。**‌
>
> 光照探针技术通过在3D空间中放置探针来接收光照信息，这些信息随后被编码成球谐函数的系数。这些系数占用的存储空间很小，并且在运行时能够快速解码，使得Shader可以存取并计算表面光照‌12。球谐函数是一种在球坐标系上定义的函数，通过几个简单的基函数和它们的系数来还原复杂的光照信息。这种方法的优势在于能够有效地减少内存消耗，同时保持较高的渲染效率‌24。
>
> 在Unity中，光照探针技术通过记录场景中每个探针的位置和方向上的光照信息，然后利用球谐函数对这些信息进行编码。运行时，通过插值技术（如四面体插值）来还原这些光照信息，从而实现高效的光照模拟‌56。这种技术特别适用于动态物体和场景中的间接光照模拟，虽然它无法完全模拟复杂的光照变化，但在大多数情况下能够提供足够真实的光照效果‌13。
>
> 然而，光照探针技术也存在一些限制。例如，在不增加探针数量的情况下，难以表现高频或斑驳的光照效果。此外，增加模拟精度会提高计算代价，因此Unity限制了计算的精度。对于大模型或平坦表面，光照效果可能不够理想。总体而言，光照探针技术在小的、凸状的物体上表现较好，同时能够提供良好的性能‌13。

要将光照探针置于场景中，必须使用已附加__光照探针组 (Light Probe Group)__ 组件的游戏对象。可从菜单 **Component > Rendering > Light Probe Group** 添加“光照探针组”组件。

可将“光照探针组”组件添加到场景中的任何对象，但最好将其添加到新的空游戏对象。

![光照探针组 (Light Probe Group) 组件](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-7.png)光照探针组 (Light Probe Group) 组件

光照探针组有自己的__编辑模式 (Edit Mode)**，可开启或关闭此模式。要添加、移动或删除光照探针，必须按** Edit Light Probes__ 按钮来开启光照探针组的编辑模式：

![Edit Light Probes 按钮](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-8.png)Edit Light Probes 按钮

使用编辑光照探针 (Edit Light Probes) 模式时，可按照与游戏对象类似的方式来操作单个光照探针，但是单个探针不是游戏对象，它们作为一组点存储在“光照探针组”组件中。

开始编辑新的光照探针组时，将从一个立方体中排列的八个探针的默认编队开始，如下所示：

![光照探针的默认排列。](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-9.png)光照探针的默认排列。

现在可以使用光照探针组 (Light Probe Group) 检视面板中的控件向该组添加新的探针位置。探针在场景中显示为黄色球体，这些球体可按照与游戏对象相同的方式定位。还可以使用常规的“复制”键盘快捷键 (ctrl+d/cmd+d) 来选择和复制单个探针或按组进行这些操作。

完成探针的编辑后，__关闭__光照探针组编辑模式，否则无法移动或操作常规对象！

##### 选择光照探针位置

与通常在对象表面具有连续分辨率的光照贴图不同，光照探针信息的分辨率完全由放置探针的紧密程度决定。

为了优化光照探针存储的数据量以及游戏在进行时完成的计算量，通常应尝试尽可能少量放置光照探针。但是，在可以接受的范围内，建议放置足够多的探针记录一个空间到另一空间的光线变化。这意味着，在具有复杂或高对比度光线的区域周围可提高光照探针的放置密度，而在光线没有显著变化的区域可降低它们的放置密度。

![在简单场景周围放置不同密度的光照探针](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-10.png)在简单场景周围放置不同密度的光照探针

在以上示例中，建筑物附近和之间的对比度和颜色变化较大，因此探针放置得较密集，而沿着道路的光照没有显著变化，因此放置密度较小。

定位探针的最简单方法是将它们排列成常规 3D 网格图案。虽然此设置简单有效，但可能会消耗不必要的内存，并可能有大量冗余的探针。例如，在以上场景中，如果沿着道路放置大量探针，将会浪费资源。光照沿着道路的长度不会发生太大变化，因此许多探针将大多数相同的光照数据存储到相邻探针中。在此类情况下，通过在稀疏探针之间插入此光照数据，效率会高得多。

光照探针不会单独存储大量信息。从技术角度来看，每个探针都是来自采样点的球形全景 HDR 图像，使用存储为 27 个浮点值的球谐函数 L2 进行编码。然而，在具有数百个探针的大型场景中，这些探针可以累加，而设置密集度过高的探针可能导致游戏中大量内存浪费。

##### 创建体积

即使游戏位于 2D 平面上（例如，在路面上行驶的汽车），光照探针也必须形成 3D 体积。

这意味着探针组中至少应有两个垂直“层”的点。

在以下示例中，在左侧看到探针仅排列在地面上。因为光照探针系统无法从探针计算明显的四面体体积，所以这种排列方式__不会产生良好光照__。

在右侧，探针排列为两层，一些低至地面，另一些高于地面，因此它们一起形成由大量单个四面体组成的一个 3D 体积。这是一种__良好__的布局。

![左图显示了光照探针位置的选择不当，因为探针定义的体积没有高度。右图显示了光照探针位置的选择较恰当。](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-11.jpg)左图显示了光照探针位置的选择不当，因为探针定义的体积没有高度。右图显示了光照探针位置的选择较恰当。

##### 放置光照探针以实现动态 GI

Unity 的实时 GI 允许移动光源针对__静态__景物投射动态反射光。但是，使用光照探针时，还可从__移动游戏对象__上的移动光源接收动态反射光。

因此，光照探针执行两个非常相似但不同的功能：它们存储静态烘焙光源，并在运行时代表动态实时全局光照（GI 或反射光）的采样点来影响移动对象上的光照。

所以，如果要使用动态移动光源，并希望移动游戏对象上有实时反射光，这可能会影响放置光照探针的位置以及将它们分组的密集程度。

在这种情况下要考虑的要点是，对于大面积范围内相对不变的*静态光源*，因为光照不会在大面积范围内变化，只需放置少数几个探针。但是，如果计划在此区域内使用*移动光源*，并且希望移动对象用来接收反射光，则需要在该区域内建立更密集的光照探针网络，以便有足够高的准确度来匹配光源的范围和样式。

放置探针需要的密集程度因光源的大小和范围、其移动速度以及希望接收反射光的移动对象的大小而异。

##### 光照探针放置问题

对光照探针位置的选择必须考虑到光照将在探针组之间进行插值的情况。如果探针不能充分适应场景中光照的变化，可能会出现问题。

以下示例显示了一个夜间场景，两侧有两个明亮街灯，中间有一个暗区。如果光照探针仅放置在街灯附近，而在暗区中没有放置，则街灯的光照将跨越黑暗间隙“渗透”到移动对象上。这是因为光照会从一个亮点插入到另一亮点，而没有关于暗区的信息。

![此图像显示了不佳的光照探针放置。两个街灯之间的暗区内没有探针，因此暗区根本不会包含在插值中。](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-12.jpg)此图像显示了不佳的光照探针放置。两个街灯之间的暗区内没有探针，因此暗区根本不会包含在插值中。

如果使用实时或混合光源，此问题可能不太明显，因为只有_间接_光才会跨越间隙渗透。如果使用完全烘焙光源，问题会更明显，因为此情况下也会从光照探针插入移动对象上的直射光。在此示例场景中会烘焙两个街灯，因此移动对象从光照探针获得直射光。在下图中，可看到结果：一个移动对象（救护车）在穿过暗区时仍然获得明亮光照，这不是想要的效果。黄色线框四面体显示出，插值发生在街道明亮光照的一端与另一端之间。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-13.jpg)

这是不想要的效果：救护车在穿过暗区时仍然获得明亮光照，因为在暗区中没有放置光照探针。

要解决此问题，应该在暗区中放置更多探针，如下所示：

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-14.jpg)

现在场景的暗区中也有了探针。因此，移动的救护车从场景的一侧移动到另一侧时，救护车上的光照更暗。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\class-LightProbeGroup-15.jpg)

现在，救护车在场景中心时，车上的光照更暗，这符合要求。

#### 反射探针

##### 反射探针优化原理

> **[Unity反射探针](https://www.baidu.com/s?sa=re_dqa_generate&wd=Unity反射探针&rsv_pq=811a3546000c7947&oq=unity反射探针的优化原理&rsv_t=675e5wySRcyKDwy3JvJjSzfVGXYH26lfjsH/f20uNsI0h+kbzv98YuDkzXAAVcCWJLQuUicF/rUq&tn=39042058_26_oem_dg&ie=utf-8)的优化原理**‌是通过在场景中的关键点对视觉环境进行采样，从而在保证高质量反射效果的同时，控制处理开销。反射探针可以在场景中的关键位置创建立方体贴图，这些立方体贴图表示了周围环境的反射信息。当反射对象靠近这些探针时，探针采样的反射可以被用于对象的反射贴图，从而实现非常逼真的反射效果‌12。
>
> ‌**反射探针的工作原理**‌基于立方体贴图，这种技术在场景中的每个关键点对视觉环境进行采样。探针通常放置在反射对象外观发生明显变化的每个点上，例如隧道、建筑物附近区域和地面颜色变化的地方。当反射对象靠近探针时，探针采样的反射被用于对象的反射贴图。此外，当几个探针位于彼此附近时，Unity可以在它们之间进行插值，实现反射的逐渐变化‌12。
>
> ‌**高级功能**‌包括互反射和盒体投影。互反射是指两个或多个物体之间的反射效果，这需要为互反射序列中的每个阶段创建多个快照。盒体投影则允许在距探针有限距离内创建反射立方体贴图，根据物体与立方体贴图墙壁的距离显示不同大小的反射。这些高级功能进一步提高了反射探针的视觉真实感‌4。

在Unity游戏开发过程中，环境渲染和物体反射效果对于提升游戏画面的真实感和沉浸感至关重要。本文将详细介绍如何在Unity中实现高质量的环境渲染以及如何使用反射[探针](https://so.csdn.net/so/search?q=探针&spm=1001.2101.3001.7020)（Reflection Probes）来模拟物体的反射效果，帮助开发者创建更加生动和逼真的游戏场景。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\772bafcb74044930b1ca40bc0ca227ed.png)

##### 环境渲染基础

Unity的环境渲染依赖于多种技术和组件的协同工作，其中最重要的是光照系统。光照系统不仅影响物体的亮度、阴影和颜色，还直接关系到物体表面的反射效果。为了获得高质量的渲染效果，开发者需要合理配置Unity中的光源（Lights）、材质（Materials）和光照贴图（Lightmaps）。

##### 光源设置

在Unity中，光源主要分为方向光（Directional Light）、点光源（Point Light）和聚光灯（Spot Light）等。方向光通常用于模拟太阳光，而点光源和聚光灯则用于模拟场景中的局部光源。合理布置这些光源，可以为场景提供丰富的光影效果。

##### 材质与光照贴图

材质是物体表面的视觉属性，包括颜色、纹理、光泽度等。在Unity中，开发者可以通过设置材质的Shader来定义物体如何响应光照。同时，光照贴图是一种用于存储场景光照信息的纹理，它可以显著提高场景的渲染效率和质量。

##### 反射探针（Reflection Probes）详解

反射探针是Unity中用于捕捉和存储周围环境反射信息的一种工具，它可以让场景中的物体呈现出逼真的反射效果。通过反射探针，Unity可以模拟光线在物体表面的反射，从而增强场景的真实感。

###### 反射探针的创建与配置

**在Unity中，创建反射探针非常简单。只需在Hierarchy视图中右键点击，选择“Light”->“Reflection Probe”即可创建一个反射探针对象。接下来，需要调整反射探针的大小和位置，以便它能够捕捉到周围环境的反射信息。**

在Inspector视图中，开发者可以设置反射探针的多种参数，如分辨率（Resolution）、刷新模式（Baking Mode）、捕捉频率（Capture Every N Frames）等。这些参数会直接影响反射效果的质量和性能。

![image-20241012113851068](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241012113851068.png)

###### 材质中的反射探针设置

要使物体使用反射探针的反射效果，需要在物体的材质中启用反射探针支持。在材质的Inspector视图中，找到“Reflection”或“Environment”相关的设置项，并选择使用反射探针。此外，还可以调整反射的混合模式（Blend Mode）和强度（Intensity）等参数，以获得最佳的反射效果。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\851ce8301b394d009c8b9407184b9cc2.png)

![image-20241012114029269](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241012114029269.png)

看倒数第三个

##### 实践案例

假设我们正在开发一个室内场景，需要让房间中的镜子和金属表面呈现出真实的反射效果。首先，我们需要在场景中合理布置反射探针，确保它们能够捕捉到周围环境的反射信息。然后，为镜子和金属表面的材质启用反射探针支持，并调整相关参数以获得满意的反射效果。

###### 实践案例：室内场景中的反射效果

> 这是我之前添加的一个探针，使用起来很简单（但是要记住范围不要超出当前场景，否则会渲染到场景外面的物体）
>
> ![image-20241012113404310](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241012113404310.png)

###### 场景设置

1. **创建场景**：在Unity中新建一个场景，并导入所需的室内环境模型、镜子和金属物体（如金属桌子、金属装饰品等）。
2. **布置光源**：
   - 添加一个方向光（Directional Light）作为主光源，模拟太阳光。
   - 在需要的地方添加点光源（Point Light）或聚光灯（Spot Light）以模拟室内灯光。
3. **材质准备**：
   - 为镜子和金属物体创建或选择适合的材质。确保这些材质支持反射效果。

###### 反射探针配置

1. 创建反射探针
   - 在Hierarchy视图中右键点击，选择`Light` -> `Reflection Probe`来创建一个新的反射探针。
   - 将反射探针放置在场景中的合适位置，通常是需要反射效果的区域中心或附近。![img](D:\DeskTop\FileSum\UnityClassResources\笔记\8ffe6c6931074363b9f92e1f28b3c641.png)
2. 调整反射探针属性
   - 在Inspector视图中，设置反射探针的`Resolution`（分辨率），这决定了反射图像的清晰度。
   - 选择合适的`Baking Mode`（烘焙模式），通常是`Realtime`（实时）或`Baked`（烘焙）。实时模式允许动态变化，但可能更消耗性能；烘焙模式则预先计算反射，性能更优但无法实时更新。
   - 调整`Capture Every N Frames`（每N帧捕捉一次）来控制反射更新的频率。
   - 使用`Box`（盒子）或`Sphere`（球体）来定义反射探针的捕捉范围。
3. 应用反射探针到材质
   - 在镜子和金属物体的材质中，找到`Reflection`或`Environment`设置项。
   - 选择`Use Reflection Probe`（使用反射探针）并指定具体的反射探针对象（如果场景中有多个反射探针）。

###### Unity代码示例（非直接配置反射探针）

虽然直接配置反射探针不需要编写代码，但如果你想通过脚本来动态控制反射探针（例如，改变其位置或启用/禁用），你可以这样做：

```cs
using UnityEngine;  

public class ReflectionProbeController : MonoBehaviour  

{  

    public ReflectionProbe reflectionProbe; // 拖拽或赋值反射探针  

    void Start()  
    {  
        // 可以在这里设置反射探针的初始状态或参数  
        // 例如，禁用反射探针的烘焙模式（如果它原本是烘焙的）  
        // 注意：这通常不是推荐的做法，因为烘焙的反射探针在运行时不能轻易更改  
        // 这里只是作为示例  
        // reflectionProbe.mode = ReflectionProbeMode.Realtime;  
    }  

    void Update()  
    {  

        // 假设你想根据某些条件动态改变反射探针的位置  

        // 这里仅作为示例，实际使用时需要根据具体需求编写逻辑  

        // 注意：频繁更改反射探针的位置可能会影响性能  

        // transform.position = new Vector3(Mathf.Sin(Time.time) * 5, 1, 0); // 假设这是反射探针的父对象  

        // 注意：实际上你应该直接修改反射探针的transform，而不是这个脚本所附加对象的transform  

        // reflectionProbe.transform.position = new Vector3(...);  

    }  

}
```

**注意**：在上面的代码示例中，我注释掉了直接修改反射探针模式的代码，因为烘焙的反射探针在运行时是不能更改其模式的。同时，我也提到了频繁更改反射探针位置可能会影响性能，因此在实际应用中需要谨慎操作。

此外，对于反射探针的`transform`属性的修改，你应该直接对反射探针对象本身进行操作，而不是通过脚本所附加的对象的`transform`来间接操作。上面的`reflectionProbe.transform.position`仅作为示例说明，你需要确保`reflectionProbe`已经正确引用到了场景中的反射探针对象。

##### 反射探针的高级使用

###### 1. 反射探针的烘焙

如果你选择了`Baked`模式作为反射探针的烘焙模式，那么你需要通过Unity的烘焙系统来预先计算反射信息。这通常是在场景设置完成后，通过Lightmapping窗口来完成的。

- 打开Window > Rendering > Lightmapping Settings。
- 在Lightmapping Settings窗口中，设置全局的烘焙参数，如分辨率、间接光照强度等。
- 确保你的反射探针已被选中为需要烘焙的对象之一。这通常在反射探针的Inspector视图中通过勾选“Bake”选项来完成。
- 点击“Bake”按钮开始烘焙过程。烘焙完成后，反射探针将包含周围环境的静态反射信息。

或者在![image-20241012114429953](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241012114429953.png)

###### 2. 反射探针的混合

在大型场景中，单一反射探针可能无法覆盖整个区域，或者你可能希望在不同区域使用不同的反射效果。这时，你可以使用多个反射探针，并通过设置它们的`Box Projection`或`Sphere Projection`参数来定义它们的捕捉范围。Unity会自动在这些反射探针之间进行混合，以提供更平滑的过渡效果。

###### 3. 实时反射与烘焙反射的结合

在某些情况下，你可能希望结合使用实时反射和烘焙反射。例如，你可以为动态物体（如玩家角色或NPC）使用实时反射探针，而为静态环境使用烘焙反射探针。这样可以在保持高性能的同时，为动态物体提供实时的反射效果。

###### 4. 反射探针的性能优化

反射探针虽然能显著提升场景的真实感，但它们也可能对性能产生较大影响，特别是当使用多个高分辨率的实时反射探针时。以下是一些优化建议：

- 尽量减少实时反射探针的数量，并尽可能使用烘焙反射探针。
- 降低实时反射探针的分辨率和更新频率。
- 对于不需要实时更新的物体，使用烘焙反射探针。
- 利用Unity的Layer和Culling Mask功能来限制反射探针的捕捉范围，只捕捉需要反射的物体。

### 6. **减少粒子系统的数量和复杂性**

   - 适当减少粒子数量，简化粒子的材质和效果设置。

#### 粒子系统对于CPU有哪些影响

在UWA报告的粒子模块中，有这些参数是需要我们关注的。 “ParticleSystem.Update”：**粒子系统更新的平均CPU耗时**； “ParticleSystem.Draw”：**粒子系统每帧提交Draw Call的平均CPU耗时**； “ParticleSystem.ScheduleGeometryJobs”：**该函数与粒子系统多线程更新任务的调度有关，一般来说，该值越大表示项目中Playing的粒子系统数量越多。**

![unity优化粒子特效 unity 粒子 优化_Gpu优化](https://s2.51cto.com/images/blog/202405/29153506_6656daaa44b5a9726.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

结合UWA的经验，一般来说主要受到的影响层面如下：

1、粒子系统数量 在UWA的性能简报里，直接搜索（ctrl+f）“粒子系统数量”，会出现这两条检测结果：

![unity优化粒子特效 unity 粒子 优化_Gpu优化_02](https://s2.51cto.com/images/blog/202405/29153506_6656daaa74c0098791.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

（1）粒子系统数量，UWA推荐不超过600（1G机型）。这个数量是指内存中所有的ParticleSystem的总数量，包含正在播放的和处于缓存池中的。

（2）Playing的粒子系统数量，这里指的是正在播放的ParticleSystem组件的数量，这个包含了屏幕内和屏幕外的，我们建议在一帧中出现的数量峰值不超过50（1G机型）。

那如何查看我项目运行过程中到底缓存了哪些粒子系统呢？这些Playing的粒子系统是否合理的呢？这里有个技巧，可以通过报告的【具体资源信息】-【粒子系统】中查看。

![unity优化粒子特效 unity 粒子 优化_粒子优化_03](https://s2.51cto.com/images/blog/202405/29153506_6656daaac9a8638618.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

如上图，蓝色线条是全部加载进内存的粒子系统，紫色是疑似冗余的，黄色是实际播放的粒子系统数量。跟随游戏运行的生命周期图，可以任选一帧时的截图，尤其是数量较高的部分，再打开【所选帧】模式，可以查看这一帧下所有的ParticleSystem和所有Play的ParticleSystem。

针对上述的2条问题，优化和分析的时候也可针对这两点：

（1）关注粒子系统数量峰值（即蓝色曲线）是否偏高。可选中某一峰值帧查看到底是哪些粒子系统缓存着，是否都合理，是否有过度缓存的现象；

（2）关注Playing数量峰值（即黄色曲线）是否偏高。可选中某一峰值帧查看到底是哪些粒子系统在播放，是否都合理，是否能做些制作上的优化（详见下文）。

2、关于ParticleSystem.Prewarm 在UWA报告的重要性能参数中我们可以留意到有个函数：ParticleSystem.Prewarm，表示当前帧有粒子系统开启了“Prewarm”选项，而开启该选项的粒子系统在场景中实例化或者由Deactive转为active时，会立即执行一次完整的模拟。以“火焰”为例，Prewarm开启时，加载后第一帧即能看到“大火”，而不是从“火苗”开始逐渐变大。

![unity优化粒子特效 unity 粒子 优化_粒子特效_04](https://s2.51cto.com/images/blog/202405/29153507_6656daab1cf4697778.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

 

![unity优化粒子特效 unity 粒子 优化_粒子系统_05](https://s2.51cto.com/images/blog/202405/29153507_6656daab600cd276.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

**但Prewarm的操作通常都有一定的耗时，建议在可以不用的情况下，将其关闭。**

![unity优化粒子特效 unity 粒子 优化_Gpu优化_06](https://s2.51cto.com/images/blog/202405/29153507_6656daabcb1ab59775.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)



#### 粒子系统对GPU端的影响

如果粒子系统问题较为严重，也会影响GPU端的表现。我们也可以结合UWA的真人真机测评报告中的Overdraw数据来排查定位，一般我们建议在中低端设备上不超过5。

![unity优化粒子特效 unity 粒子 优化_Gpu优化_07](https://s2.51cto.com/images/blog/202405/29153508_6656daac0fffa94513.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

结合Overdraw的走势图，配合对应的游戏截图，我们可以排查粒子特效是否面积过大、叠层过多等资源规范的问题。

![unity优化粒子特效 unity 粒子 优化_粒子特效_08](https://s2.51cto.com/images/blog/202405/29153508_6656daac3bb1624848.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

 

![unity优化粒子特效 unity 粒子 优化_粒子优化_09](https://s2.51cto.com/images/blog/202405/29153508_6656daac9dd4330700.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

**这里再补充一些常见的优化思路 对于低端设备尽可能降低粒子系统的复杂程度和屏幕覆盖面积，从而降低其渲染方面的开销，提升低端设备的运行流畅性。具体做法如下：**

（1）在中低端机型上降低粒子数、同屏粒子数，比如仅显示“关键”粒子特效或自身角色释放的粒子特效等，从而降低Update的CPU开销；

![image-20241012150212369](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241012150212369.png)

（2）尝试关闭离当前视域体或当前相机较远的粒子系统，离近后再进行开启，从而避免不必要的粒子系统Update的开销；

（3）尽可能降低粒子特效在屏幕中的覆盖面积，覆盖面积越大，层叠数越多，其渲染开销越大。

关于粒子系统的优化，我们曾在UWA DAY 2018中对移动游戏的GPU性能优化中做了些剖析，通过从Fillrate和Shader等几方面出发，结合大量优化过程中的实际案例分析游戏在GPU端的性能瓶颈：https://edu.uwa4d.com/course-intro/1/90

![unity优化粒子特效 unity 粒子 优化_unity优化粒子特效_10](https://s2.51cto.com/images/blog/202405/29153508_6656daacf10d928034.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

 

------

#### 如何规范大家的粒子系统（这部分还是欠缺理解和经验）

上述是提及的是粒子系统的优化点，但对于大部分的团队来说，都是在项目中期、后期的时候去做。但如果我们对粒子特效的性能压力没有全面的、科学的检测和预判，这个其实是存在风险的。那么我们如何在日常的开发中就通过一些科学的美术规范标准和检测方法，来保障最终在真机上的表现是丝滑流畅的呢？

1、本地资源检测是UWA推出的针对资源规范方面的监控服务，在日常开发过程中可对美术资源如纹理、网格、动画资源、粒子特效等进行逐一性能检测，如下图所示。

![unity优化粒子特效 unity 粒子 优化_粒子优化_11](https://s2.51cto.com/images/blog/202405/29153509_6656daad59c0a23119.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

粒子特效的检测原理本质上和ParticleEffectProfiler 类似，扫描过程需要在Game View下进行渲染，能把检测结果很直观地展现出来，对开发者是非常友好的。

报告中的检测规则为如下几条，后续还会逐渐完善增加。

（1）特效播放时平均Overdraw率过高 统计每帧参与渲染的像素的平均Overdraw，并取过程中最高的值。该值越大，特效导致GPU压力的可能性也会越高，建议对其进行检查。

（2）特效播放时DrawCall峰值过高 统计每帧的DrawCall数，并取过程中最高的值。该值较高时需要对其进行检查。

（3）特效总贴图内存过大 统计特效中包含的贴图总内存。该值较高时，可能是纹理使用过量，需要对其进行检查。

（4）特效运行时包含的ParticleSystem组件数量过多 统计特效中包含的ParticleSystem组件数。该值较高时，容易导致较高的渲染相关指标，以及较高的序列化耗时等，需要对其进行检查。

（5）特效播放时最大粒子数量过大 统计每帧的粒子数总和，并取过程中最高的值。该值越大，特效的更新开销可能也会越大。

（6）特效总贴图数量过大 统计特效中包含的贴图总数。该值较高时，容易导致较高的渲染相关指标，需要对其进行检查。

（7）开启Collision或Trigger的ParticleSystem 粒子系统建议不要开启Collison或Trigger功能，否则会有较高的物理开销。

该服务目前免费开放给开发者使用。

2、结合GOT Online的GPU耗时功能，逐一对技能的特效进行监控和优化。 （1）在真机上依次运行技能特效，技能特效的大小和位置，技术团队可以通过相机自行设定。这样在真机上就可以自动测试起来，通过真机上反馈的GPU耗时，就可以马上定位到究竟是哪些技能特效在不同档次的真机上运行起来GPU压力大。

![unity优化粒子特效 unity 粒子 优化_Gpu优化_12](https://s2.51cto.com/images/blog/202405/29153509_6656daadab2a587648.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

 

![unity优化粒子特效 unity 粒子 优化_unity优化粒子特效_13](https://s2.51cto.com/images/blog/202405/29153510_6656daae1bbe798558.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

同样的，Draw Call和Triangle也是如此，可以很快发现瓶颈。

![unity优化粒子特效 unity 粒子 优化_unity优化粒子特效_14](https://s2.51cto.com/images/blog/202405/29153510_6656daae70a3055602.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

还有团队会再进一步，把特效的实例化和Active/Deactive一起检测，这样就能知道哪些技能特效在运行时会带来性能隐患了。

![unity优化粒子特效 unity 粒子 优化_Gpu优化_15](https://s2.51cto.com/images/blog/202405/29153510_6656daaeb019f54824.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

这种流程已经在多个团队中定期执行，反馈效果很好。

除此之外，优秀的项目总有优秀的做法，这里推荐大家参阅《战双帕弥什》团队在UWA DAY上分享的大量粒子模拟方案，https://edu.uwa4d.com/course-intro/1/207

![unity优化粒子特效 unity 粒子 优化_Gpu优化_16](https://s2.51cto.com/images/blog/202405/29153511_6656daaf0c60f36716.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

以上就是在优化粒子系统时需要关注的一些问题和对应的方法，如何操作还需要大家结合项目实际情况，同时结合UWA服务可以快速地帮助大家定位到性能瓶颈。

#### GPU实例化

与 CPU 渲染相比，GPU 实例化可带来巨大的性能提升。如果希望粒子系统渲染__网格__粒子（而不是使用渲染__公告牌__粒子的默认[渲染模式](https://docs.unity3d.com/cn/2021.2/Manual/PartSysRendererModule.html)），则可使用实例化功能。

为了能够对粒子系统使用 GPU 实例化，请执行以下操作：

- 将粒子系统的[渲染器](https://docs.unity3d.com/cn/2021.2/Manual/PartSysRendererModule.html)模式设置为 Mesh
- 对支持 GPU 实例化的[渲染器](https://docs.unity3d.com/cn/2021.2/Manual/PartSysRendererModule.html)材质使用一个着色器
- 在支持 GPU 实例化的平台上运行项目

要为粒子系统启用 GPU 实例化，必须在粒子系统的 Renderer 模块中启用 Enable GPU Instancing 复选框。

![在 Renderer 模块中启用粒子系统 GPU 实例化的选项](https://docs.unity3d.com/cn/2021.2/uploads/Main/PartSysInstancingEnable.png)在 Renderer 模块中启用粒子系统 GPU 实例化的选项

Unity 带有一个支持 GPU 实例化的内置粒子着色器，但默认的粒子材质不使用该着色器，因此必须更改此设置以使用 GPU 实例化。支持 GPU 实例化的粒子着色器名为 Particles/Standard Surface。要使用它，必须创建自己的新__材质__，并将材质的着色器设置为 Particles/Standard Surface。然后，必须将此新材质分配给粒子系统[渲染器](https://docs.unity3d.com/cn/2021.2/Manual/PartSysRendererModule.html)模块中的材质字段。

![与粒子系统 GPU 实例化兼容的内置着色器](https://docs.unity3d.com/cn/2021.2/uploads/Main/PartSysInstancingShader.png)与粒子系统 GPU 实例化兼容的内置着色器

如果要为粒子使用其他不同的着色器，必须使用“#pragma 目标 4.5”或更高版本。有关更多详细信息，请参阅[着色器编译目标](https://docs.unity3d.com/cn/2021.2/Manual/SL-ShaderCompileTargets.html)。此要求高于 Unity 中的常规 GPU 实例化，因为粒子系统将其所有实例数据写入单个大缓冲区，而不是将实例化分解为多个绘制调用。

**优化原理**

> Unity 中的内置粒子着色器支持 GPU 实例化，这一功能显著提升了粒子系统的性能和效率。其主要作用是允许在 GPU 上并行处理多个粒子，从而减少 CPU 的负担，提高渲染速度。
>
> 原理上，GPU 实例化通过将多个粒子实例的属性（如位置、缩放、旋转等）存储在缓冲区中，利用 GPU 的强大并行计算能力一次性处理这些数据。具体而言，粒子系统根据每个粒子的属性生成一个实例，然后在渲染时一次性绘制所有实例，这样避免了每个粒子都需要单独进行绘制调用（draw call），从而大幅提升了性能。
>
> 使用 GPU 实例化时，粒子着色器会通过着色器编程语言（如 HLSL）来访问这些属性，并在 GPU 上完成变换和光栅化等操作，最终渲染出高效的粒子效果。在实现时，开发者需要确保传递给着色器的所有信息都在定义的缓冲区内，以便充分利用 GPU 实例化的优势。

#### (下面这部分等我学一下shader再回来啃)

#### 自定义着色器示例

也可以编写使用 GPU 实例化的自定义着色器。有关更多信息，请参阅以下部分：

- [表面着色器中的粒子系统 GPU 实例化](https://docs.unity3d.com/cn/2021.2/Manual/PartSysInstancing.html#SurfaceShader)
- [自定义着色器中的粒子系统 GPU 实例化](https://docs.unity3d.com/cn/2021.2/Manual/PartSysInstancing.html#CustomShader)
- [自定义粒子系统使用的实例数据](https://docs.unity3d.com/cn/2021.2/Manual/PartSysInstancing.html#CustomVertexStreams)（与自定义顶点流一起使用）

#### 表面着色器中的粒子系统 GPU 实例化

以下是使用粒子系统 GPU 实例化的表面着色器的完整运行示例：

```
Shader "Instanced/ParticleMeshesSurface" {
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200

        CGPROGRAM
        // 基于物理的标准光照模型，并对所有光照类型启用阴影
        // 并通过实例化支持来生成阴影 pass
        #pragma surface surf Standard nolightmap nometa noforwardadd keepalpha fullforwardshadows addshadow vertex:vert
        // 启用此着色器的实例化
        #pragma multi_compile_instancing
        #pragma instancing_options procedural:vertInstancingSetup
        #pragma exclude_renderers gles
        #include "UnityStandardParticleInstancing.cginc"
        sampler2D _MainTex;
        struct Input {
            float2 uv_MainTex;
            fixed4 vertexColor;
        };
        fixed4 _Color;
        half _Glossiness;
        half _Metallic;
        void vert (inout appdata_full v, out Input o)
        {
            UNITY_INITIALIZE_OUTPUT(Input, o);
            vertInstancingColor(o.vertexColor);
            vertInstancingUVs(v.texcoord, o.uv_MainTex);
        }

        void surf (Input IN, inout SurfaceOutputStandard o) {
            // Albedo 来自颜色着色的纹理
            fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * IN.vertexColor * _Color;
            o.Albedo = c.rgb;
            // Metallic 和 Smoothness 来自滑动条变量
            o.Metallic = _Metallic;
            o.Smoothness = _Glossiness;
            o.Alpha = c.a;
        }
        ENDCG
    }
    FallBack "Diffuse"
}
```



 

上面的示例与常规[表面着色器](https://docs.unity3d.com/cn/2021.2/Manual/SL-SurfaceShaders.html)之间存在许多微小差异，正是这些不同之处使得表面着色器可以使用粒子实例化。

首先，必须添加以下两行以启用程序实例化 (Procedural Instancing)，并指定内置顶点设置函数。此函数位于 UnityStandardParticleInstancing.cginc 中，用于加载每个实例（每个粒子）的位置数据：

```
 #pragma instancing_options procedural:vertInstancingSetup
                #include "UnityStandardParticleInstancing.cginc"
```

 

此示例中的其他修改是对顶点函数的修改，此函数增加了两行来应用每个实例的属性，具体而言就是粒子颜色和[纹理帧动画](https://docs.unity3d.com/cn/2021.2/Manual/PartSysTexSheetAnimModule.html)纹理坐标：

```
vertInstancingColor(o.vertexColor);
                        vertInstancingUVs(v.texcoord, o.uv_MainTex);
```

#### 自定义着色器中的粒子系统 GPU 实例化

以下是使用粒子系统 GPU 实例化的自定义着色器的完整运行示例。此自定义着色器添加了标准粒子着色器所不具备的功能：[纹理帧动画](https://docs.unity3d.com/cn/2021.2/Manual/PartSysTexSheetAnimModule.html)的各个帧之间的淡入淡出。



```
Shader "Instanced/ParticleMeshesCustom"
{
    Properties
    {
        _MainTex("Albedo", 2D) = "white" {}
        [Toggle(_TSANIM_BLENDING)] _TSAnimBlending("Texture Sheet Animation Blending", Int) = 0
    }
    SubShader
    {
        Tags{ "RenderType" = "Opaque" }
        LOD 100
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile __ _TSANIM_BLENDING
            #pragma multi_compile_instancing
            #pragma instancing_options procedural:vertInstancingSetup
            #include "UnityCG.cginc"
            #include "UnityStandardParticleInstancing.cginc"
            struct appdata
            {
                float4 vertex : POSITION;
                fixed4 color : COLOR;
                float2 texcoord : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };
            struct v2f
            {
                float4 vertex : SV_POSITION;
                fixed4 color : COLOR;
                float2 texcoord : TEXCOORD0;
# ifdef _TSANIM_BLENDING
                float3 texcoord2AndBlend : TEXCOORD1;   
# endif
            };
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed4 readTexture(sampler2D tex, v2f IN)
            {
                fixed4 color = tex2D(tex, IN.texcoord);
# ifdef _TSANIM_BLENDING
                fixed4 color2 = tex2D(tex, IN.texcoord2AndBlend.xy);
                color = lerp(color, color2, IN.texcoord2AndBlend.z);
# endif
                return color;
            }
            v2f vert(appdata v)
            {
                v2f o;
                UNITY_SETUP_INSTANCE_ID(v);
                o.color = v.color;
                o.texcoord = v.texcoord;
                vertInstancingColor(o.color);
# ifdef _TSANIM_BLENDING
                vertInstancingUVs(v.texcoord, o.texcoord, o.texcoord2AndBlend);
# else
                vertInstancingUVs(v.texcoord, o.texcoord);
# endif
                o.vertex = UnityObjectToClipPos(v.vertex);
                return o;
            }
            fixed4 frag(v2f i) : SV_Target
            {
                half4 albedo = readTexture(_MainTex, i);
                return i.color * albedo;
            }
            ENDCG
        }
    }
}
```



 

此示例包含与表面着色器相同的用于加载位置数据的设置代码：

```
        #pragma instancing_options procedural:vertInstancingSetup
                #include "UnityStandardParticleInstancing.cginc"
```

 

对顶点函数的修改也与表面着色器非常相似：



```
                vertInstancingColor(o.color);
                # ifdef _TSANIM_BLENDING
                                vertInstancingUVs(v.texcoord, o.texcoord, o.texcoord2AndBlend);
                # else
                                vertInstancingUVs(v.texcoord, o.texcoord);
                # endif
```



 

与上面的第一个示例相比，这里唯一的区别是纹理帧动画混合。这意味着，着色器需要一组额外的纹理坐标来读取纹理帧动画的两个帧而不是一个帧，然后将它们混合在一起。

最后，片元着色器读取纹理并计算最终颜色。



#### 粒子系统 GPU 实例化与自定义顶点流

上面的示例仅使用粒子的默认顶点流设置。这包括位置、法线、颜色和一个 UV。但是，通过使用[自定义顶点流](https://docs.unity3d.com/cn/2021.2/Manual/PartSysVertexStreams.html)，可以将其他数据发送到着色器，例如速度、旋转和大小。

在接下来这一个示例中，着色器用于显示特殊效果，使速度更快的粒子看起来更亮，更慢的粒子更暗。有一些额外的代码用于根据速度来提高粒子的亮度（使用速度顶点流）。此外，因为此着色器假定该效果不使用纹理帧动画，所以从自定义流结构中将其省略。

以下是完整的着色器：

```
Shader "Instanced/ParticleMeshesCustomStreams"
{
    Properties
    {
        _MainTex("Albedo", 2D) = "white" {}
    }
    SubShader
    {
        Tags{ "RenderType" = "Opaque" }
        LOD 100
        Pass
        {
            CGPROGRAM
# pragma exclude_renderers gles
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_instancing
            #pragma instancing_options procedural:vertInstancingSetup
            #define UNITY_PARTICLE_INSTANCE_DATA MyParticleInstanceData
            #define UNITY_PARTICLE_INSTANCE_DATA_NO_ANIM_FRAME
            struct MyParticleInstanceData
            {
                float3x4 transform;
                uint color;
                float speed;
            };
            #include "UnityCG.cginc"
            #include "UnityStandardParticleInstancing.cginc"
            struct appdata
            {
                float4 vertex : POSITION;
                fixed4 color : COLOR;
                float2 texcoord : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };
            struct v2f
            {
                float4 vertex : SV_POSITION;
                fixed4 color : COLOR;
                float2 texcoord : TEXCOORD0;
            };
            sampler2D _MainTex;
            float4 _MainTex_ST;
            v2f vert(appdata v)
            {
                v2f o;
                UNITY_SETUP_INSTANCE_ID(v);
                o.color = v.color;
                o.texcoord = v.texcoord;
                vertInstancingColor(o.color);
                vertInstancingUVs(v.texcoord, o.texcoord);
# if defined(UNITY_PARTICLE_INSTANCING_ENABLED)
                UNITY_PARTICLE_INSTANCE_DATA data = unity_ParticleInstanceData[unity_InstanceID];
                o.color.rgb += data.speed;
# endif
                o.vertex = UnityObjectToClipPos(v.vertex);
                return o;
            }
            fixed4 frag(v2f i) : SV_Target
            {
                half4 albedo = tex2D(_MainTex, i.texcoord);
                return i.color * albedo;
            }
            ENDCG
        }
    }
}
```



 

此着色器包含 `UnityStandardParticleInstancing.cginc`（其中含有未使用自定义顶点流时的默认实例化数据布局）。因此，在使用自定义流时，必须重载该头部中定义的某些默认值。这些重载必须位于包含 (include) 之前。上面的示例设置了以下自定义重载：

首先，有一行代码使用 UNITY_PARTICLE_INSTANCE_DATA 宏，告诉 Unity 为自定义流数据使用名为“MyParticleInstanceData”的自定义结构：

```
            #define UNITY_PARTICLE_INSTANCE_DATA MyParticleInstanceData
```

其次，另一个 define 语句告诉实例化系统在此着色器中不需要动画帧流 (Anim Frame Stream)，因为此示例中的效果不适用于纹理帧动画：

```
            #define UNITY_PARTICLE_INSTANCE_DATA_NO_ANIM_FRAME
```

第三，声明自定义流数据的结构：



```
            struct MyParticleInstanceData
                        {
                            float3x4 transform;
                            uint color;
                            float speed;
                        };
```



这些重载全部都是在 `UnityStandardParticleInstancing.cginc` 之前包含进来的，因此着色器不会对这些定义使用自己的默认值。

编写结构时，变量必须与粒子系统渲染器模块中检视面板 (Inspector) 中列出的顶点流相匹配。这意味着必须在渲染器模块 UI 中选择要使用的流，然后以相同的顺序将它们添加到自定义流数据结构中的变量定义，从而使它们匹配：

![渲染器模块 UI 中显示的自定义顶点流，显示了一些实例化的流和一些非实例化的流](https://docs.unity3d.com/cn/2021.2/uploads/Main/PartSysInstancingCustomVertexStreams.png)渲染器模块 UI 中显示的自定义顶点流，显示了一些实例化的流和一些非实例化的流

第一项 (Position) 是必填项，因此无法将其删除。可使用加号和减号按钮自由添加/删除其他条目，从而自定义顶点流数据。

列表中以 INSTANCED 结尾的条目包含实例数据，因此必须将它们包含在粒子实例数据结构中。直接附加到单词 INSTANCED 后面的数字（例如 INSTANCED0 中的 0 和 INSTANCED1 中的 1）表示变量必须在结构中的初始“transform”变量*之后*出现的顺序。尾部字母（.x、.xy、.xyz 或 .xyzw）表示变量的类型，并映射到着色器代码中的 float、float2、float3 和 float4 变量类型。

可在粒子实例数据结构中忽略在列表中显示但结尾*没有*单词 INSTANCED 的任何其他顶点流数据，因为它不是着色器要处理的实例化数据。此数据属于源网格，例如 UV、法线和切线。

完成我们的示例的最后一步是将速度应用于顶点着色器内的粒子颜色：

```
# if defined(UNITY_PARTICLE_INSTANCING_ENABLED)
                UNITY_PARTICLE_INSTANCE_DATA data = unity_ParticleInstanceData[unity_InstanceID];
                o.color.rgb += data.speed;
# endif
```

必须将所有实例化代码包装在 UNITY_PARTICLE_INSTANCING_ENABLED 的检查中，以便在未使用实例化时进行编译。

此时，如果要将数据传递给片元着色器，可将数据写入 v2f 结构，就像对任何其他着色器数据的处理一样。

此示例描述了如何修改自定义着色器以便用于自定义顶点流，但可以对表面着色器应用完全相同的方法来实现相同的功能。

--------------------

在Unity中降低粒子系统的复杂度和屏幕覆盖面积可以通过以下几个步骤实现：

1. **降低粒子数量**：
   - 在粒子系统的“Emission”模块中，减少“Rate over Time”或“Burst”中粒子的发射数量。

2. **简化粒子材质**：
   - 使用简单的材质，例如使用Unlit材质，避免使用复杂的着色器，减少性能开销。

3. **调整粒子大小**：
   - 在“Main”模块中，缩小“Start Size”。减小粒子大小可以降低占用的屏幕面积。

4. **减少生命周期**：
   - 在“Main”模块中，缩短“Start Lifetime”。这样粒子产生并消失的速度加快，减少屏幕上的粒子数量。

5. **使用低分辨率的纹理**：
   - 尽量使用较低分辨率的纹理图，这样可以减少内存和处理的负担。

6. **限制粒子运动**：
   - 通过调整“Velocity over Lifetime”模块，限制粒子的速度和方向变化。

7. **使用 LOD（Level of Detail）技术**：
   - 为不同的设备设置不同的粒子效果，在低端设备上使用更简单的粒子设置和模型。

8. **禁用不必要的模块**：
   - 如果粒子系统中有不必要的模块（如“Collision”、“Sub Emitters”等），可以将其禁用或删除。

9. **减少粒子系统的数量**：
   - 合并多个粒子系统为一个系统，或使用少量的粒子系统来达到相似的效果。

通过这些方法，可以有效降低低端设备上粒子系统的复杂度和屏幕覆盖面积，提高性能。

### 7. **合理设计场景**

   - **使用低多边形模型**：特别是在场景的远处背景。
   - **限制可见的场景内容**：使用较小的场景单元，仅加载当前需要可见的部分。

### 8. **后期处理优化**

   - **选择性使用后期效果**：避免不必要的后期处理效果，减少计算开销。

### 9. **性能监测**

   - 使用Unity Profiler工具监测性能瓶颈，识别需要优化的地方。

### 10. **脚本优化**

   - 避免在Update()中进行昂贵的操作，尽可能使用协程（Coroutines）和事件驱动的方式。

     **实现分帧执行**

通过上述方法，可以显著提高Unity项目的渲染性能，使得游戏在各种设备上运行更流畅。

## 后处理效果

### 一、安装

打开PackageManager，然后搜索Post，应该就能看到左边出现搜索结果，选择，然后Install安装。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/598fe506a407b86cdd2f9690f42220a2.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75d09a10c082ffa412d49906e6642726.png#pic_center)

安装完之后，会看到在Packages里面出现了PostProcessing文件夹。

### 二、使用

#### 1、添加PostProcessLayer组件

> 在urp项目中摄像机不需要添加这个组件，但是需要打开postProcessing
>
> 除此之外urp项目有独有的几种后处理，较为有效

这个组件一般是添加在摄像机上面的。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48760844a9dc4c11d10b4ae747ba89fe.png#pic_center)

#### 2、添加PostProcessVolume组件

这个组件千万不要添加在摄像机上面，原因下面会说，建一个空物体，然后在它身上添加。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16e9228d62af0f937748b31dd4605b71.png#pic_center)

#### 3、创建Profile

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b54315bf6b68d634d3f553e95fbf76e5.png#pic_center)

点New按钮，就会自动创建一个新的Profile文件，当然也可以直接拖旧的文件到这里。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/87994694f8a26edc77aa1a5472e094e7.png#pic_center)

新建后，会发现Unity其实是帮我们建立了一个新文件。

#### 4、添加效果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb441fe9a6ffd049c37fc1379fb60e1c.png#pic_center)

点击Add effect按钮
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/12056ac44418889dc66f48d9171123f6.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3aa2484b2378644f0f5b583529012e51.png#pic_center)

选择其中一个效果，并设置参数。我这里随便先了一个Bloom效果。

#### 5、改变设置

这时候发现场景里面的东西并没有Bloom的效果。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df5b10acdd12d0677279d4a01a91ef9a.png#pic_center)

下面对设置进行一些修改，先不用管为什么，下面会详细说。
先把IsGlobal选项勾上。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/487184c813b806f692181b0f66b66213.png#pic_center)

然后去PostProcessLayer，看到一个警告，说没有设置layer，因为当前是选择了Nothing。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/073ad7680bc25bfe3d8527c9e3aa867e.png#pic_center)

先随便选择一个[EveryThing](https://so.csdn.net/so/search?q=EveryThing&spm=1001.2101.3001.7020)，先不用管下面的警告，之后会说。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d1c12548ef16dc817e74f1dc54189476.png#pic_center)

这时候，看到场景已经亮起来了，有了Bloom的效果。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4ba65bc7f4ca3776b4e99ba9a83d4a9d.png#pic_center)

#### 6、创建layer

回头再来看刚才的警告
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9ff4e0b3bc78ace7eb0471294939ae9a.png#pic_center)

警告的意思是，不要用EveryThing和Default作为一个layer层遮罩使用。因为一般一个PostProcessLayer只需要看到自己指定的layer的volume，如果把所有层或者默认层都添加进去，会无端端增加了很多没有volume的对象，无端端增加了很多计算量。这一点，是一般美术同事们最容易忽略而犯错的地方。
于是就建一个新的Layer
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/502fe76ee00dfc0d772b480aff6de1a3.png#pic_center)

为了容易看，我把新的层命名为PostProcess
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38950e22d19ca06ef97143786bf8012e.png#pic_center)



然后设置volume所在的物体的层为PostProcess
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff3ca691271c000e3db2ed92fb21e7a0.png#pic_center)

PostProcessLayer指定的Layer也设置为只有PostProcess
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b35e72db574b8f32dec6d2345bf2eda.png#pic_center)

### 三、PostProcessLayer的设置

下面详细说一下PostProcessLayer的设置项：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a033f9977aa083fc867282426ef35617.png#pic_center)

#### 1、Volume blending

#### 1.Trigger

这里指定了这个PostProcessLayer针对哪个摄像机，一般习惯都是挂在摄像机上面，所以默认就是自己，或者点This指定所在物体本身的摄像机。
2.Layer
指定这个PostProcessLayer可以渲染哪些Layer的Volume。

#### 2、Anti-aliasing

抗锯齿设置
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2e40fe084b45678b54c9c8a3a54be7a8.png#pic_center)

#### 3、Stop NaN Propagation

开启之后unity会自动用黑色代替没有渲染好的像素
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3eb900c732161039ce9fdb788be5d335.png#pic_center)

说明里面提到了，在GLES2平台上，这个设置是没有效果的。

#### 4、Directly to Camera Target

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/076a31354c69b456a8c2570f6007996c.png#pic_center)

使用最终的位块到相机渲染目标进行后期处理。
提示里面说明，如果勾上这个，会减少开销，但会破坏使用旧版本的使用OnRenderImage的图片效果的兼容性。

#### 5、Toolkit

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b65a3441ea92700a25a28912193a407f.png#pic_center)

##### 1.Export frame to EXR

可以把后处理导出成EXR格式

##### 2.Select all layer volumes

快速选择场景里面所有的后处理volume

##### 3.Select all active volumes

快速选择场景里面所有激活状态的volume

##### 6、CustomEffectSorting

用于改变自定义后处理的渲染顺序。

### 四、PostVolume的设置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8aa0e8ea3804a65d1635feaffbfcbf2d.png#pic_center)

#### 1、IsGlobal

设置当前这个Volume是否全局的。
全局的意思就是，不管放在哪里，这个Volume都能生效。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8abd123215218647a1de88bf9826ee6.png#pic_center)

如果把IsGlobal去掉，会出现一个BlendDistance
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0eca3a6967bc684780f8ab5997be2459.png#pic_center)

从提示可以看出，我们想使用局部的后处理Volume，必须是先要添加一个碰撞体的。然后这个混合距离，就是后处理从无到有的过程中的一个过渡性的混合效果。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ebb8d2a44672bedb975aca1c84f71e88.png#pic_center)

于是添加一个球体碰撞器
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f6e858ed6555ee5e577d77a2deb2781.png#pic_center)

可以看到这个Volume上面出现了两个范围显示，一个是碰撞体范围，一个就是BlendDistance的范围
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b46d258c2cc6831c9ffca06c7220094a.png#pic_center)

这个时候，如果我们移动摄像机，进入触发范围，就可以看到有后处理效果，离开了范围之后，后处理效果就不再显示。
volume有全局和局部功能，还可layer与坐标有关系，所以千万千万不要把Volume挂在摄像机上面，这也是很多美术同事会忽略和犯错的地方。

#### 2、Weight

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/373ec9195c072e2770bf9adda9ed2c18.png#pic_center)

当前的Volume在场景里面占的权重值。

#### 3、Priority

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3e29bd538806d319571f23cba310241.png#pic_center)

当前Volume在堆栈里面的优先级别。

> 补充：
>
> 在计算机科学中，尤其是在数据结构和算法领域，"Volume在堆栈里面的优先级别"通常指的是在某个上下文中资源或操作的优先级。这个优先级直接影响到在堆栈中处理这些资源时的行为和顺序。
>
> 堆栈是一种后进先出（LIFO）的数据结构，通常用于管理函数调用、表达式求值或资源管理。优先级别的设置决定了在堆栈操作中的处理顺序。
>
> 如果Volume的优先级低，那么它可能会在堆栈中被延迟处理，直到高优先级的操作完成。这会导致系统在处理低优先级Volume时的响应时间变长，可能影响用户体验或引发性能瓶颈。
>
> 相反，如果Volume的优先级高，系统会优先处理这个Volume，保证其尽快完成。这在需要快速反应或及时处理某些关键操作时是有利的，但如果过度强调高优先级，可能会导致低优先级的任务长时间得不到处理，从而影响其他操作的执行。
>
> 总之，调整堆栈中的优先级别可以帮助管理不同操作的执行顺序，从而优化性能和响应时间。合理的优先级设置能有效地平衡系统资源和用户需求。
>
> 

#### 4、Profile

实际设置后处理效果的Profile文件指定。

还可以自己写shader，自定义效果，但是目前还没有学习shader。

## URP渲染管线的使用和优化

### 什么是渲染管线

> - [什么是通用渲染管线（URP）？](#related)
>
> 通用渲染管线（Universal Render Pipeline，简称URP）是Unity引擎的一种可编程渲染管线，旨在提供跨平台的高效渲染解决方案。URP为开发者提供了灵活性，同时优化了性能，特别适合于制作2D和3D游戏，以及手机、VR和AR等多种平台上的应用。
>
> ### URP的主要特点
>
> 1. **跨平台支持**：
>    - URP支持多个平台，包括移动设备、PC和控制台，让开发者能够在不同硬件上实现一致的视觉效果。
>
> 2. **优化性能**：
>    - URP设计时考虑了性能优化，能够处理较高的帧率，并适合资源受限的设备。它通过更高效的渲染管道、SRP Batcher等技术来减少绘制调用和状态切换。
>
> 3. **灵活性**：
>    - 开发者可以自定义渲染流程和着色器，允许对渲染结果进行高度定制，满足特定的视觉需求。
>
> 4. **Shader Graph支持**：
>    - URP支持Shader Graph，开发者可以无需编写代码即可创建复杂的着色器，适用于拥有各种视觉特效的游戏。
>
> 5. **现代图形特性**：
>    - URP支持多种现代图形特性，包括光照剔除、全局光照、后处理效果等，能够实现更高质量的视觉效果。
>
> 6. **简化的设置和使用**：
>    - URP的设置过程相对简单，使开发者能够快速上手并实施渲染管线，而无需深入了解底层图形编程。
>
> ### 适用场景
>
> - 适用于需要高性能和高效渲染的手机游戏。
> - 可以快速创建原型和低延迟的实时应用。
> - 适合需要简单图形支持的2D游戏开发。
>
> 总之，URP为开发者提供了一个高效、灵活的工具，帮助他们创建出色的图形效果，同时保持较好的性能和兼容性。
>
> - [SRP Batcher与传统渲染管线有何不同？](#related)
>
>   SRP Batcher与传统渲染管线主要有以下几个区别：
>
>   1. **设计理念**：
>      - **SRP Batcher**：专为Scriptable Render Pipeline（可编程渲染管线）设计。它使开发者能够按需定制渲染流程，同时通过批处理机制减少状态切换和绘制调用，以提高性能。
>      - **传统渲染管线**：基于固定渲染流程，较为僵化。它在性能方面的优化空间有限，主要依赖于Unity自动化的批处理进行简单的合批。
>
>   2. **批处理策略**：
>      - **SRP Batcher**：使用“对象始终在同一GPU状态下”的原则，使得渲染过程中可以同时处理多个材质和对象，以减少状态更改。这种方法有效减少了GPU的状态切换。
>      - **传统渲染管线**：依赖Unity自身的合批机制（如静态合批和动态合批），但在处理状态切换或大规模场景时，性能提升效果有限。
>
>   3. **性能优化**：
>      - **SRP Batcher**：可以显著提高包含大量小对象（如粒子或小道具）场景的性能。优化能够通过显著降低每帧的绘制调用数和GPU状态切换。
>      - **传统渲染管线**：在某些情况下可以利用Unity的合批机制来提高性能，但对于复杂和动态场景，可能达到性能瓶颈。
>
>   4. **灵活性**：
>      - **SRP Batcher**：提供更大的灵活性来定义和控制渲染流程，开发者可以根据游戏的需求定制渲染行为。
>      - **传统渲染管线**：功能和选项有限，通常需要开发者接受既定的渲染流程。
>
>   5. **支持的材质类型**：
>      - **SRP Batcher**：现在主要支持URP和HDRP中的“支持SRP Batcher”的自定义着色器和材质。
>      - **传统渲染管线**：仅支持固定管线中的标准材质，从而限制了自定义渲染效果的能力。
>
>   总的来说，SRP Batcher为开发者提供了更高的性能优化能力和灵活性，使其能够充分利用现代硬件的能力，而传统渲染管线则在设计和功能上较为有限。

unity版本2023.2.20f1c1，[urp版本16.0.6](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@16.0/manual/index.html)

Universal Render Pipeline (URP) ，unity提供的Scriptable Render Pipeline；

### 1. 开始使用URP：

1. 使用urp模板创建新项目：

​    ![img](https://i-blog.csdnimg.cn/direct/e1f9f09799684e40be4a76f545b2c430.png)

2. 将现有项目切换为urp：

​    1. 安装Universal RP package：

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\4f6d2a58f5124980bf18933350c1eaa1.png)

​    2. 配置urp：

​        1. 创建Universal Render Pipeline Asset：

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\76a6684cfa2243f0905e044b05cd46d9.png)

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\221a377d66244003991e79509297655d.png)

​            Universal Render Pipeline Asset 包含全局渲染和品质设置；

​            同时也创建了一个渲染管线实例（包含中间资源和渲染管线实现）；

​        2. 设置URP为激活的渲染管线：

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\53eac84177154aaa97e68a069afc2118.png)

​            也可以给不同的品质设置不同的 URP Asset；

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\d70721e4581348619db16639d402e944-17286540543366.png)

​            如果是使用模板新创建的项目，会自动包含几个不同品质的URP Asset；

3. 导入urp例子：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\c68b21b998ff4243a451a240a47cc79d.png)

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\860c889fdc42487ebbf30bd6ffe03d16.png)

​    每个例子使用各自的URP Asset，如果要构建的话，得进行Graphics设置；

4. 创建场景模板：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\ac68ae4917884a2cb86c76ea29749b5b.png)

​    Basic (URP)：包含一个摄像机和一个线性光；

​    Standard (URP)：包含一个摄像机，一个线性光和一个全局Volume（各种后处理）；

5. 可以给每个品质创建一个URP Asset，这样运行时方便动态切换URP Asset：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\8d08a0c012624c7496dedd0a8e72823a.png)

​    通过API QualitySettings.SetQualityLevel() 更改品质；

​    脚本更改URP Asset：

```cs
UniversalRenderPipelineAsset data = GraphicsSettings.currentRenderPipeline as UniversalRenderPipelineAsset;



data.renderScale = 1.0f;
```

6. 性能优化：

使用Unity Profiler 或者 GPU profiler (RenderDoc or Xcode)，检查urp使用的内存，cpu和gpu；

 配置URP Asset，可做相关优化：

​    1. 减少cpu占用，如关闭每帧刷新volumes；

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\f401812670f74c168c216ad5778624c6.png)

​    2. 减少纹理内存，如关闭HDR；**(降低内存的占用)**

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\9aa829d37ab2444c81f116b76093159d.png)

​    3. 减少渲染纹理拷贝到内存（移动平台影响很大），如关闭depth texture；

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\5c39b20e71e84a879f947073c9cfc660.png)

​    4. 减少渲染Pass，如关闭opaque texture和additional lights阴影投射；

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\ffe662df2eda4c49b33612f40ec1b575.png)

​    5. 减少发送到gpu的drawcall数，如开启SRP Batcher；

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\e3cb2d32e18d46c9af365749dfe18ae0.png)

​    6. 减少渲染到屏幕的像素数量（移动平台影响很大），如减小render scale；

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\f0b2ae7d9ae442d4810cfd181cd9ebbb.png)

优化点分类：

​    1. 选择合适的rendering path，urp支持3中pass（后续说明）；

​    2. 减少urp内存占用：

​      URP Asset:

​        关闭Depth Texture(除非shader需要采样depth)；
​        关闭Opaque Texture(所有不透明物体快照，相当于GrabPass)；

> GrabPass 是一种用于创建特殊通道的命令，可以将帧缓冲区的内容捕获到纹理中。这些纹理随后可用于实现基于图像的高级效果。在使用 GrabPass 之前，需要在 Pipeline Asset 中勾选 Opaque Texture，否则无法获取到相应的纹理。需要注意的是，GrabPass 中不能使用其他命令或代码，除了名称和标签。

​        使用延迟渲染路径时，关闭Use Rendering Layers；
​        关闭High Dynamic Range (HDR)，如果需要，设置HDR Precision 为 32 Bit；
​        减小Main Light > Shadow Resolution；
​        减小Additional Lights > Shadow Atlas Resolution；
​        关闭Light Cookies或者减小Cookie Atlas Resolution 和 Cookie Atlas Format；

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\d4e733a83b874744982afc477fb38b23.png)

​        设置Store Actions 为 Auto or Discard；

​      Universal Renderer asset（渲染实例）：

​        设置Intermediate Texture 为 Auto；

​      其他：

​        减少使用Decal Renderer Feature，因为需要额外的pass来渲染Decal；

​        shader变体裁剪；

​    3. 减少cpu处理时间：

​      URP Asset:

​        设置Volume Update Mode 为 Via Scripting，这样不必每帧刷新volumes；
​        低端设备使用Reflection Probes时，关闭Probe Blending 和 Box Projection；
​        减小Shadows > Max Distance，较少的对象执行shadow pass（节省gpu处理时间）；
​        减少Shadows > Cascade Count，这样减少了渲染pass数目（节省gpu处理时间）；
​        关闭Additional Lights > Cast Shadows，节省gpu处理时间和urp使用内存；

​      减少相机使用数量；

​    4. 减少gpu处理时间：

​      URP Asset：

​        减小或关闭Anti-aliasing (MSAA)：
​            这样URP不会使用内存带宽来拷贝帧缓冲到内存或从内存复制出来；
​        关闭Terrain Holes；

> 关闭Terrain Holes的作用是消除地形中的孔洞或缺口，确保地形显示更加连贯和自然。这通常用于游戏开发或三维建模中，以避免视觉上的不连贯和提升用户体验。修复这些孔洞可以使地形更加完整，减少潜在的物理问题，如角色穿越地形。

​        开启SRP Batcher：
​            减少GPU在draw call之间的状态设置；
​            使材质球数据持续保存在GPU内存中；

> SRP Batcher是一种用于Unity引擎的渲染优化技术，旨在提高使用通用渲染管线（URP）和高清渲染管线（HDRP）的渲染效率。它通过减少每帧渲染所需的状态变化和绘制调用数量，加速渲染流程，从而提高性能。SRP Batcher专门为Scriptable Render Pipeline设计，能显著增强游戏在不同设备上的运行效率。

​        移动平台低端设备，关闭LOD Cross Fade，这样不会使用alpha test；

> 在移动平台的低端设备上，关闭LOD Cross Fade可以优化性能，主要是通过减少渲染阶段的计算负担。LOD（Level of Detail）技术用于根据物体与摄像机的距离动态选择不同的模型细节，以节省资源。当开启LOD Cross Fade时，模型在不同细节级别之间平滑过渡，通常会涉及到使用alpha测试来处理透明度，这在渲染时会增加额外的计算量。
>
> 通过关闭此功能，可以避免alpha测试对性能的影响，因为alpha测试会导致GPU在渲染时需要额外处理透明像素。而在低端设备上，硬件资源有限，关闭LOD Cross Fade能够降低渲染复杂性，提高帧率，增强整体游戏的流畅度。这意味着在低端设备上，虽然视觉效果可能有所牺牲，但性能优化会为玩家带来更好的体验。

​        使用前向渲染时，设置Additional Lights 为 Disabled, or Per Vertex；
​        关闭Soft Shadows或者减小Quality；

​      Universal Renderer asset（渲染实例）：

​        开启Native RenderPass（Vulkan, Metal or DirectX 12 graphics APIs）；
​            URP自动减少将渲染纹理复制到内存和拷贝出去的频率，减少urp使用的内存量；
​        使用Forward or Forward+ rendering path时， 设置Depth Priming Mode：
​            对于pc或者主机平台， 设为 Auto or Forced；
​            对于移动平台，设为Disabled；
​        设置Depth Texture Mode 为 After Transparents：
​            避免在不透明通道和透明通道之间切换渲染目标；

​      其他：

​        避免使用Complex Lit shader，或者关闭Clear Coat；
​        低端设备，静态对象使用Baked Lit shader，动态对象使用Simple Lit shader；
​        使用Screen Space Ambient Occlusion (SSAO)，注意相关设置；

### 2. URP概念：

##### 1. URP Asset：

显示所有属性；

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\e0f92d80bc954e82b2950f9f6e5045ff.png)

Rendering:

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\096b7befcfac4a7cba81535cb313b80b.png)

​    Depth Texture：

​        创建 _CameraDepthTexture，作为所有相机默认的depth texture；

​    Opaque Texture：

​        创建_CameraOpaqueTexture ，相当于GrabPass；

> “关闭Opaque Texture”的意思是禁用不透明纹理的功能，这通常涉及到图形渲染中的快照处理。在一些渲染管线中，Opaque Texture可以用于捕捉场景中所有不透明物体的视觉信息，类似于GrabPass的功能。这种快照可以在后续的渲染过程中被使用，比如在后处理效果中或用于反射等特效。
>
> ------------
>
> 关闭这个功能的作用主要有两个方面：
>
> 1. 性能优化：禁用不透明纹理可以减少额外的渲染开销，尤其是在不需要这个快照的情况下，从而提升性能。
>
> 2. 避免视觉问题：在某些情况下，使用Opaque Texture可能会导致视觉上的不一致或艺术效果的不符合。因此，在不需要时将其关闭可以帮助保持画面的清晰度和一致性。
>
> 总的来说，关闭Opaque Texture是为了优化性能和确保渲染效果的质量。

​    Opaque Downsampling：

​        降采样设置；

​    Terrain Holes：

​        关闭时，移除所有Terrain hole Shader变体；

​    SRP Batcher：

​        对于使用相同 Shader 的不同材质球非常有用；

​        加上cpu渲染提交，优先于dynamic batching；

​        需要 Shader 兼容SRP；

​    Dynamic Batching：

​        自动合并使用相同材质球的较小的mesh；

​        适合于不支持 GPU instancing 的设备，否则应该关闭；

​    Debug Level：

​        Profiling: 给FrameDebugger提供更加详细的信息标签；

​    Store Actions：

​        丢弃或存储pass的渲染目标；

​        开启时内存带宽显著增大（移动平台和基于瓦片的gpu）；

​        Auto：默认为Discard，如果检测到任何注入pass，则Store；

> 设置Store Actions为Auto或Discard的作用主要与数据的存储和处理策略相关。具体来说：
>
> 1. **Auto**：当选择Auto时，系统会自动存储和处理交易或操作的数据。这意味着如果发生错误或需要回滚，系统会保留必要的数据，允许恢复或进行进一步分析。
>
> 2. **Discard**：选择Discard时，系统将不保留任何交易或操作的数据。这通常用于那些不需要记录的测试或临时操作，确保不会占用存储空间或影响性能。
>
> 总体来说，这个设置帮助管理和优化数据存储策略，根据不同的业务需求选择是否保留操作记录。

Quality：

​    HDR：
​        开启HDR，图像中最亮的值可超过1，用于获得大的范围光照计算或bloom effects;

​     HDR Precision：

​        颜色缓冲的精度；

​    Anti Aliasing (MSAA)：

​        多重采样抗锯齿；

​        移动平台不支持 StoreAndResolve 存储操作，若开启Opaque Texture，则忽略MSAA；

​    Render Scale：

​        缩放渲染目标分辨率；

​    Upscaling Filter：

​        放大渲染结果时使用的滤波器；

​    LOD Cross Fade：

​        关闭时，unity移除所有LOD cross-fade shader变体；

Lighting：

​    关闭不需要的选项，会移除相关shader变体，提供性能和减少构建时间；

​    Main Light：

​        影响主要的平行光，即被设置为Sun Source或这最亮的平行光；

​    Cast Shadows：

​        主光源是否投射阴影；

​    Shadow Resolution：

​        控制shadow map texture大小，如果内存和渲染时间是瓶颈，则减小；

​    Light Probe System：

​        Light Probe Groups (Legacy)：与内置渲染管线相同；

​        Probe Volumes: 

​            Memory Budget：控制烘培全局照明贴图的大小；

​            SH Bands：L2提供更精确的结果，但更耗资源；

​    Additional Lights：

​        额外光源相关设置；

​    Use Rendering Layers：

​        光源只影响指定layer的对象；

​    Mixed Lighting：

​        开启后构建会包含混合光照的shader变体；

Shadows：

​    Max Distance：

​        从相机开始，unity渲染阴影的最远距离；

​    Working Unit：

​        [Shadow Cascades ](https://docs.unity3d.com/Manual/shadow-cascades.html)距离单位；

​    Cascade Count：

​        增加会影响性能，作用于main light；

​    Split 1：

​        级联1结束或者级联2开始的距离；

​    Last Border：

​        设置阴影淡出距离，在Max Distance，阴影淡出到零；

​    Depth Bias：减少 [shadow acne](https://docs.unity3d.com/Manual/ShadowPerformance.html)；

​    Normal Bias：减少 [shadow acne](https://docs.unity3d.com/Manual/ShadowPerformance.html)；

​    Soft Shadows：

​        对阴影纹理额外处理，使阴影看起来平滑；

​        性能影响大；

​        Quality：

​            Low：适合移动平台；

​            Medium：适合pc平台；

​    Conservative Enclosing Sphere：

​        新项目建议开启，该选项会影响shadows cascade distances计算；

​        启用可能提高性能；

Post-processing：

​    调整全局后处理设置；

​    渲染实例上的设置：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\f7e43b4fda1d440d8d1e00be3aa1b698.png)

​        关闭的话，在构建时也会排除后处理的shader以及贴图；

​        Data：包含对后处理需要的shader和贴图的引用；

​    Grading Mode：

​        使用Low Dynamic Range；

​    LUT Size：

​        设置urp色彩分级使用的 look-up texture 大小；

​        默认32就好；

​    Fast sRGB/Linear Conversions：

​        快速伽马空间和线性空间转换；

Volumes：

​    Volume Update Mode：

​        Every Frame；

​        Via Scripting；

​    Volume Profile：

​        设置场景默认的Volume Profile；

##### 2. URP Global Settings：

Default Volume Profile：

​    所有场景使用的Default Volume；

​    urp显示所有可覆盖的属性；

Rendering Layers (3D)：

​    用于光照的layer；

Shader Stripping：

​    Shader Variant Log Level：

​    Strip Debug Variants：启用后就不能使用Rendering Debugger；

​    Strip Unused Post Processing Variants：

​        只保留Volume Profiles用到的 Shader Variants；

​    Strip Unused Variants：

​        更强力的裁剪，有可能多裁剪了；

##### 3. Universal Renderer（渲染实例）：

Rendering Paths：

​    Forward Rendering Path
​    Forward+ Rendering Path
​    Deferred Rendering Path

3种渲染路径比较：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\a1f16eb3b3594cb095a5b5712d79ac91.png)

对图片的解读

> 这张表格列出了不同渲染路径（Forward、Forward+ 和 Deferred）在一些特性上的比较。以下是对每个特性的解析：
>
> 1. **每个对象的实时光源最大数量**：
>    - **Forward**：每个对象最多支持9个光源（1个主光源和8个附加光源）。
>    - **Forward+ 和 Deferred**：光源数量无限制，但每个相机的限制仍然适用。透明对象使用Forward渲染路径，最大光源数量为9。
>
> 2. **每像素法线编码**：
>    - **Forward 和 Forward+**：不进行编码，提供准确的法线值。
>    - **Deferred**：提供两种选项：
>      - G-buffer中的法线量化（准确性降低，性能提升）。
>      - 八面体编码（提供准确法线，可能对移动GPU有显著性能影响）。
>
> 3. **MSAA（多重采样抗锯齿）**：
>    - **Forward 和 Forward+**：支持。
>    - **Deferred**：不支持。
>
> 4. **顶点光照**：
>    - **Forward**：支持。
>    - **Forward+ 和 Deferred**：不支持。
>
> 5. **相机堆叠**：
>    - **Forward 和 Forward+**：支持。
>    - **Deferred**：有限制，Unity仅使用Deferred渲染路径渲染基础相机，所有叠加相机使用Forward渲染路径。
>
> 这张表格为开发者在选择渲染路径时提供了重要的参考信息，帮助他们根据项目需求做出合适的选择。

每个相机实时光源限制：

​    Desktop and console platforms: 1 Main Light, and 256 Additional Lights；

​    Mobile platforms: 1 Main Light, and 32 Additional Lights；
​    OpenGL ES 3.0 and earlier: 1 Main Light, and 16 Additional Lights；

Universal Renderer asset 属性设置：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\e8c441df80d847dea76ad2c54e180cae.png)

Filtering：

​    Opaque Layer Mask：渲染不透明物体的Layer；

​    Transparent Layer Mask：渲染透明物体的Layer；

Rendering：

​    Rendering Path：

​    Depth Priming Mode：

​        使用一个 depth prepass 决定后续是否需要执行片元着色器；

​        Disabled：

​        Auto：

​            如果有一个渲染pass需要depth prepass，unity执行它并进行深度测试；

​            On Android, iOS, and Apple TV，不会启用；

​        Forced：基于瓦片的延迟渲染会忽略这个属性；

​    Depth Texture Mode：

​        指定在渲染管线什么阶段拷贝场景深度到深度贴图；

​            After Opaques：在不透明物体渲染pass之后；

​            After Transparents：在透明物体渲染pass之后；

​            Force Prepass：执行depth prepass来生成深度贴图；

​        在移动平台，After Transparents 可以显著降低内存带宽：

​            Copy Depth pass会导致渲染目标在不透明pass和透明pass之间进行切换；

​            渲染目标变更前，unity将颜色缓冲区的内容保存到主内存中；

​            当执行完Copy Depth pass，再将主内存中的颜色值加载到颜色缓冲区中；

​            在使用MSAA 时，更严重，因为还需要额外保存和加载MSAA数据；

​    Native RenderPass：

​        使用URP's Native RenderPass API；

​        OpenGL ES 上没效果；

​    Shadows：

​        Transparent Receive Shadows：在透明物体上渲染阴影；

​    Overrides：

​        Stencil：进行目标测试；

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\912ceb84de754036a6a2e87be3afb38c.png)

> ​    Stencil（模板测试）是一种图形渲染技术，广泛应用于计算机图形学中，特别是在图形处理单元（GPU）中。其主要原理和作用如下：
>
> ### 原理
>
> 1. **模板缓冲区**：
>    - Stencil 测试使用一个叫做模板缓冲区的内存区域，通常与颜色缓冲区和深度缓冲区并行存在。
>    - 每个像素对应一个模板值，该值可以在渲染过程中被修改。
>
> 2. **测试过程**：
>    - 在绘制每个像素之前，Stencil 测试会检查该像素的模板值是否满足特定条件（例如，是否等于、是否大于等）。
>    - 根据测试的结果，可以决定该像素是否被更新、是否绘制。
>
> 3. **模板操作**：
>    - 如果一个像素通过了模板测试，可以执行特定的操作，如更新模板值、更新颜色缓冲中的颜色，或保留原值。
>    - 常见的模板操作包括：增量、减量和替换。
>
> ### 作用
>
> 1. **遮罩**：
>    - Stencil 测试可以用作一种遮罩技术，可以控制哪些像素应该被渲染到屏幕上。例如，可以用它来创建复杂的形状或实现特殊效果（如阴影）。
>
> 2. **复杂效果**：
>    - 可以用于合理组织渲染顺序、绘制透视效果、实现反射等需要层次剔除的效果。
>
> 3. **交互和效果**：
>    - 通过动态改变模板缓冲区的值，可以实现场景中对象的高亮、选择或特殊效果，为用户提供交互体验。
>
> ### 应用实例
>
> - **阴影映射**：在渲染阴影时，可以使用 Stencil 测试来确定哪些区域被光源照亮，哪些区域在阴影之中。
> - **GUI 元素**：在图形用户界面中，可以使用 Stencil 测试来确保只渲染特定区域的元素，而不影响其他区域。
>
> 总之，Stencil 测试是图形渲染中一个强大而灵活的工具，能够实现多种复杂视觉效果，使开发者能够更好地控制渲染过程。

Compatibility：

​        Intermediate Texture：

​            是否通过中间纹理渲染；

​            Auto：需要时开启；

​            Always：在某些平台上对性能有大的影响；

​    Renderer Features：

​        Renderer Features列表；

##### 4. URP 延迟渲染路径：

启用延迟渲染：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\9277a99c08124a8b94ed449b6c0cf3ba.png)

延迟渲染额外要求：

​    Shader Model 4.5；

​    不支持OpenGL 和 OpenGL ES API；

如果设备不满足延迟渲染条件，自动回退到 Forward Rendering Path；

延迟渲染实现细节：

​    首先对每个GameObject 执行一次 G-Buffer pass，将材质球属性存储到G-Buffer中；

​    然后在屏幕空间中使用 Lighting pass，基于G-Buffer中的信息进行光照计算；

> 在Unity中，URP（Universal Render Pipeline）的延迟渲染路径指的是一种渲染方法，它允许游戏先构建场景图，然后根据需要渲染它们。启用延迟渲染后，场景中的物体不会立即渲染，而是延迟到必要的时机，这样可以提高性能。
>
> 具体来说，启用延迟渲染后，以下是一些关键点：
>
> 1. 场景中的物体先被处理并存储在内存中的缓冲区里。
> 2. 当相机移动到这些物体所在的视锥体（Frustum）内时，才进行渲染。
> 3. 这种方法可以减少不必要的渲染计算，从而提高效率。
>
> 请注意，根据实际应用，启用延迟渲染可能会影响渲染质量和性能。在使用前应谨慎评估其对游戏或应用的具体影响。

G-buffer 中内容分布：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\0922999674974358972ffb3daa638a7b.png)

对图片的解读

> 这张图片展示了一个图形渲染系统的结构，主要涉及颜色通道和渲染目标的组织。以下是对图片的详细解读：
>
> ### 1. 颜色通道
>
> - **R, G, B, A**：图像的颜色通道分别为红色（R）、绿色（G）、蓝色（B）和透明度（A）。每个通道占用8位，总共32位（4 x 8 bits），用于表示颜色和透明度信息。
>
> ### 2. 渲染目标
>
> 在图形渲染中，渲染目标是指在渲染过程中生成的图像数据。图片中列出了几个主要的渲染目标：
>
> - **Albedo (sRGB)**：表示表面的基本颜色信息，通常用于纹理映射。
> - **Specular**：反射光泽度，影响物体表面的光泽效果。
> - **Normal**：法线信息，用于计算光照和表面细节。
> - **Emissive/GI/Lighting**：自发光、全局光照和其他光照信息。
> - **MaterialFlags**：材质标志，指示材质的特性。
>
> ### 3. 可选渲染目标
>
> 这些是可选的渲染目标，通常用于增强渲染效果：
>
> - **ShadowMask**：阴影掩码，用于处理阴影效果。
> - **Rendering Layer Mask**：渲染层掩码，指示哪些层需要被渲染。
> - **Depth as Color**：深度信息以颜色形式表示，通常用于后期处理。
>
> ### 4. 深度和模板目标
>
> - **DepthStencil**：深度和模板目标，用于存储深度信息和模板测试，确保正确的图形渲染顺序。
>
> ### 总结
>
> 这张图片概述了图形渲染中颜色通道和渲染目标的结构，强调了不同信息在渲染过程中的重要性。通过合理组织这些数据，可以实现更高质量的图形效果。

延迟渲染路径事件序列：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\3ce2706ff54846168bc0fb0cc6164bb4.png)

​    Depth, or depth and normal prepass：

​        对于不支持延迟渲染的材质球，需要执行depth prepass 或 depth and normal prepass；

​        如果有 SSAO Renderer Feature，则需要执行 depth and normal prepass：

​            SSAO 使用屏幕空间深度和法线缓冲来计算环境遮挡；

> Screen Space Ambient Occlusion (SSAO)**(屏幕空间环境光遮罩)**是一种光照技术，它在游戏图形渲染中用于模拟真实世界中物体阴影部分的视觉效果。这种技术不是直接计算每个像素的深度信息，而是基于屏幕空间对周围环境进行采样，通过分析相邻像素之间的光照差异来估算暗部区域。简单来说，SSAO会模拟光源如何被周围的几何形状阻挡，使得图像看起来更自然、有深度。
>
> SSAO渲染器特征包括：
>
> 1. **实时性**：由于计算是在屏幕上进行的，而不是逐帧在3D模型上，所以它可以提供快速的性能，适用于需要实时反馈的游戏场景。
> 2. **硬件加速**：许多现代GPU都内置了优化的SSAO单元，能够利用并行计算提高效率。
> 3. **可调整性**：SSAO强度、分辨率、模糊程度等参数可以调整，允许艺术家精细控制阴影的效果。
> 4. **内存效率**：因为只依赖屏幕空间数据，所以占用的系统资源相对较少。

​    Forward-only Pass：

​        不支持延迟渲染的pass，如：

​        复杂的shader，G-buff无法存储足够多的材质球信息；

​        不需要计算实时光的shader；

​        自定义的shader，不支持延迟渲染；

##### 5. Forward+ Rendering Path：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\e0352311954c4ebf8aa0737489e6613f.png)

相比Forward Rendering Path，Forward+ Rendering Path具有如下优势：

​    每个对象不限光源数量；

​    支持2个以上reflection probes混合；

​    使用ECS时支持多个灯光；

​    程序绘制更灵活；

使用 Forward+ Rendering Path，Unity忽略一些 URP Asset 属性：

​    Main Light：默认Per Pixel；

​    Additional Lights：默认Per Pixel；

​    Additional Lights > Per Object Limit：忽略；

​    Reflection Probes > Probe Blending：默认开启；

##### 6. Pre-built effects (Renderer Features)：

​    添加内置的效果到 URP Renderer；

添加 Renderer Feature 到 Renderer：

​    选择一个Renderer：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\a692be0389e545bd843ae5e86566b7aa.png)

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\876b1f8ff2bd4733a0a562d68c5069da.png)

### 3. Renderer Feature：

对于透明材质的要选transparent，否则选择Opaque

![image-20241011220611998](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241011220611998.png)



##### 1. Render Objects Renderer Feature：

渲染所有的具有指定 Layer 的游戏对象，可指定时机和材质球；

使用 Render Objects 渲染位于物体背后的轮廓：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\5e26616e6543469287df727cb2471c2e.gif)

​    使用两个Render Objects，一个渲染位于物体后面的轮廓，一个正常渲染；

​    1. 指定一个Layer，用于渲染该类对象：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\80548a2f8e124023ae689b5540002cec.png)

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\7753d95b352d4705a4926ae27df46394.png)

​    2. 设置Renderer > Opaque Layer Mask，默认不渲染该Layer的物体：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\962ad2c5598f42f08c96c04f4f62a479.png)

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\8e2bf0eba41b49f9bacfd6224a47e8d8.png)

​    3. 添加Render Objects，使用指定的材质球渲染位于物体后面的轮廓：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\8d4db467bb28447da9367ebce0fc60a8.png)

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\4d801954c58a46e5a23e3e4e5f63312d.png)

​    4. 添加Render Objects，正常渲染指定Layer的游戏对象：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\09e1c91a93ef4f61a0fa56ae2212451b.png)

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\a924c6a5b6574c86b5107be204674763.png)

Render Objects Renderer Feature属性：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\6e49520bdfb945928db8b20d3b898976.png)

##### 2. Decal Renderer Feature：(这部分功能我没有找到）

将指定的材质球（贴花）投射到场景中其他的物体上；

贴花与场景中的灯光相互作用，并包裹在mesh周围；

使用贴花渲染功能：

1. 添加 Decal Renderer Feature 到 URP Renderer：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\51ce7f558686431fa8b7ce369726bdfc.png)

2. 创建一个材质球，使用Graphs/Decal shader，设置贴图：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\cf5c489842744c59825ab6571945ca58.png)

3. 创建一个 game object，添加Decal Projector 组件：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\ac00ccf9706d45c39b8b41c9e38ec997.png)

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\6b76c03021a24ad58363ef798c2731a4.png)

贴花不能作用在透明物体上；

Decal Renderer Feature属性：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\9ed9305ee1d045d1b1f287f5efde170d.png)

Technique：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\c588581f7b0a463e92620059f4406aee.png)

​    当前有两种技术渲染贴花；

​    Automatic：

​        根据构建平台，以及 Accurate G-buffer normals 选项；

​    DBuffer：

​        将贴花渲染到 Decal buffer (DBuffer) 中，随后在渲染不透明物体时将贴花覆盖在上面；

​        Surface Data：

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\20d1999ea4404cfda97a368883dbc953.png)

​            指定将贴花哪些表面属性与底层网格混合；

​        限制条件：

​            不支持OpenGL 和 OpenGL ES API；

​            需要先执行 DepthNormal prepass，在基于瓦片的gpu上性能低；

​            不适用于粒子和地形细节；

​    Screen Space：

​        在渲染完不透明物体后，使用从深度贴图构建的法线 或者 G-buff 中的法线 渲染贴花；

​        该技术只支持 normal blending；

​            延迟渲染开启Accurate G-buffer normals时，法线混合不支持；

​        Normal Blend：

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\bdf48f86841a49069584b36102f61d24.png)

​            从深度贴图构建法线时，设置深度贴图的采样次数；

​        Max Draw Distance：

​            unity渲染贴花距离摄像机的最大距离；

Decal Projector component：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\5a99f9aca4854b6a87612cd8600817c9.png)

​    投影贴花到场景中其他对象上；

​    必须使用具有 Decal Shader 的材质球；

​    渲染贴花时，由于使用 Material property blocks，不支持SRP Batcher，为了减少draw call，将多个贴花使用的贴图合并为一个图集，然后使用相同的材质球，开启gpu instance渲染；

##### 3. Screen Space Ambient Occlusion (SSAO) Renderer Feature：

环境遮挡效果使折痕、孔洞、交叉点和彼此接近的表面变暗；

在渲染不透明物体前先渲染SSAO纹理，然后在渲染不透明物体时使用该纹理；urp提供的shader支持该功能；

添加 SSAO Renderer Feature：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\b501cc4eefe148d68ccb8ef859039bbc.png)

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\2c5c6c187eb44a3284f2610080540bea.png)

Method：

​    噪声类型；

​    性能影响微不足道；

Intensity：

​    变暗强度；

Radius：

​    法线贴图采样半径；

​    性能影响大；

Falloff Distance：

​    应用SSAO 距离相机最大距离；

Quality：

​    Source：法线来源，SSAO 使用法线来计算表面暴露程度；

​    Downsample：渲染 SSAO 纹理尺寸缩小为 1/2；

​    After Opaque：在渲染不透明物体后再计算和应用 SSAO效果；

​        当使用Depth 作为 Source时，直接使用已存在的深度值，无需执行depth prepass；

##### 4. Screen Space Shadows Renderer Feature：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\158693e8c40b4db68fc34e390585965f.png)![img](D:\DeskTop\FileSum\UnityClassResources\笔记\6cbdb7e29c524d20be0910a2a19213f5.png)

​    屏幕空间阴影，使用单张贴图来计算和渲染主平行光阴影，避免使用多张阴影级联纹理；

​    前向渲染使用该技术可能导致渲染加快，因为不需要访问多张阴影级联纹理；

​    缺陷：

​        需要执行 depth prepass 来构建深度纹理；

​        需要内存来创建 screen-space shadows texture；

##### 5. Full Screen Pass Renderer Feature：

使用材质球渲染全屏效果，例如插图和模糊；

创建一个自定义后处理效果：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\21219869085241e8918ae2c3019d71de.png)

### 4. URP中的渲染：

##### 渲染流程：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\500a81012eb64414ba47bc937c6da6d0.png)

对图像的解读

> 这个图示展示了一个渲染循环的结构，主要分为两个部分：渲染循环（Rendering Loop）和相机循环（Camera Loop）。以下是各个部分的知识点和原理解释：
>
> ### 1. 渲染循环（Rendering Loop）
>
> - **BeginFrameRendering**: 这是渲染循环的开始，通常用于初始化渲染状态和设置帧相关的参数。
> - **EndFrameRendering**: 渲染循环的结束，通常用于清理和提交渲染结果。
>
> ### 2. 相机循环（Camera Loop）
>
> - **BeginCameraRendering**: 开始相机渲染，设置与相机相关的参数。
> - **Setup Culling Parameters**: 设置剔除参数，决定哪些对象在渲染时需要被剔除，以提高性能。
> - **Culling**: 实际执行剔除操作，排除不在视野内的对象，减少渲染负担。
> - **Build Rendering Data**: 构建渲染数据，准备需要渲染的对象和其属性。
> - **Setup Renderer**: 设置渲染器，配置渲染器的状态和参数。
> - **Execute Renderer**: 执行渲染操作，将准备好的数据渲染到屏幕上。
> - **EndCameraRendering**: 结束相机渲染，清理与相机相关的状态。
>
> ### 3. 图例说明
>
> - **Pipeline Callback**: 表示可以被重写的回调函数，允许开发者自定义渲染流程。
> - **Overridable Function**: 可重写的函数，允许在特定情况下修改默认行为。
> - **Internal Function**: 内部函数，通常是系统内部使用，不建议外部调用。
>
> ### 原理总结
>
> 这个渲染循环的设计旨在高效地管理渲染过程，通过分层结构（渲染循环和相机循环）来组织渲染任务。相机循环专注于与相机相关的渲染操作，而渲染循环则负责整体的渲染框架。通过剔除不必要的对象和优化渲染数据的构建，可以显著提高渲染性能。

Camera loop：

Setup Culling Parameters：

​    配置渲染对象，灯光和阴影的剔除参数；

Culling：

​    使用上一步的参数挑选出摄像机可见的对象，阴影投射和灯光；

Build Rendering Data：

​    构建渲染数据；

Setup Renderer：

​    构建一个pass列表；

Execute Renderer：

​    执行队列中的渲染pass；

##### Rendering Layers：

​    用于配置灯光只影响指定的游戏对象；

​    还可以在光源上单独配置阴影投射Layer：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\de743a53e7db4300bedfbb8c344b1182.png)

​    贴花也支持配置Layer；

缺陷：

​    不支持 OpenGL 和 OpenGL ES API；

### 5. urp中的光照：

##### Light component：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\8226ee47eb9a492d8b9ec2e1dafa7b5b.png)

​    Bias：避免错误的自阴影；

​        urp渲染管线不明显，使用内置渲染管线很容易观察到相关现象：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\4329164d9f1842868cfc4867cc479a56.png)

​        （未添加Bias）

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\7d11e6a526194048b4c28b3c851362f3.png)

​        （添加Bias）

​        Depth：

​            渲染阴影纹理时深度添加偏移，以便相同的位置不被自身遮挡；

​        Normal：

​            感觉是沿着法线缩小物体，使投射的阴影范围缩小；

##### Lighting Mode：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\77197862827f4cec8f77872246cfb4ea.png)

​    决定场景中混合灯光的行为；

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\f574f208e1f846b4a09ed5c820cccd03.png)

unity支持的Lighting Modes：

​    Baked Indirect；

​    Subtractive；

​    Shadowmask；

Baked Indirect：

​    **将间接光烘培到光照贴图中，其他走实时光照处理；**

​    对于动态的游戏对象：

​        接收实时直接光照；

​        使用Light Probes接收间接光照；

​    对于静态的游戏对象：

​        接收实时直接光照；

​        使用 lightmaps 接收间接光照；

Shadowmask：

​    类似 Baked Indirect ，走实时直接光照和烘培间接光照；

​    区别在于渲染阴影：

​        让unity在运行时结合实时和烘培阴影，从而在远距离渲染阴影；

​        需要一张额外的 lightmap，即 shadow mask，并在光照探针里存储额外的信息；

​    在所有照明模式中提供最高保真度的阴影，但消耗最高的性能和内存；

​    **适合渲染逼真的场景，其远处的对象是可见的，如开放世界；**

​    设置Shadowmask品质：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\569375e8a055454d88396c6fa59aaee4.png)

​    Distance Shadowmask：

​        对于动态游戏对象：

​            在 Shadow Distance 内使用实时阴影；

​            在 Shadow Distance 外使用 Light Probes 接收静态物体的烘培阴影；

​        对于静态游戏对象：

​            在 Shadow Distance 内使用实时阴影；

​            在 Shadow Distance 外使用 shadow mask 接收静态物体的烘培阴影；

​    Shadowmask：

​        对于动态游戏对象：

​            在 Shadow Distance 内接收动态物体的实时阴影；

​            在 Shadow Distance 内外使用 Light Probes 接收静态物体的烘培阴影；

​        对于静态游戏对象：

​            在 Shadow Distance 内接收动态物体的实时阴影；

​            在 Shadow Distance 内外使用 shadow mask 接收静态物体的烘培阴影；

Subtractive：

​    烘培直接和间接光照；

​    将静态对象的阴影烘培到光照贴图；

​    主平行光提供实时阴影给动态物体；

​    适用于低端设备；

​    对于动态游戏对象：

​        接收实时直接光照；

​        使用 Light Probes 接收间接光照；

​        接收 被主光源照亮的动态物体投射的实时阴影，直到Shadow Distance；

​        使用Light Probes接收静态物体投射的阴影；

​    对于静态对象：

​        使用lightmaps接收直接关和间接光；

​        使用lightmaps接收来自静态物体的烘培阴影；

​        接收 被主光源照亮的动态物体投射的实时阴影，直到Shadow Distance；

##### Universal Additional Light Data component：

​    urp用来做内部数据存储；

​    扩展和覆写标准Light组件；

##### URP 中的阴影：

urp中每盏光使用的 shadow map 数量取决于光源类型：

​    A Spot Light ：1张；

​    A Point Light：6张，每个面对应一张；

​    Directional Light：每级对应1张；

urp根据 shadow atlas 的尺寸以及 shadow map 的数量，来确定最合适的每个shadow map大小；

urp使用一个shadow map atlas给线性光，再使用一个shadow map atlas给其他的所有实时光；

##### Probe Volumes：

​    光照探针相关；

##### Reflection probes：

​    反射探针相关；

##### Lens flares：

​    镜头光晕；

### 6. Cameras：（这部分还得继续学习）

##### Universal Additional Camera Data component：

​    urp用来做内部数据存储；

​    扩展和覆盖标准相机功能；

```cs
UniversalAdditionalCameraData cameraData = camera.GetUniversalAdditionalCameraData();
```

##### Render Type：

​    Base：渲染到屏幕或渲染纹理的常规相机；

​    Overlay：在另一个相机的输出上渲染；

可将Base相机的输出与一个或多个Overlay相机结合起来，这被叫做 Camera stacking；

在场景中可以有多个 Base 相机；

Overlay 相机必须位于 Camera Stacking 系统中，有一个 Base 相机；

单独的 Overlay 相机不执行渲染流程，Overlay 相机可位于多个 Camera stacking中；

Overlay 相机只支持设置部分属性，其他的共用Base 相机；

后处理只能作用于单独的Base 相机或者Camera stacking；

##### Camera Stacking：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\2cec9a794b2341f3aa54e94352419f78.png)

##### 多个相机渲染到相同的目标：

​    可将多个Base相机和 Camera Stacks 渲染到相同的目标，实现分屏渲染效果；

​    在Base相机上设置渲染目标和渲染区域Viewport Rect；

##### 渲染到渲染纹理：

​    在urp中，渲染到纹理的相机比渲染到屏幕的相机先渲染，保证Render Texture中数据有效；

##### Motion vectors：

​    使用一个全屏纹理保存屏幕空间的运动矢量；

​    只在需要时才会去渲染运动矢量贴图；

​    使用每个像素的 rg 通道记录渲染对象在屏幕上的uv偏移；

运动矢量有两类：

Camera motion vectors：

​    单纯相机自身运动导致；

​    只需要一个全屏的pass就可以渲染相机的运动矢量；

Object motion vectors：

​    世界空间物体运动以及相机运动；

​    每个物体都需要一个专门pass来计算物体运动，同时需要考虑相机的运动；

注意mesh renderer相关设置：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\0b06c181539a40e6ba2bcf0111b63046.png)

unity渲染物体运动矢量的条件：

​    1. shader需要有一个 MotionVectors pass；

​    2. 通过如下方式渲染：

​        SkinnedMeshRenderer；

​        MeshRenderer：Motion Vectors属性不设置为 Camera Motion Only；

​        使用相关api渲染时，MotionVectorMode 不设置为 Camera；

​    3. 如下的任一条件满足：

​        1. MotionVectorMode设置为 ForceNoMotion，这时需要一个pass来清掉相机运动矢量；

​        2. 在材质球上开启MotionVectors：

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\1a66c550148540d2afdb4125b7d8d535.png)

​        3. 材质球上关闭MotionVectors：

​            模型位置矩阵在当前帧与上一帧不匹配；

​            有 skeletal animation；

MeshRenderer 属性 Motion Vectors：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\374c297eeac347058f3aedb57ca4607b.png)

​    Camera Motion Only：

​        只使用相机运动矢量，不执行每个对象各自的motion vector pass；

​    Per Object Motion：

​        每个对象执行各自的motion vector pass；

​    Force No Motion：

​        每个对象执行各自的motion vector pass，通过一个shader变量指定渲染值为0；

urp 在 BeforeRenderingPostProcessing 事件渲染运动矢量；

urp通过2步渲染运动矢量纹理：

​    1. 通过一个全屏pass渲染相机运动矢量；

​    2. 执行需要渲染 motion vector 对象的 motion vector pass；

使用 motion vector texture：

​    在自定义Renderer Feature中，在AddRenderPasses回调中

​    添加ScriptableRenderPassInput.Motion标志；

​    shader中：
​         //声明贴图
​        TEXTURE2D_X(_MotionVectorTexture);
​        SAMPLER(sampler_MotionVectorTexture); 
​        //采样
​        SAMPLE_TEXTURE2D_X(_MotionVectorTexture, sampler_MotionVectorTexture, uv);

##### 抗锯齿：

​    量化数据时，不能真实反应实际信息，存在锯齿；

摄像机后处理抗锯齿分类：

​    ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\4517a3d1ae474c549bb8a5240d70e952.png)

Fast Approximate Anti-aliasing (FXAA)：

​    使用一个全屏pass平滑像素边缘；

​    最省资源；

Subpixel Morphological Anti-aliasing (SMAA)：

​    寻找图像边缘模式并依据模式混合边缘像素，比FXAA效果好；

Temporal Anti-aliasing (TAA)：

​    在多个帧中平滑边缘，但在物体快速移动时容易出现残影；

​    使用运动矢量；

​    开启TAA后，不能使用如下功能：

​        Multisample anti-aliasing (MSAA)；

​        Camera Stacking；

​        Dynamic Resolution；

硬件抗锯齿：

Multisample Anti-aliasing (MSAA)：

​    通过采样每个像素深度和模板值来计算最终值；

​    解决空间锯齿问题，尤其三角形边缘锯齿问题；

​    不能解决shader锯齿问题，如高光和纹理锯齿；

在大多数硬件上，MSAA比其他形式的抗锯齿更耗费资源；

在瓦片gpu上，没有后处理抗锯齿和自定义render features时，MSAA 更廉价；

MSAA是一种硬件抗锯齿方式，可与其他抗锯齿结合使用（除了TAA）；

在不支持 StoreAndResolve 保存操作的移动平台，如果开启Opaque Texture， unity忽略MSAA；

### 7. 后处理：

urp包含了集成的后处理实现；

不支持 OpenGL ES 2.0；

添加后处理到场景：

​    1. 相机开启后处理：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\81f35b9c2a5642caa2b960dcd607c649.png)

​    2. 创建一个Game Object，添加 Volume 组件：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\69980642f1c34f4f8d1581f7be032d1e.png)

​    3. 创建 Volume Profile：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\260eee5ca04f48889f119c2bea84d241.png)

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\4c916b62fd074bfea341ea9893f424c1.png)

​    4. 添加 Volume Override：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\1ecddb652d074168bf9e7ce022a40c29.png)

##### Volumes：

使用 Volumes 应用后处理到部分或整个场景；

一个gameObject上可包含多个 Volume 组件；

urp遍历场景中所有激活的 Volume 组件，确定每个 Volume 组件对最终设置的权重；

使用相机与 Volume 组件的属性来确定权重，然后根据权重在多个 Volume 进行插值确定最终属性

两种 volume 类型：

​    Global：无论相机在哪都生效；

​    Local：相机靠近碰撞体时才生效；

每个 Volume 组件引用一个 Volume Profile；

有两个默认的全局 Volume ：

​    URP Global Settings：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\8085792714294a1bb62d92b59b4801f1.png)

​        列出所有的 Volume Overrides，可修改属性值，但是不能禁用和删除 Overrides；

​    每个品质对应 URP Asset：

​        ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\ba0a5952dc74497299a766865a58b738.png)

​        可修改属性，并添加或删除 Overrides；

Volume 组件：

​    Mode：该Volume影响相机的方式；

​        Global：Volume 无边界，影响场景中的每个相机；

​        Local： 使用 Collider 来指定边界，只影响范围内的相机；

​    Blend Distance：从Collider 开始，混合的最远距离；

​    Weight：使用这个乘以urp使用相机位置和Blend Distance计算的值，获得最终的权重；

​    Priority：多个Volume overrides存在时，使用Priority最大的；

Effect List：

​    各种后处理效果使用；

​    使用时参阅各个参数设置，注意性能影响；

### 8. Shaders and Materials：

内置渲染管线光照计算的shader不兼容urp，Unlit shader兼容urp；

SRP Batcher 兼容：

​    在一个名为 UnityPerMaterial 的 CBUFFER 中 声明所有的材质球属性；

​    在一个名为 UnityPerDraw 的 CBUFFER 中 声明所有的内置引擎属性；

​        unity_ObjectToWorld ，unity_WorldTransformParams

##### urp中的光照模型：

​    Physically Based Shading，基于物理的渲染；

​    Simple Shading，经典光照模型；

​    Baked Lit Shading，烘培光照；

​    No lighting，不计算光照；

Physically Based Shading：

​    遵守能量守恒，使用微面元模型；

​    光照射到表面时，部分反射，部分折射；

​        直接反射的光称为镜面反射；

​        折射到物体内部的光又经过反射后最终散射出物体，称为漫反射；

​    金属物体，表面会吸收和改变光；非金属物体反射部分光；

​    代表 shader：

​        Lit；

​        Particles Lit；

Simple Shading：

​    使用 Blinn-Phong 光照模型，非能量守恒，用于低端设备或者风格化渲染；

​    代表 shader：

​        Simple Lit；

​        Particles Simple Lit；

Baked Lit Shading：

​    不计算实时光；

​    代表 shader：

​        Baked Lit Shading；

No lighting：

​    不计算实时光和烘培光；

​    代表 shader：

​        Unlit；

​        Particles Unlit；

Material Variants：

​    类似于 Prefab Variants；

urp不自持 Surface Shaders；

##### 内置 shader 兼容urp和 SRP Batcher：

​    1. 替换 **CGPROGRAM,** **ENDCG** 为 **HLSLPROGRAM,** **ENDHLSL；**

​    2. 替换 include语句为：

​        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

​    3. 添加 "RenderPipeline" = "UniversalPipeline" shader tag；

​    4. v2f 结构体的名字替换为Varyings（urp命名约定）：

```cpp
struct Varyings



{



    float4 positionHCS : SV_POSITION;



    float2 uv: TEXCOORD0;



};
```

​    5. 在 include 语句后，struct Varyings 语句前，声明输入struct Attributes：

```cpp
struct Attributes
{
    float4 positionOS: POSITION;
    float2 uv : TEXCOORD0;

};
```

​    6. 使用 TransformObjectToHClip 将顶点从对象空间转到裁剪空间：

```cpp
o.positionHCS = TransformObjectToHClip (i.positionOS.xyz);
```

​        urp在变量后面添加后缀表示space；

​    7. 使用 CBUFFER 包含材质球的属性：

```cpp
CBUFFER_START(UnityPerMaterial)
    float4 _Color;
    sampler2D _MainTex;
CBUFFER_END
```

​        为了 SRP Batcher 兼容，同一个 subshader 有多个pass时，得使用相同的cbuff；

​    8. urp shader不支持 fixed 类型；

​    9. 让纹理采样支持 tiling and offset：

​        urp 约定主纹理名 _BaseMap；

​        添加 ShaderLab 属性 [MainTexture] 和 [MainColor]；

​        在 CBUFFER 前声明纹理和采样器：

```cpp
            TEXTURE2D(_BaseMap);        //Texture2D _BaseMap
            SAMPLER(sampler_BaseMap);   //SamplerState sampler_BaseMap
```

​        在 CBUFFER 中声明纹理缩放和偏移变量：

```cpp
            CBUFFER_START(UnityPerMaterial)
                float4 _BaseMap_ST;
                float4 _Color;
            CBUFFER_END
```

​        顶点着色器中应用贴图缩放和偏移：

```cpp
            o.uv = TRANSFORM_TEX(i.uv, _BaseMap);
```

​        片元着色器采样贴图：

```cpp
            half4 texel = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, i.uv);
```

​    10. 使用模型法线：

```cpp
            struct Attributes
            {
                float4 positionOS: POSITION;
                half3 normal: NORMAL;        //NORMAL语义
                float2 uv : TEXCOORD0;
            };
            struct Varyings
            {
                float4 positionHCS : SV_POSITION;
                float2 uv: TEXCOORD0;
                half3 normal: TEXCOORD1;    //TEXCOORD1语义
            };
            Varyings vert(Attributes i)
            {
                Varyings o;
                o.positionHCS = TransformObjectToHClip (i.positionOS.xyz);
                o.uv = TRANSFORM_TEX(i.uv, _BaseMap);
                //法线变换到世界空间
                o.normal = TransformObjectToWorldNormal(i.normal);
                return o;
            }
```

​    11. 从深度贴图获取像素的世界坐标：

​        首先设置URP Asset，获取深度贴图：

​            ![img](D:\DeskTop\FileSum\UnityClassResources\笔记\a3efcd8de75d45d09d06039e1594b092.png)

​        包含头文件：

```cpp
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl"
```

​        片元着色器计算屏幕贴图uv坐标：

```cpp
            float2 UV = IN.positionHCS.xy / _ScaledScreenParams.xy;
```

​            片元positionHCS.xy = 顶点positionHCS.xy / 顶点positionHCS.w * 屏幕宽高

​            片元positionHCS.z = 顶点positionHCS.z / 顶点positionHCS.w；

​        获取深度值：

```cpp
            // Sample the depth from the Camera depth texture.

            #if UNITY_REVERSED_Z

                //反向缓冲，深度值为1到0，1代表近处物体，0代表远处物体

                real depth = SampleSceneDepth(UV);

            #else

                // Adjust Z to match NDC for OpenGL ([-1, 1])

                //这种情况下 UNITY_NEAR_CLIP_VALUE 为 -1

                real depth = lerp(UNITY_NEAR_CLIP_VALUE, 1, SampleSceneDepth(UV));

            #endif
```

​            SampleSceneDepth 返回 [0,1] 的深度值；

​            ComputeWorldSpacePosition需要NDC下的深度值：

​                D3D, 深度值范围 [0,1]；

​                OpenGL，深度值范围 [-1, 1] ；

​        计算世界位置：

```cpp
            // Reconstruct the world space positions.
            float3 worldPos = ComputeWorldSpacePosition(UV, depth, UNITY_MATRIX_I_VP);
```

##### urp shader中的辅助方法：

###### shader库位置：

​    urp：Packages/com.unity.render-pipelines.universal/ShaderLibrary/

​    srp：Packages/com.unity.render-pipelines.core/ShaderLibrary/

###### 坐标变换相关函数（ShaderVariablesFunction.hlsl）：

float2 GetNormalizedScreenSpaceUV(float2 positionInClipSpace)：

​    获取屏幕空间uv，输入为屏幕坐标；

half3 GetObjectSpaceNormalizeViewDir(float3 positionInObjectSpace)：

​    获取对象空间的观察方向；

half3 GetWorldSpaceNormalizeViewDir(float3 positionInWorldSpace)：

float3 GetWorldSpaceViewDir(float3 positionInWorldSpace)：

​    获取世界空间的观察方向；

VertexNormalInputs GetVertexNormalInputs(float3 normalOS, float4 tangentOS)：

​    输入对象空间的法线和切线，返回世界空间下的法线，切线，父切线；

VertexPositionInputs GetVertexPositionInputs(float3 positionInObjectSpace)：

​    输入对象空间的顶点位置，获取世界空间，观察空间，裁剪空间位置以及归一化设备坐标；

###### 相机相关函数（ShaderVariablesFunction.hlsl）：

float3 GetCameraPositionWS()：

​    获取相机世界空间位置；

float4 GetScaledScreenParams()：

​    获取屏幕像素宽高；

float3 GetViewForwardDir()：

​    返回摄像机在世界空间的正向；

bool IsPerspectiveProjection()：

​    是否是透视相机；

half LinearDepthToEyeDepth(half linearDepth)：

​    将 [0,1] 之间的深度值转化为 [near, far]；

void TransformScreenUV(inout float2 screenSpaceUV)：

​    翻转uv坐标；

###### 光照相关的函数：

添加头文件：

```cpp
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
```

Light GetMainLight()：

​    获取主光源；

Light GetAdditionalLight(uint lightIndex, float3 positionInWorldSpace)：

​    返回世界空间指定位置的光源信息；

int GetAdditionalLightsCount()：

​    返回额外光源数量；

half3 LightingLambert(half3 lightColor, half3 lightDirection, half3 surfaceNormal)：

​    兰伯特漫反射光照；

half3 LightingSpecular(half3 lightColor, half3 lightDirection, half3 surfaceNormal, half3         viewDirection, half4 specularAmount, half smoothnessAmount)：

​    BlinnPhong高光反射；

half SampleAmbientOcclusion(float2 normalizedScreenSpaceUV)：

​    返回屏幕空间中遮挡值，0为遮挡，1为不遮挡；

AmbientOcclusionFactor GetScreenSpaceAmbientOcclusion(float2 normalizedScreenSpaceUV)

​    返回间接光照和直接光照的环境遮挡值；

###### 阴影相关函数：

float4 GetShadowCoord(VertexPositionInputs vertexInputs)：

​    获取阴影坐标；

float4 TransformWorldToShadowCoord(float3 positionInWorldSpace)：

​    将世界坐标下的位置转换到阴影坐标；

计算阴影：

主光源需要的指令：

```cpp
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
```

额外光源需要的指令：

```cpp
#pragma multi_compile _ _ADDITIONAL_LIGHT_SHADOWS
```

Light GetMainLight(float4 shadowCoordinates)：

​     获取主光源，包含在指定 shadowCoordinates 处的 shadowAttenuation；

half ComputeCascadeIndex(float3 positionInWorldSpace)：

​    返回级联阴影的索引；

half MainLightRealtimeShadow(float4 shadowCoordinates)：

​    返回指定位置的主光源阴影值；

half AdditionalLightRealtimeShadow(int lightIndex, float3 positionInWorldSpace)：

​    返回指定位置的指定额外光源的阴影值；

half GetMainLightShadowFade(float3 positionInWorldSpace)：

​    返回主光源阴影消失的程度；

half GetAdditionalLightShadowFade(float3 positionInWorldSpace)：

​    返回额外光源阴影消失的程度；

float3 ApplyShadowBias(float3 positionInWorldSpace, float3 normalWS, float3 lightDirection)：

​    添加阴影偏移；

##### urp Pass tags：

LightMode使渲染管线决定使用哪个pass，默认为 SRPDefaultUnlit ；

UniversalForward：

​    渲染物体几何和计算所有光照，用于前向渲染路径；

UniversalGBuffer：

​    渲染物体几何信息，用于延迟渲染路径；

UniversalForwardOnly：

​    使用前向渲染，不管 URP Renderer 使用哪个渲染路径；

DepthNormals：

​    渲染深度和法线；

DepthNormalsOnly：

​    延迟渲染路径使用；

Universal2D：

​    2D渲染使用；

ShadowCaster：

​    将光源视角的深度渲染到阴影纹理或深度纹理；

DepthOnly：

​    仅渲染摄像机视角的深度信息到深度纹理；

Meta：

​    编辑器下烘培光照贴图使用；

SRPDefaultUnlit：

​    默认值；

MotionVectors：

​    渲染运动矢量；

### 9. 自定义渲染和后处理：

##### Scriptable Render Pass：

注入脚本化渲染pass到渲染管线来获取自定义视觉效果；

创建 Scriptable Render Pass：

```cs
using UnityEngine.Rendering;



using UnityEngine.Rendering.Universal;


public class RedTintRenderPass : ScriptableRenderPass

{

    //每帧调用一次，一个相机调用一次

    //实现渲染逻辑，发送drawcall

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

    {

    }

}
```

自定义渲染pass的例子

```cs
using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

public class RedTintRenderPass : ScriptableRenderPass
{
    private Material material;
    //渲染纹理属性描述
    private RenderTextureDescriptor textureDescriptor;

    //临时纹理索引

    private RTHandle textureHandle;

    //构造函数，用于传参

    public RedTintRenderPass(Material material)

    {

        this.material = material;
        textureDescriptor = new RenderTextureDescriptor(Screen.width, Screen.height, RenderTextureFormat.Default, 0);
    }

    //在执行渲染pass前调用
    //用于配置 渲染目标，状态清除，创建临时渲染目标纹理，默认渲染到摄像机的渲染目标
    public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)

    {

        textureDescriptor.width = cameraTextureDescriptor.width;

        textureDescriptor.height = cameraTextureDescriptor.height;

        //纹理不存在或者发生变更时，回收旧的，创建新的

        RenderingUtils.ReAllocateIfNeeded(ref textureHandle, textureDescriptor);

    }

    //每帧调用一次，一个相机调用一次



    //实现渲染逻辑，发送drawcall


    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

    {

        //Get a CommandBuffer from pool.

        CommandBuffer cmd = CommandBufferPool.Get();

        //获取摄像机渲染纹理

        RTHandle cameraTargetHandle = renderingData.cameraData.renderer.cameraColorTargetHandle;

        // Blit from the camera target to the temporary render texture,

        // using the first shader pass.


        Blit(cmd, cameraTargetHandle, textureHandle, material, 0);


        // Blit from the temporary render texture to the camera target,



        // using the second shader pass.



        Blit(cmd, textureHandle, cameraTargetHandle, material, 1);

        //Execute the command buffer and release it back to the pool.



        context.ExecuteCommandBuffer(cmd);



        CommandBufferPool.Release(cmd);



    }



    //用于销毁临时资源，得在外部手动调用



    public void Dispose()



    {


        textureHandle.Release();


    }

}
```

通过 MonoBehaviour 脚本注入自定义渲染pass：

```cs
using UnityEngine;



using UnityEngine.Rendering.Universal;



using UnityEngine.Rendering;

[ExecuteAlways]


public class EnqueuePass : MonoBehaviour


{


    public Material material;


    private RedTintRenderPass redTintRenderPass;


    private void OnEnable()

    {



        redTintRenderPass = new RedTintRenderPass(material);



        redTintRenderPass.renderPassEvent = RenderPassEvent.AfterRenderingSkybox;



 



        // Subscribe the OnBeginCamera method to the beginCameraRendering event.



        //注意每个相机都会触发



        RenderPipelineManager.beginCameraRendering += OnBeginCamera;



    }



    private void OnDisable()

    {



        RenderPipelineManager.beginCameraRendering -= OnBeginCamera;


        redTintRenderPass.Dispose();

    }

    private void OnBeginCamera(ScriptableRenderContext context, Camera cam)

    {
 // Use the EnqueuePass method to inject a custom render pass

        cam.GetUniversalAdditionalCameraData().scriptableRenderer.EnqueuePass(redTintRenderPass);

    }



}
```

##### Scriptable Renderer Features：

添加到 renderer 的组件，用于更改渲染；

Scriptable Renderer Features 控制 Scriptable Render Passes 何时如何应用到renderer；

适用于实现通用的效果；

自定义 Scriptable Renderer Features：

```cs
using UnityEngine.Rendering;



using UnityEngine.Rendering.Universal;



 



public class MyRendererFeature : ScriptableRendererFeature



{



    /// <summary>



    /// When the Renderer Feature loads the first time.



    /// When you enable or disable the Renderer Feature.



    /// When you change a property in the inspector of the Renderer Feature.



    /// </summary>



    public override void Create()



    {

    }


    /// <summary>



    /// Unity calls this method every frame, once for each camera



    /// inject one or multiple ScriptableRenderPass into the scriptable Renderer



    /// </summary>



    /// <param name="renderer"></param>



    /// <param name="renderingData"></param>



    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)



    {

    }



}
```

然后就可以添加 Renderer Feature 到 Universal Renderer asset；

在 Renderer Feature 中 添加 Scriptable Render Passes：

```cpp
using UnityEngine;



using UnityEngine.Rendering;



using UnityEngine.Rendering.Universal;



 



public class MyRendererFeature : ScriptableRendererFeature



{



    [SerializeField]



    private Material material;



    private RedTintRenderPass redTintRenderPass;



 



    /// <summary>



    /// When the Renderer Feature loads the first time.



    /// When you enable or disable the Renderer Feature.



    /// When you change a property in the inspector of the Renderer Feature.



    /// </summary>



    public override void Create()



    {



        if (material && redTintRenderPass == null)



        {



            redTintRenderPass = new RedTintRenderPass(material);



            redTintRenderPass.renderPassEvent = RenderPassEvent.AfterRenderingSkybox;



        }



    }



 



    /// <summary>



    /// Unity calls this method every frame, once for each camera



    /// inject one or multiple ScriptableRenderPass into the scriptable Renderer



    /// </summary>



    /// <param name="renderer"></param>



    /// <param name="renderingData"></param>



    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)



    {



        if (redTintRenderPass == null)



            return;



 



        if (renderingData.cameraData.cameraType == CameraType.Game)



        {



            renderer.EnqueuePass(redTintRenderPass);



        }



    }



    protected override void Dispose(bool disposing)



    {



        redTintRenderPass?.Dispose();



    }



}
```

##### 使用 Renderer Feature 实现模糊效果：

自定义 Renderer Feature：

```cs
using System;



using UnityEngine;



using UnityEngine.Rendering.Universal;




[Serializable]



public class BlurSettings



{



    [Range(0, 0.4f)]



    public float horizontalBlur;



 



    [Range(0, 0.4f)]



    public float verticalBlur;



}





public class BlurRendererFeature : ScriptableRendererFeature



{



    [SerializeField]



    private BlurSettings settings;



 



    [SerializeField]



    private Material material;



 



    private BlurRenderPass blurRenderPass;



    public override void Create()



    {



        if(material && blurRenderPass == null)



        {



            blurRenderPass = new BlurRenderPass(material, settings);



            blurRenderPass.renderPassEvent = RenderPassEvent.AfterRenderingSkybox;



        }



    }



    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)



    {



        if (blurRenderPass == null)



            return;



        var cameraType = renderingData.cameraData.cameraType;



        if (cameraType == CameraType.Game || cameraType == CameraType.SceneView)



        {



            renderer.EnqueuePass(blurRenderPass);



        }



    }



    protected override void Dispose(bool disposing)



    {



        blurRenderPass?.Dispose();



    }



}
```

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\b0c5c9b985f444949ed7cf6cefabc1f2.png)

自定义 Render Pass：

```cs
using UnityEngine;



using UnityEngine.Rendering;



using UnityEngine.Rendering.Universal;



public class BlurRenderPass : ScriptableRenderPass



{



    private BlurSettings defaultSettings;



    private Material material;



    private RenderTextureDescriptor blurTextureDescriptor;



    private RTHandle blurTextureHandle;



    public BlurRenderPass(Material material, BlurSettings defaultSettings)



    {



        this.material = material;



        this.defaultSettings = defaultSettings;



        blurTextureDescriptor = new RenderTextureDescriptor(Screen.width, Screen.height, RenderTextureFormat.Default, 0);



    }


    public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)



    {



        blurTextureDescriptor.width = cameraTextureDescriptor.width;



        blurTextureDescriptor.height = cameraTextureDescriptor.height;



        blurTextureDescriptor.colorFormat = cameraTextureDescriptor.colorFormat;



        RenderingUtils.ReAllocateIfNeeded(ref blurTextureHandle, blurTextureDescriptor);



    }



    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)



    {



        CommandBuffer cmd = CommandBufferPool.Get();



        RTHandle cameraTargetHandle = renderingData.cameraData.renderer.cameraColorTargetHandle;



        UpdateBlurSettings();


        // Blit from the camera target to the temporary render texture,



        // using the first shader pass.



        Blit(cmd, cameraTargetHandle, blurTextureHandle, material, 0);



        // Blit from the temporary render texture to the camera target,



        // using the second shader pass.



        Blit(cmd, blurTextureHandle, cameraTargetHandle, material, 1);


        //Execute the command buffer and release it back to the pool.



        context.ExecuteCommandBuffer(cmd);



        CommandBufferPool.Release(cmd);



    }


    private static readonly int horizontalBlurId = Shader.PropertyToID("_HorizontalBlur");



    private static readonly int verticalBlurId = Shader.PropertyToID("_VerticalBlur");

    private void UpdateBlurSettings()

    {

        if (material == null)

            return;

        var volumeComponent = VolumeManager.instance.stack.GetComponent<BlurVolume>();



        float horizontalBlur, verticalBlur;



        if(volumeComponent && volumeComponent.horizontalBlur.overrideState)



        {



            horizontalBlur = volumeComponent.horizontalBlur.value;



            verticalBlur = volumeComponent.verticalBlur.value;



        }



        else



        {



            horizontalBlur = defaultSettings.horizontalBlur;



            verticalBlur = defaultSettings.verticalBlur;



        }


        material.SetFloat(horizontalBlurId, horizontalBlur);



        material.SetFloat(verticalBlurId, verticalBlur);



    }


    public void Dispose()



    {



        blurTextureHandle?.Release();



        blurTextureHandle = null;



    }



}
```

自定义 volume component，来设置Render Pass中使用的相关参数：

```cs
using System;



using UnityEngine.Rendering;

[Serializable]



public class BlurVolume : VolumeComponent



{



    public ClampedFloatParameter horizontalBlur = new ClampedFloatParameter(0.05f, 0, 0.5f);



 



    public ClampedFloatParameter verticalBlur = new ClampedFloatParameter(0.05f, 0, 0.5f);



}
```

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\d8a1cfbebd2c403f94710f8698af26f5.png)

编写模糊效果使用的shader：

```cpp
Shader "Custom/Blur"



{

    HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        // The Blit.hlsl file provides the vertex shader (Vert),



        // the input structure (Attributes), and the output structure (Varyings)



        #include "Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl"



        float _VerticalBlur;



        float _HorizontalBlur;

        float4 BlurVertical (Varyings input) : SV_Target



        {



            const float BLUR_SAMPLES = 64;



            const float BLUR_SAMPLES_RANGE = BLUR_SAMPLES / 2;



            



            float3 color = 0;



            float blurPixels = _VerticalBlur * _ScreenParams.y;



            



            //邻近像素卷积



            for(float i = -BLUR_SAMPLES_RANGE; i <= BLUR_SAMPLES_RANGE; i++)



            {



                float2 sampleOffset = float2 (0, (blurPixels / _BlitTexture_TexelSize.w) * (i / BLUR_SAMPLES_RANGE));



                color += SAMPLE_TEXTURE2D(_BlitTexture, sampler_LinearClamp, input.texcoord + sampleOffset).rgb;



            }


            return float4(color.rgb / (BLUR_SAMPLES + 1), 1);



        }

        float4 BlurHorizontal (Varyings input) : SV_Target

        {



            const float BLUR_SAMPLES = 64;



            const float BLUR_SAMPLES_RANGE = BLUR_SAMPLES / 2;



            UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);



            float3 color = 0;



            float blurPixels = _HorizontalBlur * _ScreenParams.x;



            for(float i = -BLUR_SAMPLES_RANGE; i <= BLUR_SAMPLES_RANGE; i++)



            {



                float2 sampleOffset = float2 ((blurPixels / _BlitTexture_TexelSize.z) * (i / BLUR_SAMPLES_RANGE), 0);



                color += SAMPLE_TEXTURE2D(_BlitTexture, sampler_LinearClamp, input.texcoord + sampleOffset).rgb;



            }



            return float4(color / (BLUR_SAMPLES + 1), 1);



        }




    ENDHLSL




    SubShader



    {



        Tags { "RenderType"="Opaque" "RenderPipeline" = "UniversalPipeline"}



        LOD 100



        ZWrite Off Cull Off



        Pass



        {



            Name "BlurPassVertical"



 



            HLSLPROGRAM


            #pragma vertex Vert



            #pragma fragment BlurVertical



            



            ENDHLSL



        }



        



        Pass



        {



            Name "BlurPassHorizontal"

            HLSLPROGRAM


            #pragma vertex Vert

            #pragma fragment BlurHorizontal



            ENDHLSL



        }



    }



}
```

##### Blit ：

复制一种纹理到目的纹理；

在urp中避免使用 CommandBuffer.Blit，需使用 Blitter.BlitCameraTexture；

创建全屏 blit 效果，并使用 _CameraOpaqueTexture 纹理：

Renderer Feature：

```cs
using UnityEngine;



using UnityEngine.Rendering.Universal;


internal class ColorBlitRendererFeature : ScriptableRendererFeature



{



    public Material m_Material;



    public float m_Intensity;

    ColorBlitPass m_RenderPass = null;

    public override void Create()

    {

        m_RenderPass = new ColorBlitPass(m_Material);

    }


    //Callback after render targets are initialized



    public override void SetupRenderPasses(ScriptableRenderer renderer, in RenderingData renderingData)



    {



        if (m_RenderPass != null && renderingData.cameraData.cameraType == CameraType.Game)



        {



            // Calling ConfigureInput with the ScriptableRenderPassInput.Color argument



            // ensures that the opaque texture is available to the Render Pass.



            // 即在shader中可使用_CameraOpaqueTexture



            m_RenderPass.ConfigureInput(ScriptableRenderPassInput.Color);

            //设置pass参数



            m_RenderPass.SetTarget(renderer.cameraColorTargetHandle, m_Intensity);



        }



    }



 



    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {



        if (m_RenderPass != null && renderingData.cameraData.cameraType == CameraType.Game)



            renderer.EnqueuePass(m_RenderPass);



    }



}
```

Render Pass：

```cs
using UnityEngine;



using UnityEngine.Rendering;



using UnityEngine.Rendering.Universal;



internal class ColorBlitPass : ScriptableRenderPass

{



    ProfilingSampler m_ProfilingSampler = new ProfilingSampler("ColorBlit");


    Material m_Material;



    RTHandle m_CameraColorTarget;



    float m_Intensity;


    public ColorBlitPass(Material material)



    {



        m_Material = material;



 



        //后处理前执行



        renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;



    }



    public void SetTarget(RTHandle colorHandle, float intensity)



    {



        m_CameraColorTarget = colorHandle;



        m_Intensity = intensity;



    }


    //在渲染相机前执行，用于设置渲染目标



    public override void OnCameraSetup(CommandBuffer cmd, ref RenderingData renderingData)



    {



        //其实不用设置，默认就是相机



        ConfigureTarget(m_CameraColorTarget);



    }



    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)



    {



        var cameraData = renderingData.cameraData;



        if (cameraData.camera.cameraType != CameraType.Game)



            return;



 



        if (m_Material == null)



            return;



 



        CommandBuffer cmd = CommandBufferPool.Get();



        using (new ProfilingScope(cmd, m_ProfilingSampler))



        {



            m_Material.SetFloat("_Intensity", m_Intensity);



            Blitter.BlitCameraTexture(cmd, m_CameraColorTarget, m_CameraColorTarget, m_Material, 0);



        }



        context.ExecuteCommandBuffer(cmd);



        CommandBufferPool.Release(cmd);



    }



}
```

ColorBlit shader：

```cpp
Shader "Custom/ColorBlit"



{



        SubShader



    {



        Tags { "RenderType"="Opaque" "RenderPipeline" = "UniversalPipeline"}



        LOD 100



        ZWrite Off Cull Off



        Pass



        {



            Name "ColorBlitPass"



 



            HLSLPROGRAM



            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"



            // The Blit.hlsl file provides the vertex shader (Vert),



            // input structure (Attributes) and output strucutre (Varyings)



            #include "Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl"



 



            #pragma vertex Vert



            #pragma fragment frag



 



            TEXTURE2D_X(_CameraOpaqueTexture);



            SAMPLER(sampler_CameraOpaqueTexture);



 



            float _Intensity;



 



            half4 frag (Varyings input) : SV_Target



            {



                UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);



                float4 color = SAMPLE_TEXTURE2D_X(_CameraOpaqueTexture, sampler_CameraOpaqueTexture, input.texcoord);



                return color * float4(0, _Intensity, 0, 1);



            }



            ENDHLSL



        }



    }



}
```

##### Scriptable Renderer Feature 方法：

AddRenderPasses：用于添加一个或多个渲染pass；

Create：初始化；

Dispose：释放资源；

SetupRenderPasses：设置渲染pass，相比AddRenderPasses，该函数调用时存在渲染目标；

##### Scriptable Render Pass 方法：

Execute：执行渲染逻辑；

OnCameraSetup：用于配置渲染目标或者状态清除；

OnCameraCleanup：用于清除渲染时创建的资源；



CPU优化

## HDRP渲染管线的使用和优化

### 入门使用

下载插件

![image-20241101204530622](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241101204530622.png)

设置配置文件

![image-20241101204701141](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241101204701141.png)

如果没有配置文件，就需要自己去创建

![image-20241101204841356](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241101204841356.png)

Render Pipeline（如图1.54所示）

为了便于开发人员快速进入HDRP的世界，Unity在两个地方提供了相关快捷工具。

- 菜单Edit->Render Pipeline（如图1.54所示）

![unity HDR哪里开 unity hdrp vr_unity HDR哪里开](D:\DeskTop\FileSum\UnityClassResources\笔记\resize,m_fixed,w_1184.webp)

在这里你可以选中的材质或者项目中所有的材质一键升级到HDRP材质（一般需要在升级完成以后在手动微调）。这些功能使用与将内置渲染管线和通用渲染管线升级到高清渲染管线。详细信息可参考文档具体描述。

- 菜单Window->Render Pipeline（如图1.55所示）

![unity HDR哪里开 unity hdrp vr_贴图_02](D:\DeskTop\FileSum\UnityClassResources\笔记\resize,m_fixed,w_1184.webp)

我们重点介绍三个窗口。因为这三个窗口分别为在项目开始阶段使用的HDRP配置窗口（HD Render Pipeline Wizard）、在项目开发阶段使用的可视化Debug窗口（Render Pipeline Debug）和模型资产查看窗口（Lock Dev）。

### （1）HD Render Pipeline Wizard（HDRP设置窗口）

通过Wizad窗口，我们可以快速检查当前项目是否拥有HDRP项目所要求的设置。
在此窗口中最终要的时HDRP、 HDRP+VR 和 HDRP+DXR这三个分区。

- HDRP分区：普通HDRP项目默认使用该快速配置检查列表。如果这里有一项没有 通过检查，则界面上会出现Fix按钮，你可以针对每一项单独修复，也可以一键修复所有检查项。
- HDRP+VR 分区：适用于针对HDRP VR 项目的配置检查。
- HDRP+DXR 分区：使用于针对HDRP光线追踪项目配置检查。
  判断最终检查结果是否正确的原则是，所有的检查项必须都是绿色打勾状态，如图1.56所示。

![unity HDR哪里开 unity hdrp vr_贴图_03](D:\DeskTop\FileSum\UnityClassResources\笔记\resize,m_fixed,w_1184.webp)

### （2）Render Pipeline Debug（HDRP Debug窗口）

Debug窗口的作用是，我们通过调整这里不同的参数，可在Game窗口或者Scene窗口实时查看相关的信息。我们使用Debug窗口中的 Material->Common Material Property参数说明。
如图1.57所示，如果我们选中Albedo（反照率贴图），则在Game窗口中可以看到模型只加载 Albedo的效果。

![unity HDR哪里开 unity hdrp vr_unity HDR哪里开_04](D:\DeskTop\FileSum\UnityClassResources\笔记\resize,m_fixed,w_1184.webp)

如图1.58所示， 如果我们选择Normal（法线贴图），则在Game窗口中可以看到模型上加载Normal（法线贴图）的效果

![unity HDR哪里开 unity hdrp vr_unity_05](D:\DeskTop\FileSum\UnityClassResources\笔记\resize,m_fixed,w_1184.webp)

如图1.59所示，入托我们选择AmbientOcclunsion（环境光则比贴图），则在Game窗口中可以看到模型上加载AmbientOcclusion（环境光遮蔽）的效果。

![unity HDR哪里开 unity hdrp vr_游戏引擎_06](D:\DeskTop\FileSum\UnityClassResources\笔记\resize,m_fixed,w_1184.webp)

### （3） Look Dev

通过LookDev窗口，我们可以将模型至于多个HDRI环境下， 一边进行快速查看和效果比较。下面我们来一起学习如何使用Look Dev窗口。

步骤1： 通过顶部菜单Window->Asset Store,打开 Unity资源商店并搜索关键字Unity HDRI Pack。下载免费的Unity HDRI集合（如图1.60所示）。此集合中包含7个 分辨率为8192像素×4096像素的HDR图片。所有HDR图片已经被转换成了默认大小为1024像素的Cubemap，一边在Look Dev窗口中使用。

![unity HDR哪里开 unity hdrp vr_unity_07](D:\DeskTop\FileSum\UnityClassResources\笔记\resize,m_fixed,w_1184.webp)

步骤2：将HDRI Pack导入项目中，导入完成后会在Project窗口中创建一个文件夹，如图1.61 所示。

步骤3：通过菜单Windows->Render Pipeline->Look Dev打开Look Dev窗口 如图1.62所示。可以向左侧窗口拖入Project窗口中的模型预制体，然后再拖入再右侧Environment下创建的Environmenet Library（用于关联HDRI图和调整曝光等参数）。

步骤4：在Project窗口中找到 WorkShop Set 这个与之题， 把它拖入Look Dev窗口。如图1.63所示

在Project窗口中找到 WorkShop Set 这个与之题， 把它拖入Look Dev窗口。如图1.63所示

步骤5：按照图1.64所示的顺序，单机Library右侧的New 按钮（1号位置）。生成一个Environment Library资源。新生成的子元默认命名为New Environment Library（2号位置）。单击3号位置的加号（+）按钮生成4号位置的预览图和底部 的Environment Settings面板，如图1.64所示



# 实际项目中的优化

## 全场景火灾逃生

项目总结1. 项目概述：简要介绍游戏项目的主题、玩法、目标受众等基本信息。

全场景消防安全体验馆旨在提供一款沉浸式消防知识学习与火灾模拟体验软件，帮助用户提升消防意识和应对突发火灾的能力。该项目主要划分为四大功能模块：主场景体验馆区，为用户提供进入其他模块的通道；教室逃生体验模块，模拟教室火灾的逃生过程；家庭厨房火灾模块，包括油锅起火和煤气罐着火的场景；卧室火灾隐患排查模块，帮助用户识别潜在火灾隐患。游戏的玩法主要依靠系统语音提示，用户需在规定时间内完成正确的操作，旨在提高各类场所人士的消防安全意识。

2. 技术选型：阐述在项目中使用的Unity版本、插件、游戏引擎的特性、性能优化策略等。

| 本项目使用Unity引擎进行开发，选择Unity 2020版本以满足项目的功能需求与性能要求。为了实现VR体验，采用Pico 4 VR眼镜，通过SteamVR插件进行串流，确保流畅的用户交互体验。在项目后期，团队将渲染管线转换为URP（统一渲染管线），有效提高了图形质量和运行速度。为优化性能，项目主要解决模型面数过高和材质贴图质量过大的问题，采用对象池技术减少CPU负担并提高效率。此外，通过分帧执行不常用方法、优化管线画质设置以及调整抗锯齿和阴影参数，降低了渲染压力。 |
| ------------------------------------------------------------ |
| 本项目使用的Unity版本为2020.3.40，采用了以下插件：1. Dotween：该插件用于制作更流畅的动画，相较于iTween，Dotween更易于使用。2. PostProcessing：通过后处理插件增强画面的真实感，为不同场景提供多样的视觉效果。选择该插件的原因在于其灵活性和可定制性。3. 2DSprite：通过裁剪技术，防止UI被拉伸而导致失真。4. Timeline：该插件简化了动画制作过程，相比传统动画，Timeline具有更强的动态交互能力，能够与复杂逻辑结合。同时，它允许对动画的不同版本进行管理，方便追踪和回溯更改，从而提高团队协作效率。5. 自定义插件：通过编写自定义插件，我们实现了字体缩放和样式的调整，在保证文字大小不变的情况下提高了字体的清晰度。6. Magica Cloth：该插件能够在较低资源消耗下实现高质量的物理模拟，即使在移动平台上也能表现出色。它支持Unity Job系统和Burst编译器，实现高速模拟计算，进一步提升了效率。与Unity内置的Cloth相比，Magica Cloth在性能和稳定性上更具优势，且能够处理更复杂的布料交互。7. Animation Rigging：该插件实现了人物骨骼的可视化，并结合Magica Cloth提高了工作效率。 |

3. 开发过程：描述项目从策划、设计、开发到测试的阶段，遇到的技术困难和解决方案。

| 项目的开发流程包括策划、设计、开发和测试。策划阶段团队考虑到社会火灾频发的现实，设计了一个结合沉浸式体验的消防安全培训系统。开发过程中，团队主要使用管理器模式来统筹管理各个功能模块，利用Point锚点管理器获取不同位置的信息，并通过SceneMgr管理器进行事件通知与场景切换。为了增强用户交互体验，开发了UI管理器和音效管理器。在开发中遇到了多种技术困难，例如模型、特效和音效资源的搜集问题。特别是在锚点获取过程中，为实现更灵活的功能，需对SteamVR插件的底层代码进行定制化修改。同时，在角色移动到锚点后，需向不同事件监听器发出通知，这一过程测试了团队在合作中的挑战与技术能力。 |
| ------------------------------------------------------------ |
| 该项目由程序团队通过市场调研进行策划。在此过程中，我们遇到了一些技术挑战，包括：1. UI字体清晰度：通过编写自定义插件，提高了字体的清晰度。2. Git空间不足：搭建本地服务器以扩展存储空间，解决了Git空间不足的问题。3. 资源加载模块卡顿：通过异步加载资源，避免了主线程的阻塞，提升了性能。4. 基于SteamVR的UI交互：原生程序不支持射线点击按钮触发事件，因此我们在手柄射线中添加了相应的判断来实现按钮点击。这导致代码混杂，不便于维护。为此，本项目引入了UI框架，并利用反射机制动态获取每个按钮的事件监听器，从而更高效地管理UI和触发事件，实现了代码的解耦，提升了后期维护效率。5. 导航功能：通过Nav Mesh Agent实现智能寻路功能，动态计算最佳路径。同时结合协程实现分帧执行，降低了性能消耗。6. 布料模拟：使用Cloth组件模拟真实布料，增强了场景的真实性。7. Teleporting功能调整：修改了原生Teleporting代码，仅触发事件而不改变实际位置，从而实现了禁止进入危险区域的效果，并提供了相应的提示。 |

4. **性能优化：分享在游戏开发过程中对于性能优化所做的努力，如资源管理、脚本优化、渲染优化等。**

| 在性能优化方面，项目主要集中在资源管理、脚本优化和渲染优化几个方面。针对模型面数过高问题，团队优化了模型的复杂度，并整理了贴图资源，避免使用高冗余度的材质。通过引入对象池机制减少频繁的创建和销毁操作，提高CPU和GPU的使用效率。同时，在项目启动阶段，通过分帧执行减少单帧的脚本调用，降低开局负担。在场景设计中，针对固定模型设置了静态合并功能，有效降低了Draw Call的数量，开启了GPU Instancing以提高渲染效率。在灯光设计上，减少实时光照的使用，改为使用光照贴图，从而降低GPU的渲染压力。 |
| ------------------------------------------------------------ |
| ****资源管理****通过异步加载资源，并将其存储在相应的字典中，运行过程中可以直接调用，从而降低因资源加载带来的性能消耗。 ****脚本优化****1. 协程执行：利用协程实现帧执行，减少同一帧内的命令数量，缓解帧压力。2. 实时计算控制：对于需要实时计算的物品，仅在接近时触发功能，远离时关闭以降低不必要的资源消耗（例如，镜子的实时渲染仅在靠近时进行计算）。3. 计数器机制：设置计数器，当程序运行达到一定次数时，自动关闭不必要的功能，避免性能消耗。4. 静态图像替代：将相机的实时渲染纹理替换为静态图片，减少不必要的资源消耗。5. 合批技术：启用静态和动态合批技术，将多个物体合并提交给GPU，减少CPU对GPU的调用次数，从而提升性能。 ****渲染优化****1. 项目升级：将项目升级为UPR，提升画面质量，同时关闭不必要的选项，如关闭HDR以减少纹理内存，关闭每帧执行以降低CPU占用。2. SRP Batcher：开启SRP Batcher，优化渲染性能，通过批量处理相同类型的渲染请求，减少CPU与GPU之间的通信开销，提高渲染效率。3. 后处理效果：通过后处理效果提升画面的真实度。4. 光照优化：对灯光进行烘焙，添加光照探针和反射探针，以提供真实的光影效果，提升整体画面质量。5. 遮挡剔除：在运行时分析场景几何结构，根据相机位置和其他物体决定哪些物体无需渲染，从而显著提高渲染性能，特别是在处理大量静态物体的场景时。 |

5. 遇到的问题和解决方案：回顾开发过程中遇到的具体问题和解决方法，提供经验和教训。

| 在开发过程中，团队面临多个具体问题，如资源搜集困难、功能整合挑战等。在解决方案上，团队进行了深入讨论，修改了模型和音效资源的获取方式，并探索了更高效的实现方法。在整合不同功能时，特别是在状态管理上采用有限状态机，确保在火灾逃生模拟中能够动态响应不同角色状态的变化。通过团队的密切合作，及时识别问题并进行调整，有效提升了最终产品的质量。经验教训方面，团队认识到编写规范化代码和详细注释的重要性，合理的命名和注释可以提高团队整体的工作效率。此外，团队的高效沟通和协调对整合过程至关重要，确保在开发不同模块时能够顺畅对接。 |
| ------------------------------------------------------------ |
| 经验教训在多人合作时，将不同的情景分隔到各自独立的场景中，可以有效避免冲突，提高协作效率。这种做法有以下几个优点：1. 减少冲突：不同团队成员在各自的场景中工作，避免了代码和资源的直接冲突。2. 提高可管理性：独立的场景使得每个部分的管理和调试更加清晰，便于跟踪问题。3. 提升协作效率：团队成员可以并行工作，无需等待其他人完成任务，从而加快开发进度。4. 简化合并过程：在合并多个场景时，能够更容易地识别和解决潜在的冲突。5. 增强灵活性：不同场景可以根据需要进行独立的更新和优化，提升整体项目的适应性。通过实施这种策略，可以显著提升团队的工作效率和项目的整体质量。 |

6.	成果展示：展示游戏项目的成果，包括视频、截图、评测等。

最终成果展示包含游戏项目的视频演示、截图和用户评测。项目不仅成功实现了目标功能，还得到了用户积极反馈，证明了其在提升消防安全意识和实用性的有效性。全场景消防安全体验馆将成为增强社会消防安全教育的重要工具。

