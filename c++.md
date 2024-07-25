## 枚举ENUM

```c++
#include <iostream>

class Log
{

public:
	enum Level
	{
		LogLevelError, LogLevelWarning, LogLevelInfo
	};
private:
	Level m_LogLevel = LogLevelInfo;

public:
	void SetLevel(Level level)
	{
		m_LogLevel = level;
	}
	void Info(const char* message)
	{
		if(m_LogLevel >=LogLevelInfo)
			std::cout << "[INFO]:" << message << std::endl;
	}
	void Warn(const char* message)
	{
		if (m_LogLevel >= LogLevelWarning)
			std::cout <<"[WANRNING]:"<< message << std::endl;
	}
	void Error(const char* message)
	{
		if (m_LogLevel >= LogLevelError)
			std::cout << "[ERROR]:" << message << std::endl;
	}
	


};


int main()
{
	Log log;
	log.SetLevel(Log::LogLevelError);
	log.Warn("hello!");
	log.Info("hello!");
	log.Error("hello!");
	std::cin.get();
};


```

---
## 纯虚函数virtual

只包含未实现的方法作为模板，在实现了所有纯虚函数之后才可以实例化。


```c++
#include <iostream>

class Printable//抽象类 
{
public:
	virtual std::string GetClassName() = 0;//虚函数/接口，只有实现了之后才可实例化
};
class Entity: public Printable
{
public:
	virtual std::string GetName() { return "Entity"; };
	std::string GetClassName() override { return "Entity"; }
};

class Player :public Entity
{
private:
	std::string m_Name;
public:
	Player(const std::string& name)
		:m_Name(name){}//构造函数 需要传入名初始化列表
	std::string GetName() override{ return m_Name; }

	std::string GetClassName() override { return "Player"; }//override重载关键字，增加可读性

};
void Print(Printable* obj)
{
	std::cout << obj->GetClassName() << std::endl;
}
int main() {
	Entity* e = new Entity();
	Player* p = new Player("jack");
	Print(e);
	Print(p);

	std::cin.get();
}

```


---

## 数组Array

`int example[5]`其中的`example`实际上是一个指针；

如果访问超出index的数据则会报错Memory access violation,所以要设置边界检查；

数组的内存空间连续，提供一些连续的内存块然后通过索引实现；

`example[2]=5;`实际上是写入指针开始2*4个字节的偏移量；(example 是int类型的指针) ==`*(ptr+2) = 5;`

```c++
int main()
{
	int example[5];//在栈上创建数组，生存期到此程序结束的括号
	for (int i = 0; i < 5; i++)
		example[i] = 2;
	int* another = new int[5];
	for (int i = 0; i < 5; i++)//在堆上创建数组，生存期直到程序将其销毁
		another[i] = 2;
	delete[] another;
	std::cin.get();
}
```

比如，如果有一个函数需要返回一个数组，必须用new的方式分配，除非传入一个数组的地址参数；

不过可能的话，还是尽量在栈上创建数组，以防止内存跳跃影响性能。（返回的是数组的地址，还要再寻址一次）

c++11版本的数组 `std::array`:可提供边界检查，记录数组大小

**永远不应该在数组内存中访问数组的大小**

```c++
int a[5];
int count = sizeof(a)/sizeof(int);//5
```

如果要检查数组大小，必须把它变成栈分配的数组。但是最好不要这样做，因为一旦变成指针就完蛋。。

```c++
static const int exampleSize = 5;
int example[exampleSize];//原始数组

std::array<int,5> another;//或者c++11的数组创建方法，记得引用<array>的头文件
```

**必须要加static因为编译器不知道这个变量是否会先于数组被释放**

***

## 字符串

`string` 是一个固定分配的代码块所以不能拓展其大小。（新版本c++好像必须用`const`修饰，不然需要用（char*）进行隐式转换）

' '默认是char，“ ”默认是char*

如果手动创建字符串则必须加入'\0'或者0代表该字符串结束，否则会自动加入字符守卫`ascii`码



在c++中有std::string，它是一个char数组和一些对char数组操作的函数

以下是一些使用实例：

