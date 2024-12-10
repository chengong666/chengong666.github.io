---
title: lua热更新篇
date: 2024-12-10 22:10:06
tags:
---
# XLUA

快速开始

## 1.1xLua简介

xLua是由腾讯维护的一个开源项目，xLua为Unity、 .Net、 Mono等C#环境增加Lua脚本编程的能力，借助xLua，这些Lua代码可以方便的和C#相互调用。自2016年初推广以来，已经应用于十多款腾讯自研游戏，因其良好性能、易用性、扩展性而广受好评。现在，腾讯已经将xLua开源到GitHub。
git下载地址：https://github.com/Tencent/xLua
xLua教程地址：https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/XLua%E6%95%99%E7%A8%8B.md

xLua在功能、性能、易用性都有不少突破，这几方面分别最具代表性的是：

可以运行时把C#实现（方法，操作符，属性，事件等等）替换成lua实现；
出色的GC优化，自定义struct，枚举在Lua和C#间传递无C# gc alloc；
编辑器下无需生成代码，开发更轻量；
除此之外，xLua另一特色功能就是代码热补丁。非常适合前期没有规划使用Lua进行逻辑开发，后期又需要在iOS这种平台获得代码热更新能力的项目。

## **课程导入：** 

**什么是IL**

http://blog.csdn.net/dodream/article/details/4726421

IL是.net上衍生出的一门语言，该语言用于将高级语言转化为IL语言，IL有泛型等语法。

 **什么样的代码执行能支持热更新**

两种模式：
**模式一：**将所有的代码指令先加载到内存--代码段，然后执行；

c++开发的程序，执行将所有的代码加载到内存中

普通unityc#开发程序，跑起来所有的c#代码都被加载到内存；

对于上面的这种模式，运行的时候就将所有的代码上传了，这种模式是不能进行热更新的，只能通过重新安装才能更新

**模式二：**将主要的核心代码加载到内存中（比如游戏框架等核心代码），所以核心代码这部分是不支持热更新的。加载了核心的代码之后，然后执行动态加载需要更新的代码，从而实现热更新。

**例子**：当你打开一个网页，看到的最初的页面相当于是核心代码，你去搜索相当于动态去加载器一部分代码。

**热更新原理图**

![image-20240402202627400](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240402202627400.png)

**为什么选择lua，做热更新，xlua是什么**

lua：游戏客户端开发比较广泛，并且lua解释执行接近与native性能

> Lua在游戏客户端开发中比较广泛，主要是因为它具有以下几个优点：
>
> 1. 轻量级：Lua的代码非常轻量级，它的解释器非常小巧，可以轻松地嵌入到游戏客户端中，而不会占用太多的系统资源。
>
> 2. 易于学习和使用：Lua的语法非常简单，易于学习和使用。这使得开发者可以快速地上手，快速地开发出高质量的游戏客户端应用。
>
> 3. 可扩展性：Lua具有很高的可扩展性，它可以通过C/C++扩展库来扩展其功能，从而满足游戏客户端开发的需求。
>
> 4. 高性能：Lua的解释执行速度非常快，接近于原生代码的执行速度。这使得它在游戏客户端开发中具有很高的性能表现。
>
> 当说Lua解释执行接近于native性能时，这意味着Lua的解释器在执行Lua代码时，具有很高的执行效率和性能表现，接近于原生代码的执行效率。这使得Lua在游戏客户端开发中可以作为一种高效、灵活的脚本语言使用，同时也可以满足游戏客户端开发中对性能的要求。

xlua：它是lua在c#环境（.net）lua解决方案；

1：为c#环境下的lua代码提供解释器（lua虚拟机）

2：xlua 在unity的接口做好了，方便lua开发的时候调用

3：可以将部分的c#代码，使用lua修正

> 可以这样理解：
>
> 1. **为c#环境下的lua代码提供解释器（lua虚拟机）**：xlua提供了一个Lua虚拟机，使得在C#环境下可以直接执行Lua代码。这样，开发者可以在Unity中使用Lua语言编写一些逻辑代码，而不必依赖于纯粹的C#。
>
> 2. **xlua 在Unity的接口做好了，方便Lua开发的时候调用**：xlua在Unity中提供了良好的接口，使得Lua脚本可以方便地调用Unity引擎的各种功能和API，从而更容易地与游戏引擎进行交互和操作。
>
> 3. **可以将部分的C#代码，使用Lua修正**：通过xlua，开发者可以将部分原本用C#编写的代码转换成Lua代码，这样可以实现一定程度的逻辑分离和动态修改，使得游戏开发更加灵活和可维护。例如，可以使用Lua来实现游戏中的一些逻辑，而不必每次修改都重新编译C#代码。
>
> 总的来说，xlua为Unity开发者提供了一种将Lua脚本与C#代码结合的解决方案，使得游戏开发更加灵活、高效，并且方便Lua脚本与Unity引擎进行交互。

**基于xlua的纯lua开发框架的基本原则有哪些**

（1）游戏场景中不放任何的物体

**原因：**

便于维护，避免冲突，通过代码将游戏场景加载出来

（2）因为运行的时候只有一个场景，不存在场景的切换。将地图、角色、特效、粒子这些资源做成预制体，便于代码调用。  所以美术就不需要管其他的事情了，只要做好自己的东西给到程序即可，从而实现美术和程序的分离。

（3）不能手动的往节点和预制体上面挂载代码，便于维护，因为如果都是通过代码添加，就可以通过文本进行搜索addcomponent，而不是一个个节点上面去找。

（4）纯assetbundle做资源代替resources

a:方便更新：更新代码通过打ab包+更新资源（assetsbundle）

b:方便打空包：1、场景+场景依赖的资源 2、Resources文件夹下的所有资源打入安装包，resources不能打空包，resources会 将所有的资源打入安装包

（5）制定一个纯lua的组件化开发框架，便于编写业务逻辑，以及逻辑计算，减少lua调用c#，减少来回调用的开销

（6）（因为每一次都打成ab包不便于调试）调试模式，调试模式加载lua代码，加载我们资源，通过assetsDatabase来加载资源；

发布模式：lua -->打包成脚本ab包，资源也打成ab包，更新加载

> 这些基本原则是为基于xlua的纯Lua开发框架设计的指导原则，旨在优化游戏开发流程并提高代码维护性。让我逐条解释一下：
>
> 1. **游戏场景中不放任何的物体**：通过代码加载游戏场景，而不是在Unity编辑器中手动放置物体，这样可以减少冲突和混乱，同时使得场景的构建更加灵活和可控。
>
> 2. **预制体化资源**：将地图、角色、特效等资源制作成预制体，便于通过代码调用，从而实现逻辑与资源的分离，让美术和程序各自专注于自己的工作，提高工作效率。
>
> 3. **不手动挂载代码**：避免手动在节点和预制体上挂载代码，通过代码添加组件，便于维护和搜索，提高代码的可读性和可维护性。
>
> 4. **使用AssetBundle替代Resources**：使用AssetBundle管理资源，方便更新和打包，避免Resources文件夹将所有资源打入安装包，提高资源管理的灵活性和效率。
>
> 5. **纯Lua组件化开发框架**：制定纯Lua的组件化开发框架，便于编写业务逻辑和逻辑计算，减少Lua调用C#的频率，降低调用开销，提高性能。
>
> 6. **调试模式和发布模式**：在调试模式下加载Lua代码和资源，通过AssetDatabase加载资源进行调试；在发布模式下，将Lua代码打包成脚本AssetBundle，资源也打包成AssetBundle，实现更新加载和发布管理。
>
> 这些原则旨在优化纯Lua开发框架的设计和实现，提高游戏开发的效率和可维护性，同时优化资源管理和调试流程。

> 基于XLua的纯Lua开发框架设计的指导原则主要包括以下几点：
>
> 1. **代码驱动场景**：避免在Unity场景中放置物体，而是通过代码来构建和管理游戏场景，以提高灵活性和可控性。
>
> 2. **预制体化资源**：将游戏中的各种元素（如角色、道具、特效等）制作成预制体，便于通过代码进行加载和调用，实现逻辑与资源的分离，提高开发效率。
>
> 3. **组件化开发**：制定纯Lua的组件化开发框架，以便于模块化、复用性和逻辑清晰，减少对C#的调用，降低调用开销，提升性能。
>
> 4. **资源管理优化**：使用AssetBundle管理资源代替Unity的Resources文件夹，以便于资源的更新、打包和加载，提高资源管理的灵活性和效率。
>
> 5. **避免手动挂载代码**：禁止手动在节点和预制体上挂载代码，通过代码添加组件，以便于维护和搜索，提高代码的可读性和可维护性。
>
> 6. **调试模式和发布模式**：建立调试模式和发布模式，调试模式下加载Lua代码和资源，便于调试和开发；发布模式下将Lua代码和资源打包成AssetBundle，实现更新加载和发布管理。
>
> 这些指导原则旨在优化基于XLua的纯Lua开发框架的设计和实现，提高游戏开发的效率、可维护性和性能。通过遵循这些原则，开发团队可以更好地组织代码结构、管理资源和提高开发效率。



## 一个简单的官方案例

界面：点击hotfix即可进行更新

![image-20240413174355337](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240413174355337.png)

如果有代码修改时开始之前一定要先进行代码的注入

![image-20240413174521630](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240413174521630.png)

如果报出乱七八糟的错误就惦记clear

```c#
using UnityEngine;
using XLua;

namespace XLuaTest
{
    [Hotfix]
    public class HotfixTest : MonoBehaviour
    {
        LuaEnv luaenv = new LuaEnv();

        private int tick = 0;

        // Use this for initialization
        void Start()
        {

        }
        // Update is called once per frame
        [LuaCallCSharp]//因为lua中使用了c#代码，并且进行热更新的是update部分的代码
        void Update()
        {
            if (++tick % 50 == 0)
            {
                Debug.Log(">>>>>>>>Update in C#, tick = " + tick);
            }
        }

        void OnGUI()
        {
            if (GUI.Button(new Rect(10, 10, 300, 80), "Hotfix"))
            {
                luaenv.DoString(@"
                xlua.hotfix(CS.XLuaTest.HotfixTest, 'Update', function(self)
                    local a =CS.UnityEngine.GameObject.Find('Main Camera');
                    CS.UnityEngine.Debug.Log(a.name);
                end)
            ");
            }

            string chHint = @"在运行该示例之前，请细致阅读xLua文档，并执行以下步骤：

1.宏定义：添加 HOTFIX_ENABLE 到 'Edit > Project Settings > Player > Other Settings > Scripting Define Symbols'。
（注意：各平台需要分别设置）

2.生成代码：执行 'XLua > Generate Code' 菜单，等待Unity编译完成。

3.注入：执行 'XLua > Hotfix Inject In Editor' 菜单。注入成功会打印 'hotfix inject finish!' 或者 'had injected!' 。";
            string enHint = @"Read documents carefully before you run this example, then follow the steps below:

1. Define: Add 'HOTFIX_ENABLE' to 'Edit > Project Settings > Player > Other Settings > Scripting Define Symbols'.
(Note: Each platform needs to set this respectively)

2.Generate Code: Execute menu 'XLua > Generate Code', wait for Unity's compilation.


3.Inject: Execute menu 'XLua > Hotfix Inject In Editor'.There should be 'hotfix inject finish!' or 'had injected!' print in the Console if the Injection is successful.";
            GUIStyle style = GUI.skin.textArea;
            style.normal.textColor = Color.red;
            style.fontSize = 16;
            GUI.TextArea(new Rect(10, 100, 500, 290), chHint, style);
            GUI.TextArea(new Rect(10, 400, 500, 290), enHint, style);
        }
    }
}

```



## **编写一个基于xlua的基本框架**

**step1：项目的目录结构**

1：AssetsPackage用于存放所有的游戏紫苑，打ab包就是将这个文件夹下面的内容打成ab包

2：Scenes：用于存放我们的游戏场景：运行时候Main场景，美术，策划使用的场景，主场景的对象通过代码展示出来

3：Scripts：用于存放游戏的C#代码：存放一些框架性质的代码，也就是那些不支持热更新的代码

4：LuaScripts：用于存放lua代码，实现热更新，这一部分的代码都是可以实现热更新



5：StreammingAssets:该文件夹用于存放ab包，如果不打空包，打算将ab包的资源放入安装包。那么就可以将ab包复制到StreammingAssets文件夹下。

好处1：如果不打算打空包，可以将ab包打包到安装包，打包的资源可以通过UnityWebRequest读取出来

6：Editor：可以用于扩展我们的编辑器

![image-20240409220141066](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240409220141066.png)

![image-20240409220204537](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240409220204537.png)

![image-20240410191145501](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240410191145501.png)

**step2：项目的目录结构**

1：全局唯一的实例:单例

对于工具类的代码需要放在Utils文件夹下

Utils：游戏的工具性质代码

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class SingleTon<T> where T : new()
{
    private static T _Instance;
    private static object mutex = new object();//通过这个保证我们的实例是线程安全的
    public static T Instance { 
        get 
        {
            if (_Instance == null)
            { 
                lock (mutex)//保证单例是线程安全的，保证只有一个线程访问，多个线程同时访问，会出问题
                {
                    if (_Instance == null)
                    {
                        _Instance = new T();
                    }
                }
            }
            return _Instance;
        }
    }
}

