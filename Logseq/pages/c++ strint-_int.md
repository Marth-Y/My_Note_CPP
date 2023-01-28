title:: c++ strint->int

- 以编程细节看成败
- C++项目常用的需求是将string转为int，但是编程者经常不考虑异常情况，这里说一下几种方法的注意事项：
- 1. std::stoi
- std::stoi函数将字符串转换为整数。stoi函数是在 C++11 中引入的。它分别在错误输入或整数溢出时引发std::invalid_argument或std::out_of_range异常。值得注意的是，它会将字符串转换10xyz为整数10。用它的时候，必须加catch异常的情况！
- #include <iostream>
  #include <string>
  
  int main()
  {
    std::string s = "10";
  
    try {
        int i = std::stoi(s);
        std::cout << i << std::endl;
    }
    catch (std::invalid_argument const &e) {
        std::cout << "Bad input: std::invalid_argument thrown" << std::endl;
    }
    catch (std::out_of_range const &e) {
        std::cout << "Integer overflow: std::out_of_range thrown" << std::endl;
    }
  
    return 0;
  }
  2.std::stringstream(推荐使用）
- std::stringstream可用于转换std::string为其他数据类型，与std::stoi相同的问题，即它会将字符串转换10xyz为 整数10。If the value of the string can't be represented as an int, 0 is returned.
- #include <iostream>
  #include <string>
  #include <sstream>
  
  int main()
  {
    std::string s = "10";
  
    int i;
    std::istringstream(s) >> i;
  
    std::cout << i << std::endl;
  
    return 0;
  }
  3.使用 boost::lexical_cast
- boost::bad_lexical_cast当无法构造有效整数或溢出时，报异常。（不常用）
- #include <iostream>
  #include <boost/lexical_cast.hpp>
  #include <string>
  
  int main() {
  
    std::string s = "10";
    int i;
  
    try {
        i = boost::lexical_cast<int>(s);
        std::cout << i << std::endl;
    }
    catch (boost::bad_lexical_cast const &e) {        // bad input
        std::cout << "error" << std::endl;
    }
  
    return 0;
  }
  4.std::atoi
- std::atoi函数将字符串转换为 int。它需要一个 C 字符串作为参数，所以我们需要将我们的字符串转换为 C 字符串。我们可以使用std::string::c_str.
- 请注意，如果转换后的值超出整数数据类型的范围，则其行为未定义。如果字符串的值不能表示为 int，则返回 0。
- #include <iostream>
  #include <string>
  #include <cstdlib>
  
  int main() {
  
    std::string s = "10";
    int i = std::atoi(s.c_str());
  
    std::cout << i << std::endl;
  
    return 0;
  }
  5. 使用sscanf()功能
- #include <iostream>
  #include <cstdio>
  #include <string>
  
  int main() {
  
    std::string s = "10";
    int i;
  
    if (sscanf(s.c_str(), "%d", &i) == 1) {
        std::cout << i << std::endl;
    }
    else {
        std::cout << "Bad Input";
    }
  
    return 0;
  }
  6.from_chars
- from_chars方法是 C++17 中对utilities库的补充，定义在<charconv>头文件中。它可以分析具有指定模式的字符序列。这样做的好处是处理时间更快，并且在解析失败的情况下能更好地处理异常。虽然对输入字符串有一些限制，即：前导空格/其他字符，甚至基数 16 的0x/0X前缀也不能处理。对于有符号的整数，只识别前导减号。
- from_chars(const char* first, 
          const char* last, 
          TYPE &value,
          int base = 10) 
  #include <iostream>
  #include <string>
  #include <charconv>
- using std::cout;
  using std::cin;
  using std::endl
  using std::string;
  using std::stoi;
  using std::from_chars;
- int main(){
    string s1 = "123";     string s2 = "-123";
    string s3 = "123xyz";  string s4 = "-123xyz";
    string s5 = "7B";      string s6 = "-7B";
    string s7 = "1111011"; string s8 = "-1111011";
    string s9 = "    123"; string s10 = "x123";
- int num1, num2, num3, num4, num5, num6, num7, num8, num9, num10;
- from_chars(s1.c_str(), s1.c_str() + s1.length(), num1);
    from_chars(s2.c_str(), s2.c_str() + s2.length(), num2);
    from_chars(s3.c_str(), s3.c_str() + s3.length(), num3);
    from_chars(s4.c_str(), s4.c_str() + s4.length(), num4);
    from_chars(s5.c_str(), s5.c_str() + s5.length(), num5, 16);
    from_chars(s6.c_str(), s6.c_str() + s6.length(), num6, 16);
    from_chars(s7.c_str(), s7.c_str() + s7.length(), num7, 2);
    from_chars(s8.c_str(), s8.c_str() + s8.length(), num8, 2);
    from_chars(s9.c_str(), s9.c_str() + s9.length(), num9);
    from_chars(s10.c_str(), s10.c_str() + s10.length(), num10);
- cout << "num1: " << num1 << " | num2: " << num2 << endl;
    cout << "num3: " << num3 << " | num4: " << num4 << endl;
    cout << "num5: " << num5 << " | num6: " << num6 << endl;
    cout << "num7: " << num7 << " | num8: " << num8 << endl;
    cout << "num9: " << num9 << " | num10: " << num10 << endl;
- return EXIT_SUCCESS;
  }