#auto #decltype

При выводе типов для `auto` используется тот же механизм, что и для вывода типов в шаблонных функциях:
```C++
// будет выведен тот же тип, что и тут
// template <typename T>
// void f(T x) {}
auto  x = 5;
// будет выведен тот же тип, что и тут
// template <typename T>
// void f(T& x) {}
auto& y = x;
// будет выведен тот же тип, что и тут
// template <typename T>
// void f(const T& x) {}
const auto& z = y;
// будет выведен тот же тип, что и тут
// template <typename T>
// void f(T&& x) {}
auto&& t = x; // t - lvalue-ссылка на 'x' из-за reference-collapsing
			  // (auto считается int&)
auto&& t = std::move(x); // t - rvalue-ссылка, справа у нас rvalue-expression
						 // (auto считается int)
```

Можно сделать `auto` на возвращаемое значение функции:
```C++
template <typename T>
auto foo(T x) {
	return ++x;
}

auto x = foo(1);
```

Можно даже сделать вот так:
```C++
// `auto` в одном случае будет `int`, а в другом `uint`
// `if constexpr` вычисляется на этапе компиляции раньше чем `auto`
template <typename T>
auto foo() {
	if constexpr (std::is_save_v<T, int>)
		return 0;
	else
		return 1u;
}
```

Также `auto` можно использовать в `trailing return type`, чтобы улучшить читаемость кода в случае, если возвращаемый тип очень длинный:
```C++
// trailing return type
template <typename T>
auto move(T&& value) -> std::remove_reference_t<T>&&
{
	return static_cast<std::remove_reference_t<T>&&>(value);
}
```

Начиная с C++20 можно писать `auto` в принимаемых типах функций.
```C++
void f(auto x) {}
// все равно, что и
template <typename T>
void f(T x) {}
```

38 минут