# peach-joystick
> vue虚拟摇杆插件<br>
> Virtual Joystick component for Vue.js

## 在线预览 | Preview demo
https://taoqi0813.github.io/#/peach-joystick

## 快速上手 | Getting started
### 安装 | Installation
```
npm install -S peach-joystick

```
### 在main.js引用 | In Main.js:  
```
import PeachJoystick from 'peach-joystick'
Vue.use(PeachJoystick)
```

### 用法 | Usage
```
<peach-joystick :options="options" @move="move/>
```

### 配置 | Options

+ size
  - 摇杆的尺寸(宽=高) | joystick size(width = height)
  - 类型: String | Type: String
  - 默认值: '120' | Default: '120'
  - 不要带单位，固定px | no unit, is 'px'
  
+ hidden
  - 显示状态 | joystick hidden
  - 类型: Boolean | Type: Boolean
  - 默认值: false | Default: false

+ position
  - 摇杆的位置 | joystick position
  - 类型: String | Type: String
  - 默认值: 'left-bottom' | Default: 'left-bottom'
  - 可选值: 'left-bottom/right-bottom' | Options: 'left-bottom/right-bottom'

+ background
  - 自定义摇杆的底图 | custom joystick background image
  - 类型: String | Type: String
  - 支持网络路径 | support online path

+ joystick
  - 自定义摇杆的按钮图 | custom joystick button image
  - 类型: String | Type: String
  - 支持网络路径 | support online path

+ padding
  - 摇杆距操作区的内边距 | joystick padding in control area
  - 类型: Number | Type: Number
  - 默认值: 25 | Default: 25

### 事件 | Events
#### move()
+ 摇杆移动事件 | joystick move event: 
  - 回调参数 { moveForward: Boolean, moveBackward: Boolean, moveLeft: Boolean, moveRight: Boolean } | Backcall Param { moveForward: Boolean, moveBackward: Boolean, moveLeft: Boolean, moveRight: Boolean }

## 更新日志 | Log
|  版本号   | 更新时间  | 更新内容  |
|  ----  | ----  | ----  |
| 0.1.1  | 2020/6/23 | 修改组件名称 |
| 0.1.0  | 2020/6/23 | 初始化项目 |