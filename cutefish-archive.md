# CutefishOS Wayland 移植 — 归档文档（三期）
> 一期 + 二期全部完成记录。新对话开始时上传此文件。

---

## 用户环境

```
OS:        CachyOS x86_64
DE:        KDE Plasma 6 / GNOME（Wayland 会话，调试时切换到 GNOME 避免 KDE 冲突）
Qt 版本:   6.10.2 / KDE Frameworks 6.24.0 / 内核 6.19.10-1s
用户名:    yong / 主目录: /home/yong
代理端口:  7890（克隆前需设置 export https_proxy=http://127.0.0.1:7890）
会话类型:  wayland
下载目录:  ~/下载/（中文路径，tar xzf 时注意）
```

---

## 重要背景知识

- **FishUI**：Qt5 编译，Qt6 不可加载。所有 QML 必须完全去掉 FishUI，用原生 Qt6 替代。
- **Theme 单例**：`qml/Theme.qml` 替代 FishUI，根类型 `Item`，singleton，子目录用 `import "../"` 访问。
- **QT_QUICK_CONTROLS_STYLE=Basic**：main.cpp 最早处 setenv，在 QApplication 前。
- **音频插件**：用 pactl 命令行，2秒轮询。
- **settings-daemon 需手动启动**：`cutefish-settings-daemon &`
- **KDE Plasma 调试冲突**：建议切换到 GNOME 会话调试 cutefish-settings。
- **QML 编译进二进制**：修改 QML 后必须删除 `qrc_resources.cpp` 强制重编：
  ```bash
  rm ~/cutefish-settings/build/cutefish-settings_autogen/UVLADIE3JM/qrc_resources.cpp
  make -j$(nproc)
  sudo cp ~/cutefish-settings/build/cutefish-settings /usr/bin/cutefish-settings
  ```
- **git clone 认证问题**：用 curl 下载 zip 绕过：
  ```bash
  curl -x http://127.0.0.1:7890 -L https://github.com/cutefishos/xxx/archive/refs/heads/master.zip -o xxx.zip
  ```

---

## 已完成组件总览

### cutefish-dock ✅
LayerShellQt 贴底部，exclusiveZone 生效，图标/右键菜单正常。窗口激活/高亮空实现（PlasmaWindowManagement 协议限制）。

### cutefish-statusbar ✅
LayerShellQt 贴顶部，系统托盘/时钟/关机菜单/控制中心正常。音量滑块用 pactl 实现，亮度滑块依赖 settings-daemon。

### cutefish-core 各子组件 ✅

| 组件 | 关键改动 |
|------|---------|
| session | 直接编译，无需修改 |
| shutdown-ui | Qt5→Qt6，去 FishUI/GraphicalEffects，纯深色渐变背景 |
| settings-daemon | 去所有 X11/XCB；mouse/touchpad 改用 kwinrc+KWin DBus；theme 去 XResources |
| screen-brightness | Qt5→Qt6，源码无需修改 |
| notificationd | Qt5→Qt6/KF6，去 KWindowSystem X11，去 FishUI，注册 icontheme provider |
| gmenuproxy | Qt5→Qt6/KF6，去 XCB，窗口属性 no-op |
| powerman | Qt5→Qt6/KF6，去 XCB/DPMS，DPMS 调用 no-op |
| polkit-agent | Qt5→Qt6，PolkitQt5→PolkitQt6，去 FishUI，拖动改 startSystemMove() |
| chotkeys | XCB xcb_grab_key 完全重写为 KF6GlobalAccel C++ API |
| clipboard | Qt5→Qt6，QApplication→QGuiApplication |

### cutefish-screenshot ✅
通过 XDG Desktop Portal 截图（Wayland 兼容）。

### cutefish-launcher ✅
### cutefish-screenlocker ✅
### cutefish-filemanager ✅
### cutefish-icons ✅（纯文件安装，无 Qt 依赖）
### cutefish-wallpapers ✅（纯文件安装，无 Qt 依赖）

