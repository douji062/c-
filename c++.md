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

**拷贝构造函数：**一种构造函数，当复制第二个字符串时，它会被调用

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
	String second = string;//调用了拷贝构造函数

	second[2] = 'a';

	PrintString(string);
	PrintString(second);
	
	std::cin.get();
}
```

**const引用传递可以减少拷贝**

---

## 箭头运算符

* 最常见的使用方式：

```c++
#include <iostream>

class Entity
{
public:
	int x;
public:
	void Print()const
	{
		std::cout << "Hello" << std::endl;
	}
}; 


int main()
{
	Entity e;
	e.Print();
	
	Entity* ptr = &e;
	//(*ptr).Print();
	ptr->Print();//手动逆向引用的快捷方式
	ptr->x = 2;

	std::cin.get();
}
```

* 你也可以重载并在自定义类中使用它：

```c++
#include <iostream>

class Entity
{
public:
	int x;
public:
	void Print()const
	{
		std::cout << "Hello" << std::endl;
	}
}; 
class ScopePtr
{
private:
	Entity* m_Obj;
public:
	ScopePtr(Entity* entity)
		:m_Obj(entity) 
	{

	}
	~ScopePtr()
	{
		delete m_Obj;
	}

	//Entity* GetObject() { return m_Obj; }

	Entity* operator->() 
	{
		return m_Obj;
	}

	const Entity* operator->() const
	{
		return m_Obj;
	}
};

int main()
{
	const ScopePtr entity = new Entity();
	//entity.GetObject()->Print();//这种方式就需要调用方法再加箭头，所以我们选择重载操作符让它更加clear
	entity->Print();

	std::cin.get();
}
```

可以在自己的类型中定义自己的构造函数，并实现自动化

* 使用箭头操作符获取成员变量的偏移量

```c++
#include <iostream>
struct Vector3
{
	float x, y, z;//在内存布局中把x的偏移量看成0
};
int main()
{
	int offset =(int) & ((Vector3*)0)->z;
    //先把0或者nullptr化成Vector3形式的指针再指向需要查看偏移量的变量
    //取地址并转换成int类型
	std::cout << offset << std::endl;
	std::cin.get();
}
```

---

## C++动态数组(std::vector)

如果创建一个更大的数组，他会在内存中创建一个比第一个更大的新数组，把所有东西复制到这里然后删除旧的那个

动态数组是内存连续的数组

在创建时< >尽量使用对象而不是指针（对象是连续分配而指针是随机分配）

```c++
#include <iostream>
#include <vector>
struct Vertex
{
	float x, y, z;//在内存布局中把x的偏移量看成0
};

std::ostream& operator<<(std::ostream& stream, const Vertex& vertex)
{
	stream << vertex.x << "," << vertex.y << "," << vertex.z;
	return stream;
}

void Function(const std::vector<Vertex>& vertices) {//写函数时候保证是传引用的方式以防止复制

}
int main()
{
	std::vector<Vertex> vertices;//这是c++类模板所以不需要指定类类型，可以指定原始类型
	vertices.push_back({ 1,2,3 });
	vertices.push_back({ 4,5,6 });
	Function(vertices);

	/*for (int i = 0; i < vertices.size(); i++) {
		std::cout << vertices[i] << std::endl;
	}*/

	for(Vertex& v:vertices)//如果不加&实际上是copy每个vertex到for循环中
		std::cout << v << std::endl;

	vertices.erase(vertices.begin()+1);//去掉第二个//需要一个迭代器作为index

	for (int i = 0; i < vertices.size(); i++) {
		std::cout << vertices[i] << std::endl;
	}

	std::cin.get();
}
```

* std::vector的使用优化

复制并重新分配是拖慢进程的主要因素。所以如何避免复制是优化程序的关键。

优化前：

```c++
#include <iostream>
#include <vector>
struct Vertex
{
	float x, y, z;//在内存布局中把x的偏移量看成0

	Vertex(float x, float y, float z)
		:x(x),y(y),z(z)
	{
	}
	Vertex(const Vertex& vertex)
		:x(vertex.x), y(vertex.y), z(vertex.z)
	{
		std::cout << "Copied" << std::endl;
	}

};

std::ostream& operator<<(std::ostream& stream, const Vertex& vertex)
{
	stream << vertex.x << "," << vertex.y << "," << vertex.z;
	return stream;
}

int main()
{
	std::vector<Vertex> vertices;//这是c++类模板所以不需要指定类类型，可以指定原始类型
	vertices.push_back(Vertex(1, 2, 3));//在main内部构造vertex对象，然后放入vector vertices中产生了一次复制
	vertices.push_back(Vertex(4, 5, 6));
	vertices.push_back(Vertex(7, 8, 9));//调整了三次大小1+2+3，所以优化策略是一开始分配足够三个对象的内存

	//copy了六次
	std::cin.get();
}
```

优化后：

优化思路，取消在main中构造vertex对象。并且一次性分配足够三个对象存储的内存防止多次修改再进行复制。

```c++
#include <iostream>
#include <vector>
struct Vertex
{
	float x, y, z;//在内存布局中把x的偏移量看成0