public class UnitySingleton<T> : MonoBehaviour where T :  Component
{ 
    private static T _instance = null;
    public static T Instance { 
        get
        {
            if (_instance == null)
            {
                _instance = FindObjectOfType(typeof(T)) as T;
                if (_instance == null)
                {
                    GameObject obj = new GameObject();
                    _instance = (T)obj.AddComponent(typeof(T));
                    obj.hideFlags = HideFlags.DontSave;
                    `obj.hideFlags = HideFlags.DontSave;
这段代码的作用是设置对象的 `hideFlags` 属性为 `HideFlags.DontSave`，从而将对象标记为不保存到场景中。这个属性的含义是告诉引擎不要将该对象保存到场景中，避免在场景中重复出现。
在某些情况下，我们可能需要在运行时动态创建一些对象，例如单例对象或者一些临时对象。这些对象不需要保存到场景中，因为它们不是场景中的实际物体，而是程序中的辅助对象。如果不将这些对象标记为不保存到场景中，它们就会被保存到场景中，导致场景中出现重复的对象，从而影响程序的正确性和性能。
因此，通过将对象的 `hideFlags` 属性设置为 `HideFlags.DontSave`，我们可以确保这些对象不会被保存到场景中，避免出现重复对象的问题，提高程序的性能和可维护性。
                    obj.name = typeof(T).Name;
                }
            }
            return _instance;
        }
    }
    public virtual void Awake()
    { 
        DontDestroyOnLoad(this.gameObject);//标记这个节点，不能让其被删除
        if (_instance == null)
        {
            _instance = this as T;
        }
        else
        {
            //如果存在多个实例，就删除一个
            GameObject.Destroy( this.gameObject);
        }
    }
}
```



对于第一种单例的解释

双重检查锁定（Double-Checked Locking）机制是一种用于在多线程环境下延迟实例化对象的技术。其原理涉及到两个关键点：懒加载和线程安全。

**原理：**

1. **懒加载（Lazy Initialization）**：懒加载是指在需要时才创建对象实例，而不是在程序启动时就创建。这有助于提高程序的性能和资源利用率。

2. **线程安全（Thread Safety）**：在多线程环境下，如果多个线程同时访问某个资源，可能会导致竞态条件（Race Condition）和数据不一致性问题。双重检查锁定机制旨在解决这一问题。

**双重检查锁定的步骤：**

1. 线程进入 `Instance` 属性的 `get` 方法。
2. 第一次检查 `_Instance` 是否为 null。如果不为 null，直接返回 `_Instance`。
3. 如果 `_Instance` 为 null，进入同步块。
4. 在同步块内，再次检查 `_Instance` 是否为 null。由于进入同步块的线程只有一个，此时如果 `_Instance` 仍为 null，则该线程负责实例化对象。
5. 实例化对象后，离开同步块并返回 `_Instance`。
6. 后续其他线程再次访问 `Instance` 属性时，由于 `_Instance` 已经被实例化，直接返回已存在的实例，无需再进入同步块。

**双重检查锁定的优点和注意事项：**

- **延迟加载**：只有在需要时才实例化对象，避免了不必要的开销。
- **线程安全**：通过双重检查，确保只有一个线程可以实例化对象，避免了竞态条件。
- **注意事项**：在某些情况下，双重检查锁定可能存在问题，如在不同平台上可能有不同的表现，或者在某些语言或编译器中可能存在指令重排等问题。因此，在使用双重检查锁定时需要谨慎考虑平台和语言特性。

在多线程编程中，`mutex`（互斥量）是一种同步机制，用于确保在同一时间只有一个线程可以访问共享资源，从而避免竞态条件（Race Condition）和数据不一致性问题。在双重检查锁定机制中，`mutex`的作用是确保在同步块内只有一个线程可以执行关键代码段，从而保证单例对象的正确初始化。

**为什么要lock这个mutex：**

1. **线程安全保证**：通过使用 `mutex`，在同步块内对关键代码段进行加锁，确保只有一个线程可以执行该代码段，避免多个线程同时访问导致的竞态条件问题。

2. **双重检查**：在双重检查锁定机制中，第一次检查 `_Instance` 是否为 null是在不加锁的情况下进行的，这可以避免每次访问单例时都进入同步块，提高性能。只有在 `_Instance` 为 null 且需要实例化对象时，才需要进入同步块进行第二次检查和实例化操作。

3. **避免重复实例化**：通过加锁，确保只有一个线程可以进入同步块实例化对象，避免多个线程重复实例化对象，保证单例对象的唯一性。

因此，`lock`这个 `mutex` 的目的是为了确保在多线程环境下，只有一个线程可以进入同步块执行关键代码段，从而保证单例对象的正确初始化和线程安全性。



对于第二个单例的解释

这段代码主要实现了两种单例模式：通用的单例模式（SingleTon<T>）和针对Unity中MonoBehaviour组件的单例模式（UnitySingleton<T>）。我将逐步解释这两种单例模式的实现细节：

**Unity中的单例模式（UnitySingleton<T>）：**

1. `UnitySingleton<T>` 是一个泛型类，继承自 MonoBehaviour，泛型参数 T 表示单例组件的类型。
2. `private static T _instance = null;` 声明了一个静态变量 `_instance` 用于存储单例组件的实例。
3. `public static T Instance { get { ... } }` 是一个静态属性，用于获取单例组件的实例。
4. 在 `Instance` 的 `get` 方法中，首先检查 `_instance` 是否为 null。
5. 如果 `_instance` 为 null，则尝试通过 `FindObjectOfType(typeof(T)) as T` 方法查找是否已经存在该组件的实例。
6. 如果未找到该组件的实例，则创建一个新的 GameObject，并在其上添加该组件，并设置 GameObject 的一些属性。
7. `Awake()` 方法用于在组件被唤醒时执行，其中调用了 `DontDestroyOnLoad(this.gameObject)` 以防止该节点被删除。
8. 在 `Awake()` 方法中也进行了单例的处理，如果 `_instance` 为 null，则将当前组件实例赋值给 `_instance`，否则销毁当前 GameObject。

总的来说，这段代码实现了单例模式的两种变体，其中通用的单例模式适用于非 MonoBehaviour 类，而 UnitySingleton<T> 则适用于 MonoBehaviour 类的单例实现。

基本流程：初始化-->热更新-->启动游戏(gamelunch需要挂载在场景中)

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Animations;

public class GameLunch : UnitySingleton<GameLunch>
{

    // Start is called before the first frame update
    public override void Awake()
    {
        base.Awake();
        //初始化游戏框架：声音管理、资源管理、网络管理、日志关联、lua脚本管理
        //end
    }
    IEnumerator checkHotUpdate()
    {
        yield return 0;     //热更新资源加代码
        //end

    }

    IEnumerator GameStart() 
    {
        yield return StartCoroutine(this.checkHotUpdate());
        //等热更新完成，再打开游戏
        Debug.Log("Game Start!!");
    }
    void Start()
    {
   
        //进入游戏 通过lua虚拟机，进入lua的逻辑代码，跑起来
        this.StartCoroutine(this.GameStart());
        //end
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```

## 实现独立的lua脚本开发

step1：内置lua虚拟机，用于解释lua代码

​			 managers--->用于放置lua虚拟机代码，xluaMgr

![image-20240409223200667](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240409223200667.png)

step2:将lua代码与c#代码脱离出来，虚拟机加载我们的文本代码，并执行

c#lua虚拟机的装载执行的第一个lua脚本，main.lua

step3:启动代码main.lua

utils/managers:Lua框架性质代码

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEditor;
using UnityEngine;
using XLua;
public class XluaMgr : UnitySingleton<XluaMgr>
{
    LuaEnv luaEnv = null;
    //lua文件夹的位置
    private static string luaScriptsFolder = "LuaScripts";
    private bool isGameStarted = false;
    public override void Awake()
    {
        base.Awake();
        //初始化luaEnv
        this.InitLuaEnv();
    }   
    public byte[] LuaScriptsLoader(ref string filepath) {
        //自定义的加载器
#if UNITY_EDITOR//如果是编辑器模式luascripts
        string scriptPath = Application.dataPath + "/LuaScripts/" + filepath + ".lua.txt";
        return System.Text.Encoding.UTF8.GetBytes(File.ReadAllText(scriptPath));
#endif
    }
    
    void Start()
    {
        
    }
    private void InitLuaEnv()
    {
        this.luaEnv = new LuaEnv();
        this.luaEnv.AddLoader(this.LuaScriptsLoader);
        this.isGameStarted = false;
    }
    // Update is called once per frame
    void Update()
    {
        if (isGameStarted)
        {
            this.luaEnv.DoString("main.update()");
        }
        
    }
    private void FixedUpdate()
    {
        if (isGameStarted)
        {
            this.luaEnv.DoString("main.fixedupdate()");

        }
    }
    private void LateUpdate()
    {
        if (isGameStarted) {
            this.luaEnv.DoString("main.lateupdate()");
        }
    }
    public void enterGame()
    {
        //实现lua脚本和c#脚本分离，从而达到可以不用立即执行lua代码，先热更新再执行
        this.isGameStarted = true;//此处游戏正式开始
        //用于写游戏逻辑，跑lua代码
        this.luaEnv.DoString("require('main')");
        this.luaEnv.DoString("main.init()");

    }
}
```

![image-20240410190852786](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240410190852786.png)

lua代码：

```lua
main = {}

--local GameApp = require("GameApp.lua")//转载游戏代码
local function init()
    --初始化lua框架自定义事件
    print("init")
    --end
    --进入游戏逻辑
   --GameApp.EnterGameha()
end
main.init = init
local function update()
    print("update")
end
main.update = update

local function fixedupdate()
    print("fixedupdate")
end
main.fixedupdate = fixedupdate

local function lateupdate()
    print("lateupdate")
end
main.lateupdate = lateupdate
```

```lua
local GameApp = {}

function EnterGameScene()
--释放游戏地图
print("entergamemap")
--释放游戏角色，怪物，道具，物品到场景
--释放ui界面
end
GameApp.EnterGameha = function()
	EnterGameScene()
end

return GameApp
```

game：游戏逻辑代码

step4:自定义一个lua代码装载器，当我们请求装载代码的时候，到我们项目制定的位置加载lua代码

```c#
        luaEnv.AddLoader((ref string path) => {
            string path1 = Application.dataPath + "/LuaScripts/" + path + ".lua.txt";
            return System.Text.Encoding.UTF8.GetBytes(File.ReadAllText(path1));
        });
```

step5:装载模式

调试模式：luascripts这个路径下加载就可以



发布模式：luascripts---->ab包--->从ab包里面加载lua代码内容

 ![image-20240410190525165](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240410190525165.png)

**总结：**热更新就是将最新的游戏逻辑代码传入到服务器上面，在游戏运行的时候，将最新的代码加载下来，替换掉原来的代码，从而实现热更新

## 基于Xlua手写纯lua开发框架

## 使用建议

![image-20240413180654290](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240413180654290.png)

![image-20240413181940380](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240413181940380.png)

## Uniyt AB包 上传和下载

![image-20240821172831172](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821172831172.png)



**学习内容**![image-20240821173123972](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821173123972.png)

前提知识

![image-20240821173329815](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821173329815.png)

![image-20240821173439022](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821173439022.png)

### 上传部分

1、准备AB包资源

![image-20240821173841881](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821173841881.png)







2、准备打包工具

将文件夹啊拖入到assets文件夹下

![image-20240821194926300](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821194926300.png)

打开打包工具

![image-20240821195007916](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821195007916.png)



![image-20240821195238306](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821195238306.png)

![image-20240821195404619](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821195404619.png)

创建ab包并对资源进行标记，或者直接拖动也可以（直接拖动的前提是有这个打包工具）

![image-20240821195626508](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821195626508.png)

![image-20240821195736726](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821195736726.png)

#### 获取ab包的MD5码

![image-20240821201732279](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821201732279.png)

首先创建一个脚本用于实现MD5码的计算

什么是MD5码

![image-20240821204559974](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821204559974.png)

MD5码的作用，用于生成资源对比文件（热更新）

![image-20240821204722976](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821204722976.png)



如何获取MD5编码

![image-20240821204842789](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821204842789.png)

代码如下

```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Security.Cryptography;
using System.Text;
using UnityEditor;
using UnityEngine;

public class Lesson_MD5 : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        Debug.Log(GetMD5(Application.dataPath + "\\ArtRes\\AB\\PC\\lua"));
    }
    //获取MD5编码
    private string GetMD5(string filepath)
    {
        //将文件以流的形式打开
        using (FileStream file = new FileStream(filepath,FileMode.Open))
        {
            //声明一个MD5对象，用于生成MD5编码
            MD5 md5 = new MD5CryptoServiceProvider();
            //通过api计算md5编码 16个字节数组
            byte[] md5info = md5.ComputeHash(file);
            //关闭文件流
            file.Close();
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < md5info.Length; i++)
            {
                //转换为16进制,为了减少md5的长度
                sb.Append(md5info[i].ToString("x2"));
            }
            return sb.ToString();
        }
    }
    // Update is called once per frame
    void Update()
    {
        
    }
}
```

#### 生成AB包资源对比文件

![image-20240821205335073](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240821205335073.png)

首先创建脚本

![image-20240822154532012](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822154532012.png)



编写脚本

遍历所有AB包，然后生成对比文件

代码如下

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class CreateABCompare
{
    [MenuItem("AB包工具/创建对比文件")]
    public static void CreateABCompareFile()
    {
        //获取文件夹信息
        DirectoryInfo directory = Directory.CreateDirectory(Application.dataPath+"/ArtRes/AB/PC/");
        //获取该目录下的所有文件信息
        FileInfo[] fileinfos = directory.GetFiles();
        //用于存储信息的字符串
        string abCompareInfo = "";
        foreach (FileInfo info in fileinfos)
        {
            //没有后缀名的那一部分才是ab包
            if (info.Extension == "")
            {
                //拼接一个AB包的信息
                abCompareInfo += info.Name+" "+info.Length+" "+GetMD5(info.FullName);
                //通过分割符号，进行不同文件之间信息的分隔
                abCompareInfo += "|";
            }
        }
        //截取最后一个字符串
        abCompareInfo = abCompareInfo.Substring(0,abCompareInfo.Length-1);
        Debug.Log(abCompareInfo);
        //将文件写入
        File.WriteAllText(Application.dataPath+"/ArtRes/AB/PC/ABCompareInfo.txt",abCompareInfo);
        AssetDatabase.Refresh();//通过代码刷新
        Debug.Log("对比文件生成完成");
    }

    static private string GetMD5(string filepath)
    {
        //将文件以流的形式打开
        using (FileStream file = new FileStream(filepath, FileMode.Open))
        {
            //声明一个MD5对象，用于生成MD5编码
            MD5 md5 = new MD5CryptoServiceProvider();
            //通过api计算md5编码 16个字节数组
            byte[] md5info = md5.ComputeHash(file);
            //关闭文件流
            file.Close();
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < md5info.Length; i++)
            {
                //转换为16进制,为了减少md5的长度
                sb.Append(md5info[i].ToString("x2"));
            }
            return sb.ToString();
        }
    }
}

```

#### 搭建服务器

![image-20240822154900501](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822154900501.png)



![image-20240822154955745](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822154955745.png)

下载软件并解压

![image-20240822161747176](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822161747176.png)

下载链接

https://www.taikr.com搜索唐老师热更新

下载安装然后新建域

![image-20240822162021386](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162021386.png)

![image-20240822162102719](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162102719.png)

![image-20240822162122427](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162122427.png)

![image-20240822162143019](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162143019.png)

![image-20240822162155947](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162155947.png)

![image-20240822162211420](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162211420.png)

![image-20240822162234284](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162234284.png)

![image-20240822162249447](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162249447.png)

![image-20240822162312179](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162312179.png)

![image-20240822162405679](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162405679.png)

![image-20240822162522871](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162522871.png)

![image-20240822162557346](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162557346.png)



![image-20240822162610395](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162610395.png)

![image-20240822162625159](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162625159.png)

![image-20240822162639442](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162639442.png)

![image-20240822162654589](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162654589.png)

那么如何访问你，输入127.0.0.1，或者内网地址192.168.

如果访问不了，就需要关闭防火墙

![image-20240822162753672](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822162753672.png)

#### 上传AB包和资源对比文件

![image-20240822164506191](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822164506191.png)

首先创建脚本

![image-20240822173725398](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822173725398.png)

其次编写脚本

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.IO;
using System.Net;
using System;
using UnityEditor.TestTools;

using System.Threading.Tasks;

public class UpLoadAB
{
    [MenuItem("AB包工具/上传AB包和对比文件")]
    private static void UpLoadABFile()
    {
        //获取文件夹信息
        DirectoryInfo directory = Directory.CreateDirectory(Application.dataPath + "/ArtRes/AB/PC/");
        //获取该目录下的所有文件信息
        FileInfo[] fileinfos = directory.GetFiles();
        //用于存储信息的字符串
        string abCompareInfo = "";
        foreach (FileInfo info in fileinfos)
        {
            //没有后缀名的那一部分才是ab包,还需要后缀为.txt的资源对比文件
            if (info.Extension == "" || info.Extension == ".txt")
            {
                //上传文件
                FtpUpLoadFile(info.FullName,info.Name);
            }
        }
    }

    //如果使用同步上传会导致上传过程卡顿，所以需要通过异步上传，加一个async关键字
    private async static void FtpUpLoadFile(string filePath, string fileName)
    {
        //通过匿名函数，开一个线程处理逻辑，用于实现异步加载，防止出现卡死的感觉
        await Task.Run(() =>
            {
                try
                {
                    //1.创建一个ftp链接，用于上传,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
                    FtpWebRequest req = FtpWebRequest.Create(new Uri("ftp://127.0.0.1/AB/PC/" + fileName)) as FtpWebRequest;
                    //2.设置一个通信凭证 这样才能上传
                    NetworkCredential n = new NetworkCredential("CG", "123456789");
                    req.Credentials = n;
                    //3.其他设置
                    //设置代理为null
                    req.Proxy = null;//设置代理为空防止冲突
                                     //请求完毕后 是否关闭控制连接
                    req.KeepAlive = false;
                    //操作指令上传
                    req.Method = WebRequestMethods.Ftp.UploadFile;
                    //指定传输的类型
                    req.UseBinary = true;//二进制上传
                                         //4.上传文件
                                         //ftp流对象,用于写入文件
                    Stream upLoadStream = req.GetRequestStream();
                    //读取文件信息 写入该流对象
                    using (FileStream file = File.OpenRead(filePath))
                    {
                        Byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                        int contentLength = file.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                              //循环上传文件中的数据
                        while (contentLength != 0)
                        {
                            //写入文件流
                            upLoadStream.Write(bytes, 0, contentLength);
                            contentLength = file.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                        }
                        file.Close();
                        upLoadStream.Close();
                    }
                    Debug.Log(fileName + "上传成功");

                }
                catch (Exception ex)
                {
                    Debug.Log(fileName + "上传出错" + ex.Message);
                    throw;
                }
            }
            ); 
    }
}

```

#### **异步操作中控制权问题**

##### **在Unity中，控制权的管理主要通过以下几种方式：**

1. **协程（Coroutines）**：使用` StartCoroutine()`函数可以启动协程。**协程在每次迭代时会暂停并给其他代码或协程执行的机会，从而管理控制权**。
2. **异步操作**：异步操作允许在后台线程中处理任务，而不会阻塞主线程。如Addressables系统就是通过异步操作实现的，允许任务在后台处理，防止主线程的阻塞。
3. **`await`运算符和`async`关键字**：`await`运算符用于**等待一个异步操作完成**，并将控制权返回给调用方法。`async`关键字用于声明支持`await`的异步方法。

##### await的作用

`await Task.Run(() => ...)` 是 C# 中用于异步编程的一种方式。它主要用于在后台线程中**执行耗时的操作**，从而避免阻塞主线程（通常是 UI 线程），以保持应用程序的响应性。下面我将详细解释它的作用、原理，并提供一些使用示例。

###### 作用

1. **避免阻塞主线程**：在 Unity 中，主线程负责更新游戏的渲染和处理用户输入。如果在主线程中执行耗时的操作（如文件读取、网络请求等），会导致帧率下降或应用程序无响应。使用 `await Task.Run(...)` 可以将这些操作移到后台线程中执行。

2. **简化异步编程**：使用 `async` 和 `await` 关键字，可以让异步代码看起来像同步代码，易于理解和维护。

###### 原理

- **Task**：`Task` 是 C# 中用于表示异步操作的类。`Task.Run(...)` 方法会在一个线程池线程上启动一个新的任务。
- **await**：`await` 关键字用于等待一个异步操作的完成。在等待期间，**控制权会返回给调用者**（玩家可以继续输入，操作），允许其他操作继续执行。

###### 在 Unity 中的使用

在 Unity 中，通常需要在主线程中执行大多数操作，但可以使用 `await Task.Run(...)` 来处理耗时的任务。下面是一个简单的示例：

```csharp
using System;
using System.Threading.Tasks;
using UnityEngine;

public class AsyncExample : MonoBehaviour
{
    async void Start()
    {
        Debug.Log("开始处理数据...");
        await ProcessDataAsync();
        Debug.Log("数据处理完成!");
    }

    private async Task ProcessDataAsync()
    {
        // 在一个后台线程中执行耗时操作
        await Task.Run(() =>
        {
            // 模拟耗时操作
            System.Threading.Thread.Sleep(3000); // 例如，等待 3 秒
            Debug.Log("数据正在处理...");
        });
    }
}
```

###### 解释示例

1. **Start 方法**：当 MonoBehaviour 开始时，调用 `Start` 方法。在这里，我们调用 `ProcessDataAsync` 方法，并使用 `await` 等待其完成。
2. **ProcessDataAsync 方法**：这是一个异步方法，返回一个 `Task`。在这个方法中，我们使用 `Task.Run` 来启动一个后台任务。
3. **耗时操作**：在 `Task.Run` 的 lambda 表达式中，我们模拟了一个耗时的操作（例如，使用 `Thread.Sleep`）。在实际应用中，这里可能是文件读取或网络请求等。

###### 注意事项

- Unity 的大多数 API 只能在主线程中调用，因此在异步操作完成后，如果需要更新 UI 或执行 Unity 特有的操作，必须回到主线程。可以使用 `UnityMainThreadDispatcher` 等工具来实现这一点。
- 在使用 `await` 时，要确保方法返回 `Task` 或 `Task<T>`，并且使用 `async` 关键字修饰方法。

通过使用 `await Task.Run(...)`，你可以在 Unity 中有效地管理耗时操作，提高应用的性能和用户体验。



在异步编程中，控制权的管理是一个重要的概念，尤其是在使用 `async` 和 `await` 关键字时。下面我将详细解释控制权的作用，以及为什么在 `await` 时要等到操作完成才归还控制权。

###### 控制权的作用

1. **保持应用程序的响应性**：
   - 在异步编程中，控制权的管理允许应用程序在等待某些操作（如网络请求、文件读取等）完成时，继续响应用户输入和其他事件。**这样，用户在等待过程中不会感到程序无响应。**

2. **简化代码结构**：
   - 使用 `await` 可以让异步代码看起来像同步代码，简化了错误处理和控制流。程序员可以在代码中自然地使用顺序逻辑，而不需要使用复杂的回调或状态机。

3. **资源管理**：
   - **通过将控制权返回给调用者，系统可以更有效地管理线程和资源。线程池可以用来处理其他任务，而不是闲置等待某个操作完成。**

考虑以下代码示例：

```csharp
async Task<int> GetDataAsync()
{
    // 模拟异步数据获取
    await Task.Delay(2000); // 假设这是一个耗时的操作
    return 42; // 返回结果
}

async void Start()
{
    int result = await GetDataAsync(); // 等待 GetDataAsync 完成
    Debug.Log(result); // 现在可以安全地使用 result
}
```

在这个例子中：

- `GetDataAsync` 方法模拟了一个耗时的操作，并返回一个整数结果。
- 在 `Start` 方法中，我们使用 `await` 等待 `GetDataAsync` 完成。在这段时间内，Unity 的主线程仍然可以处理其他事件（如渲染和用户输入）。
- 只有在 `GetDataAsync` 完成并返回结果后，才能安全地使用 `result` 变量。

###### 总结

控制权的管理是异步编程的核心，它确保了应用程序在执行耗时操作时仍然保持响应性，并且能够安全地处理结果和错误。通过在 `await` 时等待操作完成，程序可以保证在继续执行后续代码时，所有必要的结果和状态都是可用和一致的。

#### async 和 await 关键字

在 Unity 开发中，处理异步任务是一项常见需求，尤其是在处理网络请求、加载资源或执行耗时操作时。Unity 提供了协程（Coroutines）来处理这些任务，但自 C# 7.0 起**，async 和 await 关键字提供了一种更现代、更直观的方式来处理异步编程**。本文将探讨在 Unity 中如何使用 async/await，以及它们与协程的比较。

##### 协程：Unity 中的传统异步处理方式

在介绍 async/await 之前，让我们快速回顾一下协程。**协程是一种允许你在每一帧执行一部分代码的方法**，这样你可以在不阻塞主线程的情况下执行长时间运行的任务。

示例：使用协程加载资源

```c#
IEnumerator LoadAssetAsync(string path)
{
    var request = Resources.LoadAsync<Texture2D>(path);
    yield return request;
    // 当资源加载完成时继续执行
    if (request.isDone)
    {
        var texture = request.asset as Texture2D;
        // 使用 texture
    }
}
```

在这个例子中，LoadAsync 方法异步加载资源，而 yield return 语句允许协程在资源加载完成前暂停。

##### async/await：现代异步编程

C# 的 async 和 await 关键字提供了一种更简洁的方式来编写异步代码。使用 async/await，**你可以像编写同步代码一样编写异步代码，极大地提高了代码的可读性和维护性。**

示例：使用 async/await 加载资源

```c#
async Task<Texture2D> LoadAssetAsync(string path)
{
    var request = Resources.LoadAsync<Texture2D>(path);
    await request;
    // 当资源加载完成时继续执行
    return request.asset as Texture2D;
}
```

在这个例子中，await 关键字告诉程序在资源加载完成前暂停当前方法。一旦资源加载完成，程序将继续执行后续代码。

##### Unity 中的 async/await（现在我直接使用也是可以的，下面的内容仅作为参考）

Unity 对 async/await 的支持需要一些特殊的处理。**由于 Unity 的主线程是帧循环驱动的，直接在主线程中使用 await 可能会导致死锁**。为了解决这个问题，Unity 提供了 UniTask，这是一个第三方库，它提供了对 Unity 的 async/await 的支持。

安装 UniTask
首先，你需要安装 UniTask。可以通过 Unity Package Manager (UPM) 安装：

打开 Unity。
转到 Window > Package Manager。
点击 + 图标，选择 Add package from git URL。
输入 https://github.com/Cysharp/UniTask.git?path=/src/UniTask/Assets/Plugins/UniTask 并点击 Add。
示例：使用 UniTask 加载资源

```c#
async UniTask<Texture2D> LoadAssetAsync(string path)
{
    var request = Resources.LoadAsync<Texture2D>(path);
    await request.ToUniTask();
    // 当资源加载完成时继续执行
    return request.asset as Texture2D;
}
```

在这个例子中，ToUniTask() 方法将标准的 Unity 异步操作转换为 UniTask，允许我们使用 await 关键字。

##### 比较：协程 vs. async/await

可读性：**async/await 提供了更接近同步代码的编写方式，使得代码更易于理解和维护。**
错误处理：**async/await 支持使用 try-catch 块进行错误处理，而协程则不支持。**
集成：**async/await 更容易与 C# 的其他异步特性集成，如异步流（Async Streams）。**
性能：**协程是 Unity 特有的，针对 Unity 的性能进行了优化。UniTask 也针对 Unity 进行了优化，但可能不如原生协程高效。**
**结论**
async/await 提供了一种现代、简洁的方式来处理 Unity 中的异步任务。虽然协程仍然是 Unity 开发中的一个重要工具，但 async/await 提供了更多的灵活性和更好的错误处理能力。通过使用 UniTask，Unity 开发者可以充分利用 async/await 的优势，同时保持与 Unity 的兼容性和性能。

原文链接：https://blog.csdn.net/qq_44103359/article/details/138614058

### 下载相关

#### 创建ab包下载管理器脚本

![image-20240822202724667](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822202724667.png)

首先创建代码

![image-20240822203949276](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822203949276.png)

创建一个继承mono的单例类

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }
    public void test()
    {
        Debug.Log(1111);
    }
    private void OnDestroy()
    {
        instance = null;//如果切场景了销毁了就要将单例赋值为空
    }
}

```

#### 下载资源对比文件

![image-20240822220827087](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822220827087.png)



首先在管理器中编写脚本(下载并解析存储)

![image-20240822221023163](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240822221023163.png)

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.IO.IsolatedStorage;
using System.Net;
using System.Runtime.CompilerServices;
using UnityEngine;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    private Dictionary<string,ABInfo> remoteABInfo = new Dictionary<string,ABInfo>();
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }
    public void DownLoadABCompareFile()
    {
        print(Application.persistentDataPath);
        //从资源服务器上面下载资源对比文件
        DownLoadFile("ABCompareInfo.txt", Application.persistentDataPath+ "/ABCompareInfo.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
        //2读取然后对对比信息进行拆分
        string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo.txt");
        string[] strs = info.Split('|');
        string[] infos = null;
        for (int i = 0; i < strs.Length; i++)
        {
            infos = strs[i].Split(' ');
            //进行远端的存储
            remoteABInfo.Add(infos[0],new ABInfo(infos[0], infos[1], infos[2]));
        }
        print("远端ab包对比文件，加载结束");
    }
    private void DownLoadFile(string fileName,string localPath)
    {
        try
        {
            //1.创建一个ftp链接，用于下载,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
            FtpWebRequest req = FtpWebRequest.Create(new Uri("ftp://127.0.0.1/AB/PC/" + fileName)) as FtpWebRequest;
            //2.设置一个通信凭证 这样才能上传，如果有匿名函数可以不设置凭证，但是实际开发中，不建议设置匿名函数，因为安全性无法保证
            NetworkCredential n = new NetworkCredential("CG", "123456789");
            req.Credentials = n;
            //3.其他设置
            //设置代理为null
            req.Proxy = null;//设置代理为空防止冲突
                             //请求完毕后 是否关闭控制连接
            req.KeepAlive = false;
            //操作指令上传
            req.Method = WebRequestMethods.Ftp.DownloadFile;
            //指定传输的类型
            req.UseBinary = true;//二进制上传
                                 //4.上传文件
                                 //ftp流对象,用于写入文件
                                 //下面这部分需要修改，和上传不同
            FtpWebResponse res = req.GetResponse() as FtpWebResponse;
            Stream downLoadStream = res.GetResponseStream();//下载的流
            using (FileStream file = File.Create(localPath))
            {
                Debug.Log("本地路径：" + localPath);
                byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                int contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                      //循环上传文件中的数据
                while (contentLength != 0)
                {
                    Debug.Log(contentLength);
                    //写入文件流
                    file.Write(bytes, 0, contentLength);
                    contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                }
                file.Close();
                downLoadStream.Close();
            }
            Debug.Log("下载成功"+fileName);
        }
        catch (Exception ex)
        {
            Debug.Log(fileName+ "下载失败"+ ex.Message);
            throw;
        }
       
    }
    /// <summary>
    /// 创建一个类用于实现ab对比信息的存储
    /// </summary>
    private class ABInfo
    {
        public string name;
        public long size;
        public string md5;
        public ABInfo(string name,string size,string md5)
        { 
            this.name = name;
            this.size = long.Parse(size);
            this.md5 = md5;
        }
    }
    private void OnDestroy()
    {
        instance = null;//如果销毁了就要将单例赋值为空
    }
}

```

调用脚本

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Main : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        ABUpdateMgr.Instance.DownLoadABCompareFile();
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}
```

#### 下载AB包资源

![image-20240823162943253](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823162943253.png)



所要实现的功能

![image-20240823163032751](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823163032751.png)

代码的实现

主要部分代码如下

```c#
public async void DownLoadABFile(UnityAction<bool> overCallBack,UnityAction<int,int> updatePro)
{
    //按正常顺序这一步需要进行资源对比，然后添加
    foreach (string name in  remoteABInfo.Keys)
    { 
        downLoadList.Add(name);
    }
    //本地存储的路径，由于多线程不能访问unity相关的一些内容，比如application 所以声明在外部
    string localPath = Application.persistentDataPath + "/";
    //是否下载成功
    bool isOver = false;
    List<string> tempList = new List<string>();//用于零时存储已经下载了的文件名称
    int reDownLoadMaxNum = 5;//用于限制最大下载次数
    int downLoadOverNum = 0;//下载成功的资源数
    int downLoadMaxNum = downLoadList.Count;//资源总数
    //进行n次重新下载  
    while (downLoadList.Count>0 && reDownLoadMaxNum > 0)
    {
        for (int i = 0; i < downLoadList.Count; i++)
        {
            isOver = false;
            
            await Task.Run(() => {
                //下载对应的资源
                isOver = DownLoadFile(downLoadList[i], localPath + downLoadList[i]);
            });
            if (isOver)
            {
                //提示下载进度,将加载的进度放在外部进行
                updatePro(++downLoadOverNum, downLoadMaxNum);
                tempList.Add(downLoadList[i]);
            }
        }
        for (int i = 0; i < tempList.Count; i++)
        {
            downLoadList.Remove(tempList[i]);
            --reDownLoadMaxNum;
        }
    }
    //所有内容下载完成，告诉外部下载完成
    overCallBack(downLoadList.Count == 0);
    
}
```



其中用到了一个异步函数，他在等待的过程中，会返还控制权，给那个调用了这个函数的地方，继续执行这个函数后面的逻辑，如下面的11111111111就是调用它的地方的后面的逻辑，并不像同步函数那样阻塞主线程

![image-20240823155421295](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823155421295.png)

总的代码如下

实现处的代码

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.IO.IsolatedStorage;
using System.Net;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    private Dictionary<string,ABInfo> remoteABInfo = new Dictionary<string,ABInfo>();
    private List<string> downLoadList = new List<string>();
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }
    public async void DownLoadABFile(UnityAction<bool> overCallBack,UnityAction<int,int> updatePro)
    {
        //按正常顺序这一步需要进行资源对比，然后添加
        foreach (string name in  remoteABInfo.Keys)
        { 
            downLoadList.Add(name);
        }
        //本地存储的路径，由于多线程不能访问unity相关的一些内容，比如application 所以声明在外部
        string localPath = Application.persistentDataPath + "/";
        //是否下载成功
        bool isOver = false;
        List<string> tempList = new List<string>();//用于零时存储已经下载了的文件名称
        int reDownLoadMaxNum = 5;//用于限制最大下载次数
        int downLoadOverNum = 0;//下载成功的资源数
        int downLoadMaxNum = downLoadList.Count;//资源总数
        //进行n次重新下载  
        while (downLoadList.Count>0 && reDownLoadMaxNum > 0)
        {
            for (int i = 0; i < downLoadList.Count; i++)
            {
                isOver = false;
                
                await Task.Run(() => {
                    //下载对应的资源
                    isOver = DownLoadFile(downLoadList[i], localPath + downLoadList[i]);
                });
                if (isOver)
                {
                    //提示下载进度,将加载的进度放在外部进行
                    updatePro(++downLoadOverNum, downLoadMaxNum);
                    tempList.Add(downLoadList[i]);
                }
            }
            for (int i = 0; i < tempList.Count; i++)
            {
                downLoadList.Remove(tempList[i]);
                --reDownLoadMaxNum;
            }
        }
        //所有内容下载完成，告诉外部下载完成
        overCallBack(downLoadList.Count == 0);
        
    }
    public void DownLoadABCompareFile()
    {
        print(Application.persistentDataPath);
        //从资源服务器上面下载资源对比文件
        DownLoadFile("ABCompareInfo.txt", Application.persistentDataPath+ "/ABCompareInfo.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
        //2读取然后对对比信息进行拆分
        string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo.txt");
        string[] strs = info.Split('|');
        string[] infos = null;
        for (int i = 0; i < strs.Length; i++)
        {
            infos = strs[i].Split(' ');
            //进行远端的存储
            remoteABInfo.Add(infos[0],new ABInfo(infos[0], infos[1], infos[2]));
        }
        print("远端ab包对比文件，加载结束");
    }
    private bool DownLoadFile(string fileName,string localPath)
    {
        try
        {
            //1.创建一个ftp链接，用于下载,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
            FtpWebRequest req = FtpWebRequest.Create(new Uri("ftp://127.0.0.1/AB/PC/" + fileName)) as FtpWebRequest;
            //2.设置一个通信凭证 这样才能上传，如果有匿名函数可以不设置凭证，但是实际开发中，不建议设置匿名函数，因为安全性无法保证
            NetworkCredential n = new NetworkCredential("CG", "123456789");
            req.Credentials = n;
            //3.其他设置
            //设置代理为null
            req.Proxy = null;//设置代理为空防止冲突
                             //请求完毕后 是否关闭控制连接
            req.KeepAlive = false;
            //操作指令上传
            req.Method = WebRequestMethods.Ftp.DownloadFile;
            //指定传输的类型
            req.UseBinary = true;//二进制上传
                                 //4.上传文件
                                 //ftp流对象,用于写入文件
                                 //下面这部分需要修改，和上传不同
            FtpWebResponse res = req.GetResponse() as FtpWebResponse;
            Stream downLoadStream = res.GetResponseStream();//下载的流
            using (FileStream file = File.Create(localPath))
            {
                Debug.Log("本地路径：" + localPath);
                byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                int contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                      //循环上传文件中的数据
                while (contentLength != 0)
                {
                    
                    //写入文件流
                    file.Write(bytes, 0, contentLength);
                    contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                }
                file.Close();
                downLoadStream.Close();
            }
            Debug.Log("下载成功"+fileName);
            return true;
        }
        catch (Exception ex)
        {
            Debug.Log(fileName+ "下载失败"+ ex.Message);
            return false;
        }
       
    }
    /// <summary>
    /// 创建一个类用于实现ab对比信息的存储
    /// </summary>
    private class ABInfo
    {
        public string name;
        public long size;
        public string md5;
        public ABInfo(string name,string size,string md5)
        { 
            this.name = name;
            this.size = long.Parse(size);
            this.md5 = md5;
        }
    }
    private void OnDestroy()
    {
        instance = null;//如果销毁了就要将单例赋值为空
    }
}

```

调用处的代码

```c#
 using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Main : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        ABUpdateMgr.Instance.DownLoadABCompareFile();
        ABUpdateMgr.Instance.DownLoadABFile((isOver) => { 
            if (isOver)
            {
                Debug.Log("所有ab包下载成功");
            }
            else
            {
                Debug.Log("下载出错，检查网络是否出错");
            }
        }, (nowNum,maxNum) => { 
            //处理加载进度
            print("下载进度" + nowNum + "/" + maxNum); });
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}
```

#### 完善资源对比文件的下载，添加了下载成功与否的判断

![image-20240823165124620](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823165124620.png)

主要修改的代码

```c#
public async void DownLoadABCompareFile(UnityAction<bool> overCallBack)
{
    print(Application.persistentDataPath);
    bool isOver = false;
    int reDownLoadMaxNum = 5;
    string localPath = Application.persistentDataPath + "/";
    //从资源服务器上面下载资源对比文件
    while (!isOver &&reDownLoadMaxNum>0)
    {
        await Task.Run(() => {
            isOver = DownLoadFile("ABCompareInfo.txt", localPath + "ABCompareInfo.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
        });
        --reDownLoadMaxNum;
    }
    //告诉外部下载是否成功
    overCallBack?.Invoke(isOver);
}
/// <summary>
/// 解析下载下来的ab包下载文件
/// </summary>
public void GetRemoteABCompareFileInfo()
{
    //2读取然后对对比信息进行拆分
    string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo.txt");
    string[] strs = info.Split('|');
    string[] infos = null;
    for (int i = 0; i < strs.Length; i++)
    {
        infos = strs[i].Split(' ');
        //进行远端的存储
        remoteABInfo.Add(infos[0], new ABInfo(infos[0], infos[1], infos[2]));
    }
    print("远端ab包对比文件，加载结束");
}
```

整体的代码如下

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.IO.IsolatedStorage;
using System.Net;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    private Dictionary<string,ABInfo> remoteABInfo = new Dictionary<string,ABInfo>();
    private List<string> downLoadList = new List<string>();
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }
    public async void DownLoadABFile(UnityAction<bool> overCallBack,UnityAction<int,int> updatePro)
    {
        //按正常顺序这一步需要进行资源对比，然后添加
        foreach (string name in  remoteABInfo.Keys)
        { 
            downLoadList.Add(name);
        }
        //本地存储的路径，由于多线程不能访问unity相关的一些内容，比如application 所以声明在外部
        string localPath = Application.persistentDataPath + "/";
        //是否下载成功
        bool isOver = false;
        List<string> tempList = new List<string>();//用于零时存储已经下载了的文件名称
        int reDownLoadMaxNum = 5;//用于限制最大下载次数
        int downLoadOverNum = 0;//下载成功的资源数
        int downLoadMaxNum = downLoadList.Count;//资源总数
        //进行n次重新下载  
        while (downLoadList.Count>0 && reDownLoadMaxNum > 0)
        {
            for (int i = 0; i < downLoadList.Count; i++)
            {
                isOver = false;
                
                await Task.Run(() => {
                    //下载对应的资源
                    isOver = DownLoadFile(downLoadList[i], localPath + downLoadList[i]);
                });
                if (isOver)
                {
                    //提示下载进度,将加载的进度放在外部进行
                    updatePro(++downLoadOverNum, downLoadMaxNum);
                    tempList.Add(downLoadList[i]);
                }
            }
            for (int i = 0; i < tempList.Count; i++)
            {
                downLoadList.Remove(tempList[i]);
                --reDownLoadMaxNum;
            }
        }
        //所有内容下载完成，告诉外部下载完成
        overCallBack(downLoadList.Count == 0);
        
    }
    public async void DownLoadABCompareFile(UnityAction<bool> overCallBack)
    {
        print(Application.persistentDataPath);
        bool isOver = false;
        int reDownLoadMaxNum = 5;
        string localPath = Application.persistentDataPath + "/";
        //从资源服务器上面下载资源对比文件
        while (!isOver &&reDownLoadMaxNum>0)
        {
            await Task.Run(() => {
                isOver = DownLoadFile("ABCompareInfo.txt", localPath + "ABCompareInfo.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
            });
            --reDownLoadMaxNum;
        }
        //告诉外部下载是否成功
        overCallBack?.Invoke(isOver);
    }
    /// <summary>
    /// 解析下载下来的ab包下载文件
    /// </summary>
    public void GetRemoteABCompareFileInfo()
    {
        //2读取然后对对比信息进行拆分
        string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo.txt");
        string[] strs = info.Split('|');
        string[] infos = null;
        for (int i = 0; i < strs.Length; i++)
        {
            infos = strs[i].Split(' ');
            //进行远端的存储
            remoteABInfo.Add(infos[0], new ABInfo(infos[0], infos[1], infos[2]));
        }
        print("远端ab包对比文件，加载结束");
    }
    private bool DownLoadFile(string fileName,string localPath)
    {
        try
        {
            //1.创建一个ftp链接，用于下载,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
            FtpWebRequest req = FtpWebRequest.Create(new Uri("ftp://127.0.0.1/AB/PC/" + fileName)) as FtpWebRequest;
            //2.设置一个通信凭证 这样才能上传，如果有匿名函数可以不设置凭证，但是实际开发中，不建议设置匿名函数，因为安全性无法保证
            NetworkCredential n = new NetworkCredential("CG", "123456789");
            req.Credentials = n;
            //3.其他设置
            //设置代理为null
            req.Proxy = null;//设置代理为空防止冲突
                             //请求完毕后 是否关闭控制连接
            req.KeepAlive = false;
            //操作指令上传
            req.Method = WebRequestMethods.Ftp.DownloadFile;
            //指定传输的类型
            req.UseBinary = true;//二进制上传
                                 //4.上传文件
                                 //ftp流对象,用于写入文件
                                 //下面这部分需要修改，和上传不同
            FtpWebResponse res = req.GetResponse() as FtpWebResponse;
            Stream downLoadStream = res.GetResponseStream();//下载的流
            using (FileStream file = File.Create(localPath))
            {
                Debug.Log("本地路径：" + localPath);
                byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                int contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                      //循环上传文件中的数据
                while (contentLength != 0)
                {
                    
                    //写入文件流
                    file.Write(bytes, 0, contentLength);
                    contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                }
                file.Close();
                downLoadStream.Close();
            }
            Debug.Log("下载成功"+fileName);
            return true;
        }
        catch (Exception ex)
        {
            Debug.Log(fileName+ "下载失败"+ ex.Message);
            return false;
        }
       
    }
    /// <summary>
    /// 创建一个类用于实现ab对比信息的存储
    /// </summary>
    private class ABInfo
    {
        public string name;
        public long size;
        public string md5;
        public ABInfo(string name,string size,string md5)
        { 
            this.name = name;
            this.size = long.Parse(size);
            this.md5 = md5;
        }
    }
    private void OnDestroy()
    {
        instance = null;//如果销毁了就要将单例赋值为空
    }
}

```

调用处的代码也需要修改，因为我们从同步函数修改成异步函数，所以调用的顺序上面需要修改

```c#
 using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Main : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        ABUpdateMgr.Instance.DownLoadABCompareFile((isOver)=>{
            if (isOver)
            {
                //解析ab包对比文件
                ABUpdateMgr.Instance.GetRemoteABCompareFileInfo();
                //下载ab包
                ABUpdateMgr.Instance.DownLoadABFile((isOver) => {
                    if (isOver)
                    {
                        Debug.Log("所有ab包下载成功");
                    }
                    else
                    {
                        Debug.Log("下载出错，检查网络是否出错");
                    }
                }, (nowNum, maxNum) => {
                    //处理加载进度
                    print("下载进度" + nowNum + "/" + maxNum);
                });
            }
            else
            {
                Debug.Log("下载ab资源对比文件出错，检查网络是否出错");
            }
        });
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```

改进方面：  

上面的委托可以修改成事件

![image-20240823175503594](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823175503594.png)



### 资源更新

#### 生成本地默认资源

首先创建代码

![image-20240823175527334](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823175527334.png)

然后编写代码

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.IO;
public class MoveABToStreaming
{
    [MenuItem("AB包工具/移动选中的资源到StreamingAssets中")]
    public static void MoveABtoStreamingAssets()
    {
        //通过编辑器Selection类中方法 获取在project窗口中选中的资源
        object[] selectedAsset = Selection.GetFiltered(typeof(Object),SelectionMode.DeepAssets);
        if (selectedAsset.Length == 0)
        {
            //如果没有选中，就不执行后续的代码
            return;
        }
        //用于拼接本地默认AB包资源信息的字符串
        string abCompareInfo = "";
        //循环处理文件
        foreach (Object asset in selectedAsset)
        {
			//获取对应的路径
            string assetPath = AssetDatabase.GetAssetPath(asset);
            string fileName = assetPath.Substring( assetPath.LastIndexOf('/'));//截取最后/后面的部分 ，也就是名称  
            //判断是否有后缀，有后缀的不处理，也可以通过fileinfo去获取扩展名称
            if (fileName.IndexOf('.') != -1)
            {
                continue;
            }
            Debug.Log(fileName);
            //拷贝到对应的位置
            AssetDatabase.CopyAsset(assetPath,"Assets/StreamingAssets"+fileName);
            //获取拷贝到streamingAssets文件夹中的文件的全部信息
            FileInfo fileInfo = new FileInfo(Application.streamingAssetsPath+fileName);
            abCompareInfo += fileInfo.Name + " " + fileInfo.Length + " " + CreateABCompare.GetMD5(fileInfo.FullName);//getmd5的那个位子要修改成公有的，原本是私有的
            abCompareInfo += "|";

        }
        abCompareInfo = abCompareInfo.Substring(0, abCompareInfo.Length - 1);
        Debug.Log(abCompareInfo);
        //将本地默认资源的对比信息写入到对应路径
        File.WriteAllText(Application.streamingAssetsPath + "/ABCompareInfo.txt", abCompareInfo);
        AssetDatabase.Refresh();//通过代码刷新
        Debug.Log("对比文件生成完成");
    }
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```



#### 游戏功能--默认资源转存问题

![image-20240823215823428](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823215823428.png)



![image-20240823220048531](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823220048531.png)



![image-20240823220210975](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823220210975.png)



![image-20240823220346744](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823220346744.png)



![image-20240823220537408](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823220537408.png)

##### 那么最好的方法是避免转存

那么就是先从persistentdatapath下面下载，这个位置是最新的，如果没有找到资源，那么就从默认资源中查找

![image-20240823220754542](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823220754542.png)

##### 第一次进入和非第一次进入游戏的加载区别 

![image-20240823220851850](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823220851850.png)



#### 获取远端对比文件

![image-20240824110237507](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824110237507.png)

采用第一种方法的代码修改(只是修改了文件名)

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.IO.IsolatedStorage;
using System.Net;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    private Dictionary<string,ABInfo> remoteABInfo = new Dictionary<string,ABInfo>();
    private List<string> downLoadList = new List<string>();
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }
    public async void DownLoadABFile(UnityAction<bool> overCallBack,UnityAction<int,int> updatePro)
    {
        //按正常顺序这一步需要进行资源对比，然后添加
        foreach (string name in  remoteABInfo.Keys)
        { 
            downLoadList.Add(name);
        }
        //本地存储的路径，由于多线程不能访问unity相关的一些内容，比如application 所以声明在外部
        string localPath = Application.persistentDataPath + "/";
        //是否下载成功
        bool isOver = false;
        List<string> tempList = new List<string>();//用于零时存储已经下载了的文件名称
        int reDownLoadMaxNum = 5;//用于限制最大下载次数
        int downLoadOverNum = 0;//下载成功的资源数
        int downLoadMaxNum = downLoadList.Count;//资源总数
        //进行n次重新下载  
        while (downLoadList.Count>0 && reDownLoadMaxNum > 0)
        {
            for (int i = 0; i < downLoadList.Count; i++)
            {
                isOver = false;
                
                await Task.Run(() => {
                    //下载对应的资源
                    isOver = DownLoadFile(downLoadList[i], localPath + downLoadList[i]);
                });
                if (isOver)
                {
                    //提示下载进度,将加载的进度放在外部进行
                    updatePro(++downLoadOverNum, downLoadMaxNum);
                    tempList.Add(downLoadList[i]);
                }
            }
            for (int i = 0; i < tempList.Count; i++)
            {
                downLoadList.Remove(tempList[i]);
                --reDownLoadMaxNum;
            }
        }
        //所有内容下载完成，告诉外部下载完成
        overCallBack(downLoadList.Count == 0);
        
    }
    public async void DownLoadABCompareFile(UnityAction<bool> overCallBack)
    {
        print(Application.persistentDataPath);
        bool isOver = false;
        int reDownLoadMaxNum = 5;
        string localPath = Application.persistentDataPath + "/";
        //从资源服务器上面下载资源对比文件
        while (!isOver &&reDownLoadMaxNum>0)
        {
            await Task.Run(() => {
                isOver = DownLoadFile("ABCompareInfo.txt", localPath + "ABCompareInfo_TMP.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
            });
            --reDownLoadMaxNum;
        }
        //告诉外部下载是否成功
        overCallBack?.Invoke(isOver);
    }
    /// <summary>
    /// 解析下载下来的ab包下载文件
    /// </summary>
    public void GetRemoteABCompareFileInfo()
    {
        //2读取然后对对比信息进行拆分
        string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
        string[] strs = info.Split('|');
        string[] infos = null;
        for (int i = 0; i < strs.Length; i++)
        {
            infos = strs[i].Split(' ');
            //进行远端的存储
            remoteABInfo.Add(infos[0], new ABInfo(infos[0], infos[1], infos[2]));
        }
        print("远端ab包对比文件，加载结束");
    }
    private bool DownLoadFile(string fileName,string localPath)
    {
        try
        {
            //1.创建一个ftp链接，用于下载,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
            FtpWebRequest req = FtpWebRequest.Create(new Uri("ftp://127.0.0.1/AB/PC/" + fileName)) as FtpWebRequest;
            //2.设置一个通信凭证 这样才能上传，如果有匿名函数可以不设置凭证，但是实际开发中，不建议设置匿名函数，因为安全性无法保证
            NetworkCredential n = new NetworkCredential("CG", "123456789");
            req.Credentials = n;
            //3.其他设置
            //设置代理为null
            req.Proxy = null;//设置代理为空防止冲突
                             //请求完毕后 是否关闭控制连接
            req.KeepAlive = false;
            //操作指令上传
            req.Method = WebRequestMethods.Ftp.DownloadFile;
            //指定传输的类型
            req.UseBinary = true;//二进制上传
                                 //4.上传文件
                                 //ftp流对象,用于写入文件
                                 //下面这部分需要修改，和上传不同
            FtpWebResponse res = req.GetResponse() as FtpWebResponse;
            Stream downLoadStream = res.GetResponseStream();//下载的流
            using (FileStream file = File.Create(localPath))
            {
                Debug.Log("本地路径：" + localPath);
                byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                int contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                      //循环上传文件中的数据
                while (contentLength != 0)
                {
                    
                    //写入文件流
                    file.Write(bytes, 0, contentLength);
                    contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                }
                file.Close();
                downLoadStream.Close();
            }
            Debug.Log("下载成功"+fileName);
            return true;
        }
        catch (Exception ex)
        {
            Debug.Log(fileName+ "下载失败"+ ex.Message);
            return false;
        }
       
    }
    /// <summary>
    /// 创建一个类用于实现ab对比信息的存储
    /// </summary>
    private class ABInfo
    {
        public string name;
        public long size;
        public string md5;
        public ABInfo(string name,string size,string md5)
        { 
            this.name = name;
            this.size = long.Parse(size);
            this.md5 = md5;
        }
    }
    private void OnDestroy()
    {
        instance = null;//如果销毁了就要将单例赋值为空
    }
}

```

#### 获取本地对比文件信息

![image-20240824153235198](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824153235198.png)







主要修改的代码

```c#
 //获取本地ab包对比加载
 public void GetLocalABCompareFileInfo(UnityAction<bool> overCallBack)
 {
     //首先判断可读可写文件夹中有没有
     if (File.Exists(Application.persistentDataPath+ "/ABCompareInfo.txt"))
     {
         StartCoroutine(GetLocalABCompareFileInfoIE(Application.persistentDataPath + "/ABCompareInfo.txt",overCallBack));
     }
     else if (File.Exists(Application.streamingAssetsPath+ "/ABCompareInfo.txt"))
     {
         //其次查看默认文件夹中有没有，也就是第一次进入游戏的情况
         StartCoroutine(GetLocalABCompareFileInfoIE(Application.streamingAssetsPath + "/ABCompareInfo.txt",overCallBack));
     }
     else
     {
         //本地没有也就是空的，也合理
         overCallBack(true);
     }
     //如果两个都没有的话，那么表示第一次进入，并且没有默认资源
     //StartCoroutine(GetLocalABCompareFileInfoIE());
 }
 /// <summary>
 /// 协同程序调用
 /// </summary>
 /// <param name="filePath"></param>
 /// <returns></returns>
 public IEnumerator GetLocalABCompareFileInfoIE(string filePath, UnityAction<bool> overCallBack)
 {
     UnityWebRequest req = UnityWebRequest.Get(filePath);
     yield return req.SendWebRequest();
     Debug.Log(req.downloadHandler.text);
     if (req.result == UnityWebRequest.Result.Success)
     {
         //解析本地的信息
         GetRemoteABCompareFileInfo(req.downloadHandler.text, localABInfo);
         overCallBack(true);
     }
     else
     {
         overCallBack(false);
     }
 }
```



```c#
 public void CheckUpdate(UnityAction<bool> overCallBack,UnityAction<string> updateInfoCallBack)
 {
     //加载远端资源对比文件
     DownLoadABCompareFile((isOver) => {
         updateInfoCallBack("开始更新资源");
         if (isOver)
         {
             updateInfoCallBack("对比文件下载结束");
             string remoteInfo = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
             updateInfoCallBack("解析对比文件");
             GetRemoteABCompareFileInfo(remoteInfo,remoteABInfo);//进行远端资源的加载
             updateInfoCallBack("解析远端对比文件");
             //2加载本地对比文件
             GetLocalABCompareFileInfo((isover) => {
                 if (isover)
                 {
                     updateInfoCallBack("解析本地对比文件完成");
                 }
                 else
                 {
                     overCallBack(false);
                 }
             });
         }
         else 
         {
             overCallBack(false);//加载失败再调用
         }
     });
 }
```





修改后update代码

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.IO.IsolatedStorage;
using System.Net;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.Networking;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    private Dictionary<string,ABInfo> remoteABInfo = new Dictionary<string,ABInfo>();
    private Dictionary<string,ABInfo> localABInfo = new Dictionary<string,ABInfo>();
    private List<string> downLoadList = new List<string>();
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }

    public void CheckUpdate(UnityAction<bool> overCallBack,UnityAction<string> updateInfoCallBack)
    {
        //加载远端资源对比文件
        DownLoadABCompareFile((isOver) => {
            updateInfoCallBack("开始更新资源");
            //如果下载成功，就可以进一步解析
            if (isOver)
            {
                updateInfoCallBack("对比文件下载结束");
                string remoteInfo = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
                updateInfoCallBack("解析对比文件");
                GetRemoteABCompareFileInfo(remoteInfo,remoteABInfo);//进行远端资源的加载
                updateInfoCallBack("解析远端对比文件");
                //2加载本地对比文件
                GetLocalABCompareFileInfo((isover) => {
                    if (isover)
                    {
                        updateInfoCallBack("解析本地对比文件完成");
                    }
                    else
                    {
                        overCallBack(false);
                    }
                });
            }
            else 
            {
                overCallBack(false);//加载失败再调用
            }
        });
    }


    public async void DownLoadABFile(UnityAction<bool> overCallBack,UnityAction<int,int> updatePro)
    {
        //按正常顺序这一步需要进行资源对比，然后添加，做到现在其实这个函数还没有用到，因为下一步确定下载哪一些文件
        foreach (string name in  remoteABInfo.Keys)
        { 
            downLoadList.Add(name);
        }
        //本地存储的路径，由于多线程不能访问unity相关的一些内容，比如application 所以声明在外部
        string localPath = Application.persistentDataPath + "/";
        //是否下载成功
        bool isOver = false;
        List<string> tempList = new List<string>();//用于零时存储已经下载了的文件名称
        int reDownLoadMaxNum = 5;//用于限制最大下载次数
        int downLoadOverNum = 0;//下载成功的资源数
        int downLoadMaxNum = downLoadList.Count;//资源总数
        //进行n次重新下载  
        while (downLoadList.Count>0 && reDownLoadMaxNum > 0)
        {
            for (int i = 0; i < downLoadList.Count; i++)
            {
                isOver = false;
                
                await Task.Run(() => {
                    //下载对应的资源
                    isOver = DownLoadFile(downLoadList[i], localPath + downLoadList[i]);
                });
                if (isOver)
                {
                    //提示下载进度,将加载的进度放在外部进行
                    updatePro(++downLoadOverNum, downLoadMaxNum);
                    tempList.Add(downLoadList[i]);
                }
            }
            for (int i = 0; i < tempList.Count; i++)
            {
                downLoadList.Remove(tempList[i]);
                --reDownLoadMaxNum;
            }
        }
        //所有内容下载完成，告诉外部下载完成
        overCallBack(downLoadList.Count == 0);
        
    }
    //从远端下载文件
    public async void DownLoadABCompareFile(UnityAction<bool> overCallBack)
    {
        print(Application.persistentDataPath);
        bool isOver = false;
        int reDownLoadMaxNum = 5;
        string localPath = Application.persistentDataPath + "/";
        //从资源服务器上面下载资源对比文件
        while (!isOver &&reDownLoadMaxNum>0)
        {
            await Task.Run(() => {
                isOver = DownLoadFile("ABCompareInfo.txt", localPath + "ABCompareInfo_TMP.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
            });
            --reDownLoadMaxNum;
        }
        //告诉外部下载是否成功
        overCallBack?.Invoke(isOver);
    }
    /// <summary>
    /// 解析下载下来的ab包下载文件，可以是远端的也可以是本地的文件
    /// </summary>
    public void GetRemoteABCompareFileInfo(string info,Dictionary<string,ABInfo> ABInfo)
    {
        //2读取然后对对比信息进行拆分直接从外部传入
        //string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
        string[] strs = info.Split('|');
        string[] infos = null;
        for (int i = 0; i < strs.Length; i++)
        {
            infos = strs[i].Split(' ');
            //进行远端的存储
            ABInfo.Add(infos[0], new ABInfo(infos[0], infos[1], infos[2]));
        }
        print("对比文件，加载结束");
    }
    //获取本地ab包对比加载
    public void GetLocalABCompareFileInfo(UnityAction<bool> overCallBack)
    {
        //首先判断可读可写文件夹中有没有
        if (File.Exists(Application.persistentDataPath+ "/ABCompareInfo.txt"))
        {
            StartCoroutine(GetLocalABCompareFileInfoIE(Application.persistentDataPath + "/ABCompareInfo.txt",overCallBack));
        }
        else if (File.Exists(Application.streamingAssetsPath+ "/ABCompareInfo.txt"))
        {
            //其次查看默认文件夹中有没有，也就是第一次进入游戏的情况
            StartCoroutine(GetLocalABCompareFileInfoIE(Application.streamingAssetsPath + "/ABCompareInfo.txt",overCallBack));
        }
        else
        {
            overCallBack(true);
        }
        //如果两个都没有的话，那么表示第一次进入，并且没有默认资源
        //StartCoroutine(GetLocalABCompareFileInfoIE());
    }
    /// <summary>
    /// 协同程序调用
    /// </summary>
    /// <param name="filePath"></param>
    /// <returns></returns>
    public IEnumerator GetLocalABCompareFileInfoIE(string filePath, UnityAction<bool> overCallBack)
    {
        //通过www获取本地的资源
        UnityWebRequest req = UnityWebRequest.Get(filePath);
        yield return req.SendWebRequest();
        Debug.Log(req.downloadHandler.text);
        if (req.result == UnityWebRequest.Result.Success)
        {
            //解析本地的信息，修改原来的函数，直接传入文本，进行解析
            GetRemoteABCompareFileInfo(req.downloadHandler.text, localABInfo);
            overCallBack(true);
        }
        else
        {
            overCallBack(false);
        }
        
        
    }
    private bool DownLoadFile(string fileName,string localPath)
    {
        try
        {
            //1.创建一个ftp链接，用于下载,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
            FtpWebRequest req = FtpWebRequest.Create(new Uri("ftp://127.0.0.1/AB/PC/" + fileName)) as FtpWebRequest;
            //2.设置一个通信凭证 这样才能上传，如果有匿名函数可以不设置凭证，但是实际开发中，不建议设置匿名函数，因为安全性无法保证
            NetworkCredential n = new NetworkCredential("CG", "123456789");
            req.Credentials = n;
            //3.其他设置
            //设置代理为null
            req.Proxy = null;//设置代理为空防止冲突
                             //请求完毕后 是否关闭控制连接
            req.KeepAlive = false;
            //操作指令上传
            req.Method = WebRequestMethods.Ftp.DownloadFile;
            //指定传输的类型
            req.UseBinary = true;//二进制上传
                                 //4.上传文件
                                 //ftp流对象,用于写入文件
                                 //下面这部分需要修改，和上传不同
            FtpWebResponse res = req.GetResponse() as FtpWebResponse;
            Stream downLoadStream = res.GetResponseStream();//下载的流
            using (FileStream file = File.Create(localPath))
            {
                Debug.Log("本地路径：" + localPath);
                byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                int contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                      //循环上传文件中的数据
                while (contentLength != 0)
                {
                    
                    //写入文件流
                    file.Write(bytes, 0, contentLength);
                    contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                }
                file.Close();
                downLoadStream.Close();
            }
            Debug.Log("下载成功"+fileName);
            return true;
        }
        catch (Exception ex)
        {
            Debug.Log(fileName+ "下载失败"+ ex.Message);
            return false;
        }
       
    }
    /// <summary>
    /// 创建一个类用于实现ab对比信息的存储
    /// </summary>
    public class ABInfo
    {
        public string name;
        public long size;
        public string md5;
        public ABInfo(string name,string size,string md5)
        { 
            this.name = name;
            this.size = long.Parse(size);
            this.md5 = md5;
        }
    }
    private void OnDestroy()
    {
        instance = null;//如果销毁了就要将单例赋值为空
    }
}


```



main代码修改了

```c#
 using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Main : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        ABUpdateMgr.Instance.CheckUpdate((isOver) => {
            if (isOver)
            {
                Debug.Log("检查更新结束，隐藏进度条，进入游戏界面");
            }
            else
            {
                
                Debug.Log("网络错误，可以提示玩家去检查网络或者重启游戏");
            }
        }, (str) => { 
            //以后可以在这里处理更新加载界面上面显示信息的相关逻辑    
            Debug.Log(str); 
        });
        //ABUpdateMgr.Instance.GetLocalABCompareFileInfo();
        //ABUpdateMgr.Instance.DownLoadABCompareFile((isOver)=>{
        //    if (isOver)
        //    {
        //        //解析ab包对比文件
        //        ABUpdateMgr.Instance.GetRemoteABCompareFileInfo();
        //        //下载ab包
        //        ABUpdateMgr.Instance.DownLoadABFile((isOver) => {
        //            if (isOver)
        //            {
        //                Debug.Log("所有ab包下载成功");
        //            }
        //            else
        //            {
        //                Debug.Log("下载出错，检查网络是否出错");
        //            }
        //        }, (nowNum, maxNum) => {
        //            //处理加载进度
        //            print("下载进度" + nowNum + "/" + maxNum);
        //        });
        //    }
        //    else
        //    {
        //        Debug.Log("下载ab资源对比文件出错，检查网络是否出错");
        //    }
        //});
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```

#### 下载更新删除资源

![image-20240824170321313](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824170321313.png)



修改后的代码如下

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.IO.IsolatedStorage;
using System.Net;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.Networking;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    private Dictionary<string,ABInfo> remoteABInfo = new Dictionary<string,ABInfo>();
    private Dictionary<string,ABInfo> localABInfo = new Dictionary<string,ABInfo>();
    private List<string> downLoadList = new List<string>();
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }

    public void CheckUpdate(UnityAction<bool> overCallBack,UnityAction<string> updateInfoCallBack)
    {
        //加载远端资源对比文件
        DownLoadABCompareFile((isOver) => {
            updateInfoCallBack("开始更新资源");
            //如果下载成功，就可以进一步解析
            if (isOver)
            {
                updateInfoCallBack("对比文件下载结束");
                //读取了远端的资源对比文件
                string remoteInfo = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
                updateInfoCallBack("解析对比文件");
                GetRemoteABCompareFileInfo(remoteInfo,remoteABInfo);//进行远端资源的加载
                updateInfoCallBack("解析远端对比文件");
                //2加载本地对比文件
                GetLocalABCompareFileInfo((isover) => {
                    if (isover)
                    {
                        updateInfoCallBack("解析本地对比文件完成");
                        //解析完成之后，就进行ab资源的下载
                        updateInfoCallBack("开始对比");
                        foreach (string abName in remoteABInfo.Keys)
                        {
                            //1判断那些资源是新的，记录然后下载
                            if (!localABInfo.ContainsKey(abName))
                            { 
                                //由于本地对比信息中没有叫这个名字的ab包，所以我们记录下载它
                                downLoadList.Add(abName);
                            }
                            else
                            {
                                //如果本地有同名的ab包，那么就需要对比md5，从而判断是否是同一个资源，资源是否发生更改
                                if (localABInfo[abName].md5 != remoteABInfo[abName].md5)
                                {
                                    downLoadList.Add(abName);
                                }
                                //否则就是相同的资源不需要更改
                                //那么如果有的资源，就需要移除对比信息，解决本地比远端多一个文件的情况
                                localABInfo.Remove(abName);
                            }
                        }
                        updateInfoCallBack("对比完成");
                        updateInfoCallBack("删除无用的ab包文件");
                        foreach (string abName in localABInfo.Keys)
                        {
                            //删除剩余的文件
                            if (File.Exists(Application.persistentDataPath+"/"+abName))
                            {
                                File.Delete(Application.persistentDataPath + "/" + abName);
                            }
                        }
                        //下载待更新的ab包
                        updateInfoCallBack("下载和更新ab包文件");
                        DownLoadABFile ((isOver) => {
                        if (isOver)
                        {
                            //把本地的ab包对比文件，更新为最新
                            //将之前的对比信息存储到本地
                            updateInfoCallBack("更新ab包对比文件信息");
                            File.WriteAllText(Application.persistentDataPath + "/ABCompareInfo.txt",remoteInfo);
                        }
                        //直接传给外部截距
                        overCallBack(isOver);
                    },updateInfoCallBack);
                    }
                    else
                    {
                        overCallBack(false);
                    }
                });
            }
            else 
            {
                overCallBack(false);//加载失败再调用
            }
        });
    }


    public async void DownLoadABFile(UnityAction<bool> overCallBack,UnityAction<string> updatePro)
    {
        //按正常顺序这一步需要进行资源对比，然后添加，做到现在其实这个函数还没有用到，因为下一步确定下载哪一些文件
        //foreach (string name in  remoteABInfo.Keys)
        //{ 
        //    downLoadList.Add(name);
        //}
        //本地存储的路径，由于多线程不能访问unity相关的一些内容，比如application 所以声明在外部
        string localPath = Application.persistentDataPath + "/";
        //是否下载成功
        bool isOver = false;
        List<string> tempList = new List<string>();//用于零时存储已经下载了的文件名称
        int reDownLoadMaxNum = 5;//用于限制最大下载次数
        int downLoadOverNum = 0;//下载成功的资源数
        int downLoadMaxNum = downLoadList.Count;//资源总数
        //进行n次重新下载  
        while (downLoadList.Count>0 && reDownLoadMaxNum > 0)
        {
            for (int i = 0; i < downLoadList.Count; i++)
            {
                isOver = false;
                
                await Task.Run(() => {
                    //下载对应的资源
                    isOver = DownLoadFile(downLoadList[i], localPath + downLoadList[i]);
                });
                if (isOver)
                {
                    //提示下载进度,将加载的进度放在外部进行
                    updatePro(++downLoadOverNum+"/"+downLoadMaxNum);
                    tempList.Add(downLoadList[i]);
                }
            }
            for (int i = 0; i < tempList.Count; i++)
            {
                downLoadList.Remove(tempList[i]);
                --reDownLoadMaxNum;
            }
        }
        //所有内容下载完成，告诉外部下载完成
        overCallBack(downLoadList.Count == 0);
        
    }
    //从远端下载文件
    public async void DownLoadABCompareFile(UnityAction<bool> overCallBack)
    {
        print(Application.persistentDataPath);
        bool isOver = false;
        int reDownLoadMaxNum = 5;
        string localPath = Application.persistentDataPath + "/";
        //从资源服务器上面下载资源对比文件
        while (!isOver &&reDownLoadMaxNum>0)
        {
            await Task.Run(() => {
                isOver = DownLoadFile("ABCompareInfo.txt", localPath + "ABCompareInfo_TMP.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
            });
            --reDownLoadMaxNum;
        }
        //告诉外部下载是否成功
        overCallBack?.Invoke(isOver);
    }
    /// <summary>
    /// 解析下载下来的ab包下载文件，可以是远端的也可以是本地的文件
    /// </summary>
    public void GetRemoteABCompareFileInfo(string info,Dictionary<string,ABInfo> ABInfo)
    {
        //2读取然后对对比信息进行拆分直接从外部传入
        //string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
        string[] strs = info.Split('|');
        string[] infos = null;
        for (int i = 0; i < strs.Length; i++)
        {
            infos = strs[i].Split(' ');
            //进行远端的存储
            ABInfo.Add(infos[0], new ABInfo(infos[0], infos[1], infos[2]));
        }
        print("对比文件，加载结束");
    }
    //获取本地ab包对比加载
    public void GetLocalABCompareFileInfo(UnityAction<bool> overCallBack)
    {
        //首先判断可读可写文件夹中有没有
        if (File.Exists(Application.persistentDataPath+ "/ABCompareInfo.txt"))
        {
            StartCoroutine(GetLocalABCompareFileInfoIE(Application.persistentDataPath + "/ABCompareInfo.txt",overCallBack));
        }
        else if (File.Exists(Application.streamingAssetsPath+ "/ABCompareInfo.txt"))
        {
            //其次查看默认文件夹中有没有，也就是第一次进入游戏的情况
            StartCoroutine(GetLocalABCompareFileInfoIE(Application.streamingAssetsPath + "/ABCompareInfo.txt",overCallBack));
        }
        else
        {
            overCallBack(true);
        }
        //如果两个都没有的话，那么表示第一次进入，并且没有默认资源
        //StartCoroutine(GetLocalABCompareFileInfoIE());
    }
    /// <summary>
    /// 协同程序调用
    /// </summary>
    /// <param name="filePath"></param>
    /// <returns></returns>
    public IEnumerator GetLocalABCompareFileInfoIE(string filePath, UnityAction<bool> overCallBack)
    {
        //通过www获取本地的资源
        UnityWebRequest req = UnityWebRequest.Get(filePath);
        yield return req.SendWebRequest();
        Debug.Log(req.downloadHandler.text);
        if (req.result == UnityWebRequest.Result.Success)
        {
            //解析本地的信息，修改原来的函数，直接传入文本，进行解析
            GetRemoteABCompareFileInfo(req.downloadHandler.text, localABInfo);
            overCallBack(true);
        }
        else
        {
            overCallBack(false);
        }
        
        
    }
    private bool DownLoadFile(string fileName,string localPath)
    {
        try
        {
            //1.创建一个ftp链接，用于下载,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
            FtpWebRequest req = FtpWebRequest.Create(new Uri("ftp://127.0.0.1/AB/PC/" + fileName)) as FtpWebRequest;
            //2.设置一个通信凭证 这样才能上传，如果有匿名函数可以不设置凭证，但是实际开发中，不建议设置匿名函数，因为安全性无法保证
            NetworkCredential n = new NetworkCredential("CG", "123456789");
            req.Credentials = n;
            //3.其他设置
            //设置代理为null
            req.Proxy = null;//设置代理为空防止冲突
                             //请求完毕后 是否关闭控制连接
            req.KeepAlive = false;
            //操作指令上传
            req.Method = WebRequestMethods.Ftp.DownloadFile;
            //指定传输的类型
            req.UseBinary = true;//二进制上传
                                 //4.上传文件
                                 //ftp流对象,用于写入文件
                                 //下面这部分需要修改，和上传不同
            FtpWebResponse res = req.GetResponse() as FtpWebResponse;
            Stream downLoadStream = res.GetResponseStream();//下载的流
            using (FileStream file = File.Create(localPath))
            {
                Debug.Log("本地路径：" + localPath);
                byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                int contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                      //循环上传文件中的数据
                while (contentLength != 0)
                {
                    
                    //写入文件流
                    file.Write(bytes, 0, contentLength);
                    contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                }
                file.Close();
                downLoadStream.Close();
            }
            Debug.Log("下载成功"+fileName);
            return true;
        }
        catch (Exception ex)
        {
            Debug.Log(fileName+ "下载失败"+ ex.Message);
            return false;
        }
       
    }
    /// <summary>
    /// 创建一个类用于实现ab对比信息的存储
    /// </summary>
    public class ABInfo
    {
        public string name;
        public long size;
        public string md5;
        public ABInfo(string name,string size,string md5)
        { 
            this.name = name;
            this.size = long.Parse(size);
            this.md5 = md5;
        }
    }
    private void OnDestroy()
    {
        instance = null;//如果销毁了就要将单例赋值为空
    }
}
```



主要修改的部分

```c#
 public void CheckUpdate(UnityAction<bool> overCallBack,UnityAction<string> updateInfoCallBack)
 {
     //加载远端资源对比文件
     DownLoadABCompareFile((isOver) => {
         updateInfoCallBack("开始更新资源");
         //如果下载成功，就可以进一步解析
         if (isOver)
         {
             updateInfoCallBack("对比文件下载结束");
             //读取了远端的资源对比文件
             string remoteInfo = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
             updateInfoCallBack("解析对比文件");
             GetRemoteABCompareFileInfo(remoteInfo,remoteABInfo);//进行远端资源的加载
             updateInfoCallBack("解析远端对比文件");
             //2加载本地对比文件
             GetLocalABCompareFileInfo((isover) => {
                 if (isover)
                 {
                     updateInfoCallBack("解析本地对比文件完成");
                     //解析完成之后，就进行ab资源的下载
                     updateInfoCallBack("开始对比");
                     foreach (string abName in remoteABInfo.Keys)
                     {
                         //1判断那些资源是新的，记录然后下载
                         if (!localABInfo.ContainsKey(abName))
                         { 
                             //由于本地对比信息中没有叫这个名字的ab包，所以我们记录下载它
                             downLoadList.Add(abName);
                         }
                         else
                         {
                             //如果本地有同名的ab包，那么就需要对比md5，从而判断是否是同一个资源，资源是否发生更改
                             if (localABInfo[abName].md5 != remoteABInfo[abName].md5)
                             {
                                 downLoadList.Add(abName);
                             }
                             //否则就是相同的资源不需要更改
                             //那么如果有的资源，就需要移除对比信息，解决本地比远端多一个文件的情况
                             localABInfo.Remove(abName);
                         }
                     }
                     updateInfoCallBack("对比完成");
                     updateInfoCallBack("删除无用的ab包文件");
                     foreach (string abName in localABInfo.Keys)
                     {
                         //删除剩余的文件
                         if (File.Exists(Application.persistentDataPath+"/"+abName))
                         {
                             File.Delete(Application.persistentDataPath + "/" + abName);
                         }
                     }
                     //下载待更新的ab包
                     updateInfoCallBack("下载和更新ab包文件");
                     DownLoadABFile ((isOver) => {
                     if (isOver)
                     {
                         //把本地的ab包对比文件，更新为最新
                         //将之前的对比信息存储到本地
                         updateInfoCallBack("更新ab包对比文件信息");
                         File.WriteAllText(Application.persistentDataPath + "/ABCompareInfo.txt",remoteInfo);
                     }
                     //直接传给外部处理
                     overCallBack(isOver);
                 },updateInfoCallBack);
                 }
                 else
                 {
                     overCallBack(false);
                 }
             });
         }
         else 
         {
             overCallBack(false);//加载失败再调用
         }
     });
 }
