#move

> [!TIP]
> Если не вводить для класса операторы move-копирования и move-присваивания, move семантика будет работать как обычные копирующие аналоги.

> [!INFO]
> Если написать =default для move-constructor'а или move-assignment operator'а это будет просто почленный move полей (для примитивных типов это будет то же самое, что и копирование).

```C++
// move-constructor
string(string&& other)
	    : arr(othrt.arr), sz(other.zs), cap(othrt.cap) {
	othrt.arr = nullptr;
	othrt.sz = other.cap = 0;
}
// move-assignment operator
string& operator=(string&& other) {
	delete[] arr;
	arr = othrer.arr;
	other.arr = nullptr;
	sz = other.sz; other.sz = 0;
	cap = other.cap; other.cap = 0;
	return *this;
}
```

***
## std::move
#move 

Если компилятор видит `rvalue` - он вызовет `move` операцию, а если `lvalue` копирующую, но можно принудительно заставить его вызвать move операцию для `lvalue`. Для этого используется функция `std::move` .

```C++
// версия для l-value
S( const std::string& data ): data(data) {}
// версия для r-value
S( std::string&& data ): data(std::move(data)) {}
```

Наивная реализация `std::move` :
```C++
// первое приближение
// умеет принимать только lvalue, может в некоторых случаях
// возвращать T& из-за правила Reference collapsing
template <typename T>
T&& move (T& x) {
	 return static_cast<T&&>(x);
}

// как нужно
// снимаем с T все амперсанды, если они были, а потом навешиваем &&
// это позволяет избежать Reference collapsing rule на возвращаемый тип
// теперь функция move всегда будет возвращать T&& и никогда T&
template <typename T>
std::remove_reference_t<T>&& move(T&& value) {
	return static_cast<std::remove_referenve_t<T>&&>(value);
}
```

> [!INFO]
> Reference collapsing rule: если в момент вывода типа переменной в шаблоне у нас накладываются амперсанды один на другой -  они преобразуются по следующему правилу:
>   & + &  = &
>   && + & = &
>   & + && = &
>   && + && = &&

***
## std::forvard
#forvard

Чтобы иметь возможность передавать аргументы дальше в функции с сохранением вида value по которому мы их приняли, нужно использовать функцию `std::forvard`
```C++
void construct(U* ptr, Args&& ... args) {
	new (ptr) U(std::forvard<Args>(args)...);
	// '...' здесь просто разварачивает паттерн и означает
	// std::forvard<1starg>(1starg), std::forvard<2ndarg>(2ndarg), и т.д.
}
```

Наивная реализация `std::forward`:
```C++
// важно, чтобы forward не умел выводить тип шаблонного параметра неявно
template <typename T>
T&& forward(std::remove_reference_t<T>& value) noexcept {
	return static_cast<T&&>(value);
}

// перегрузка forward
// обрабатывается довольно редкий случай, когда мы вызываем std::forward
// не просто от аргументов, а от результата вызова функции от наших
// аргументов (который иногда может вернуть rvalue)
template <typename T>
T&& forward(std::remove_reference_t<T>&& value) noexcept {
	static_assert(!std::is_lvalue_reference_v<T>);
	return static_cast<T&&>(value);
}
```

Пример приема произвольного количества строк, где в зависимости от типа `value` будем или мувать или копировать их:
```C++
struct S {
	std::vector<string> v;

	template<typename... Args>
	S(Args&&... args) {
		// (1) static_assert is_convartable to string
		// https://en.cppreference.com/w/cpp/types/is_convertible
		// (2) https://en.cppreference.com/w/cpp/language/sizeof...
		v.resize(sizeof...(args));
		// (3) в зависимости от типа value происходит move или copy
		(v.push_back(std::forvard<Args>(args)), ...);
	}
};
```