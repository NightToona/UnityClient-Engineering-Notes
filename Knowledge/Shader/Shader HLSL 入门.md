题目教程来源网站：shader-learning.com


# Building-in functions 【基础构建方法】

## （一）API-函数部分
## T-31 【Atan2使用】

> **原题目：**
> Create a gradient that transitions from red to green. The weight of each pixel should be determined by the angle between the center-to-pixel ray and the x-axis. Limit the gradient to a circular area with a diameter equal to the height of the texture. Use `3.14` as the approximate value for π.

该题主要以`atan2` / `atan`函数为主，由前一题T-30转变提高而来（*第30题为“宽/高比”，用于校准原以上下两边为相对坐标1的坐标，将其拉伸缩放至X-Y轴长度相等*）。

通过先确定实际圆的缩放后坐标，限定圆的范围（相当于做了个圆的Mask）。随后将旋转变色单独提取出来成为独立单元，最后相乘取得实际图案的渲染变化。以下为代码内容：

``` HLSL
cbuffer Uniforms : register(b0)
{
  float2 iResolution;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;

  //求坐标
  float2 coord = uv * 2.0 - 1.0;
  float ratio = iResolution.x / iResolution.y;
  coord.x *= ratio;

  //划定圆范围
  float dis = distance(coord, float2(0.0, 0.0));
  float mask = step(dis, 1.0);

  //上色
  float angle = abs(atan2(coord.y, coord.x));
  float PI = 3.14;
  float t = angle / PI;
  
  return float4((1.0 - t) * mask, t * mask, 0.0, 1.0);
}
```

**结果与正确答案参考图：**

![[Pasted image 20260331095208.png]]

## T-32【Sin使用】

> **原题目：**
> Draw an image with `5` sinusoidal waves coming out from the center of the screen. The waves should be based on the distance from each pixel to the center. Adjust the sine values to a range from `0` to `1`, and use these values to control the brightness of the red color. This will create a repeating pattern of peaks and troughs of the waves. Use `3.14` as the approximate value for π.

该题是以使用sin正弦波动的逻辑，实现红色与黑色两色之间的过渡。该题和上一题T-30含有相同的点，都是将原点转化为以图中心为坐标轴，并进行X轴缩放（*第30题为“宽/高比”，用于校准原以上下两边为相对坐标1的坐标，将其拉伸缩放至X-Y轴长度相等*）。

实际难点在于将周期渐变应用在极坐标轴上，所以应该将半径从`uv.x`改为求`dist`，符合极坐标的原理。同时还要将`sin()`的周期从`[-1, 1]`转变为`[0, 1]`。（加1除以2就可变换得到）以下为实现代码：

```HLSL
cbuffer Uniforms : register(b0)
{
  float2 iResolution;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;
  float PI = 3.14;
  float ratio = iResolution.x / iResolution.y;

  // 坐标轴转换与缩放比
  uv = uv * 2.0 - 1.0;
  uv.x *= ratio;
  float dis = distance(uv, float2(0.0, 0.0));

  float frequency = 5.0;
  float angle = dis * frequency * PI;
  float r = (sin(angle) + 1.0) / 2.0;
  
  return float4(r, 0.0, 0.0, 1.0);
}
```

**结果与正确答案参考图：**

![[Pasted image 20260403113734.jpg]]

## T-33 【矢量归一化】（题目巨坑）

> **原题目：**
> Write a shader program that divides the screen into two equal parts. The origin of the left part is at (0.25,0.5)(0.25,0.5), and the origin of the right part is at (0.75,0.5)(0.75,0.5). In the left part, display the cosine of the angle between the positive X-axis and the vector directed from the origin to the current pixel position. In the right part, display the sine of the angle between the positive X-axis and the vector directed from the origin to the current pixel position.

这题需要给一下翻译了，因为这里特别特别坑，坑点就在于他的含义。

> **题目译文：**
> 写一个着色器程序，将屏幕分成两相等的部分。左侧部分的原点为(0.25,0.5)，右部分的原点为(0.75,0.5).左侧显示正X轴与从原点到当前像素位置的矢量之间的夹角余弦。右侧显示正X轴与从原点到当前像素位置的矢量之间的正弦。

其实从含义上来说的话其实他的表述在一定程度上并没有错误，只是其中的两个信息从一开始就让我们走错了思考的方向，这两个内容分别是：左侧原点（0.25，0.5）、右侧原点（0.75，0.5）。