---

## cutefish-settings ✅ 全部完成

### 编译方法
```bash
cd ~/cutefish-settings/build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
rm cutefish-settings_autogen/UVLADIE3JM/qrc_resources.cpp
make -j$(nproc) && sudo cp cutefish-settings /usr/bin/cutefish-settings
```

### 已完成页面
About / Notification / Power / Proxy / Touchpad / Fonts / DateTime / DefaultApp /
Battery / Dock / Language / Display（亮度+缩放）/ Appearance / Wallpaper / Sound /
Cursor（图标占位）/ User / WLAN / **Wired** / **Bluetooth**

### 关键 C++ 改动
- Qt5→Qt6/KF6，去 X11Extras
- `background.cpp`：std::sort lambda 修复
- `cursor/cursortheme.cpp`：QX11Info → Qt6 native interface
- `fonts/kxftconfig.cpp`：QX11Info::appDpiY() → primaryScreen()->logicalDotsPerInchY()

---

## libcutefish Qt6 移植 ✅

### 仓库位置 / 编译方法
```bash
mkdir -p ~/libcutefish/build && cd ~/libcutefish/build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make -j$(nproc)
sudo make install
```

### 已完成插件

| 插件 | 状态 | 关键改动 |
|------|------|---------|
| `Cutefish.System` | ✅ | Qt5→Qt6，CMakeLists 仅改依赖名 |
| `Cutefish.Accounts` | ✅ | Qt5→Qt6，`qt5_add_dbus_interface`→`qt6_add_dbus_interface` |
| `Cutefish.NetworkManagement` | ✅ | KF5→KF6，见踩坑记录 |

### 跳过插件

| 插件 | 原因 |
|------|------|
| `Cutefish.Screen` | 已用 kscreen-doctor 绕过 |
| `Cutefish.Audio` | statusbar 已用 pactl 实现 |
| `Cutefish.Bluez` | Bluetooth 页面改用系统 KDE 插件，不再需要 |
| `Cutefish.Mpris` | 暂不需要 |

---

## ⚠️ 踩坑详细记录

### 坑1：libcutefish 根 CMakeLists.txt 的 qmake 查找方式
Qt5 写法 `get_target_property(QT_QMAKE_EXECUTABLE ...)` 在 Qt6 报错。

**修复：**
```cmake
find_program(QT_QMAKE_EXECUTABLE NAMES qmake6 qmake)
execute_process(COMMAND ${QT_QMAKE_EXECUTABLE} -query QT_INSTALL_QML
    OUTPUT_VARIABLE INSTALL_QMLDIR
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
```

### 坑2：networkmanagement 的 WITH_MODEMMANAGER_SUPPORT 宏
CachyOS 无 `kf6-modemmanager-qt`。改可选后仍不够，因为 sed 误操作导致双重 `#if`。

**最终解决：** 直接删掉整行：
```bash
sed -i '/add_definitions(-DWITH_MODEMMANAGER_SUPPORT)/d' ~/libcutefish/networkmanagement/CMakeLists.txt
```
注意：不要用 `=0` 方式，会重定义警告。

### 坑3：appletproxymodel.cpp 的 filterRegExp → filterRegularExpression
`QRegularExpression` 没有 `isEmpty()`，需用 `.pattern().isEmpty()`。

### 坑4：Hideable.qml 有无用的 QtGraphicalEffects import
删掉 `import QtGraphicalEffects 1.0` 那行即可，内容完全没用到。

### 坑5：UserDelegateItem.qml 的 Switch Component.onCompleted 问题
Qt6 里 `Layout.alignment` 与 `Component.onCompleted` 附加属性冲突，改用属性绑定：
```qml
checked: currentUser.automaticLogin
onCheckedChanged: currentUser.automaticLogin = checked
```

### 坑6：WLAN/Main.qml 的 Component.onCompleted 位置问题
根元素里写会报 `Non-existent attached object`，改用 Timer：
```qml
Timer {
    interval: 10200; repeat: true; running: control.visible
    triggeredOnStart: true
    onTriggered: handler.requestScan()
}
```

