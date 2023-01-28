- ```cpp
  DrawPoint(image_uv_color_keeping, common::Point2f(lane_uf_pts[i][j].x, lane_uf_pts[i][j].y), ins_id_colors_table[i], 3, -1, 0.0f, 0.0f);
  void DrawPoint(cv::Mat& image, const common::Point2_<_Tp>& pt,
                 const cv::Scalar color, const int radius, int thickness = 1,
                 const _Tp& x_offset = 0, const _Tp& y_offset = 0) {
    cv::Point pt_center;
    pt_center.x = pt.x + x_offset;
    pt_center.y = pt.y + y_offset;
    circle(image, pt_center, radius 3, color, thickness -1);
  }
  ```
- `circle`：
	- ```cpp
	  图像：它是要绘制圆圈的图像。
	  center_coordinates：它是圆的中心坐标。坐标表示为两个值的元组，即（X坐标值，Y坐标值）。
	  半径：它是圆的半径。
	  颜色：它是要绘制的圆的边界线的颜色。对于BGR，我们传递一个元组。例如：（255， 0， 0） 表示蓝色。
	  厚度：它是圆边框线的粗细，单位为px。厚度为 -1 px将按指定颜色填充圆形。
	  ```
- `DrawLanePoints`:
	- `DrawLanePoints(image_uv_color, pts_uv[spline_idx], color, false, false, thickness, MIDDLE);`
	- ```cpp
	  void LaneDetectionHelper::DrawLanePoints(
	      cv::Mat& image, const std::vector<common::Point2f>& lane_points,
	      const cv::Scalar& color, const bool draw_solid, const bool draw_slow_down,
	      const int thickness, const uint32_t dual_lane_index,
	      const float& y_offset) {
	    	uint32_t count = lane_points.size();
	    	for (size_t i = 1; i < count;) {
	        if (!draw_solid && i % 10 == 0) {
	          i += 5;
	          continue;
	      }
	      cv::Point2f start(lane_points[i - 1].x, lane_points[i - 1].y);
	      cv::Point2f end(lane_points[i].x, lane_points[i].y);
	      cv::Point2f shift(0.0f, y_offset);
	      if ((i == 1 || i == 5 || i == 9) && draw_slow_down) {
	        cv::arrowedLine(image, end, start, color, thickness, 8, 0, 3);
	      }
	      if (dual_lane_index == LaneShift::LEFT) {
	        shift.x = -17.0 * (count - i) / count - 3.0;
	      } else if (dual_lane_index == LaneShift::RIGHT) {
	        shift.x = 17.0 * (count - i) / count + 3.0;
	      }
	      start += shift;
	      end += shift;
	      cv::line(image, start, end, color, thickness);
	      ++i;
	    }
	  }
	  ```
		-
- `GetCurrentLaneIdx`
	- 先找到el和er的索引
	-