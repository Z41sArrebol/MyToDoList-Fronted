# MyToDoList-Fronted
## 项目概述
这是一个基于 Vue 实现的本地待办事项（TodoList）应用。用户可以添加、删除、修改任务，并通过 `localStorage` 实现数据持久化存储。界面简洁直观，支持任务状态切换、清空任务等功能。
## 功能列表

### 基本需求
- [x] **任务管理**：支持输入框添加新任务，点击任务项切换完成状态（如勾选/取消勾选），删除单条任务。
- [x] **数据存储**：使用 `localStorage` API 实现任务数据的本地存储，确保页面刷新后数据不丢失。
- [x] **UI 设计**：绘制简洁直观的界面，包含任务输入区、任务列表区、清空全部任务按钮。

### 扩展需求（部分实现）
- [ ] **任务分类**：支持为任务添加标签（如“工作”“生活”“学习”），并可按标签筛选任务。
- [ ] **本地搜索**：实现任务名称关键词搜索功能。
- [ ] **过期删除**：支持给任务设置 deadline，过期后自动删除。

## 技术栈
- Vue.js
- HTML / CSS / JavaScript
- localStorage

## 快速开始

- npm install
- npm run serve