```

### 优化部分

#### 编辑器窗口设计

![image-20240824172423938](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824172423938.png)

创建窗口并且添加功能

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using Codice.Client.BaseCommands.Merge;
public class ABTools : EditorWindow
{
    private int nowSelIndex = 0;
    private string[] targetStrings = new string[] { 
        "PC","IOS","ANDROID"
    };
    //资源服务器默认地址
    private string serverIP = "ftp://127.0.0.1";
    [MenuItem("AB包工具/打开工具窗口")]
    private static void OpenWindow()
    {
        //获取一个ABTools 编辑器窗口对象
        ABTools windown = EditorWindow.GetWindowWithRect(typeof(ABTools),new Rect(0,0,350,220)) as ABTools;
        windown.Show();//用于显示，如果没有这个，其实也没有什么影响，有上面那段代码也能实现显示
    }
    private void OnGUI()
    {
        //参数，x,y,宽高
        GUI.Label(new Rect(10,10,150,15),"平台选择");
        //通过一个这样的功能可以实现页面的选择，实现不同平台的选择 
        nowSelIndex = GUI.Toolbar(new Rect(10,30,250,20),nowSelIndex,targetStrings);
        GUI.Label(new Rect(10,60,150,15),"资源服务器地址");
        //通过输入框进行赋值
        serverIP = GUI.TextField(new Rect(10,80,150,20),serverIP);
        //创建对比文件的按钮，返回值是bool类型的
        if (GUI.Button(new Rect(10, 110, 100, 40), "创建对比文件"))
        { }
        if (GUI.Button(new Rect(115, 110, 225, 40), "保存默认资源到StreamingAssets"))
        { }
        if (GUI.Button(new Rect(10, 160, 330, 40), "上传AB包和对比文件"))
        { }
    }
}

```

