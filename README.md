# SMPL-LBS 线性混合蒙皮可视化实验
苏文丽 202411081022
基于 PyTorch+smplx 拆解SMPL完整LBS流程，分阶段可视化中间量，手写LBS并与官方实现误差校验。

## 1. 实验目标
1. 掌握模板网格、形状/姿态参数、关节回归、蒙皮权重的关联逻辑；
2. 理解LBS四阶段计算：
   (a) 模板网格 $\bar{T}$ + 蒙皮权重 $\mathcal{W}$
   (b) 形状校正网格 $\bar{T}+B_S(\beta)$ + 回归关节 $J(\beta)$
   (c) 姿态校正网格 $T_P(\beta,\theta)=\bar{T}+B_S(\beta)+B_P(\theta)$
   (d) LBS加权得到最终人体网格
3. 提取SMPL官方lbs()中间变量并分阶段可视化。

## 2. 实验原理
### 2.1 LBS四阶段公式
1. **模板与权重**
T-pose模板网格$\bar{T}$，每个顶点存储多关节影响权重$\mathcal{W}$，用于后续加权骨骼变换。
2. **形状校正**
$$T_{shape} = \bar{T} + B_S(\beta),\quad J(\beta) = \mathcal{J}(T_{shape})$$
$\beta$控制人体体型，关节由形变网格回归得到。
3. **姿态校正**
$$T_P(\beta,\theta) = \bar{T} + B_S(\beta) + B_P(\theta)$$
通过姿态偏移$B_P(\theta)$修正骨骼旋转带来的皮肤塌陷问题。
4. **线性混合蒙皮**
$$v_i' = \sum_{k=1}^{K} w_{ik} G_k(\theta,J(\beta)) \begin{bmatrix}v_i^{posed}\\1\end{bmatrix}$$
各关节全局变换矩阵按顶点权重加权，得到最终顶点坐标。

### 2.2 核心变量区分
1. `v_template`：原始模板顶点
2. `v_shaped`：仅形状形变顶点
3. `J`：形状网格回归关节
4. `v_posed`：形状+姿态校正顶点
5. `verts`：LBS计算完成的最终顶点

## 3. 实验任务
### 任务1：加载SMPL并输出基础信息
1. 加载中性SMPL模型`SMPL_NEUTRAL.pkl`，`model_type='smpl'`、`gender='neutral'`；
2. 打印顶点数、面片数、关节数、betas维度。

### 任务2：阶段(a) 模板网格+蒙皮权重可视化
输出：`outputs/stage_a_template_weights.png`
可选：`outputs/all_joint_weights.png`
1. 绘制T-pose模板网格；
2. 单关节权重映射为热力图；可选绘制各顶点主导关节分布图。

### 任务3：阶段(b) 形状校正与关节可视化
输出：`outputs/stage_b_shaped_joints.png`
1. 设置非零$\beta$计算`v_shaped`；
2. 回归关节$J$，同图叠加网格与关节点。

思考题：关节为何不固定，需要从形变网格回归？

### 任务4：阶段(c) 姿态偏移校正可视化
输出：`outputs/stage_c_pose_offsets.png`
1. 设置姿态$\theta$计算`pose_offsets`与`v_posed`；
2. 将偏移幅值映射为网格颜色，可视化姿态修正区域。

思考题：若无姿态校正，人体弯曲处会出现什么问题？

### 任务5：阶段(d) 完整LBS结果可视化
输出：`outputs/stage_d_lbs_result.png`
1. 求解关节全局变换，按权重加权蒙皮；
2. 渲染最终姿态网格与关节。

思考题：$J$与姿态变换后$J_{transformed}$有何区别？

### 任务6：四阶段对比总图
输出：`outputs/comparison_grid.png`
2×2子图，标注：(a)模板权重 (b)形状关节 (c)姿态偏移 (d)最终蒙皮。

### 任务7：手写LBS与官方误差校验
输出：`outputs/summary.txt`
1. 相同输入分别运行手写LBS、官方SMPL前向；
2. 计算MAE平均绝对误差、MaxAE最大绝对误差，写入文本。

## 实验输出可视化结果
### 阶段(a) 模板网格与单关节权重热力图
![模板网格+单关节权重热力图](outputs/stage_a_template_weights.png)

### 全关节主导权重分布图
![全关节主导权重分布](outputs/all_joint_weights.png)

### 阶段(b) 形状校正网格与回归关节
![形状形变网格+人体关节可视化](outputs/stage_b_shaped_joints.png)

### 阶段(c) 姿态偏移校正可视化
![姿态偏移修正区域着色图](outputs/stage_c_pose_offsets.png)

### 阶段(d) 完整LBS蒙皮最终姿态
![线性混合蒙皮最终人体姿态](outputs/stage_d_lbs_result.png)

### 四阶段对比总图
![四大计算阶段对比2×2网格图](outputs/comparison_grid.png)

<img width="876" height="369" alt="image" src="https://github.com/user-attachments/assets/b44e35aa-fd94-4fcb-a243-2b873da7f676" />

# 单骨骼旋转蒙皮动画实验
## 实验目的
固定模型Shape参数，驱动单关节从0°渐变旋转至目标角度，导出GIF/帧图，观察蒙皮权重区域平滑形变效果。

## 实现要点
1. 锁定网格Shape，不修改模型外形
2. 单关节角度线性插值：0° → 目标角度
3. 逐帧渲染，输出图片序列或合成GIF/MP4
4. 观测权重过渡区随骨骼运动的平滑拉伸效果


## 效果演示
<img width="1077" height="1120" alt="image" src="https://github.com/user-attachments/assets/a0710ac8-ac09-4e2a-a41e-bdc452e9ab51" />





## 4. 运行环境
```bash
pip install torch smplx numpy trimesh matplotlib
