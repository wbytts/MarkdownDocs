# Hello World

```c++
/*
    iostream包含了cout和endl等下面使用到的东西
*/
#include <iostream>

/*
 使用std名称空间
 如果不写这个，下面使用cout和endl等需要写成：
 	std::cout 和 std::endl 
*/
using namespace std;

int main(int argc, char* argv[])
{
	/*
		cout：输出流
		endl：换行
		<< ：流插入运算符
	*/
 
	cout << "Hello World" << endl;
	
	return 0;
}
```


