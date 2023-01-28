title:: lane_fusion

-
- lane_perception后面，后处理。lane_perception做点的筛选、更新pitch、地面拟合出线
- lane_perception
- 入口：system.cpp中TrackCurrentFrame
	- ```cpp
	  void LaneSlamSystem::TrackCurrentFrame(
	    unsigned long frame_id,const Odometry &odom,
	    const std::vector<FusionSpline> &input_splines,
	    float *dev_data,int height,int width)
	  ```
- lane_perception分支区别
	- dualkf
		- 支持两个相机，分时复用
	- master...cut
		- 适配任意多个相机，所有相机的车道先都会融合
		- common中做下面的。dualkf合并一起的
	- pitch估计不同、坐标系转换不同
-