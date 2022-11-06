# 1 C++学习记录

## 1.1 指针与引用

引用在语法层不占用内存，在底层占用内存使用指针进行实现。

## 1.2 复制构造函数

```c++
class Base{
private:
    char* m_Buffer;
    int m_Size;
public：
    Base(const char* string){
    
    	m_Size=strlen(string);
    	m_Buffer=new char[m_Size+1];
    	memcpy(m_Buffer,string,m_Size);
    	m_Buffer[m_Size]=0;
    	
	}
    //复制构造函数
    Base(const Base& string):
    	m_Size=(string.m_Size)
    {
        m_Buffer=new char[m_Size+1];
        memcpy(m_Buffer,string,m_Size+1);
         
    }
}
```

## 1.3 动态链接与静态链接

+ 静态链接是将**.lib**文件写入到**.exe**文件中

+ .dll文件可能有配套的.lib文件存放对.dll文件中的函数等进行定位

## 1.4 智能指针

+ unique_ptr，不能够被复制

```c++
#include<memory>
std::unique_ptr<Base> base_ptr=std::make_unique<Base>()
```

+ share_ptr，能够被复制

+ weak_ptr

## 1.5 vector

  

+ **vector**在进行***push_back***的过程中如果容量不足，将会进行复制。将容器内的元素复制到另外一片内存区域，因此推荐使用***erase_back***进行入队操作

+ **vector**使用***reserve***进行容量的申请

## 1.6 函数指针

+ **处理回调函数（callback）**

```cpp
#include<iostream>
#include<vector>
#include<string>
void PrintMessage(int value) {
    std::cout << "value is" << value << std::endl;
}
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (const int &i:values) {
        function(i);
    }
}
int main() {
    std::vector<int> values = { 1,2,1 };
    void(*function)(int) = PrintMessage;//这样可以
    typedef void(*printFunction)(int);//取个别名
    printFunction Print = PrintMessage;//命名变量
    ForEach(values, PrintMessage);
    ForEach(values, function);
    ForEach(values, Print);
    ForEach(values, [](int value) {
        std::cout << "value is" << value << std::endl;
        });//使用lamada函数进行传递
}

```

+ **lamada**函数（匿名函数）**[ ]**进行捕获，捕获变量

  ```cpp
  auto lambda=[](int value){dosomething;} ;
  int a = 5;
  auto lambda2 = [&](int value){std::cout<<a;}//使用引用的方式捕获到a
  ```

## 1.7 多线程

### 1.7.1 多线程使用

**sd::thread** 类

**std::this_thread** 类

  ```c++
  #include<thread>
  void DoWork(){
      using namespace std::literals::chrono_literals;
      while(true){
          std::cout<<"working...";
          std::this_thread::sleep_for(1s);
      }
  }
  int main()
  {
      std::thread work(DoWork);//开启一个线程
      work.join();//等同于wait将主线程阻塞在这里，直到work线程完成
  }
  ```

  + 计时器

  ```cpp
  #include<iostream>
  #include<thread>
  #include<chrono>
  
  struct Timer{
      
      Timer(){
          
          
      }
      ~Timer(){
          
      }
  }
  int main(){
     using namespace std::literals::chrono_literals;
  	auto start = std::chrono::high_resolution_clock::now();
  	std::this_thread::sleep_for(1s);
  	auto end = std::chrono::high_resolution_clock::now();
  	std::chrono::duration<float> time = end - start;
  	std::cout << time.count() << "s" << std::endl;
  }
  ```

### 1.7.2 多线程提升性能 

使用std::async 包含在#include<furture>（c++11新特性）中:

```cpp
#include<future>
#include<iostream>
#include<fstream>
#include <string>
#include"Timer.h"
enum class Color {
	red = 0,
	green,
	blue
};
static std::mutex s_TxtMutex;
void LoadTxtFile(std::vector<std::string> *FileContent,const std::string filepath) {


	std::ifstream stream(filepath);
	std::string line;
	
	while (std::getline(stream, line)) {
		std::lock_guard<std::mutex> lock(s_TxtMutex);
		FileContent->push_back(line);
		cout << "thread id--------" << this_thread::get_id() << endl;
	}

}	
	
int main() {
	Color color=Color::red;
	switch (color)
	{
	case Color::red:
		std::cout << "red" << std::endl;
		break;
	case Color::green:
		std::cout << "green" << std::endl;
		break;
	default:
		break;
	}

	Timer time;
	std::ifstream stream("src/data.txt");
	std::string line;
	std::vector<std::string> txtFilePaths;
	std::vector<std::string> FileContent;
	std::future<void> mfuture;
	std::future<void> mainfuture;
	while (std::getline(stream, line))
		txtFilePaths.push_back(line);
#define Async 
#ifdef Async

	for (const auto& file : txtFilePaths)
		mfuture=std::async(std::launch::async, LoadTxtFile, &FileContent,file);
	mfuture.wait();
#else
	for (const auto& file : txtFilePaths)
		LoadTxtFile(&FileContent, file);
#endif // Async

	for (const auto& line : FileContent) {
		std::cout << "-------line------" << line<<std::endl;
	}
	//std::async(std::launch::async,);
}
```

