# huaguang_tools

[![Python](https://img.shields.io/badge/Python-3.7%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.x-5C3EE8?logo=opencv&logoColor=white)](https://opencv.org/)

做视觉项目时反复手写的那些脏活——切视频、抽帧、改文件名、转标注格式、清无效样本、
pth 转 ONNX 再转 Paddle Lite——攒成了这一套小脚本。每个脚本都是**单文件、零配置、直接
`python xxx.py` 就能跑**，复制到你自己的工程里也毫无依赖负担。

来自华光人工智能教育机器人团队时期的开发实践（[control_scnu](https://github.com/HuaGuangEdu/control_scnu)、
[blocklyFiles](https://github.com/HuaGuangEdu/blocklyFiles) 的配套工具）。

---

## 脚本一览

### 视频与图像 I/O

| 脚本 | 作用 | 说明 |
| --- | --- | --- |
| `take_video.py` | 摄像头录制视频 | XVID 编码，30 fps，640×480，画面自动镜像 |
| `video2img.py` | 视频批量切帧 | 遍历 `./myvideo2/` 下所有视频，逐帧写到 `./mypic/` |
| `video2pic.py` | 按时间点抽帧 | 支持指定起止时间、时间间隔、输出前缀，也可只打印视频信息 |
| `pic2video.py` | 图片序列合成视频 | 按文件名数字序排序后合成，适合做演示片 |
| `fps.py` | 实时帧率测量 | 每 0.1 s 统计一次摄像头帧率，用于验证设备实际性能 |

### 数据集构建与清洗

| 脚本 | 作用 | 说明 |
| --- | --- | --- |
| `getpos.py` | 鼠标取点 | 在图片上点击取坐标，用于标定 ROI 或核对标注框 |
| `clear_useless.py` | 清理无效样本 | 删除没有对应 XML 标注的图片，避免训练时报错 |
| `train_test_split.py` | 划分训练/验证集 | 按比例随机划分并拷贝到目标目录，保持图-标注同名配对 |
| `rename_all.py` | 批量重命名 | 从指定起始编号开始连续命名，统一数据集命名规范 |
| `change_filenames.py` | 批量重命名（连续序号） | 直接把目录内文件重排为 `0.jpg`、`1.jpg`… |
| `voc2coco.py` | VOC → COCO | 把 Pascal VOC 的 XML 标注转成 COCO 的 `instances_*.json`，省去手写转换 |
| `getdataset.py` | 自动采集数据集 | 依赖 `control` 包（`shijue1` 模块），按颜色阈值自动抓取样本 |

### 颜色与视觉调试

| 脚本 | 作用 | 说明 |
| --- | --- | --- |
| `hsvTiaoshi.py` | HSV 阈值调试 | 拖动滑动条实时预览 HSV 上下界，调完直接抄数值 |
| `HSVpro.py` | HSV 颜色分割 | 带跨平台延时适配（Windows 1 ms / Linux 40 ms）、警告抑制的完整实现 |

### 模型转换与部署

| 脚本 | 作用 | 说明 |
| --- | --- | --- |
| `pth2onnx.py` | PyTorch → ONNX | 固定输入尺寸导出 ONNX，便于接 OpenVINO / ONNX Runtime |
| `convert2paddlelite.py` | Paddle → Paddle Lite | 导出 `naive_buffer` 格式、目标平台 `arm`，直接扔进树莓派跑 |

### 其他

| 脚本 | 作用 |
| --- | --- |
| `logger.py` | 一个同时把输出写到终端和文件的 `Logger` 类，支持 `with` 语句 |

> `check.py`、`resizeall.py` 目前是占位文件，功能待补。

---

## 使用方式

绝大多数脚本把**路径写死在文件开头**，用之前改两行就行：

```python
# 例：rename_all.py
path = r'Q:\datasets\custom\footfinal\images/'   # 改成你的目录
count = 1000                                     # 起始编号
```

然后直接运行：

```bash
python rename_all.py
```

### 典型工作流

```bash
# 1. 录一段素材
python take_video.py

# 2. 切成图片
python video2img.py

# 3. 标注完，统一命名 + 清理漏标样本
python change_filenames.py
python clear_useless.py

# 4. VOC 标注转 COCO
python voc2coco.py

# 5. 划分数据集
python train_test_split.py

# 6. 训练完，模型转 ONNX → Paddle Lite，部署到树莓派
python pth2onnx.py
python convert2paddlelite.py
```

## 依赖

```bash
pip install opencv-python numpy scikit-learn
# 仅模型转换相关脚本需要：
pip install torch onnx paddlelite
```

`getdataset.py` 额外依赖团队内部的 `control` 包，单独使用时需要替换为等价的取图逻辑。

## 注意

- 这些脚本面向**一次性任务**，为了短小直接牺牲了通用性：路径写死、参数不解析、失败不重试。
  要长期复用，建议先包成函数再加 `argparse`。
- `video2img.py` 会把目标目录当成空目录直接写入，跑之前确认输出目录里没有要保留的文件。
- 标注转换（`voc2coco.py`）默认按「图片与 XML 同名」配对，命名不一致时先跑重命名脚本。

## 相关仓库

- [ReID_using-openvino](https://github.com/yiyiliu0415/ReID_using-openvino) —— 用 OpenVINO 做行人重识别检索
- [Goal-line-technology](https://github.com/yiyiliu0415/Goal-line-technology) —— 足球门线判定