#### 窗口逻辑部分实现

![image-20240824195501857](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824195501857.png)

逻辑部分代码的实现，其实就是将menuitem部分移动到统一的窗口中

```c#
using UnityEngine;
using UnityEditor;
using System.IO;
using System.Security.Cryptography;
using System.Text;

using System.Net;
using System;
using System.Threading.Tasks;

public class ABTools : EditorWindow
{
    private int nowSelIndex = 0;
    private string[] targetStrings = new string[] { 
        "PC","IOS","ANDROID"
    };
    //资源服务器默认地址
    private string serverIP = "ftp://127.0.0.1";
    [MenuItem("AB包工具/打开工具窗口")]
    private static void OpenWindow()
    {
        //获取一个ABTools 编辑器窗口对象
        ABTools windown = EditorWindow.GetWindowWithRect(typeof(ABTools),new Rect(0,0,350,220)) as ABTools;
        windown.Show();//用于显示，如果没有这个，其实也没有什么影响，有上面那段代码也能实现显示
    }
    private void OnGUI()
    {
        //参数，x,y,宽高
        GUI.Label(new Rect(10,10,150,15),"平台选择");
        //通过一个这样的功能可以实现页面的选择，实现不同平台的选择 
        nowSelIndex = GUI.Toolbar(new Rect(10,30,250,20),nowSelIndex,targetStrings);
        GUI.Label(new Rect(10,60,150,15),"资源服务器地址");
        //通过输入框进行赋值
        serverIP = GUI.TextField(new Rect(10,80,150,20),serverIP);
        //创建对比文件的按钮，返回值是bool类型的
        if (GUI.Button(new Rect(10, 110, 100, 40), "创建对比文件"))
        {
            CreateABCompareFile();
        }
        if (GUI.Button(new Rect(115, 110, 225, 40), "保存默认资源到StreamingAssets"))
        {
            MoveABtoStreamingAssets();
        }
        if (GUI.Button(new Rect(10, 160, 330, 40), "上传AB包和对比文件"))
        {
            UpLoadABFile();
        }
    }

    public void CreateABCompareFile()
    {
        //获取文件夹信息
        //要根据选择的平台读取对应文件夹下面的内容，进行对比文件的生成
        DirectoryInfo directory = Directory.CreateDirectory(Application.dataPath + "/ArtRes/AB/" + targetStrings[nowSelIndex]);
        //获取该目录下的所有文件信息
        FileInfo[] fileinfos = directory.GetFiles();
        //用于存储信息的字符串
        string abCompareInfo = "";
        foreach (FileInfo info in fileinfos)
        {
            //没有后缀名的那一部分才是ab包
            if (info.Extension == "")
            {
                //拼接一个AB包的信息
                abCompareInfo += info.Name + " " + info.Length + " " + GetMD5(info.FullName);
                //通过分割符号，进行不同文件之间信息的分隔
                abCompareInfo += "|";
            }
        }
        //截取最后一个字符串
        abCompareInfo = abCompareInfo.Substring(0, abCompareInfo.Length - 1);
        Debug.Log(abCompareInfo);
        //将文件写入到对应的位置
        File.WriteAllText(Application.dataPath + "/ArtRes/AB/"+ targetStrings[nowSelIndex] + "/ABCompareInfo.txt", abCompareInfo);
        AssetDatabase.Refresh();//通过代码刷新
        Debug.Log("对比文件生成完成");
    }

    public string GetMD5(string filepath)
    {
        //将文件以流的形式打开
        using (FileStream file = new FileStream(filepath, FileMode.Open))
        {
            //声明一个MD5对象，用于生成MD5编码
            MD5 md5 = new MD5CryptoServiceProvider();
            //通过api计算md5编码 16个字节数组
            byte[] md5info = md5.ComputeHash(file);
            //关闭文件流
            file.Close();
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < md5info.Length; i++)
            {
                //转换为16进制,为了减少md5的长度
                sb.Append(md5info[i].ToString("x2"));
            }
            return sb.ToString();
        }
    }
    public void MoveABtoStreamingAssets()
    {
        //通过编辑器Selection类中方法 获取在project窗口中选中的资源
        UnityEngine.Object[] selectedAsset = Selection.GetFiltered(typeof(UnityEngine.Object), SelectionMode.DeepAssets);
        if (selectedAsset.Length == 0)
        {
            //如果没有选中，就不执行后续的代码
            return;
        }
        //用于拼接本地默认AB包资源信息的字符串
        string abCompareInfo = "";
        foreach (UnityEngine.Object asset in selectedAsset)
        {

            string assetPath = AssetDatabase.GetAssetPath(asset);
            string fileName = assetPath.Substring(assetPath.LastIndexOf('/'));//截取最后/后面的部分   
            //判断是否有后缀，有后缀的不处理
            if (fileName.IndexOf('.') != -1)
            {
                continue;
            }
            Debug.Log(fileName);
            //拷贝到对应的位置
            AssetDatabase.CopyAsset(assetPath, "Assets/StreamingAssets" + fileName);
            //获取拷贝到streamingAssets文件夹中的文件的全部信息
            FileInfo fileInfo = new FileInfo(Application.streamingAssetsPath + fileName);
            abCompareInfo += fileInfo.Name + " " + fileInfo.Length + " " + CreateABCompare.GetMD5(fileInfo.FullName);
            abCompareInfo += "|";

        }
        abCompareInfo = abCompareInfo.Substring(0, abCompareInfo.Length - 1);
        Debug.Log(abCompareInfo);
        //将本地默认资源的对比信息写入到对应路径
        File.WriteAllText(Application.streamingAssetsPath + "/ABCompareInfo.txt", abCompareInfo);
        AssetDatabase.Refresh();//通过代码刷新
        Debug.Log("对比文件生成完成");
    }


    private  void UpLoadABFile()
    {
        //获取文件夹信息
        DirectoryInfo directory = Directory.CreateDirectory(Application.dataPath + "/ArtRes/AB/"+ targetStrings[nowSelIndex] + "/");
        //获取该目录下的所有文件信息
        FileInfo[] fileinfos = directory.GetFiles();
        //用于存储信息的字符串
        string abCompareInfo = "";
        foreach (FileInfo info in fileinfos)
        {
            //没有后缀名的那一部分才是ab包,还需要后缀为.txt的资源对比文件
            if (info.Extension == "" || info.Extension == ".txt")
            {
                //上传文件
                FtpUpLoadFile(info.FullName,info.Name);
            }
        }
    }

    //如果使用同步上传会导致上传过程卡顿，所以需要通过异步上传
    private async void FtpUpLoadFile(string filePath, string fileName)
    {
        //通过匿名函数，开一个线程处理逻辑，用于实现异步加载，防止出现卡死的感觉
        await Task.Run(() =>
            {
                try
                {
                    //1.创建一个ftp链接，用于上传,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
                    FtpWebRequest req = FtpWebRequest.Create(new Uri(serverIP+"/AB/" + targetStrings[nowSelIndex] + "/"+ fileName)) as FtpWebRequest;
                    //2.设置一个通信凭证 这样才能上传
                    NetworkCredential n = new NetworkCredential("CG", "123456789");
                    req.Credentials = n;
                    //3.其他设置
                    //设置代理为null
                    req.Proxy = null;//设置代理为空防止冲突
                                     //请求完毕后 是否关闭控制连接
                    req.KeepAlive = false;
                    //操作指令上传
                    req.Method = WebRequestMethods.Ftp.UploadFile;
                    //指定传输的类型
                    req.UseBinary = true;//二进制上传
                                         //4.上传文件
                                         //ftp流对象,用于写入文件
                    Stream upLoadStream = req.GetRequestStream();
                    //读取文件信息 写入该流对象
                    using (FileStream file = File.OpenRead(filePath))
                    {
                        byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                        int contentLength = file.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                              //循环上传文件中的数据
                        while (contentLength != 0)
                        {
                            //写入文件流
                            upLoadStream.Write(bytes, 0, contentLength);
                            contentLength = file.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                        }
                        file.Close();
                        upLoadStream.Close();
                    }
                    Debug.Log(fileName + "上传成功");

                }
                catch (Exception ex)
                {
                    Debug.Log(fileName + "上传出错" + ex.Message);
                    throw;
                }
            }
            ); 
    }
}

```