+ std::async 必须有返回值，如果没有返回值，返回的临时对象会被析构，析构函数中需要等待线程结束，
+ 多线程工作函数中不能存在引用参数，按值移动或者复制，必须被包装，使用std :: ref或std :: cref

  ## 1.8 类型双关

  ```cpp
  int a = 5;
  double b= *(int*)&a;//a,与b两个变量地址不同
  double& b= *(int*)&a;//可以操作a首地址之后的8字节数据
  
  struct Entity{
      int x,y;
  }//如果struct是空的至少有一个字节
  Entity e={5,8};
  int* ptr=(int*)&e//取e的地址转换为int指针之后可以进行访问地址
  prt[0];//5
  prt[1];//8
  ```



  ```

union中的所有成员**共享**一片内存

​```cpp
struct Vector2{
    int x,y;
}
struct Vector4{
    union {
        struct{
            int x,y,z,w;
        };
        struct{
            Vcetor2 a,b;
        };
    };
}
//用于类型双关
  ```
类型转换

dynamic_cast用于继承层次的强制类型转换，用于验证一个对象是否是给定的类型

```cpp

static_cast<int>//静态类型转换
dynamic_cast<Derived*>//动态类型转换
    
reinterpret _cast<int*>(&a)//类型双关
    class Entity{
        virtual void PrintName(){}
    };
class Enemy:public Entity{
    
};
class Player:public Entity{
    
};

Entity* actuallyPlayer=new Player();
Entity* actuallyEnemy=new Enemy();
Player* p0 = dynamic_cast<Player*>(actuallyPlayer);
if(p0){
    dosomething;
}
Player* p1 = dynamic_cast<Player*>(actuallyEnemy);
if(p1){
    dosomething;
}
   
 
```



## 1.9 虚析构函数

```cpp
class Base{
    Base(){cout<<"Base Constructor"<<endl;}
    ~Base(){cout<<"Base Destructor"<<endl;}
}
class Derived{
    Derived(){cout<<"Derived Constructor"<<endl;}
    ~Derived(){cout<<"Derived Destructor"<<endl;}
}

int main(){
    Base *poly=new Derived();
    delete poly;//可能会造成内存泄漏
    /*
    Base Constructor
    Derived Constructor
    Base Destructor
    
    */
}
//如果把Base的析构函数标记为虚析构函数就能通知编译器，嘿，我是虚的我还有儿子的资源需要释放
```

# 2 新特性

## 2.1 结构化绑定

```cpp
#include<iostream>
#include<tuple>
using namespace std;
tuple<string,int> CreatePerson() {
	return { "zyb",22 };

}
int main() {
	//past
	tuple<string, int> person = CreatePerson();
	string& name = get<0>(person);
	int& age = get<1>(person);
	//past
	string name2;
	int age2;
	tie(name2, age2) = CreatePerson();
	//systemBinding
	auto[namenew, agenew] = CreatePerson();
}
```

## 2.2 Optional

### 2.2.1 处理Optional数据 optional

处理可能存在或不存在的数据

```cpp
#inlcude<iostream>
#include<optional>
#inlcude<fstream>
optional<string> ReadFileAsString(const string& filepath) {
	ifstream stream(filepath);
	if (stream) {
		string result;
		cout << result;
		stream.close();
		return result;
	}

	return {};
}

int main(){
    optional<string> data = ReadFileAsString("data.txt");
    data.value_or("Not present");
    if(data){
        string& filedata=*data;
    }
}
```



### 2.2.2 单一变量存放多类型数据 variant

variant更加安全，但是会消耗更多内存空间

```cpp
#include<variant>
using namespace std;
variant<string,int>data;
data='zyb';
data=2;
//获取数据
get<string>(data)//返回zyb
//根据data具体类型 获取类型 sting 0, int 1
data.index();
//get_if
get_if<string>(&data)//返回指针类型
```



### 2.2.3 存储任意类型数据 any

基本不需要用any

```cpp
std::any data = std::make_any<int>(2);
	int& d = std::any_cast<int&>(data);
	std::cout << d;
	return 0;
```

## 2.3 移动语义

### 2.3.1 左值和右值

左值是某种存储支持的变量，右值是临时值

```cpp
#include<iostream>
using namespace std;
void PrintName(const string& name) {

	cout << "[lvalue]" << name << endl;

}
void PrintName(string&& name) {

	cout << "[rvalue]" << name << endl;
}
int main() {
	int i = 10;		
	
	string firstname = "yb";
	string secondname = "z";
	string name = firstname + secondname;

	PrintName(firstname);
	PrintName(name);
	PrintName(firstname + secondname);
}
```



+ 左值引用必须来自左值
+ 非Const引用必须是左值
+ const 引用接受右值

 ### 2.3.2 

## 2.4 C++中的单例模式

 两种方式实现

```cpp

class Singleton
{
public:
	Singleton(const Singleton&) = delete;
	static Singleton& GetInstance() {
		return s_instance;
	}

private:
	Singleton(){}

	static Singleton s_instance;

};
Singleton Singleton::s_instance;
```

```cpp
class Random
{
public:
	Random(const Random&) = delete;
	static Random& GetInstance() {
		static Random Instance;
		return Instance;
	}
	static float Float() {
		return GetInstance().IFloat();
	}

private:
	Random(){}
	float IFloat() {
		return m_randomseed;
	}
	float m_randomseed = 0.5f;

};
```

### 2.5 跟踪内存分配

Valgrind 可以用来查看内存分配，内存管理软件。

```cpp

```







