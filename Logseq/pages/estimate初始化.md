- ```cpp
  bool PitchByLane::Init(const EstimatePitchInitConfig &config, common::CAMERA_POSITION &cam) {
    roi_top_ = config.lane_estimate_config.rois.at(cam).top;
    roi_bottom_ = config.lane_estimate_config.rois.at(cam).bottom;
    roi_left_ = config.lane_estimate_config.rois.at(cam).left;
    roi_right_ = config.lane_estimate_config.rois.at(cam).right;
  
    // emperical amount of ultra fast anchor in roi, TODO: use actual anchor in roi instead
    max_vertical_lane_cnt_uv_ = (roi_bottom_ - roi_top_) / 2.5;
  
    num_pts_to_fit_line_ = config.lane_estimate_config.num_pts_to_fit_line;
    num_iter_ransac_line_ = config.lane_estimate_config.num_iter_ransac_line;
    num_threads_omp_ = config.lane_estimate_config.num_threads_omp;
  
    ego_lane_check_info_.sigma_lane_width_m =
        config.lane_estimate_config.ego_lane_check_info.sigma_lane_width_m;
    ego_lane_check_info_.mu_lane_width_m =
        config.lane_estimate_config.ego_lane_check_info.mu_lane_width_m;
    ego_lane_check_info_.sigma_ego_lane_center_m =
        config.lane_estimate_config.ego_lane_check_info.sigma_ego_lane_center_m;
  
    // min/max lane width params
    factor_lane_width_min_ = 0.75;
    factor_lane_width_max_ = 2.25;
    return true;
  }
  ```