#### 客户端热更新路径优化

主要是修改了一些路径上面的，需要添加一些文件协议

 ![image-20240824210225288](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824210225288.png)

![image-20240824213645609](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824213645609.png)

![image-20240824213546415](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824213546415.png)

![image-20240824213605169](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824213605169.png)

本次目标

![image-20240824210353678](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824210353678.png)

主要修改的部分



![image-20240824210447355](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824210447355.png)

![image-20240824210515582](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824210515582.png)





![image-20240824210550419](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824210550419.png)



最终的代码为

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.IO.IsolatedStorage;
using System.Net;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.Networking;

public class ABUpdateMgr : MonoBehaviour//这里为什么要继承mono，因为如果不继承的话，使用协程的时候，就需要自己去封装工具。
{
    private static ABUpdateMgr instance;
    private Dictionary<string,ABInfo> remoteABInfo = new Dictionary<string,ABInfo>();
    private Dictionary<string,ABInfo> localABInfo = new Dictionary<string,ABInfo>();
    private List<string> downLoadList = new List<string>();
    private string serverIP = "ftp://127.0.0.1/AB/";
    public static ABUpdateMgr Instance { 
        get {
            if (instance == null)
            {
                //通过代码进行挂载
                GameObject obj = new GameObject("ABUpdateMgr");
                instance = obj.AddComponent<ABUpdateMgr>();
            }    
            return instance; 
        }
    }

    public void CheckUpdate(UnityAction<bool> overCallBack,UnityAction<string> updateInfoCallBack)
    {
        //加载远端资源对比文件
        DownLoadABCompareFile((isOver) => {
            updateInfoCallBack("开始更新资源");
            //如果下载成功，就可以进一步解析
            if (isOver)
            {
                updateInfoCallBack("对比文件下载结束");
                //读取了远端的资源对比文件,这里无需添加文件协议
                string remoteInfo = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
                updateInfoCallBack("解析对比文件");
                GetRemoteABCompareFileInfo(remoteInfo,remoteABInfo);//进行远端资源的加载
                updateInfoCallBack("解析远端对比文件");
                //2加载本地对比文件
                GetLocalABCompareFileInfo((isover) => {
                    if (isover)
                    {
                        updateInfoCallBack("解析本地对比文件完成");
                        //解析完成之后，就进行ab资源的下载
                        updateInfoCallBack("开始对比");
                        foreach (string abName in remoteABInfo.Keys)
                        {
                            //1判断那些资源是新的，记录然后下载
                            if (!localABInfo.ContainsKey(abName))
                            { 
                                //由于本地对比信息中没有叫这个名字的ab包，所以我们记录下载它
                                downLoadList.Add(abName);
                            }
                            else
                            {
                                //如果本地有同名的ab包，那么就需要对比md5，从而判断是否是同一个资源，资源是否发生更改
                                if (localABInfo[abName].md5 != remoteABInfo[abName].md5)
                                {
                                    downLoadList.Add(abName);
                                }
                                //否则就是相同的资源不需要更改
                                //那么如果有的资源，就需要移除对比信息，解决本地比远端多一个文件的情况
                                localABInfo.Remove(abName);
                            }
                        }
                        updateInfoCallBack("对比完成");
                        updateInfoCallBack("删除无用的ab包文件");
                        foreach (string abName in localABInfo.Keys)
                        {
                            //删除剩余的文件
                            if (File.Exists(Application.persistentDataPath+"/"+abName))
                            {
                                File.Delete(Application.persistentDataPath + "/" + abName);
                            }
                        }
                        //下载待更新的ab包
                        updateInfoCallBack("下载和更新ab包文件");
                        DownLoadABFile ((isOver) => {
                        if (isOver)
                        {
                            //把本地的ab包对比文件，更新为最新
                            //将之前的对比信息存储到本地
                            updateInfoCallBack("更新ab包对比文件信息");
                            File.WriteAllText(Application.persistentDataPath + "/ABCompareInfo.txt",remoteInfo);
                        }
                        //直接传给外部截距
                        overCallBack(isOver);
                    },updateInfoCallBack);
                    }
                    else
                    {
                        overCallBack(false);
                    }
                });
            }
            else 
            {
                overCallBack(false);//加载失败再调用
            }
        });
    }


    public async void DownLoadABFile(UnityAction<bool> overCallBack,UnityAction<string> updatePro)
    {
        //按正常顺序这一步需要进行资源对比，然后添加，做到现在其实这个函数还没有用到，因为下一步确定下载哪一些文件
        //foreach (string name in  remoteABInfo.Keys)
        //{ 
        //    downLoadList.Add(name);
        //}
        //本地存储的路径，由于多线程不能访问unity相关的一些内容，比如application 所以声明在外部
        string localPath = Application.persistentDataPath + "/";
        //是否下载成功
        bool isOver = false;
        List<string> tempList = new List<string>();//用于零时存储已经下载了的文件名称
        int reDownLoadMaxNum = 5;//用于限制最大下载次数
        int downLoadOverNum = 0;//下载成功的资源数
        int downLoadMaxNum = downLoadList.Count;//资源总数
        //进行n次重新下载  
        while (downLoadList.Count>0 && reDownLoadMaxNum > 0)
        {
            for (int i = 0; i < downLoadList.Count; i++)
            {
                isOver = false;
                
                await Task.Run(() => {
                    //下载对应的资源
                    isOver = DownLoadFile(downLoadList[i], localPath + downLoadList[i]);
                });
                if (isOver)
                {
                    //提示下载进度,将加载的进度放在外部进行
                    updatePro(++downLoadOverNum+"/"+downLoadMaxNum);
                    tempList.Add(downLoadList[i]);
                }
            }
            for (int i = 0; i < tempList.Count; i++)
            {
                downLoadList.Remove(tempList[i]);
                --reDownLoadMaxNum;
            }
        }
        //所有内容下载完成，告诉外部下载完成
        overCallBack(downLoadList.Count == 0);
        
    }
    //从远端下载文件
    public async void DownLoadABCompareFile(UnityAction<bool> overCallBack)
    {
        print(Application.persistentDataPath);
        bool isOver = false;
        int reDownLoadMaxNum = 5;
        string localPath = Application.persistentDataPath + "/";
        //从资源服务器上面下载资源对比文件
        while (!isOver &&reDownLoadMaxNum>0)
        {
            await Task.Run(() => {
                isOver = DownLoadFile("ABCompareInfo.txt", localPath + "ABCompareInfo_TMP.txt");//Application.persistentDataPath,在任何平台都是可读可写的,因为我们需要下载
            });
            --reDownLoadMaxNum;
        }
        //告诉外部下载是否成功
        overCallBack?.Invoke(isOver);
    }
    /// <summary>
    /// 解析下载下来的ab包下载文件，可以是远端的也可以是本地的文件
    /// </summary>
    public void GetRemoteABCompareFileInfo(string info,Dictionary<string,ABInfo> ABInfo)
    {
        //2读取然后对对比信息进行拆分直接从外部传入
        //string info = File.ReadAllText(Application.persistentDataPath + "/ABCompareInfo_TMP.txt");
        string[] strs = info.Split('|');
        string[] infos = null;
        for (int i = 0; i < strs.Length; i++)
        {
            infos = strs[i].Split(' ');
            //进行远端的存储
            ABInfo.Add(infos[0], new ABInfo(infos[0], infos[1], infos[2]));
        }
        print("对比文件，加载结束");
    }
    //获取本地ab包对比加载
    public void GetLocalABCompareFileInfo(UnityAction<bool> overCallBack)
    {
        //首先判断可读可写文件夹中有没有
        if (File.Exists(Application.persistentDataPath+ "/ABCompareInfo.txt"))
        {
            StartCoroutine(GetLocalABCompareFileInfoIE("file:///" + Application.persistentDataPath + "/ABCompareInfo.txt",overCallBack));
        }
        else if (File.Exists(Application.streamingAssetsPath+ "/ABCompareInfo.txt"))
        {
            //因为这个地方安卓平台无需夹对应文件协议，所以需要进行判断 本次修改
            string path =
            #if UNITY_ANDROID
            Application.streamingAssetsPath;
            #else
            "file:///" + Application.streamingAssetsPath;
            #endif
            //其次查看默认文件夹中有没有，也就是第一次进入游戏的情况
            StartCoroutine(GetLocalABCompareFileInfoIE(Application.streamingAssetsPath + "/ABCompareInfo.txt",overCallBack));
        }
        else
        {
            overCallBack(true);
        }
        //如果两个都没有的话，那么表示第一次进入，并且没有默认资源
        //StartCoroutine(GetLocalABCompareFileInfoIE());
    }
    /// <summary>
    /// 协同程序调用
    /// </summary>
    /// <param name="filePath"></param>
    /// <returns></returns>
    public IEnumerator GetLocalABCompareFileInfoIE(string filePath, UnityAction<bool> overCallBack)
    {
        //通过www获取本地的资源
        UnityWebRequest req = UnityWebRequest.Get(filePath);
        yield return req.SendWebRequest();
        Debug.Log(req.downloadHandler.text);
        if (req.result == UnityWebRequest.Result.Success)
        {
            //解析本地的信息，修改原来的函数，直接传入文本，进行解析
            GetRemoteABCompareFileInfo(req.downloadHandler.text, localABInfo);
            overCallBack(true);
        }
        else
        {
            overCallBack(false);
        }
        
        
    }
    private bool DownLoadFile(string fileName,string localPath)
    {

        try
        {
            //进行多路测试，通过判断当前属于哪个平台来决定使用哪一个文件夹，发亮的为当前平台，当前为pc平台
            string pInfo =
                        #if UNITY_IOS
                        "IOS";
                        #elif UNITY_ANDROID
                        "Android";
                        #else
                        "PC";
                        #endif
            //1.创建一个ftp链接，用于下载,虽然我访问直接用127.0.0.1，没有加前缀，但是你在写代码的时候需要加前缀
            FtpWebRequest req = FtpWebRequest.Create(new Uri(serverIP + pInfo+"/" + fileName)) as FtpWebRequest;
            //2.设置一个通信凭证 这样才能上传，如果有匿名函数可以不设置凭证，但是实际开发中，不建议设置匿名函数，因为安全性无法保证
            NetworkCredential n = new NetworkCredential("CG", "123456789");
            req.Credentials = n;
            //3.其他设置
            //设置代理为null
            req.Proxy = null;//设置代理为空防止冲突
                             //请求完毕后 是否关闭控制连接
            req.KeepAlive = false;
            //操作指令上传
            req.Method = WebRequestMethods.Ftp.DownloadFile;
            //指定传输的类型
            req.UseBinary = true;//二进制上传
                                 //4.上传文件
                                 //ftp流对象,用于写入文件
                                 //下面这部分需要修改，和上传不同
            FtpWebResponse res = req.GetResponse() as FtpWebResponse;
            Stream downLoadStream = res.GetResponseStream();//下载的流
            using (FileStream file = File.Create(localPath))
            {
                Debug.Log("本地路径：" + localPath);
                byte[] bytes = new byte[2048];//创建一个两个字节的数组，用于一点点传输数据
                int contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                                                                      //循环上传文件中的数据
                while (contentLength != 0)
                {
                    
                    //写入文件流
                    file.Write(bytes, 0, contentLength);
                    contentLength = downLoadStream.Read(bytes, 0, bytes.Length);//返回值表示读取了多少个字节
                }
                file.Close();
                downLoadStream.Close();
            }
            Debug.Log("下载成功"+fileName);
            return true;
        }
        catch (Exception ex)
        {
            Debug.Log(fileName+ "下载失败"+ ex.Message);
            return false;
        }
       
    }
    /// <summary>
    /// 创建一个类用于实现ab对比信息的存储
    /// </summary>
    public class ABInfo
    {
        public string name;
        public long size;
        public string md5;
        public ABInfo(string name,string size,string md5)
        { 
            this.name = name;
            this.size = long.Parse(size);
            this.md5 = md5;
        }
    }
    private void OnDestroy()
    {
        instance = null;//如果销毁了就要将单例赋值为空
    }
}
```

### 总结：

![image-20240824210933354](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824210933354.png)

![image-20240824210944922](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824210944922.png)

![image-20240824211007449](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824211007449.png)

![image-20240824211048506](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824211048506.png)

![image-20240824211116262](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240824211116262.png)



## Unit y5 如何做资源管理和增量更新


Unity 中的资源来源有**三个途径**：一个是**Unity自动打包资源**，一个是**Resources**，一个是**AssetBundle**。

1、Unity自动打包资源是指在Unity场景中直接使用到的资源会**随着场景被自动打包到游戏中，这些资源会在场景加载的时候由unity自动加载。这些资源只要放置在Unity工程目录的Assets文件夹下即可，程序不需要关心他们的打包和加载，这也意味着这些资源都是静态加载的。**但在实际的游戏开发中我们一般都是会动态创建GameObject，资源是动态加载的，因此这种资源其实不多。
2、Resources资源是指在Unity工程的Assets目录下面可以建一个Resources文件夹，在这个文件夹下面放置的所有资源，**不论是否被场景用到，都会被打包到游戏中**，并且可以通过Resources.Load方法动态加载。这是平时开发是常用的资源加载方式，**但是缺点是资源都直接打包到游戏包中了，没法做增量更新。**
3、**AssetBundle**资源是指我们可以通过编辑器脚本来将资源打包成多个独立的AssetBundle。这些AssetBundle和游戏包是分离的，可以通过WWW类来加载。AssetBundle的使用很灵活：可以用来做**分包发布**，例如大多数页游资源是随着游戏的过程增量下载的，或者有些手游资源过大，渠道要求发布的包限制在100M以内，那只能把一开始玩不到的内容做成增量包，**等玩家玩到的时候通过网络下载。**AssetBundle 也可以用来做我们下面讨论的自动增量更新。
Unity5相比之前的版本，AssetsBundle的打包过程有所简化。之前打包需要通过代码来设置需要打入包的资源并自己建立包的依赖关系，Unity5可以通过每个资源Inspector底部的AssetBundle下拉来指定该资源要打入哪个包，不指定就是不打包。打包过程只需要BuildPipeline.BuildAssetBundles一句话就行了，Unity5会根据依赖关系自动生成所有的包。每个包还会生成一个**manifest文件**，**这个文件描述了包大小、crc验证、包之间的依赖关系等等**，通过这个manifest打包工具在下次打包的时候可以判断哪些包中的资源有改变，只打包资源改变的包，加快了打包速度。manifest只是打包工具自己用的，发布包的时候并不需要。

关于自动生成依赖关系这个有必要提下，Unity确实会自动给你建立依赖关系，**前提是你依赖的资源必须已经在Inspector中设置了BundleName**。如果没有，Unity会把这个公用的资源重复打到多个用到的包中，因为这个公用资源不在一个独立的包中，Unity也不会智能地给你发现它是公用的，然后生成一个公用包。更深的坑在于，如果你公用的是一个FBX模型，你只给这个模型设置BundleName还不行，它用到的贴图，材质都要设，否则模型是公用了，**贴图没有公用（导致资源重复），结果贴图还是被打包到多个包中了。所以设置BundleName这个工作最好还是由编辑器脚本来完成。**

### 方案

Unity提供的就这些了，下面就自己发挥：**如何做一个方便的资源管理方案，既可以开发时方便，又可以方便发布更新包呢？开发过程全用AssetsBundle是不合适的，因为开发中资源经常添加和更新，每次添加或者更新都生成一下AssetsBundle才能运行是很麻烦的。**而且我们要做的是自动更新而不是分包下载，这也就是说在发布游戏的时候这些资源应该都是在游戏包中的，所以他们也不该从AssetsBundle加载。

分析完需求，方案也就出来了：**资源还是放在Resources下面，但是这些资源同时也会打包到AssetBundle中。代码中所有加载资源的地方都通过自己的ResourceManager来加载，由ResourceMananger来决定是调用Resources.Load来加载资源还是从AssetsBundle加载。在开发环境下(Editor)这些资源显然是直接从Resources加载的，发布的完整安装包资源也是从Resources加载，只有当有一个增量版本时，游戏主程序才会去服务器把增量的AssetBundle下载下来，然后从AssetBundle加载资源。**

### 实现

实现中我们首先要考虑的是**AssetBundle的粒度**，即每个AssetBundle包含多少资源。增量包的最小粒度就是AsssetBundle， 如果单个AssetBundle过大，只要这个AssetBundle中有一个资源改变了就需要重新下载整个AssetBundle，浪费流量和玩家的等待时间；如果单个AssetBundle过小，极端情况是每个资源一个AssetBundle，虽然实现了更新最小化，但是带来了额外开销：AssetBundle本身也是有大小的，而且查找加载AssetBundle也是需要时间的。大家都往U盘里面拷过东西，拷一个1G的文件比拷1千个1M的文件要快很多。比较合理的做法是根据逻辑来，例如每个角色可以有独立的AssetBundle，公用的一些UI资源可以打到一个AssetBundle里面，每个场景独立的UI资源可以打成独立的AssetBundle。这样做资源预加载的时候也方便，每个场景需要用到几个Bundle就加载几个Bundle,无关的资源不会被加载。

**下面要考虑的是如何来确定一个资源是从Resources加载还是AssetBundle加载。**为此我们需要一个配置文件**resourcesinfo**。这个文件随打包过程自动生成。里**面包含了资源版本号version，所有包的名字，每个包的HashCode以及每个包里面包含的资源的名字。**HashCode直接可以从Unity生成的manifest中得到（AssetBundleManifest.GetAssetBundleHash），用来检查包的内容是否发生变化**（类似资源对比文件）**。这个resourceinfo每次打包AssetBundle时都会生成一个，**发布增量时将它和新的Bundle一起全部复制到服务器上。**同时在Resources文件夹下也存一份，随完整安装包发布，**这就保证了新安装游戏的玩家手机上也有一份完整的资源配置文件**，记录了这个完整包包含的资源。

当游戏启动时，首先请求服务器检查版本号，前端用的版本号就是Resources下面的这个resourcesinfo中的version。**服务器比对这个版本号来告诉前端是否需要更新。**如果需要更新，**前端就去获取服务器端的新resourcesinfo，然后比对里面每个bundle的HashCode，把HashCode不同的bundle记录下来，然后通过WWW类来下载这些发生改变的bundle**，当然如果服务器版的**resourcesinfo中包含了本地resourceinfo中所没有的Bundle**，这些Bundle就是新增的，也需要下载下来。所有下载完成后，**前端将这个新的resourceinfo保存到本地存储中，后面前端的所有操作都将以这个resourceinfo为准而不再是Resources下面的resourceinfo了**。Resources下的resourceinfo可以退出历史舞台了，除非一种情况：本地存储的resourceinfo被认为删除了。手机端玩家清理应用的数据就会造成下载的bundle以及resourceinfo被删除。**没关系，这时候前端由于找不到外部的resourceinfo了，还会使用Resources下面的resourceinfo和服务器比对，把新的bundle重新下载下来。**

现在从哪里加载资源就很明确了：ResourceMananger先读取resourcesinfo，知道了游戏中所有的Bundle和每个Bundle包含的资源，然后去外部存储查找这些Bundle是否存在，**如果存在，就记录下这个Bundle的资源应该从外部的AssetBundle加载，如果不存在，就从内部的Resources加载。在开发过程（Editor）中，由于不存在外部存储的bundle，资源自然都是从Resources加载的，达到了我们开发方便的目的。**这个过程隐含了一点：不是所有的资源都需要有BundleName而被打包到AssetBundle中，游戏内**不需要后续更新的资源就不要设置BundleName**，它们不会被打包更新，这样的资源ResourceManager在resourceinfo中是找不到的，**直接去Resources文件夹下面读取就行了**。

加载AssetBundle，我们直接使用WWW类而不用WWW.LoadFromCacheOrDownload, 因为我们的资源在游戏开始的时候已经下载到外部存储了，不要再Download也不要再Cache。注意WWW类加载是异步的，在游戏中我们需要同步加载资源的地方就要注意把资源预加载好存在ResourceManager中，不然等用的时候加载肯定要写异步代码了。大部分时候我们应该在一个场景初始化时就预加载好所有资源，用的时候直接从ResourceManager的缓存取就可以了。

### 资源加载卸载

最后简单说下资源的加载卸载，这个网上也有很多文章介绍。
从我理解来看Resources是一个缺省自动打包的特殊AssetBundle。无论从WWW还是AssetBundle.CreateFromFile创建AssetBundle其实是创建了一个文件内存镜像。这时候是没有Asset的。AssetBundle.LoadAsset 和Resource.Load才真正创建出了Asset，而Instaniate复制了这个Asset。注意这个复制有两种，学C++的**都知道浅拷贝和深拷贝，这里的复制有的是正真的复制，有的是引用。为什么要这样呢？**因为有些游戏资源是只读的，像贴图Texture，这么大而且只读，当然不需要再去完全复制一份。但像GameObject这种资源它的属性是可以通过脚本改变的，必须要复制一份。所以一个资源从AssetBundle到场景中被实例化，其实有3块内存被创建，这3快内存的释放是有不同方法的。

**文件内存镜像**是通过AssetBundle.Unload(false)来释放的（引用关系）。
**Instaniate出来的Object内存通过Object.Destory来释放。**
**AssetBundle.Unload(true)不单会释放文件内存镜像，还会释放AssetBundle.Load创建的Assets。**这个方法是不安全的，除非你能保证这些Assets没有Object在引用，否则就出问题了。
Resources.UnloadAsset和Resources.UnloadUnusedAssets可以用来释放Asset。
下面这个图很直观：

![image-20240823205640948](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823205640948.png)

这是老Unity的图，Unity5已经把AssetBundle.Load 改成了AssetBundle.LoadAsset。这个改动让我们更明确了Load出来的是Asset这块内存区域。什么时候把Resource.Load也改了吧。

### 注意事项（坑）

Resources.Load方法传入的资源路径需是从Resources文件夹下一级开始的相对路径且不能包含扩展名；而AssetBundle.LoadAsset方法传入的资源名需是从Assets文件开始的全路径且要包含扩展名。路径不区分大小写，建议全用小写，因为AssetBundle.GetAllAssetNames方法返回的资源名都是小写的。
Unity5打包AssetBundle时会自动处理依赖关系，但是在运行时加载的时候却不会，程序需要自己处理，先加载依赖包。
AssetBundle.CreateFromFile不能加载压缩过的AssetBundle，所以我们只能用WWW来异步加载AssetBundle。
目前我用的Unity5.0.2f1的Resources.Load方法在手机端比原来慢了很多，如果以前可以不缓存每次用的时候都调用Resource.Load现在就不行了。频繁的调用会导致明显的性能开销，不知道是不是Bug。



原文链接：https://blog.csdn.net/ring0hx/article/details/46376709

### unity中资源加载方式

在Unity实际开发中，资源加载可以使用如下几种方法：

1. **Resources Folder**: 将资源放在Unity编辑器的Resources文件夹中，这样可以方便地加载不需要动态生成或者频繁变更的资源。`Resources`**这种动态加载方式是只读的，在游戏打包后，就无法对文件夹的内容进行修改**

   

2. **AssetBundle**: 对于大量或大型资源，使用AssetBundle可以有效地进行资源的异步加载和管理，这有助于提高游戏的加载效率和性能。

3. **Addressables**: Unity的可寻址系统（Addressables）是一种更灵活的资源管理系统，它支持在不影响游戏性能的情况下，异步加载和卸载资源。

4. **UnityWebRequest**: 当需要从服务器加载资源时，可以使用UnityWebRequest类来从网络请求资源。

5. **Prefab**: 预制体（Prefab）是Unity中常用的资源加载方式，用于实例化游戏对象，可以用来快速部署和重用组件和资产。

6. **ObjectPooler**: 对于频繁创建和销毁的游戏对象，使用对象池（ObjectPooler）是一种有效的资源管理策略。

### 两种主要的加载方式

**前言：**
在游戏开发学习初期，游戏体量较小，如果游戏场景需要Asset中的资源，我们可能会通过拖动的方式，将其添加到游戏场景中。而到了实际工作中，会发现再这样做就会使得各种拖动的资源非常复杂，难以查找与维护

**关于资源：**

在Unity中，在Asset文件下的文件都可以称为游戏的资源，例如预制体、模型、材质、纹理、音频、视频、数据文档、场景等等
为了避免这样的情况，Unity游戏引擎为我们提供了很多种加载Asset中资源的方式，而我们最常用的主要是Resources和AssetBundle两种方式

资源加载的两种常用方式

#### 第一种：Resources：

首先第一种比较简单好用的就是Resources方式，只需要将需要加载到场景中的资源放置再Asset目录下的Resources文件中，就可以通过Unity提供的API来加载这些资源了

注意：

首先Resources方式加载Asset资源只能加载位于命名为Resources的文件夹下的资源，因此如果要使用这种加载方式时，首先需要先创建命名为Resources的文件夹，然后将需要加载的资源放置于该文件夹下
另一点比较关键的是，Resources这种动态加载方式是只读的，在游戏打包后，就无法对文件夹的内容进行修改
使用的方式，首先创建一个文件夹在Asset目录下，并命名为Resources,同时在场景中创建一个Cube重命名为Test，并将其拖入到刚刚创建的Resources文件夹内：

![image-20240823213801926](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823213801926.png)


接下来，就可以删除场景中的Test对象，然后通过脚本来动态获取这个Test预制体，当然可以选择在脚本中Public一个GameObject，但是这不是本次学习的目标，我们需要通过Resources的一些方法来进行将该预制体加载到场景内：

**关于Resources的方法：**

FindObjectsOfTypeAll:返回某一种类型的所有资源
Load：通过路径加载资源
LoadAll：加载该Resources下的所有资源
LoadAsync：异步加载资源，通过协程实现
UnloadAsset：卸载加载的资源
UnloadUnusedAssets：卸载在内存分钟未使用的资源
本次来使用一个简单的同步加载的案例，来演示一下用法，首先创建一个脚本拖入场景中的任意一个物体上，然后通过脚本来加载到Resources文件夹下的Test预制体，并在场景中实例化出一个Test物体，并将坐标归零：

```c#
void Start()
{
    AddObjToScene();
}