	Vertex(float x, float y, float z)
		:x(x),y(y),z(z)
	{
	}
	Vertex(const Vertex& vertex)
		:x(vertex.x), y(vertex.y), z(vertex.z)
	{
		std::cout << "Copied" << std::endl;
	}

};

std::ostream& operator<<(std::ostream& stream, const Vertex& vertex)
{
	stream << vertex.x << "," << vertex.y << "," << vertex.z;
	return stream;
}

int main()
{
	std::vector<Vertex> vertices;//这是c++类模板所以不需要指定类类型，可以指定原始类型
	vertices.reserve(3);//reserve确保我们有足够的内存
	vertices.emplace_back(1, 2, 3);//不是传递已经构造的vertex对象而是传递构造的参数列表，这样就可以直接在vector内存中构造
	vertices.emplace_back(4, 5, 6);
	vertices.emplace_back(7, 8, 9);

	std::cin.get();
}
```



---

## C++中使用库(静态链接)

在实际解决方案的实际项目文件中保留使用的库版本

link with binary（GLFW库)

下载库是32还是64位取决于目标程序

C++创建一个动态链接库，编译后会生成两个可用的文件一个是lib文件一个是dll文件，那么这个lib文件是干嘛的呢？

在使用动态库的时候，往往提供两个文件：一个引入库和一个DLL。引入库包含被DLL导出的函数和变量的符号名，DLL包含实际的函数和数据。在编译链接可执行文件时，只需要链接引入库，DLL中的函数代码和数据并不复制到可执行文件中，在运行的时候，再去加载DLL，访问DLL中导出的函数。

1.  Load-time Dynamic Linking 载入时动态链接
   这种用法的前提是在编译之前已经明确知道要调用DLL中的哪几个函数，编译时在目标文件中只保留必要的链接信息，而不含DLL函数的代码；当程序执行时，利用链接信息加载DLL函数代码并在内存中将其链接入调用程序的执行空间中，其主要目的是便于代码共享。
2.  Run-time Dynamic Linking 运行时动态链接
   这种方式是指在编译之前并不知道将会调用哪些DLL函数，完全是在运行过程中根据需要决定应调用哪个函数，并用LoadLibrary和GetProcAddress动态获得DLL函数的入口地址。

头文件提供声明告诉我们哪些函数是可用的，库文件为我们提供定义，让我们可以链接到那些函数并在c++中正确调用

添加外部依赖项

```c++
#include <iostream>
#include "GLFW/glfw3.h"


int main()
{
	int a = glfwInit();
	std::cout << a << std::endl;
	std::cin.get();
}
```

---

## 动态库

动态链接是链接发生在运行时，静态链接是链接在编译时发生的。静态链接允许更多的优化发生。

可执行文件实际上知道动态链接库的存在并需要，但是动态链接库仍然是一个单独的文件模块并在运行时加载。

1.需要现场加载动态链接库，已经知道里面有什么函数并使用

2.想任意加载这个动态库，在不知道里面有什么的情况下进行

---

## 创建与使用库

静态链接时，所有的东西都被放入了这个exe文件而无需任何外部文件依赖。

---

## C++如何处理多返回值

* 可以利用输入参数返回多类型返回值

* 也可以使用数组返回。std::array或者std::vector,主要区别是array在栈上创建而vector在堆上创建因此返回array更快，当然这些方法仅对单一类型的返回值有效。

* tuple（元组）

  ```c++
  #include "Engine.h"
  #include <iostream>
  #include <functional>
  static std::tuple<std::string, std::string> ParseShader(const std::string & filepath)
  {
  	std::string vs;
  	std::string fs;
  	return std::make_pair(vs,fs);
  }
  
  
  int main() {
  	std::tuple<std::string, std::string> sources = ParseShader("/Bacic.shader");
  	std::string vs = std::get<0>(sources);//从元组里获取参数 std::get<1>(sources)
  	std::cin.get();
  }
  ```

* pair 

和tuple的唯一区别是获取参数除了可以用std::get，还可以用sources.first

* struct

  

  ```c++
  #include "Engine.h"
  #include <iostream>
  #include <functional>
  
  struct SharderProgramSource
  {
  	std::string VertexSourse;
  	std::string FragmentSourse;
  };
  static SharderProgramSource ParseShader(const std::string & filepath)
  {
  	std::string vs;
  	std::string fs;
  	return { vs,fs };//返回一个结构体
  }
  
  
  int main() {
  	auto sources = ParseShader("/Bacic.shader");
  	std::string vs = sources.VertexSourse;//从结构体获取元素//这种方式可以直接定位到元素的名字而不是用1、2这种容易混淆
  	std::cin.get();
  }
  ```
---

## C++的模板template

类似Java的泛型但是比他们厉害得多，泛型受限于类型系统而模板强大的多。

模板允许你定义一个可以根据你的用途进行编译的模板，你可以让编译器基于一套规则为你写代码

  ```c++
#include <iostream>

template<typename T>
void Print(T value)
{
	std::cout << value << std::endl;
}