```c++
#include <string>
#include <iostream>

void printString(const std::string& string)//传引用防止字符串复制，占用内存
{
	std::cout << string << std::endl;//这边只需要打印而不需要修改
}
int main()
{
	std::string name = "Cherno";
	name += ",hello!";
	name.size();
	bool contains = name.find("no")!=std::string::npos;//检查是否含有，find会找到no开始的位置
	std::cout << name << std::endl;//操作符允许发送字符串到流是string的重载版本
	printString(name);
	std::cin.get();
}
```



---

## 字符串字面量（即在“ ”内的部分）

```c++
#include <iostream>
#include <string>


int main()
{
	using namespace std::string_literals;
	std::string name0 = std::string("Cherno") + ",hello";
	std::string name = "Cherno"s + ",hello";

	const char* example = R"(Line1
Line2
Line3
Line4)";
	const char* ex = "Line1\n"
		"Line2"
		"Line3";

	const char* name1 = "Cherno";
	const wchar_t* name2 = L"Cherno";
	const char16_t* name3 = u"Cherno";
	const char32_t* name4 = U"Cherno";

	std::cin.get();
}
```

**字符串字面量永远保存在内存的只读区域内。**

---

## CONST

相当于Java的final,声明保证其不会改变

ptr可以改变指向内存的内容，也可以改变指向的对象地址

`const +数据类型*+ ptr /数据类型+ const* +ptr`只是不能改变指针指向的内容

`数据类型*+const +ptr`则不能改变指针指向其他对象

- 类和方法中使用const

  ```c++
  #include <iostream>
  #include <string>
  
  class Entity
  {
  private:
  	int m_X, m_Y;
  	mutable int var;
  public:
  	int GetX() const//只在类中有效，意味着这个方法不会修改实际的类，只读不写
  	{
  		var = 2;//mutable可变，即使是在const中
  		return m_X;
  	}
  	int GetX()
  	{
  		return m_X;
  	}
  	void SetX(int x)
  	{
  		m_X = x;
  	}
  };
  void PrintEntity(const Entity& e)//传引用保证不再复制一个Entity
  {
  	std::cout << e.GetX() << std::endl;//这边e不允许修改，所以在调用方法时候方法也必须包含const
  }
  int main()
  {
  	std::cin.get();
  }
  ```

  



---

## mutable(可变)

运用在const类中或者lambda表达式(const中的应用可参考上一个程序)

Lambda中的实例：

```c++
#include <iostream>

int main()
{
	int x = 8;
	auto f = [=]() mutable//如果不使用mutable，则需要搞一个另外的变量把x赋值给它，然后修改这个变量，很麻烦
		{
			x++;
			std::cout << x << std::endl;
		};
	f();

	std::cin.get();
}
```



---

## 初始化列表

```c++
class Entity
{
private:
	std::string m_Name;
	int m_Score;
public:
	Entity()
		:m_Name("unKnown"),m_Score(0)//如果在这里去掉初始化列表并在下面括号内赋值，实际上会被构造两次
	{
	}
	Entity(const std::string& name)
		:m_Name(name) 
	{
	}

};
```

需要注意的是，在成员变量的区域，<u>并不意味着不会运行代码并创建对象</u>

```c++
#include <iostream>
class Example
{
public:
	Example()
	{
		std::cout << "创建一个Entity" << std::endl;
	}
	Example(int x)
	{
		std::cout << "创建一个Entity和" <<x<< std::endl;
	}
};
class Entity
{
private:
	std::string m_Name;
	int m_Score;
	Example m_Example;
public:
	Entity()
		:m_Name("unKnown"), m_Score(0), m_Example(8)//如果在这里去掉初始化列表并在下面括号内赋值，实际上会被构造两次,因为默认有一次
	{
		/*m_Example = Example(8);*/
	}
	Entity(const std::string& name)
		:m_Name(name) 
	{
	}
	//const std::string& GetName() const { return m_Name; }

};
int main()
{
	Entity e;
	std::cin.get();
}
```

**因此必须使用初始化成员列表，否则会浪费性能**

---

## 三元操作符

