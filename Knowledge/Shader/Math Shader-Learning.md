
> [!NOTE] 引言
> 该部分主要以HLSL中的数学部分为核心，通过常见的数学方法来解决视觉问题

# Math

## T-1 软圆形

在 HLSL 中最简单的画圆方法就是通过判断像素点是否在圆的半径之内，从而控制遮罩渲染。

采用的是`1 - step(r, dist)` 的方法渲染。

但是这里有个问题，渲染出来的圆为：硬边圆，就是渲染出来的圆会携带锯齿边。

为了渲染出软边，这时候就只能通过控制边缘的遮罩做平滑过渡。所以这里我们该使用：

`smoothstep(begin, end, x)`

实际上这个函数和我们之前用过的 `lerp()` 方法极其相似，只不过一个为线性一个为平滑。

使用的方式和step一样，当在圆内和圆外时，会取到最大最小值0和1控制遮罩。
而在圆边线的 `±edge` 范围内，则通过获取x的变化，控制边缘遮罩的柔滑。、

来看看这题的代码吧：
```hlsl
cbuffer Uniforms : register(b0) {
  float2 iResolution;
  float iTime;
};

struct PSInput {
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target {
  float2 uv = input.position.xy / iResolution.xy;
  uv *= 2.0;
  uv -= 1.0;
  uv.x *= iResolution.x / iResolution.y;

  float2 circleCenter = float2(cos(iTime), 0.0); 
  // float  circle = 0.0;

  float3 shapeColor = float3(0.38, 0.12, 0.93);
  float3 backColor = float3(0.12, 0.12, 0.12);
  // float3 result = lerp(backColor, shapeColor, circle);
  
  float r = 0.6;
  float edge = 0.01;
  float mask = 1 - smoothstep(r - edge, r + edge, distance(circleCenter, uv));
  float3 result = (shapeColor * mask) + (backColor * (1 - mask));

  return float4(result, 1.0);
}
```

这里注释掉的两个位置实际上用处不大，但提供的lerp其实是给了个思路方向。

效果图：
![[MT-1.jpg]]

## T-7 Dot-Classify（点积含义）+ T-8 Dot Product Cosine

点积，是求在某方向上的向量大小。使用方法API为：`dot(A, B)`

计算公式的话实际上就是：`A × B = ax ​⋅ bx ​+ ay ​⋅ by`​ 或者 `A × B =|a||b|cos(θ)`

这里计算出来的点积符号(±)表示了两个向量的相对方向：
- **同向**：向量大致指向相同方向（`θ`介于0°以及90° )  
- **零**：向量是垂直的（`θ` = 90° )  
- **反向**：向量指向相反方向。(`θ`介于90°以及180°)

![[Pasted image 20260805221537.png]]

这里第7、8题都使用到了这个方法来控制一定范围的遮罩显示：
- 第7题：旋转中，对半划分颜色（黑、红）
- 第8题：以旋转单位向量两侧，左右45°内渲染为红色，其余为黑色。

第7题难度不高，主要是第8题，难在控制左右范围以及求准确的 `cos(45°)` 值。

来看代码：
```hlsl
cbuffer Uniforms : register(b0) {
  float2 iResolution;
  float iTime;
};

struct PSInput {
  float4 position : SV_Position;
};

float4 main(PSInput input) : SV_Target {
  float2 uv = input.position.xy / iResolution.xy;
  uv *= 2.0;
  uv -= 1.0;
  uv.x *= iResolution.x / iResolution.y;

  float2 a = float2(0.0, 0.0);
  float2 b = a + float2(cos(iTime), sin(iTime)) * 0.4;

//核心实现
  float maxValue = dot(normalize(float2(1.0, 1.0)), float2(1.0, 0.0)); //45度cos值
  float2 v = float2(cos(iTime), sin(iTime));
  float dirMask = step(maxValue, dot(normalize(v), normalize(uv)));
//核心实现

  float3 shapeColor = float3(01.0, 0.3, 0.3);
  float3 backColor = float3(0.12, 0.12, 0.12);
  float3 result = lerp(backColor, shapeColor, dirMask);
  
  return float4(result, 1.0);
}
```

这里主要看核心代码部分即可，剩余代码为框架代码。看看实际效果吧：
![[MT-8.jpg]]


## T-
