# Eary's Spatial Expander（Mac 版）

**Eary's Spatial Expander** 是一款免费开源的**频域空间上混（Upmix）VST3 插件**：对输入信号做短时傅里叶变换（STFT），在频域内进行多级"幻象中置提取"（Phantom-Center Extraction），再把每一路结果各自做逆变换还原成音频。最终输出**最多 9 个独立空间声道 + LFE**，每个声道都是全频段、相干性完整的信号。

支持 **3.0 / 5.1 / 7.1 / 9.1** 输出格式（9.1 需要 9.1.6 总线容器，即使不启用高度声道），并会自动识别 DAW 的环绕总线布局。

**演示视频：**
- B 站：https://www.bilibili.com/video/BV1FMTC6GEpe
- YouTube：https://www.youtube.com/watch?v=maIZU22Czx4

> **本仓库说明**：这是原项目（EaryChow/Earys-Spatial-Expander）的 fork，在原基础上**新增了 macOS（Apple Silicon）构建**。原项目仅提供 Windows x64 版；本仓库的 Releases 提供可直接安装的 Mac 版 VST3，并附中文构建说明。

---

## 下载与安装（macOS）

**前置条件**：Mac 必须是 **Apple Silicon（M1/M2/M3/M4 系列）**。Intel 版暂未提供。

### 1. 下载

前往本仓库 **Releases** 页面，下载 `SpatialExpander_macOS_arm64_v1.0.0.zip`。

### 2. 解压

用 Mac 自带的**归档实用工具**双击 zip 解压（不要用微信/网盘的在线解压，可能破坏插件结构），得到 `SpatialExpander.vst3` 文件夹。

### 3. 安装（二选一）

```bash
# 方式 A：安装到系统级目录（所有用户可用，需要管理员密码）
sudo cp -R SpatialExpander.vst3 /Library/Audio/Plug-Ins/VST3/

# 方式 B：安装到用户级目录（仅当前用户，无需密码）
mkdir -p ~/Library/Audio/Plug-Ins/VST3
cp -R SpatialExpander.vst3 ~/Library/Audio/Plug-Ins/VST3/
```

### 4. ⚠️ 解除 Gatekeeper 隔离（下载版必做）

从网络下载的文件会被 macOS 标记 `com.apple.quarantine` 隔离属性，宿主会提示**"已损坏，无法打开"**或拒绝加载。在终端执行（把路径换成你的实际位置）：

```bash
xattr -dr com.apple.quarantine SpatialExpander.vst3
```

如果提示权限不足，加 `sudo`：

```bash
sudo xattr -dr com.apple.quarantine SpatialExpander.vst3
```

**验证是否生效**：执行 `xattr SpatialExpander.vst3`，如果输出为空，说明隔离标记已清除。

### 5. 重新扫描

在 DAW（Element / Reaper / Cubase / Studio One 等）里重新扫描插件或重启宿主，即可在 VST3 列表中找到 **Spatial Expander**。

> 如果之前安装过旧路径的插件导致列表里出现重复/失效条目，请在宿主里清除插件缓存后重新扫描。

---

## 使用说明

### 1. 插入插件

把 Spatial Expander 加载到包含立体声信号的环绕音轨或总线上（多数 VST 宿主要求输入/输出格式一致；节点式宿主如 Cantabile Lite 是例外）。

**输入格式**：选择 **Stereo、5.1 或 7.1** 匹配你的音源。
- **Stereo 模式**：只处理前两个声道（L/R）
- **5.1 / 7.1 模式**：先做后中置提取，再把所有床声道合成一个合成立体声场，再进入级联

可识别的输入格式：**Stereo、3.0、5.1、7.1、5.1.x、7.1.x、9.1.x**。选了 Stereo 模式但输入超过 2 声道时会给出警告。

### 2. 设置输出格式

打开插件界面，选择目标布局：

