<!--

 * @Author: zhanghao
 * @Date: 2022-09-06 12:59:44
 * @LastEditTime: 2022-12-03 16:14:49
 * @FilePath: /hao_evo/readme.md
 * @Description: 
-->
# hao_evo

**evo github 地址**：https://github.com/MichaelGrupp/evo

**不同格式的轨迹文件及相互转换：**https://github.com/MichaelGrupp/evo/wiki/Formats

**问题汇总：**[https://github.com/MichaelGrupp/evo/issues?q=is%3Aissue+is%3Aclosed](https://github.com/MichaelGrupp/evo/issues?q=is:issue is:closed)

**使用手册：**https://github.com/MichaelGrupp/evo/wiki

max：表示最大误差；
mean：平均误差；
median：误差中位数；
min：最小误差；
rmse：均方根误差；
sse：和方差、误差平方和；
std：标准差

Metrics:
evo_ape - 绝对姿态误差
evo_rpe - 相对姿态误差
evo_rpe -for-each -sub-sequence-wise averaged pose error

Tools:
evo_traj - 分析、绘制、输出一个或多个轨迹
evo_res - 比较一个或多个evo_ape/eco_rpe的结果文件
evo_fig - 重新打开序列化图像（存储为serialize_plot)
evo_config - 全局设置和配置文件操作

