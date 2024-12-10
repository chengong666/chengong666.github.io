---
title: unity插件合集
date: 2024-12-10 22:08:02
tags:
---
# AmplifyShaderEditor

## 插件介绍

![image-20241019114846786](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019114846786.png)

![image-20241019115055673](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019115055673.png)

通过图形化的界面制作shader，实现一些效果，可以结合代码进行优化

界面介绍

![image-20241019115151437](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019115151437.png)



<img src="D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019115233142.png" alt="image-20241019115233142"  />



![image-20241019115614272](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019115614272.png)

![image-20241019115723512](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019115723512.png)

![image-20241019115848919](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019115848919.png)

![image-20241019115904131](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019115904131.png)



![image-20241019120026007](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019120026007.png)



![image-20241019120127263](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019120127263.png)



![image-20241019120207774](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019120207774.png)



![                  ](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019120220029.png)

**提醒，对于节点的命名最好不要用中文**

### 函数介绍                                         

#### Append 节点

Append 节点*（快捷键：V 键）*输出一个**向量**，该向量是来自不同输入源的通道组合的结果。

![image-20241019170420199](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019170420199.png)

Input ports amount （**输入端口量**） 取决于所选的 [Output Type （输出类型](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Append#paramOutputType)） 和 incoming data type（**传入数据类型**）。 P.e. 如果 *Output Type （输出类型*） 设置为 *Vector3 （矢量 3*)，则默认情况下，它将显示三个输入端口 *X*、*Y*、*Z*。当将 *Vector2* 数据类型连接到输入端口 *X* 时，它会自动将其名称更改为 *XY*，下一个端口也将重命名为 *Z* 并隐藏第三个不必要的端口。

**注意：**仅当下一个通道未被占用时，才会自动使用端口。例如，如果将 Vector3 类型的数据连接到 Append 节点的 X 输入端口，而该端口的 Z 输入端口上已经创建了连接，则只会占用 X 和 Y 通道。**(人话：也就是如果你其中的z已经被占用，这个时候即使你传入一个V3的值，那么他也不会影响到z)**

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\AppendNode.jpg)
使用的节点：[Texture Object](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Object)、[Vertex Position](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Vertex_Position)、Append、[Texture Sample](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)、[Vertex Normal](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Vertex_Normal)、[Lerp](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp)

| 节点参数 |                             描述                             | 默认值 |
| :------: | :----------------------------------------------------------: | :----: |
| 输出类型 | 定义生成的向量数据类型。此选项对注入新数据的可用输入量有直接影响。可用的输出类型包括：**Vector2：**2 通道数据类型**Vector3：**3 通道数据类型**Vector4：**4 通道数据类型**颜色：**4 通道数据类型 |   0    |