```c++
#include <iostream>
static int s_Level = 9;
static int s_Speed = 2;
int main()
{

	s_Speed = s_Level > 5 ? 10 : 5;//如果真则赋值10否则5
	std::string rank = s_Level > 10 ? "Master" : "Biginner";
	//少了一个中间字符串赋值原因是返回值优化
	s_Speed = s_Level > 5 ? s_Level > 10 ? 15 : 10 : 5;//(<5 -> 5,5<x<10 -> 10,>10 -> 15)
	std::cout << s_Speed << std::endl;
	std::cin.get();

}
```

---

## 创建并初始化c++对象

* 在栈stack上声明对象：生存期为声明存在的作用域

* 在堆heap上声明对象：会一直存在除非delete将其销毁。所以当对象需要跨作用域存在或者有时候栈的内存空间过小而不够分配时，选择在堆上初始化对象。

*scope可以不仅仅是函数，比如一个花括号什么的，都可能是。*

```c++
#include <iostream>
using String = std::string;

class Entity
{
private:
	String m_Name;
public:
	Entity() : m_Name("UnKnown"){}
	Entity(const String& name):m_Name(name){}

	const String& GetName() const { return m_Name; }
};
int main()
{
	Entity* e;
	{
		Entity* entity1 = new Entity("Chernoheap");//在堆上创建对象，但是需要手动内存管理
		Entity entity("Cherno");//有默认构造方法可以初始化//如果可以则这样创建对象，除非你想把这个函数放在函数生存期之外
		e = &entity;
		std::cout << entity.GetName() << std::endl;
		std::cout << entity1->GetName() << std::endl;//entity1是指针所以->

		delete entity1;
	}
	//在这里时entity已经被释放，e指针指向的地址内容已经无

	
	std::cin.get();

}
```

---

## New关键字

new just a operator like plus 

可以override the operator

<u>point 其实是无类型的，需要类型只是需要方式来操纵指针，但是它的实质只是一串数字（地址）</u>

```c++
#include <iostream>
using String = std::string;

class Entity
{
private:
	String m_Name;
public:
	Entity() : m_Name("UnKnown"){}
	Entity(const String& name):m_Name(name){}

	const String& GetName() const { return m_Name; }
};
int main()
{
	Entity* e = new Entity();//两行代码的唯一区别是它调用了Entity构造函数
	Entity* e1 = (Entity*)malloc(sizeof(Entity));//分配内存并返回一个指向那个内存的指针，没有构造函数，但是不要这样在cpp里分配内存
    int* b = new int[50];
    delete[] b;//new[],在delete时候也必须包含[]
	
	delete e;
	delete e1;//使用了c的free函数，释放malloc分配的内存
	std::cin.get();

}
```

---

## 隐式转换与explicit

c++允许编译器对其做一次隐式转换

explicit禁止隐式转换，比如用在数学库中防止数字变成向量，低级封装时候防止做意外转换，导致性能问题或者bug

```c++
#include <iostream>
using String = std::string;

class Entity
{
private:
	String m_Name;
	int m_Age;
public:
	explicit Entity(int age) : m_Name("UnKnown"),m_Age(age){}//防止隐式转换
	Entity(const String& name):m_Name(name),m_Age(-1){}

	//const String& GetName() const { return m_Name; }
};
void PrintEntity(const Entity& entity) 
{

}
int main()
{
	//PrintEntity(22);//最好不要这样做因为可读性比较差
	PrintEntity(String("Cherno"));//只允许一次隐式转换 char->string->entity 所以需要强转一次
	Entity b = (Entity)22;//cast强制转换//或者Entity(22)这样构造
	
	std::cin.get();

}
```

---

## 运算符及其重载

 operator just a function

c++的独特之处——你可以用操作符来代替函数，让你的代码更加clear