/// <summary>
/// 定义一个加载Resources文件夹内资源的方法
/// </summary>
public void AddObjToScene()
{
    //将资源加载到游戏进程中
    var obj = Resources.Load("Test") ;
    //实例化一个资源到场景中
    GameObject instance = Instantiate(obj) as GameObject;
    instance.transform.position = Vector3.zero;
    
    obj = null;
    Resources.UnloadUnusedAssets();    
}

```

#### 第二种：通过AssetBundle来完成：

首先要了解什么是AssetBundle，与Resources不同，AssetBundle主要是用于热更新

对于Unity的热更可以通过两方面来进行，首先是音频资源的更新，需要通过AB包来进行，另一方面，是需要通过Lua 语言来进行C#脚本的更新

![image-20240823213912025](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823213912025.png)

使用流程：

**1,安装AB包插件**

要将资源打包成为AB包，需要通过Unity官方提供的插件来完成，在导航栏中Window选项下找到Package Manager，打开下面的资源包管理器，并在其中找到Asset Bundle Browser插件，点击Install安装即可：

![image-20240823213936849](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823213936849.png)

**2，将需要的资源打包**

点击需要打包的物体，将其调整为可以打包的格式，以之前创建的预制体Test为案例，在Asset中找到预制体点击后，可以在Inspector面板看到AssetBundle选项，如图：

![image-20240823214003725](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214003725.png)

点击下标选择New选项创建一个AB包资源命名为abtest：

![image-20240823214102896](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214102896.png)

除了这样一个一个的添加，也可以批量点击来添加，但是注意，批量后是在一个命名的资源下，更简单的，批量操作不是为了快捷操作，而是一种打包方式，就像Char与String结构：

完成打包后，可以在导航栏中Window中看到AssetBundle Browser选项，点击后打开一个AB包管理窗口，就可以看到我们刚刚命名的abtest包内的Test预制体![image-20240823214118862](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214118862.png)


在我们完成对所有想添加的资源的添加操作后，就可以对这些资源实施打包动作了，在AssetBundle窗口上面的三个选项中选择中间的Build选项，就可以对资源打包进行一些参数调整来满足自己的打包条件：

![image-20240823214129559](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214129559.png)

这里面的控制参数还是挺多的，这里大概解释一下：

Build Target：打包的平台选择，默认是Window
Output Path: 打包保存位置
Clear Folder：是否清空打包保存位置文件夹（尽量勾选，不然每一次打包都会多一些资源）
Copy to StreamingAsset:是否复制打包后的资源到Unity项目中
接下来在第Advanced Setting中有几个比较重要的参数，首先第一个就是Commpression，即打包出去的资源的压缩方式，可以根据需求选择你想要的压缩方式，但是如果你不知道如果选择，那么LZ4应该是一种比较适合的方式：

![image-20240823214259192](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214259192.png)
完成操作后，点击Build按钮就可以完成AB包的打包工作，然后我们可以在Asset的同级目录看到刚刚创建的AssetBundles文件：![image-20240823214328156](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214328156.png)

点击进去就可以查看到我们打包的资源：![image-20240823214354353](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214354353.png)

这样就完成了整个的打包工作，而同时，如果你勾选了Copy to StreamingAsset，在你的项目中的Asset文件中同样会有一份拷贝的打包的AB包资源![image-20240823214443298](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20240823214443298.png)


如果你勾选后没有，在Asset面板下右键选择Refresh刷新一下即可

3，调用场景中的AB包中的资源

关于如何从服务器获取AB包到客户端是一个比较复杂的技术，所以我们直接在本地使用AB包来讲解关于AB包中资源的调用，由于我们在前面AB包打包的时候在项目中拷贝了一份在项目中，所以直接以其为案例来开始介绍：

关于具体的加载细节可以通过脚本来查看，本次给出一个简单的同步加载案例：

```c#
void Start()
{
    AddObjToScene();
}

/// <summary>
/// 定义一个加载Resources文件夹内资源的方法
/// </summary>
public void AddObjToScene()
{
    //首先加载包，加载我们创建的abtest包，而Application.streamingAssetsPath为我们拷贝的路径的接口写法
    AssetBundle abTest = AssetBundle.LoadFromFile(Application.streamingAssetsPath + "/" + "abTest");

    var test= abTest.LoadAsset("Test");

    GameObject obj = Instantiate(test) as GameObject;
    obj.transform.position = Vector3.zero;
    //卸载所有加载的AB包，如果参数为True，则同时将AB包加载的资源一并卸载
    AssetBundle.UnloadAllAssetBundles(false);
}
```

我们发现其基本写法是和Resources资源加载方式基本相同，主要流程都是加载资源，实例资源，最后卸载资源,但是两者在细节方面还是有稍微的不同，需要在使用时注意

#### 进阶

1、使用Resources写一个资源管理器

首先声明一个单例类，使得全局唯一

接下来可以定义一个字典作为资源管理容器

当某一个功能需要加载资源，通过该资源管理器来获取，如果字典中已经存在，则直接获取，如果字典中不存在，则通过Resources来加载出资源，并且同时存储于字典中，最后返回资源：



```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// 资源管理器
/// </summary>
public class ResourceMgr 
{
/*--------单例模式-------------*/
    private static ResourceMgr s_instance;
    public static ResourceMgr instace
    {
        get
        {
            if(s_instance==null)
            {
                s_instance = new ResourceMgr();
            }
            return s_instance;
        }
    }
/*使用字典保存需要加载的物体*/
Dictionary<string, object> m_res = new Dictionary<string, object>();
//T为加载物体的游戏数据类型
public T LoadRes<T> (string resPath) where T:Object
{
    if(m_res.ContainsKey(resPath))
    {
        return m_res[resPath] as T;
    }
    T t = Resources.Load<T>(resPath);
    m_res[resPath] = t;
    return t;
}   
}
```

完成后，需要进行测试

我们依旧在Test脚本中将位于Resources文件下的Test预制体加载到场景中，并且坐标归零，如果成功加载，则说明脚本产生了效果：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{  
    void Start()
    {
        AddObjToScene();
    }
    /// <summary>
    /// 测试资源加载器
    /// </summary>
    public void AddObjToScene()
    {
        string resPath = "Test";
        GameObject obj = ResourceMgr.instace.LoadRes<GameObject>(resPath);
        GameObject instance = Instantiate(obj);
        instance.transform.position = Vector3.zero;
    }     
}
```

最终成功在场景中生成一个Test预制体

2、关于资源的异步加载

由于资源管理器使用了Resources来说明，资源的异步加载就使用AssetBundle来完成，一般我们说到异步，就避免不了协程，通过协程来完成这样一个案例：

协程：

协程在Unity是一个很重要的概念，其存在意义是协助程序来完成一个同步功能
任一时间同一脚本中只能存在一个运行的协程
协程不是线程，协程依旧在主线程中运行
协程是基于迭代器来实现的
接下来就使用异步加载来完成一个测试：



```c#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
void Start()
{
    AddObjToScene();
}
/// <summary>
/// 测试资源加载器
/// </summary>
public void AddObjToScene()
{
    //开启协程
    StartCoroutine(LoadABRes("abtest", "Test"));
    AssetBundle.UnloadAllAssetBundles(false);
}

IEnumerator LoadABRes(string ABName,string resName)
{
    //加载资源包
    AssetBundleCreateRequest abtest = AssetBundle.LoadFromFileAsync(Application.streamingAssetsPath + "/" + ABName);

    yield return abtest;
    //加载资源
    AssetBundleRequest test = abtest.assetBundle.LoadAssetAsync(resName,typeof(GameObject));
    yield return test;

    GameObject obj = Instantiate(test.asset) as GameObject;
    obj.transform.position = Vector3.zero;
}
}
```

#### 总结

场景资源的加载有很多，但是最常用的还是Resources和AssetBundle，两者有不同的应用场景，同时也有不同的使用方式，可以根据情况来选择

原文链接：https://blog.csdn.net/xinzhilinger/article/details/115408934

### 关于资源加载的一些问题

#### 在实际开发中unity资源加载用的框架是什么样的，同时在打包发布过程又是如何处理resources文件夹下面的资源的

在 Unity 的实际开发中，资源加载通常使用以下几种常见的框架和方法，结合 `Resources` 文件夹的处理方式：

##### 资源加载框架

1. **Resources.Load()**
   - Unity 提供的基本资源加载方法。开发者可以将资源放在 `Resources` 文件夹中，使用 `Resources.Load("path/to/resource")` 方法在运行时加载资源。
   - 优点：简单易用，适合小型项目和快速原型制作。
   - 缺点：所有放在 `Resources` 文件夹中的资源在构建时都会被打包，可能导致构建体积过大。

2. **Asset Bundles**
   - Asset Bundles 是 Unity 提供的一种高级资源管理方式，允许开发者将资源打包成独立的文件，在需要时动态加载。
   - 优点：可以按需加载资源，减少初始下载包的大小，适合大型项目和需要动态更新的游戏。
   - 处理方式：
     - 在编辑器中创建 Asset Bundles，使用 `BuildPipeline.BuildAssetBundles` 方法打包。
     - 在运行时使用 `AssetBundle.LoadAsset()` 方法加载资源。

3. **Addressables**
   - Addressables 是 Unity 提供的一个更高级的资源管理系统，旨在简化资源的加载和管理。
   - 优点：支持异步加载、资源版本控制、自动处理依赖关系，适合复杂项目。
   - 处理方式：
     - 使用 Addressables API 定义和管理资源，使用 `Addressables.LoadAssetAsync<T>()` 方法加载资源。
     - 可以通过 Addressables Groups 管理资源和打包。

##### 打包发布过程中的 Resources 文件夹处理

1. **打包时处理**
   - 在构建时，Unity 会将 `Resources` 文件夹中的所有资源打包到最终的构建中。这些资源会被序列化并包含在游戏的资源文件中。
   - 如果资源在代码中没有被引用，它们仍然会被打包，但在运行时无法加载。

2. **构建体积优化**
   - 开发者应定期检查 `Resources` 文件夹中的资源，确保只放置必要的资源，避免不必要的资源导致构建体积过大。
   - 可以使用 Unity 的 Profiler 工具分析资源使用情况，找出未使用或冗余的资源。

3. **运行时加载**
   - 在游戏运行时，开发者可以使用 `Resources.Load()` 方法加载 `Resources` 文件夹中的资源。所有加载的资源在使用后需要手动释放，以避免内存泄漏。

##### 总结

在 Unity 的实际开发中，资源加载框架主要包括 `Resources.Load()`、Asset Bundles 和 Addressables。选择合适的框架取决于项目的规模和复杂性。对于 `Resources` 文件夹，Unity 会在打包时将其中的资源打包到最终构建中，开发者需要合理管理和使用这些资源，以优化构建体积和运行时性能。

#### 你说到要Resources确保只放置必要的资源，那么其他的资源如何放置

在 Unity 中，为了更好地管理和组织资源，开发者可以使用多种方法和文件夹结构来放置资源，而不仅仅是依赖于 `Resources` 文件夹。以下是一些常见的最佳实践和建议：

##### 1. 使用不同的文件夹结构

- **按类型组织**：
  - 将资源按类型分开存放，例如：
    - `Assets/Models/`
    - `Assets/Textures/`
    - `Assets/Audio/`
    - `Assets/Animations/`
  - 这种方式使得资源管理更加清晰，便于查找和维护。

- **按功能模块组织**：
  - 根据游戏的不同模块或功能进行组织，例如：
    - `Assets/Gameplay/`
    - `Assets/UI/`
    - `Assets/Environment/`
  - 这种方式可以帮助团队成员快速找到相关资源。

##### 2. 使用 Asset Bundles

- **创建 Asset Bundles**：
  - 将资源打包到 Asset Bundles 中，根据游戏的需求和场景进行分组。
  - 通过 Asset Bundles，可以按需加载资源，避免将所有资源打包到一个大的构建中。

##### 3. 使用 Addressables

- **利用 Addressables 系统**：
  - 使用 Addressables 进行资源管理，可以将资源放在任何文件夹中，并通过 Addressables Groups 进行管理。
  - Addressables 允许异步加载和版本控制，适合大型项目和动态更新。

##### 4. 使用 Prefabs

- **创建 Prefabs**：
  - 将常用的 GameObject 组合成 Prefabs，并放在 `Assets/Prefabs/` 文件夹中。这可以减少重复工作，并方便在场景中使用。

##### 5. 使用 StreamingAssets

- **特定文件存放**：
  - 对于需要在运行时直接访问的文件（如视频、文本文件等），可以将它们放在 `StreamingAssets` 文件夹中。这些文件不会被 Unity 的资源管理系统处理，但会被原样复制到构建中。

##### 6. 资源版本控制

- **使用版本控制系统**：
  - 使用 Git、SVN 等版本控制系统管理资源文件，确保团队成员能够协同工作，并避免资源冲突。

##### 7. 定期清理和优化

- **定期审查资源**：
  - 定期检查项目中的资源，删除未使用的资源，优化资源的大小和格式，以提高构建性能和运行时性能。

##### 总结

通过合理组织资源、利用 Asset Bundles 和 Addressables、使用 Prefabs 和 StreamingAssets，开发者可以有效管理 Unity 项目中的资源。这不仅有助于保持项目的整洁性和可维护性，还能提高游戏的性能和加载速度。

#### 我有一个问题，在开发的过程中我们四将资源存放在Resources中的，加载也是从resources中加载的，但是当我们要发布的时候，如何处理resources这个文件夹下面的资源

在 Unity 开发中，如果你在 `Resources` 文件夹中存放了资源，并且在加载时使用了 `Resources.Load()` 方法，那么在发布游戏时，你需要考虑以下几个方面来处理 `Resources` 文件夹中的资源：

##### 1. 检查和优化资源

- **审查资源**：
  - 在发布之前，仔细检查 `Resources` 文件夹中的所有资源，确保只保留必要的资源。
  - 删除未使用的资源，以避免无谓的构建体积。

- **优化资源**：
  - 对于纹理、音频等资源，考虑压缩格式和分辨率，确保它们在质量和大小之间取得良好的平衡。

##### 2. 使用 Profiler 工具

- **分析资源使用情况**：
  - 使用 Unity Profiler 工具分析项目，查看哪些资源被加载，哪些未被使用。这样可以帮助你识别冗余的资源。

##### 3. 考虑使用 Asset Bundles 或 Addressables

- **转移到 Asset Bundles 或 Addressables**：
  - 如果项目规模较大，考虑将 `Resources` 文件夹中的资源转移到 Asset Bundles 或使用 Addressables 系统进行管理。
  - 这样可以按需加载资源，减少初始构建体积，并提高资源管理的灵活性。

##### 4. 资源引用管理

- **确保引用完整**：
  - 确保在代码中对 `Resources` 文件夹中的资源有明确的引用。如果某个资源在代码中没有被引用，Unity 可能会在构建时将其排除。

##### 5. 发布构建

- **构建设置**：
  - 在 Unity 中，打开 `File > Build Settings`，选择目标平台并设置构建选项。
  - 确保在构建过程中选择了适当的场景和资源。

- **测试构建**：
  - 在发布之前，进行测试构建，确保所有 `Resources` 中的资源都能正常加载，并且没有出现缺失或错误。

##### 6. 文档和说明

- **记录和说明**：
  - 如果团队中有多名开发者，确保记录和说明 `Resources` 文件夹的使用情况，便于后续维护和更新。

##### 总结

在发布之前，仔细审查和优化 `Resources` 文件夹中的资源是至关重要的。考虑将资源迁移到 Asset Bundles 或 Addressables，以提高资源管理的灵活性和效率。确保所有引用完整，并在构建之前进行充分的测试，以确保资源能够正常加载。

#### 如何利用下面两者的优点，规避两者的缺点

Resources.Load()

Unity 提供的基本资源加载方法。开发者可以将资源放在 Resources 文件夹中，使用 Resources.Load("path/to/resource") 方法在运行时加载资源。
优点：简单易用，适合小型项目和快速原型制作。
缺点：所有放在 Resources 文件夹中的资源在构建时都会被打包，可能导致构建体积过大。
Asset Bundles

Asset Bundles 是 Unity 提供的一种高级资源管理方式，允许开发者将资源打包成独立的文件，在需要时动态加载。
优点：可以按需加载资源，减少初始下载包的大小，适合大型项目和需要动态更新的游戏。
处理方式：
在编辑器中创建 Asset Bundles，使用 BuildPipeline.BuildAssetBundles 方法打包。
在运行时使用 AssetBundle.LoadAsset() 方法加载资源。

要结合 `Resources.Load()` 和 Asset Bundles 的优点，同时规避它们的缺点，可以采用以下策略：

##### 1. 资源组织

- **小型资源使用 `Resources`**：
  - 对于小型项目或快速原型制作，可以继续使用 `Resources` 文件夹来存放少量的、经常使用的资源。这些资源应该是相对小且频繁使用的，以避免构建体积过大。

- **大型资源使用 Asset Bundles**：
  - 对于大型项目，尤其是那些包含大量资产或需要动态更新的游戏，使用 Asset Bundles 来组织和管理资源。将不常用或较大的资源放在 Asset Bundles 中，以减少初始构建体积。

##### 2. 资源加载策略

- **优先使用 Asset Bundles**：
  - 在运行时，优先尝试从 Asset Bundles 中加载资源。这样可以确保只在需要时加载资源，从而有效管理内存和加载时间。

- **后备方案使用 `Resources.Load()`**：
  - 如果某个资源在 Asset Bundles 中不可用，可以使用 `Resources.Load()` 作为后备方案。这可以确保即使在没有 Asset Bundles 的情况下，游戏仍然能够正常运行。

##### 3. 资源版本控制

- **使用 Asset Bundles 进行版本管理**：
  - 利用 Asset Bundles 的版本控制功能，确保能够在更新时平滑地替换资源，而不需要重新下载整个包。

- **定期检查和更新 `Resources` 文件夹**：
  - 定期审查 `Resources` 文件夹中的资源，确保只保留必要的资源，避免冗余和过大的构建体积。

##### 4. 资源打包和构建流程

- **自动化构建流程**：
  - 使用 Unity 的 `BuildPipeline` 自动化 Asset Bundles 的构建过程，确保在每次构建时都能生成最新的 Asset Bundles。

- **集成测试**：
  - 在构建后进行集成测试，确保所有 Asset Bundles 和 `Resources` 中的资源都能正常加载并且没有缺失。

##### 5. 性能监控和优化

- **使用 Profiler 工具**：
  - 在开发过程中，使用 Unity Profiler 工具监控资源的加载和内存使用，确保在使用 Asset Bundles 和 `Resources` 时性能达到最佳。

- **动态加载和卸载**：
  - 动态加载和卸载 Asset Bundles 中的资源，以保持内存使用的高效性。在不再需要某些资源时，及时卸载它们。

##### 6. 文档和团队协作

- **记录资源使用规范**：
  - 为团队成员提供关于如何使用 `Resources` 和 Asset Bundles 的文档，以确保大家遵循相同的资源管理策略。

##### 总结

通过将 `Resources.Load()` 和 Asset Bundles 结合使用，可以充分利用两者的优点，同时规避各自的缺点。小型项目可以使用 `

## 捕鱼达人：实际案例开发

**项目地址：**

链接：https://pan.baidu.com/s/1W3C8u9h5hyR2Udj0MNH4fg?pwd=1234 
提取码：1234 

 

### 下载xlua

 链接：[xLua: xlua官方库 (gitee.com)](https://gitee.com/LonelyTears/xLua)

下载之后将![image-20240420125423697](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420125423697.png)

复制到我们的unity中的assets文件夹下面

### XLUA环境配置

在没有配置之前只有两个选项

![image-20240420131251107](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420131251107.png)

打开宏，保证功能正常正常使用

输入**HTFIX_ENABLE**

![image-20240420131353531](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420131353531.png)

开启之后会多一个功能，注入lua代码

![image-20240420131519228](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420131519228.png)

如果代码发生了改吗，在运行之前需要点击，generate code,然后运行hotfixinject，但是在这个之前需要导入工具

![image-20240420131739879](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420131739879.png)

复制到asseets同级目录下

**注意：目录要用英文，用中文会导致未知的错误**

出现啊下面两段提示，表示代码生成和注入成功

![image-20240420132020449](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420132020449.png)

如果出现下面这种提示，就表示没有进行代码的生成

![image-20240420132553663](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420132553663.png)

### **简单的使用**

对于需要热更新的代码需要打上[Hotfix]标签

![image-20240420132349273](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420132349273.png)

对于需要更改的方法需要加上[luacallcsharp]标签，对于那些需要再lua中调用修改的方法

**xlua.private_accessible(CS.HotFixEmpty)：**保证lua脚本中可以访问c#脚本中的代码

**xlua.hotfix(CS.FishMaker,"MakeFishes",function(self)：**第一个参数表示需要修改的脚本，第二个为需要修改的函数，第三个为目标函数

### 发布的问题

如果要进行打包，发布。

在打包盒发布之前要将example删除，否则会导致很多错误

![image-20240420132951893](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420132951893.png)

删除之后会出现一堆的报错，直接点击xlua中的清理，然后重新生成和注入

![image-20240420133049219](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420133049219.png)

### 开始打补丁

首先创建代码

![image-20240420133257628](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420133257628.png)



然后创建lua脚本,lua脚本的编码格式要和读取的格式匹配，可以试试修改成utf-8

![image-20240420133635108](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420133635108.png)

如果lua脚本不是放在resource下面，就需要自己定义一个加载器

```c#
    private byte[] MyLoader(ref string filePath)
    {
        //@让\被正确识别为文件路径
        string absPath = @"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\" + filePath + ".lua.txt";
        return System.Text.Encoding.UTF8.GetBytes( File.ReadAllText(absPath));
    }
