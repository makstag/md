## RTTI

Для полиморфных типов (у которого есть хоть одна виртуальная функция) поддерживается RunTime Type Information. 

Оператор typeid определяет динамический тип.

```C++
#include <iostream>

struct Base {
    virtual void f() {
        std::cout << "Base\n";
    }
};

struct Derived: public Base {
    void f() override {
        std::cout << "Derived\n";
    }
};

int main() {
    Derived d;
    Base& b = d; // у b статический тип Base, но динамический тип - Derived
    
    b.f(); // "Derived"
    std::cout << typeid(b).name() << std::endl; // "7Derived"
}
```

## Vtable

#todo сделать наглядные графики с vtable

2) https://shaharmike.com/cpp/
3) [habr vtable и отладка gdb](https://habr.com/ru/companies/otus/articles/479802/)
4) [C++ Vtable Example](https://itanium-cxx-abi.github.io/cxx-abi/cxx-vtable-ex.html)

```C++
#include <iostream>

struct A{};   // sizeof(A) - 1

struct B{     // sizeof(B) - 8, есть указатель на Vtable
	virtual void f(){}
	int b;
};

struct Derived : public Base{ // sizeof(Derived) - 16
	void f() override {}
	int d;
};

int main(){}
```
#### Проблемы с vtable
Первая проблема в том, что аргуметы по умолчанию подставляются в compile-time.

```C++
#include <iostream>

struct Base {
	virtual void f(int x = 1) const {
		std::cout << "Base " << x << '\n';
	}
};

struct Derived: public Base {
	void  f(int x = 2) const override {
		std::cout << "Derived " << x << '\n';
	}
};

int main() {
	const Base& b = Derived(); // "Derived 1"
	b.f(); 
}
```

Вторая проблема в том, что пока мы в конструкторе Base, v_pointer стоит на таблицу, как если бы это был Base. После того как Base создан и мы начинаем создавать Derived, значение v_pointer'a поменяется. Та же ситуация будет и для деструктора.

```C++
#include <iostream>

struct Base {
	Base() {
		f(0);
	}
	virtual void f(int x = 1) const {
		std::cout << "Base " << x << '\n';
	}
};

struct Derived: public Base {
	Derived() {
		f(1);
	}
	void f(int x = 2) const override {
		std::cout << "Derived " << x << '\n';
	}
};

int main() {
	const Base& b = Derived();  // "Base 0"
								// "Derived 1"
	b.f(2); // Derived 2
}
```

Третья проблема в том, что виртуальные функции у классов должны иметь определение, иначе компилятор не сможет создать vtable.

```C++
#include <iostream>

struct Base1 {
	void f();
};

struct Base2 {
	virtual void f();  // объявлена, но не определена '{}'
};

int main() {
	Base1 b1; // OK
//	Base2 b2; // undefined reference to `vtable for Base2'
			  // компилятор не смог создать vtable для Base
}
```