- `FitLaneLine(detected_lanes.lane_uv_pts, detected_lanes.lane_line_grd_pts, detected_lanes.lane_line_splines);`
	- 拟合车道线
	- 定义一个数组：
		- ```cpp
		  const std::vector<LanePositionType> lane_pos = {
		        LANE_POSITION_LEFT_ADJACENT_LEFT,
		        LANE_POSITION_LEFT_ADJACENT_RIGHT,
		        LANE_POSITION_EGO_LEFT,
		        LANE_POSITION_EGO_RIGHT,
		        LANE_POSITION_RIGHT_ADJACENT_LEFT,
		        LANE_POSITION_RIGHT_ADJACENT_RIGHT,
		        LANE_POSITION_LEFT_ADJACENT_LEFT,
		        LANE_POSITION_LEFT_ADJACENT_RIGHT,
		        LANE_POSITION_RIGHT_ADJACENT_LEFT,
		        LANE_POSITION_RIGHT_ADJACENT_RIGHT};
		  ```
	- 然后对`lane_uv_pts`的图从0开始遍历，也就对应上面数组的`leftleft`开始遍历。
		- 从这里可以看出`lane_uv_pts`内部的组织方式，前九张图就是上述九个类型的车道线。
	- 如果`lane_uv_pts`的某一个图是空，则跳过，说明没有这条线。
	- ```cpp
	  Eigen::MatrixXf lane_uv_pts_mat(3, lane_uv_pts[line_idx].size());理解为lane_uv_pts的每一个点表示为x,y,z三行的矩阵。z是置信度
	  Eigen::MatrixXf lane_grd_pts_mat(2, lane_uv_pts_mat.cols());将lane_uv_pts的图像坐标系改为世界坐标系xy
	  ```
	- `Pts2Mat`：把lane_uv_pts的图转为矩阵
		- 矩阵容易进行计算
	- `TransformUV2Veh`：将图像坐标系改为世界坐标系xy
		- 矩阵容易计算
	- `Mat2Pts`：将世界坐标系的点转化为数组vector点集。
		- 这样容易遍历
	- 定义车道最大宽度和车道线最远距离（有效范围）
		- ```cpp
		  const float max_lat_range = 10.0f;宽10m
		  const float max_long_range = 500.0f;最远500m
		  ```
	- 因为定义了有效范围，因此对上面的点进行一波筛选，将符合条件的筛选进入`std::vector<common::Point3f> lane_grd_pts_in_range;`，加上之前的置信度。此时的xy就是世界坐标系的xy了，不再是图像坐标系的xy
	- int sp_degree = 2;
	  background-color:: #497d46
		- 开启的是抛物线拟合的方程。如果改为`=3`，则使用的是三次方程的线进行拟合车道线
	- `FitSplineByRansac`
		- ```cpp
		  const int kmax_iter_num = 10;最大迭代次数，即：最大拟合几次
		  const int num_to_picked = degree + 1;选择点的数目，抛物线二次方程选择三个点，三次方程选择四个点
		  const float kdist_thres = 0.2f;好点的标准。即最后拟合的曲线上点到曲线的距离小于0.2m即为好点
		  const float kratio_thres = 0.95f;好线的标准（就是超过这个值就说明已经拟合的很好了，之后不用在迭代了，直接返回）
		  const float kratio_good = 0.666f;好线的最低标准
		  ```
		- `std::vector<float> sampling_weights;`抽取点的权重。我们目前采用的是平均抽取，全是1.
		- `best_inliers_index`好点集合
		- `PolyFit`计算c0c1c2c3三个值。即计算选出点所构成的线的参数
		- 接下来就是计算所拟合出的这条线的分数是否合格，分数计算方式如下：
			- 计算每一个点到这条线的距离，如果小于上述的`0.2m`则加上该点的置信度，最终就可以计算出该条线的分数了。
				- 同时也要记录好点的索引
			- 每一次迭代记录下到这次迭代为止的最大的分数，以及所对应的线的好点的集合。
			- 如果拟合的线的好点的数目超过95%，那么这条线就拟合的很好，就可以直接返回了，不用在继续拟合。
				- ```cpp
				  if (static_cast<float>(best_inliers_index.size()) / static_cast<float>(pts.size()) > kratio_thres) {
				    break;
				  }
				  ```
		- 之后就是判断拟合的最好的线是否符合最低标准
			- 判断一次好点的占比是否低于66.6%，低于则拟合失败。
		- 最后再对所有的好点重新进行一次拟合，拟合为一条曲线。
	- 接下来也是对这条线是否符合标准做出一些判断。
		- 1.比较参数c0c2是否符合标准
		- 2.比较起始点的x和终点的x的大小，如果终点x小于起点x，那么就有问题
		- 3.线上点的数目小于10，线不合格
		- 全部合格后，对线计算他的自信度confidence。先计算所有被选中点的自信度的和，然后除以点的总数，得到平均值作为线的自信度
			- 如果自信度太小，则拟合的很差，抛弃这条线。
			- 否则就对这条线的拟合情况的属性进行设置：
				- ```cpp
				  spline.lane_quality =
				          spline.confidence >= SplineInternal::kHighQualityScore?
				    		LaneQuality::HIGH_QUALITY: (spline.confidence >= SplineInternal::kMediumQualityScore? 
				                                      LaneQuality::MEDIUM_QUALITY
				                     : LaneQuality::LOW_QUALITY);
				  ```
		- ```cpp
		  spline.type_position = lane_pos[idx];初始化车道线位置
		  ```
