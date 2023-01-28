- 一条车道线占一个channel，
	- keeping 状态有六条车道线，六个channel
		- [[draws/2022-11-24-18-30-54.excalidraw]]
	- channing 状态有五条车道，留个channel：
		- [[draws/2022-11-24-19-01-54.excalidraw]]
- `SetFeatureBuffer(inference_features);`
	- 我们的推理结果是在GPU上面的，因此需要将GPU上的推理结果图移入CPU中，
	- ```cpp
	  auto &dst = host_inference_features_[cam_pos];
	  const auto &src = inference_features.at(cam_pos);
	  #ifdef CUDA_ACC
	  	从设备移入CPU
	  #else
	      dst = src;//这是我们测试，此时图片就在主机上不用移动了
	  #endif
	  ```
- `RunOnce`
	- 先将所有推理的结果图加入`feature_ptrs`
	- `static common::EigenArray grid_idx = Eigen::RowVectorXf::LinSpaced( uf_lane_grid_nums_, 0, uf_lane_grid_nums_ - 1);`
		- 这里是对grid_idx取0~158的值，grid的取值范围在配置文件中设定好了为159.因为最后一个格子要作为标记这条van有没有车道线，所以不能选用。
	- `UltraFastSoftmax(feature_ptrs[c], grid_idx, c, lane_uf_pts[c]);`
		- 对每一个channel图（可以理解为一条小箱子）将其转化为2D的坐标系上的点。
		- ```cpp
		  Eigen::MatrixXd::Index maxRow, maxCol;
		    for (int h = input.rows() - 1; h >= 0; --h) {
		      int y = UFv_[h];//y值是设定好的，就是vanchor中的值。
		      common::EigenArray anchor = input.row(h);这里是读取channel的一行小格子
		      double max = anchor.maxCoeff(&maxRow, &maxCol);找出该行最大值
		      if (maxCol < input.cols() - 1) {//最大行不是最后一行：
		        common::EigenArray tmp = anchor.leftCols(input.cols() - 1).exp();取除最后一行之前所有行（最后一行是标记位），做softmax算法
		        anchor = tmp / tmp.sum();//得到概率值
		        // almost 3 sigma from 1w frames
		        float prb = anchor(maxCol);//获取最大的那行的概率
		        if (prb < uf_prb_th_) {
		          // out.emplace_back(common::Point3f(-1.f, y, 0.f));
		          continue;
		        }
		        anchor = anchor * grid_idx; anchor概率*grid_idx。这样他们的和会越靠近最大值所在的位置
		        float x = (anchor.sum() + 0.5) * (net_input_width_ / uf_lane_grid_nums_);+0.5是一个经验值。*后面的东西是为了转化为二维的坐标系的点。
		        out.emplace_back(common::Point3f(x, y, prb));//x是计算出的，y是一个设定好的值，因为取的是一个固定的vanchor格子行，prb是置信度，一个概率。
		      } else {
		        // out.emplace_back(common::Point3f(-1.f, y, 0.f));//如果最大行是最后一行，那么就无效，不做任何操作
		      }
		    }
		  ```
		- 最终的结果，也就相当于是把3D的点转化为了2D的坐标系的图，存入uv中。
	- `GatherLanePts(lane_uf_pts, lane_uv_pts);`
		- ==做车道线进行合并，就是合并7和9或者8和9（channing场景下channel）==
			- [[draws/2022-11-24-18-48-16.excalidraw]]
		- ```cpp
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
		  ```
			- 从这里就可以看出会将7和9合并（如果有7），或者将8和9合并（如果有8）。
			- 合并是从下网上进行合并的，从最底下一行坐标，一行一行往上进行合并。
			- 其中的`case_7_reserved`和`case_8_reserved`是因为新的方式中不管7或者8存不存在，都需要设置一条线，暂时不用管。
	- `SelectActiveScene(lane_uv_pts, active_scene);`
		- 对车道线场景进行分类是`keeping`还是`channing`。
		- 其做法是：先会对keeping、和channing场景的channel都会进行模型分析出一个图，然后分别计算图中每一行的点的前面算出的概率值的和（置信度的和），
		- 然后：`score_keeping /= (uf_lane_pt_nums_ * uv_lane_keeping_nums_);`，除以该图中车道线占用的所有的点，即：`uf_lane_nums_`：对一条车道线每一行取一个点（模型去取的，不用管取那个）。`uv_lane_keeping_nums_`：车道线的数量。这样相乘计算出的就是所有keeping场景下所有车道线占用点的数量。除以他去求一个均值score。然后比较这个score，大的就是所在的场景了。
		-