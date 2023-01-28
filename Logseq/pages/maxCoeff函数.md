- 返回矩阵中最大值，参数为传入传出参数，返回最大值所在的行、列
- ```cpp
  int main()
  {
      MatrixXd Mat(4,4);
      MatrixXd::Index maxRow,maxCol;
      Mat << 0,1,2,3,
           4,5,6,7,
           8,9,70,80,
           15,56,48,1000;
      double max = Mat.maxCoeff(&maxRow,&maxCol);
      cout << "max = " << max << endl;
      cout << Mat << endl;
      cout << maxRow << " " << maxCol << endl;
  }
  ```
- output:
  background-color:: #497d46
- ```cpp
  max = 1000
     0    1    2    3
     4    5    6    7
     8    9   70   80
    15   56   48 1000
  3 3
  ```