- `FitLaneBoundary`拟合路沿线，过程同车道线
- `GroupLaneLine`
	- 由于模型的原因，对于同一条车道线可能依旧会拟合出两道车道线。在三种情况下：
		- keeping场景：
			- lr和el
			- rl和er
		- changing场景：
			- lr和rl
	- 这种情况是不合理的，需要删除一条线，采用删除自信度低的那一条。同时也在这个函数中判断汽车changing时的ego车道
	- 采用比较两条车道线上选取的点的距离来判断，小于一定的距离那么他们就是一条线
- `SetAttributesMap`
	- 拷贝进车道线的类型、颜色map
- `SetAttributes`
	- 设置车道线的类型、颜色属性
	- ```cpp
	  // set spline lane type
	  SetSplineType(sampled_uv_pts[i], spline, i % 2 == 0);
	  // set spline lane color
	  SetSplineColor(sampled_uv_pts[i], spline.lane_color);
	  ```
	- 思路就是对车道线左右一小块区域选取点，看点的类型的数目，最高的那种就是对应的类型或颜色（模型输出不同类型、不同颜色矩阵值不一样）
	- ```cpp
	  for (uint32_t i = 0; i < pts_uv_left.size(); ++i) {
	    if (pts_uv_left[i].x >= 0 && pts_uv_left[i].x < net_input_width_ &&
	        pts_uv_left[i].y >= 0 && pts_uv_left[i].y < net_input_height_ &&
	        pts_uv_right[i].x >= 0 && pts_uv_right[i].x < net_input_width_ &&
	        pts_uv_right[i].y >= 0 && pts_uv_right[i].y < net_input_width_) {
	      // row-major
	      const float *lane_type_ptr = lane_type_map_ptr +
	        (int)pts_uv_left[i].y * net_input_width_ +
	        (int)pts_uv_left[i].x;
	      for (int j = 0; j < pts_uv_right[i].x - pts_uv_left[i].x; ++j) {
	        #ifdef DSP_COMPILER
	        lane_pts_type[round(lane_type_ptr[j])]++;
	        #else
	        lane_pts_type[std::round(lane_type_ptr[j])]++;
	        #endif
	      }
	    }
	  }
	  ```
		- 因为`pts_uv_right`和`pts_uv_left`取点距离不一样，所以这样就形成了一个小区域。从而检测这个小区域内点的类型。
			- ```cpp
			  enum LANE_TYPE_CN {
			    TYPE_BACKGROUND = 0,
			    TYPE_SOLID = 1,
			    TYPE_DASHED = 2,
			    TYPE_DOT = 3,
			    TYPE_SLOWDOWN = 4,
			    TYPE_DOUBLESOLID = 5,
			    TYPE_DOUBLEDASHED = 6
			  };
			  ```
	-
- `UpdateInsideSplines`
	- 因为模型导出车道线为单实线，上个函数补充好车道线类型后，如果为双线(其中一条为虚线)，那么就需要得到ego车道的两条车道线的新的拟合曲线，因此需要对原来的模型导出的曲线进行一个平移。
		- 选取的是一个中点，重新计算了c0
-