- `common::CameraParam camera_param_front_near = {321.428571433, 321.428571433, 321.012836833,192.030494433, 86.560509554,  1.2999994754791260,89.50579834,   2.0, -0.08,1.20};`
	- near同上
- 放入一个map中：`map<common::CAMERA_POSITION,common::CameraParam> camera_params`
- 从这个map读取数据初始化CoordinateTransform单例对象：`common::CoordinateTransform::GetInstance().Init(camera_params);`
	- ```cpp
	  void CoordinateTransform::Init(const std::map<CAMERA_POSITION, CameraParam> &camera_params) 
	  {
	    if (is_initialized_)
	      return;
	    for (const auto &pair : camera_params) {
	      cameras_offline_param_[pair.first] = pair.second;
	      cameras_intrinsic_[pair.first] = GenIntrinsic(pair.second);3x3 Matrix3f
	      cameras_rcw_[pair.first] = GenRcw(pair.second);3x3 Matrix3f 结果为三个矩阵相乘的结果。
	      cameras_twc_[pair.first] = GenTwc(pair.second);3x3 Matrix3f camera_param.intrinsics.fx, 0.f, camera_param.intrinsics.cx, 
	        															0.f,camera_param.intrinsics.fy, camera_param.intrinsics.cy, 
	      															0.f, 0.f, 1.f;
	      cameras_initialized_[pair.first] = true;
	      GenUndistort(pair.first, pair.second);就是初始化单例类中的几个矩阵，初始化 6x1 和 2x1 的矩阵
	    }
	    UpdateTfVeh2UV();
	    UpdateTfVeh2Cam();
	    is_initialized_ = true;
	  }
	  ```
- `LaneDetParam lane_det_param = GenerateInitParameters();`从配置文件`test_lane_detection_config.json`读取数据进行初始化
- `common::EstimatePitchBase::GetInstance()->Init(lane_det_param.estimate_pitch_init_config);`估算pitch角
	- ```cpp
	  bool EstimatePitch::Init(const EstimatePitchInitConfig &config) {
	    PitchFilterKf::GetInstance();初始化PitchFilterKf对象
	    std::map<CAMERA_POSITION, CameraParam> camera_params = CoordinateTransform::GetInstance().GetCameraOfflineParams();从单例对象初始化map
	    if (camera_params.count(CAMERA_FRONT_NEAR) > 0) {
	      main_cam_ = CAMERA_FRONT_NEAR;
	    } else if (camera_params.count(CAMERA_FRONT_MIDDLE) > 0) {
	      main_cam_ = CAMERA_FRONT_MIDDLE;
	    } else if (camera_params.count(CAMERA_FRONT_FAR) > 0) {
	      main_cam_ = CAMERA_FRONT_FAR;
	    } else {
	      NM_ERROR("No suitable camera config, please init CoordinateTransform "
	               "before estimate_pitch.");
	      return false;
	    } 检测相机存在
	    net_info_ = config.net_info; // net input size  这个是从配置文件读取出来的
	    up_thresh_ = 10;             // angle
	    down_thresh_ = -10;          // angle
	  
	    inited = pitch_by_depth_->Init(config, main_cam_) && pitch_by_lane_->Init(config, main_cam_);
	    return inited;
	  }
	  ```
- 开启tracking，则读取vehicle_state.log文件。
	- ```cpp
	  std::map<int, DataFrame> data_frames;
	  if (lane_det_param.tracker_info.is_lane_tracking) {
	    bool success = ReadVehicleState(str_time_stamp, str_vehicle_state,
	                                    idx_start, idx_end, data_frames);
	    if (!success) {
	      NM_ERROR(
	        "Please disable is_lane_tracking in the config/nm_xxx.json file "
	        "since some logs are lost");
	      exit(-1);
	    }
	  }
	  ```
- `nullmax_perception::LaneDetectionHelper lane_det;`
- `lane_det.Init(lane_det_param);`
	- 初始化lane_det
