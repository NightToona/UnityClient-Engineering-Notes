
> [!NOTE] 引言
> 该部分主要以HLSL中的数学部分为核心，通过常见的数学方法来解决视觉问题

# Math

## T-1

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

## T-