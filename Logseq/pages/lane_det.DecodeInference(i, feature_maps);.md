title:: lane_det.DecodeInference(i, feature_maps);

- ```cpp
  void LaneDetection::DecodeInference(const size_t frame_id,const std::map<common::CAMERA_POSITION, std::vector<float *>> &inference_features) 
  {
    frame_id_ = frame_id; 就是i，处理到了第几帧画面
    SetFeatureBuffer(inference_features); 还是在初始化，将inference_features中的内容放入lane_det对象host_inference_features_中，如果有cuda,则会复制进显卡设备
    for (const auto &pair : host_inference_features_) {   里面存的是模型分析的结果，对两个摄像头分析的结果保存为了两个数组，用map对应
      // decoder
      lane_decoders_.at(pair.first).RunOnce(frame_id, pair.second, detected_lanes_[pair.first], active_scenes_[pair.first]);
      	std::map<common::CAMERA_POSITION, LaneDecoder> lane_decoders_;
          初始化detected_lanes_结构体的lane_uf_pts，计算得出lane_uv_pts、active_scenes_状态
            
          std::map<common::CAMERA_POSITION, DetectedLanes> detected_lanes_;
          std::map<common::CAMERA_POSITION, ACTIVE_SCENE> active_scenes_;
    }
  }
  ```
-
- `RunOnce(frame_id, pair.second, detected_lanes_[pair.first], active_scenes_[pair.first]);`
- ```cpp
  void LaneDecoder::RunOnce(size_t frame_id,
                            const std::vector<float *> &inference_feature,
                            DetectedLanes &detected_lanes,  //一个结构体
                            ACTIVE_SCENE &active_scene) {    //enum class ACTIVE_SCENE { UNKNOWN = 0, KEEPING = 1, CHANGING = 2 };
    frame_id_ = frame_id;
    std::vector<std::vector<common::Point3f>> &lane_uf_pts = detected_lanes.lane_uf_pts;
    			typedef Point3_<float> Point3f;float类型的三维的点
    std::vector<std::vector<common::Point3f>> &lane_uv_pts = detected_lanes.lane_uv_pts;
    	//std::vector<std::vector<common::Point3f>> lane_uf_pts;  // uf pts (x, y, prb)
    	//std::vector<std::vector<common::Point3f>> lane_uv_pts;  // uv pts (x, y, prb)
  
    static const int uf_size = (uf_lane_grid_nums_ + 1) * uf_lane_pt_nums_;160*96
  
    std::vector<float *> feature_ptrs;
    for (int i = 0; i != uf_keeping_channels_; ++i) {6
      feature_ptrs.emplace_back(inference_feature[map_uf_keeping_index_] + i * uf_size);3
    }
    for (int i = 0; i != uf_changing_channels_; ++i) {5
      feature_ptrs.emplace_back(inference_feature[map_uf_changing_index_] + i * uf_size);4
    }
    for (int i = 0; i != uf_roadedge_channels_; ++i) {2
      feature_ptrs.emplace_back(inference_feature[map_uf_roadedge_index_] + i * uf_size);5
    }
    这里读入keeping、changing、roadedge三种图
      feature_ptrs大小为13
  
    static common::EigenArray grid_idx = Eigen::RowVectorXf::LinSpaced( uf_lane_grid_nums_, 0, uf_lane_grid_nums_ - 1);159大小的vector
    		typedef Eigen::Array<float, Eigen::Dynamic, Eigen::Dynamic, Eigen::RowMajor> EigenArray;行优先的float类型动态矩阵
          typedef Matrix< float, Dynamic, 1 > 	Eigen::VectorXf
    static const int uf_lane_channels = uf_keeping_channels_ + uf_changing_channels_ + uf_roadedge_channels_;13
    lane_uf_pts.clear();
    lane_uf_pts.resize(uf_lane_channels); resize为 13
  
    for (int c = 0; c < uf_lane_channels; ++c) {
      UltraFastSoftmax(feature_ptrs[c], grid_idx, c, lane_uf_pts[c]);
    }
  
    GatherLanePts(lane_uf_pts, lane_uv_pts);初始化lane_uv_pts
    SelectActiveScene(lane_uv_pts, active_scene);根据lane_uv_pts判断是keeping状态还是changing状态
  }
  ```