### 坑7：RegExpValidator → RegularExpressionValidator
```qml
validator: RegularExpressionValidator { regularExpression: /^.../ }
```

### 坑8：FishUI.Window 对话框替换
ConnectDialog / NewNetworkDialog / PairDialog 原来继承 `FishUI.Window`，改用标准 `Dialog`：
```qml
Dialog {
    modal: true
    x: parent ? (parent.width - width) / 2 : 0
    y: parent ? (parent.height - height) / 2 : 0
    function show() { open() }  // 兼容原来的 show() 调用
}
```

### 坑9：Wired 页面原代码 bug
原代码用 `wwanHwEnabled`/`wwanEnabled`/`enableWwan`（移动网络 API）控制有线网，是上游 bug。
**修复：** 改用 `enabledConnections.networkingEnabled` / `handler.enableNetworking()`。

### 坑10：Bluetooth 页面不用 Cutefish.Bluez
`Bluetooth/Main.qml` 实际用的是系统 KDE 插件，直接替换：
- `org.kde.bluezqt 1.0` — 蓝牙管理，device.connectToDevice() / disconnectFromDevice() / pair()
- `org.kde.plasma.private.bluetooth 254.0` — DevicesProxyModel（注意版本号必须是 254.0）

`DevicesProxyModel` 的 role 里**没有** `Connecting`/`Disconnecting`（那些在 `DevicesStateProxyModel` 里），
直接用这两个 role 会得到 `undefined`，在 QML 里被当 true，导致 BusyIndicator 一直转。解决方法：不加 BusyIndicator。

### 坑11：cutefish-settings QML 重编必须删 qrc_resources.cpp
```bash
rm ~/cutefish-settings/build/cutefish-settings_autogen/UVLADIE3JM/qrc_resources.cpp
make -j$(nproc)
sudo cp ~/cutefish-settings/build/cutefish-settings /usr/bin/cutefish-settings
```

---

## 已知遗留问题

| 问题 | 原因 | 优先级 |
|------|------|--------|
| Dock/Statusbar 窗口激活/高亮缺失 | PlasmaWindowManagement 协议限制 | 低 |
| Statusbar 背景色写死灰色 | 需壁纸服务 | 低 |
| Display 分辨率/旋转不可用 | 依赖 Cutefish.Screen | 中 |
| Cursor 页面图标空白 | cursor.svg 未正确编译进 qrc | 低 |
| settings-daemon 未自动启动 | 缺 autostart 集成 | 中 |
| WLAN 信号图标为 Unicode | 原 SVG 依赖 FishUI 路径 | 低（视觉） |
| Bluetooth 配对 agent 未实现 | PairDialog 为简化版，无 pin 确认回调 | 中 |

---

## 跳过组件

- `cutefish-core/xembed-sni-proxy` — 纯 X11
- `cutefish-core/cupdatecursor` — 纯 X11（注意：运行会写入光标配置导致光标变大）
- `cutefish-core/sddm-helper` — 暂不处理
- `cutefish-calculator` — 独立工具，按需移植
- `cutefish-qt-plugins` — 按需评估

---

## 后续可考虑的方向

- **settings-daemon autostart**：写 `.desktop` 文件到 `~/.config/autostart/`
- **Display 页面分辨率/旋转**：移植 `Cutefish.Screen` 或改用 kscreen DBus API
- **Bluetooth 配对 agent**：实现完整 BluezQt agent，支持 pin 确认
- **WLAN 信号强度图标**：改用真实 SVG（已有 qrc 资源）替代 Unicode
- **Cutefish.Mpris**：如需媒体控制，移植此插件
- **自有 Wayland compositor**：解锁窗口激活/高亮等功能

---

*最后更新：2026-04-13*
*当前进度：移植主体工作全部完成。cutefish-settings 所有页面可用。*
