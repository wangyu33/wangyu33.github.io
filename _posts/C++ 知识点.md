#### 1. C++类中this指针的用法

this是C++中的一个关键字，也是一个const指针，它指向当前对象，通过它可以访问当前对象的所有成员；

（1）this可以访问类的所有成员，包括private、protected、public；

（2）this是const指针，它的值不能被修改的。

（3）this只能在成员函数内部使用，不能在其他地方使用。

（4）只有当对象被创建后this才有意义，因此不能在static成员函数中使用。

（6）友元函数没有this，要访问非static成员时，需要对象做参数；要访问static成员或全局变量时，则不需要对象做参数；

https://blog.csdn.net/weixin_43751983/article/details/91147918

#### 2. C++ 友元函数

类的友元函数是**定义在类外部**，**但有权访问类的所有私有（private）成员和保护（protected）成员**。尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。

友元可以是一个函数，该函数被称为友元函数；友元也可以是一个类，该类被称为友元类，在这种情况下，整个类及其所有成员都是友元。

如果要声明函数为一个类的友元，需要在类定义中该函数原型前使用关键字 **friend**，如下所示：

```c++
class Box
{
   double width;
public:
   double length;
   friend void printWidth( Box box );
   void setWidth( double wid );
};
```

请看下面的程序：

实例

```C++
#include <iostream>  
using namespace std;  
class Box {   
	double width; 
public:   
	friend void printWidth( Box box );   
	void setWidth( double wid ); 
	};  
// 成员函数定义 
void Box::setWidth( double wid ) {    
	width = wid; 
}  
// 请注意：printWidth() 不是任何类的成员函数 
void printWidth( Box box ) {   
/* 因为 printWidth() 是 Box 的友元，它可以直接访问该类的任何成员 */   
	cout << "Width of box : " << box.width <<endl; 
}  // 程序的主函数 
int main( ) {   
	Box box;    
	// 使用成员函数设置宽度   
	box.setWidth(10.0);      
	// 使用友元函数输出宽度   
	printWidth( box );    
	return 0; 
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
Width of box : 10
```