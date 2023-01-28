title:: common::EstimatePitchBase::GetInstance()->Run(detected_uv_pts, nullptr, nullptr, ego_lane_geometry);

- ```cpp
  ```
-
- [[Line类]]
- [[estimate初始化]]
- ```cpp
  // TODO: adjust ransac params
      if (MyFitLineByRansac(line, pts[index], rect, num_pts_to_fit_line_,
                            num_iter_ransac_line_ * 3, num_threads_omp_)) {
        line.score /= max_vertical_lane_cnt_uv_;
        if (line.score > 1) {
          line.score = 1;
        }
        lines_uv.emplace_back(line);
      }
    }
  }
  - static const float score_thresh =
      0.6 * 0.6; // empirical thresh, 0.6 * ufpoint avg score
  return (lines_uv.size() == 2 &&
          std::fmin(lines_uv[0].score, lines_uv[1].score) > score_thresh);
  }
  - bool MyFitLineByRansac(Line &line_uv_fitted, const std::vector<Point3f> &pts,
                       const Rect &rect, const uint32_t &num_pts_fit,
                       const int &num_iter_ransac, const int &num_threads_omp) {
  if (pts.size() < num_pts_fit) {
    return false;
  }
  - // int num_iter_ransac = ransac_info_.num_iter_ransac_line * 2;
  - float score_max = 0.0f;
  Line line_uv_best;
  LineABC line_abc_uv_best;
  - Line line_uv_best_debug;
  - std::vector<float> scores(num_iter_ransac, 0.0f);
  std::vector<LineABC> lines_abc_uv(num_iter_ransac);
  std::vector<Line> lines_uv(num_iter_ransac);
  - float dist_threshold = 2.0f;
  - #ifdef OPEN_OMP_ACC
  #pragma omp parallel for num_threads(num_threads_omp)
  #endif
  for (int i = 0; i < num_iter_ransac; i++) {
    std::vector<Point3f> pts_picked;
    RandSelectPts(pts, num_pts_fit, pts_picked);
  - LineABC line_uv_abc_cur;
    Line line_uv_cur;
  - FitLine(pts_picked, rect, line_uv_abc_cur, line_uv_cur);
  - // compute score of fitted line
    float score = ComputeLineScore(line_uv_abc_cur, pts, dist_threshold);
  - lines_uv[i] = line_uv_cur;
    lines_abc_uv[i] = line_uv_abc_cur;
    scores[i] = score;
  }
  - std::vector<float>::iterator it;
  it = std::max_element(scores.begin(), scores.end());
  int idx_max = std::distance(scores.begin(), it);
  score_max = scores[idx_max];
  - line_uv_best = lines_uv[idx_max];
  line_abc_uv_best = lines_abc_uv[idx_max];
  - // refine line by good pts
  std::vector<Point3f> pts_good;
  for (uint32_t j = 0; j < pts.size(); j++) {
    // compute distance to line
    float dist = fabs(pts[j].x * line_abc_uv_best.a +
                      pts[j].y * line_abc_uv_best.b + line_abc_uv_best.c);
    // check distance
    if (dist <= dist_threshold) {
      pts_good.emplace_back(pts[j]);
    }
  }
  - // refit line using good pts
  if (pts_good.size() >= num_pts_fit) {
    FitLine(pts_good, rect, line_abc_uv_best, line_uv_best);
  }
  - line_uv_fitted = line_uv_best;
  line_uv_fitted.score = score_max;
  - return true;
  }
  ```