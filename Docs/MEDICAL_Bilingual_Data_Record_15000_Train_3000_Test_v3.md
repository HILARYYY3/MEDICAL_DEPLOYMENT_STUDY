# MEDICAL 数据处理与数据集配置记录（中英双语）
# MEDICAL Data Processing and Dataset Configuration Record (Bilingual)

**记录版本 / Record version:** v3.0  
**记录日期 / Record date:** 2026-08-30  
**项目 / Project:** YOLO-CXR reproduction on VinDr-CXR / VinBigData  
**记录状态 / Record status:** 最新实验定义，已纳入 14 类标签重建程序 / Current experimental definition with the 14-class label-reconstruction pipeline incorporated
**替代版本 / Supersedes:** v2.0

---

## 1. 当前实验定义 / Current Experimental Definition

| 项目 | 中文记录 | English record |
|---|---|---|
| 任务 | 胸部 X 光多异常目标检测 | Multi-abnormality object detection on chest X-rays |
| 数据来源 | VinDr-CXR / Kaggle VinBigData Chest X-ray Abnormalities Detection | VinDr-CXR / Kaggle VinBigData Chest X-ray Abnormalities Detection |
| 官方训练集 | 15,000 张；目前全部作为训练数据使用 | 15,000 images; all are currently used as training data |
| 官方测试集 | 3,000 张；作为独立测试集，不进入训练 | 3,000 images; retained as an independent test set and excluded from training |
| 当前检测类别 | 14 类胸片异常 | 14 thoracic abnormality classes |
| No finding | 作为阴性/背景图像保留；YOLO 标签文件为空，不作为第 15 个框类别 | Retained as negative/background images; represented by empty YOLO label files rather than a 15th box class |
| 单类别模式 | 关闭，`single_cls=False` | Disabled, `single_cls=False` |
| 训练输入尺寸 | YOLO 训练阶段使用 `imgsz=640` | YOLO uses `imgsz=640` during training |

### 当前数据组织方式 / Current Data Organization

15,000 张图片没有因为合并训练集而被物理移动。原来的 `images/train`、`images/val` 和 `images/test` 目录可以继续保留；`train_15000.txt` 通过路径清单统一引用三个目录中的全部 15,000 张图片。因此，“全部用于训练”是**逻辑训练集合并**，不是磁盘文件夹合并。

The 15,000 images were not physically moved when the training set was consolidated. The original `images/train`, `images/val`, and `images/test` directories may remain unchanged. Instead, `train_15000.txt` references all 15,000 images across the three directories. Therefore, “all images used for training” means a **logical consolidation through a path list**, not a physical folder merge.

当前本地数据根目录记录为：

```text
I:\MEDICAL\数据集\345410650\vindr_nodule_mass_cleaning_final_v6\b68b716ab287\policy-0aa11e41e90a\dataset
```

> 注意 / Note: 目录名称仍包含 `nodule_mass`，这是历史输出名称，不代表当前实验仍是单类别任务。

---

## 2. 14 类检测映射 / Fourteen-Class Detection Mapping

| Class ID | Class name |
|---:|---|
| 0 | Aortic enlargement |
| 1 | Atelectasis |
| 2 | Calcification |
| 3 | Cardiomegaly |
| 4 | Consolidation |
| 5 | ILD |
| 6 | Infiltration |
| 7 | Lung Opacity |
| 8 | Nodule/Mass |
| 9 | Other lesion |
| 10 | Pleural effusion |
| 11 | Pleural thickening |
| 12 | Pneumothorax |
| 13 | Pulmonary fibrosis |

该映射已在 `vindr_all_classes_labels_only.ipynb` 中通过 `CLASS_NAMES` 和自动生成的 `CLASS_TO_ID` 固定。程序会拒绝原始 CSV 中未配置的异常类别，避免类别名称或编号被静默改变。

This mapping is fixed by `CLASS_NAMES` and the automatically generated `CLASS_TO_ID` dictionary in `vindr_all_classes_labels_only.ipynb`. The program rejects any unconfigured abnormality class found in the source CSV, preventing silent changes to class names or IDs.

---

## 3. 影像预处理记录 / Image Preprocessing Record

以下参数从 `640-preprocessing-final(1).ipynb` 的实际代码提取。

