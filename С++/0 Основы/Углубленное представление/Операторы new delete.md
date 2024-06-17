#operator #new #delete

[cppreference new](https://en.cppreference.com/w/cpp/memory/new/operator_new)

> [!WARNING]
> `new` состоит из двух частей: первая занимается выделением памяти, а вторая вызовом конструкторов на выделенной памяти и перегрузить можно только первую. `delete` сначала вызывает деструкторы объекта (объектов), а затем освобождает память и перегрузить можно только второе

> [!WARNING]
> Если нужно просто выделить память (не вызывая конструктор) следует использовать `operator new` вместо `new`

Помимо обычного вызова new, есть еще placement new, new не бросающий исключений или оператор new с произвольным набором параметров

## non-throwing new
```C++
#include <iostream>
#include <new>

// Output:
// 'std::bad_alloc'
// 'Allocation returned nullptr'
int main()
{
    try
    {
        while (true)
        {
            new int[100000000ul];   // throwing overload
        }
    }
    catch (const std::bad_alloc& e)
    {
        std::cout << e.what() << '\n';
    }
 
    while (true)
    {
        int* p = new(std::nothrow) int[100000000ul]; // non-throwing overload
        if (p == nullptr)
        {
            std::cout << "Allocation returned nullptr\n";
            break;
        }
    }
}
```

***
## оператор new с произвольным набором параметров

 > [!INFO]
> Если в конструкторе нетривиального объекта будет вызвано исключение, стандартом гарантируется, что будет вызван симметричный delete (кастомный) от таких же параметров и только после этого исключение полетит. Если был объявлен массив из нескольких элементов и только n-ный бросил исключение комплилятор должен также гарантировать вызов деструкторов для всех объектов, которые уже были созданы

```C++
// кастомный operator new с произвольным набором параметров
void* operator new(size_t n, int a, double d) {
	std::cout << n << " bytes allocated with custom new " << a << ' '
	          << b << '\n';
	return malloc(n);
}

// кастомный operator delete с симметричным набором параметров
void operator delete(void* ptr, int a, double d) {
	std::cout << "custom delete called " << a << ' '
	          << b << '\n';
	return free(ptr);
}

// бросающая исключение в конструкторе структура
struct S {
	S() { throw 1; }
};

int main() {
	// (1)
	// вызов new с произвольным набором параметров
	int* p = new(1, 3.14) int(5);

	// OK, вызвали симметричный operator detete, который определили сами
	// если вызвать дефолтный - может быть UB (мы не знаем конкретную реализацию)
	operator delete(p, 2, 5.25);

	// not OK, нужно вызывать симметричный operator delete
	// с таким же набором параметров
	// delete p; 

	// (2)
	try {
		S* p = new(1, 3.14) S();
	} catch (...) {
		std::cout << "caught!";
	}
}
```

***
## переопределение операторов new и delete

Наивная реализация
```C++
#include <iostream>
#include <vector>
#include <stinrg>

void* operator new(size_t n) {
	std::cout << n << " bytes allocated\n";
	return malloc(n);
}

void operator delete(void* ptr) {
	std::cout << "deallocated";
	free(ptr);
}

void* operator new[](size_t n) {
	std::cout << n << " [] bytes allocated\n";
	return malloc(n);
}

void operator delete[](void* ptr) {
	std::cout << "[] deallocated";
	free(ptr);
}

int main() {
	std::vector<int> v;
	for (itn i = 0; i<50; ++i) {
		v.push_back(i);
	}

	std::cout << sizeof(std::string) << '\n';  // 32
	// "328 [] bytes allocated" - дополнительные 8 байт хранят
	// число деструкторов, которые нужно вызвать
	std::string* ps = new std::string[10];
	delete[] ps;
}
```

***
## пример со сдвигами указателя

Пример из - "Лекция 36. Allocator-aware контейнеры. Перегрузка new и delete"
```C++
#include <new>

struct Example {
	int* x;
	Example(): x(new int) {}
	~Example() { delete x; }
};

int main() {
	// (1) OK
	Example* example = reinterpret_cast<Example*>(new char[sizeof(Example)]);
	new (example) Example(); // placement new

	// (2) not OK, operator delete[] не сдвинет на 8 указатель
	// Example* example = new Example[1];

	example[0].~Example();      // вызываем деструктор
	operator delete[](example); // освобождаем пямять

	// delete[](example) // ОК для второго случая (неявно делает сдвиг на 8)
}
```

***
## тестирую placement new
#placementnew
```C++
#include <iostream>
#include <vector> 

struct Test {
    Test() {std::cout << "T";}
    explicit Test(const Test& t) {std::cout << " copy constructor ";}
    ~Test() {std::cout << "~T";}
    void operator=(const Test& t) { std::cout << "="; }
};
 
int main()
{
	// TT
    Test* t = new Test[2];

    Test* newarr = reinterpret_cast<Test*>(malloc(sizeof(Test)*2));
    // copy constructor
    new(newarr)     Test(t[0]);
    // copy constructor
    new(newarr + 1) Test(t[1]);

	// ~T~T
    delete[] t;
}
```