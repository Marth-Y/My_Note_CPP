- # 一、静态数据成员
	- 位于全局/静态区，不会占据类的存储空间，是被整个类的**所有对象**所**共享**的
	- 静态数据成员不能在类体内直接进行初始化，静态数据成员受private控制符作用（初始化语句不受）
	- ```CPP
	  //staticComputer.cc
	  class Computer
	  {
	    ...
	  private:
	    static double _totalPrice;
	  };
	  //全局位置，不需要加static
	  //这是静态数据成员的初始化的固定写法。不受私有成员类外不能访问的约束
	  //虽然位置其位置是在类之外，但还是得理解成在类内部进行初始化。
	  double Computer::_totalPrice = 0;
	  
	  ```
- # 二、静态成员函数
	- 形式：在一个成员函数的前面加上`static`关键字
		- 不能是构造析构之类
	- 特点：
		- ==**1.静态成员函数没有隐含的this指针**==
			- 因为没有this指针，就找不到对象的地址，也就无法访问非静态数据成员和成员函数。
		- ==**2.不能直接访问非静态的数据成员和成员函数**==
		- ==**3.只能访问静态数据成员和静态成员函数**==
		- 4.因为静态成员函数是共享的，与某一个对象无关。所以我们一般直接用类名调用
			- `类名::函数`
- # 三、const成员函数
	- 形式：就是在成员函数的参数列表之后，函数执行体之前的位置加上const关键字，该函数就称为const成员函数
	- 作用：const成员函数无法对数据成员进行修改，具有只读属性
		- **即：为什么加上const后，就具有了只读属性？因为改变了this指针的形态，变成了双重限定**
			- 由编译器完成
	- ==**当const和非const重载时，只有const对象才能调用const版本的成员函数**==，非const对象调用非const版本的成员函数。
	- const对象不能调用非const版本的成员函数（因为非const版本函数内部可能对数据有修改）。
	- 非const对象可以调用const版本的成员函数
	- **所以只要该成员函数不会修改类的数据成员，该函数就应该被设置成const版本的。**保证const对象也可以访问。
	- ```CPP
	  constPoint.cc
	  void print()
	  {
	    ...可以和const成员函数构成重载
	  }
	  void print() const
	  {
	    this->_ix = 100;//error
	  }
	  所以他们的参数不一样。
	  上面参数：Point * const this
	  下面参数：Point const * const this  双重限定this指针
	  ```
- # 四、对象数组
	- ```CPP
	  Point points[5];//对象数组,会有五次构造函数调用
	  Point points[5] = {
	    Point(1,2),
	    Point(3,4),
	    ...
	  }
	  Point points[3] = {//因为类型已经确定了，所以只需要传数据就可以了
	    {1,2},
	    {1,3},
	    ...
	  }
	  ```
- # ==五、单例模式== #面试常考
	- 设计模式一共23种
	- 单例模式需求：某一个类只能产生一个实例
	- 如：Point类只能创建出一个对象 。没有考虑多线程。**最简单的设计模式，2分钟内写出来** ==必考题==
		- 1.构造函数私有化
		- 2.定义一个静态的指向本类型的指针  _pInstance
		- 3.在public区域定义一个静态成员函数  getInstance
		- 4.为了进行防御性的措施，防止误用。可以禁止复制
			- 将复制控制语义的函数私有化  C+++98
			- 将复制控制语义的函数声明为delete  C++11
				- `Singleton(const Singleton&) = delete;`
				- `Singleton& operator=(const Singleton&) = delete;`
		- ```CPP
		  class Point
		  {
		  public:
		    //1.为了能在外部创建对象：
		    Point getInstance()
		    {
		       Point pt(1,2);
		       return pt;//此时pt能够复制，自然也就不是唯一的。
		    }
		    //2.不能调用拷贝构造函数。所以可以删除拷贝构造函数
		    Point(const Point &) = delete;//C++11的用法。该方法在类中被删除掉
		    //3.现在就不能直接返回对象，因为拷贝构造被删了,所以可以返回指针,
		    //但是上面是局部对象，所以需要返回堆空间对象
		    Point* getInstance()
		    {
		       Point *pt = new Point(1,2);
		       return pt;
		    }
		    //4.现在没有对象，但又需要用对象调用函数创建一个对象，所以改为静态成员函数，可以用类名直接调
		    static Point* getInstance()
		    {
		       Point *pt = new Point(1,2);
		       return pt;
		    }
		    //5.此时可以多次调用这个函数，这样堆空间仍旧可以产生多个对象.所以我们需要
		    //只生成一个对象，所以只有第一次执行时，才创建对象，有需要一个值保存第一次创建的
		    //对象，又是在静态成员函数内，所以需要静态对象指针
		    //此时构造函数的参数需要变了，因为不一定是固定参数。所以创建init函数初始化数据成员。
		    //即二段式构造：先创建对象，再初始化
		    static Point* getInstance()
		    {
		      if(nullptr == _pInstance)
		      {
		         _pInstance = new Point();
		      }
		      return _pInstance;
		    }
		    void init(int ix,int iy)
		    {
		      _ix = ix;
		      _iy = iy;
		    }
		  private:
		    static Point* _pInstance;
		    
		    //6.此时考虑如何回收堆空间资源，虽然可以delete，但是只有一个对象，可能重复free，
		    //而且代码不对称，不优雅，前面没有new。所以我们不希望他自动调用，可以将析构私有化，单独去调用
		    //static使得其可以用类名调用，与创建对称
		  public:
		    static void destroy()
		    {
		      if(_pInstance)
		      {
		        delete _pInstance;
		        _pInstance = nullptr;//防止多次调用Point::destroy;Point::destroy;Point::destroy;
		        //这样重复delete。所以第一次delete之后要将指针 = nullptr;因为delete之后，只是销毁了
		        //申请的空间，是不会销毁指针的！变为了一个野指针。这是free的原理哦。
		        //这就是一个SAFE_FELETE
		      }
		    }
		  private:
		    ~Point(){}
		    
		    //7.补充：此时可能会发生*p1 = *p2;虽然他们是一样的，但我们不能让他编译通过，
		    //他会调赋值运算符函数。所以可以删除赋值运算符函数。
		    Point& operator=(const Point&) = delete;
		    
		    //8.delete之后要将指针变为空指针。SAFE_DELETE。
		    
		  private:
		    Point(int ix = 0,int iy = 0)//为了外部无法直接调用构造函数，将其私有化，
		      //不然外部能无限创造对象
		      :_ix(ix)
		      ,_iy(iy)
		      {}
		    int _ix;
		    int _iy;
		  };
		  
		  Point* Point::_pInstance = nullptr;
		  ```
	- ```CPP
	  单例对象模板
	  Singleton.cc
	  ```
	- 单例模式应用场景:
		- 在项目中，全局唯一的实例都可以用单例模式，如：
			- 1.以前使用**全局变量**的地方都可以用单例模式进行替换
			- 2.**配置文件**的读取和存储 11：48√
				- 因为它是全局唯一的。只有一个配置文件，所以可以用单例模式创建对象进行存储。
			- 3.搜索引擎中**网页库，词典库，倒排索引库**（全局唯一的，都可以考虑用单例模式）