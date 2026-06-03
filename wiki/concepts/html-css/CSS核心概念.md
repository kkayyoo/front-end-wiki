---
title: CSS核心概念：盒模型、BFC、布局、选择器
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - https://github.com/wxlvip/Interviewer
tags: [CSS, 盒模型, BFC, Flex, 布局, 选择器, 面试题]
---

# CSS 核心概念

## CSS 选择器及优先级

**选择器权重（从高到低）**：

```
!important > 内联样式(1000) > ID选择器(0100) > 类/属性/伪类(0010) > 元素/伪元素(0001) > 通配符/关系(0000)
```

常用选择器：id(#)、类(.)、属性([attr])、伪类(:hover)、标签、相邻(h1+p)、子(ul>li)、后代(li a)、通配符(*)

## CSS 盒模型

CSS 盒模型包含：**content + padding + border + margin**

- **标准盒模型（W3C）**：`width` 只包含 content，总宽 = width + padding + border + margin
- **IE 盒模型（怪异盒）**：`width` 包含 content + padding + border，总宽 = width + margin

```css
box-sizing: content-box; /* 标准盒模型，默认 */
box-sizing: border-box;  /* IE 盒模型，实际开发更常用 */
```

> **实际开发**：现代项目通常全局设置 `* { box-sizing: border-box }`，更符合直觉——设置宽度后 padding/border 不会撑大元素。在 Azure Data 类项目的表格、表单布局中尤其重要。

## BFC（块级格式化上下文）

**BFC** 是一个独立的渲染区域，内部布局不影响外部。

**触发 BFC 的条件**：
- 根元素 `<html>`
- `float` 值不为 none
- `position` 为 absolute 或 fixed
- `display` 为 inline-block、table-cell、table-caption
- `overflow` 值不为 visible（常用 `overflow: hidden`）

**BFC 的布局规则**：
- 内部 Box 垂直依次放置
- 同一 BFC 的相邻 Box margin 会发生折叠
- BFC 区域不会与 float box 重叠
- 计算高度时，浮动元素也参与计算

**BFC 的使用场景**：
- 清除浮动（父元素高度塌陷问题）
- 防止 margin 折叠
- 避免元素被浮动元素覆盖
- 实现两栏布局

## position 定位属性

| 值 | 说明 |
|---|---|
| `static` | 默认，正常文档流 |
| `relative` | 相对自身原位置偏移，仍占文档流空间 |
| `absolute` | 相对最近已定位祖先元素，脱离文档流 |
| `fixed` | 相对视口固定，脱离文档流 |
| `sticky` | 滚动到阈值前 relative，之后 fixed |

> **实际踩坑**：`sticky` 失效常见原因：父元素设置了 `overflow: hidden/auto/scroll`，会打断粘性定位。

## 水平垂直居中方案

```css
/* 方案1：Flex（最推荐） */
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 方案2：绝对定位 + transform */
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* 方案3：Grid */
.parent {
  display: grid;
  place-items: center;
}
```

## Flex 布局

**容器属性**：
- `flex-direction`：主轴方向（row/column）
- `justify-content`：主轴对齐（center/space-between/space-around）
- `align-items`：交叉轴对齐（center/stretch/flex-start）
- `flex-wrap`：是否换行

**子项属性**：
- `flex: 1`（flex-grow: 1, flex-shrink: 1, flex-basis: 0）
- `align-self`：单个子项交叉轴对齐
- `order`：排列顺序

## 元素隐藏的三种方式

| 方式 | 占位 | 事件触发 | 重排重绘 |
|---|---|---|---|
| `display: none` | 否 | 否 | 回流+重绘 |
| `visibility: hidden` | 是 | 否 | 重绘 |
| `opacity: 0` | 是 | **是** | 重绘 |

## 清除浮动

```css
/* 推荐：伪元素清除浮动 */
.clearfix::after {
  content: '';
  display: block;
  height: 0;
  visibility: hidden;
  clear: both;
}

/* 或触发 BFC */
.parent {
  overflow: hidden;
}
```

## CSS3 新特性

- **过渡**：`transition: all 0.3s ease`
- **动画**：`@keyframes` + `animation`
- **变换**：`transform: translate/rotate/scale`
- **阴影**：`box-shadow` `text-shadow`
- **渐变**：`linear-gradient` `radial-gradient`
- **弹性布局**：Flexbox
- **网格布局**：Grid
- **媒体查询**：`@media`
- **CSS 变量**：`--color: red; color: var(--color)`

## CSS 预处理器（Sass/Less/Stylus）

| 特性 | Sass | Less | Stylus |
|---|---|---|---|
| 扩展名 | `.scss/.sass` | `.less` | `.styl` |
| 变量前缀 | `$` | `@` | 任意 |
| 共同特性 | 嵌套、运算、混入、继承、导入 |

## 面试高频题

**Q: 什么是回流（reflow）和重绘（repaint）？**

> - **回流**：元素的尺寸、位置、布局发生变化，浏览器需要重新计算几何属性，性能代价大
> - **重绘**：元素外观（颜色、背景）变化，不影响布局，只重新绘制
> - 回流一定触发重绘，重绘不一定触发回流
> - 优化：批量修改样式（classList）、使用 transform 代替 top/left 动画、避免频繁读取 offsetWidth

**Q: CSS 渐进增强和优雅降级的区别？**

> - **渐进增强**：先针对低版本浏览器构建基本功能，再向上增强体验（从底向上）
> - **优雅降级**：先构建完整功能，再对低版本做兼容降级（从顶向下）