The following parameters were extracted from the actual code in `640-preprocessing-final(1).ipynb`.

| 参数 / Parameter | 中文记录 | English record | 实际值 / Value |
|---|---|---|---|
| 输入格式 / Input format | 读取二维 DICOM 胸片 | Read two-dimensional DICOM chest radiographs | 2D DICOM only |
| 预处理设备 / Processing device | 仅使用 CPU | CPU-only preprocessing | CPU |
| 随机种子 / Random seed | 固定随机种子 | Fixed random seed | `2026` |
| Pixel Padding | 从强度统计中排除 Pixel Padding；输出中对应像素设为 0 | Exclude Pixel Padding from intensity statistics and set corresponding output pixels to 0 | Enabled when metadata exists |
| Modality LUT | 首先应用 DICOM Modality LUT | Apply the DICOM Modality LUT first | Enabled |
| VOI LUT | 若 DICOM 包含 VOI LUT 或 Window Center/Width，则尝试应用 | Attempt VOI LUT when VOI LUT or Window Center/Width metadata is present | Conditional |
| 灰度极性 / Photometric interpretation | 只接受 `MONOCHROME1` 或 `MONOCHROME2` | Accept only `MONOCHROME1` or `MONOCHROME2` | Required |
| 反转 / Inversion | 对 `MONOCHROME1` 条件反转 | Conditionally invert `MONOCHROME1` images | Conditional |
| 无 VOI 时窗口 / Fallback window | 没有成功应用 VOI 时，用有效像素的 0.5–99.5 百分位 | Use the 0.5th–99.5th percentiles of valid pixels when VOI is unavailable or fails | `0.5`, `99.5` |
| 有 VOI 时窗口 / Window after VOI | VOI 成功时使用有效像素最小值和最大值 | Use the minimum and maximum valid values after successful VOI application | Min–max |
| 强度裁剪 / Intensity clipping | 将像素裁剪到选定窗口 | Clip pixel values to the selected intensity range | Enabled |
| 强度归一化 / Intensity normalization | 线性映射到 0–255，四舍五入并转换为 8 位无符号整数 | Linearly map to 0–255, round, and convert to unsigned 8-bit integers | `uint8` |
| 输出颜色模式 / Output mode | 单通道灰度 | Single-channel grayscale | PIL mode `L` |
| 输出格式 / Output format | PNG | PNG output | `.png` |
| PNG 压缩 / PNG compression | 使用 PNG 压缩等级 6 | Use PNG compression level 6 | `6` |
| 离线尺寸 / Offline size | 不是固定 640×640；最长边超过 1024 时才缩小到 1024 | Not fixed at 640×640; downscale only when the longest side exceeds 1024 pixels | `PREPROCESS_MAX_SIDE=1024` |
| 长宽比 / Aspect ratio | 保持原始长宽比 | Preserve the original aspect ratio | Preserved |
| 重采样 / Resampling | 缩小时使用 Lanczos | Use Lanczos interpolation for downscaling | `LANCZOS` |
| 填充 / Offline padding | 预处理输出阶段不补成正方形 | No square padding during offline preprocessing | None |
| 训练阶段缩放 / Training-time resizing | YOLO 在训练时按照 `imgsz=640` 等比例缩放并补边 | YOLO performs aspect-ratio-preserving resize and letterboxing at `imgsz=640` during training | `640` |
| 并发 / Preprocessing workers | 正式预处理单张串行，降低高分辨率 DICOM 的内存占用 | Process one image at a time to reduce memory use for high-resolution DICOM files | `1` worker |
| 垃圾回收 / Garbage collection | 每 25 张执行一次清理 | Run garbage collection every 25 images | `25` |
| 进度保存 / Progress checkpoint | 每 100 张保存一次进度 | Save progress every 100 images | `100` |
| 断点续跑 / Resume | 允许复用已完成且校验通过的输出 | Reuse previously completed and validated outputs | `True` |
| 错误策略 / Error policy | 正式运行遇到预处理错误时失败并停止 | Fail and stop on preprocessing errors during the formal run | `True` |

### 已执行结果 / Recorded Execution Result