```c++
#include <iostream>

struct Vector2
{
	float x, y;
	Vector2(float x,float y)
		: x(x),y(y){}

	Vector2 Add(const Vector2& other) const//可以写两个(函数或者重载)给调用你的api的人选择
	{
		return Vector2(x + other.x, y + other.y);
		//return operator+(other);//完全可用只是有点奇怪
	}

	Vector2 operator+ (const Vector2& other) const
	{
		return Add(other);//操作符重载和上面的功能完全相同
	}

	Vector2 operator* (const Vector2& other) const
	{
		return Vector2(x * other.x, y * other.y);
	}
    
    bool operator==(const Vector2& other) const
	{
		return x == other.x && y == other.y;
	}

	bool operator!=(const Vector2& other) const
	{
		return !(*this == other);
	}


};


std::ostream& operator<< (std::ostream& str,const Vector2& vector)
{
	str << vector.x << "," << vector.y;
	return str;
}//类似Java的ToString函数重写


int main()
{
	Vector2 position(4.0f, 4.0f);
	Vector2 speed(0.5f, 1.5f);
	Vector2 powerup(1.1f, 1.1f);

	Vector2 result = position + speed * powerup;

	std::cout << result << std::endl;//不能直接这样操作因为<<右边没有对应的类型匹配，需要对<<进行重载
	std::cin.get();

}
```

---

## this关键字

**this是一个指向当前对象的指针，该方法属于这个对象实例**

```c++
#include <iostream>
void PrintEntity(const Entity& e);
class Entity
{
public:
	int x, y;

	Entity(int x, int y) 
	{
		this->x = x;
		this->y = y;

		Entity& e = *this;
		PrintEntity(*this);
	}

	int GetX() const
	{
		const Entity& e = *this;//this是一个指向当前实例的指针
	}
};
void PrintEntity(const Entity& e)
{

}
int main()
{
	
	std::cin.get();

}
```

**要避免使用delete this，因为这代表从成员函数中释放内存，你再也不能访问任何类的成员数据**

---

## 对象在栈作用域生存期

栈是一种数据结构

对象在栈作用域生存期一旦出作用域就结束了，所以**不要创建一个基于栈的对象，尝试返回它的指针**，因为一旦出作用域，对象将会被销毁

下面这也是智能指针的原理：

```c++
#include <iostream>

class Entity
{
public:
	Entity()
	{
		std::cout << "创建" << std::endl;
	};
	~Entity()
	{
		std::cout << "销毁" << std::endl;
	};

};

class ScopedPtr//作用域指针，出作用域自动销毁
{
private:
	Entity* m_Ptr;
public:
	ScopedPtr(Entity* ptr)
		:m_Ptr(ptr)
	{
	}
	~ScopedPtr()
	{
		delete m_Ptr;
	}
};
int main()
{
	{
		ScopedPtr e = new Entity();//ScopedPtr对象是在栈上创建的，意思是e被自动删除时会自动销毁m_Ptr
	}
	std::cin.get();
}
```

可以使用的场景：计时器，互斥锁

---

## 智能指针

当你用智能指针在heap上创建对象的时候，无需调用delete，甚至不需要new

* unique_ptr

  必须是唯一的，如果复制一个unique_ptr它们会指向同一个内存块，其中一个die，另一个会指向已经被释放的内存，所以它不能复制

  ```c++
  #include <iostream>
  #include <memory>
  
  
  class Entity
  {
  public:
  	Entity()
  	{
  		std::cout << "创建" << std::endl;
  	};
  	~Entity()
  	{
  		std::cout << "销毁" << std::endl;
  	};
  	void Print(){}
  };
  
  
  int main()
  {
  	{
  		std::unique_ptr<Entity> entity = std::make_unique<Entity>();//unique_ptr参数是explicit的，必须显式构造
  		//如果构造函数抛出异常，会安全一些，因为不会得到一个没有引用的空指针，造成内存泄漏
  		entity->Print();
  		//这个指针不能复制
  	}
  	std::cin.get();
  }
  ```

  

* shared_ptr

  使用引用计数。跟踪引用数目，一旦引用数目为0则被删除。

  ```c++
  #include <iostream>
  #include <memory>
  
  
  class Entity
  {
  public:
  	Entity()
  	{
  		std::cout << "创建" << std::endl;
  	};
  	~Entity()
  	{
  		std::cout << "销毁" << std::endl;
  	};
  	void Print(){}
  };
  
  
  int main()
  {
  	{
  		std::shared_ptr<Entity> e0;
  		{
  			std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();
  			//shared_ptr需要分配另一块内存称之为控制块，用来记录引用计数
  			e0 = sharedEntity;
  		}
  
  	}
  	//在这里销毁
  	std::cin.get();
  }
  ```

