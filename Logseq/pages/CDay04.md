#王道

- # 一、算术运算符
  collapsed:: true
	- +-*/%
	- 注意事项：
	  collapsed:: true
		- 1. +-*/可以用于整数和浮点数，但是%只能用于整数
		- 两个整数相除，结果为整数。（向零取整）
		  collapsed:: true
			- ```CPP
			  	int i = 5 / 3;
			  	int j = -5 / 3;
			  ```
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/取余.png)
		- `i%j`的结果可能为负，符号与被除数 i 相同
		  collapsed:: true
			- ![](https://marth.oss-cn-shenzhen.aliyuncs.com/Mypicture/取余结果验证.png)
			- 是因为为了满足公式：i%j = i-(i/j)*j
	- 判断奇数：
	  collapsed:: true
		- ```CPP
		  bool is_odd(int n)
		  {
		  	return n & 1;
		  }
		  bool is_odd(int n)
		  {
		  	return n % 2 ！= 0;
		  }
		  ```
	-
- # 二、赋值运算符
  collapsed:: true
	- 简单赋值：=
	- v=e
	  collapsed:: true
		- 把表达式e的值赋值给变量v（副作用），整个表达式的值为赋值之后变量v的值
	- **注意事项**
	  collapsed:: true
		- collapsed:: true
		  1. 在赋值过程中，可能发生隐式类型转换
			- `int i = 3.14;`但实际上 i 是3；发生了隐式类型转换
		- 2.从右向左结合
		  collapsed:: true
			- ```CPP
			  float f;
			  int i;
			  f= i =3.33F;
			  /*
			   *从右向左结合，所以其实相当于：f=(i=3.33F);
			   *()内就是赋值之后i的值，3
			   *所以f=3
			   */
			  ```
	- **复合赋值运算符**
	  collapsed:: true
		- 如：+=、&=...
	-
- # 三、^^自增和自减运算符^^
  card-last-interval:: 4
  card-repeats:: 1
  card-ease-factor:: 2.36
  card-next-schedule:: 2022-07-13T01:08:55.529Z
  card-last-reviewed:: 2022-07-09T01:08:55.531Z
  card-last-score:: 3
  collapsed:: true
	- 自增：i++、++i
	- 自减：i--、--i
	- i++：整个表达式的值为 i ，副作用是 i 自增。
	- ++i：整个表达式的值为i+1，副作用是 i 自增。
	  collapsed:: true
		- ```CPP
		  int i = 1;
		  printf("%d\n", i++);
		  printf("%d\n", i);
		  printf("%d\n", ++i);
		  printf("%d\n", i);
		  /*
		   *1 2 3 3
		   */
		  ```
	- ^^注意事项：^^
	  collapsed:: true
		- 1. i=i++（会产生未定义的行为）结果是什么不知道
		- ```CPP
		  j=i++;
		  /*
		   *模拟汇编：temp，一块寄存器
		   *temp = i;
		   *j = temp;
		   *i=i+1;
		   *4、5两句是可以交换顺序的
		   *temp = i;
		   *i=i+1;
		   *j=temp;
		   */
		  ```
		- `j=(i++)+(i++);`C语言是：当一条语句执行完后，i 肯定会自增，没有规定i什么时候自增。所以第一个位置执行完后i有没有自增是不知道的，所以也会产生未定义的行为
		- `a[i]=b[i++];`会产生未定义的行为，不知道 i 到底有没有+1，[]也是运算符
		  collapsed:: true
			- `a[i]=b[i]; i++;`正确做法
	- i--：整个表达式的值为 i ，副作用是 i 自减。
	- --i：整个表达式的值为i+1，副作用是 i 自减。
	  collapsed:: true
		-
- # 四、关系运算符
  collapsed:: true
	- <、<=、>、>=
	- 其运算结果要么为0（false），要么为1（true）
	- **注意事项**
	  collapsed:: true
		- 表示范围i<j<k。先计算 i < j （0或1），然后把其结果和 k 比较。
		-
- # 五、判等运算符
  collapsed:: true
	- ==，!=
	- 其结果要么为0，要么为1
- # 六、逻辑运算符
  collapsed:: true
	- &&、||、!
	- **注意事项**
	  collapsed:: true
		- &&和||会发生短路现象（规定了求值顺序）
	- 好处：方便程序员编写程序
	  collapsed:: true
		- ```cpp
		  int i = 3, j = 0;
		  if(j!=0)
		  {
		    if(i/j>3)
		    {}
		  }
		  //利用短路原则优化:
		  if(j!=0 && i/j>3){}
		  ```
	-
- # ^^七、位运算符^^
  collapsed:: true
	- <<、>>、&、^、~
	- ## 1.移位运算符
	  collapsed:: true
		- i<<j：将 i 左移 j 位，在右边补0.
		  collapsed:: true
			- 若没有发生溢出，左移j位，相当于乘以2^j
		- i>>j：将 i 右移 j 位，
		  collapsed:: true
			- 若 i 为无符号整数或非负数，则左边补0.
			- 若 i 为负数，则他的行为由具体实现决定，有的补0，有的补1。
		- ^^**注意事项**^^
		  collapsed:: true
			- ^^为了可移植性，不要对有符号整数进行移位运算。^^
		- （非负数）右移j位，相当于除以2^j（向下取整）
	- ## 2.按位运算符
	  collapsed:: true
		- i ^ j
		  collapsed:: true
			- ^^**特性:**^^
			- a^0=a
			- a^a=0
			- a^b=b^a（交换性）
			- (a^b)^c=a^(b^c)（结合性）
		- 正是因为异或这些特性，所以它常用于加密领域
		-
		- ### 1.如何判断一个数是否是奇数？
		  collapsed:: true
			- `n & 1`
		- ### 2.如何判断一个整数是否为2的幂？ 1 2 4 8...
		  collapsed:: true
			- ```CPP
			  bool isPowerOf2(int n)
			  {
			      unsigned i = 1;
			      while(i<n)
			      {
			          i<<=1;
			      }
			      return i==n;
			  }
			  ```
			- 2的幂的二进制表示只有一个1，其余位为0.
			  collapsed:: true
				- ```CPP
				  bool isPowerOf2(int n)
				  {
				      return n&(n-1)==0;
				  }
				  ```
		- ### 3.给定一个不为0的整数，找到值为1且权重最低的位（last set bit）
		  collapsed:: true
			- 与补码按位与，n&(-n)
		- ### 4.给定一个整数数组，里面的数都是成双成对的，只有一个数例外
		  collapsed:: true
			- 全部异或
		- ### 5.给定一个整数数组，里面的数都是成双成对的，只有两个数例外找出这两个数
		  collapsed:: true
			- 可以分类：
			  collapsed:: true
				- a1,a1,s1,a2,a2...分一类，
				- b1,b1,b2,b2,s2...分一类
			- 如何分类？
			  collapsed:: true
				- r = s1^s2≠0，r至少有一位为1.找到r的last set bit，s1,s2在这一位不相同。
				- 根据这一位将所有数据分成两类
				-
- # 八、语句
  card-last-interval:: 4
  card-repeats:: 1
  card-ease-factor:: 2.6
  card-next-schedule:: 2022-07-13T01:16:04.379Z
  card-last-reviewed:: 2022-07-09T01:16:04.379Z
  card-last-score:: 5
  collapsed:: true
	- 表达式语句（表达式后面添加;）
	- 选择语句（if、switch）
	- 循环语句（while、do、for）
	- 跳转语句（contiue、break、goto、return）
	- 复合语句（{statements}将多条语句用大括号括起来，组成一条语句）
	- 空语句（一个;）
	-
- # 九、选择语句
  card-last-interval:: 4
  card-repeats:: 1
  card-ease-factor:: 2.6
  card-next-schedule:: 2022-07-11T01:10:58.212Z
  card-last-reviewed:: 2022-07-07T01:10:58.213Z
  card-last-score:: 5
  collapsed:: true
	- ## 1.if语句
	  collapsed:: true
		- 格式1：`if(expr)statement`
		  collapsed:: true
			- 如何容if语句控制多条语句？--->复合语句
		- 格式2：
		  collapsed:: true
			- ```CPP
			  if(expr)statement1
			  else statement2
			  ```
		- 级联式if语句本质上是if else的嵌套
		  collapsed:: true
			- ```CPP
			  if()
			    ...;
			  eles
			  	if()
			        ...;
			  	else
			        ...;
			  
			  //级联式if语句
			  if()
			    ...;
			  eles if()
			    ...;
			  else
			    ...;
			  ```
	- ## 2.switch语句
	  collapsed:: true
		- switch：旋钮式开关
		- 最常见的格式：
		- ```CPP
		  switch(expr)
		  {
		    case const_expr:statements
		    case const_expr:statements
		    case const_expr:statements
		    default:statements
		  }
		  //const_expr(一定是编译期间就可以知道的常量表达式)
		  ```
		- **注意事项**
		  collapsed:: true
			- 1. expr必须是整数类型（char也是整数），不能是字符串（\=\=比较的是地址）和浮点数（因为switch和case是用=\=比较的）
			- 2. case后面的标签必须在编译期间能够求值
			- 3. 不能够有重复的标签
			- 4. 多个标签可以共用一组语句
			- collapsed:: true
			  5. 可以省略break语句，可能会出现case穿越现象
				- 是因为switch比较找到case语句后，就会一路执行，不会再次比较
		-
	- ^^**级联式if语句和switch语句的比较**^^
	  collapsed:: true
		- 级联式if语句更加通用
		- switch语句的可读性高
		- switch语句的执行效率比级联式if语句高
		  collapsed:: true
			- switch语句的case比较多时，switch会查表，相当于直接跳到对应分支，只比较一次，而if语句要一个一个比。
	- ## 3.条件运算符（三目运算符）
	  collapsed:: true
		- `?:;`
		- 格式 `expr1?expr2:expr3;`
		- 规定了求值顺序
		- 计算步骤：计算expr1的值，若非0（true），则计算expr2的值，并把expr2的值当作整个表达式的值；若为0，则计算expr3的值，并把expr3的值当作整个表达式的值
		- **结合性：从右向左结合**
		- ```CPP
		  expr1?expr2:expr3?expr4:expr5?expr6:expr7;
		  (expr1?expr2:(expr3?expr4:(expr5?expr6:expr7)));
		  ```
		-
- # 十、循环语句
  collapsed:: true
	- ## 1.while语句
	  collapsed:: true
		- 格式：`while(expr)statement`
		  collapsed:: true
			- expr:控制表达式
			- statement：循环体
	- ## 2.do语句
	  collapsed:: true
		- 格式：`do statement while(expr)`
	-
	- **do与while语句唯一区别**：当初始条件不成立时，while语句不会执行循环体，但是do语句会执行一次。
	-
	- ## 3.for语句
	  collapsed:: true
		- 格式`for(expr1;expr2;expr3)statement`
		  collapsed:: true
			- expr1：初始化表达式，只会执行一次
			- expr2：控制表达式，若为真则执行循环体，若为假则结束循环
			- expr3：执行循环体之后要执行的步骤
		- 注意事项：expr1，expr2，expr3都可以省略。如果省略expr2，默认值为true
		  collapsed:: true
			- **惯用法：**`for(;;){ }`--->无限循环
			- `while(1){ }`--->无限循环
- # 十一、跳转语句
  collapsed:: true
	- ## 1.break语句
	  collapsed:: true
		- 作用：跳出switch语句和循环语句。
		- 注意事项：如果switch语句和循环语句嵌套的时候，break只能跳出包含break语句的最内层嵌套。
		- 如何能跳出外层的while语句？
		- collapsed:: true
		  ```cpp
		  while()
		  {
		    ...
		    switch()
		    {
		        ...
		        break;
		    }
		    ...
		  }
		  ```
			- 从break跳出while循环`goto`!
		-
	- ## 2.continue语句
	  collapsed:: true
		- continue语句和break语句的区别
		  collapsed:: true
			- 1. break语句可以用于switch和循环语句，但是continue只能用于循环语句
			- 2. break跳出的是整个循环，continue语句是跳转到循环体的末尾。
			- ```CPP
			  for(int i = 0;i<10;i++)
			  {
			    int n;
			    scanf("%d",&n);
			    if(n==2)
			    {
			      continue;
			    }
			  }
			  continue是跳到末尾，仍旧会执行上面的i++
			  ```
	- ## 3.goto语句
	  collapsed:: true
		- goto可以在函数内进行任意跳转。（只能在同一个函数内）
		- 为什么还需要break语句和continue语句？
		  collapsed:: true
			- goto可读性太差
			- goto很容易导致一些bug。《goto statement considered harmful》
		- 最佳实践：尽量少用goto语句
		- 格式：`goto 标签;`
		- 使用场景
		  collapsed:: true
			- 跳出外层嵌套
			  collapsed:: true
				- ```CPP
				  while()
				  {
				    switch()
				    {
				        ...
				        goto loop_done;
				    }
				  }
				  loop_done:
				  	...
				  ```
			- 错误处理：
			  collapsed:: true
				- ```cpp
				  if(...)
				  {
				    goto error_handle;
				  }
				  ...
				  erroe_handle:
				  	进行错误处理
				  ```
		-
- # 十二、数组
  collapsed:: true
	- 请用自己的语言描述数组的模型
	  collapsed:: true
		- 连续的内存空间，并且这片内存空间被划分为大小相等的小空间。
	- 数组是如何划分为大小相等的小空间的？为什么要这么做？
	  collapsed:: true
		- 存储同一种类型的数据 以划分相同空间
		- 随机访问元素：在O(1)时间内访问数组的任意一个元素
	- 为什么在大多数语言中，数组的索引都是从0开始的？
	  collapsed:: true
		- 早期是为了节省运算资源，少做i-1的运算。
		- 寻址公式：`i_addr=base_addr+i*sizeof(elem_type)` i从0
		- 寻址公式：`i_addr=base_addr+(i-1)*sizeof(elem_type)` i从1
		- 现在有从1开始的，他省略了0号空间，仍旧用i从0的寻址公式
	- 为什么说数组的效率优于链表？（一般来说）
	  collapsed:: true
		- 数组可以更好地利用CPU的Cache（局部性原理，预读）
		- 数组只需要存储数据，链表还需要存储指针域
	- ## 1.数组的声明
	  collapsed:: true
		- `elem_type arr_name[len];`
		- `int arr[10];`
		  collapsed:: true
			- len的值必须在编译期间能够确定大小
		-
	- ## 2.数组的初始化
	  collapsed:: true
		- ```CPP
		  int arr[10] = {1,2,3,4,5,6,7,8,9,10};
		  int arr[10] = {1,2,3};
		  int arr[10] = {0};
		  int arr[] = {1,2,3,4,5,6,7,8,9,10};//大小：10
		  //编译器在编译阶段会根据数组的初始化式{...}，决定数组的大小
		  ```
		-
		-
	- ## 3.对数组使用sizeof运算符
	  collapsed:: true
		- `int arr[10] = {1,2,3,4,5,6,7,8,9,10};`
		- arr：大小为10的整型数组
		- sizeof(arr)=40
		- `int len = sizeof(arr)/sizeof(arr[0]);`可以得到数组的长度
		  collapsed:: true
			- `arr[0]`是因为不知道数组类型
			- 很常用，可以封装为宏函数
			  collapsed:: true
				- ```CPP
				  #define SIZE(a) (sizeof(a)/sizeof(a[0]))
				  ```
		-
	-
- # 十三.多维数组（二维数组）
  collapsed:: true
	- 类比矩阵（逻辑上）
	- `int matrix[3][4];`
	- 物理上二维数组的内存空间是连续的。（行优先存储）
	- ## 1.二维数组的初始化（元素是一维数组的数组）
	  collapsed:: true
		- ```CPP
		  int matrix[3][4] = {{1,2,3,4},{2,2,3,4},{3,2,3,4}};
		  int matrix[3][4] = {{1,2,3,4},{2,2,3,4}};
		  int matrix[3][4] = {{1,2,3},{2,2,3}};
		  int matrix[3][4] = {1,2,3,4,2,2,3,4,3,2,3,4};//不推荐,写错就完了
		  int matrix[3][4] = {0};
		  int matrix[][] = {{1,2,3,4},{2,2,3,4},{3,2,3,4}};//不行,列的大小不能省略
		  int matrix[][4] = {{1,2,3,4},{2,2,3,4},{3,2,3,4}};
		  ```
		-
		-