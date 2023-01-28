- ```cpp
  struct Line {
    Line(){};
  
    Line(int x_start, int y_start, int x_end, int y_end,
         const float _score = 0.0f) {
      start_point.x = x_start;
      start_point.y = y_start;
      end_point.x = x_end;
      end_point.y = y_end;
  
      score = _score;
    }
    Line(const Point2f &_start_point, const Point2f &_end_point,
         const float _score = 0.0f) {
      start_point = _start_point;
      end_point = _end_point;
      score = _score;
    }
    Point2f start_point, end_point;
    float score;
  };
  ```
- 包含两个点以及一个score