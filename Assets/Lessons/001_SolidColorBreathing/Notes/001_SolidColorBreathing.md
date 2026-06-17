# 001 Solid Color Breathing (纯色自发光呼吸灯)

## 效果目标
创建一个自发光材质，使物体表面颜色亮度随时间产生周期性的平滑呼吸起伏变化。

## 涉及知识点
- Time (_Time.y 寄存器)
- Sine (正弦波函数)
- Remap (区间缩放映射)
- Lerp (线性插值)
- Multiply (向量/标量乘法运算)
- Emission (自发光通道，配合后处理 Bloom)

## 实现思路
1. 波动发生：用系统时间 `Time` 乘以速度 `Speed` 驱动 `Sine` 函数，得到 $[-1, 1]$ 之间的周期波动值。
2. 区间重构：通过 `Remap` 节点将信号由 $[-1, 1]$ 映射至 $[0, 1]$（消除正弦波负半周带来的死黑截断停顿）。
3. 强度混合：将映射后的 $[0, 1]$ 标量作为权重系数传入 `Lerp`，在 `MinIntensity` 和 `MaxIntensity` 之间插值。
4. 最终着色：用基础 `Color`（Vector4）乘以当前计算出的强度值（Vector1），将结果输出到 `Emission`（自发光通道）。

## 参数说明
- Color (Vector4)：呼吸灯的基础色相。
- MinIntensity (Float)：呼吸谷值的亮度强度（设为 0 则全黑）。
- MaxIntensity (Float)：呼吸峰值的亮度强度（设为大于 1 的值可触发 HDR 辉光拉丝效果）。
- Speed (Float)：呼吸频率。

## 复盘与底层思考
- Sine 的原生值域是 $[-1, 1]$，直接做插值因负数会被硬件截断导致长达半个周期的死黑。在底层代码中，Remap 的本质等同于：`factor = sin(t) * 0.5 + 0.5;`。
- 保持先在一维空间（Float）计算出最终的 Intensity，最后再与四维向量（Color）做 `Color * Intensity` 的乘法。这保证了色相不变，且榨干了 GPU 算力。
- Speed 较低时曲线平滑更具生命感；Speed 较高时光流急促更具科幻/故障闪烁感。