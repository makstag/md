#auto 
[auto, decltype(auto) cppreference](https://en.cppreference.com/w/cpp/language/auto)

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

---
## Decltype
#decltype
[decltype cppreference](https://en.cppreference.com/w/cpp/language/decltype)
[auto, decltype(auto) cppreference](https://en.cppreference.com/w/cpp/language/auto)

> [!INFO]
> `decltype()` вычисляется в `compile-time`
> `decltype()` не вычисляет выражение под собой, а просто смотрит тип под ним

`decltype` разным образом работает с `entity` и `expression`.  `decltype(auto)` говорит компилятору вывести тип самостоятельно, но как если бы мы применяли `decltype` к этому выражению, а не просто как `auto` (в точности дает тип, который был после `return` не отбрасывая амперсанды). Следующие примеры кода (3), (4) выполняют одно и то же:
```C++
// (1) всегда вызвращает по значению
auto getElement(Container& cont, size_t index)
{
    return cont[index];
}

// (2) всегда вызвращает по ссылке
auto& getElement(Container& cont, size_t index)
{
    return cont[index];
}

// (3) иногда возвращает ссылку, а иногда значение в
//     зависимости от того что написано после return
template <typename Container>
auto getElement(Container& cont, size_t index)
    -> decltype(cont[index])
{
    return cont[index];
}

// (4) C++14, иногда возвращает ссылку, а иногда значение в
//     зависимости от того что написано после return
template <typename Container>
decltype(auto) getElement(Container& cont, size_t index) {
    return cont[index];
}

// (5) C++14 всегда будет возвращаться ссылка из-за () !!!!
template <typename Container>
decltype(auto) getElement(Container& cont, size_t index) {
    return (cont[index]);
}

// (6) Error - формально, cont и index еще не объявлены
// template <typename Container>
// decltype(cont[index]) getElement(Container& cont, size_t index)
// {
//     return cont[index];
// }
```
##### Как проверить какой тип вывел decltype

```C++
#include <iostream>
#include <vector>

template <typename T>
struct Debug {
	Debug() = delete;
};

int main() {
	int x = 0;
	int& y = x;
	const decltype(y) z = y;
  
	decltype(x) t1 = std::move(x);
	decltype(x)&& t2 = std::move(x);
	decltype(auto) t3 = std::move(x);
  
	Debug<decltype(t1)>();
	Debug<decltype(t2)>();
	Debug<decltype(t3)>();
}
```