所附 notebook 的输出记录显示：发现 15,000 张官方训练 DICOM；预处理完成 15,000/15,000，失败数为 0。该次执行复用了上一版兼容输出，因此记录为新增处理 0、跳过/导入已完成结果 15,000。

The attached notebook output records 15,000 official training DICOM images. Preprocessing completed for 15,000/15,000 images with zero failures. This run reused a compatible previous output, so it recorded zero newly processed images and 15,000 skipped/imported completed results.

---

## 4. 14 类标签重建与框处理记录 / Fourteen-Class Label Reconstruction and Box Processing

本节来自 `vindr_all_classes_labels_only.ipynb`。该程序只重新生成标签，不复制、缩放、裁剪、删除或重新编码现有 15,000 张预处理图片。

This section is based on `vindr_all_classes_labels_only.ipynb`. The program regenerates labels only; it does not copy, resize, crop, delete, or re-encode the existing 15,000 preprocessed images.

### 数据来源与输出 / Sources and Outputs

| 项目 / Item | 中文记录 | English record |
|---|---|---|
| 标注来源 / Annotation source | Kaggle 官方原始 `train.csv` | Original official Kaggle `train.csv` |
| 图片—标注连接键 / Image–annotation key | 旧清洗数据集 `manifest.csv` 中唯一的 `image_id` | Unique `image_id` in the cleaned dataset's `manifest.csv` |
| 图片修改 / Image modification | 否 | No |
| 标签修改 / Label modification | 是；从单类标签重建为 14 类 YOLO TXT | Yes; rebuild Nodule/Mass-only labels as 14-class YOLO TXT files |
| No finding | 不生成检测框；生成同名空 TXT | Does not generate a detection box; produces a matching empty TXT file |
| 输出根目录 / Output root | `/kaggle/working/vindr_all_classes_labels_v1` | `/kaggle/working/vindr_all_classes_labels_v1` |
| 标签目录 / Label directories | `labels/train`、`labels/val`、`labels/test` | `labels/train`, `labels/val`, and `labels/test` |
| 主要配置 / Main configuration | `run_config.json`、`data.yaml` | `run_config.json` and `data.yaml` |
| 打包结果 / Packaged output | `vindr_all_classes_labels_v1.zip` | `vindr_all_classes_labels_v1.zip` |

### 原始框清洗规则 / Source-Box Cleaning Rules

1. 仅保留 14 个异常类别；`No finding` 行不生成框。  
   Retain only the 14 abnormality classes; `No finding` rows do not generate boxes.
2. 将坐标转换为数值并删除非有限坐标。  
   Convert coordinates to numeric values and drop non-finite coordinates.
3. 删除不满足 `x_max>x_min` 或 `y_max>y_min` 的框。  
   Drop boxes that do not satisfy `x_max>x_min` or `y_max>y_min`.
4. 将越界坐标裁剪到原始图像边界，裁剪记录写入问题报告但继续保留。  
   Clip out-of-bound coordinates to the original image boundary; record the clipping in the issue report while retaining the box.
5. 裁剪后宽或高小于 2 像素的框被删除。  
   Drop boxes whose width or height is below 2 pixels after clipping.
6. 按 `image_id + class_name + rad_id + 四个坐标` 删除完全重复框。  
   Remove exact duplicate boxes using `image_id + class_name + rad_id + four coordinates`.
7. 原始 CSV 中不属于当前 15,000 张 manifest 的标注不会进入标签，并单独报告。  
   Annotations whose image IDs are outside the current 15,000-image manifest are excluded and separately reported.

### 多医生标注融合 / Multi-Radiologist Annotation Fusion

| 参数 / Parameter | 中文记录 | English record | 实际值 / Value |
|---|---|---|---|
| 融合分组 / Fusion group | 同一图像且同一类别 | Same image and same class | `image_id + class_name` |
| IoU 阈值 / IoU threshold | 当前框与病灶簇均值框的 IoU 达到阈值时允许加入 | A box may join a lesion cluster when its IoU with the cluster mean box reaches the threshold | `0.50` |
| 医生约束 / Radiologist constraint | 同一名医生的第二个框不能加入同一病灶簇 | A second box from the same radiologist cannot join the same lesion cluster | Enforced |
| 多候选选择 / Multiple-candidate selection | 若可加入多个病灶簇，选择 IoU 最大者 | If multiple clusters are eligible, select the one with the highest IoU | Maximum IoU |
| 融合坐标 / Fused coordinates | 对病灶簇内所有框取坐标算术均值 | Use the arithmetic mean of all box coordinates in the lesion cluster | Mean coordinates |
| 最低医生共识 / Minimum consensus | 至少一名医生；单医生框保留 | At least one radiologist; single-radiologist boxes are retained | `1` |
| 低共识复核 / Low-consensus review | 单医生融合框写入专门复核表 | Write single-radiologist fused boxes to a dedicated review table | `single_radiologist_box_review.csv` |
| 标签精度 / Label precision | YOLO 坐标保存 8 位小数 | Store YOLO coordinates with eight decimal places | `8` |

