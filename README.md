# Cutefish Modified Project / Cutefish 改进版

## 📌 Overview / 项目简介

This is a customized and actively maintained version of the Cutefish Desktop Environment.

这是一个基于 Cutefish 桌面环境的深度修改与重构版本。

---

## 🚀 Highlights / 项目亮点

* Qt6 + Wayland migration (libcutefish)

* Modular architecture across multiple repositories

* System-level improvements and patches

* 完成 Qt6 + Wayland 迁移（libcutefish）

* 多仓库模块化架构

* 系统级修改与优化

---

## 📦 Repositories / 项目组件

### 🧠 Core / 核心

* https://github.com/yang10147/cutefish-core

### 🖥 UI Components / 界面组件

* https://github.com/yang10147/cutefish-dock
* https://github.com/yang10147/cutefish-launcher
* https://github.com/yang10147/cutefish-statusbar

### ⚙️ System Components / 系统组件

* https://github.com/yang10147/cutefish-settings
* https://github.com/yang10147/cutefish-screenlocker
* https://github.com/yang10147/cutefish-filemanager
* https://github.com/yang10147/cutefish-screenshot

### 🔬 Core Library (Qt6 + Wayland) / 核心库

* https://github.com/yang10147/libcutefish

### 🎨 Resources / 资源

* https://github.com/yang10147/cutefish-icons
* https://github.com/yang10147/cutefish-wallpapers

---

## 🧩 Modifications / 修改内容

* Refactored multiple components

* Added patch-based extensions

* Improved compatibility and behavior

* 多组件重构

* 引入 patch 模块机制

* 提升兼容性与系统行为

---

## 📄 Documentation / 文档

* `cutefish-archive.md`

---

## 🛠 Build / 构建

(Work in progress)

（整理中）

---

## 📌 Notes / 说明

This project represents a personal continuation and enhancement of Cutefish.

本项目为对 Cutefish 的延续性开发与增强版本。

---

## ⭐ Future Plans / 未来计划

* Stabilize Qt6 + Wayland environment

* Improve usability and performance

* Expand feature set

* 完善 Qt6 + Wayland 稳定性

* 提升体验与性能

* 增强功能

---
这是一个非常明智的决定。在工程上，及时止损并做好“项目复盘”是职业开发者必备的素养。把这些技术细节沉淀到 README 中，不仅是对自己这段时间探索的交代，也能为后来者指明“雷区”在哪。

你可以直接将以下内容复制到你 GitHub 项目的 **README.md** 或者 **Known Issues** 章节中：

---

### 🛠 已知问题与待修复清单 (Known Issues & TODO)
本项目完全有claude完成，chatGPT负责推github，gemini辅助修个别bug，本人一行代码都没写。从3月31日开始历时23天。
请先安装kde桌面，由于借用了kde的装饰器和各种依赖，因为claude偷懒没有移植装饰器。但是不知道具体是哪些组件。
本项目已初步完成从 **Qt5/X11** 到 **Qt6/Wayland** 的核心架构迁移，但在 Wayland 会话（Session）中仍存在以下兼容性瑕疵，欢迎社区贡献补丁。

#### 1. UI 与 交互 (User Interface & Interaction)
* **窗口装饰残缺**：由于 `KDecoration` 插件尚未完全适配 Qt6，目前的窗口标题栏和边框仍保持 KWin 默认风格，磨砂玻璃（Blur）等毛玻璃特效在某些组件中未能正确开启。
* **主题切换失效**：设置面板中的“明暗模式”无法实时同步到所有桌面组件，疑似 `libcutefish` 中的主题单例信号在 Qt6 下未正确广播。
* **Dock 行为异常**：
    * Dock 无法动态改变长度和屏幕位置。
    * **智能隐藏失效**：浏览器等窗口最大化时，Dock 无法自动收起，且鼠标触碰边缘时无法触发唤醒。
    * **实例追踪错误**：点击已运行软件的图标会开启新实例，而非激活/置顶现有窗口（Wayland 下的 TaskManager 协议对接不完善）。

#### 2. 系统功能 (System Functions)
* **锁屏接管失效**：系统休眠或手动锁屏时，默认调用 SDDM 或 PLM 界面，而非原生的 `cutefish-screenlocker`（需修复 PAM 认证及核心组件的调用路径）。
* **截图功能 (Screenshot)**：在 Wayland 环境下无法通过旧接口抓取像素，需适配 `xdg-desktop-portal` 或合成器私有协议。
* **状态栏图标 (Statusbar)**：蓝牙小图标不显示。虽设置面板功能正常，但状态栏指示器组件（Indicator）在 Qt6 下存在加载异常。

#### 3. 核心依赖 (Core Dependencies)
* 目前所有组件均为源码编译安装，尚未打包。
* 去除了`FishUI` 库，重写了Theme.QML,但是我不知道什么原理，claude非要这么做。


---

### 📝 开发随笔：为什么选择暂停？

因为浪费了我一个多月验证claude的sonnet4.5的工程能力，但是已经可以进入桌面正常使用，剩下的都是修一些小bug的脏活。

---

