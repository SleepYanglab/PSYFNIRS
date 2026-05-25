# PSYFNIRS

fNIRS 数据仓库 — 院内 **CRB35** 近红外成像仪采集，任务：视频观看期间大脑激活。

## 数据格式

文件扩展名为 `.bin`，实为 **Homer2 `.nirs` 格式**（MATLAB 5.0 MAT 文件）。  
可用 `scipy.io.loadmat(filepath, appendmat=False)` 直接读取。

| 变量 | 说明 |
|------|------|
| `d` | 原始光强矩阵 `(n_times, n_channels)` |
| `t` | 时间轴 `(n_times,)`，单位 s |
| `ml` | 通道列表 `(n_channels, 4)`: `[src, det, type, wl_idx]`（**第4列为波长索引**：1=690nm, 2=830nm） |
| `s` | 刺激触发 `(n_times, n_stim)` |
| `aux` | 辅助通道 `(n_times, 8)` |
| `SD` | Source-Detector 配置，含 `Lambda`（波长）、`SrcPos`、`DetPos` |

## 目录结构

```
PSYFNIRS/
├── 0/                        # 被试 0
│   ├── 20230424_0053_03.bin
│   └── 20230424_0549_01.bin
├── 1/                        # 被试 1
│   └── 20230424_0053_03.bin
└── notebooks/
    └── qc_analysis.ipynb     # 数据质量检查 Notebook
```

## 采集参数

| 参数 | 值 |
|------|----|
| 设备 | CRB35 |
| 采样率 | 50 Hz |
| 波长 | 690 nm / 830 nm |
| 通道数 | 70（35 S-D 对 × 2 波长） |
| 单次时长 | ~40 分钟 |

## 数据质量概览（QC）

基于 Scalp Coupling Index（SCI ≥ 0.75 为好道）：

| 被试 | 文件 | 好道/总对 | 平均 SCI | 备注 |
|------|------|-----------|----------|------|
| Sub-0 | 20230424_0053_03.bin | 21/35 (60%) | 0.632 | 中等质量 |
| Sub-0 | 20230424_0549_01.bin | 3/35 (9%)  | 0.172 | **质量差，慎用** |
| Sub-1 | 20230424_0053_03.bin | 21/35 (60%) | 0.632 | 中等质量 |

详细 QC 报告见 `notebooks/qc_analysis.ipynb`。

## 环境配置

本项目使用 [uv](https://docs.astral.sh/uv/) 管理 Python 环境。

```bash
# 激活虚拟环境（Windows）
.venv\Scripts\activate.bat

# 或用 uv 直接运行
uv run jupyter lab
```

主要依赖：`scipy` · `numpy` · `matplotlib` · `pandas` · `mne` · `mne-nirs` · `jupyter`

## 快速开始

```python
import scipy.io
import numpy as np

mat = scipy.io.loadmat('0/20230424_0053_03.bin', appendmat=False)
d   = mat['d']          # (120600, 70) — 原始光强
t   = mat['t'].flatten()  # 时间轴（秒）
ml  = mat['ml']         # 通道描述，第4列为波长索引（1 or 2）

print(f"采样率: {1/np.median(np.diff(t)):.1f} Hz")
print(f"时长: {t[-1]/60:.1f} min")
print(f"通道数: {d.shape[1]}")
```
