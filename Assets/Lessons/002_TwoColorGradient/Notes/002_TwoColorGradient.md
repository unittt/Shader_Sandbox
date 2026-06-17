# 002 Two Color Gradient

## 效果目标

创建一个双色渐变材质，能够通过同一个 Shader Graph 做出纵向渐变和横向渐变。重点理解 UV 坐标中 `UV.x` 与 `UV.y` 在 `0..1` 范围内的变化。

## 涉及知识点

- UV
- Split
- Lerp
- Base Color

## 实现思路

使用 UV 节点读取模型的纹理坐标，再用 Split 将 UV 拆成 R/G 分量。R 对应 `UV.x`，通常从左到右由 `0` 变化到 `1`；G 对应 `UV.y`，通常从下到上由 `0` 变化到 `1`。

通过 AxisBlend 参数在 `UV.y` 和 `UV.x` 之间做插值切换：

```text
Factor = Lerp(UV.y, UV.x, AxisBlend)
FinalColor = Lerp(ColorA, ColorB, Factor)
```

当 `AxisBlend = 0` 时，Lerp 完全取第一个输入 `UV.y`，得到纵向渐变；当 `AxisBlend = 1` 时，Lerp 完全取第二个输入 `UV.x`，得到横向渐变。`0..1` 中间值会混合两个方向，可以得到斜向过渡。

## 参数说明

- ColorA：渐变起始颜色。
- ColorB：渐变结束颜色。
- AxisBlend：方向混合值，`0` 为纵向，`1` 为横向，中间值为混合方向。

## 材质示例

- M_002_TwoColorGradient_Vertical：`AxisBlend = 0`，使用 `UV.y` 做纵向渐变。
- M_002_TwoColorGradient_Horizontal：`AxisBlend = 1`，使用 `UV.x` 做横向渐变。

## 复盘

- `UV.x` 可以理解为横向坐标，左侧接近 `0`，右侧接近 `1`。
- `UV.y` 可以理解为纵向坐标，底部接近 `0`，顶部接近 `1`。
- Lerp 的 T 参数决定颜色混合比例：`0` 输出 ColorA，`1` 输出 ColorB，中间值输出过渡色。
- 同一个 Shader Graph 通过不同材质参数可以复用出不同方向的渐变效果。
- 这里没有使用 bool，是为了练习 Lerp 的选择逻辑：`Lerp(A, B, 0)` 等于 A，`Lerp(A, B, 1)` 等于 B。
- 观察 UV 效果时优先使用 Quad 或 Plane。压扁 Cube 虽然看起来像平面，但不同面的 UV 方向可能不直观，容易干扰判断。
