## Welcome

Link [Edit this page](https://github.com/illusionaireal/blog/edit/gh-pages/index.md) [Jekyll](https://jekyllrb.com/) 

#LVISAM 踩坑记录


##参数设置

###KITTI篇
KITTI 数据采集平台硬件配置如下
惯性导航系统（GPS / IMU）：OXTS RT 3003
1台激光雷达：Velodyne HDL-64E
2台灰度相机，1.4百万像素：Point Grey Flea 2（FL2-14S3M-C）
2个彩色摄像头，1.4百万像素：Point Grey Flea 2（FL2-14S3C-C）
4个变焦镜头，4-8毫米：Edmund Optics NT59-917
####raw_sync 数据
#####kitti 激光雷达、摄像头数据融合：
要将Velodyne坐标中的点x投影到左侧的彩色图像中y：
使用公式：y = P_rect_2 * R0_rect *Tr_velo_to_cam * x

将Velodyne坐标中的点投影到右侧的彩色图像中：

使用公式：y = P_rect_3 * R0_rect *Tr_velo_to_cam * x

Tr_velo_to_cam * x    ：是将Velodyne坐标中的点x投影到编号为0的相机（参考相机）坐标系中

R0_rect *Tr_velo_to_cam * x    ：是将Velodyne坐标中的点x投影到编号为0的相机（参考相机）坐标系中，再修正

P_rect_2 * R0_rect *Tr_velo_to_cam * x     ：是将Velodyne坐标中的点x投影到编号为0的相机（参考相机）坐标系中，再修正，然后投影到编号为2的相机（左彩色相机）

注意：

P_rect_2 (标号为2的摄像机的内参矩阵，只和相机内部参数有关，比如焦距和光心位置)

R0_rect（相机0的矫正旋转矩阵）

Tr_velo_to_cam（点云到相机的外参矩阵）

3、kitti 提供的校正文件解析：以（2011_09_26_calib文件夹中的文件为例）文件夹中包含三个文件

（1）calib_cam_to_cam.txt(相机到相机的标定)：

其中
– S_xx：1×2 矫正前的图像xx的大小 
– K_xx：3×3 矫正前摄像机xx的校准矩阵 
– D_xx：1×5 矫正前摄像头xx的失真向量 
– R_xx：3×3 （外部）的旋转矩阵(从相机0到相机xx)
– T_xx：3×1 （外部）的平移矢量(从相机0到相机xx)
– S_rect_xx：1×2 矫正后的图像xx的大小 
– R_rect_xx：3×3 纠正旋转矩阵(使图像平面共面)
– P_rect_xx：3×4 矫正后的投影矩阵 （内参矩阵，前面第一行为前面四个数据，依次三行）

KITTI标定数据解读插图

（2）calib_velo_to_cam.txt

其中
– R：3×3旋转矩阵 
– T：3×1平移向量 
– delta_f：弃用 
– delta_c：弃用

– Tr_velo_to_cam = （R | T）（点云到相机的外参矩阵3×4）

（3）calib_imu_to_velo.txt

Y = P_rect_xx * R_rect_00 * (R|T)_velo_to_cam * (R|T)_imu_to_velo * X

4、 传感器标定

（1）calib_cam_to_cam.txt （P_rect_xx）

相机的标定，即为通过某个已知的目标，求取相机内参矩阵的过程。最常用的标定目标就是棋盘格。

准备好棋盘格照片之后采用matlab 自带的tools Camera Calibrator进行标定

单目摄像机需要标定的参数双目都需要标定
双目摄像机比单目摄像机多标定的参数（R和T）主要是描述两个摄像机相对位置关系的参数，这些参数在立体校正和对极几何中用处很大
（2）calib_velo_to_cam.txt

主要是得到点云到图像的旋转平移矩阵：Tr_velo_to_cam = （R | T）
以上来自页面[KITTI标定数据解读](https://bingxiong.vip/?p=18523)