- 得到这几个onnx文件
	- ```cpp
	  LaneTensorrt *seg_net = nullptr;
	  if (!is_read_prb_bin) {
	    const std::string instance_config_path = "../config/v1_5/0231/front_onnx/";
	    seg_net = new LaneTensorrt(
	      instance_config_path + "dl.onnx", instance_config_path + "dl.onnx",
	      "input.1",
	      std::vector<std::string>{"693", "694", "695", "696", "697", "698",
	                               "699"},
	      net_input_height, net_input_width, TensorrtModelType::ONNX);
	  } else {
	    NM_WARN("Read Prb Bin Mode is not supported yet.");
	    exit(-1);
	  }
	  
	  const int map_size = map_row_cnt * map_col_cnt;
	  
	  std::map<common::CAMERA_POSITION, float *> host_feature_maps;六张 模型分析结果和ultra fast 图，六张可以从后面源码读出，后面ptr_feature_maps读入了六张图
	  std::map<common::CAMERA_POSITION, float *> host_freespace_feature_maps;可行驶区域图片map
	  
	  for (const auto &c : lane_det_param.camera_launch_setting) {
	    float *host_feature_map = new float[map_size * channel_count];
	    float *host_freespace_feature_map = new float[map_size];
	    host_feature_maps.insert(std::make_pair(c, host_feature_map));
	    host_freespace_feature_maps.insert(
	      std::make_pair(c, host_freespace_feature_map));
	  }为front_far和front_near相机的host_feature_maps和host_freespace_feature_maps申请空间，后面会进行初始化。用一个map对应起来map<position,feature>
	  ```
- 在显卡设备上申请空间，因为cuda不能直接操纵内存，所以要把内存的数据移入显卡
	- ```cpp
	  std::map<common::CAMERA_POSITION, float *> dev_feature_maps;
	  for (const auto &c : lane_det_param.camera_launch_setting) {
	    float *dev_feature_map = nullptr;
	    CHECK_CUDA(cudaMalloc((void **)&dev_feature_map,
	                          sizeof(float) * map_size * channel_count));
	    dev_feature_maps.insert(std::make_pair(c, dev_feature_map));
	  }
	  ```
- 此时进入for循环，读取每一张图片进行操作
  background-color:: #497d46
- 如果开启tracking，读取之前保存的vehicle_state.log的相应信息
	- ```cpp
	  if (lane_det_param.tracker_info.is_lane_tracking) {
	    const auto &vehicle_state = data_frames.at(i).vehicle_state;
	    lane_det.SetVehicleStateInfo(i, vehicle_state.time_stamp,
	                                 vehicle_state.speed, vehicle_state.yaw_rate,
	                                 vehicle_state.pos_x, vehicle_state.pos_y,
	                                 vehicle_state.pos_theta);
	  }
	  ```
- 从far、near相机读取图片进入：`std::map<common::CAMERA_POSITION, cv::Mat> images_uv_color;`
- 定义一个`std::map<common::CAMERA_POSITION, std::vector<float *>> feature_maps;`feature_maps保存了相机位置和图数组的映射
- 对上面读取的每一个uv原始图片进行模型分析。结果保存到上面申请的`host_feature_maps`和`host_freeture_maps`中，然后从该map移动到显卡设备中。
- 从`host_feature_maps`读取六张图进入`ptr_feature_maps`，然后`insetrt`进`feature_maps`中。
- [[lane_det.DecodeInference(i, feature_maps);]]
	- 将`feature_maps`中的内容放入`lane_det`对象`host_inference_features_`中
	- 初始化`detected_lanes_`结构体的`lane_uf_pts`，计算得出`lane_uv_pts`、`active_scenes_`状态
	  id:: 637ad655-fcfb-4666-bd06-4341da69c86c
	- 最终这个函数得出了`keeping`状态或者`changing`状态
- ```cpp
  struct EgoLaneGeometry {
    float lane_width = 3.3;
  };
  ```
- `common::EgoLaneGeometry ego_lane_geometry;`就是3.3f
-
- ```cpp
  std::map<common::CAMERA_POSITION, std::vector<std::vector<common::Point3f>>> detected_uv_pts;
  lane_det.GetDetectedUvLanePts(detected_uv_pts);提取出 lane_det 对象的detected_lanes_ map中的lane_uv_pts
  ```
	- ```cpp
	  void LaneDetection::GetDetectedUvLanePts(std::map<common::CAMERA_POSITION, std::vector<std::vector<common::Point3f>>> &uv_lane_pts) {
	    uv_lane_pts.clear();
	    const std::map<common::CAMERA_POSITION, DetectedLanes> &detected_lanes = GetDetectedLanes();DetectedLanes就是上面的放uv、uf的结构体 return detected_lanes_;直接返回了对象本身的这个map
	    for (auto det_lane : detected_lanes) {
	      uv_lane_pts[det_lane.first] = det_lane.second.lane_uv_pts;提取出对象本身的lane_uv_pts
	    }
	  }
	  ```
- 估算pitch角
	- [[common::EstimatePitchBase::GetInstance()->Run(detected_uv_pts, nullptr, nullptr, ego_lane_geometry);]]
-