|   输入端口   |                             描述                             |                             类型                             |
| :----------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| X、Y、Z 或 W | 要追加到新 *Vector* 输出中的数据。输入端口的数量和名称都取决于当前的 *Output Type* 和 incoming 数据类型。 | 矢量4[[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Append#anchor) |



#### Multiply 节点

是矩阵还是浮点，这个我有点疑惑

乘法节点*（快捷键：M 键 ）*对两个或多个值（最多 10 个）或 **（ [A](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply#parama) \* [B](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply#parama) \* [...](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply#paramn) ）** **进行算术乘法运算。**默认情况下，该节点仅显示两个输入端口，因此，如果需要添加两个以上的值，则每当选取输出端口连线时，都会动态添加新端口。如果未连接这些额外的 inputs，它们也会被动态删除。此节点还处理矩阵乘法。在这种情况下，节点不再允许额外的端口来防止错误的矩阵操作。

多通道数据类型之间的乘法是按通道完成的。如果 A 和 B 具有不同的通道数量，则执行强制转换以匹配具有最多通道的通道。

**注意：**乘法*是*可交换的，所以 value 的顺序*并不*重要，但是，矩阵乘法*不是*可交换的，顺序*很重要*。在进行矩阵乘法时请记住这一点。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\MultiplyNode.jpg)
使用的节点：[Texture Sample](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)、Multiply

| 节点参数 |                    描述                    | 默认值 |
| :------: | :----------------------------------------: | :----: |
|   一个   | 第一个值。仅当未连接相应的输入端口时可见。 |   0    |
|    B     | 第二个值。仅当未连接相应的输入端口时可见。 |   0    |



| 输入端口 |                           描述                           |                             类型                             |
| :------: | :------------------------------------------------------: | :----------------------------------------------------------: |
|   一个   |                         第一个值                         | **浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply#anchor)** |
|    B     |                         第二个值                         | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply#anchor) |
|    #     | 第 #n 个值，最多 10 个。仅在已连接或选择输出接线时可见。 | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply#anchor) |

------

1. **[^](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply#ref1)**端口会自动适应除 [Sampler](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Object) 类型之外的所有连接类型。



#### 插值节点(Lerp)

Lerp 节点*（快捷键：L 键 ）*按 [Alpha](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp#paramAlpha) 计算两个值 [A](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp#paramA) 和 [B](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp#paramB) 之间的线性插值。换句话说，它将根据第三个称为 Alpha 的混合值在 A 和 B 之间生成一个新的混合值，又名插值器，使用表达式 **（ （ 1 - l） \* A + B \* I ）。**插值器值的范围介于 [0 - 1] 之间，如果 *I = 0*，则返回 *A*，如果 *I = 1*，则返回 *B*。

**注意：**多通道数据类型之间的线性插值是按通道完成的。如果 *A* 和 *B* 具有不同的通道数量，则执行强制转换以匹配具有最多通道的通道。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\LerpNode.jpg)
使用的节点：[Texture Sample](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)、Lerp

| 节点参数 |                             描述                             | 默认值 |
| :------: | :----------------------------------------------------------: | :----: |
|   一个   | 插值器操作的第一个值，当 *Alpha = 0* 时完全输出。仅当未连接相应的输入端口时可见。 |   0    |
|    B     | 插值器操作的最后一个值，在 *Alpha = 1* 时完全输出。仅当未连接相应的输入端口时可见。 |   0    |
|  阿尔法  |      Interpolator 值。仅当未连接相应的输入端口时可见。       |   0    |



| 输入端口 |                        描述                         |                             类型                             |
| :------: | :-------------------------------------------------: | :----------------------------------------------------------: |
|   一个   |  插值器操作的第一个值。当 *Alpha = 0* 时完全输出。  | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp#anchor) |
|    B     | 插值器操作的最后一个值。当 *Alpha = 1* 时完全输出。 | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp#anchor) |
|  阿尔法  |    Interpolator 值。此端口被限制在范围 [0-1] 。     | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp#anchor) |

------

#### Add Node

加节点（快捷键：A键）用于对两个或更多值进行算术加法，最多可加到10个（即 A + B + ...）。默认情况下，节点只显示两个输入端口，因此如果需要加超过两个值，当你拖动输出端口的连线时，会动态添加一个新端口。如果这些额外的输入未连接，也会动态移除。对于多通道数据类型，按通道进行加法。如果 A 和 B 的通道数量不同，则会进行类型转换，以匹配通道最多的那个。注意：加法是交换律的，因此值的顺序并不重要。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\AddNode.jpg)

 

| 节点参数 |                    描述                    | 默认值 |
| :------: | :----------------------------------------: | :----: |
|   一个   | 第一个值。仅当未连接相应的输入端口时可见。 |   0    |
|    B     | 第二个值。仅当未连接相应的输入端口时可见。 |   0    |



| 输入端口 |                           描述                           |                             类型                             |
| :------: | :------------------------------------------------------: | :----------------------------------------------------------: |
|   一个   |                         第一个值                         | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Add#anchor) |
|    B     |                         第二个值                         | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Add#anchor) |
|    #     | 第 #n 个值，最多 10 个。仅在已连接或选择输出接线时可见。 | 浮点型 [[1](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Add#anchor) |

#### Subtract 节点

“减去”节点*（快捷键：S 键）*进行算术减法，其中 [A](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Subtract#parama) 减去 [B](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Subtract#paramb) 或 **（ [A](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Subtract#parama) - [B](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Subtract#paramb) ）。**

多通道数据类型之间的减法是按通道完成的。如果 A 和 B 具有不同的通道数量，则执行强制转换以匹配具有最多通道的通道。

**注意：**减法*不是*交换的，因此 value *的**顺序很重要**。*

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\SubtractNode.jpg)
使用的节点：[Texture Sample](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)、Subtract

| 节点参数 |                             描述                             | 默认值 |
| :------: | :----------------------------------------------------------: | :----: |
|   一个   | minuend （减号） 表示要从中减去的值。仅当未连接相应的输入端口时可见。 |   0    |
|    B     |  subtrahend，您要减去的值。仅当未连接相应的输入端口时可见。  |   0    |

| 输入端口 |                 描述                  |                             类型                             |
| :------: | :-----------------------------------: | :----------------------------------------------------------: |
|   一个   | minuend （减号） 表示要从中减去的值。 | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Subtract#anchor) |
|    B     |      subtrahend，您要减去的值。       | 浮点型 [[1\]](http://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Subtract#anchor) |

#### Divide 节点

除法节点*（快捷键：D 键）*进行算术除法，其中 [A](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Divide#parama) 除以 [B](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Divide#paramb) 或 **（ [A](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Divide#parama) / [B](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Divide#paramb) ）。**除以 0 将导致*无穷大*，这通常会转化为非预期结果。

多通道数据类型之间的划分是按通道完成的。如果 A 和 B 具有不同的通道数量，则执行强制转换以匹配具有最多通道的通道。

**注意：**除法*不是*可交换的，所以 value 的顺序*很重要*。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\DivideNode.jpg)
使用的节点：[Texture Sample （纹理采样](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)）、Divide （划分）

| 节点参数 |                            描述                            | 默认值 |
| :------: | :--------------------------------------------------------: | :----: |
|   一个   | 被除数，你想要分割的价值。仅当未连接相应的输入端口时可见。 |   0    |
|    B     |   除数，您要被除以的值。仅当未连接相应的输入端口时可见。   |   0    |

| 输入端口 |            描述            |                             类型                             |
| :------: | :------------------------: | :----------------------------------------------------------: |
|   一个   | 被除数，你想要分割的价值。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Divide#anchor) |
|    B     |   除数，您要被除以的值。   | 浮点型 [[1](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Divide#anchor) |

#### Abs 节点

Abs 节点提供标量或向量的各个分量的绝对值。在实践中，这意味着删除值的任何负号，只留下*绝对*值。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\AbsGraphicNode.png)

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\AbsNode.gif)

| 节点参数 |                            描述                             | 默认值 |
| :------: | :---------------------------------------------------------: | :----: |
|   输入   | 要在 *Abs* 计算中使用的值。仅当未连接相应的输入端口时可见。 |   0    |

| 输入端口 |            描述             |                             类型                             |
| :------: | :-------------------------: | :----------------------------------------------------------: |
|  *输入*  | 要在 *Abs* 计算中使用的值。 | 浮点型 [[1](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Abs#anchor) |

#### 一个减节点(one Minus)

One Minus 节点*（快捷键：O 键 ）*输出 1 减去指定值 *1 - [Input](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/One_Minus#paramInput)*。 这对于颜色或 UV 坐标非常有用，因为它会反转它们的值。

**注意：**多通道数据类型的 1 个 Minus 是按通道完成的。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\OneMinusGraphicNode.png)

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\OneMinusNode.jpg)
使用的节点：[Color](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Color)、[Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)、One Minus、[Multiply](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply)

| 节点参数 |                            描述                             | 默认值 |
| :------: | :---------------------------------------------------------: | :----: |
|   输入   | 用于 *One Minus* 计算的值。仅当未连接相应的输入端口时可见。 |   0    |

| 输入端口 |            描述             |                             类型                             |
| :------: | :-------------------------: | :----------------------------------------------------------: |
|  *输入*  | 用于 *One Minus* 计算的值。 | 浮点型 [[1](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/One_Minus#anchor) |

#### Power Node

Power 节点*（快捷键：E key ）*使用幂运算符 [Base](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Power#paramBase)[Exp](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Power#paramExp) 计算值。

- *当 Exp* 大于 0 时，它对应于将基数乘以自身 exp 乘以，p.e *Base = 2，Exp* *= 3* 得到 *2 x 2 x 2*
- *当 Exp* 低于 0 时，它对应于上述逆运算，p.e *Base = 2，Exp* *= -3* 结果为 *1/（2 x 2 x 2）*
- 另一方面，如果 *Exp* 等于 0，则它将始终输出 1，与指定的 *Base* 值无关。

打开多通道数据类型的电源会生成相同类型的输出，并按通道应用运算符。

**注意：**请查看下表，了解电源操作的完整行为，具体取决于其输入值。

| Base | Exp  |            Result            |
| :--: | :--: | :--------------------------: |
| < 0  | any  |             NaN              |
| > 0  | == 0 |              1               |
| == 0 | > 0  |              0               |
| == 0 | < 0  |           Infinite           |
| > 0  | < 0  |         1 / Base-Exp         |
| == 0 | == 0 | Depends on GPU (0, 1 or NaN) |

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\PowerNode.jpg)
使用的节点：[Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float)、[Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)、Power

| 节点参数 |                        描述                        | 默认值 |
| :------: | :------------------------------------------------: | :----: |
|   基础   | 基于幂运算符的值。仅当未连接相应的输入端口时可见。 |   0    |
|   实验   |   Exponent 值。仅当未连接相应的输入端口时可见。    |   0    |
| 安全电源 |  启用后，确保 Base 始终大于 0，以避免不确定/NaN。  |   假   |

| 输入端口 |                 描述                  |                             类型                             |
| :------: | :-----------------------------------: | :----------------------------------------------------------: |
|   基础   |          基于幂运算符的值。           | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Power#anchor) |
|   实验   | Exponent 值。此端口将最小值设置为 0。 |                              浮                              |



#### Clamp 节点

Clamp 节点输出其 [Input](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Clamp#paramInput) 值或限制在 [ [Min](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Clamp#paramMin) ， [Max](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Clamp#paramMax) ] 范围内的向量的单个分量。

- **最小值：**如果输入值小于 Min，则返回此值
- ***输入：***如果输入值介于 Min 和 Max 之间，则返回此值
- **麦克斯：**如果输入值大于 Max，则返回此值

**注意：**如果每个输入端口具有不同的通道数量，则执行强制转换以匹配具有最多通道的端口。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\ClampNode.gif)
使用的节点：[Vertex TexCoord](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Vertex_TexCoord)、[Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float)、Clamp、[Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)

| 节点参数 |                             描述                             | 默认值 |
| :------: | :----------------------------------------------------------: | :----: |
|  *输入*  | 要限制在 [ Min ， Max ] 范围之间的值。仅当未连接相应的输入端口时可见。 |   0    |
|  最小值  | clamp 操作的最小值，如果 value 小于 Min ，则为返回值。仅当未连接相应的输入端口时可见。 |   0    |
|  麦克斯  | clamp 操作的最大值，如果 value 大于 Max than，则返回值。仅当未连接相应的输入端口时可见。 |   1    |

| 输入端口 |                             描述                             |                             类型                             |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  *输入*  | 要限制在 [ Min ， Max ] 范围之间的值。当 *Alpha = 0* 时完全输出。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Clamp#anchor) |
|  最小值  |    clamp 操作的最小值，如果 value 小于 Min ，则为返回值。    | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Clamp#anchor) |
|  麦克斯  |   clamp 操作的最大值，如果 value 大于 Max than，则返回值。   | 浮点型 [[](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Clamp#anchor) |



#### Remap重映射节点

Remap 节点将在其 [Input](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Remap#paramInput) 上设置的值从 [ [Min Old](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Remap#paramMinOld) ， [Max Old](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Remap#paramMaxOld) ] 之间的范围转换为由 [ [Min New](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Remap#paramMinNew) ， [Max New](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Remap#paramMaxNew) ] 定义的新值。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\RemapNode.jpg)
使用的节点：[Texture Coordinates](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates)、[Tau](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Tau)、[Multiply](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply)、[Sin](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Sin)、[Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float)、Remap

| 节点参数 |                          描述                          | 默认值 |
| :------: | :----------------------------------------------------: | :----: |
|   输入   | 要在范围之间转换的值。仅当未连接相应的输入端口时可见。 |   0    |
| Min Old  |   低于原始范围的值。仅当未连接相应的输入端口时可见。   |   0    |
| Max Old  |   原始范围的上限值。仅当未连接相应的输入端口时可见。   |   0    |
|  最小新  |   新范围中的较低值。仅当未连接相应的输入端口时可见。   |   0    |
| 最大新建 |    新范围的上限值。仅当未连接相应的输入端口时可见。    |   0    |



| 输入端口 |          描述          | 类型 |
| :------: | :--------------------: | :--: |
|  *输入*  | 要在范围之间转换的值。 |  浮  |
| Min Old  |   低于原始范围的值。   |  浮  |
| Max Old  |   原始范围的上限值。   |  浮  |
|  最小新  |   新范围中的较低值。   |  浮  |
| 最大新建 |    新范围的上限值。    |  浮  |

#### Step 节点

Step 节点实现一个 Step 函数，如果 [B](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Step#paramB) 的值或向量分量大于或等于 [A](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Step#paramA) 的值或相应的向量分量，则返回 1。否则，它将返回 0。

- **0：**如果 B < A，则返回此值
- **1：**如果 B >= A

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\StepNode.jpg)
使用的节点：[Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)、[Vertex TexCoord](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Vertex_TexCoord)、[Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float)、Step、[Lerp](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp)

| 节点参数 |                         描述                          | 默认值 |
| :------: | :---------------------------------------------------: | :----: |
|   一个   | step 操作的第一个值。仅当未连接相应的输入端口时可见。 |   0    |
|    B     | step 操作的第二个值。仅当未连接相应的输入端口时可见。 |   0    |



| 输入端口 |         描述          |                             类型                             |
| :------: | :-------------------: | :----------------------------------------------------------: |
|   一个   | step 操作的第一个值。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Step#anchor) |
|    B     | step 操作的第二个值。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Step#anchor) |

#### Smoothstep 节点

如果 [Smoothstep](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#paramInput) 节点的 Input 值在 [ [Min](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#paramMin) ， [Max](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#paramMax) ] 范围内，则计算一个介于 **0** 和 **1** 之间的平滑 Hermite 插值。
假设 *Max* 值大于 *Min*，则如果 *Alpha* 小于 *Min*，则返回值 **0**。如果反之，则 *Alpha* 大于 *Max*，则返回值 **1**。

**注 1：**多通道数据类型之间的 Smoothstep 是按通道完成的。如果 *Min* 和 *Max* 具有不同的通道数量，则执行强制转换以匹配具有最多通道的通道。
**注 2：**避免将 [Min](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#paramMin) 和 [Max](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#paramMax) 设置为相等的值，因为它会引发除以 0 的警告。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\SmoothstepGraphicNode.png)

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\SmoothstepNode.jpg)
使用的节点：[Color](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Color)、[Vertex TexCoord](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Vertex_TexCoord)、[Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float)、Smoothstep、[Lerp](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Lerp)

| 节点参数 |                             描述                             | 默认值 |
| :------: | :----------------------------------------------------------: | :----: |
|  *输入*  |       指定要插值的值。仅当未连接相应的输入端口时可见。       |   0    |
|  最小值  | 指定最小参考值以开始插值。如果 Alpha 的值低于 this，则输出 0。仅当未连接相应的输入端口时可见。 |   0    |
|  麦克斯  | 指定最大参考值以结束插值。如果 Alpha 的值高于 this，将输出 1。仅当未连接相应的输入端口时可见。 |   0    |

| 输入端口 |                             描述                             |                             类型                             |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  *输入*  |                       指定要插值的值。                       | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#anchor) |
|  最小值  | 指定最小参考值以开始插值。如果 Alpha 的值低于 this，则输出 0。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#anchor) |
|  麦克斯  | 指定最大参考值以结束插值。如果 Alpha 的值高于 this，将输出 1。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Smoothstep#anchor) |

#### Rotator

Rotator （旋转器） 节点从 [Anchor （锚点](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Rotator#paramAnchor)） 旋转 UV 或其他 Vector 2 位置 [（向量](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Rotator#paramTime) 2） 位置 时间 角度值。 如果未在 [Time](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Rotator#paramTimePort) input port 上设置输入，则使用 Unity 计时器连续递增内部角度值并提供旋转动画。在这种情况下，Time 属性充当 Unity 计时器值的乘数。
另一方面，如果在 Time input 端口上建立了连接，则它将假定该值为最终值，并且在内部不使用任何类型的计时器。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\RotatorNode.gif)
使用的节点：[Texture Coordinates](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates)、[Vector2](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Vector2)、[Time](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Time)、Rotator、[Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)

| 节点参数 |                             描述                             | 默认值 |
| :------: | :----------------------------------------------------------: | :----: |
|    uv    |         要旋转的点。仅当未连接相应的输入端口时可见。         |  0,0   |
|  anchor  |      旋转时使用的锚点。仅当未连接相应的输入端口时可见。      |  0,0   |
|  timer   | 时间乘数，允许扩展 Unity 内部计时器。仅当未连接相应的输入端口时可见。 |   0    |



| 输入端口 |                    描述                    |  类型   |
| :------: | :----------------------------------------: | :-----: |
|    uv    |                要旋转的点。                | vector2 |
|  anchor  |             旋转时使用的锚点。             | vector2 |
|   time   | 要应用于指定点的旋转角度（以弧度为单位）。 |  float  |

#### Fresnel Node菲涅尔节点

The Fresnel node outputs the result of a Fresnel effect. It defines how light behaves when it reaches an interface between two materials with different refractive indices, how much of it is reflected and refracted.Fresnel 节点输出 Fresnel 效果的结果。它定义了光在到达具有不同折射率的两种材料之间的界面时的行为方式，以及光的反射和折射量。
This node in particular deals with the reflection part and calculates the Fresnel Reflection Coefficient which is defined by the following expression **ReflectionCoefficient = [Bias](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramBias) + [Scale](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramScale) x ( 1 + [N](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramNormal).I )[Power](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramPower).

**Each member of the equation can be modified except the **I** variable which defines an Incident vector from the camera to the object and is internally calculated by the node as the inverted World View Direction vector.该节点特别处理反射部分并计算菲涅耳反射系数，该系数由以下表达式定义：ReflectionCoefficient = [Bias](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramBias) + [Scale](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramScale) x （ 1 + [N](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramNormal).I ） [Power](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Fresnel#paramPower)。 方程式的每个成员都可以修改，但 I 变量除外，**该**变量定义从相机到对象的 Incident 向量，并由节点在内部计算为反转的 World View Direction 向量。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\FresnelNode.jpg)
Nodes used: [Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float), [Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample), Fresnel, [Color](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Color), [Multiply](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply)使用的节点： [浮点型](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float) / [纹理样本](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample) / 菲涅耳 / [颜色](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Color) / [乘](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Multiply)

|                   Node Parameter节点参数                    |                       Description描述                        | Default Value 默认值  |
| :---------------------------------------------------------: | :----------------------------------------------------------: | :-------------------: |
|                          Type 类型                          | Specifies Fresnel type指定菲涅耳类型**Standard:** More well known fresnel effect**标准：**更广为人知的菲涅耳效果**Schlick:** Approximation for fresnel equation using reflectance at normal incidence**施利克：**使用法向入射反射率的菲涅耳方程的近似值**Schlick IOR:** Approximation for fresnel equation using index of refraction**施利克 IOR：**使用折射率的菲涅耳方程的近似值 |     Standard 标准     |
|                   Normal Vector 法线向量                    | Specifies in which coordinates space the normal vector is指定法线向量所在的坐标空间**Tangent Normal:** Normal vector is in a tangential space coordinates**Tangent Normal（切线法线）：**法向量位于切向空间坐标中**World Normal:** Normal vector is in a world space coordinates**世界法线（World Normal）：**法线向量位于世界空间坐标中**Half Vector:** Specified vector is an half vector (Vector between Light Dir and View Dir)**Half Vector（半向量）：**指定的向量是半向量（Light Dir 和 View Dir 之间的向量） | World Normal 世界法线 |
|                    View Vector 查看向量                     | Specifies what vector is set on View Dir input port指定在 View Dir input（视图方向）input port 上设置的向量**View Dir:** Specified vector is view direction**视图目录：**指定的向量是视图方向**Light Dir:** Specified vector is light direction**Light Dir（光源方向）：**指定的向量为光方向 |   View Dir 视图目录   |
|        Available only on Standard Type仅适用于标准型        |                                                              |                       |
|                          Bias 偏见                          | Defines the Bias variable of the Fresnel equation. Only visible if the respective input port is not connected.定义菲涅耳方程的 Bias 变量。仅当未连接相应的输入端口时可见。 |           0           |
|                         Scale 规模                          | Defines the Scale variable of the Fresnel equation. Only visible if the respective input port is not connected.定义 Fresnel 方程的 Scale 变量。仅当未连接相应的输入端口时可见。 |           1           |
|                         Power 权力                          | Defines the Power variable of the Fresnel equation. Only visible if the respective input port is not connected.定义菲涅耳方程的 Power 变量。仅当未连接相应的输入端口时可见。 |           5           |
|     Available only on Schlick Type仅适用于 Schlick Type     |                                                              |                       |
|                             F0                              |      Reflectance at normal incidence法向入射时的反射率       |           0           |
| Available only on Schlick IOR Type仅适用于 Schlick IOR 类型 |                                                              |                       |
|                             IOR                             |                  Index of Refraction折射率                   |           1           |
|                  Other properties其他属性                   |                                                              |                       |
|                 Normalize Vectors规范化向量                 | Forces all received vectors to be normalized强制所有接收到的向量都归一化 |       False 假        |
|                     Safe Power 安全电源                     | Guarantees that internal value used on powers base operation is positive to avoid NaN.保证 powers base 操作中使用的内部值为正，以避免 NaN。 |       False 假        |



|                    Input Port输入端口                    |                       Description描述                        |   Type 类型    |
| :------------------------------------------------------: | :----------------------------------------------------------: | :------------: |
| World/Tangent Normal or Half Vector世界/切线法线或半向量 |       Entry vector to be used.要使用的 Entry vector。        | Float 3 浮动 3 |
|                        Bias 偏见                         | Defines the Bias variable of the Fresnel equation.定义菲涅耳方程的 Bias 变量。 |    Float 浮    |
|                        Scale 规模                        | Defines the Scale variable of the Fresnel equation.定义 Fresnel 方程的 Scale 变量。 |    Float 浮    |
|                        Power 权力                        | Defines the Power variable of the Fresnel equation.定义菲涅耳方程的 Power 变量。 |    Float 浮    |
|                    IOR or F0IOR 或 F0                    | Index of refraction or reflectance at normal incidence value to be used.要使用的折射率或正常入射时的反射率值。 |    Float 浮    |

[Back to Node List返回节点列表](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Nodes)

数学相关

1.Fresnel效果

一般来说，当光到达2种材质的接触面时，一些光在接触面的表面被反射出去，而另一部分光将发生折射穿过接触面，这个现象称为Fresnel效果。Fresnel公式描述了多少光被反射和多少光被折射。下面是Fresnel公式的一个近似：

reflectionCoefficient=max(0,min(1,bias+[scale](https://so.csdn.net/so/search?q=scale&spm=1001.2101.3001.7020)×(1+I·N)power))

反射系数reflectionCoefficient的范围被限制在[0,1]之间，我们根据下面的公式使用reflectionCoefficient来混合反射和折射向量（C代表颜色）：

CFinal=reflectionCoefficient×CReflected+(1-reflectionCoefficient)×CRefracted

还有一个比较常用的近似计算Fresnel的公式：
fastFresnel = r + ( 1-r ) * pow ( 1.0 – dot ( viewDir, normal ), 5.0);
反射系数r的值可以通过物理测量获得，例如空气到水面的反射系数约为0.02037。
从而
CFinal=fastFresnel×CReflected+(1-fastFresnel)×CRefracted

2.颜色色散

折射量除了基于表面法向量、入射角和折射系数比率外，还决定于入射光的波长。例如，红光要比蓝光折射得多，这个现象称为颜色色散（chromatic dispersion）。这就是为什么当白光进入棱镜会形成一个彩虹的原因。

真正的光有一个波段的波长，而不仅仅只有3个特殊的离散波长。但我们可以近似地只模拟光的红、绿和蓝色分量的波长。实践证明，这种近似是有效的。

结合了颜色色散的Fresnel效果将创建一种彩虹效果，就好像被渲染的物体是由水晶做成的一样。

转载于:https://www.cnblogs.com/skyman/archive/2008/04/12/fresnel.html

**这个节点用于将相应到达物体表面的光进行处理，处理时反射还是折射**，从而实现物体表面的与光的真是效果

人们从不同的角度观察物体可能会得到不同的亮度，和反射强度

##### 菲涅尔反射弧的意义

![image-20241116175313430](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116175313430.png)

那么在unity中的使用

![image-20241116175508140](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116175508140.png)

实现的一个效果

从不同的角度，得到的物体不一样

![image-20241116175522776](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116175522776.png)

[Jump to navigation跳到导航](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#mw-head)[Jump to search跳转到搜索](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#searchInput)

[Back to Node List返回节点列表](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Nodes)

#### Voronoi NodeVoronoi 节点

The Voronoi node creates a voronoi pattern value under a [0 1] range based using the method specified by [Type](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramMethod) according to the values specified by [UV](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramUV), [Angle](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramAngle) and [Scale](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramScale).Voronoi 节点使用 [Type](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramMethod) 指定的方法，根据 [UV](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramUV)、[Angle](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramAngle) 和 [Scale](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Voronoi#paramScale) 指定的值，在 [0 1] 范围内创建 voronoi 图案值。
**NOTE:** Input data must vary across the the geometry since equal values will generate the same voronoi value. A simply way to achieve this is to connect a [Texture Coordinates](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates) node into its input.**注意：**输入数据必须随几何图形而变化，因为相等的值将生成相同的 voronoi 值。实现此目的的一种简单方法是将 [Texture Coordinates （纹理坐标](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates)） 节点连接到其输入。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\VoronoiNode.jpg)
Nodes used: [Texture Coordinate](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates), [Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float), Voronoi使用的节点： [纹理坐标](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates) / [浮点型](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float) / 沃罗诺伊

|                    Node Parameter节点参数                    |                       Description描述                        |  Default Value 默认值  |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :--------------------: |
|                         Method 方法                          | Method used to generate the voronoi pattern. Each method provides visually different patterns on top of the same seed input.用于生成 voronoi 模式的方法。每种方法在相同的 seed input 之上提供视觉上不同的 pattern。**Cells细胞****Cristal晶体****Glass玻璃****Caustics焦 散****Distance距离** |       Cells 细胞       |
| Available to all Methods except Distance可用于除 Distance 之外的所有 Method |                                                              |                        |
|                  Distance Function距离函数                   | Pattern is calculated via the distance to each seed. Each distance function generates different results on top of the same voronoi type.模式是通过到每个种子的距离来计算的。每个距离函数在相同的 voronoi 类型之上生成不同的结果。**Euclidian^2欧几里得^2****Euclidian欧几里得****Manhattan曼哈顿****Chebyshev切比雪夫****Minkowski闵可夫斯基** | Euclidian^2 欧几里得^2 |
| Available only on Minkkowsky Methof except Distance仅在 Minkkowsky Methof 上可用，但 Distance 除外 |                                                              |                        |
|                Minkowski Power 闵可夫斯基动力                | Controls the smoothness of the pattern. Value has a [1 5] range.控制图案的平滑度。Value 的范围是 [1 5]。 |           1            |
|                    SearchQuality 搜索质量                    | Controls the amount of iterations done to calculate the distance and check which seed is closest. Increased number means increase computation but finer results.控制为计算距离和检查哪个种子最近的迭代次数。增加数量意味着增加计算量，但结果更精细。**9 Cells9 格****25 Cells25 节****49 Cells49 格** |      9 Cells 9 格      |
|                       Octaves 八度音程                       | Controls the amount of iterations done per fragment setting smooth sub-divisions on each cell. Value has a [1 8] range, where the higher it gets, the more sub-divided the pattern is (and more expensive as well ).控制每个片段完成的迭代次数，在每个单元格上设置平滑的细分。value 有一个 [1 8] 范围，其中越高，模式的细分就越多（也更昂贵）。 |           1            |
|                       Tileable 可平铺                        | Forces the generated pattern to be tileable强制生成的图案可平铺 |        False 假        |
| Available only if Tileable is active仅当 Tileable 处于活动状态时可用 |                                                              |                        |
|                     Tile Scale 平铺比例                      |          Amount of tiling to apply.要应用的平铺量。          |           2            |
|     Only available on Cells Method仅适用于 Cells Method      |                                                              |                        |
|                         Smooth 光滑                          | Smooths out the distance value when calculating distance to seed.在计算到种子的距离时平滑距离值。 |        False 假        |
| Available only if Smooth is active仅当 Smooth 处于活动状态时可用 |                                                              |                        |
|                     Apply To ID应用于 ID                     | Applies smoothness to the ID output similar to Blender software.将平滑度应用于 ID 输出，类似于 Blender 软件。 |        False 假        |
|               Unity's Voronoi Unity 的 Voronoi               | Calculates voronoi pattern similiar to Shader Graph [Voronoi node](https://docs.unity3d.com/Packages/com.unity.shadergraph@12.0/manual/Voronoi-Node.html).计算 voronoi 模式，类似于 Shader Graph [Voronoi 节点](https://docs.unity3d.com/Packages/com.unity.shadergraph@12.0/manual/Voronoi-Node.html)。 |        False 假        |



| Input Port输入端口 |                       Description描述                        |    Type 类型    |
| :----------------: | :----------------------------------------------------------: | :-------------: |
|         UV         | Value used to generate pattern. The same entry value always generate the same noise value.用于生成 pattern 的值。相同的输入值始终生成相同的噪声值。 | Float2 浮点数 2 |
|     Angle 角度     |   Desired angle between each cell每个单元格之间的所需角度    |    Float 浮     |
|     Scale 规模     | Value to scale the input given over the UV port.值可缩放通过 UV 端口给定的输入。 |    Float 浮     |
|  Smoothness 平滑   | Smoothness value to be applied over a smooth step in the distance function要应用于距离函数中平滑步长的平滑度值 |    Float 浮     |



| Output Port输出端口 |                Description描述                |    Type 类型    |
| :-----------------: | :-------------------------------------------: | :-------------: |
|       Out 外        |     Calculated pattern value计算的模式值      | Float4 浮点数4  |
|         ID          |      ID of the current seed当前种子的 ID      | Float2 浮点数 2 |
|         UV          | Generated UV for each seed为每个种子生成的 UV | Float2 浮点数 2 |

https://wiki.amplify.pt/index.php?title=Category:Nodes&action=edit&redlink=1)

实际运用

Voronoi 图在多个领域有广泛的实际应用，以下是一些主要的应用场景：

1. **地理信息系统（GIS）**：
   - Voronoi 图可以用于分析地理数据，帮助确定服务区域（如医院、学校、消防站等）的覆盖范围，优化资源分配。

2. **城市规划**：
   - 在城市规划中，Voronoi 图可以用于确定最佳位置以建立新设施，确保服务区域的合理分布，减少重叠和空白区域。

3. **机器人路径规划**：
   - 在机器人导航中，Voronoi 图可以帮助机器人规划路径，确保机器人在环境中移动时保持一定的安全距离，避免障碍物。

4. **图像处理**：
   - Voronoi 图可用于图像分割和特征提取，帮助识别图像中的不同区域或对象。

5. **生物学**：
   - 在生态学和生物学研究中，Voronoi 图可以用于分析动物的栖息地分布、种群动态等。

6. **网络设计**：
   - 在通信网络和传感器网络设计中，Voronoi 图可以帮助优化节点的布局，以提高网络的覆盖率和效率。

7. **计算机图形学**：
   - Voronoi 图可以用于生成自然现象的模拟，如地形生成、纹理映射等，增强视觉效果的真实感。

8. **游戏开发**：
   - 在游戏设计中，Voronoi 图可以用于创建地图、生成随机场景、优化角色的移动路径等。

这些应用展示了 Voronoi 图在解决实际问题中的重要性和灵活性。通过将 Voronoi 图与其他算法和技术结合，能够为各种复杂问题提供有效的解决方案。

##### 相关运用及其原理

###### 一、说明

​     **Voronoi图**（也称为*狄利克雷镶嵌*或*泰森多边形*）在自然界中无处不在。你已经遇到过他们数千次了，但也许没有这样称呼它。Voronoi图很简单，但它们具有令人难以置信的特性，在制图，生物学，计算机科学，统计学，考古学，一直到建筑和艺术等领域都有应用。

###### 二、什么是沃罗诺伊图？

​    假设您有 n 个点分散在一个平面上，这些点的 Voronoi 图将平面细分为正好 *n 个*单元格，这些单元格包围了最接近每个点的平面部分。这将产生完全覆盖平面的镶嵌。作为说明，在图 1 中，我绘制了 100 个随机点及其相应的 Voronoi 图。如您所见，每个点都包含在一个像元中，该像元的边界在两个或多个点之间正好等距。换句话说，单元格中包含的所有区域都最靠近单元格中的点，而不是任何其他点。

![img](https://i-blog.csdnimg.cn/blog_migrate/201cd617ff832a481dbcfa3718c35fde.png)

图1：沃罗诺伊图


###### 三、沃罗诺伊模式无处不在

**3.1 自然界中的沃罗诺伊模式**

​    由Voronoi图创建的模式在自然界中是常见的。在图 2 中，我制作了一些自然发生的类似 Voronoi 的图案的小拼贴画。从洋葱皮中的微观细胞，到菠萝蜜的壳和长颈鹿的皮毛。这些模式无处不在！

​    它们无处不在的第一个原因是它们形成了有效的形状。正如我们前面提到的，Voronoi 图完全细分了平面：因此，所有空间都被使用。如果您想在有限的空间内尽可能多地挤压，例如在肌肉纤维或蜂箱中，这将非常方便。其次，Voronoi图是一种自发模式，每当某物从不同的点以均匀的增长率增长时（见图3）。例如，这就解释了为什么长颈鹿表现出这种模式。长颈鹿胚胎具有分散分布的黑色素分泌细胞，这是长颈鹿斑点深色色素沉着的原因。在妊娠过程中，这些细胞释放黑色素 - 因此斑点向外辐射。有兴趣的读者可以参考[这篇论文](https://www.iro.umontreal.ca/~poulin/fournier/papers/Walter-2001-ISP/Walter-2001-ISP.pdf)，其中作者使用Voronoi图对动物外套上的斑点进行计算机渲染建模。

![img](https://i-blog.csdnimg.cn/blog_migrate/a55cf290d343ebb99e97b7d1f1b49067.jpeg)

图2：Voronoi模式在自然界中无处不在。
（从左上到右下：肌肉的横截面，长颈鹿的毛纹，蜻蜓的翅膀，大蒜鳞茎，玉米，挂在树上的菠萝蜜。
来源：[维基百科](https://en.wikipedia.org/wiki/Skeletal_muscle#/media/File:Denervation_atrophy_-_atp94_-_intermed_mag.jpg)，[Nirav Shah](https://www.pexels.com/it-it/foto/alberi-animale-fauna-selvatica-mammifero-7320450/)，[Karolina Grabowska](https://www.pexels.com/it-it/foto/cibo-insalata-salutare-cena-4022177/)，[StarGlade](https://pixabay.com/vectors/dragonfly-wing-bug-insect-fly-1531038/)，[Mali Maeder](https://www.pexels.com/it-it/foto/mais-giallo-547263/)，[Abi Jacob](https://www.pexels.com/it-it/foto/cibo-foglie-albero-crescita-7789303/)

![img](https://i-blog.csdnimg.cn/blog_migrate/7eddf4c7815f632c46d110454647d820.gif)

图 3：从分散点
向外不断增长获得 Voronoi 图 来源：[维基百科](https://en.wikipedia.org/wiki/Voronoi_diagram#/media/File:Voronoi_growth_euclidean.gif)

**3.2 建筑和艺术中的沃罗诺伊模式**

​    也许是因为它们自发的“自然”外观，或者仅仅是因为它们令人着迷的随机性，Voronoi模式被有意地在人造结构中实现。一个建筑例子是2008年北京奥运会期间为容纳水上运动而建造的“水立方”。它的天花板和立面上有Voronoi图（图4）。选择Voronoi图是因为它们回忆起气泡1。这个类比在晚上非常明显，当整个立面被蓝色照亮并活跃起来时。

![img](https://i-blog.csdnimg.cn/blog_migrate/aa7575cef48ee77d18758d0c1a036baf.png)

图4：北京


​    但中国人对沃罗诺伊图案的欣赏肯定比这座建筑更古老。宋代的[关](https://en.wikipedia.org/wiki/Guan_ware)、[葛](https://en.wikipedia.org/wiki/Ge_ware)皿有独特的噼啪作响的釉。陶瓷在冷却过程中很容易破裂，但是关和葛瓷的裂纹是不同的——它们是故意的。它们因其美学品质而受到追捧。由于表面的Voronoi图案，每件作品都是独一无二的。迄今为止，这是最常被模仿的瓷器风格之一（图5）。

![img](https://i-blog.csdnimg.cn/blog_migrate/c6336cc7a1bbb259e1269f9c9de802ae.png)

图5：关和葛炳


​    Voronoi图在图形艺术中也很常见，用于创建“抽象”图案。我认为它们可以制作出色的背景图像。例如，我通过生成随机点并构建Voronoi图来创建这篇文章的缩略图。然后，我根据每个单元格的点与框中随机选择的点的距离对每个单元格进行着色（图 6）。无穷无尽的“抽象”背景图像可以通过这种方式生成。

![img](https://i-blog.csdnimg.cn/blog_migrate/fc654aa1588d6a3bf6f74afa693e8395.gif)

图6：彩色沃罗诺伊图


###### 四、数学定义和一些有趣的属性

​    到目前为止，我们已经提出了一个简单的二维Voronoi图。然而，相同类型的结构可以推广到 n 维空间。假设 P={p1，p2,...,pm} 是我们 *n* 维空间中的*一组 m* 个点。然后，可以将空间划分为 *m* Voronoi 单元 *Vi*，其中包含 *Rn* 中比任何其他点更接近 pi 的所有点。

![img](https://i-blog.csdnimg.cn/blog_migrate/086d221e53156c86f27348d6afcb56b2.png)

​    其中函数 *d（x，y）* 给出其两个参数之间的距离 （*a*）。通常，使用欧几里得距离（l2 距离）：

![img](https://i-blog.csdnimg.cn/blog_migrate/027a5b5a6f52b64d453cc0578ec4055e.png)

​    但是，可以使用其他距离函数设计Voronoi图。例如，图 7 显示了使用曼哈顿或城市街区距离（*l1* 距离）获得的 Voronoi 图。Manahattan 距离是两点之间的距离，如果您必须遵循常规网格（例如曼哈顿的城市街区）。结果是一个更“四四方方”的Voronoi图。

![img](https://i-blog.csdnimg.cn/blog_migrate/c37ba26931c80156cea557c2e7f75d8a.png)

![img](https://i-blog.csdnimg.cn/blog_migrate/9d6ce7bb236f20bfcdd7d6e81c5f8efe.png)

图7：欧几里得距离


![img](https://i-blog.csdnimg.cn/blog_migrate/72e75281d5d9372f8c25d363d8a569ee.png)

图 8：同一组点
使用欧几里得（左）和曼哈顿（右）距离的 Voronoi 图比较 来源：[维基百科](https://en.wikipedia.org/wiki/File:Manhattan_Voronoi_Diagram.svg)

​    欧几里得距离是Voronoi图科学应用中最常见的距离度量。它还具有生成**凸**面Voronoi细胞的优点。也就是说，如果您在单元格内取任意两个点，则连接这两个点的线将完全位于单元格内。

​    最后，还应该注意的是，Voronoi 图与 k 最近邻算法 （k-NN） 紧密相连，该算法是分类、回归和聚类问题中非常流行的算法。该算法使用训练数据集中最接近的 *k* 个示例进行值预测。由于 Voronoi 图在包含每个种子最近的点的多边形中划分空间，因此 Voronoi 像元的边与简单的 1-最近邻问题的决策边界完全对应。

###### 五、德劳奈三角测量

​    如果您从 Voronoi 图中获取每个点并将其与其相邻单元格中的点链接，您将获得一个名为 **Delaunay 三角测量**的图形。用数学术语来说，德劳奈三角测量是沃罗诺伊图的[对偶图](https://www.wikiwand.com/en/Dual_graph)。在下图中，从一组点绘制了Voronoi图（黑色）和Delaunay三角测量（灰色）。

![img](https://i-blog.csdnimg.cn/blog_migrate/6e03ac147bbdc6992f6404bb54460afa.png)

图9：带有德劳奈三角测量的Voronoi图
来源：作者图片

​    德劳奈三角测量与沃罗诺伊图一样令人惊叹。顾名思义，它会产生一组连接我们点的三角形。这些三角形是这样的，如果要在这些三角形的顶点上画一个圆，圆内就不会有其他点（见图 10）。此外，德劳奈三角测量还具有最大化三角三角形中最小角度的特性。因此，德劳奈三角测量倾向于避免具有锐角的三角形。

![img](https://i-blog.csdnimg.cn/blog_migrate/78fb7dddaab2540c529f68b424cd2861.png)

图 10：Delaunay 三角形的构造使得没有点落在环绕每个三角形的圆内
来源：[维基百科](https://en.wikipedia.org/wiki/File:Delaunay_circumcircles_vectorial.svg)

​    这些属性使其在从一组点对表面和对象进行建模时非常有用。例如，Delaunay 三角测量用于为[有限元方法](https://people.eecs.berkeley.edu/~jrs/meshpapers/delnotes.pdf)生成网格，为计算机动画构建 3D 模型以及在 GIS 分析中对地形进行建模。

###### 六、劳埃德松弛算法

​    **Llyod**算法是一种与Voronoi图相关的有用算法。该算法包括在构建Voronoi图和查找每个单元的质心（即质心）之间反复交替（见图11）。在每次迭代中，该算法将点分开并产生更均匀的Voronoi单元。

![img](https://i-blog.csdnimg.cn/blog_migrate/d164fea12531ee71a112538c4b4fadf5.png)

图 11：劳埃德松弛算法
中的步骤 来源：作者图片

​    经过几次迭代后，单元格将已经具有“更圆”的方面，并且点将更均匀地分布。下图对此进行了说明，其中我绘制了一组随机点的劳埃德算法的前 30 次迭代。对于每个点，我还记录它们的起始位置（灰色空心圆圈），以更好地跟踪每个单元格的运动。对于大量迭代，该图倾向于收敛到稳定的Voronoi图，其中每个种子也是细胞的质心 - 也称为[质心Voronoi图](https://en.wikipedia.org/wiki/Centroidal_Voronoi_tessellation)。有趣的是，在2D中，Voronoi单元往往会变成六边形，因为它们提供了在平面中包装形状的最有效方式。正如任何建造蜂巢的蜜蜂都可以证明的那样，六边形细胞具有两大优势：1）它们确保细胞之间没有空白空间（即镶嵌平面），以及2）六边形提供细胞表面和周长之间的最高比例。这个所谓的[蜂窝猜想](https://en.wikipedia.org/wiki/Honeycomb_conjecture)，花了数学家两千年的时间才证明。

![img](https://i-blog.csdnimg.cn/blog_migrate/ec58999b4e7dbc80659809a0b6d7bf80.gif)

图 12：劳埃德算法
的 30 次迭代 来源：作者图片

​    在数据科学中，劳埃德算法是k均值聚类的基础，**k-means**聚类是最流行的聚类算法之一。k 均值聚类通常是通过在空间中获取 *k* 个随机“质心”来启动的。然后，通过交替将数据点分组为 *k* 个簇：1） 将数据点分配给最接近的“质心”（这相当于为质心构建 Voronoi 图并检查哪个点在单元内）和 2） 通过计算每个单元内点的平均值来更新质心（参见图 14）。

![img](https://i-blog.csdnimg.cn/blog_migrate/2f896a3b4af0d2254787be134f4f0332.gif)

图 14：k 均值聚类


​    除了数据科学，劳埃德的算法还用于各种应用。例如，它在量化和有损数据压缩算法（例如[劳埃德-马克斯算法](http://helmut.knaust.info/presentations/2011/20110106_JMM_DWT.pdf)）中很常见。每当人们想要间隔良好的随机点时，它也非常有用。例如，它可用于从 Delaunay 三角测量生成平滑网格，用于[抖动](https://en.wikipedia.org/wiki/Dither)图像，或作为视频游戏中[程序地图生成](https://en.wikipedia.org/wiki/Procedural_generation)的基础。

###### 七、如何构建沃罗诺伊图？

​    人们可以通过逐个构建每个单元格来构建Voronoi图。如果扩展连接每个点组合的段的平分线，则可以获得Voronoi单元的轮廓（图15）。但是，这种技术效率很低。考虑到有*0.5\*（n-1）n*个点的组合，这种算法的复杂性会随着点数的增加呈二次增加。

![img](https://i-blog.csdnimg.cn/blog_migrate/f62fb5c0e8e01b28867b09faf87f0009.png)

图15：Voronoi细胞
的构建 来源：作者图片

​    已经提出了更有效的替代办法。例如，[扫描线算法](https://en.wikipedia.org/wiki/Fortune's_algorithm)通过使用二叉搜索树和优先级队列操作按顺序逐步构建 Voronoi 单元（图 16）。可以[在此处](https://www.ams.org/publicoutreach/feature-column/fcarc-voronoi)找到此算法的良好描述。构建Voronoi图的另一种方法是首先构建Delaunay三角测量。获得三角测量后，延伸三角形边的平分线通向Voronoi图。无需考虑每对点即可获得德劳内三角测量。例如，一种有效的技术是在更高维度上投影抛物面上的点。将凸包重新投影回原始空间，使德劳内三角测量。

![img](https://i-blog.csdnimg.cn/blog_migrate/e8214f2dfe384cd7facd47fd44e2eaae.gif)

图 16：扫描线算法
来源：[维基百科](https://en.wikipedia.org/wiki/Fortune's_algorithm)

​    有关计算Voronoi图的不同算法及其复杂性的讨论，请参见此处，此处和[此处](https://www.ic.unicamp.br/~rezende/ensino/mo619/Sacristan, Voronoi Diagrams.pdf)。不断提出新的算法来提高不同情况下的计算效率（例如Yan et al. 2011， [Qin et al. 2017](https://onlinelibrary.wiley.com/doi/abs/10.1111/cgf.13248)）。 还有一些技术需要恒定时间来生成近似的Voronoi图（例如[跳跃洪水算法](https://fbellelli.com/posts/2021-07-08-the-fascinating-world-of-voronoi-diagrams/Jump Flooding Algorithm)）。

###### 八、其他材料的链接

- [本文](https://plus.maths.org/content/uncovering-cause-cholera)讲述了约翰·斯诺（John Snow）如何使用Voronoi图来显示1854年伦敦爆发期间水泵与霍乱传播之间的联系的故事。
- [Amit Patel](https://simblob.blogspot.com/search/label/voronoi)有一个关于游戏开发的非凡博客。我强烈推荐他关于使用 Voronoi 图生成程序地图的文章。
- [David Austin的这篇文章](https://www.ams.org/publicoutreach/feature-column/fcarc-voronoi)很好地解释了用于计算Voronoi图的扫描线算法。
- [这张由](https://www.jasondavies.com/maps/voronoi/airports/)杰森·戴维斯（Jason Davies）绘制的精美地图是基于世界各地机场位置的Voronoi图。
- 空间镶嵌：Voronoi图[的概念和应用是一本关于Voronoi图](https://onlinelibrary.wiley.com/doi/book/10.1002/9780470317013)的圣经。如果您对Voronoi图有任何疑问，您一定会在这里找到答案。
- [Vincent](https://perso.uclouvain.be/vincent.legat/documents/meca2170/meca2170-cours7-Voronoi.pdf) Legat 的这些幻灯片包括一些针对不同构造算法的精美图纸。
- Voronoi图通常用于对森林中的树木进行建模（例如[Abellanas](https://pommerening.org/wiki/images/a/a4/Vorest.pdf)等人，2016年，[Bohler等人，2018](https://www.sciencedirect.com/science/article/abs/pii/S0925772117300652)年）。
- Voronoi图也可用于确定机器人的路径。检查这些文章：文章 [1](http://www.cs.columbia.edu/~pblaer/projects/path_planner/)、[文章 2](https://gamedevelopment.tutsplus.com/tutorials/how-to-use-voronoi-diagrams-to-control-ai--gamedev-11778)。
- Voronoi图有数千种应用程序。从在森林中建模树木到规划机器人路径。在这篇文章中，我几乎没有触及表面。这些链接包含有趣的应用程序列表：链接 [1](https://www.ics.uci.edu/~eppstein/gina/voronoi.html)、链接 [2](https://www.ics.uci.edu/~eppstein/gina/scot.drysdale.html)、链接 [3](https://en.cnki.com.cn/Article_en/CJFDTotal-GCTX200402022.htm)、[链接 4](https://ratt.ced.berkeley.edu/readings/read_online/Aurenhammer.  Voronoi diagrams.  A survey of a fundamental geometric data structure.pdf)、[链接 5](https://www.sciencedirect.com/topics/physics-and-astronomy/voronoi-diagrams)。
- 本文参考资料 [弗朗切斯科·贝莱利](https://medium.com/@fsabellelli?source=post_page-----da8fc700fa1b--------------------------------)

#### Depth Fade节点

[Jump to navigation跳到导航](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Depth_Fade#mw-head)[Jump to search跳转到搜索](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Depth_Fade#searchInput)

[Back to Node List返回节点列表](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Nodes)



The Depth Fade node outputs a linear gradient representing the distance between the surface of an object and geometry behind it. The gradient range or fading distance can be set by adjusting the [Distance](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Depth_Fade#paramDistance) parameter.Depth Fade （深度淡化） 节点输出一个线性渐变，表示对象表面与其后面的几何体之间的距离。渐变范围或淡化距离可以通过调整 [Distance](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Depth_Fade#paramDistance) 参数来设置。
More specifically what this parameter does is to create a value in a [0 1] range when the distance between the surface of an object and geometry behind it is within the value specified.更具体地说，当对象表面与其后面的几何体之间的距离在指定值范围内时，**此参数的作用是创建一个 [0 1] 范围内的值。**
**NOTE:** The shader must have its **Render Queue** value set to **Transparent** or higher ( p.e. **Blend Mode** set either to **Transparent** or **Translucent** ) so the object is not written on the Depth buffer. This is an essential configuration since the node is internally calculating the distance value by subtracting the [Surface Depth](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Surface_Depth) by the value fetched on the Depth buffer. If the object is being written on the depth buffer then this operation will give unintended results.**注意：**着色器的 **Render Queue** 值必须设置为 **Transparent** 或更高（例如，**将 Blend Mode** 设置为 **Transparent** 或 **Translucent** ），以便对象不会写入 Depth 缓冲区。 这是一个基本配置，因为节点在内部通过减去 [Surface Depth （表面深度](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Surface_Depth)） 除以在 Depth 缓冲区上获取的值来计算距离值。如果对象正在深度缓冲区上写入，则此操作将产生意外结果。


![img](http://wiki.amplify.pt/images/NodeDetail/DepthFadeNode.jpg)
Nodes used: [Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float), Depth Fade使用的节点： [浮点型](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float) / 深度淡化

|   Node Parameter节点参数    |                       Description描述                        | Default Value 默认值 |
| :-------------------------: | :----------------------------------------------------------: | :------------------: |
|  Vertex Position 顶点位置   | Don't use this internal data as it's ignored internally. Only visible if the respective input port is not connected.不要使用此内部数据，因为它在内部会被忽略。仅当未连接相应的输入端口时可见。 |          0           |
|        Distance 距离        | Distance in world space coordinates to which the fade should take place. Only visible if the [Dist](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Depth_Fade#paramDist) input port is not connected.淡化应达到的世界空间坐标中的距离。仅在未连接 [Dist](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Depth_Fade#paramDist) input 端口时可见。 |          1           |
| Convert to Linear转换为线性 | Convert depth values read from depth buffer from a logarithmic to a linear scale.将从深度缓冲区读取的深度值从对数刻度转换为线性刻度。 |       True 真        |
|         Mirror 镜子         | Applies an [Abs](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Abs) over the final value, guaranteeing the final value is always positive.对最终值应用 [Abs](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Abs)，保证最终值始终为正值。 |       True 真        |
|        Saturate 饱和        | Applies a [Saturate](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Saturate) over the final value, guaranteeing that the final value is on a 0 to 1 range.对最终值应用 [Saturate](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Saturate)，保证最终值在 0 到 1 的范围内。 |       False 假       |



|    Input Port输入端口    |                       Description描述                        |    Type 类型    |
| :----------------------: | :----------------------------------------------------------: | :-------------: |
|      Distance 距离       | Distance in world space coordinates to which the fade should take place.淡化应达到的世界空间坐标中的距离。 |    Float 浮     |
| Vertex Position 顶点位置 | Allow the specification of a custom vertex position. Uses current vertex position if left unconnected.允许指定自定义顶点位置。如果未连接，则使用当前顶点位置。 | Float3 浮点数 3 |

[Back to Node List返回节点列表](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Nodes)

[Categories](https://wiki.amplify.pt/index.php?title=Special:Categories): 

- [Nodes](https://wiki.amplify.pt/index.php?title=Category:Nodes&action=edit&redlink=1)
- [Surface Data表面数据](https://wiki.amplify.pt/index.php?title=Category:Surface_Data&action=edit&redlink=1)

[分类](https://wiki.amplify.pt/index.php?title=Special:Categories)： [节点](https://wiki.amplify.pt/index.php?title=Category:Nodes&action=edit&redlink=1)

### 相关节点





#### Texture Coordinate

Texture Coordinate 节点*（快捷键：U 键）*使用网格 UV，并根据纹理平铺和偏移参数对其进行操作，以定义纹理映射到 3D 资产的方式。如果 [Tex 端口](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#inputtex)连接到[纹理对象](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Object)，或者在节点属性面板中引用了纹理对象，则它们将由材质检查器中的该纹理字段进行转换。否则，将使用节点的参数或输入转换坐标。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\TextureCoordinates.jpg)
使用的节点：[Texture Object](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Object)、Texture Coordinates、[Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample)

| 节点参数 |                             描述                             |  默认值  |
| :------: | :----------------------------------------------------------: | :------: |
|   参考   | 指向现有 [Texture Object](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Object) 或 [Texture Sample](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Sample) 节点**没有：**让我们完全关闭引用**选项 #：**所有其他选项都是动态生成的，并指向图表中所有可用的纹理节点 |   没有   |
| 坐标大小 | 主输出端口的大小，允许您从顶点坐标读取更多打包数据。相应地动态更改输出端口。**浮点数[2,4]：**将大小从 Float2 更改为 Float4 |  float2  |
|  UV 集   | 使用的 UV 通道，在某些应用中也称为 UV Index。Set 2 通常用于光照贴图 UV 坐标。**[1,8]：**将设置为指定数字的次数从集合 1 更改为集合 8。请注意，Unity 2018.1 及更低版本仅允许 1 到 4。 |    1     |
| 其他参数 |                                                              |          |
| tilling  | 用作 UV 的恒定乘数，使它们的大小发生变化，从而有效地创建**平铺**图案。仅当 [Reference](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramreference) 设置为 **None** 或未连接等效输入端口时可用。 | ( 1, 1 ) |
|  offset  | 向 UV 添加常量值，使它们沿所需方向移动，从而有效地从原始位置**偏移**。仅当 [Reference](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramreference) 设置为 **None** 或未连接等效输入端口时可用。 | ( 0, 0 ) |



| 输入端口 |                             描述                             |   类型    |
| :------: | :----------------------------------------------------------: | :-------: |
|   tex    | 此端口接受允许使用其纹理 UV 参数的 [Texture Object](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Object)。覆盖节点平铺和偏移，并改用材质检视器。 | Sampler2D |
| tilling  | 参数 [Tilling](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramtilling) 的动态版本。仅当 [Reference](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramreference) 设置为 **None** 时可用，否则将被锁定以指示使用材质检查器中纹理属性的平铺。 |   float   |
|  offset  | 参数 [Offset](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramoffset) 的动态版本。仅当 [Reference](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramreference) 设置为 **None** 时可用，否则将被锁定以指示使用材质检查器中纹理属性的 Offset。 |   float   |

|   输出端口    |                             描述                             |     类型      |
| :-----------: | :----------------------------------------------------------: | :-----------: |
| 紫外线 （WT） | 返回向量 2、向量 3 或向量 4，向量 2 分别包含 U 和 V 坐标，向量 3 为 UVW 坐标，向量 4 为 UVWT | 浮点数2（34） |
|       U       |                   返回包含 U 坐标的浮点数                    |     float     |
|       V       |                   返回包含 V 坐标的浮点数                    |     float     |
|       W       | 返回包含 W 坐标的浮点数。仅当 [Coord Size](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramcoordsize) 设置为 Float3 时可用。 |     float     |
|       T       | 返回包含 T 坐标的浮点数。仅当 [Coord Size](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Texture_Coordinates#paramcoordsize) 设置为 Float4 时可用。 |     float     |

**tilling的使用，通过添加常量值，让其平铺**

![image-20241020200018638](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020200018638.png)

#### 总结点参数介绍

##### **标准 Shader 输入**

标准表面输出本部分的部分内容直接来自 Unity 材质参数参考文档。了解更多：[Unity 材质参数参考](https://docs.unity3d.com/Manual/StandardShaderMaterialParameters.html)标准着色器为您提供材质参数列表。这些参数会略有不同，具体取决于您选择是在 Metallic 工作流程模式还是 Specular 工作流程模式下工作。大多数参数在两种模式下都是相同的，本页将介绍所有这些参数。

 ![OutputNode2.jpg](https://wiki.amplify.pt/images/Nodes/OutputNode2.jpg)



- 反照率：反照率参数控制着色器表面的底色，接受颜色值或纹理贴图。
- 法线（Normal）：法线贴图是一种特殊的纹理，允许添加表面细节，例如从高多边形网格、凹凸、凹槽和划痕传输的着色信息。您可以在此处连接法线贴图或自定义法线向量。
- Emission（发射）：Emission （发射） 控制从表面发射的光的颜色和强度，而不管照明条件如何;接受完整的 RGB 值。
- 金属感（仅限金属感工作流程）：在“金属感”工作流程中工作时，表面的反射率和光响应会同时受到“金属感”和“平滑度”级别的修改;两个灰度输入。您可以使用从 0 到 1 的值范围，影响整个表面，甚至连接一个纹理，这将控制 Metallic 值，同时为表面的不同区域提供不同的值;0 是电介质（非金属），1 是全金属。
- 镜面反射（仅限镜面反射工作流程）：在镜面反射模式下工作时，镜面反射参数中的 RGB 颜色控制镜面反射率的强度和色调。
- 平滑度（两个工作流程）：平滑度的概念适用于镜面反射和金属感工作流程，并且在两者中的工作方式非常相似;也是灰度。值为 1 的完全平滑表面提供清晰的反射，而设置为 0 的粗糙表面则创建没有清晰反射的漫反射颜色;在某些引擎中，它也被称为“粗糙度”。
- Ambient Occlusion（环境光遮蔽）：Occlusion （遮挡） 输入接受遮挡纹理贴图或自定义值，该值用于提供有关模型哪些区域应显示高或低间接照明（来自环境照明和反射）的信息。遮挡贴图是一个灰度图像，其中白色表示应接收完全间接光照的区域，而黑色表示没有间接光照。
- 透射（Transmission）：透射（Transmission） 是一种高度优化的方式，用于近似光散射。换句话说，它定义了从后面照亮时有多少光线穿过表面，这非常适合细节较少的资源，如树叶、布料甚至蜡质对象;接受全 RGB 输入。
- 半透明（Translucency）：半透明（Translucency） 输入允许您通过控制光线法线角度衰减偏移，以简单和优化的方式近似 SSS（次表面散射）效果。它通常用于皮肤效果，但对于其他用途也足够灵活;接受全 RGB 输入。
- 折射 （ 透明渲染类型 ）：折射输入需要一个 Render Type 设置为 Transparent 的着色器，用于模拟通过玻璃或水等介质看到的对象的变形效果，这是通过折射背景像素的屏幕空间 UV 偏移来实现的;接受全 RGB 输入。这种技术是对光现象的简单近似，当波从具有给定折射率的介质传播到另一种介质以倾斜角度传播时，通常会发生光现象。
- 不透明度（透明渲染类型）：不透明度输入需要一个着色器，其渲染类型设置为透明，负责设置整个表面的透明度，使用 0 到 1 之间的值范围，分别从完全透明到完全不透明;接受全 RGB 输入。
- 不透明蒙版（透明渲染类型）：不透明蒙版需要其渲染类型设置为透明或蒙版混合模式的着色器，其工作方式类似于不透明度，因为它采用 0 到 1 之间的值，从完全透明到完全不透明，但不考虑介于两者之间的值，从而在特定区域中产生完全可见或完全不可见的表面。对于定义复杂实体表面（如金属丝网或链节）的材质，它是完美的解决方案，因为不透明部分仍将考虑照明;接受灰度输入。
- Local Vertex Offset （ Relative Vertex Output） （局部顶点偏移量） （Local Vertex Offset） ） ：Local Vertex Offset （局部顶点偏移量） 输入可用于通过顶点操作来改变表面的形状，其中 XYZ 坐标将定义每个顶点如何从其相对位置偏移。
- Local Vertex Position （ Absolute Vertex Output） （ 局部顶点位置（Local Vertex Position） （局部顶点位置）） 的工作原理类似于 Local Vertex Offset （局部顶点偏移） 输入，但是，它不会从每个顶点的相对位置偏移每个顶点，而是在绝对世界空间方向上偏移顶点。
- Local Vertex Normal：Local Vertex Normal 允许调整任何偏移表面的法线方向，因为网格法线不是实时计算的。这个过程通常被称为正常重建。
- Tessellation：Tessellation 输入允许细分网格的三角形，在运行时将它们分割成更小的三角形，以增加任何给定网格的表面细节。
- Debug：Debug 输入会生成一个预览着色器，该着色器会覆盖所有其他活动输入，仅绘制插入到其输入端口的内容。请注意，并非所有节点或特定组合都可以在调试模式下预览。

#### 如果节点

If 节点比较两个浮点输入 [A](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/If#paramA) 和 [B](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/If#paramB)，并根据比较结果从其其中一个输入 [A > B](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/If#paramA>B) 、[A == B](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/If#paramA==B) 或 [A < B]上输出一个值。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\IfNode.jpg)
使用的节点：[World Position](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/World_Position)、[Float](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Float)、[World Normal](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/World_Normal)、[Vertex TexCoord](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Vertex_TexCoord)、If

| 节点参数 |                         描述                         | 默认值 |
| :------: | :--------------------------------------------------: | :----: |
|    A     | 比较运算的第一个值。仅当未连接相应的输入端口时可见。 |   0    |
|    B     | 比较运算的第二个值。仅当未连接相应的输入端口时可见。 |   0    |
| 动态分支 |  切换以让编译器知道此条件操作应该在实际分支上编译。  |   假   |



| 输入端口 |                 描述                 |                             类型                             |
| :------: | :----------------------------------: | :----------------------------------------------------------: |
|    A     |         比较运算的第一个值。         |                              浮                              |
|    B     |         比较运算的第二个值。         |                              浮                              |
|  A > B   | 如果 A 的值大于 B 的值，则输出的值。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/If#anchor) |
|  A == B  | 如果 A 的值等于 B 的值，则输出的值。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/If#anchor) |
|  A < B   | 如果 A 的值小于 B 的值，则输出的值。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/If#anchor) |

debug switch

Debug Switch 根据当前选定的[选项](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Debug_Switch#paramCurrent)输出其输入之一。该值以后无法在构建时更改，因为仅分析当前选定的 switch option input port 并生成其源代码。
**注意：**这是一个 Debug 节点，它只为所选端口生成源。这意味着不会为其他端口生成任何属性，并且可能会丢失信息。

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\DebugSwitchNode.jpg)
使用的节点：[Color （颜色](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Color)）、Debug Switch （调试开关）

|  节点参数  |                             描述                             | 默认值 |
| :--------: | :----------------------------------------------------------: | :----: |
| Max Amount | 最大开关选项数量。确定可用输入端口的数量（ 可能介于 2 和 8 之间）。 |   2    |
|  Current   |                     当前选定的交换机选项                     |  在 0  |



| 输入端口 |                        描述                        |                             类型                             |
| :------: | :------------------------------------------------: | :----------------------------------------------------------: |
|   In #   | 如果 # 设置为 toggle value，则要在输出上设置的值。 | 浮点型 [[1\]](https://wiki.amplify.pt/index.php?title=Unity_Products:Amplify_Shader_Editor/Debug_Switch#anchor) |

## 快速开始

### 基于urp的渲染管线

**第一步将项目升级为urp**

然后创建对应的shader(也可以选择surface)

![image-20241018233226583](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018233226583.png)

![image-20241018233457754](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018233457754.png)

**第二步创建材质，并将shader赋给材质**

![image-20241018233405854](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018233405854.png)

![image-20241018233442994](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018233442994.png)

**第三步**

双击材质进入shader编辑界面

基本组件的介绍

一个shader中对应的属性

![image-20241018234002144](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018234002144.png)

**baseColor**：基础色

WorkflowMode ：选择适合[纹理](https://so.csdn.net/so/search?q=纹理&spm=1001.2101.3001.7020)的工作流。选择金属或镜面反射。

__surface :控制的是物体的渲染模式，简单理解就是设定物体是透明的渲染模式还是不透明的渲染模式

Render Face: 控制物体的渲染方式，向前、向后、或者双面。

Albedo：其实就是材质的基础固有色，你可以给它一张纹理贴图也可以选择单色，当你选择纹理贴图的时候这个调色板则为纹理之上的叠加色

**Metallic & Smoothness：**这里则是用来控制材质的**金属度和光滑度**，金属度只是决定了物体最终渲染的结果是以高光为主，还是漫反射为主。

Source：可以理解为就是选择光滑贴图，它的数据可以来源于金属贴图，也可以是色彩贴图

**NormalMap：**法线贴图，它属于凹凸贴图的一种，通常**用来给模型增加一些微小的凸起、凹槽和划痕等细节**，在逐像素计算光照时，每个像素都会根据该点的法向量来计算最终该点的光照结果，可以通过法线贴图，改变这个点法线方向，影响它的光照结果，进而影响模型表面凹凸感。

HeightMap：视差贴图，又可以称之为高度贴图，**是法线贴图的改进版**，属于一个经常被忽略的高级功能。大家都知道法线，可以将一个平面做成凹凸不平的效果，但是当视角方向水平于该平面的时候，理论上凸起的部分会遮挡住后面的部分，而法线贴图却没有这个效果，但是高度贴图，就可以。那原理呢，就是根据该点的高度以及该点指向摄像机的向量，计算出一个UV偏移，来影响之后的采样。

**Occlusion**：**环境光遮蔽贴图**，通常又会叫**AO贴图**，在PBR中计算光照的时候，一般直接通过采样IBL来得到环境光，这个环境光是该点上一个半球上的积分。但是因为自身的之间会有凹凸，在凹陷的地方，环境光会被周围给遮挡，所以看起来并不是那么亮，通过AO贴图我们可以让调整环境光的大小，从而达到更真实的效果。**（也就是让物体表面没有接收到光照的地方更加的暗，更加有层次感）**

> ## Occlusion Map
>
> ![在这里插入图片描述](D:\DeskTop\FileSum\UnityClassResources\笔记\d720f9440cef9348c9252ec0a07a7ff1.png)
>
> 遮挡贴图用于提升模型间接光影效果。间接光源可能来自Ambient Lighting（环境光），因此模型中凹凸很明显的凹下去那部分，如裂缝或褶皱，实际上不会接收到太多的间接光。
>
> 遮挡纹理贴图通常有3D建模软件生成。
>
> 一张遮挡贴图应该就是张灰度图，白色代表应该接收到的间接光效果会多一些，黑色代表少一些（全黑说明一点间接光都不接收）。
>
> 另外，生成遮挡纹理稍微更复杂一些。例如，在场景中有个角色戴了一帽子，在帽子和角色的头之间会有些边缘是几乎接收不到间接光的。在这种情况，这种遮挡贴图通常是由美术同学使用3D软件根据模型自动生成的。
> ![This occlusion map identifies areas on a character’s sleeve that are exposed or hidden from ambient lighting. It is used on the model pictured below.](D:\DeskTop\FileSum\UnityClassResources\笔记\e6dd6d5b0c3ab86e335394440d0c86d9.jpeg)
> *遮挡贴图定义了角色的头戴的披肩附近的部分，哪些相对Ambient lighting（环境光）是曝光，哪些是挡光的。如下图显示。*
> ![Before and after applying an occlusion map. The areas that are partially obscured, particularly in the folds of fabric around the neck, are lit too brightly on the left. After the ambient occlusion map is assigned, these areas are no longer lit by the green ambient light from the surrounding wooded environment.](D:\DeskTop\FileSum\UnityClassResources\笔记\5be4491d6865411091359a8efcd4c65b.jpeg)
> *在应用遮挡贴图前后的效果图（左边没使用，右边使用了）。部分模型的区域被遮挡了，特别是在脖子周围的头戴饰品遮挡的那部分，左边的图片显示太亮了，右边的图片设置了环境光遮挡贴图后的效果，脖子那些绿色的、来自环境草木的环境光就没那么亮了。*

**Emission：**材质的自发光属性，通常用于控制从表面发出的光的颜色和强度，用的比较多的地方像如霓虹灯、LED屏幕等等。

**Tiling & Offset：**控制的则是以上所有贴图的Tiling Offset

**主材质**：用于显示基本的图形

![image-20241018235055563](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235055563.png)

![image-20241018234926854](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018234926854.png)

![image-20241018235322215](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235322215.png)

**textureSample**:用于将主纹理转换成

![image-20241018235142026](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235142026.png)

![image-20241018235248914](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235248914.png)

color:用于与图像混合

![image-20241018235436741](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235436741.png)

multiply：用于两个数之间相乘

![image-20241018235530922](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235530922.png)

vector4：四维的一个数

![image-20241018235657282](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235657282.png)

append：用于将两个数合并成一个（底层原理是什么）

![image-20241018235728517](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235728517.png)

Textrue Coordinates:用于控制uv,什么是uv

![image-20241018235829076](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235829076.png)

float:一个数，通过滑动调节

![image-20241018235928420](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241018235928420.png)

ToggleSwitch:开关，bool值

![image-20241019000052307](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019000052307.png)

lerp:平滑插值，从a到b 转换，每次增加总体的alpha

![image-20241019000238607](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019000238607.png)

基本框架的搭建

第一步：用于主要图像的显示，将图像与颜色进行混合

![image-20241019161231334](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019161231334.png)

并将其赋给baseColor

![image-20241019161552301](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019161552301.png)

生成的界面如下

![image-20241019161939312](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019161939312.png)

第二步法线贴图

**其中的滑动条用于调节贴图的强度**

![image-20241019162048356](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019162048356.png)

生成的界面如下所示

![image-20241019162155776](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019162155776.png)

第三步：

**用于控制是否自发光**

![image-20241019162554975](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019162554975.png)

生成的界面

也就是两者选其一

![image-20241019162810005](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019162810005.png)

第四步：物体的金属度

如果没有给贴图的话，那么就需要自己添加一张贴图

![image-20241019162913142](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019162913142.png)

第五步:用于实现光滑度

![image-20241019163056870](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019163056870.png)

生成的界面

![image-20241019163426923](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019163426923.png)

第六步：实现OA贴图，也就是环境光遮罩

颜色设置成白色，相当于没有

![image-20241019163549949](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019163549949.png)

![image-20241019163645281](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019163645281.png)

第七步：

设置uv的偏移，对所有的uv进行统一的管理

![image-20241019163832808](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019163832808.png)

![image-20241019163848616](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019163848616.png)

对于法线贴图的uv设置，需要单独控制

![image-20241019163924537](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241019163924537.png)

总结：

1、对于两种颜色进行叠加的，都会使用multiple进行叠加

2、对于颜色的初始值，一般设置成白色

## 案例

### 实现遮罩功能

首先添加两张textrue

![image-20241020210444102](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020210444102.png)

打开锁头，保证计算的和实际显示的名字是不一样的

![image-20241020210600976](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020210600976.png)

接下来实现图片的移动，通过uv控制移动

![image-20241020210723546](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020210723546.png)

最终整体如下

![image-20241020210850646](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020210850646.png)

关闭深度写入，那么关闭和显示有什么区别呢

在图形渲染中，写入和不写入深度缓冲区的区别主要体现在渲染对象如何与场景中的其他对象进行深度比较。  

1. 写入深度缓冲区：当几何图形写入深度缓冲区时，它的深度值会被记录，渲染引擎可以根据深度值判断哪些物体在前面，哪些物体在后面。这有助于实现遮挡效果，确保只显示可见的部分。

2. 不写入深度缓冲区：如果几何图形不写入深度缓冲区，那么它们的深度值不会被记录，这意味着渲染引擎不会考虑这些物体的深度关系。这通常用于一些特效或透明度效果，允许底下的物体继续被显示，甚至创建特定的视觉效果。

总结来说，写入会影响场景的深度管理和渲染结果，不写入则会导致物体的遮挡关系被忽略，可能用于实现特殊效果。

![image-20241020211344793](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020211344793.png)

![image-20241020211046865](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020211046865.png)

修改渲染模式

![image-20241020211959784](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241020211959784.png)



解释这几个有什么区别，分别在什么时候使用

![img](D:\DeskTop\FileSum\UnityClassResources\笔记\9be69afca3684cab970bf196a6bd4549-1729431514.png)

在图形渲染中，混合模式用于控制如何将一个图像（源）与另一个图像（目标）结合。以下是你提到的几种混合模式的区别及其适用场景：

1. **<OFF>**：关闭混合，源图像将直接覆盖目标图像，适用于需要完全不透明的效果。

2. **Custom**：自定义混合模式，允许用户根据需要设置特定的混合公式，适用于特殊效果。

3. **Alpha Blend**：基于源图像的 alpha 通道进行混合，适用于需要透明度效果的场景，如半透明物体。

4. **Premultiplied**：预乘 alpha 混合，源颜色在计算前已与 alpha 值相乘，适用于需要更高质量的透明效果，尤其是在处理光照和阴影时。

5. **Additive**：加法混合，源颜色与目标颜色相加，适用于发光效果，如火焰、光线等。

6. **Soft Additive**：软加法混合，类似于加法混合，但在边缘处有更柔和的过渡，适用于需要柔和光晕效果的场景。

7. **Multiplicative**：乘法混合，源颜色与目标颜色相乘，适用于阴影和暗色效果，通常用于创建更深的颜色。

8. **2x Multiplicative**：双倍乘法混合，源颜色与目标颜色相乘后再乘以2，适用于需要更强烈阴影效果的场景。

9. **Particle Additive**：粒子加法混合，专为粒子系统设计，适用于粒子效果，如烟雾、火花等。

选择合适的混合模式取决于你想要实现的视觉效果和场景需求。

### 实现溶解效果

添加一个unlit的shader

需要调节：

关闭深度写入

增加tags

![image-20241021104801020](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021104801020.png)

> 原理解释
>
> 在Amplify Shader Editor中，您提到的属性主要用于设置着色器的渲染管线和渲染队列。以下是这两个属性的具体作用和使用方法：
>
> ### 1. RenderPipeline（渲染管线）
>
> - **作用**：这个属性用于指定着色器所使用的渲染管线。在您的示例中，值为`UniversalPipeline`，表示该着色器将使用Unity的通用渲染管线（Universal Render Pipeline, URP）。
> - **使用方法**：在创建或编辑着色器时，确保选择与项目中使用的渲染管线相匹配的值。如果您使用的是URP，就保持这个设置。
>
> ### 2. Queue（渲染队列）
>
> - **作用**：这个属性用于定义渲染队列的类型。在您的示例中，队列设置为`Transparent`，这意味着该着色器将用于透明物体的渲染。渲染队列决定了物体的渲染顺序，透明物体通常需要在不透明物体之后渲染。
> - **使用方法**：根据需要渲染的物体类型选择合适的队列。对于不透明物体，可以选择`Opaque`；对于透明物体，选择`Transparent`。您还可以通过`Index`字段进一步自定义渲染顺序。
>
> ### 总结
>
> 在Amplify Shader Editor中，正确设置这些属性可以确保您的着色器在Unity中按预期工作。确保渲染管线和渲染队列与您的项目需求相匹配，以获得最佳效果。
>
> 渲染队列（Render Queue）在图形渲染中起着至关重要的作用，主要用于控制物体的渲染顺序。以下是渲染队列的具体作用和一些相关概念：
>
> ### 渲染队列的作用
>
> 1. **控制渲染顺序**：
>    - 渲染队列决定了游戏对象在屏幕上的渲染顺序。物体是按照队列的顺序进行渲染的，这对于正确定义物体间的显示顺序至关重要，例如，前景物体须在背景物体之前渲染。
>
> 2. **处理透明度**：
>    - 透明物体的渲染需要在不透明物体之后进行。这是因为透明物体可能会影响后面渲染的物体的像素。如果透明物体在不透明物体之前渲染，可能会导致视觉错误。因此，透明物体通常被分配到“Transparent”队列。
>
> 3. **优化渲染性能**：
>    - **通过合理配置渲染队列，可以帮助合并绘制调用（draw calls），减少GPU的负担，提高渲染性能。**
>
> ### 渲染队列的类型
>
> 在Unity中，常见的渲染队列包括：
>
> - **Opaque（不透明）**：通常默认为2000。适用于不透明物体的渲染。
> - **Transparent（透明）**：通常默认为3000。用于透明物体的渲染。
> - **Overlay（覆盖）**：用于GUI或其他层叠效果的渲染，通常是更高的值，比如4000。
>
> ### 小结
>
> 设置合适的渲染队列可以确保正确的视觉效果和渲染顺序，从而使画面呈现更为自然和美观。在设定Shader属性时，注意选择合适的渲染队列，这样可以避免由于渲染顺序错误导致的视觉问题。

选择Blend RGB为alpha blend

![image-20241021105030770](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021105030770.png)

首先了解溶解原理，也就是通过逐步去降低透明度，并且不是统一的关闭，而是需要一个透明度不统一的图片作为参考，结合step函数进行对比，逐步去设置透明度

首先添加两张图片

![image-20241021105231735](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021105231735.png)

其次添加

因为我们需要重新设置透明通道，所以需要展示将原先主纹理的透明通道分离出来

通过下面的组件

![image-20241021105436270](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021105436270.png)

![image-20241021105548505](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021105548505.png)

然后通过下面的函数进行比较，实现溶解的关键之处（黑色是0，白色为1）

![image-20241021105632950](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021105632950.png)

![image-20241021105906133](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021105906133.png)

![image-20241021105922460](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021105922460.png)

与原先的A通道相乘，也就是叠加

![image-20241021110122329](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021110122329.png)

最后输出

![image-20241021110145920](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021110145920.png)

对比build in 渲染管线

![image-20241021110747396](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241021110747396.png)

### 实现光边溶解的效果（纸片）燃烧

效果展示

![image-20241110154727250](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110154727250.png)

参数设置

![image-20241110154808026](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110154808026.png)

![image-20241110154825813](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110154825813.png)

![image-20241110154848027](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110154848027.png)

首先设置一张图片

![image-20241110155006890](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110155006890.png)

同上设置透明度

![image-20241110155135291](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110155135291.png)

为了实现那个边缘的效果，可以略微增加一点透明度数值，当然后面还需要和原图相减，得到边缘，然后同样设置显示隐藏的效果

![image-20241110155406007](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110155406007.png)

![image-20241110155628806](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110155628806.png)

得到边缘还需要进一步对边缘进行颜色的改变

![image-20241110155723173](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110155723173.png)

然后对两张图片进行混合

![image-20241110161330545](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110161330545.png)

下面这一步我不太懂

但是这里必须设置那个添加了的，因为我们的光边必须要在大的那一边显示，结合上述可以知道

![image-20241110161700201](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110161700201.png)

经过测试发现直接设置透明度结果也是一样的

![image-20241110162047545](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110162047545.png)

最终的结果

![image-20241110161754414](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110161754414.png)

内置渲染管线

![image-20241110165605661](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110165605661.png)

注意如果要效果更加明显，需要添加后处理效果

![image-20241117164836169](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241117164836169.png)

### UV扰动（如）水波纹

参数设置

![image-20241110164918975](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110164918975.png)

![image-20241110164937399](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110164937399.png)

![image-20241110164956224](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110164956224.png)

首先添加一张图片

![image-20241110165048292](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110165048292.png)

其次添加一张UV扰动的节点，同时设置两张参数，用于防止uv强度过大导致的图像偏移

![image-20241110165122091](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110165122091.png)

设置UV扰动效果

![image-20241110165339310](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110165339310.png)

最后进行混合

![image-20241110165408413](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110165408413.png)

内置渲染管线如下

![image-20241110165503164](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241110165503164.png)

### 实现边缘光

首先参数配置

![image-20241113211546910](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113211546910.png)

![image-20241113211617970](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113211617970.png)

![image-20241113211625928](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113211625928.png)

#### 第一种方式和：frenel

第一步添加图片

![image-20241113212010146](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113212010146.png)

第二步添加菲涅尔节点（后续解释），用于实现边缘光

![image-20241113212051538](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113212051538.png)

相乘得到如下结果

![image-20241113212125116](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113212125116.png)

添加一个颜色，并且将颜色设置HDR

![image-20241113212201819](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113212201819.png)

因为直接使用frenel,会出现闪点等情况，所以我们可以自己写一个，通过Saturate进行过滤，限制在0-1之间

#### 第二种方式，自己做一个frenel

第一步，摄像机方向向量和球表面的法线向量平行，所以相乘为1，对于球边缘的法线和摄像机方向向量垂直，所以结果为0

![image-20241113212339155](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113212339155.png)

通过1减去一个值，中间为1，所以中间空了，两边0，所以1-0仍然为1

![image-20241113212753215](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113212753215.png)

因为中间有很多模糊的点，那么我们为了让其边缘更加清晰，就需要将这些点过滤掉，其实就是把他变小即可

那么就可以exp实现，将那些小于0的平方，让他们更小

![image-20241113212906105](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113212906105.png)

因为进行指数处理，有些值可能会特别大，那么就可以通过saturate，将值固定在0-1之间

![image-20241113213012286](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113213012286.png)

最终的效果都是一样的

![image-20241113213039208](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241113213039208.png)

### 实现塞尔达火焰效果

#### 原理：

通过三个波动波将对图片进行处理，实现火焰效果

#### 参数的设置

![image-20241116154947282](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116154947282.png)

![image-20241116155011535](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116155011535.png)

![image-20241116155022246](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116155022246.png)

![image-20241116155040572](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116155040572.png)

![image-20241116155048756](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116155048756.png)

添加一张图片，通过一个Intensity调节图片强度

![image-20241116155317113](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116155317113.png)

通过三个波叠加图片，实现火焰效果

![image-20241116155759928](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116155759928.png)

第一个波（高密的）

通过一个密度高的波动加上一个密度小的波动，从而实现更加真实的火焰效果

其中的angle，表示旋转值

![image-20241116160128781](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116160128781.png)

第二个波动

![image-20241116160401669](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116160401669.png)

当然这两个波还是不够，还需要个更密的，否则火源会很分散

![image-20241116160729407](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116160729407.png)

到这一步，已经完成

最终的效果为

![image-20241116160804865](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241116160804865.png)

在URP中如何设置参数

![image-20241117164551556](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241117164551556.png)

### 什么是变体

如何查看，最优show的左边的值

![image-20241117165207460](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241117165207460.png)

什么是变体呢？

就是当你开启了很多功能的时候，那么就会出现这个功能的多种搭配，这种不同的搭配也就是所谓的变体

变体越多消耗的性能越大

其中ase的unlit变体的魔板要少，性能更优

![image-20241117165339622](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241117165339622.png)

### 深度渐变

参数设置

![image-20241119211252620](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119211252620.png)

![image-20241119211443709](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119211443709.png)

![image-20241119211508385](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119211508385.png)

![image-20241119211543288](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119211543288.png)

添加一个深度渐变

其中distance:用于调整渐变的一个范围

VertexPos:一般不做调整

与物体结束的地方为1，否则为0

![image-20241119213246163](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119213246163.png)

接下来反转，并且将值限制到0-1之间

![image-20241119213433952](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119213433952.png)

添加一个颜色（开启HDR）

![image-20241119213450275](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119213450275.png)

为了更好看，所以添加一个贴图

![image-20241119213519157](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119213519157.png)

但是目前还有一个问题 ，就是看放大物体的时候，物体上面的uv图形会变大变小，以为目前是模型UV,所以我们需要将他转换成屏幕UV

![image-20241119213618509](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119213618509.png)

vector用于模拟屏幕的大小

最终的一个效果

![image-20241119213704493](D:\DeskTop\FileSum\UnityClassResources\笔记\image-20241119213704493.png)

应用：

一个实现蚕食的效果，还有就是可以实现一些边缘 效果，比如实现一些水与海岸的一个浪花效果

