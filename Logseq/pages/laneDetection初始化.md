- ```cpp
  void LaneDetection::Init(const LaneDetParam &lane_det_param) {
    NM_INFO("LaneDetection::Init In!");
    detected_lanes_.clear();
  
    frame_id_ = 0;
    is_lane_tracking_ = lane_det_param.tracker_info.is_lane_tracking;  读取初始化tracking
  
    for (const auto cam_pos : lane_det_param.camera_launch_setting) {
      camera_launch_setting_.insert(cam_pos);
      map_infos_[cam_pos].net_info = lane_det_param.net_info.at(cam_pos);
      const int inference_feature_map_num =
          map_infos_.at(cam_pos).map_index_info.map_num;
      host_inference_features_.insert(std::make_pair(cam_pos, std::vector<float *>(inference_feature_map_num, nullptr)));
  
      // init decoder
      lane_decoders_.insert(std::make_pair(cam_pos, LaneDecoder(cam_pos, map_infos_.at(cam_pos))));
      lane_decoders_.at(cam_pos).Init();
      	//void LaneDecoder::Init() {}
      // init detector
      lane_detectors_.insert(std::make_pair(cam_pos, LaneDetector(cam_pos, map_infos_.at(cam_pos))));
      lane_detectors_.at(cam_pos).Init();
  		//void LaneDetector::Init() {}
      NM_INFO("lane detection {} launched.", CAMERA_POSITION_Str[int(cam_pos)]);
    }
  
    lane_slam_fusion_ = lane_slam::CreateLaneSlamSystem();
    NM_INFO("LaneDetection::Init Done!");
  };
  ```
- ```cpp
  LaneDecoder(const common::CAMERA_POSITION camera_position,
                const MapInfo &map_info)
        : camera_position_(camera_position),
          net_input_height_(map_info.net_info.net_input_height),
          net_input_width_(map_info.net_info.net_input_width),
          uf_lane_grid_nums_(map_info.net_info.ugrid_num),
          uf_lane_pt_nums_(map_info.net_info.vanchor_num),
          UFv_(map_info.net_info.vanchors),
          map_uf_keeping_index_(map_info.map_index_info.map_uf_keeping_index),3
          map_uf_changing_index_(map_info.map_index_info.map_uf_changing_index),4
          map_uf_roadedge_index_(map_info.map_index_info.map_uf_roadedge_index),5
          uf_keeping_channels_(map_info.map_ch_num_info.map_uf_keeping_ch_num),6
          uf_changing_channels_(map_info.map_ch_num_info.map_uf_changing_ch_num),5
          uf_roadedge_channels_(map_info.map_ch_num_info.map_uf_roadedge_ch_num) {2
    }
  			std::map<common::CAMERA_POSITION, MapInfo> map_infos_;
  			struct MapInfo {
                NetInfo net_info;
                MapIndexInfo map_index_info;
                MapCHNInfo map_ch_num_info;
                MapIDInfo map_id_info;
            	};
  			struct MapIndexInfo {
                int map_type_index = 1;
                int map_color_index = 2;
                int map_uf_keeping_index = 3;
                int map_uf_changing_index = 4;
                int map_uf_roadedge_index = 5;
                int map_num = 6;
              };
  			struct MapCHNInfo {
              int map_type_ch_num = 7;
              int map_color_ch_num = 5;
              int map_uf_keeping_ch_num = 6;
              int map_uf_changing_ch_num = 5;
              int map_uf_roadedge_ch_num = 2;
            };
  ```