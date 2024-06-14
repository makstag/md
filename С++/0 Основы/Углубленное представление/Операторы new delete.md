#operator #new #delete

> [!WARNING]
> Оператор new состоит из двух частей: первая занимается выделением памяти, а вторая вызовом конструкторов на выделенной памяти и перегрузить можно только первую
## Переопределение операторов new и delete

Наивные реализация
```C++
#include <iostream>
#include <vector>

void* operator new(size_t n) {
	std::cout << n << " bytes allocated\n";
	return malloc(n);
}

void operator delete(void* ptr) {
	std::cout << "deallocated";
	free(ptr);
}

int main() {
	std::vector<int> v;
	for (itn i = 0; i<50; ++i) {
		v.push_back(i);
	}
}
```

***
## Тестирую placement new
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