这套 14 类算法与旧 Nodule/Mass notebook 不同：旧程序使用 IoU 0.35 和坐标中位数；最新版标签程序使用 IoU 0.50、不同医生约束和坐标均值。当前 14 类记录应以后者为准。

This 14-class algorithm differs from the historical Nodule/Mass notebook: the old program used IoU 0.35 and coordinate medians, whereas the latest label program uses IoU 0.50, a cross-radiologist constraint, and coordinate means. The current 14-class record therefore follows the latter.

### YOLO 标签生成 / YOLO Label Generation

- 格式：`class_id x_center y_center width height`。  
  Format: `class_id x_center y_center width height`.
- 坐标使用原始 DICOM 宽高归一化。  
  Coordinates are normalized using the original DICOM width and height.
- 因为 PNG 只进行等比例缩放且无裁剪、无离线填充，原图归一化坐标可以直接用于预处理 PNG。  
  Because the PNG files undergo aspect-ratio-preserving resizing without cropping or offline padding, coordinates normalized against the original dimensions remain valid for the preprocessed PNG files.
- 程序要求预处理前后相对长宽比误差不超过 0.002。  
  The program requires the relative aspect-ratio error between original and preprocessed images to be no greater than 0.002.
- 每张 manifest 图片必须生成一个同名标签文件，包括无框阴性图。  
  Every image in the manifest must receive a matching label file, including negative images without boxes.

### 标签程序自动审计 / Automatic Label Audit

程序只有在以下检查全部通过时才写出 `STATUS: LABELS_READY`：

The program writes `STATUS: LABELS_READY` only after all of the following checks pass:

- 每个 split 的标签文件 ID 集合与 manifest 完全一致；  
  The label-file ID set for each split exactly matches the manifest.
- 每行标签恰好包含 5 列；  
  Every label line contains exactly five columns.
- `class_id` 位于 0–13；  
  Every `class_id` falls within 0–13.
- 中心坐标位于 `[0,1]`，宽高位于 `(0,1]`；  
  Center coordinates fall within `[0,1]`, and width/height fall within `(0,1]`.
- TXT 中解析出的总框数与融合框表总数一致；  
  The total parsed TXT box count equals the fused-box table count.
- 标签文件数量与 manifest 图片数量一致。  
  The number of label files equals the number of images in the manifest.

所附 notebook 是未执行的代码副本，所有代码单元的 `execution_count` 均为空。因此，本记录确认的是**标签生成规则与审计标准**；实际类别分布、阳性图数量、阴性图数量、融合框总数和单医生框总数应从成功运行后生成的 `generation_summary.json`、`final_class_distribution.csv` 与 `final_split_label_distribution.csv` 填入实验结果记录。

The attached notebook is an unexecuted code copy: all code cells have an empty `execution_count`. This record therefore verifies the **label-generation rules and audit criteria**. The actual class distribution, positive-image count, negative-image count, fused-box count, and single-radiologist-box count must be taken from `generation_summary.json`, `final_class_distribution.csv`, and `final_split_label_distribution.csv` after a successful run.

---

## 5. 数据清洗与审计记录 / Data Cleaning and Audit Record

以下设置来自 `only-datacleaning-final(2).ipynb` V6。

The following settings come from V6 in `only-datacleaning-final(2).ipynb`.