## 01 EVO install
```
sudo apt-get install python3
sudo apt install python3-pip
pip install evo --upgrade --no-binary evo
//reboot
evo_ape -h
```
或者通过github下载源码后(https://github.com/MichaelGrupp/evo)，使用源码安装：

pip install --editable . --upgrade --no-binary evo

## 02 EVO 

### 2.1 参数设置 Example configuration for print-quality plots

#### 1. Set the plot grid and background

The default plot settings have a dark background. This looks good on the screen, but is not suitable for printing. We can change the background to a grid with white background by changing the style parameter of [Seaborn](https://seaborn.pydata.org/) with:

```
evo_config set plot_seaborn_style whitegrid
```

#### 2. Set the font type and scale

The default font doesn't really fit to the rest of our paper, which uses a serif font. The relative size of the text labels in the plot figure could also be increased for better readability. We can switch to a serif font with larger scale by calling:

```
evo_config set plot_fontfamily serif plot_fontscale 1.2
```

To match the smaller font, we also reduce the line width:

```
evo_config set plot_linewidth 1.0
```

There are some other things you can change, for example the line style of the reference trajectory:

```
evo_config set plot_reference_linestyle -
```

#### 3. Set the default figure size

You can adjust the default plot figure size, too. For example to a width of 5 and a height of 4.5:

```
evo_config set plot_figsize 5 4.5
```

#### 4. Use LaTeX renderer

Because we use LaTeX to write the paper, we want to render the fonts of the plots with LaTeX, too:

```
evo_config set plot_usetex
```

You may need to change the `plot_texsystem` parameter to the LaTeX system that's installed on your machine if this doesn't work on the first try, see `evo_config show`.

**Advanced:** plots can be also exported in pgf format (`--save_plot plot.pgf`).

#### 5. Restore default settings

```
evo_config reset
```

**重要参数：**

-p或–plot 绘图
-v或–verbose 输出相关信息（均值，方差等）
-f或–full_check检查相关信息（时间戳是否对应，四元数是不是单位四元数）
-a或–align对轨迹进行配准，用ICP的方法，并不是仅仅将起点对齐
运用配准时真实轨迹文档前需要给定–ref=

单目相机会存在尺度的不确定性，evo_traj 支持使用-s（或 --correct_scale）参数进行Sim(3)上的对齐（旋转、平移与尺度缩放）

可以在命令行通过-h参数查看当前evo指令的参数及相关说明。例如：

```
evo_traj tum –h
```

**查看evo参数：**

```
evo_config show
```

```
evo_config set plot_export_format pdf
evo_config set plot_figsize 6 6
evo_config set plot_fontscale 1.8
evo_config set plot_linewidth 1.8
evo_config set plot_reference_color black
evo_config set plot_reference_linestyle -
evo_config set plot_seaborn_style white
```

### 2.2 Coordinate axis markers坐标轴标记

Coordinate axis markers can be activated by setting `plot_axis_marker_scale` to a non-zero value, for example:

```
evo_config set plot_axis_marker_scale 0.1
```

Reference trajectories have a separate parameter `plot_reference_axis_marker_scale`.

You probably need to play around with this scale value to adjust it to the size of your trajectories. Set it to `0` again if you don't need it anymore.

<img src="https://github.com/MichaelGrupp/evo/raw/master/doc/assets/markers.png" alt="img" style="zoom:50%;" />

### 2.3 Pose correspondence markers姿势对应标记

You can enable markers that connect the reference trajectory's poses with the corresponding poses in the other trajectories. This setting applies to `evo_ape` and `evo_rpe`, but also to `evo_traj` if you specify a `--ref` reference and the other trajectories are either synced or aligned.

Activate:

```
evo_config set plot_pose_correspondences true
```

<img src="https://raw.githubusercontent.com/MichaelGrupp/evo/master/doc/assets/pose_corr_markers.png" alt="img" style="zoom:50%;" />

### 2.4 2D ROS maps 2D ROS地图

You can insert 2D maps that follow the ROS map_server format into plots. The format is described here: http://wiki.ros.org/map_server#Map_format

Given that you have such a pair of files (map.pgm and map.yaml), just add `--ros_map_yaml map.yaml` and the map will be shown in addition to the trajectories:

![img](https://github.com/MichaelGrupp/evo/raw/master/doc/assets/ros_map.png)

Note that this follows the ROS convention:

- the map is assumed to be in the x-y plane, so it will be only shown in the xy/yx plot modes
- the image will be transformed to the x-y plane's origin as specified in the .yaml file

In the above image the unknown cells of the map grid are not shown. To make this work, you might have to tweak the setting for the uint8 value that represents unknown cells in a ROS map image. ROS' standard map_saver uses 205, other tools might not (for example, Cartographer uses 128 for images of probability grids):

```
evo_config set ros_map_unknown_cell_value 128
```

But this can also be completely disabled. Then the bounding box of the map will be visible.

```
evo_config set ros_map_unknown_cell_value false
```

### 2.5 绘制轨迹

**绘制单个轨迹：**

```
evo_traj tum groundtruth.txt -p --plot_mode=xyz
```

**绘制多个轨迹：**

```
evo_traj tum CameraTrajectory.txt KeyFrameTrajectory.txt groundtruth.txt -p --plot_mode=xyz
cd test/data
evo_traj kitti KITTI_00_ORB.txt KITTI_00_SPTAM.txt --ref=KITTI_00_gt.txt -p --plot_mode=xz
```

**轨迹对齐后绘制轨迹，配准时真实轨迹要加-ref=，以及-a**

```
evo_traj tum CameraTrajectory.txt KeyFrameTrajectory.txt --ref=groundtruth.txt -p -a --plot_mode=xyz
evo_traj kitti KITTI_00_ORB.txt KITTI_00_SPTAM.txt --ref=KITTI_00_gt.txt -p --plot_mode=xz
```

### 2.6 误差分析

#### **evo_ape Absolute pose error**

绝对位姿误差(absolute pose error)，用于整体评估整条轨迹的全局一致性；绝对位姿误差，常被用作绝对轨迹误差，比较估计轨迹和参考轨迹并计算整个轨迹的统计数据，适用于测试轨迹的全局一致性。

轨迹对齐-a与尺度缩放-s,若增加可选参数-p，可以绘制误差相关曲线：

```cpp
evo_ape tum realTraj.txt estTraj.txt -a –s -p
```

注意：在进行评估时，若经过了缩放，在命令行中应将真实轨迹（参考轨迹）放在估计轨迹（计算轨迹）前方，避免在缩放时参考轨迹错误而造成误差被错误缩放。

**命令语法：**命令 格式 参考轨迹 估计轨迹 [可选项]。可选项有对齐命令、画图、保存结果等。

```
evo_ape tum groundtruth.txt CameraTrajectory.txt -r full -va -p --plot_mode=xyz --save_results results/ORB.zip
evo_ape kitti KITTI_00_gt.txt KITTI_00_ORB.txt -va --plot --plot_mode xz --save_results results/ORB.zip
evo_ape kitti KITTI_00_gt.txt KITTI_00_SPTAM.txt -va --plot --plot_mode xz --save_results results/SPTAM.zip
```

计算考虑平移和旋转部分误差的ape,进行平移和旋转对齐,以详细模式显示保存至.zip文件，画图并保存为results文件夹下的ORB.zip。
使用ape的时候无需给出–ref 的文件，tum 后第一个文件即为标准文件。
–plot_mode选择画图模式，二维图或者三维图，可选参数为[xy, xz, yx, yz, zx, zy, xyz]，默认为xyz。
其中-r表示ape所基于的姿态关系，不添加-r/–pose_relation和可选项，则默认为trans_part：

```
evo_ape tum groundtruth.txt CameraTrajectory.txt -r full -va -p --plot_mode=xyz --save_results results/ORB.zip
```

<img src="https://img-blog.csdnimg.cn/298343a75fd149a4a812cbe5665e5805.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Zu-54G144CC,size_20,color_FFFFFF,t_70,g_se,x_16" alt="在这里插入图片描述" style="zoom:67%;" />

-v表示verbose mode,详细模式，-a表示采用SE(3) Umeyama对齐，其余可选项如下表所示。不加-s表示默认尺度对齐参数为1.0，即不进行尺度对齐：

<img src="https://img-blog.csdnimg.cn/b55a732b876c4ddcadd8b3c1d87d0ffd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Zu-54G144CC,size_20,color_FFFFFF,t_70,g_se,x_16" alt="在这里插入图片描述" style="zoom:67%;" />

```

evo_traj bag 2022-08-31-21-42-33.bag /odom --ref /location_pose -p --plot_mode xy --align

evo_ape bag 2022-08-31-21-42-33.bag /odom  /location_pose -p --plot_mode xy --align -r full

```

#### **evo_rpe 相对位姿误差(relative pose error)**

用于评价轨迹局部的**准确性**。

**evo_rpe：**相对位姿误差，不进行绝对位姿的比较，相对位姿误差比较运动（姿态增量）。相对位姿误差可以给出局部[精度](https://so.csdn.net/so/search?q=精度&spm=1001.2101.3001.7020)，例如slam系统每米的平移或者旋转漂移量。
**命令语法：**命令语法：命令 格式 参考轨迹 估计轨迹 [可选项]。可选项有对齐命令、画图、保存结果等。

```
evo_rpe tum groundtruth.txt CameraTrajectory.txt -r angle_deg --delta 1 --delta_unit m -va -p --plot_mode=xyz
```

求每米考虑旋转角的rpe，以详细模式显示并画图。
其中-r表示ape所基于的姿态关系：

<img src="https://img-blog.csdnimg.cn/f18f441a4cbb42c3851ea2f1fab70e40.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Zu-54G144CC,size_20,color_FFFFFF,t_70,g_se,x_16" alt="在这里插入图片描述" style="zoom:67%;" />

不添加-r/–pose_relation和可选项，则默认为trans_part。
–d/–delta表示相对位姿之间的增量
–u/–delta_unit表示增量的单位，可选参数为[f, d, r, m],分别表示[frames, deg, rad, meters]。
–d/–delta -u/–delta_unit合起来表示衡量局部精度的单位，如每米，每弧度，每百米等。其中–delta_unit为f时，–delta的参数必须为整形，其余情况下可以为浮点型。–delta 默认为1，–delta_unit默认为f。
当在命令中加上–all_pairs,则计算rpe时使用位置数据中所有的对而不是仅连续对，此时，可以通过-t/–delta_tol控制–all_pairs模式下的相对增量的容差（relative delta tolerance）。需要注意–all_pairs下不能使用–plot函数。

**evo_res 从一个度量中处理多个结果**

evo_ape/evo_rpe中将结果保存为.zip文件后，可以利用evo_res对不同的结果进行比较。

evo_res可用于比较指标中的多个结果文件，即：

​    打印信息和统计信息（默认）打印结果并将统计信息保存在表格中。在这里，我们使用上面的结果生成一个图和一个表：

```
mkdir results
evo_ape kitti KITTI_00_gt.txt KITTI_00_ORB.txt -va --plot --plot_mode xz --save_results results/ORB.zip
evo_ape kitti KITTI_00_gt.txt KITTI_00_SPTAM.txt -va --plot --plot_mode xz --save_results results/SPTAM.zip
evo_res results/*.zip -p --save_table results/table.csv

evo_res ORB.zip -v -p
```