#exeptions #исключения

[Metanit exeptions](https://metanit.com/cpp/tutorial/6.2.php)

>Можно писать catch (...) , чтобы поймать ошибку любого типа

Конструкция try - catch очень "дорогая" в использовании и в некоторых кодстайлах из-за этого запрещена.

Все исключения в языке C++ описываются типом exception, который определен в заголовочном файле **exception**. Ловить через конструкцию try catch можно только то, что было брощено оператором throw. Деление на 0, доступ out of range и т.д являются ошибками, но не бросают throw.

Глобально, стандартные исплючения разделяются на два больших подвида: logic_error и runtime_error. Идея следующая: logic_error - ошибка в которой виноват программист, а runtime_error - ошибка в которой он не виноват, но что-то пошло не так.

Стандартные операторы, которые бросают исключения:
- new
- dynamic_cast
- throw
- typeid

```C++
#include <iostream>
#include <cstdlib>

int divide(int a, int b) {
	if (b == 0) {
		throw std::logic_error("Divide by zero!");
	}
	return a/b;
}

int main() {
	try {
		divide(1, 0);
	}
	catch (std::logic_error& err) {
		std::cout << err.what();
	}
}
```

Низкоуровневые причины из-за которых программы падают
**1 Segmentation fault:**
```mermaid
graph TD;
	node4{{array out of bounds}}-->node1([Segmentation fault]);
	node5{{nullptr dereference}}-->node1([Segmentation fault]);
	node6{{stack overflow}}-->node1([Segmentation fault]);
```
**2 Floating Point Exeption:**
```mermaid
graph TD;
	node2{{devide by zero}}-->node1([Floating Point Exeption]);
```
**3 Aborted:**
```mermaid
graph TD;
	node2{{std::abort}}-->node1([Aborted]);
	node3{{std::terminated}}-->node1([Aborted]);
	node4{{uncaught exeption - бросаются throw}}-->node3([std::terminated]);
	node5{{pure virtual function call}}-->node3([std::terminated]);
	node6{{false assersion}}-->node2{{std::abort}};
```

#### Exeptions handling

Можно передать **throw** без аргументов - в таком случае исключение будет передаваться дальше, чтобы его поймало catch на более высоком уровне вложенности. Повторного копирования в таком случае не будет.

Если исключение выброшено, но не поймано вплоть до main - будет UB. Гарантируется, что мы упадем с вызовом terminate, но нет гарантий вызова деструкторов и т.д.

>В статической памяти заранее резервируется место под std::bed_alloc. В случае ниже будет копирование локальной переменной типа A в динамическую память.

```C++
#include <iostream>

struct A {
    int i = 46;
	A()  { std::cout << "A\n"; }
	A(const A&) { std::cout << "copy\n"; }
	~A() { std::cout << "~A\n"; }
};

void f(int x) {
	A a;
    std::cout << &a;
	if (x == 0) {
		throw a;
	}
}

int main() {
	// A
	// 0x7ffed8bf874c
	// copy
	// ~A
	// caught! 46 0x1f9cf40
	// A
	try {
		f(0);
	} catch (A& a) {
		std::cout << "caught! " << a.i << " " << &a << '\n';
	}
}
```


https://www.youtube.com/watch?v=4Erj9hR6UsM 30:00