问题就出现在这两个点的坐标。当我们看到这两个点的时候自然而然的就会以为这题是以左下角为原点往右0～1的X-Y坐标轴。

所以我们会将这两个点直接用于左半边和右半边的矢量归一化（即：`uv - origin`, `origin`为归一化参考原点）。这来看看这个时候的代码和结果吧：

**⚠️⚠️⚠️请注意此代码为错误答案⚠️⚠️⚠️**
```HLSL
cbuffer Uniforms : register(b0)
{
  float2 iResolution;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;
  float2 origin1 = float2(0.25, 0.5);
  float2 origin2 = float2(0.75, 0.5);
  
  //左半侧
  float2 V1 = uv - origin1;
  float2 N1 = normalize(V1);
  float color1 = N1.x;

  //右半侧
  float2 V2 = uv - origin2;
  float2 N2 = normalize(V2);
  float color2 = N2.y;
  
  float side = step(uv.x, 0.5);
  float result = (color1 * side) + (color2 * (1 - side));
  
  return float4(result, result, result, 1.0);
}
```

**输出的图像是这样的：**

![[IMG_1540.jpeg]]

**所以你发现问题所在了吗？没发现？那就来看看差异度对比吧：（选择的都是 x1 差异下的）**

![[IMG_1543.jpeg]]

对的，问题就出现在从0～90度时的过渡变化速率。这个时候的代码渐变过渡速率是符合正圆的角度渐变的（即渐变的 cos / sin 值不受横向坐标压缩的影响），属于正常渐变，导致左半边速率较慢、右半边的速率较快。

所以再回头看看原句子开头的一句话就会明白，
> **……，将屏幕分成两相等的部分。**

所以这里并不是使用直接对uv进行操作，而是需要将左右两半各分为一个从0到1的渲染范围。本质上就是将原来沾满渲染框的图压缩至一半，才能使得速率和他给出的示例相同。

对于最后的结果代码，那就需要先对uv进行二次操作，见如下的**⚠️正确代码⚠️：**

```HLSL
cbuffer Uniforms : register(b0){……};
struct PSInput{……}

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;
  float2 origin = float2(0.5, 0.5);
  
  //左半侧
  float2 leftUV = uv;
  leftUV.x = uv.x / 0.5;
  
  float2 V1 = leftUV - origin;
  float2 N1 = normalize(V1);
  float color1 = N1.x;

  //右半侧
  float2 rightUV = uv;
  rightUV.x = (uv.x - 0.5) / 0.5;
  
  float2 V2 = rightUV - origin;
  float2 N2 = normalize(V2);
  float color2 = N2.y;
  
  float side = step(uv.x, 0.5);
  float result = (color1 * side) + (color2 * (1 - side));
  
  return float4(result, result, result, 1.0);
}
```


而对应的实际渲染效果那么就和前面给出的题目要求的案例演示相符，此处就不再将图片贴上去了。

## T-34 【点】(找到T-33和本体的巨坑了)

**不用说了，还是出现了同样的问题，当我们按照题目的标准以及心照不宣的代码中给出的归一化UV坐标的时候，我们从这一步就已经走进误区了，包括前面的那题也一样！**

直接先看题目

