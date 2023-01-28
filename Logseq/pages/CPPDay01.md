- ==面向对象四大特征：抽象、封装、继承、多态==
- # 一、C++发展历史
	- C语言诞生时间：1072
	- C++诞生时间：1983
	- 编译器：
		- 常见GCC、CLANG
		- CLANG的**编译速度**是GCC的5-10倍
		- 大型项目，光编译就可能是小时作为单位的。
	- 只需要学C++11/14之后的就可以了吗
		- 在企业中有很多的遗留项目，用的还是C++0x/C++98的标准。
		- 所以我们先从C++0x/C++98入手，逐步引入C++11/C++14的内容
	- C++擅长的领域
		- 1.通信领域
			- 华为、中兴、烽火
		- 2.金融领域
			- 与证券、股市相关的
		- 3.音视频领域
			- 直播、腾讯会议
		- 4.区块链
			- BTC/ETC/EOS
			- 有位道友年薪七位数
		- 5.智能驾驶
			- 目前薪资非常高
	- C++技术天花板很高，无35
- # 二、C++学习方式
	- C++primer当成**字典**使用
	- 跟着课程的节奏走，有不懂的翻一下primer
	- 课程本身是提取除了核心主干知识（**主餐**）
	- **敲代码**为主，通过**实际演练**是学习C++更好的方式
	-
	- C++primer更经典，片段式的代码，**常看常新**的
	- C++primer plus 每一个程序是完整的
	- C++必看书籍：
		- 《C++编程规范》、《Effective C++》、《More Effective C++》、《深度探索C++对象模型》（偏理论）
- # 三、C++的helloworld程序
	- 1.源程序后缀名.cc\.cpp，.hpp/.hh与C的头文件区分开来
	- 2.编译命令
		- `g++`命令同gcc
	- ```C
	  #include "stdio.h"//从当前路径下开始查找头文件，如果当前路径下没有，再从系统下查找
	  #include <stdio.h>//直接从系统路径下查找
	  
	  #include <iostream>//C++标准库头文件没有.h的后缀
	  //C++的头文件都是用模板实现的
	  
	  using namespace std;//standard 引入了C++的命名空间
	  
	  int main()
	  {
	    printf("hello,wolrd\n");
	    //cout是一个对象
	    //<<输出流运算符，本质也是一个函数
	    //endl 本质是一个函数 换行符
	    cout<<"hello,world"<<endl;
	  }
	  ```
- # 四、命名空间
	- ## 1.为什么提出命名空间？93年7月引入了命名空间
		- C语言在开发的过程中，如果涉及到多公司合作时，公司内部都会去对一些实体（变量、函数等）进行定义，这些实体都是采用相同的名字，那就会碰到**命名冲突**的问题
			- 以前的解决方案：做一个约定。不同人变量的命名加一个前缀
				- 大项目的变量名字会很冗余，很长、麻烦。
		- 因此C++引入了命名空间namespace
	- ## 2.什么是命名空间
		- 命名空间又称为名字空间，是程序员命名的内存区域，程序员根据需要指定一些有名字的空间域，把一些全局实体分别存放到各个命名空间中，从而与其他全局实体分隔开。通俗的说，每个名字空间都是一个名字空间域，存放在名字空间域中的全局实体只在本空间域内有效。名字空间对全局实体加以域的限制，从而合理的解决命名冲突。
		- ```C
		  namespace wd
		  {
		  int number = 0;//实际写的过程中，顶格写，不缩进,因为里面有很多东西
		  //还有其他函数，缩进就不好看，不合理了
		  void display()
		  {
		    cout<<"display()"<<endl;
		  }
		  }//end of namespace wd
		  
		  //假设不知道cout就是std空间中的实体，那就有可能写出以下代码
		  void cout()
		  {
		    printf("hello,cout\n");//再用cout就会报错二义性
		  }
		  
		  void test0()
		  {
		    cout<<"number:"<<number<<endl;//命名空间外无法访问wd中的number
		    cout<<"number:"<<wd::number<<endl;//要拿到需要作用域限定符 ::
		    wd::display();
		  }
		  ```
	- ## 3.命名空间的使用方式
		- 1. ::    作用域限定符
			- `wd::number`、`wd::display()`
		- 2.`using`编译指令
			- `using namespace wd;`
			- 特点：**他会把命名空间中的所有实体一次性全部移入**
			- 带来问题：当我们不知道命名空间中存在有那些实体的情况下，**依然会造成命名冲突的问题**。
				- **注意**：因此这种方式虽然简单，但对于初学者来说，不适合，因此不推荐使用。
		- 3.`using`声明机制-----==**大力推荐使用**==
			- `using std::cout;`、`using std::endl;`
			- 只会引入该空间中的某一个实体
			- 用哪一个实体就引入哪一个实体
	- ## 4.命名空间嵌套使用
		- **命名空间可以嵌套使用，有一个层级的概念**
		- ```Cpp
		  namespace wd
		  {
		  int number = 0;
		  void display()
		  {
		    cout<<"display()"<<endl;
		  }
		    
		  namespace lwh
		  {
		  int number;
		  void display()
		  {
		    cout<<"lwh,display()"<<endl;
		  }
		  }
		  }
		  //使用
		  wd::lwh::number;
		  wd::lwh::dislay();
		  ```
	- ## 5.C++怎么兼容C的？
		- 把C的标准库放入了一个匿名命名空间
			- 匿名命名空间(没有名字)
		- ```C
		  #include <cstdio>//C++对C头文件的一个封装
		  #include <sstring>
		  #include <cstdlib>
		  
		  #include <stdio.h>//推荐用这种，更好区分C和C++的头文件
		  
		  void test()
		  {
		    ::printf("hello,world\n");
		  }
		  ```
		- 我们也可以定义匿名命名空间
		- 为了在本模块（一个.c/.cpp文件）内部使用变量，类似于static变量的作用，不具备外部性
		- ```C
		  //全局变量
		  int number = 2;//可以跨模块调用
		  
		  static int number = 1;与下命名空间等同
		  namespace
		  {
		  int number = 1;//不能跨模块调用，相当于C语言中的static变量
		  //只能在本模块内使用
		  }//匿名命名空间
		  
		  void test()
		  {
		    cout<<number<<endl;//error
		  }
		  ```