```

简单的执行代码

```c#
    private LuaEnv luaEnv;
    //用于存储预制体
    // Start is called before the first frame update
    private void Awake()
    {
        //要将打补丁的放在所有的游戏逻辑之前 
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyLoader);
        luaEnv.DoString("require 'fish'");
    }
```

完整代码

注意：热更新代码需要放在awake，因为需要先加载热更新代码，然后再执行游戏逻辑，start的执行受电脑性能的影响，所以放在awake中最好

```c#
        //要将打补丁的放在所有的游戏逻辑之前 
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyLoader);
        luaEnv.DoString("require 'fish'");
```



```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.CompilerServices;
using UnityEngine;
using UnityEngine.Networking;
using XLua;

public class HotFixScript : MonoBehaviour
{
    private LuaEnv luaEnv;
    //用于存储预制体
    public static Dictionary<string,GameObject> prefabDict = new Dictionary<string,GameObject>();
    // Start is called before the first frame update
    private void Awake()
    {
        //要将打补丁的放在所有的游戏逻辑之前 
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyLoader);
        luaEnv.DoString("require 'fish'");
    }
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
    }
    private byte[] MyLoader(ref string filePath)
    {
        //@让\被正确识别为文件路径
        string absPath = @"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\" + filePath + ".lua.txt";
        return System.Text.Encoding.UTF8.GetBytes( File.ReadAllText(absPath));
    }
    private void OnDisable()
    {
        luaEnv.DoString("require 'fishDispose'");
    }
    private void OnDestroy()
    {
        luaEnv.Dispose();
    }
    [LuaCallCSharp]
    //最好写在一个单独的管理脚本上面
    public void LoadResource(string resName,string filePath)
    {
        Debug.Log("正在加载资源"+resName);
        StartCoroutine(LoadResourceCorotine( resName,  filePath));
        //本地加载方式public static void LoadResource(string resName,string filePath)
        //AssetBundle ab = AssetBundle.LoadFromFile(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\FishMaster\AssetBundles\"+filePath);
        //GameObject gameObject = ab.LoadAsset<GameObject>(resName);
        //prefabDict.Add(resName, gameObject);//存储方便取用
    }
    IEnumerator LoadResourceCorotine(string resName, string filePath)//http://localhost:8080/
    {
        UnityWebRequest request = UnityWebRequestAssetBundle.GetAssetBundle(@"http://localhost:8080/AssetBundles/" + filePath);
        yield return request.SendWebRequest();//发送请求，连接网络，下载
        AssetBundle ab = (request.downloadHandler as DownloadHandlerAssetBundle).assetBundle;//转换成ab变量
        GameObject fish = ab.LoadAsset<GameObject>(resName);
        
        Debug.Log("加入"+resName);
        Debug.Log(fish);
        prefabDict.Add(resName, fish);//存储方便取用
        Debug.Log("实际获取的物体" + prefabDict[resName]+"数量"+prefabDict.Count);
    }

    /// <summary>
    /// 获取资源
    /// </summary>
    /// <param name="goName"></param>
    /// <returns></returns>
    [LuaCallCSharp]
    public static GameObject GetGameObject(string goName)
    {
        Debug.Log("获取----"+goName);
        Debug.Log("这里是没有获取吗" +"数量" +prefabDict.Count);
        if (prefabDict.Count > 0)
        return prefabDict[goName];
        else return null;
    }
}

```

**补丁流程**

对于需要打补丁的代码是需要提前打上标签的，如果无法预知哪些类需要打补丁，可以添加几个空类，用于添加新的功能

![image-20240420134336532](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420134336532.png)

目前需求，就是修改小鱼只能直行的情况

首先在我们需要热补丁的类上面打上标签

我们需要修改的类如下

```c#
using UnityEngine;
using System.Collections;
using XLua;
[Hotfix]
public class FishMaker : MonoBehaviour
{
    public Transform fishHolder;
    public Transform[] genPositions;
    public GameObject[] fishPrefabs;
    public HotFixScript hotFixScript;
    public float fishGenWaitTime = 0.5f;
    public float waveGenWaitTime = 0.3f;

    void Start()
    {
        //发布之后就不能再c#里面修改，只能修改一些应急问题
        InvokeRepeating("MakeFishes", 0, waveGenWaitTime);
        //AssetBundle ab = AssetBundle.LoadFromFile("文件路径");
        //GameObject gameObject = ab.LoadAsset<GameObject>("资源名称");
    }
    [LuaCallCSharp]
    void MakeFishes()
    {
        int genPosIndex = Random.Range(0, genPositions.Length);
        int fishPreIndex = Random.Range(0, fishPrefabs.Length);
        int maxNum = fishPrefabs[fishPreIndex].GetComponent<FishAttr>().maxNum;
        int maxSpeed = fishPrefabs[fishPreIndex].GetComponent<FishAttr>().maxSpeed;
        int num = Random.Range((maxNum / 2) + 1, maxNum);
        int speed = Random.Range(maxSpeed / 2, maxSpeed);
        //int moveType = Random.Range(0, 2);      //0：直走；1：转弯
        int moveType = 0;      //0：直走；1：转弯 -- 需要修改的地方
        int angOffset;                          //仅直走生效，直走的倾斜角
        int angSpeed;                           //仅转弯生效，转弯的角速度
        if (moveType == 0)
        {
            angOffset = Random.Range(-22, 22);
            StartCoroutine(GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset));
        }
        else
        {
            if (Random.Range(0, 2) == 0)        //是否取负的角速度
            {
                angSpeed = Random.Range(-15, -9);
            }
            else
            {
                angSpeed = Random.Range(9, 15);
            }
            StartCoroutine(GenTrunFish(genPosIndex, fishPreIndex, num, speed, angSpeed));
        }
    }

    IEnumerator GenStraightFish(int genPosIndex,int fishPreIndex,int num,int speed,int angOffset)
    {

        for (int i = 0; i < num; i++)
        {
            GameObject fish = Instantiate(fishPrefabs[fishPreIndex]);
            fish.transform.SetParent(fishHolder, false);
            fish.transform.localPosition = genPositions[genPosIndex].localPosition;
            fish.transform.localRotation = genPositions[genPosIndex].localRotation;
            fish.transform.Rotate(0, 0, angOffset);
            fish.GetComponent<SpriteRenderer>().sortingOrder += i;
            fish.AddComponent<Ef_AutoMove>().speed = speed;
            yield return new WaitForSeconds(fishGenWaitTime);
        }
    }

    IEnumerator GenTrunFish(int genPosIndex, int fishPreIndex, int num, int speed, int angSpeed)
    {
        for (int i = 0; i < num; i++)
        {
            GameObject fish = Instantiate(fishPrefabs[fishPreIndex]);
            fish.transform.SetParent(fishHolder, false);
            fish.transform.localPosition = genPositions[genPosIndex].localPosition;
            fish.transform.localRotation = genPositions[genPosIndex].localRotation;
            fish.GetComponent<SpriteRenderer>().sortingOrder += i;
            fish.AddComponent<Ef_AutoMove>().speed = speed;
            fish.AddComponent<Ef_AutoRotate>().speed = angSpeed;
            yield return new WaitForSeconds(fishGenWaitTime);
        }
    }
}
```

编写lua脚本

​    local moveType = math.floor(UnityEngine.Random.Range(0,2))--这段代码就是修改直走的

加入了lua代码，那么执行的时候就会执行这段代码，绕过原先的代码

（下面的是完整代码）

```lua
print('123')
--导入，引擎
local UnityEngine = CS.UnityEngine
xlua.hotfix(CS.FishMaker,"MakeFishes",function(self)
	local genPosIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
	local fishPreIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
    local maxNum
    local angSpeed
    local maxSpeed
    local fishMaker = CS.FishMaker
    local newfish = CS.HotFixScript.GetGameObject('bigeyefish')
    if newfish ~= nil then
        self.fishPrefabs:SetValue(newfish,1);
    end
    
    maxNum = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxNum
    maxSpeed = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxSpeed
    local num = UnityEngine.Random.Range((maxNum/2)+1,maxNum)
    local speed = UnityEngine.Random.Range(maxSpeed/2,maxSpeed)
    speed = math.floor(speed)
    local moveType = math.floor(UnityEngine.Random.Range(0,2))--这段代码就是修改直走的
    num = math.floor(num)
    if moveType == 0 then
        local angOffset = math.floor(UnityEngine.Random.Range(-22,22))
        --print(genPosIndex..fishPreIndex..num..speed..angOffset)
        print("dfdf1"..angOffset)
        self:StartCoroutine(self:GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset))
    else
        if UnityEngine.Random.Range(0,2) == 0 then
            angSpeed = math.floor(UnityEngine.Random.Range(-15,-9))
        else
            angSpeed = math.floor(UnityEngine.Random.Range(9,15))
        end
        --print(genPosIndex..fishPreIndex..num..speed..angSpeed)
        print("dfdf2"..angSpeed)
        self:StartCoroutine(self:GenTrunFish(genPosIndex, fishPreIndex, num, speed,angSpeed ))
    end
end
)

xlua.hotfix(CS.FishMaker,"Start",function(self)
    self:InvokeRepeating("MakeFishes", 0, self.waveGenWaitTime)
    print("-----------------------------------------------------")
    --CS.HotFixScript.LoadResource("cy","gameobject\\enemy.ab")--从本地加载的方式
    --local newfish = self.hotFixScript:GetGameObject('cy')4
    self.hotFixScript:LoadResource('bigeyefish',"gameobject\\enemy.ab")--从网上加载，将loadresources从静态方法修改成普通方法，调用的方式也发生了改变
end
) 
xlua.private_accessible(CS.HotFixEmpty)--保证可以访问其中的私有变量
xlua.hotfix(CS.HotFixEmpty,"Start",function(self)
        print("this is HotFixEmpty")
        self:BehaviourMethod()
end
)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",function(self,other) 
    
end
)

xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",function(self)
    print("this is BehaviourMethod")
end
)
```

小结：

对于c#中静态的代码我们需要通过(.)访问，对于普通的成员变量需要通过（：）进行访问

对于c#中的代码需要通过local UnityEngine = CS.UnityEngine进行调用

xlua.private_accessible(CS.HotFixEmpty)--保证可以访问其中的私有变量

执行的时候如果报错如下

![image-20240420140305396](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420140305396.png)

上面错误的原因在于我们调用了一个已经释放的对象

![image-20240420140952611](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240420140952611.png)

所以我们需要写一个代码去释放

```lua
hotfix(CS.FishMaker,'MakeFishes',nil)
xlua.hotfix(CS.FishMaker,"Start",nil)
xlua.hotfix(CS.HotFixEmpty,"Start",nil)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",nil)
xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",nil)
luaEnv.Dispose();
```

这段代码在hotfixscripts下执行

```c#
    private void OnDisable()
    {
        luaEnv.DoString("require 'fishDispose'");
    }
```

打补丁汇总

**注：先采用本地加载的方式**

第一个脚本：HotFixScripts

![image-20240422224106264](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240422224106264.png)

code:

```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.CompilerServices;
using UnityEngine;
using UnityEngine.Networking;
using XLua;

public class HotFixScript : MonoBehaviour
{
    private LuaEnv luaEnv;
    //用于存储预制体
    public static Dictionary<string,GameObject> prefabDict = new Dictionary<string,GameObject>();
    // Start is called before the first frame update
    private void Awake()
    {
        //要将打补丁的放在所有的游戏逻辑之前 
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyLoader);
        luaEnv.DoString("require 'fish'");
    }
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
    }
    private byte[] MyLoader(ref string filePath)
    {
        //@让\被正确识别为文件路径
        string absPath = @"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\" + filePath + ".lua.txt";
        return System.Text.Encoding.UTF8.GetBytes( File.ReadAllText(absPath));
    }
    private void OnDisable()
    {
        luaEnv.DoString("require 'fishDispose'");
    }
    private void OnDestroy()
    {
        luaEnv.Dispose();
    }
    [LuaCallCSharp]
    //最好写在一个单独的管理脚本上面
    public void LoadResource(string resName,string filePath)
    {
        Debug.Log("正在加载资源"+resName);
        public static void LoadResource(string resName,string filePath)
        AssetBundle ab = AssetBundle.LoadFromFile(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\FishMaster\AssetBundles\"+filePath);//这里的地址表示放资源的位置
        GameObject gameObject = ab.LoadAsset<GameObject>(resName);
        prefabDict.Add(resName, gameObject);//存储方便取用
    }
    /// <summary>
    /// 获取资源
    /// </summary>
    /// <param name="goName"></param>
    /// <returns></returns>
    [LuaCallCSharp]
    public static GameObject GetGameObject(string goName)
    {
        if (prefabDict.Count > 0)
        return prefabDict[goName];
        else return null;
    }
}

```

第二个脚本HotFixEmpty

这个脚本存在的原因，这个脚本存在的原因就是我们对于一些无法预测到需要热更新的部分，我们可以通过预先写一个空类，在热更新的时候再将我们需要热更新的部分功能实现

code:

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;
//空类，用于补充没有实现的功能
[Hotfix]
public class HotFixEmpty : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
    private void OnTriggerEnter(Collider other)
    {
        
    }
    private void BehaviourMethod()
    {
        
        
    }
}

```

第三个脚本：也就是我们需要修改的脚本

code:

```c#
using UnityEngine;
using System.Collections;
using XLua;
[Hotfix]
public class FishMaker : MonoBehaviour
{
    public Transform fishHolder;
    public Transform[] genPositions;
    public GameObject[] fishPrefabs;
    public HotFixScript hotFixScript;
    public float fishGenWaitTime = 0.5f;
    public float waveGenWaitTime = 0.3f;

    void Start()
    {
        //发布之后就不能再c#里面修改，只能修改一些应急问题
        InvokeRepeating("MakeFishes", 0, waveGenWaitTime);
        //AssetBundle ab = AssetBundle.LoadFromFile("文件路径");
        //GameObject gameObject = ab.LoadAsset<GameObject>("资源名称");
    }
    [LuaCallCSharp]
    void MakeFishes()
    {
        int genPosIndex = Random.Range(0, genPositions.Length);
        int fishPreIndex = Random.Range(0, fishPrefabs.Length);
        int maxNum = fishPrefabs[fishPreIndex].GetComponent<FishAttr>().maxNum;
        int maxSpeed = fishPrefabs[fishPreIndex].GetComponent<FishAttr>().maxSpeed;
        int num = Random.Range((maxNum / 2) + 1, maxNum);
        int speed = Random.Range(maxSpeed / 2, maxSpeed);
        //int moveType = Random.Range(0, 2);      //0：直走；1：转弯
        int moveType = 0;      //0：直走；1：转弯
        int angOffset;                          //仅直走生效，直走的倾斜角
        int angSpeed;                           //仅转弯生效，转弯的角速度
        if (moveType == 0)
        {
            angOffset = Random.Range(-22, 22);
            StartCoroutine(GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset));
        }
        else
        {
            if (Random.Range(0, 2) == 0)        //是否取负的角速度
            {
                angSpeed = Random.Range(-15, -9);
            }
            else
            {
                angSpeed = Random.Range(9, 15);
            }
            StartCoroutine(GenTrunFish(genPosIndex, fishPreIndex, num, speed, angSpeed));
        }
    }

    IEnumerator GenStraightFish(int genPosIndex,int fishPreIndex,int num,int speed,int angOffset)
    {

        for (int i = 0; i < num; i++)
        {
            GameObject fish = Instantiate(fishPrefabs[fishPreIndex]);
            fish.transform.SetParent(fishHolder, false);
            fish.transform.localPosition = genPositions[genPosIndex].localPosition;
            fish.transform.localRotation = genPositions[genPosIndex].localRotation;
            fish.transform.Rotate(0, 0, angOffset);
            fish.GetComponent<SpriteRenderer>().sortingOrder += i;
            fish.AddComponent<Ef_AutoMove>().speed = speed;
            yield return new WaitForSeconds(fishGenWaitTime);
        }
    }

    IEnumerator GenTrunFish(int genPosIndex, int fishPreIndex, int num, int speed, int angSpeed)
    {
 

        for (int i = 0; i < num; i++)
        {
            GameObject fish = Instantiate(fishPrefabs[fishPreIndex]);
            fish.transform.SetParent(fishHolder, false);
            fish.transform.localPosition = genPositions[genPosIndex].localPosition;
            fish.transform.localRotation = genPositions[genPosIndex].localRotation;
            fish.GetComponent<SpriteRenderer>().sortingOrder += i;
            fish.AddComponent<Ef_AutoMove>().speed = speed;
            fish.AddComponent<Ef_AutoRotate>().speed = angSpeed;
            yield return new WaitForSeconds(fishGenWaitTime);
        }
    }
}
```

第四个脚本是lua脚本，我们实现热更新的部分就写在这里

这个脚本的作用就是将小鱼的轨道修改成随机轨道

```lua
print('123')
local UnityEngine = CS.UnityEngine
xlua.hotfix(CS.FishMaker,"MakeFishes",function(self)
	local genPosIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
	local fishPreIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
    local maxNum
    local angSpeed
    local maxSpeed
    local fishMaker = CS.FishMaker
    local newfish = CS.HotFixScript.GetGameObject('bigeyefish')
    if newfish ~= nil then
        self.fishPrefabs:SetValue(newfish,1);
    end
    
    maxNum = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxNum
    maxSpeed = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxSpeed
    local num = UnityEngine.Random.Range((maxNum/2)+1,maxNum)
    local speed = UnityEngine.Random.Range(maxSpeed/2,maxSpeed)
    speed = math.floor(speed)
    local moveType = math.floor(UnityEngine.Random.Range(0,2))
    num = math.floor(num)
    if moveType == 0 then
        local angOffset = math.floor(UnityEngine.Random.Range(-22,22))
        --print(genPosIndex..fishPreIndex..num..speed..angOffset)
        print("dfdf1"..angOffset)
        self:StartCoroutine(self:GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset))
    else
        if UnityEngine.Random.Range(0,2) == 0 then
            angSpeed = math.floor(UnityEngine.Random.Range(-15,-9))
        else
            angSpeed = math.floor(UnityEngine.Random.Range(9,15))
        end
        --print(genPosIndex..fishPreIndex..num..speed..angSpeed)
        print("dfdf2"..angSpeed)
        self:StartCoroutine(self:GenTrunFish(genPosIndex, fishPreIndex, num, speed,angSpeed ))
    end
end
)

xlua.hotfix(CS.FishMaker,"Start",function(self)
    self:InvokeRepeating("MakeFishes", 0, self.waveGenWaitTime)
    print("-----------------------------------------------------")
    --CS.HotFixScript.LoadResource("cy","gameobject\\enemy.ab")
    --local newfish = self.hotFixScript:GetGameObject('cy')4
    self.hotFixScript:LoadResource('bigeyefish',"gameobject\\enemy.ab")
end
) 
xlua.private_accessible(CS.HotFixEmpty)
xlua.hotfix(CS.HotFixEmpty,"Start",function(self)
        print("this is HotFixEmpty")
        self:BehaviourMethod()
end
)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",function(self,other) 
    
end
)

xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",function(self)
    print("this is BehaviourMethod")
end
)
```

最后一个脚本用于实现释放已经null的功能

```lua
hotfix(CS.FishMaker,'MakeFishes',nil)
xlua.hotfix(CS.FishMaker,"Start",nil)
xlua.hotfix(CS.HotFixEmpty,"Start",nil)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",nil)
xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",nil)
luaEnv.Dispose();
```

### 资源热补丁：用于更新资源

第一步创建Editor文件夹，用于扩展编辑器菜单

![image-20240521145552033](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521145552033.png)

创建扩展脚本

```c#
using Codice.CM.Common.Serialization;
using PlasticPipe.PlasticProtocol.Messages;
using UnityEditor;
using System.IO;
public class CreateAssetBundles
{
    [MenuItem("Assets/Build AssetBundles")]//指定添加功能在那个层级下面
    static void BuildAllAssetBundles()
    {
        string dir = "AssetBundles";
        if (Directory.Exists(dir) == false) {
            Directory.CreateDirectory(dir);
        }
        //路径，压缩格式，发布的目标平台,用于打ab包，将只打了标签的资源进行打包
        BuildPipeline.BuildAssetBundles(dir,BuildAssetBundleOptions.None,BuildTarget.StandaloneWindows64);
    }
}
```

编译成功之后，就可以在asset下找到

​	![image-20240521150533128](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521150533128.png)

在打包的时候需要注意一点,在进行打包的时候，预制体上面的动画控制器等资源也会一起打包，但是要注意打包的目标位置是否有之前挂载的脚本，如果没有就会导致脚本丢失

![image-20240521151300655](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521151300655.png)

然后为需要加上的资源打上标签，在打标签之前，需要先创建好标签，标签名称自定义

![image-20240521151721680](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521151721680.png)

创建后缀名称

![image-20240521151843006](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521151843006.png)

![image-20240521170944292](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521170944292.png)

然后再assert中执行创建ab包，就可以导出ab包

![image-20240521152401732](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521152401732.png)

编写加载资源的脚本，最好单独存储。

```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.CompilerServices;
using UnityEngine;
using UnityEngine.Networking;
using XLua;

public class HotFixScript : MonoBehaviour
{
    private LuaEnv luaEnv;
    //用于存储预制体
    public static Dictionary<string,GameObject> prefabDict = new Dictionary<string,GameObject>();
    // Start is called before the first frame update
    private void Awake()
    {
        //要将打补丁的放在所有的游戏逻辑之前 
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyLoader);
        luaEnv.DoString("require 'fish'");
    }
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
    }
    private byte[] MyLoader(ref string filePath)
    {
        //@让\被正确识别为文件路径
        string absPath = @"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\" + filePath + ".lua.txt";
        return System.Text.Encoding.UTF8.GetBytes( File.ReadAllText(absPath));
    }
    private void OnDisable()
    {
        luaEnv.DoString("require 'fishDispose'");
    }
    private void OnDestroy()
    {
        luaEnv.Dispose();
    }
    [LuaCallCSharp]
    //最好写在一个单独的管理脚本上面
    public void LoadResource(string resName,string filePath)
    {
        //本地加载方式public static void LoadResource(string resName,string filePath)
        AssetBundle ab = AssetBundle.LoadFromFile(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\FishMaster\AssetBundles\"+filePath);
        GameObject gameObject = ab.LoadAsset<GameObject>(resName);
        prefabDict.Add(resName, gameObject);//存储方便取用
    }
    
    IEnumerator LoadResourceCorotine(string resName, string filePath)//http://localhost:8080/
    {
        UnityWebRequest request = UnityWebRequestAssetBundle.GetAssetBundle(@"http://localhost:8080/AssetBundles/" + filePath);
        yield return request.SendWebRequest();//发送请求，连接网络，下载
        AssetBundle ab = (request.downloadHandler as DownloadHandlerAssetBundle).assetBundle;//转换成ab变量
        GameObject fish = ab.LoadAsset<GameObject>(resName);
        
        Debug.Log("加入"+resName);
        Debug.Log(fish);
        prefabDict.Add(resName, fish);//存储方便取用
        Debug.Log("实际获取的物体" + prefabDict[resName]+"数量"+prefabDict.Count);
    }

    /// <summary>
    /// 获取资源
    /// </summary>
    /// <param name="goName"></param>
    /// <returns></returns>
    [LuaCallCSharp]
    public static GameObject GetGameObject(string goName)
    {
        Debug.Log("获取----"+goName);
        Debug.Log("这里是没有获取吗" +"数量" +prefabDict.Count);
        if (prefabDict.Count > 0)
        return prefabDict[goName];
        else return null;
    }
}

```