| 项目 | 中文记录 | English record | 设置 / Setting |
|---|---|---|---|
| 审计模式 / Audit mode | 对正式数据执行全量审计 | Run a full audit on the formal dataset | `full` |
| 灰度要求 / Grayscale requirement | 所有输出图必须为灰度图 | All exported images must be grayscale | `True` |
| 文件哈希 / File hash | 计算文件 SHA-256 | Compute file-level SHA-256 | Enabled |
| 像素哈希 / Pixel hash | 计算解码灰度像素 SHA-256 | Compute SHA-256 over decoded grayscale pixels | Enabled |
| 复制校验 / Copy verification | 已存在的复制文件也验证哈希 | Verify hashes of existing copied files | Enabled |
| 自动精确去重 / Automatic exact deduplication | 关闭；相同像素且标签一致的副本只报告、不自动删除 | Disabled; exact pixel duplicates with identical labels are reported but not automatically removed | `False` |
| 标签冲突重复 / Duplicate label conflict | 相同像素但标签冲突时隔离整组并阻断训练 | Quarantine the entire group and block training when identical pixels have conflicting labels | Blocking |
| 感知哈希近重复 / Perceptual near-duplicates | 只生成复核报告，不自动删除 | Report for review only; do not automatically delete | Report only |
| 低对比度阈值 / Low-contrast threshold | 灰度标准差低于 3.0 时进入复核报告 | Add images with grayscale standard deviation below 3.0 to the review report | `3.0` |
| 极端饱和阈值 / Extreme-saturation threshold | 饱和像素比例达到 0.98 时进入复核报告 | Add images with a saturation fraction of 0.98 or above to the review report | `0.98` |
| 无效图像自动隔离 / Invalid-image quarantine | 无法读取或结构无效的图像允许自动隔离 | Automatically quarantine unreadable or structurally invalid images | Enabled |
| 自动隔离总比例上限 / Overall quarantine limit | 自动隔离图像比例不得超过 0.5% | Automatically quarantined images must not exceed 0.5% | `0.005` |
| 阳性隔离比例上限 / Positive quarantine limit | 自动隔离阳性图像比例不得超过 1% | Automatically quarantined positive images must not exceed 1% | `0.01` |
| 导出方式 / Export mode | 复制图像到正式输出目录 | Copy images into the formal output directory | `copy` |
| 训练闸门 / Training gate | 来源、图像、哈希、标签、几何、重复和完整性检查全部通过后才允许训练 | Permit training only after provenance, image, hash, label, geometry, duplicate, and completeness checks all pass | `TRAINING_READY=True` only after all checks |

---

## 6. 原 notebook 划分与当前实验划分的区别 / Difference Between the Notebook Split and the Current Experiment

| 项目 | 原 notebook / Original notebook | 当前实验 / Current experiment |
|---|---|---|
| 15,000 张内部划分 | 12,000 train + 1,500 val + 1,500 test | 15,000 张全部进入训练清单 / all 15,000 images are included in the training list |
| 类别 | Nodule/Mass 单类 / Nodule/Mass only | 14 类 / 14 classes |
| 原内部 val/test | 固定保留 / permanently retained | 不再作为独立的 1,500/1,500 实验集合 / no longer treated as independent 1,500/1,500 experimental subsets |
| 官方 3,000 Test | 未包含在这两份 notebook 的 15,000 输出中 / not included in the 15,000-image outputs of these notebooks | 独立测试 / independent testing |
| 文件移动 | 按 split 目录输出 / exported into split directories | 不重新移动；通过 `train_15000.txt` 统一调用 / no new movement; consolidated through `train_15000.txt` |

当前实验定义优先于 notebook 中历史性的 `TRAIN_RATIO=0.80`、`VAL_RATIO=0.10`、`TEST_RATIO=0.10`。这些比例应保留在来源追踪记录中，但不得继续描述为当前训练划分。

The current experimental definition supersedes the historical notebook values `TRAIN_RATIO=0.80`, `VAL_RATIO=0.10`, and `TEST_RATIO=0.10`. These ratios should remain in provenance records but must not be reported as the current training split.

`vindr_all_classes_labels_only.ipynb` 为了与现有图片逐一对应，仍会按旧目录生成 `labels/train`、`labels/val`、`labels/test`，并在它自己的 `data.yaml` 中写入旧的三个目录。这只是**标签文件的存放结构**。安装新标签后，当前训练配置必须把 `train` 改为 `train_15000.txt`，才能真正把三个目录中的 15,000 张全部交给训练器。

