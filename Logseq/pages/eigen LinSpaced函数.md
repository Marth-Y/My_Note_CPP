- 函数LinSpaced(size,low,high)只适用与向量和一维数组，它让某一大小的向量或数组的系数均匀地从低到高的填满，如下所示：
- ```cpp
  ArrayXXf table(10, 4);
  table.col(0) = ArrayXf::LinSpaced(10, 0, 90);
  table.col(1) = M_PI / 180 * table.col(0);
  table.col(2) = table.col(1).sin();
  table.col(3) = table.col(1).cos();
  std::cout << " Degrees Radians Sine Cosine\n";
  std::cout << table << std::endl;
  //output
  Degrees Radians Sine Cosine
  0 0 0 1
  10 0.175 0.174 0.985
  20 0.349 0.342 0.94
  30 0.524 0.5 0.866
  40 0.698 0.643 0.766
  50 0.873 0.766 0.643
  60 1.05 0.866 0.5
  70 1.22 0.94 0.342
  80 1.4 0.985 0.174
  90 1.57 1 -4.37e-08
  ```