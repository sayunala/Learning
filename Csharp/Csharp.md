# Csharp
### 函数
**引用传递**
```Csharp
returnVar functionName(ref int a,ref int b);
```
### 结构体
```Csharp
Struct StructName{
  public int member1;
  public int member2;
}
```
### 类
```Csharp
<访问修饰符>class ClassName:parentClass{

  public int member1;
  private int member2;
  int member3;//默认私有
  public <return type> func1(ref int a,ref int b);//函数


}
```
**结构体和类的区别**：  
+ 结构体存储在栈分配空间，类存储在堆上分配空间
+ 结构体可以不用new进行instance，类要用new进行instance
+ 结构体不支持继承

#### 继承
```Csharp
<访问修饰符> class <基类>
{

}
class <派生类> : <基类>
{

}
//例子
namespace testSpace{
  class BaseClass{
    public BaseClass()
    {

    };//构造函数

  }
  class class1:BaseClass{
    public class1():BaseClass()//父类的初始化
    {

    }
  }
}

```
#### 多态
+ 运算符重载，函数重载（静态多态性）
+ 抽象类,虚函数,接口

```Csharp
abstract class BaseClass{

  abstract public void func1();//
};

class ChildClass1:BaseClass{

  public void func1(){
    do something;
  };
}

/*                  接口                          */
interface IParentInterface{
  public void Method1(ref int a,ref int b);

}

class ImplementClass:IParentInterface{
  public void Method1(ref int a,ref int b)
  {
    do somethis...;

  }
  public static void main(string []arg)
  {
    IParentInterface instance=new ImplementClass();
    instance.Method1();

  }
}
```
**接口和抽象类的区别**
+ 如果两个对象联系紧密使用抽象类，如果两个对象联系并不是非常紧密使用接口。如鸟和飞机都能飞，把飞的动作封装成接口，飞机和鸟从飞（接口）中取得功能。铁门木门都是门（抽象类）。**抽象类定义了你是什么，接口指定了你能做什么**



+ 虚函数
```Csharp
public class BaseClass{

  public int a = 0;
  public int b = 0;

  public virtual void Method1();

}

public class ChildClass1:BaseClass{

  public void Method1()
  {
    do something;
  }

  public void Method2(BaseClass arg)
  {
    arg.Method1();
  }
  public static void main(string[] arg)
  {
    BaseClass obj = new ChildClass1();
    obj.Method1();
    obj.Method1(obj);


  }
}
```
