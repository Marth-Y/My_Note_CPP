#王道

- # 一、常量数组
  card-last-interval:: 4
  card-repeats:: 1
  card-ease-factor:: 2.6
  card-next-schedule:: 2022-07-13T01:16:25.335Z
  card-last-reviewed:: 2022-07-09T01:16:25.336Z
  card-last-score:: 5
  collapsed:: true
	- `const int arr[] = { 1,2,3,4,5,6,7,8,9 };`
	  collapsed:: true
		- `const int`说明数组元素不能够修改
		- 存放一些静态数据
	- 随机发牌，打印手牌（52张）
	  collapsed:: true
		- 首先考虑如何表示牌：两个属性：花色和点数
		  collapsed:: true
			- 用两个常量数组，一个表花色，一个表点数
		- 如何生成随机数？
		  collapsed:: true
			- rand()--->stdlib.h头文件中，返回伪随机数
			- srand()--->srand((unsigned int)time(NULL));--->因为rand伪随机，所以用srand，srand如果每次随机数种子一样，就会返回相同的序列，所以利用time函数生成不同的随机数种子。
			  collapsed:: true
				- time(NULL)返回从1970年1月1日0时0分0秒到现在的秒数
		- 如何避免生成重复的牌？
- # 二、函数
  collapsed:: true
	- 数学上的函数：必须有参数，必须有返回值，并且不会产生副作用。
	- C语言上的函数：可以没有参数，也可以没有返回值，并且还可能在函数调用过程中产生副作用。
	- **函数的定义**
	  collapsed:: true
		- ```cpp
		  return_type fn_name(parameter list){
		  	statements;//函数体
		  }
		  ```
	- ## 1.参数是如何传递的？（值传递） ^2ce56a
	  collapsed:: true
		- C语言中只有值传递（没有引用传递）
		  collapsed:: true
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/C语言只有值传递.png)
			-
		- 如何在函数中改变main函数的局部变量？
		  collapsed:: true
			- 指针
		- ### 1. 一维数组作为参数传递，^^**会退化为指针**^^。（本质就是一种类型转换）==指针传递还是值传递，是在函数内申请了一片空间，保存数组的地址，在其内更改指针的值，是不会影响到main函数内指针的值的，需要用到二级指针==
		  collapsed:: true
			- [[CDay08#^fe5894|不可以返回栈上的数组]]、[[CDay08#^a71c95|二级指针]]
			- 形式上是数组，但他其实是一个指针
			  collapsed:: true
				- ```CPP
				  int main()
				  {
				    int arr[]={...};
				    sum_arry(arr);
				  }
				  int sum_arry(int arr[])
				  {
				    //arr退化为指针！！sizeof(arr)=4。因为是int型指针，不一定是4，是int型指针的大小
				    ...
				  }
				  ```
			- 缺点：丢失了类型信息（是一个指针，不知道是数组）、丢失了数组长度细节
			- 优点：可以避免复制数据（节省内存空间，提高性能）、可以修改原数组的元素、函数调用会更加灵活
			  collapsed:: true
				- 比如数组长10，我只需要前五个元素，就可以只传递5的大小
				- `int sum_arr(int arr[],int n)//传n=5`
				- [[CDay11#^8e00ad|比如二叉搜索树那节，用前中序、中后序建树中，对于遍历的序列就不需要每次都传那么大，只需要传相应的大小就好。]]
		- ### 2.二维数组作为参数传递
		  collapsed:: true
			- 也会退化为指针。
			- ```CPP
			  intmain()
			  {
			    int arr[3][4]={...};
			    sum_arry(arr);
			  }
			  int sum_arry(int arr[][4],int n)
			  {
			    //可以省略行，但是不可以省略列
			    ...
			  }
			  ```
			- 为什么不能省略列的长度？
			  collapsed:: true
				- 因为求址公式：要用到，所以必须传(防用户不输)
				- `i_j_address = base_addr + i*columns*sizeof(elem_type)+j*sizeof(elem_type)`
			- 如何传递一个列数不固定的二维数组？
			  collapsed:: true
				- 二维数组是元素为一维数组的一维数组，所以可以传递元素是指针的一维数组，其指针指向一维数组。也就是**指针数组**
				  collapsed:: true
					- [[CDay10#^ddc4ae| 就像拉链法实现的哈希表的结构，一维数组存指针，每个指针指向一个一维数组]]
					- 其实可以就传递二维数组名，进去退化为指针，然后通过地址调用即可。但是要传递列数不固定的，还得是指针数组。
	- ## 2.返回值
	  collapsed:: true
		- 返回值的类型不能是数组类型
		  collapsed:: true
			- ```CPP
			  int[3] generate()
			  {
			    int arr[3]={...};
			    return arr;//报错
			  }
			  int* generate()
			  {
			    int arr[3]={...};
			    return arr;//这样可以
			  }
			  ```
			- 因为返回数组，需要拷贝，会浪费大量时间，所以没有这么设计
			- [[#^2ce56a| 2可以，但是不要返回栈上的数组，它随着栈帧的退出而销毁。]]
	- ## 3.程序如何终止
	  collapsed:: true
		- 操作系统调用main函数--->程序的开始
		- main函数将状态码返回给操作系统--->程序的终止
		- 如何不在main函数中终止程序
		  collapsed:: true
			- `void exit(int exit_code)`
			  collapsed:: true
				- `exit(0)`是正常结束
			-
- # 三、局部变量和外部变量
  collapsed:: true
	- 局部变量：定义在函数体里面的变量。--->**块作用域** { int i;...} **从变量定义开始，到块的末尾**。
	- 外部变量：定义在函数外面的变量。（C又称全局变量）--->**文件作用域**：**从变量定义开始，到文件的末尾。**
	  collapsed:: true
		- 定义在代码中间，就只有之后的代码能访问，前面的代码是无法访问的。
	- # 1.存储期限
	  collapsed:: true
		- 即：生命周期
		- 自动存储期限：存放在栈里的数据，具有自动存储期限；变量的生命周期随着栈帧的入栈而开始，随着栈帧的出栈而结束
		- 静态存储期限：拥有永久的存储单元，在程序的整个执行期间都存在
		- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/静态存储期限和自动存储期限.png)
		-
		- 外部变量：静态存储期限
		- 局部变量：默认是自动存储期限，可以通过static关键字设为静态存储期限。
		- ```CPP
		  //linux下
		  int main()
		  {
		    foo();
		    foo();
		    foo();
		  }
		  
		  foo()
		  {
		    int i;
		    printf("i = %d\n",i++);
		  }
		  ```
		- ```CPP
		  foo()
		  {
		    static int i;
		    printf("i = %d\n",i++);
		  }
		  ```
		- 第二个依次会打印 i++ 因为static将变量定义到了代码/数据段
- # ^^四、递归^^
  card-last-interval:: -1
  card-repeats:: 1
  card-ease-factor:: 2.5
  card-next-schedule:: 2022-07-06T16:00:00.000Z
  card-last-reviewed:: 2022-07-06T08:07:56.677Z
  card-last-score:: 1
  collapsed:: true
	- recursion = re + cur + sion (前缀+词根+后缀)
	  collapsed:: true
		- re：重复
		- cur：走
		- sion：名词后缀
		- 走重复的路
	- ### 1.求斐波那契数列
	  collapsed:: true
		- 如何避免重复计算
		  collapsed:: true
			- 顺序求解子问题，避免出现重复的计算--->动态规划
			- O(n)级时间复杂度
		- ```CPP
		  int main()
		  {
		  	long long n = fib2(50);
		  	printf("%lld\n", n);
		  	return 0;
		  }
		  
		  long long fib2(int n)
		  {
		  	if (n == 0)return 0;
		  	if (n == 1)return 1;
		  	long long a = 0, b = 1;
		  	for (int i = 2; i <= n; i++)
		  	{
		  		long long temp = a + b;
		  		a = b;
		  		b = temp;
		  	}
		  	return b;
		  }
		  ```
		- 能不能通过通项公式求解？
		  collapsed:: true
			- 不可以，浮点数不精确
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/斐波那契数列通项公式.png)
		- 有没有更快的方法？
		  collapsed:: true
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/矩阵求斐波那契数列.png)
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/矩阵求斐波那契数列2.png)
	- ### 2.汉诺塔问题
	  collapsed:: true
		- collapsed:: true
		  1. 如何移动？
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/汉诺塔问题递归公式.png)
			- ```CPP
			  void hanoi(int n, char start, char middle, char target)
			  {
			  	//边界条件：
			  	if (n == 1)
			  	{
			  		printf("%c--->%c\n", start, target);
			  		return 0;
			  	}
			  	//递归公式
			  	hanoi(n - 1, start, target, middle);
			  	printf("%c--->%c\n", start, target);
			  	hanoi(n - 1, middle, start, target);
			  }
			  ```
		- collapsed:: true
		  2. 最少移动多少次？
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/202206101614051.png)
			-
	- ### 3.约瑟夫环
	  collapsed:: true
		- ```CPP
		  //假如每两个人做掉一个人
		  int joseph(int n)
		  {
		  	//边界条件
		  	if (n == 1 || n == 2)return 1;
		  	//递归公式
		  	if (n & 1)//分奇偶
		  	{
		  		return 2 * joseph((n - 1) / 2) + 1;
		  	}
		  	else
		  	{
		  		return 2 * joseph(n / 2) - 1;
		  	}
		  }
		  ```
		- 更一般的情况：josephh(n,m)
		- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/约瑟夫环递归.png)
		- 总结：
		  collapsed:: true
			- collapsed:: true
			  1. 什么情况下可以考虑使用递归？
				- 一个大问题可以分解成若干个小问题，且小问题的求解方式与大问题一致（递）
				- 可以将小问题的解，合并成大问题的解。
			- collapsed:: true
			  2. 使用递归需要注意的问题
				- 边界条件：避免stackoverflow--->stackoverflow.com
				- 重复计算问题
			- collapsed:: true
			  3. 如何写递归
				- 边界条件
				- ^^**递推公式**^^
	-
	-
- # ^^**五、指针基础**^^
  collapsed:: true
	- 指针基础、指针与数组的关系、指针的高级应用
	- ## 1.指针基础
	  collapsed:: true
		- 计算机的最小寻址单位：字节
		- 变量的地址：变量第一个字节的地址。
		- 指针：简单来说，指针就是地址。
		- 指针变量：存放地址的变量。有时候会把指针变量称为指针
		-
		- **指针变量只是存放了变量的首地址**，那么如何通过指针访问指针指向的对象
		  collapsed:: true
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/指针变量存储首地址.png)
			- 声明指针时，需要指定指针的基础类型（指针指向对象的类型）
			- `int *p;`
			  collapsed:: true
				- int:基础类型
				- 注意事项：
				  collapsed:: true
					- 1. 变量名为：p，而不是*p
					- 变量类型是`int*`，而不是int
					- `const int *`和`const * int`
					-
		- **两个基本操作：&和\***
		  collapsed:: true
			- 取地址运算符：&（`int i = 1; int *p = &i;`*p相当于i的别名，修改*p就是修改i）
			  collapsed:: true
				- 区别：i是直接访问该内存空间（访问一次内存），*p是间接访问（访问两次内存）
			- 解引用运算符：*（通过指针可以访问指针指向的对象）
			-
		- **野指针问题**
		  collapsed:: true
			- 野指针：未初始化的指针或者是指向未知区域的指针。
			  collapsed:: true
				- `int *p; int *q = 0x7F;`都是野指针，因为我们不写OS，所以q的具体值是多少不是我们说的算的，所以也是野指针。
			- **注意事项：对野指针进行解引用运算，会导致未定义的行为**
			  collapsed:: true
				- 编译器不会野指针报错，但只要解引用就会出错。
				-
		- **指针变量的赋值**
		  collapsed:: true
			- collapsed:: true
			  1. 使用取地址运算符
				- `p = &i`
			- collapsed:: true
			  2. 通过另外一个指针变量赋值
				- `p = q;`
			- 注意事项：`p = q`和`*p = *q`不一样，一个是指针指向赋值，一个是解引用赋值。
		-
		- **指针作为参数**
		  collapsed:: true
			- 好处：可以在被调用函数中，通过指针修改主调函数中的值。
			- 指针就是为了弥补C语言值传递的缺陷