| 格式 | 空间声道 | 说明 |
|------|----------|------|
| **Auto** | 自动检测总线 | 自动读取总线布局 |
| **3.0** | Center、Front L、Front R | 最简上混 |
| **5.1** | + Rear L、Rear R | 经典环绕 |
| **7.1** | + Side L、Side R | 前后之间增加侧声道 |
| **9.1** | + Wide L、Wide R | 前与侧之间增加宽声道（需 9.1.6 总线容器） |

下拉框显示 **Auto（已检测格式）**，大部分宿主直接选 Auto 即可。Cantabile Lite 没有总线概念，Auto 会回退到 5.1，需要手动选输出格式。

### 3. 质量 / 延迟取舍

| 模式 | 延迟（样本） | 48 kHz 下延迟 |
|------|-------------|---------------|
| Low | 496 | **10.3 ms** |
| Low-Mid | 992 | **20.7 ms** |
| Mid | 1984 | **41.3 ms** |
| High | 3968 | **82.7 ms** |
| Bypass/Realtime | 0 | **0 ms** |

延迟越大 = FFT 越大 = 频率分辨率越高、提取越干净。多数离线 DAW 会自动补偿延迟，实时宿主（如 Cantabile Lite）不会，需要注意。

**Bypass/Realtime** 零延迟直通、CPU 占用极低，适合 A/B 对比。

> 播放中切换延迟、输出格式、5.1 环绕作为后置、串扰、拉伸或后置偏置，松手后会有约 50–100 ms 的短暂重新校准。

### 4. 节点式宿主路由

如果你的宿主支持多总线插件（如 Cantabile Lite），Spatial Expander 会暴露辅助输出总线：
**Front（立体声）、Center（单声道）、LFE（单声道）、Wide（立体声）、Side（立体声）、Rear（立体声）**，可以路由到独立音箱或外部处理器。

---

## 主要参数

### 输入格式
选择源床布局：**Stereo / 5.1 / 7.1**。决定插件如何解读输入声道。

### 输出格式
选择环绕床大小。推荐 **Auto**——自动检测总线布局并适配提取级联。

### 5.1 环绕作为 7.1/9.1 的后置
在 7.1/9.1 总线上使用 5.1 输出时，控制 5.1 环绕声道如何路由与校准：
- **勾选**：5.1 环绕作为**后置**声道，路由到总线后置输出，按 ITU-R BS.1770-4 后置权重（0 dB）校准
- **不勾选（默认）**：作为**侧置**声道，按侧置权重（+1.5 dB）校准

### 拉伸（Stretch）
控制立体声场在环绕环上的拉伸强度：
- **1.0（完全拉伸）**：硬边信号送到后置音箱，中间信号送侧/宽。默认值，即"环绕"听感
- **0.0（塌缩）**：所有提取的环绕内容折回前置 LCR，环绕音箱静音
- **中间值**：线性交叉淡化（如 7.1 下 0.5 时侧声道减半、能量分给 Center 和 Front L/R）

> 3.0 模式下 Stretch 无听感影响（没有环绕音箱可重映射），控件禁用。

### 泄漏中置（Leak Center）
把 **Center 音箱**能量重分配给 **Front L/R**，适合喜欢幻象中置而非独立中置音箱的场景，听流行歌推荐开启：
- **0.0**：真隔离中置，100% 中置信号留在 Center 音箱
- **1.0**：等能量 LCR，Center/Front L/Front R 各得等份
- **1.5**：中置减弱，约 75% 中置功率移到 Front L/R，Center 保留约 25%

### LFE 截止（LFE Cutoff）
LFE 低通滤波器截止频率。范围 **40–200 Hz**，默认 **80 Hz**。

### LFE 电平（LFE Level）
LFE 声道增益。范围 **−12 至 +12 dB**，默认 0 dB。拉到最左显示 "−inf dB" 即完全静音。

### 前置放大（Preamp）
输入增益。范围 **−6 至 +6 dB**，在 STFT 之前应用，用于给提取器做增益匹配。

