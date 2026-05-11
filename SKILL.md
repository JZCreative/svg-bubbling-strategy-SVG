---
name: "svg-bubbling-strategy-skill"
description: "用于纯 SVG 无 ID 冒泡编组交互设计。用户要做微信 SVG、深层嵌套、事件冒泡、image 接入或排查 transform 串层时调用。"
---

# SVG Bubbling Strategy Skill

适用于以下任务：

- 用户要在纯 SVG 中实现复杂交互动画
- 约束条件包含：无 JS、无块级 CSS、无 `id/class/href/xlink:href/defs/embed`
- 需要依赖深层 `<g>` 嵌套、透明触发器、事件冒泡
- 需要设计或排查“施加 -> 抵消 -> 再施加”的编组动画
- 需要把文本卡、`<image>`、外部 SVG、局部姿态动画接入同一时间轴
- 需要排查“平级而非嵌套”“补偿层缺失”“opacity 误伤深层”“时序错位”等问题

## 核心使用原则

1. 永远先判断这是不是“真正的无 ID 冒泡编组问题”
2. 永远先搭结构，再写动画
3. 永远先区分世界层 / 遭遇层 / 触发层
4. 永远把最深层透明触发器放在整条父子链的最底部
5. 永远把每个遭遇物做成可闭环模块：施加层 -> 姿态层 -> 抵消层
6. 永远先统一交互模式，再统一全链实现

## 当前项目验证过的正确方法

### 1. 嵌套规则

- 不是“很多 `<g>` 就够了”，而是必须形成连续不间断的绝对父子链
- 任何平级断链都会让冒泡和补偿逻辑失效

### 2. 变换规则

- 标准写法是：施加 -> 抵消 -> 再施加
- 父层负责改变参考系
- 子层负责拿回局部控制权
- 更深层通过补偿层回到干净坐标基准

### 3. 交互规则

当前项目的正式落地版本采用：

```xml
<animateTransform
  begin="click"
  dur="30s"
  fill="freeze"
  restart="never"
  ... />
```

当前策略的固定要求：

- 所有交互动画统一 `begin="click"`
- 所有交互动画统一 `fill="freeze"`
- 所有交互动画统一 `restart="never"`
- 删除所有旧的 `touchend;mouseup` 回位链
- 不允许一部分层保留旧长按逻辑，一部分层改成 click-only

## `<image>` 与外部素材接入规则

- `<image>` 不能直接丢到 SVG 根部
- 必须挂在对应对象的局部姿态层或遭遇层内部
- 它的位移、旋转、出场时机应该由外层 `<g>` 控制
- 宽高、位置全部写成属性，不依赖外部 CSS
- 多张 `<image>` 可以在同一组里通过不同 `x/y` 形成排布序列

示例：

```xml
<g>
  <animateTransform attributeName="transform" type="translate"
      begin="click" dur="80s"
      values="..."
      keyTimes="..."
      fill="freeze" restart="never" />

  <g>
    <animateTransform attributeName="transform" type="rotate"
        begin="click" dur="80s"
        values="..."
        keyTimes="..."
        fill="freeze" restart="never" />

    <image x="0" y="-800" width="609" height="639" href="..." />
    <image x="0" y="-1500" width="609" height="634" href="..." />
  </g>
</g>
```

## 排错顺序

按下面顺序排查，不要跳步：

1. 先查是否还是绝对父子链
2. 再查触发器是否真的在最深层
3. 再查补偿层是否成对存在
4. 再查坐标是否跑出视口
5. 再查 `opacity/visibility` 是否误伤深层
6. 最后查 `keyTimes` 与出现窗口

## 常见错误与固定修法

### 错误 1：把平级 `<g>` 当深层结构

- 现象：结构看起来很多层，但事件链断了
- 修法：回到单一路径父子链

### 错误 2：只施加，不抵消

- 现象：后续遭遇物继承前一层轨迹
- 修法：每个遭遇物都补上自己的抵消层

### 错误 3：局部闪烁挂在公共父层

- 现象：红灯闪一下，飞机或更深层对象一起透明
- 修法：把 `opacity` 动画缩到最小局部组

### 错误 4：文案或收尾字幕放对视觉，放错逻辑

- 现象：看起来居中，但脱离最深层编组体系
- 修法：如果是对象文案，就绑对象局部层；如果是全局收尾，也要放在最深层补偿后的内部视口坐标层

### 错误 5：交互模式半新半旧

- 现象：主动画改成 `click`，但提示文案、遮挡层、回位层还保留旧逻辑
- 修法：全链统一替换并复查

## 工作建议

- 如果是新项目，先读 `README.md`
- 如果是结构设计，优先看 `01-core-principles.md`
- 如果是接素材与排错，优先看 `02-image-integration-and-pitfalls.md`