- [[eigen LinSpaced函数]]
- [[laneDetection初始化]]
- [[maxCoeff函数]]
- 算出了点的x、y的坐标，以及该点的置信度，存入vector<Point3f>，再存入vector<vector<Point3f>>即：最终结果放入`lane_uf_pts`
	- ```cpp
	  void LaneDecoder::UltraFastSoftmax(const float *feature_map, const common::EigenArray &grid_idx, const int channel, 
	                                     std::vector<common::Point3f> &out) {
	    Eigen::Map<const common::EigenMat> input(feature_map, uf_lane_pt_nums_, uf_lane_grid_nums_ + 1);160*96
	    	将feature中的内容映射到input矩阵中
	    Eigen::MatrixXd::Index maxRow, maxCol;
	    for (int h = input.rows() - 1; h >= 0; --h) {
	      int y = UFv_[h]; const vector<int>类型  配置文件中读取的
	      common::EigenArray anchor = input.row(h); 行数组
	      double max = anchor.maxCoeff(&maxRow, &maxCol);//返回最大值
	      if (maxCol < input.cols() - 1) {
	        common::EigenArray tmp = anchor.leftCols(input.cols() - 1).exp();
	        anchor = tmp / tmp.sum();
	        // almost 3 sigma from 1w frames
	        float prb = anchor(maxCol);
	        if (prb < uf_prb_th_) { 0.3f
	          // out.emplace_back(common::Point3f(-1.f, y, 0.f));
	          continue;
	        }
	        anchor = anchor * grid_idx;160*1
	        float x = (anchor.sum() + 0.5) * (net_input_width_ / uf_lane_grid_nums_);
	        out.emplace_back(common::Point3f(x, y, prb));
	      } else {
	        // out.emplace_back(common::Point3f(-1.f, y, 0.f));
	      }
	    }
	  }
	  ```
- 分类车道线，先分析参数是什么
- `RunOnce(frame_id, pair.second,detected_lanes_[pair.first],active_scenes_[pair.first]);`
	- `pair.second`：就是`inference_features`。`map<common::CAMERA_POSITION, std::vector<float *>>`类型
		- floa*存放的是模型分析的结果图片
	- `detected_lanes_[pair.first]`：
		- ```cpp
		  std::map<common::CAMERA_POSITION, DetectedLanes> detected_lanes_;
		  struct DetectedLanes {
		    int frame_id = 0;
		    // below fill by LaneDecoder
		    std::vector<std::vector<common::Point3f>> lane_uf_pts;  // uf pts (x, y, prb)
		    std::vector<std::vector<common::Point3f>> lane_uv_pts;  // uv pts (x, y, prb)
		    ...
		  };
		  ```
	- `active_scenes_`
		- ```cpp
		  std::map<common::CAMERA_POSITION, ACTIVE_SCENE> active_scenes_;
		  enum class ACTIVE_SCENE { UNKNOWN = 0, KEEPING = 1, CHANGING = 2 };
		  ```
	-
	- `GatherLanePts(lane_uf_pts, lane_uv_pts);`
		- lane_uf_pts在上面softmax算法初始化
	- ```cpp
	  void LaneDecoder::GatherLanePts(
	      const std::vector<std::vector<common::Point3f>> &lane_uf_pts,
	      std::vector<std::vector<common::Point3f>> &lane_uv_pts) {
	    lane_uv_pts.clear();
	    lane_uv_pts.resize(uv_lane_nums_);12
	    bool case_7_reserved = true;
	    bool case_8_reserved = true;
	    if (lane_uf_pts[7].empty()) case_7_reserved = false;
	    if (lane_uf_pts[8].empty()) case_8_reserved = false;
	    if (lane_uf_pts[7].empty() && lane_uf_pts[8].empty()) {
	      case_7_reserved = true;
	      case_8_reserved = true;
	    }
	  
	    for (size_t i = 0; i != lane_uv_pts.size(); ++i) {
	      int inputs_idx1 = -1;
	      int inputs_idx2 = -1;
	      bool merge = false;
	      switch (i) {
	        case 0:
	          inputs_idx1 = 0;
	          break;
	        case 1:
	          inputs_idx1 = 1;
	          break;
	        case 2:
	          inputs_idx1 = 2;
	          break;
	        case 3:
	          inputs_idx1 = 3;
	          break;
	        case 4:
	          inputs_idx1 = 4;
	          break;
	        case 5:
	          inputs_idx1 = 5;
	          break;
	        case 6:
	          inputs_idx1 = 6;
	          break;
	        case 9:
	          inputs_idx1 = 10;
	          break;
	        case 10:
	          inputs_idx1 = 11;
	          break;
	        case 11:
	          inputs_idx1 = 12;
	          break;
	  
	        case 7:
	          inputs_idx1 = 7;
	          inputs_idx2 = 9;
	          merge = true && case_7_reserved;
	          break;
	        case 8:
	          inputs_idx1 = 8;
	          inputs_idx2 = 9;
	          merge = true && case_8_reserved;
	          break;
	  
	        default:
	          break;
	      }
	      if (merge == false && inputs_idx1 != -1) {
	        for (size_t j = 0; j != lane_uf_pts[inputs_idx1].size(); ++j) {
	          if (lane_uf_pts[inputs_idx1][j].x > 0.f) {
	            lane_uv_pts[i].emplace_back(lane_uf_pts[inputs_idx1][j]);
	          }
	        }
	      }
	      // merge and sort from belower to upper
	      if (merge == true && inputs_idx1 != -1 && inputs_idx2 != -1 &&
	          (lane_uf_pts[inputs_idx1].size() > 0 ||
	           lane_uf_pts[inputs_idx2].size() > 0)) {
	        int pidx1 = 0, pidx2 = 0;
	        int pnum1 = lane_uf_pts[inputs_idx1].size(),
	            pnum2 = lane_uf_pts[inputs_idx2].size();
	        for (; pidx1 < pnum1 && pidx2 < pnum2;) {
	          while (lane_uf_pts[inputs_idx1][pidx1].x < 0.f && pidx1 < pnum1)
	            ++pidx1;
	          if (pidx1 >= pnum1) break;
	          while (lane_uf_pts[inputs_idx2][pidx2].x < 0.f && pidx2 < pnum2)
	            ++pidx2;
	          if (pidx2 >= pnum2) break;
	          auto &p1 = lane_uf_pts[inputs_idx1][pidx1];
	          auto &p2 = lane_uf_pts[inputs_idx2][pidx2];
	          // p1 is belower
	          if (p1.y > p2.y) {
	            lane_uv_pts[i].emplace_back(p1);
	            ++pidx1;
	          } else {
	            lane_uv_pts[i].emplace_back(p2);
	            ++pidx2;
	          }
	        }
	        if (pidx1 < pnum1 && pidx2 >= pnum2) {
	          for (int j = pidx1; j < pnum1; ++j) {
	            if (lane_uf_pts[inputs_idx1][j].x > 0.f) {
	              lane_uv_pts[i].emplace_back(lane_uf_pts[inputs_idx1][j]);
	            }
	          }
	        } else if (pidx1 >= pnum1 && pidx2 < pnum2) {
	          for (int j = pidx2; j < pnum2; ++j) {
	            if (lane_uf_pts[inputs_idx2][j].x > 0.f) {
	              lane_uv_pts[i].emplace_back(lane_uf_pts[inputs_idx2][j]);
	            }
	          }
	        } else {
	          NM_ERROR("should not get here! {}/{}, {}/{}", pidx1, pnum1, pidx2,
	                   pnum2);
	        }
	      }
	    }
	  }
	  ```
