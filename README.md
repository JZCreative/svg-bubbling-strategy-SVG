# svg-bubbling-strategy-skill

这个文件夹把当前项目里已经验证过的纯 SVG 编组交互方法，整理成一个可复用的 SKILL。

适用范围：

- 微信 SVG / 小程序 SVG 等强约束环境
- 纯 SVG、无 JS、无块级 CSS 的交互动画
- 无 `id` / `class` 条件下，通过深层 `<g>` 嵌套和事件冒泡组织复杂动画
- 文本卡、`<image>`、外部 SVG、局部姿态和全局时间轴的统一编排

## 文件结构

- `SKILL.md`
  - 技能入口文件
  - 定义何时调用、核心规则、排错顺序和常见错误

- `01-core-principles.md`
  - 整套方法论的核心原则
  - 包括洋葱式嵌套、施加/抵消、对象模块化、交互统一等

- `02-image-integration-and-pitfalls.md`
  - 记录 `<image>` 接入方式
  - 记录外部素材、字幕层、文本卡、局部闪烁的典型错误与修法

## 推荐阅读顺序

1. 先读 `SKILL.md`
2. 再读 `01-core-principles.md`
3. 最后按需要查 `02-image-integration-and-pitfalls.md`

## 适用规范

在使用这套 SKILL 之前，建议先明确两层前置规范：

### 1. SVG AttributeName 白名单规范

- 本项目遵循 SVG AttributeName 白名单规范
- 参考链接：[SVG AttributeName 白名单详解（2025版）](https://mp.weixin.qq.com/s/1J6SrNqk2UQDx0dSP2TLtQ)

这意味着：

- 不是所有 SVG 动画写法都能在目标平台中保留
- 超出白名单的动画标签、动画指令或属性名，有可能被平台直接过滤
- 在微信生态中做纯 SVG 交互时，必须先按白名单理解“能做什么”，再设计具体方案

README 内直接保留一份精简白名单速查表：

| 元素 | Name | 说明 |
| --- | --- | --- |
| `animate` | `x` | 控制简单几何体 x 轴方向移动 |
| `animate` | `y` | 控制简单几何体 y 轴方向移动 |
| `animate` | `width` | 控制简单几何体宽度变化 |
| `animate` | `height` | 控制简单几何体高度变化 |
| `animate` | `opacity` | 控制透明度变化，数值通常为 0 到 1 |
| `animate` | `d` | 控制贝塞尔曲线补间，表现可能有随机性 |
| `animate` | `points` | 控制多边形补间，表现可能有随机性 |
| `animate` | `stroke-width` | 控制描边宽度 |
| `animate` | `stroke-linecap` | 控制描边端点样式 |
| `animate` | `stroke-dashoffset` | 控制描边偏移，常用于线性遮罩/进度线 |
| `animate` | `fill` | 控制填充色过渡变化 |
| `set` | `visibility` | 控制可见性，如 `visible / hidden / collapse / inherit` |
| `animateTransform` | `translate` | 控制路径和编组位移 |
| `animateTransform` | `scale` | 控制路径和编组缩放，可用于翻转 |
| `animateTransform` | `rotate` | 控制路径和编组旋转 |
| `animateTransform` | `skewX` | 控制 x 轴倾斜 |
| `animateTransform` | `skewY` | 控制 y 轴倾斜 |
| `animateMotion` | `path` | 复杂轨迹动画，可配合 `rotate` 定义朝向 |

另外，本项目还默认遵守以下禁用边界：

- 不使用 `id`
- 不使用 `class`
- 不使用 `href` / `xlink:href` 作为样式与结构依赖
- 不使用 `defs`
- 不使用 `embed`
- 不使用块级 CSS 与 JS 动画
- 样式尽量以内联属性或 `style=""` 的最小必要形式书写

白名单完整原始表格文件位于：

- [白名单表格-index.html](file:///Users/yaoyao/Documents/Marshall工作文档/AIGC/Gemini/连续触发认知/白名单表格-index.html)

### 2. 《融媒体 SVG 交互设计技术规范》

- 建议初学开发者同步掌握 `T/CASME 1609—2024`
- 参考链接：[中华人民共和国《融媒体 SVG 交互设计技术规范》](https://www.fudan.design/svg.html)

建议重点关注：

- XML / SVG 语法规范
- 不同平台的自有技术规范
- 移动端 iOS / Android 的内核差异
- 触发器、反馈、动画缓动、防误触等交互设计要求

这套 SKILL 的方法论，是在上述规范边界内进一步沉淀出的工程化实现经验，而不是脱离规范单独成立的技巧库。

## 当前项目收敛结论

当前 `ufo-demo-phase1.html` 验证出的最终稳定规范是：

```xml
begin="click"
fill="freeze"
restart="never"
```

并且：

- 最深层透明触发器必须在整条父子链底部
- 每个遭遇物必须拥有自己的施加层、姿态层、抵消层
- 所有对象文案和 `<image>` 都必须绑定在正确的局部组里
- 即使是收尾字幕，也应该放在最深层内部补偿后的视口坐标层里，而不是脱链挂在最外面

## 实践项目

这套 SKILL 不是抽象总结，而是基于两个已经落地的真实实践项目沉淀出来的。

### 1. AI 最先杀死的，是谁？

- 项目链接：[AI 最先杀死的，是谁？](https://mp.weixin.qq.com/s/a_oKNMlSQLuAMfDMsOD0Wg)
- 项目特征：
  - 偏信息叙事型 SVG 交互
  - 强调点击 / 长按后的内容展开与视觉编排
  - 适合作为“纯 SVG 信息表达 + 交互触发”的案例

截图建议：

- 建议将项目截图保存为：
  - `screenshots/project-ai-kills-frontend.png`

插图预留：

```md
![AI 最先杀死的，是谁？示意图](./screenshots/project-ai-kills-frontend.png)
```

### 2. 美国公布的 UFO 文件到底有什么？

- 项目链接：[美国公布的 UFO 文件到底有什么？](https://mp.weixin.qq.com/s/yquAobUj1JLLmd0pniN2Dw)
- 项目特征：
  - 偏叙事探索型 SVG 场景动画
  - 包含深层 `<g>` 嵌套、对象遭遇链、文本卡、`<image>` 素材接入、收尾字幕等完整方法链
  - 是这套 `svg-bubbling-strategy-skill` 的直接实战来源

截图建议：

- 建议将项目截图保存为：
  - `screenshots/project-ufo-uap-files.png`

插图预留：

```md
![美国公布的 UFO 文件到底有什么？示意图](./screenshots/project-ufo-uap-files.png)
```

## 截图放置建议

如果要把实践截图真正附到 README 中，建议在本技能目录下新增：

```text
screenshots/
  project-ai-kills-frontend.png
  project-ufo-uap-files.png
```

然后将上面的 Markdown 图片引用解除注释或直接插入到对应小节下。
