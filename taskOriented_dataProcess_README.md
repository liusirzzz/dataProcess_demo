# Global Ocean Surface Velocity (u,v) WebDataset

## Overview
本数据集面向海洋表面流速场的超分辨率重建（Super-Resolution）与相关回归任务。数据来自 Copernicus Marine（CMEMS）全球海洋的表层日数据，处理为 `448x448` 的东向（u/uo）与北向（v/vo）速度补丁。

## Data Source
Source HDF5: /data/uovo_data/processed/uovo_1997-01-01_to_1997-12-31.h5，具体信息可查看/data/copernicus_uv_data/README.md

## Directory Structure
WebDataset 分片目录（建议）：
```text
/data/OceanSR/copernicus_uv_data/
├── train/                      # 训练集
│   ├── 1997-01-01.tar
│   ├── 1997-01-11.tar
│   └── ...
├── val/                        # 验证集
│   ├── 1997-09-08.tar
│   └── ...
├── test/                       # 测试集
│   ├── 1997-11-17.tar
│   └── ...
├── README.md
└── scripts/
    ├── pack_shards.py          # HDF5 -> WebDataset 分片打包脚本
    ├── example.ipynb           # 读取/统计/可视化示例
    ├── stats.json              # train/val/test 统计信息
    ├── uv_distribution.png     # train/val/test 分布图
    └── WebDataset_dataFlow.png # WebDataset数据流示例图
```

分片文件名为该分片覆盖的首日日期（例如 `1997-01-01.tar`），分片内样本按时间顺序连续编号（9 位 key）。

## Processing Pipeline
1) 生成低质量样本 LQ：对 GT 做 s 倍（默认 4）bicubic下采样后bicubic上采样回原尺寸（NaN-aware，保持与 GT 同步的 NaN 掩码）。  
2) 将同一天的样本按时间递增 9 位 key 编号，按 `days_per_shard`（默认 10 天）打成一个 `.tar` 分片。  
3) 按时间将分片划分为 `train/val/test`（示例：1-10 月为 train，11 月为 val，12 月为 test）。  
注：`pack_shards.py` 提供从 HDF5 到 WebDataset 的自动化打包与参数化控制。

## Tar File Structure
每个 `.tar` 内按 9 位 key 组织样本，包含：
- `000000123.gt.npy`     float32, shape `[2,H,W]`：高分辨率 GT `[u,v]`（海洋为数值、陆地为 NaN）
- `000000123.lq.npy`     float32, shape `[2,H,W]`：从 GT 生成的低质量版本（下采样后上采样），NaN 与 GT 对齐
- `000000123.lon.npy`    float32, shape `[H,W]`：经度（度）
- `000000123.lat.npy`    float32, shape `[H,W]`：纬度（度）
- `000000123.mask.npy`   bool,    shape `[H,W]`：1=Land，0=Ocean
- `000000123.date.txt`   UTF-8 文本日期 `YYYY-MM-DD`  
命名规则：分片名为分片首日日期；分片内 key 递增且时间连续。

Notes
- NaN 语义：`gt` 与 `lq` 中的 NaN 表示陆地；`lq` 的 NaN 掩码与 `gt` 完全一致。  
- 可视化建议对经度做 unwrap（跨 180° 经线）以获得连续坐标。  
- 训练时通常忽略 NaN 区域损失（可结合 `mask` 约束）。

## Data Statistics
全局统计（基于源数据处理结果，忽略 NaN），单位 m/s：
| Statistic         | uo (Eastward)           | vo (Northward)          |
| :---------------- | :----------------------- | :---------------------- |
| Mean              | 0.0063662025596672046    | 0.006140309626745585    |
| Standard Dev.     | 0.18706705950830543      | 0.1470535952269027      |
| Min               | -1.9061861000955105      | -2.2461622953414917     |
| Max               | 2.633136995136738        | 2.2211371175944805      |

总有效patch（陆地占比 <80%）：53655

仓库提供基于 WebDataset 的分割统计与可视化：
- `stats.json`：对 `train/val/test` 分别计算 `u,v` 的均值与标准差（忽略 NaN/Inf）
- `uv_distribution.png`：`train/val/test`各自的`u,v`分布图  
以上由 `scripts/example.ipynb` 一键生成。

使用与验证可参考 `scripts/example.ipynb`（DataLoader、统计、直方图与样本可视化）。  