### 后置偏置（Rear Bias）
控制后置声道如何导出：**0.0** 时后置与中置信号完全隔离，清晰度最高；**1.0** 时使用原始未提取残差，后置更饱满、更有包围感；默认 **0.5** 平滑混合。低值后置成像更干净，高值包围感更强。

### 串扰（Crosstalk）
把每个音箱的信号泄漏给相邻音箱，让声像移动更平滑。范围 **0.0–0.5**，默认 **0.1**。0 时声道完全独立。

### 每声道增益（Per-Channel Gain）
对每只音箱做独立的 ±12 dB 微调：Center、Front L/R、Wide L/R、Side L/R、Rear L/R。在自动校准**之后**应用，仅用于内容平衡。房间与音箱校准应在功放/声卡上做。

---

## 响度表

内置 **ITU-R BS.1770-4** 响度表：
- **瞬时（Momentary，400 ms）**
- **短时（Short-term，3 s）**及短时最大值
- **整体（Integrated）LUFS**

默认**暂停**状态。点 **Resume** 开始累积、**Pause** 暂停、**Refresh** 重置。

---

## 算法原理（简述）

1. **STFT 前端**：Kaiser-Bessel 窗（β=0.5）+ **96.875% 重叠**（hop = FFT/32），合成窗满足 COLA 完美重建约束
2. **幻象中置提取**：每个频点比较两路输入复频谱，用**最小幅度规则**提取相干"幻象中置"
3. **多层级联（纯频域）**：连续多层提取，层间**不转回时域**：第 1 层 (L,R)→Center；第 2 层分出 Front/Rear；第 3 层分出 Side；第 4 层分出 Wide，最终最多 9 声道 + LFE
4. **多声道输入路径（5.1/7.1）**：先提取后中置，再把床声道用恒定功率声像（constant-power panning）映射成合成立体声，进入既有级联；级联后把提取的后中置回注后置音箱
5. **相位相干恢复**：每个派生声道从原始父声道继承相位（L 系→L 相位，R 系→R 相位，Center 保持），防止环绕合奏时"飘相"
6. **空间拉伸**：Stretch 参数在频域内重映射提取声道到输出音箱
7. **自动校准**：启动及任何格式/延迟/拉伸/串扰/后置偏置变化时，用平坦频谱相关白噪声扫 0.1 dB ILD 步进，按 ITU-R BS.1770-4/BS.2051 感知权重测量输出功率，生成增益表 `G[ILD]`，让每个声像位置听感响度一致（不是物理等能量）。校准**不**归一化内容响度，只保证级联前后响度不变。无自动峰值限制器，削波时用 Preamp 调整输入增益
8. **LFE 提取**：(L+R)/2 经过**4 阶 Bessel-Thomson 低通**（两节级联双二阶），群延迟平坦；LFE 路径带延迟补偿线，与主声道样本对齐
9. **串扰与输出**：串扰在频域内、校准前应用，增益表自动补偿响度变化；iSTFT 后 Leak Center 重分配 Center 到 Front L/R；声道按标准类型标识（Left/Right/Centre/LFE/Side/Rear/Wide）映射到 DAW 总线

---

## 构建

- **macOS（本 fork 新增）**：见 [BUILD_MACOS.md](BUILD_MACOS.md)（JUCE 8.0.13 + CMake + Ninja，Apple Silicon）
- **Windows**：JUCE 项目 + Visual Studio 2022。用 Projucer 打开 `*.jucer` 添加 JUCE 依赖并导出，VS2022 中切 Release 构建 `SpatialExpander_VST3`，产物在 `Builds\VisualStudio2022\x64\Release\VST3\`。作者仅在 Windows x64 上验证过

---

## 致谢与来源

本插件受作者在 [Cantabile Lite](https://www.cantabilesoftware.com/free-vst-host) 中级联 [Bertom Audio Phantom Center](https://bertomaudio.com/phantom-center.html) 的体验启发。单层幻象中置提取是成熟技术；多层级联与空间拉伸概念为该项目原创。项目由大规模迭代式 LLM 辅助开发，作者主导架构并进行了多轮设计与实现审查。
