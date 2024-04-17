#### Dependent names
#typename #template

>Tho phase translation - когда комилятор компилирует шаблонный код, ему приходится делать это в два прохода. Сначала до подставления шаблона T, потом после подставления шаблона T

По умолчанию поля шаблонных классов считаются компилятором за переменные. Для того, чтобы явно указать, что это тип переменной, а не сама переменная нужно дописать ключевое слово typename. (Случай 1)

В (Случай 2) недостаточно ключевого слово typename, нужно также явно указать, что B - шаблонный тип или компилятор будет парсить это как expression. Но, вроде G++11 справляется и без "template".

В (Случай 3) также придется явно указывать ключевое слово template, чтобы выражение не парсилось как expression.

В (Случай 4) если мы хотим обратиться к полю шаблонного родителя - нужно явно писать this (или явно разрешать поле видимости "Base::"), иначе компилятор не воспримет это как поле родителя.

```C++
#include <iostream>
#include <array>

template <typename T>
struct S {
	using A = int;

	template <int N>
	using B = std::array<int, N>;
};

template <>
struct S<double> {
	static const int A = 5;
};

template <typename T>
void f() {
	//  (1)
//	S<T>::A* x;          // считается expression по умолчанию
	typename S<T>::A* x; // now it's declaration

	//  (2)
	typename S<T>::template B < 10 > x;
}

int main() {
	f<int>();
}
```

```C++
#include <iostream>
#include <array>

template <typename T>
struct S {
	template <int N>
	void foo(int) {}
};

template <typename T>
void bar(int x, int y) {
	//  (3)
	S<T> s;
	s.template foo<5>(x+y);
}

int main() {
	f<int>();
}
```

```C++
#include <iostream>
#include <array>

template <typename T>
struct Base {
	int x = 0;
};

template <>
struct Base<double> {
};

template<typename T>
struct Derived : Base<T> {
	void f() {
//      ++x           // ERROR
		++this->x;    // OK
		++Base<T>::x; // OK
	}
};

int main() {
	f<int>();
}
```
#### Variadic templates C++11

Начиная с C++11 можно создвать темплейты от произвольного количества шаблонных параметров. В примере, указанном ниже, функция print выводит первый шаблонный параметр и рекурсивно вызывает саму себя. Также нужна база print() без шаблонных параметров, которая ничего не выводит. 

> существует оператор std::sizeof...(tail), который возвращает в compile-time число, равное размеру пакета

```C++
#include <iostream>

void print() {}

template <typename Head, typename... Tail>
void print(const Head& head, const Tail&... tail) {
	std::cout << head << ' ';
	print(tail);
}

int main() {}
```

Пример проверки на однородность:
```C++
#include <iostream>

void print() {}

template <typename First, typename Second typename... Types>
struct is_homogeneous {
	static constexpr bool value = std::is_same_v<First, Second>
		&& is_homogeneous<Second, Types...>::value;
}

template <typename First, typename Second>
struct is_homogeneous<First, Second> {
	static constexpr bool value = std::is_same_v<First, Second>;
}


int main() {}
```

#### Metafunctions and type traits

Для удобства работы с метафункциями в стандартной библиотеке есть #type_traits 
Начиная с C++14 в него добавили using'ги типа conditional_t, а начиная с С++17 добавили шаблонные переменные типа is_same_v.  Все это облегчает работу и позволяет писать меньше кода. Ниже приведены примеры реализации некоторых шаблонов из стандартной библиотеки. 
##### std::is_same

> if constexpr появился в C++17
> std::is_same появилась в C++11

```C++
#include <iostream>
#include <type_traits>

// Пример реализации std::is_same
template <typename T, typename U>
struct my_is_same {
	static constexpr bool value = false;
};

template <typename T>
struct my_is_same<T,T> {
	static constexpr bool value = true;
};

//c++17 добавили еще шаблонную переменную:
template <typename T, typename U>
const bool is_same_v = is_same<T, U>::value;
////

template <typename T, typename U>
void f(T x, U y) {
	// ...
	if constexpr (is_same<T, U>::value) {
		
	}
	// ...
}

int main() {
	f<int, std::string>(5, "abc");
}
```
##### std::remove_reference

```C++
#include <iostream>
#include <type_traits>

// Пример реализации std::remove_reference
template <typename T>
struct my_remove_reference {
	using type = T;
};

template <typename T>
struct my_remove_reference<T&> {
	using type = T;
};
////


template <typename T>
void f() {
	typename remove_reference<T>::type x;
}

int main() {}
```
##### std::conditional

```C++
// мета-тернарный оператор
// Пример реализации std::conditional
template <bool B, typename T, typename F>
struct conditional {
    using type = F;
};

template <bool B, typename T, typename F>
struct conditional<true, T, F> {
    using type = T;
};

// шаблонный using для conditional начиная с C++14
template <bool B, typename T, typename F>
using conditional_t = typename conditional<B, T, F>::type;
```
