---
title: HTML5新特性与语义化
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - https://github.com/wxlvip/Interviewer
tags: [HTML5, 语义化, 新特性, 面试题]
---

# HTML5新特性与语义化

## 语义化

**概念**：合理正确地使用语义化标签来创建页面结构，"正确的标签做正确的事"。

**语义化标签**：`header` `nav` `main` `article` `section` `aside` `footer`

**语义化优点**：
- 无 CSS 样式时页面仍有清晰结构
- 代码结构清晰，易于阅读和维护
- 利于设备解析（屏幕阅读器等辅助工具）
- 有利于 SEO，爬虫根据标签赋予不同权重

## HTML5 新特性

- 语义化标签（header/nav/main/article/section/aside/footer）
- 音视频：`<audio>` `<video>`
- Canvas / WebGL
- 拖拽释放 Drag and Drop API
- History API（pushState / replaceState）
- requestAnimationFrame
- 地理位置 Geolocation API
- WebSocket
- Web 存储：`localStorage` / `sessionStorage`
- 新表单控件：`date` `time` `email` `url` `search` 等

## 面试高频题

**Q: HTML5 语义化有什么意义？**

> 语义化的核心价值在于"可读性"。代码让人读，也让机器读——屏幕阅读器、SEO 爬虫、团队协作都依赖语义。

**Q: localStorage 和 sessionStorage 的区别？**

> - `localStorage`：持久化存储，关闭浏览器也不会清除，同源页面共享
> - `sessionStorage`：会话级存储，关闭标签页即清除，不在 tab 之间共享
