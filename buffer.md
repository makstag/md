#todo очистить буффер

5.8 Виртуальные функции с виртуальным наследованием

#todo перенести с листка

5.9 Some problems related to dynamuc dispatching

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

Вторая проблема в том, что пока мы в конструкторе Base, v_pointer стоит на таблицу, как если бы это был Base. После того как Base создан и мы начинаем создавать Derived, значение v_pointer'a поменяется.

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

 C++ 13. Vtables, шаблоны.  49:30
https://www.youtube.com/watch?v=uCdJeq57TeM&list=PL4_hYwCyhAvazfCDGyS0wx_hvBmnAAf4h&index=13&t=1216s