* weak_ptr

  不会增加引用计数。但是它也不会让底层对象保持存活。

  ```c++
  int main()
  {
  	{
  		std::weak_ptr<Entity> e0;
  		{
  			std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();
  			//shared_ptr需要分配另一块内存称之为控制块，用来记录引用计数
  			e0 = sharedEntity;
  		}
  //在这里就被销毁
  	}
  	
  	std::cin.get();
  }
  ```

  

使用智能指针可以使内存管理自动化，防止内存泄漏。**尽量使用unique_ptr因为它有一个较低的开销**

---

## 复制与拷贝构造函数

```c++
#include <iostream>

struct Vector2
{
	float x, y;
};



int main()
{
	Vector2* a = new Vector2();
	Vector2* b = a;//实际上a和b指向的是同一块内存地址
	b->x = 2;//a和b中的x都被改变

	//把一个引用赋值另一个引用时候只是改变指向，因为引用实际上只是别名
	std::cin.get();
}
```

**把一个引用赋值另一个引用时候只是改变指向，因为引用实际上只是别名**

浅拷贝：

```c++
#include <iostream>

class String
{
private:
	char* m_Buffer;
	unsigned int m_Size;
public:
	String(const char* string)
	{
		m_Size = strlen(string);
		m_Buffer = new char[m_Size+1];
		/*for (int i = 0; i < m_Size; i++) {
			m_Buffer[i] = string[i];
		}*/
		memcpy(m_Buffer, string, m_Size+1);//拷贝目标，拷贝源，长度
	}
	~String()
	{
		delete[] m_Buffer;
	}

	friend std::ostream& operator<<(std::ostream& stream, const String& string);
};


std::ostream& operator<<(std::ostream& stream, const String& string)
{
	stream << string.m_Buffer;
	return stream;
}

int main()
{
	String string = "Cherno";
	String second = string;//实际上m_Buffer的内存地址对于两个string对象是相同的

	std::cout << string << std::endl;
	std::cout << second << std::endl;
	std::cin.get();//所以析构函数被调用了两次，试图两次释放内存，所以程序崩溃
}
```

我们应该做的是分配一个新的char数组来储存复制的字符串，而不是简单地复制指针，导致两个指向相同的内存缓冲区。（浅拷贝）

所以我们希望第二个字符串应该有自己的指针，以拥有自己唯一的内存块。

**深拷贝**

拷贝构造函数：一种构造函数，当复制第二个字符串时，它会被调用

```c++
#include <iostream>

class String
{
private:
	char* m_Buffer;
	unsigned int m_Size;
public:
	String(const char* string)
	{
		m_Size = strlen(string);
		m_Buffer = new char[m_Size+1];
		/*for (int i = 0; i < m_Size; i++) {
			m_Buffer[i] = string[i];
		}*/
		memcpy(m_Buffer, string, m_Size+1);//拷贝目标，拷贝源，长度
	}

	String(const String& other)//拷贝构造函数
		:m_Size(other.m_Size)//默认的拷贝构造函数中，m_Buffer是复制的地址
	{
		std::cout << "拷贝构造" << std::endl;
		m_Buffer = new char[m_Size + 1];
		memcpy(m_Buffer, other.m_Buffer, m_Size + 1);
	}
	~String()//析构函数
	{
		delete[] m_Buffer;
	}

	char& operator[](unsigned int index) 
	{
		return m_Buffer[index];
	}

	friend std::ostream& operator<<(std::ostream& stream, const String& string);
};


std::ostream& operator<<(std::ostream& stream, const String& string)
{
	stream << string.m_Buffer;
	return stream;
}

void PrintString(const String& string)//always通过const去传递对象
{
	std::cout << string << std::endl;
}

int main()
{
	String string = "Cherno";
	String second = string;//实际上m_Buffer的内存地址对于两个string对象是相同的

	second[2] = 'a';

	PrintString(string);
	PrintString(second);
	
	std::cin.get();//所以析构函数被调用了两次，试图两次释放内存，所以程序崩溃
}
```

**const引用传递可以减少拷贝**