To preserve one-to-one correspondence with the existing images, `vindr_all_classes_labels_only.ipynb` still writes `labels/train`, `labels/val`, and `labels/test`, and its own `data.yaml` points to the historical three directories. This is only the **label storage layout**. After installing the new labels, the current training configuration must set `train` to `train_15000.txt` so that all 15,000 images across the three directories are actually supplied to the trainer.

---

## 7. 训练调用记录 / Training Input Record

当前训练应从统一清单读取 15,000 张图片：

The current training run should read all 15,000 images from the consolidated list:

```yaml
train: train_15000.txt
test: test_3000.txt
nc: 14
single_cls: false
```

其中：

- `train_15000.txt`：引用官方 Train 中全部 15,000 张已预处理图片。
- `train_15000.txt`: references all 15,000 preprocessed images from the official Train set.
- `test_3000.txt`：引用官方 Test 中全部 3,000 张图片；不参与训练。
- `test_3000.txt`: references all 3,000 images from the official Test set and is excluded from training.
- 训练读取图片路径时，YOLO 会自动寻找对应的 `labels` 路径。
- When image paths are loaded, YOLO automatically resolves the corresponding `labels` paths.

---

## 8. 需要在本地最终核验的项目 / Final Local Verification Items

当前已经具备独立的 14 类标签重建 notebook，但所附副本未保存执行输出；15,000 张训练清单也属于之后的项目级更新。因此，正式训练前应保存以下核验结果：

A dedicated 14-class label-reconstruction notebook is now available, but the attached copy does not contain saved execution outputs. The 15,000-image training list is also a later project-level update. Therefore, the following checks should be recorded before formal training:

1. `train_15000.txt` 恰好包含 15,000 条唯一、可读取的图片路径。  
   `train_15000.txt` contains exactly 15,000 unique and readable image paths.
2. 当前 `data.yaml` 为 `nc: 14`，类别顺序与本记录一致。  
   The current `data.yaml` uses `nc: 14`, with the same class order as this record.
3. 所有非空标签中的 `class_id` 均在 0–13 范围内。  
   Every `class_id` in non-empty labels falls within 0–13.
4. 每张训练图片存在对应标签文件；阴性图使用空 TXT。  
   Every training image has a matching label file; negative images use empty TXT files.
5. 当前标签目录不再引用 `labels_nodule_mass_backup`。  
   The current label directory no longer references `labels_nodule_mass_backup`.
6. 官方 3,000 张 Test 与 15,000 张 Train 的 `image_id` 无重叠。  
   No `image_id` overlaps between the official 3,000-image Test set and the 15,000-image Train set.
7. 训练配置为 `single_cls=False`，避免把 14 类重新折叠为一类。  
   The training configuration uses `single_cls=False` so that the 14 classes are not collapsed into one.
8. 标签 notebook 成功生成 `generation_summary.json`，其中 `status` 为 `LABELS_READY` 且 `audit_errors` 为 0。  
   The label notebook produces `generation_summary.json` with `status=LABELS_READY` and `audit_errors=0`.
9. 保存 `final_class_distribution.csv` 与 `final_split_label_distribution.csv`，作为论文中类别分布和样本分布的正式依据。  
   Preserve `final_class_distribution.csv` and `final_split_label_distribution.csv` as the formal source for class and sample distributions reported in the paper.

---

## 9. 已废弃的实验描述 / Deprecated Experimental Descriptions

以下表述不能再用于当前实验报告：

The following descriptions must no longer be used for the current experiment:

- “仅检测 Nodule/Mass” / “Nodule/Mass-only detection”
- `single_cls=True`
- “12,000 train + 1,500 validation + 1,500 test”
- “11,999 train + 1,500 validation + 1,500 test”
- “70% / 15% / 15% split”
- 将 `labels_nodule_mass_backup` 作为当前训练标签 / Using `labels_nodule_mass_backup` as the current training labels
- 将离线预处理误写为固定 640×640 / Describing offline preprocessing as a fixed 640×640 resize

---

## 10. 可直接用于报告的方法描述 / Report-Ready Method Statement

### 中文

