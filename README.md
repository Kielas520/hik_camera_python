# HikCamera Python

本项目提供了一个基于 Python 的海康机器人（Hikrobot）工业相机调用封装。通过结合海康 MVS SDK 与 OpenCV，实现了设备的快速枚举、图像拉流、实时曝光控制以及 Bayer 格式到 RGB 的自动转换。
---

## 核心特性

* **面向对象封装**：通过 `HikCamera` 类简化了 MVS SDK 繁琐的句柄创建、设备打开和取流流程。
* **OpenCV 深度集成**：自动将相机的原始 Buffer 数据转换为 NumPy 数组，并处理基础的色彩空间转换（如 BayerRG 到 RGB），方便直接接入视觉算法。
* **动态参数控制**：支持在取流过程中实时调整相机的曝光时间，支持手动与自动曝光模式切换。
* **安全的资源管理**：内置 `__del__` 方法与 `close` 方法，确保异常退出时能够正确停止抓图并销毁句柄。

---

## 目录结构要求

在运行本项目之前，需要将海康机器人的 MVS SDK 相关依赖项放入指定的同级目录中。项目的预期结构如下：

```text
hik_camera_python/
├── hik_lib/               # 需手动放入：海康 MVS SDK 的底层 DLL 运行库
├── MvImport/              # 需手动放入：海康提供的 Python API (如 MvCameraControl_class.py 等)
├── src/
│   └── hik_camera.py      # 相机核心封装代码
├── main.py                # 示例运行脚本
└── pyproject.toml         # 项目依赖配置文件
```

link : https://www.hikrobotics.com/cn/machinevision/service/download/?module=0

hik_lib/ -> C:\Program Files (x86)\Common Files\MVS\Runtime\Win64_x64

MV_Import -> C:\Program Files (x86)\MVS\Development\Samples\Python\MvImport


> **注意**：`hik_lib` 和 `MvImport` 目录不包含在源码中，你需要从海康机器人的官方 MVS 客户端安装路径下提取这些库文件和 Python 接口文件。

---

## 环境依赖

* **Python 版本**：严格要求 `==3.10.*`
* **第三方库**：`opencv-python >= 4.13.0.92`，`numpy`

### 安装步骤

1. 克隆本仓库：
   ```bash
   git clone https://github.com/Kielas520/hik_camera_python.git
   cd hik_camera_python
   ```

2. 安装依赖：
   由于项目使用了 `pyproject.toml` 进行依赖管理，你可以使用 `pip` 进行安装：
   ```bash
   pip install .
   ```
   或者直接安装基础依赖：
   ```bash
   pip install opencv-python numpy
   ```

---

## 快速使用

项目中提供了一个即插即用的示例脚本 `main.py`。

### 运行示例

将海康工业相机连接至电脑（确保 IP 配置正确或 USB 连接正常），然后运行：

```bash
python main.py
```

### 快捷键操作

在弹出的 OpenCV 图像窗口中，支持以下键盘交互：

| 按键 | 功能说明 |
| :--- | :--- |
| **W** | 增加曝光时间（每次增加 1000us / 1ms） |
| **S** | 减少曝光时间（每次减少 1000us / 1ms，最低限制为 100us） |
| **Q** | 安全退出程序并关闭相机 |

---

## API 简要说明

如果你需要在自己的代码中集成该相机模块，可以参考以下 API：

```python
from src.hik_camera import HikCamera

# 1. 实例化相机对象（默认连接索引为 0 的设备）
cap = HikCamera(device_index=0)

# 2. 打开设备并开始取流
if cap.open():

    # 3. 设置曝光 (单位: 微秒)
    cap.set_exposure(10000, auto_mode=False)

    # 4. 获取当前曝光
    current_exposure = cap.get_exposure()

    # 5. 读取一帧图像
    # ret 为布尔值，frame 为转换后的 numpy 数组格式图像
    ret, frame = cap.read()

    # 6. 释放资源
    cap.close()
```