- 计算是keeping状态还是changing状态
- ```cpp
  void LaneDecoder::SelectActiveScene(
      std::vector<std::vector<common::Point3f>> &lane_uv_pts,
      ACTIVE_SCENE &active_scene) {
    active_scene = ACTIVE_SCENE::UNKNOWN;
    float score_keeping = 0.0f;
    for (int k = 0; k < uv_lane_keeping_nums_; ++k) {6
      float score_each = 0.0f;
      for (const auto &pt : lane_uv_pts[k]) {
        score_each += pt.z;
      }
      score_keeping += score_each;
    }
    score_keeping /= (uf_lane_pt_nums_ * uv_lane_keeping_nums_);/=96*6
    float score_changing = 0.0f;
    for (int c = uv_lane_keeping_nums_;
         c < uv_lane_keeping_nums_ + uv_lane_changing_nums_; ++c) {
      float score_each = 0.0f;
      for (const auto &pt : lane_uv_pts[c]) {
        score_each += pt.z;
      }
      score_changing += score_each;
    }
    score_changing /= (uf_lane_pt_nums_ * uv_lane_changing_nums_);
    算出 score_changing 和 score_keeping 两个值
  
    if (score_keeping >= score_changing) {
      for (int c = uv_lane_keeping_nums_;6
           c < uv_lane_keeping_nums_ + uv_lane_changing_nums_; ++c) { <10 clear掉4个changing图
        lane_uv_pts[c].clear();
      }
      active_scene = ACTIVE_SCENE::KEEPING; keeping状态
    } else {
      for (int k = 0; k < uv_lane_keeping_nums_; ++k) { <6 clear掉6个keeping图
        lane_uv_pts[k].clear();
      }
      active_scene = ACTIVE_SCENE::CHANGING; changing状态
    }
    NM_DEBUG("response score: keeping({}), changing({})", score_keeping,
             score_changing);
  }
  ```