在lua中编写lua代码对ab资源进行调用,采用本地调用方式

```lua
print('123')
--修改生成yu
local UnityEngine = CS.UnityEngine
xlua.hotfix(CS.FishMaker,"MakeFishes",function(self)
	local genPosIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
	local fishPreIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
    local maxNum
    local angSpeed
    local maxSpeed
    local fishMaker = CS.FishMaker
    local newfish = CS.HotFixScript.GetGameObject('bigeyefish')
    if newfish ~= nil then
        self.fishPrefabs:SetValue(newfish,1);
    end
    
    maxNum = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxNum
    maxSpeed = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxSpeed
    local num = UnityEngine.Random.Range((maxNum/2)+1,maxNum)
    local speed = UnityEngine.Random.Range(maxSpeed/2,maxSpeed)
    speed = math.floor(speed)
    local moveType = math.floor(UnityEngine.Random.Range(0,2))
    num = math.floor(num)
    if moveType == 0 then
        local angOffset = math.floor(UnityEngine.Random.Range(-22,22))
        --print(genPosIndex..fishPreIndex..num..speed..angOffset)
        print("dfdf1"..angOffset)
        self:StartCoroutine(self:GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset))
    else
        if UnityEngine.Random.Range(0,2) == 0 then
            angSpeed = math.floor(UnityEngine.Random.Range(-15,-9))
        else
            angSpeed = math.floor(UnityEngine.Random.Range(9,15))
        end
        --print(genPosIndex..fishPreIndex..num..speed..angSpeed)
        print("dfdf2"..angSpeed)
        self:StartCoroutine(self:GenTrunFish(genPosIndex, fishPreIndex, num, speed,angSpeed ))
    end
end
)
--调用资源---------------------------------------------
xlua.hotfix(CS.FishMaker,"Start",function(self)
    self:InvokeRepeating("MakeFishes", 0, self.waveGenWaitTime)
    CS.HotFixScript.LoadResource("cy","gameobject\\enemy.ab")--本地加载
    local newfish = self.hotFixScript:GetGameObject('cy')
end
) 
------从本地加载资源
xlua.private_accessible(CS.HotFixEmpty)--访问其中的私有方法
xlua.hotfix(CS.HotFixEmpty,"Start",function(self)
        print("this is HotFixEmpty")
        self:BehaviourMethod()
end
)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",function(self,other) 
    
end
)

xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",function(self)
    print("this is BehaviourMethod")
end
)
```

对于那些无法预知需要添加的功能，在游戏已经上线之后该如何处理呢？
可以定义几个空类。后期如果有需要添加的功能只要对空类进行修补就可以了

定义空类：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;
//空类，用于补充没有实现的功能
[Hotfix]
public class HotFixEmpty : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
    private void OnTriggerEnter(Collider other)
    {
        
    }
    private void BehaviourMethod()
    {
        
        
    }
}
```

在lua中进行调用

```lua
xlua.private_accessible(CS.HotFixEmpty)--访问其中的私有方法
xlua.hotfix(CS.HotFixEmpty,"Start",function(self)
        print("this is HotFixEmpty")
        self:BehaviourMethod()
end
)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",function(self,other) 
    
end
)

xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",function(self)
    print("this is BehaviourMethod")
end
)
```

### 实现从服务器上加载资源

首先搭建本地服务器：本文使用netbox进行搭建，搭建教程来自

看了好多都没找到完整的傻瓜式教学，我就结合自己的尝试过程分享一下我的傻瓜式教程

安装NetBox
下载地址：[Netbox](https://www.netbox.cn/)

下载完之后解压，双击进行安装，选择安装路径，其他一路next就好

搭建本地资源虚拟服务器
选一个你喜欢的位置新建一个文件夹，例如web，这个web就将作为你的虚拟服务器。接着在web的同级目录中新建一个main.txt文件，将其后缀改为box

接着在sample文件夹中，有个WebServer样例，用文本编辑器打开其中的main.box文件将其内容复制至你刚刚新建的main.box中

```
Dim httpd
Shell.Service.RunService "NBWeb", "NetBox Web Server", "NetBox Http Server Sample"
'---------------------- Service Event ---------------------
Sub OnServiceStart()
      Set httpd = NetBox.CreateObject("NetBox.HttpServer")
      If httpd.Create("", 80) = 0 Then
            Set host = httpd.AddHost("", "/web")
            host.EnableScript = true
            host.AddDefault "index.html"
            httpd.Start
     else
           Shell.Quit 0
           end if
     End Sub

Sub OnServiceStop()
      httpd.Close
End Sub

Sub OnServicePause()
     httpd.Stop
End Sub

Sub OnServiceResume()
      httpd.Start
End Sub
```

这段代码中只需要改两个地方。一个httpd.Create("", 80)处的80为端口号，如果已被使用可以改为81，8080等；另一个就是httpd.AddHost("", "/web")处，斜杠（貌似左斜右斜没有关系）后填写刚刚新建的文件夹名，也就是虚拟服务器的名字

测试
在web文件夹中新建一个index.html，可以在里面编辑内容。然后双击main.box启动服务器，或者右键直接build这个文件生成一个.exe的可执行文件，**接着在浏览器的网址中输入127.0.0.1或者localhost:+端口号，比如我这里就是localhost:80，就可以看到index.html中编辑的内容了，这也说明链接成功了**

接着就可以把热更资源放在web文件夹中进行测试，但注意index.html不能删除，我猜测是需要一个默认资源，说错了的话还望大佬们指教，感谢！

最后再叠一遍甲，我不清楚这些其中的原理，只是傻瓜式的完成了链接，说错的地方还望各位多多包涵，多多指教

原文链接：https://blog.csdn.net/qq_43645024/article/details/128999117

然后新建一个html文件

![image-20240521172129844](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521172129844.png)

然后找到main.box,以记事本的方式打开，并且修改其中的参数

```
Dim httpd
Shell.Service.RunService "NBWeb", "NetBox Web Server", "NetBox Http Server Sample"
'---------------------- Service Event ---------------------
Sub OnServiceStart()
      Set httpd = NetBox.CreateObject("NetBox.HttpServer")
      If httpd.Create("", 8080) = 0 Then
            Set host = httpd.AddHost("", "/NetEnv")
            host.EnableScript = true
            host.AddDefault "index.html"
            httpd.Start
     else
           Shell.Quit 0
           end if
     End Sub
 
Sub OnServiceStop()
      httpd.Close
End Sub
 
Sub OnServicePause()
     httpd.Stop
End Sub
 
Sub OnServiceResume()
      httpd.Start
End Sub
```

如果通过localhost：80可以访问表示服务器搭建成功



将所有的资源放到该文件夹下

![image-20240521172448670](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521172448670.png)

![image-20240521172501854](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521172501854.png)

### 从网络上加载资源

编写从网路上加载资源的脚本

```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.CompilerServices;
using UnityEngine;
using UnityEngine.Networking;
using XLua;

public class HotFixScript : MonoBehaviour
{
    private LuaEnv luaEnv;
    //用于存储预制体
    public static Dictionary<string,GameObject> prefabDict = new Dictionary<string,GameObject>();
    // Start is called before the first frame update
    private void Awake()
    {
        //要将打补丁的放在所有的游戏逻辑之前 
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyLoader);
        luaEnv.DoString("require 'fish'");
    }
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
    }
    private byte[] MyLoader(ref string filePath)
    {
        //@让\被正确识别为文件路径
        string absPath = @"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\" + filePath + ".lua.txt";
        return System.Text.Encoding.UTF8.GetBytes( File.ReadAllText(absPath));
    }
    private void OnDisable()
    {
        luaEnv.DoString("require 'fishDispose'");
    }
    private void OnDestroy()
    {
        luaEnv.Dispose();
    }
    [LuaCallCSharp]
    //最好写在一个单独的管理脚本上面
    public void LoadResource(string resName,string filePath)
    {
        Debug.Log("正在加载资源"+resName);
        StartCoroutine(LoadResourceCorotine( resName,  filePath));
    }
    //携程要在一个普通的方法里开启，如果直接定义的话，就得在start等函数中调用，在静态方法中是无法开启一个携程的，因为静态方法中只能调用静态方法，那么解决方法就是将静态方法修改成普通方法
    IEnumerator LoadResourceCorotine(string resName, string filePath)//http://localhost:8080/
    {
        //获取ab资源，参数是文件路径
        UnityWebRequest request = UnityWebRequestAssetBundle.GetAssetBundle(@"http://localhost:8080/AssetBundles/" + filePath);
        yield return request.SendWebRequest();//发送请求，连接网络，下载
        //downloadHandler表示处理器，要获取对应的资源，需要进一步转化
        AssetBundle ab = (request.downloadHandler as DownloadHandlerAssetBundle).assetBundle;//转换成ab变量
        GameObject fish = ab.LoadAsset<GameObject>(resName);
        prefabDict.Add(resName, fish);//存储方便取用
    }

    /// <summary>
    /// 获取资源
    /// </summary>
    /// <param name="goName"></param>
    /// <returns></returns>
    [LuaCallCSharp]
    public static GameObject GetGameObject(string goName)
    {
        Debug.Log("获取----"+goName);
        Debug.Log("这里是没有获取吗" +"数量" +prefabDict.Count);
        if (prefabDict.Count > 0)
        return prefabDict[goName];
        else return null;
    }
}
```

那么在lua脚本中调用的方式也要发生相应的改变（标蓝色的代码），否则引用不到

但是在调用前要先在对应的c#加上![image-20240521210725809](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521210725809.png)

```lua
print('123')
--修改生成yu
local UnityEngine = CS.UnityEngine
xlua.hotfix(CS.FishMaker,"MakeFishes",function(self)
	local genPosIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
	local fishPreIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
    local maxNum
    local angSpeed
    local maxSpeed
    local fishMaker = CS.FishMaker
    local newfish = CS.HotFixScript.GetGameObject('bigeyefish')
    if newfish ~= nil then
        self.fishPrefabs:SetValue(newfish,1);
    end
    
    maxNum = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxNum
    maxSpeed = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxSpeed
    local num = UnityEngine.Random.Range((maxNum/2)+1,maxNum)
    local speed = UnityEngine.Random.Range(maxSpeed/2,maxSpeed)
    speed = math.floor(speed)
    local moveType = math.floor(UnityEngine.Random.Range(0,2))
    num = math.floor(num)
    if moveType == 0 then
        local angOffset = math.floor(UnityEngine.Random.Range(-22,22))
        --print(genPosIndex..fishPreIndex..num..speed..angOffset)
        print("dfdf1"..angOffset)
        self:StartCoroutine(self:GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset))
    else
        if UnityEngine.Random.Range(0,2) == 0 then
            angSpeed = math.floor(UnityEngine.Random.Range(-15,-9))
        else
            angSpeed = math.floor(UnityEngine.Random.Range(9,15))
        end
        --print(genPosIndex..fishPreIndex..num..speed..angSpeed)
        print("dfdf2"..angSpeed)
        self:StartCoroutine(self:GenTrunFish(genPosIndex, fishPreIndex, num, speed,angSpeed ))
    end
end
)

xlua.hotfix(CS.FishMaker,"Start",function(self)
    self:InvokeRepeating("MakeFishes", 0, self.waveGenWaitTime)
    print("-----------------------------------------------------")
    --CS.HotFixScript.LoadResource("cy","gameobject\\enemy.ab")--本地加载
    --local newfish = self.hotFixScript:GetGameObject('cy')
    self.hotFixScript:LoadResource('bigeyefish',"gameobject\\enemy.ab")--网络加载
end
) 
xlua.private_accessible(CS.HotFixEmpty)--访问其中的私有方法
xlua.hotfix(CS.HotFixEmpty,"Start",function(self)
        print("this is HotFixEmpty")
        self:BehaviourMethod()
end
)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",function(self,other) 
    
end
)

xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",function(self)
    print("this is BehaviourMethod")
end
)
```

从网络上面加载lua脚本（因为脚本需要经常更新的，所以最好从网络上加载下来，然后写入本地文本中）

编写的位置可以放在开始界面，要先加载脚本，然后再加载资源，否则逻辑跑不通

脚本:实现从网络上加载资源，然后写入到本地

```c#
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.Networking;
using System.IO;
public class StartSceneUI : MonoBehaviour
{
    public void NewGame()
    {
        PlayerPrefs.DeleteKey("gold");
        PlayerPrefs.DeleteKey("lv");
        PlayerPrefs.DeleteKey("exp");
        PlayerPrefs.DeleteKey("scd");
        PlayerPrefs.DeleteKey("bcd");
        SceneManager.LoadScene(1); 
    }

    private void Awake()
    {
        //开启携程
        StartCoroutine(LoadResourceCoroutine());
    }
    public void OldGame()
    {
        
        SceneManager.LoadScene(1);
    }

    public void OnCloseButton()
    {
        Application.Quit();
    }
    //下面的是核心代码
    IEnumerator LoadResourceCoroutine()
    {
        UnityWebRequest request = UnityWebRequest.Get(@"http://localhost:8080/fish.lua.txt");
        yield return request.SendWebRequest();//发送请求，连接网络，下载
        string str = request.downloadHandler.text;
        //写入本地
        File.WriteAllText(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\fish.lua.txt",str);

        UnityWebRequest request1 = UnityWebRequest.Get(@"http://localhost:8080/fishDispose.lua.txt");
        yield return request1.SendWebRequest();//发送请求，连接网络，下载
        string str1 = request1.downloadHandler.text;
 File.WriteAllText(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\fishDispose.lua.txt",str1);
    }
}

```

对于打标签的一些操作（扩展，对于需要打标签比较多的情况）

![image-20240521213916261](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240521213916261.png)

## 代码汇总

hotfixscripts

```c#
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.CompilerServices;
using UnityEngine;
using UnityEngine.Networking;
using XLua;

public class HotFixScript : MonoBehaviour
{
    private LuaEnv luaEnv;
    //用于存储预制体
    public static Dictionary<string,GameObject> prefabDict = new Dictionary<string,GameObject>();
    // Start is called before the first frame update
    private void Awake()
    {
        //要将打补丁的放在所有的游戏逻辑之前 
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyLoader);
        luaEnv.DoString("require 'fish'");
    }
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
    }
    private byte[] MyLoader(ref string filePath)
    {
        //@让\被正确识别为文件路径
        string absPath = @"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\" + filePath + ".lua.txt";
        return System.Text.Encoding.UTF8.GetBytes( File.ReadAllText(absPath));
    }
    private void OnDisable()
    {
        luaEnv.DoString("require 'fishDispose'");
    }
    private void OnDestroy()
    {
        luaEnv.Dispose();
    }
    [LuaCallCSharp]
    //最好写在一个单独的管理脚本上面
    public void LoadResource(string resName,string filePath)
    {
        Debug.Log("正在加载资源"+resName);
        StartCoroutine(LoadResourceCorotine( resName,  filePath));
        //本地加载方式public static void LoadResource(string resName,string filePath)
        //AssetBundle ab = AssetBundle.LoadFromFile(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\FishMaster\AssetBundles\"+filePath);
        //GameObject gameObject = ab.LoadAsset<GameObject>(resName);
        //prefabDict.Add(resName, gameObject);//存储方便取用
    }
    //携程要在一个普通的方法里开启，如果直接定义的话，就得在start等函数中调用，在静态方法中是无法开启一个携程的，因为静态方法中只能调用静态方法，那么解决方法就是将静态方法修改成普通方法
    IEnumerator LoadResourceCorotine(string resName, string filePath)//http://localhost:8080/
    {
        //获取ab资源，参数是文件路径
        UnityWebRequest request = UnityWebRequestAssetBundle.GetAssetBundle(@"http://localhost:8080/AssetBundles/" + filePath);
        yield return request.SendWebRequest();//发送请求，连接网络，下载
        //downloadHandler表示处理器，要获取对应的资源，需要进一步转化
        AssetBundle ab = (request.downloadHandler as DownloadHandlerAssetBundle).assetBundle;//转换成ab变量
        GameObject fish = ab.LoadAsset<GameObject>(resName);
        prefabDict.Add(resName, fish);//存储方便取用
    }

    /// <summary>
    /// 获取资源
    /// </summary>
    /// <param name="goName"></param>
    /// <returns></returns>
    [LuaCallCSharp]
    public static GameObject GetGameObject(string goName)
    {
        Debug.Log("获取----"+goName);
        Debug.Log("这里是没有获取吗" +"数量" +prefabDict.Count);
        if (prefabDict.Count > 0)
        return prefabDict[goName];
        else return null;
    }
}

```

hotfixempty

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;
//空类，用于补充没有实现的功能
[Hotfix]
public class HotFixEmpty : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
    private void OnTriggerEnter(Collider other)
    {
        
    }
    private void BehaviourMethod()
    {
        
        
    }
}

```

fishmakeer

```c#
using UnityEngine;
using System.Collections;
using XLua;
[Hotfix]
public class FishMaker : MonoBehaviour
{
    public Transform fishHolder;
    public Transform[] genPositions;
    public GameObject[] fishPrefabs;
    public HotFixScript hotFixScript;
    public float fishGenWaitTime = 0.5f;
    public float waveGenWaitTime = 0.3f;

    void Start()
    {
        //发布之后就不能再c#里面修改，只能修改一些应急问题
        InvokeRepeating("MakeFishes", 0, waveGenWaitTime);
        //AssetBundle ab = AssetBundle.LoadFromFile("文件路径");
        //GameObject gameObject = ab.LoadAsset<GameObject>("资源名称");
    }
    [LuaCallCSharp]
    void MakeFishes()
    {
        int genPosIndex = Random.Range(0, genPositions.Length);
        int fishPreIndex = Random.Range(0, fishPrefabs.Length);
        int maxNum = fishPrefabs[fishPreIndex].GetComponent<FishAttr>().maxNum;
        int maxSpeed = fishPrefabs[fishPreIndex].GetComponent<FishAttr>().maxSpeed;
        int num = Random.Range((maxNum / 2) + 1, maxNum);
        int speed = Random.Range(maxSpeed / 2, maxSpeed);
        //int moveType = Random.Range(0, 2);      //0：直走；1：转弯
        int moveType = 0;      //0：直走；1：转弯
        int angOffset;                          //仅直走生效，直走的倾斜角
        int angSpeed;                           //仅转弯生效，转弯的角速度
        if (moveType == 0)
        {
            angOffset = Random.Range(-22, 22);
            StartCoroutine(GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset));
        }
        else
        {
            if (Random.Range(0, 2) == 0)        //是否取负的角速度
            {
                angSpeed = Random.Range(-15, -9);
            }
            else
            {
                angSpeed = Random.Range(9, 15);
            }
            StartCoroutine(GenTrunFish(genPosIndex, fishPreIndex, num, speed, angSpeed));
        }
    }

    IEnumerator GenStraightFish(int genPosIndex,int fishPreIndex,int num,int speed,int angOffset)
    {

        for (int i = 0; i < num; i++)
        {
            GameObject fish = Instantiate(fishPrefabs[fishPreIndex]);
            fish.transform.SetParent(fishHolder, false);
            fish.transform.localPosition = genPositions[genPosIndex].localPosition;
            fish.transform.localRotation = genPositions[genPosIndex].localRotation;
            fish.transform.Rotate(0, 0, angOffset);
            fish.GetComponent<SpriteRenderer>().sortingOrder += i;
            fish.AddComponent<Ef_AutoMove>().speed = speed;
            yield return new WaitForSeconds(fishGenWaitTime);
        }
    }

    IEnumerator GenTrunFish(int genPosIndex, int fishPreIndex, int num, int speed, int angSpeed)
    {
 

        for (int i = 0; i < num; i++)
        {
            GameObject fish = Instantiate(fishPrefabs[fishPreIndex]);
            fish.transform.SetParent(fishHolder, false);
            fish.transform.localPosition = genPositions[genPosIndex].localPosition;
            fish.transform.localRotation = genPositions[genPosIndex].localRotation;
            fish.GetComponent<SpriteRenderer>().sortingOrder += i;
            fish.AddComponent<Ef_AutoMove>().speed = speed;
            fish.AddComponent<Ef_AutoRotate>().speed = angSpeed;
            yield return new WaitForSeconds(fishGenWaitTime);
        }
    }
}

```

start scene ui

```c#
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.Networking;
using System.IO;
public class StartSceneUI : MonoBehaviour
{
    public void NewGame()
    {
        PlayerPrefs.DeleteKey("gold");
        PlayerPrefs.DeleteKey("lv");
        PlayerPrefs.DeleteKey("exp");
        PlayerPrefs.DeleteKey("scd");
        PlayerPrefs.DeleteKey("bcd");
        SceneManager.LoadScene(1); 
    }

    private void Awake()
    {
        //开启携程
        StartCoroutine(LoadResourceCoroutine());
    }
    public void OldGame()
    {
        
        SceneManager.LoadScene(1);
    }

    public void OnCloseButton()
    {
        Application.Quit();
    }
    //下面的是核心代码
    IEnumerator LoadResourceCoroutine()
    {
        UnityWebRequest request = UnityWebRequest.Get(@"http://localhost:8080/fish.lua.txt");
        yield return request.SendWebRequest();//发送请求，连接网络，下载
        string str = request.downloadHandler.text;
        //写入本地
        File.WriteAllText(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\fish.lua.txt",str);

        UnityWebRequest request1 = UnityWebRequest.Get(@"http://localhost:8080/fishDispose.lua.txt");
        yield return request1.SendWebRequest();//发送请求，连接网络，下载
        string str1 = request1.downloadHandler.text;
        File.WriteAllText(@"C:\Users\86187\Desktop\FileSum\UnityClassResources\HOT\fishDispose.lua.txt",str1);
    }
}

```

fish.lua.txt

```lua
print('123')
--修改生成yu
local UnityEngine = CS.UnityEngine
xlua.hotfix(CS.FishMaker,"MakeFishes",function(self)
	local genPosIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
	local fishPreIndex = math.floor(UnityEngine.Random.Range(0,self.genPositions.Length))
    local maxNum
    local angSpeed
    local maxSpeed
    local fishMaker = CS.FishMaker
    local newfish = CS.HotFixScript.GetGameObject('bigeyefish')
    if newfish ~= nil then
        self.fishPrefabs:SetValue(newfish,1);
    end
    
    maxNum = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxNum
    maxSpeed = self.fishPrefabs[fishPreIndex]:GetComponent("FishAttr").maxSpeed
    local num = UnityEngine.Random.Range((maxNum/2)+1,maxNum)
    local speed = UnityEngine.Random.Range(maxSpeed/2,maxSpeed)
    speed = math.floor(speed)
    local moveType = math.floor(UnityEngine.Random.Range(0,2))
    num = math.floor(num)
    if moveType == 0 then
        local angOffset = math.floor(UnityEngine.Random.Range(-22,22))
        --print(genPosIndex..fishPreIndex..num..speed..angOffset)
        print("dfdf1"..angOffset)
        self:StartCoroutine(self:GenStraightFish(genPosIndex, fishPreIndex, num, speed, angOffset))
    else
        if UnityEngine.Random.Range(0,2) == 0 then
            angSpeed = math.floor(UnityEngine.Random.Range(-15,-9))
        else
            angSpeed = math.floor(UnityEngine.Random.Range(9,15))
        end
        --print(genPosIndex..fishPreIndex..num..speed..angSpeed)
        print("dfdf2"..angSpeed)
        self:StartCoroutine(self:GenTrunFish(genPosIndex, fishPreIndex, num, speed,angSpeed ))
    end
end
)

xlua.hotfix(CS.FishMaker,"Start",function(self)
    self:InvokeRepeating("MakeFishes", 0, self.waveGenWaitTime)
    print("-----------------------------------------------------")
    --CS.HotFixScript.LoadResource("cy","gameobject\\enemy.ab")
    --local newfish = self.hotFixScript:GetGameObject('cy')4
    self.hotFixScript:LoadResource('bigeyefish',"gameobject\\enemy.ab")--网络加载
end
) 
xlua.private_accessible(CS.HotFixEmpty)--访问其中的私有方法
xlua.hotfix(CS.HotFixEmpty,"Start",function(self)
        print("this is HotFixEmpty")
        self:BehaviourMethod()
end
)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",function(self,other) 
    
end
)

xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",function(self)
    print("this is BehaviourMethod")
end
)
```

fishdispose.lua.txt

```lua
xlua.hotfix(CS.FishMaker,'MakeFishes',nil)
xlua.hotfix(CS.FishMaker,"Start",nil)
xlua.hotfix(CS.HotFixEmpty,"Start",nil)
xlua.hotfix(CS.HotFixEmpty,"OnTriggerEnter",nil)
xlua.hotfix(CS.HotFixEmpty,"BehaviourMethod",nil)
luaEnv.Dispose();

```

