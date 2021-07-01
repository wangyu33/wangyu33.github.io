# JAVA

alt+enter修改错误

javadoc -encoding UTF-8 -charset UTF-8 Doc.java 生成DOC文件

#### 修饰符

类变量

static 	可以在类函数中直接使用，无需 new

常量

final	设置后不允许改变



#### 命名规范

```
所有变量、方法、类名:见名知意
类成员变量:首字母小写和驼峰原则: monthSalary 除了第一个单词以外，后面的单词首字母大写lastname lastName
局部变量:首字母小写和驼峰原则
常量:大写字母和下划线:MAX_VALUE
类名:首字母大写和驼峰原则: Man,GoodMan
方法名:首字母小写和驼峰原则: run(). runRun()
```

#### 运算符

三元运算符

```
x?y:z
如果x为真为y否则为z
```

#### 包机制

package本质为文件夹，为了解决定义重复

命名机制 一般利用**公司域名倒置**作为包名 如  com.baidu

为了能够使用某一个包的成员，我们需要在Java程序中明确导入该包。使用"import"语句可完成此功能



#### 流程控制

```
Scanner对象 获取用户输入  java.util.Scanner

基本用法
scanner s = new Scanner(System.in);

通过Scanner类的next()与nextLine()方法获取输入的字符串，在读取前我们一般需要使用hasNext()hasNextLine()判断是否还有输入的数据。

```