>  **原题目:**
>  Write a program that draws a triangle in the center of the screen. The triangle should have an apex at `(0.5, 0.75)` in normalized device coordinates, and an apex angle of `120`degrees. The height of the triangle should be `0.5` times the height of the screen.
>  **Hint**
>  First, we can shoot a ray **h** from the apex perpendicular to the base of the triangle. This **h** vector divides the top corner of the triange into two 60-degree parts. 
>  ![](https://whale-app-toyuq.ondigitalocean.app/shader-learning-api/files/image/dot-task.png)
>  If we take the dot product of the height vector **h** and the vector from apex to the triangle bottom vertex, we get a value equal to the cosine of 60 degrees. 
>  Think what happens to the value of the dot product when the fragment fits into a triangle and when it doesn't.

题目啥意思呢？大概就是这样：在屏幕中央画一个三角形。三角形在归一化设备坐标中应有顶点`(0.5, 0.75)`，顶点角为`120`度。三角形的高度应该`0.5`乘以屏幕高度。

要是按照正常逻辑的话，那就是分为两个角度限制矢量在三角形内符合题意，此处用到两个`step()`，一个用于判断cos值是否`>= 0.5`，一个用于判断在多少度的情况下，向量长度最长不超过`h / cos`。

写完之后，一运行，诶，好了，出现问题了。自己写的渲染出来发现边的斜率比例题的更大，看下图：

![[IMG_1549.jpeg]]

很明显吧？甚至我还标出来了，一看明显就是斜率不对，绝对是被压缩了，但是一直找不到为啥会错。因为这题比上面那题更隐蔽，只能靠修正宽高比才行。

从两个函数的最根本逻辑讲一下吧。我们已知有函数`dot() / normalize()`两个。但是我们在学习的时候忽略了一个最重要的问题，就是这两个函数的正确使用方法是：在**标准的欧几里得空间**中进行运算才是画出真实需求下的正确图形。

但问题就在于另两个函数了：`iResolution.xy / uv.xy`。为什么要提及这个呢？因为刚开始我们很自然的就觉得uv归一化就只是将实际像素坐标转化为更加便于运算的坐标。

但是，在转化后我们会忘记一个事，那就是在uv化后，此时原先长方形的像素渲染范围，就会被压缩成标准的正方体。这就导致了uv化后的空间不属于**标准的欧几里得空间**，因此计算也会导致出现偏差。

所以最终的解决方案就是将uv标准化，即对x轴进行等比化校准（T-30/31有提及过）。

看看正确代码吧，图片就不放出来了，最后结果就是示例结果：

```HLSL
cbuffer Uniforms : register(b0)
{
  float2 iResolution;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;
  float2 Apex = float2(0.5, 0.75);

  float2 dir = uv - Apex;
  
  //不要使用归一化uv，图会出现微微变形，因为不是欧几里得几何下的标准
  float ratio = iResolution.x / iResolution.y;
  dir.x *= ratio;
  
  float2 ef = normalize(dir);
  float2 down = float2(0.0, -1.0);
  float cosValue = 0.5;

  float angleArea = step(cosValue, dot(ef, down));
  float highArea = step(dir.y, 0.0) * step(-0.5, dir.y);

  float red = angleArea * highArea;
  return float4(red, 0.0, 0.0, 1.0);
}
```

最后要感谢Discord上某位老哥提出来的解决方法：
https://discord.com/channels/1193522220249653350/1193522220769742954/1451593650004955260

## T-35 【钳制 - Clamp】

很巧妙的一题，这一题基本上是只有两种情况，一个就是完全没想法或是做出来是错的，但是另一种就是完全做得出来，就先看看题目吧。

>  **原题目：**
>  Write a program that draws a diagonal line from the bottom left corner of the texture to the top right corner. The line should have a width of `0.2` in normalized coordinates and be colored in `(1.0, 0.3, 0.3)`. Additionally, ensure that the line is limited to values between `0.25` and `0.75` in Y coordinate.

大概翻译一下吧，就是要求你用`clamp()`这个函数，绘制一个先横着再斜向上最后再横着的一根粗线。

为啥说想不到写不出、想出来就一定写得出呢？因为本质上是对`X-Y轴`相互转化映射的一个逻辑。就类似一个动点，通过 X轴 的动点变化，控制 Y轴的波动范围，通过这个方式就可以控制绘制出来一根粗细大小可控的折线。

在刚开始想不到的时候，很多人可能会想着通过“三步走”的方式（即将该线段拆分成三部分，通过`step()`进行判断），但实际运行后的结果就是：在三者交界处的地方无法完整的连接起来，存在误差。所以这就是为啥：想得到就做的出来，想不到就做不对。

好直接我们上代码吧，而且也正是因为代码量所以说很巧妙：

```HLSL
cbuffer Uniforms : register(b0)
{
  float2 iResolution;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;
  float lineWidth = 0.2;
  float3 lineColor = float3(1.0, 0.3, 0.3);

  float yLine = clamp(uv.x, 0.25, 0.75);
  float mask = step(abs(uv.y - yLine), lineWidth / 2.0);

  return float4(lineColor * mask, 1.0);
}

```

对，实际实现的代码就是`yLine`和`mask`这两个计算式作为核心，很简单吧？下面参考图，可以看看：

![[Pasted image 20260420193908.png]]

## （二）贴图、三维及动画(函数混合使用)

## T-36 【Texture - 贴图】

直接上原题

> **原题目：**
> Write a shader program that renders a texture on the screen. The texture is attached to the shader program through . Lets assume the texture has a **fixed aspect ratio of 1:1** (square). If the screen's aspect ratio causes the coordinates to exceed the `iChannel0`[0,1] range, the shader must **repeat** the texture content rather than stretching it or leaving empty space.

题目啥意思呢？就是要你把这个`iChannel0`输入贴图，通过`Texture2D.Smaple(SamplerState, float2)`方法，转化为每次读取出来的`float4`颜色值。然后按照向右延伸的方式，不变形输出。

**该题目共使用到以下函数与方法：`frac(float)`、`坐标轴比例校准`；**
直接来看看不太准确的代码吧（后文会写）：

```HLSL
Texture2D iChannel0 : register(t0);
SamplerState samplerDefault : register(s0);

cbuffer Uniforms : register(b0)
{
  float2 iResolution;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;

  float ratio = iResolution.x / iResolution.y;
  uv.x *= ratio;
  float2 crood = float2(frac(uv.x), uv.y);
  float4 color = iChannel0.Sample(samplerDefault, crood);
  
  return color;
}
```

为什么会说这个代码其实是**不完备**的呢？其实原因在于，我第一次直接使用`.Sample()`输出时发现在Y轴方向上原贴图并没有产生形变，只有在X轴上出现形变。于是第一反应就是只需要修正X轴上的变形即可。

但是问题在于！玩意在边框外还存在着范围或者Y轴在不同坐标下一缩放，就会出现X轴是正常的但Y轴被无限拉高。所以正确的解决方法是直接对修改后的`uv`进行去小数点，而不是多开个`crood`处理。

最后来看看图片吧：

![[Pasted image 20260420203448.png]]

关于原点坐标上的问题，请注意以下内容⚠️

> [!⚠️关于HLSL和GLSL上的差异]
> **🇬🇧英文版：**
> Although both HLSL and GLSL use normalized texture coordinates ranging from`(0.0, 0.0)` to `(1.0, 1.0)`, they interpret the origin point differently:
> 
> - In GLSL (OpenGL), the coordinate `(0.0,0.0)` refers to the bottom-left corner of the texture. 
> - In HLSL (DirectX), the same coordinate refers to the top-left corner.    
>
> This means that sampling the same texture with identical `uv` coordinates may produce vertically flipped results between the two APIs.
> 
> **Common solution**: to avoid visual inconsistencies, developers typically **flip the image** vertically during GPU upload - either by reversing the row order in CPU memory or using a texture loading library that handles this automatically. This ensures that the texture appears correctly regardless of the coordinate system used by the shading language.
> 
> **🇨🇳中文版：**
> 虽然HLSL和GLSL都使用从`(0.0, 0.0)`到`(1.0, 1.0)`的归一化纹理坐标，但它们对原点的解释不同：
> - 在GLSL（OpenGL）中，`(0.0, 0.0)`坐标指的是纹理的左下角。
> - 在HLSL（DirectX）中，左上角也使用相同的坐标。
>
> 这意味着在相同纹理上采样且`uv`坐标相同时，可能会在两个API之间产生垂直翻转的结果。
> 
> **常见的解决方案**是：为了避免视觉不一致，开发者通常在GPU上传时将**图像垂直翻转**——要么在CPU内存中反转行顺序，要么使用自动处理的纹理加载库。这确保无论渲染语言使用何种坐标系，纹理都能正确呈现。

## T-37 【镜像旋转】（6，纯数学题）

这题就纯粹让你思考到底如何将一个线性方向的坐标转化为有分布点指向性的结构，就像下图一样：

```
标准坐标轴：
0->1->2->3->4->5-> ……
目标坐标结构：
0->1<-0->1<-0->1<- ……
```

说实话，这题纯靠推理计算就可以算出来了，但还是写一遍逻辑吧：

```txt
=> *0.5
	0->0.5->1->1.5->2->2.5-> ……
=> frac()
	0->0.5->1 0->0.5->1 0->0.5-> ……
=> -0.5
	-0.5->0->0.5 -0.5->0->0.5 -0.5->0-> ……
=> *2
	-1->0->1 -1->0->1 -1->0-> ……
=> abs()
	1<-0->1 1<-0->1 1<-0-> ……
=> -1
	0<--1->0 0<—-1->0 0<--1-> ……
=> abs()
	0->1<-0->1<-0->1<- ……
```

缩写后这个代码公式就可以写为   `uv = abs(abs(2 * frac(uv * 0.5) - 1.0) - 1.0);`。那接下来就是看题和代码了：

> **原题目：**
> Write a shader program that renders a texture on the screen. The texture is attached to the shader program through `iChannel0`. Lets assume the texture has a **fixed aspect ratio of 1:1** (square).
> **Mirror Wrapping**: when coordinates extend beyond the standard [0,1][0,1] range (due to the screen's aspect ratio), the shader must implement **mirrored repeat** behavior. This means the texture should flip its orientation every time it repeats.

代码内容在这里，超级简单的一行就搞定了：

```HLSL
Texture2D iChannel0 : register(t0);
SamplerState samplerDefault : register(s0);

cbuffer Uniforms : register(b0)
{
  float2 iResolution;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy * 4.0;
  uv *= float2(iResolution.x / iResolution.y, 1.0); 

  uv = abs(abs(2 * frac(uv * 0.5) - 1.0) - 1.0);
  
  return iChannel0.Sample(samplerDefault, uv);
}
```

最后的渲染效果图片：

![[Pasted image 20260421100932.png]]

## T-38 【丢弃 - discard】（知识点）

在 **HLSL**（High-Level Shading Language）中，`discard` 是一个在 **像素着色器（Pixel Shader）** 中使用的特殊语句，用来**丢弃当前片元（pixel/fragment）**，使它不再写入渲染目标（Render Target）或深度缓冲（Depth Buffer）。

**基本语法**
- **没有参数**，直接写 `discard;` 即可。
- 一旦执行，当前像素的后续计算（包括颜色输出、深度写入等）会被中止（一般配合`if()`使用）。
- 常用于实现 **透明裁剪（Alpha Test）**、**抖动透明（Dither Transparency）** 等效果。

**注意事项**
1. **只能在像素着色器中使用**，在顶点或几何着色器中会报错。
2. 被丢弃的像素不会触发深度写入，也不会参与混合（Blend）。
3. 过多使用 `discard` 可能影响性能，因为它会破坏 GPU 的早期深度测试（Early-Z）。
4. 在某些硬件上，`clip()` 可能比 `discard` 更高效，因为它可以与 Early-Z 更好地配合。

## T-40 【精灵动画 - Sprite Animation】

挺有趣的，就是逻辑上面有点考验数学。先看看题吧：

> **原题目：**
> You are provided with a texture containing a sprite sheet with `2` columns and `4` rows of images. Write a shader program that implements animation using the sprite sheet based on time with a frame rate of `10` frames per second.
> The `iTime` uniform provides the current time in seconds.

题目意思就是要你用一张完整的Sprite图（无切割）进行动画制作。这题的精髓在于对精灵图的坐标轴理解。重要的一点就是，给出的精灵图是不能变化的，但是我们可以控制输入的uv状态。

所以我们此时应当将渲染窗口缩小到精灵图单张图的大小。随后再通过取模和除法，定位每一帧的情况下，对应的图片应该在哪，求的的值即为渲染范围的偏移值（即在第几个位置进行渲染）。

主要使用到的基本上也只是前面提到过的`floor()`函数，其它基本上没用上。看代码吧：

```HLSL
Texture2D iChannel0 : register(t0);
SamplerState samplerDefault : register(s0);

cbuffer Uniforms : register(b0)
{
  float2 iResolution;
  float iTime;
};

struct PSInput
{
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;
  float hight = 4.0;
  float width = 2.0;
  float2 cellSize = 1.0 / float2(width, hight);
  uv *= cellSize; //缩小实际窗口大小到单sprite大
  
  float fps = 10.0;
  float frame = floor(iTime * 10.0);
  frame = frame % (hight * width);
  
  float col = frame % width;
  float row = floor(frame / width);

  float2 offset = float2(col, row) * cellSize;
  uv += offset;

  float4 color = iChannel0.Sample(samplerDefault, uv);
  return color;  
}
```

接下来是渲染的图片（因为是动图无法展示，自己gun去网站上看）：

![[Pasted image 20260421144744.png]]

## T-41 （快门）径向遮罩【混合复习】

首先来复习一下之前学过的所有API功能吧，不然太久不写也容易忘记：

| API                   | 参数                          | 功能/性质              | API              | 参数            | 功能/性质            |
| --------------------- | --------------------------- | ------------------ | ---------------- | ------------- | ---------------- |
| abs()                 | (T x)                       | 求绝对值               | floor() / ceil() | (T x)         | 向上/向下取整          |
| min(,) / max(,)       | (T a, T b)                  | 取最小/最大值            | round()          | (T x)         | 四舍五入             |
| clamp(,,)             | (T x, T minVal, T maxVal)   | 钳制（限制x值范围）         | frac()           | (T x)         | 求小数              |
|                       |                             |                    |                  |               |                  |
| lerp(,,)              | (T start, T end, T weight)  | 公式a(1-t)+bt，求过渡或切换 | step(,)          | (T edge, T x) | x大于等于edge等于1，反之0 |
| sin() / cos() / tan() | (T x)                       | 三角函数               | atan2(y,x)       | ——            | 求方向角             |
|                       |                             |                    |                  |               |                  |
| length()              | (T x)                       | 求点的长度              | normalize()      | (T v)         | 归一化（单位化）         |
| distance(,)           | (T p0, T p1)                | 求两点间距离             | dot(,)           | (T v0, T v1)  | 点积               |
|                       |                             |                    |                  |               |                  |
| Texture.Sample(,)     | (SamplerState s, float2 uv) | 图片采样，输出float4      | iTime            | ——            | 时间               |
| iResolution           | ——                          | 实际显示分辨率            |                  |               |                  |
==*注： p 为点坐标，x 为普通参数（未定义限制），v 为向量，a/b 指普通单值*==

来开始看题目吧，这个题并没有并没有什么特别的地方，算是前面所学的内容的综合实现，接下来直接看看题目吧

> **原题目**：
> You are provided with two texture slides. Write a GLSL program that switches textures using a radial shutter animation. Use sine or cosine to loop the animation, the total switching time is 2PI seconds.
> Radial shutter should work with any aspect ratio.
> The uniform provides the current time in seconds.`iTime`

这题主要是要求实现 “快门径向遮罩” + “背景贴图切换” 二者混合实现。
示例里面给出的缩放波动，我直接在这里给出 `sin(iTime + (PI / 2.0)` 。通过这个变化的时间，进行圆形的遮罩缩放，这样就实现了第一个效果。

其次就是对背景贴图切换，由于是根据遮罩缩到最小时切换，因此一张图片要持续 `2PI` 的时间长度才切换，且遮罩缩放的时间偏移 `（PI / 2.0）`也需要使用在这上面。最后可以使用 `step()` 控制 0/1 达到切换图片。

接下来看看实现代码和实现图片吧：

```HLSL
SamplerState samplerDefault : register(s0);
Texture2D iChannel0 : register(t0);
Texture2D iChannel1 : register(t1);

cbuffer Uniforms : register(b0)
{
  float2 iResolution;
  float iTime;
};

struct PSInput
{
  float4 position : SV_Position;
};

#define PI 3.141592653589

float4 main(PSInput input) : SV_Target
{
  float2 uv = input.position.xy / iResolution.xy;

  // 径向快门实现
    //求对角线
  float aspect = iResolution.y / iResolution.x;
  float maxR = length(float2(1, aspect));
  
  float2 coord = (uv - 0.5) * 2.0; //x-y
  coord.y *= iResolution.y / iResolution.x;

  float r = maxR * (sin(iTime + (PI / 2.0)) + 1) / 2.0;
  float dis = distance(coord, float2(0, 0));
  float mask = step(dis, r);

  // 背景切换
  float change = step(0, sin(0.5 * iTime + (PI / 2.0)));
  float4 color1 = iChannel0.Sample(samplerDefault, uv);
  float4 color2 = iChannel1.Sample(samplerDefault, uv);

  float4 mix = (color1 * change) + (color2 * (1 - change));
  mix.rgb *= mask;
  
  return mix;
}
```

（因为是动图无法展示，自己回去网站上去看去……）

![[Pasted image 20260522111723.png]]

==❗️❗️但是❗️❗️==，这里遇到了一个很常见，会搞混的地方，那就是：

> rgb 决定颜色 、 alpha 决定透明度。透明后显示什么，看后面的背景颜色。
> 不是说只要 Alpha = 0 背景就一定是黑色！！！

所以要让遮罩外面成为黑色，那就用 `float4(0.0, 0.0, 0.0, 1.0)` ，这个才是黑色！
⚠️一定要注意，这是个很小很小的坑。

## T-