本研究采用 VinDr-CXR/VinBigData 胸部 X 光数据集进行多异常目标检测实验。官方训练集中的 15,000 张胸片全部纳入训练清单，官方 3,000 张测试图像作为独立测试集，不参与训练。原始二维 DICOM 图像首先应用 Modality LUT；当 VOI LUT 或窗宽窗位信息可用时应用 VOI 变换，否则采用有效像素的 0.5–99.5 百分位进行强度裁剪。对 MONOCHROME1 图像执行条件反转，随后将像素线性归一化至 0–255 并保存为单通道 PNG。离线输出保持原始长宽比，仅在最长边超过 1024 像素时使用 Lanczos 插值缩小；模型训练阶段再由 YOLO 按 `imgsz=640` 等比例缩放并补边。检测标签从 Kaggle 原始 `train.csv` 重建，覆盖 14 类胸片异常；`No finding` 图像作为空标签阴性样本保留。对同一图像、同一类别且来自不同放射科医生的框，在 IoU 不低于 0.50 时进行聚类，并取簇内坐标均值作为融合框；同一医生的第二个框不允许加入同一病灶簇。单医生框为保持敏感性予以保留，并单独纳入低共识复核报告。训练时关闭单类别模式。数据清洗包括全量解码校验、文件与像素 SHA-256、标签和几何一致性检查、重复样本审计以及训练完整性闸门；精确重复和近重复样本不自动删除，只有像素相同但标签冲突的样本组会被隔离并阻断训练。

### English

This study uses the VinDr-CXR/VinBigData chest X-ray dataset for multi-abnormality object detection. All 15,000 images from the official training set are included in the training list, while the official 3,000 test images are retained as an independent test set and are not used for training. Each two-dimensional DICOM image is first processed with the Modality LUT. A VOI transformation is applied when a VOI LUT or window center/width information is available; otherwise, intensity clipping is performed using the 0.5th and 99.5th percentiles of valid pixels. MONOCHROME1 images are conditionally inverted, after which pixel intensities are linearly normalized to 0–255 and saved as single-channel PNG files. The offline outputs preserve the original aspect ratio and are downscaled with Lanczos interpolation only when the longest side exceeds 1024 pixels. During model training, YOLO subsequently performs aspect-ratio-preserving resizing and letterboxing at `imgsz=640`. Detection labels are rebuilt from the original Kaggle `train.csv` for 14 thoracic abnormality classes, while `No finding` images are retained as negative samples with empty label files. Boxes for the same image and class from different radiologists are clustered when their IoU is at least 0.50, and the coordinate mean within each cluster is used as the fused box. A second box from the same radiologist cannot join the same lesion cluster. Single-radiologist boxes are retained to preserve sensitivity and are separately listed for low-consensus review. Single-class mode is disabled during training. Data cleaning includes full image decoding, file- and pixel-level SHA-256 checks, label and geometric consistency validation, duplicate auditing, and a training-readiness gate. Exact and perceptual near-duplicates are not automatically removed; only groups with identical pixels but conflicting labels are quarantined and block training.

---

## 11. 记录依据 / Record Basis

- `640-preprocessing-final(1).ipynb`：影像预处理参数、原始 15,000 张运行结果及历史 80/10/10 划分。
- `640-preprocessing-final(1).ipynb`: image preprocessing parameters, the recorded 15,000-image run, and the historical 80/10/10 split.
- `only-datacleaning-final(2).ipynb`：V6 清洗、审计、哈希、重复处理、几何检查和训练闸门参数。
- `only-datacleaning-final(2).ipynb`: V6 cleaning, auditing, hashing, duplicate handling, geometry validation, and training-gate parameters.
- `vindr_all_classes_labels_only.ipynb`：14 类映射、原始框清洗、多医生融合、YOLO 标签生成、标签审计和输出报告定义。
- `vindr_all_classes_labels_only.ipynb`: 14-class mapping, source-box cleaning, multi-radiologist fusion, YOLO label generation, label auditing, and output-report definitions.
- 项目当前更新：15,000 张通过 `train_15000.txt` 统一作为训练集，采用 14 类检测；官方 3,000 张作为独立测试集。
- Current project update: all 15,000 images are consolidated for training through `train_15000.txt`, the task uses 14 detection classes, and the official 3,000 images form the independent test set.