int main() {
	Print<int>(5);
	Print("Cherno");//编译器可以自动隐式推导出类型并填入T
	std::cin.get();
}
  ```

**当模板没有被调用时，函数其实并没有被创建。**

编译器做的是推导类型并拷贝模板，填入到Typename的位置成为真正的函数。

* 模板也可以生成类

  ```c++
  #include <iostream>
  #include <string>
  
  template<typename T,int size>//可以同时传入多个模板参数
  class Array
  {
  private:
  	T m_Array[size];
  public:
  	int GetSize()const
  	{
  		return size;
  	}
  };
  int main() {
  	Array<int,5> array;
  	std::cout << array.GetSize() << std::endl;
  	std::cin.get();
  }
  ```

  当你有一个可以包含不同类型的统一缓冲区时，一定程度上模板是非常有用的。

---

## 堆与栈内存的比较

栈和堆是ram中实际存在的两个区域。

fundamentally it do the same thing,the difference is how it allocate our memory

栈分配：![](C:\Users\p'c\Desktop\27502_20161202113008\QQ图片20240726202817.png)

我们所做的只是移动栈指针然后返回栈指针的地址。（中间的cc是在调试模式下的安全守卫防止溢出）

```c++
#include <iostream>
#include <string>
struct Vector3
{
	float x, y, z;
	
	Vector3()
		:x(10), y(11), z(12)
	{
	}
};

int main() {
	int value = 5;
	int array[5];
	array[0] = 1;
	array[1] = 2;
	array[2] = 3;
	array[3] = 4;
	array[4] = 5;
	Vector3 vector;

	int* hvalue = new int;
	*hvalue = 5;
	int* harray = new int[5];
	harray[0] = 1;
	harray[1] = 2;
	harray[2] = 3;
	harray[3] = 4;
	harray[4] = 5;
	Vector3* hvector = new Vector3();

	delete hvalue;
	delete[] harray;
	delete hvector;
	std::cin.get();
}
```

堆分配：

实际上调用了一个叫malloc的函数，当启动时，会得到一定数量的物理ram，然后程序会维护一个叫空闲列表的东西，它会跟踪哪块内存块是空闲的，在需要动态内存时候使用动态堆内存，返回指向那块内存的指针然后记录。

如果程序需要的内存超过空闲链表的内存，程序会问操作系统寻求更多内存，而这会造成麻烦的后果，所以潜在成本巨大。

**allocate memory in heap is a hole thing!!while allocate in stack is like a cpu instruction.**

---

## 宏

预处理器来宏化代码

```c++
#include <iostream>
#include <string>

#define WAIT std::cin.get()
int main() 
{
	WAIT;//不要这样使用宏！因为如果在其他代码中看见这个还需要查找定义，令人困惑
}
```

```c++
#include <iostream>
#include <string>

#define LOG(x) std::cout<<x<<std::endl
int main() 
{

	LOG("Hello");
	std::cin.get();

}
```

应用场景：比如可以在relese模式下去掉日志记录而在debug中保留

![QQ图片20240726213857](C:\Users\p'c\Desktop\27502_20161202113008\QQ图片20240726213857.png)

```c++
#include <iostream>
#include <string>

#if PR_DEBUG == 1
#define LOG(x) std::cout<<x<<std::endl
#elif defined(PR_RELEASE)
#define LOG(x)
#endif 
// DEBUG模式下才会打印log信息


int main() 
{

	LOG("Hello");
	std::cin.get();

}
```

`#if 0`和`#endif`可以用来禁用这一段宏定义

可以在每一行末尾加上\（转义字符）来写多行定义的宏，注意\之后不要含有空格

---

## auto

c++自动推导数据类型，在这里c++似乎变成了弱类型语言，同时也代表当我们对变量进行修改时，也许不一定需要修改其类型，如果使用了auto关键字的话。

一方面如果改变api，客户端的auto不需要做出任何改变，但是不知道api发生了改变也有可能破坏特定类型的代码。

```c++
#include <iostream>
#include <string>

const char* GetName()
{
	return "Cherno";
}

int main() 
{
	std::string name = GetName();//auto name = GetName();则不行，因为其实包含了隐式转换,不转换成string类型则.size()无法调用
	int a = name.size();
	std::cin.get();

}
```

然而有时候使用auto是一个好主意：

```c++
#include <iostream>
#include <string>
#include <vector>
#include <unordered_map>

const char* GetName()
{
	return "Cherno";
}
class Device{};
class DeviceManager
{
private:
	std::unordered_map<std::string, std::vector<Device*>> m_Devices;
public:
	const auto& GetDevices() const
	{
		return m_Devices;
	}
};
int main() 
{
	std::vector<std::string> strings;
	strings.push_back("apple");
	strings.push_back("orange");

	//for (std::vector<std::string>::iterator it = strings.begin(); it != strings.end(); it++)
	for (auto it = strings.begin(); it != strings.end(); it++)//迭代器//当类型非常长时用auto可以增强可读性
	{
		std::cout << *it << std::endl;
	}
	DeviceManager dm;
	const auto& devices = dm.GetDevices();
	std::cin.get();

}
```

**auto在c++11可以使用后置返回类型，c++14中可用于函数**

---

## std::array

