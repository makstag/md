#SFINAE

> [!INFO]
> Substitution Failure Is Not An Error

Если в момент подстановки T в **объявление** функции у компилятора получается некорректный тип -> это не считается ошибкой компиляции. Просто компилятор выкидывает эту версию из множества кандидатов на перегрузку и выбирает следующую, лучше всего подходящую.
```C++
#include <iostream>
#include <vector>

// будет работать только для типов T, которые
// имеют ::value_type
template <typename T>
auto f(T) -> typename T::value_type {
	std::cout << 1 << std::endl;
	return {};
}

// будет работать для всех остальных типов T
template <typename... Types>
void f(Types...) {
	std::cout << 2 << std::endl;
}

int main() {
	std::vector<int> v;

	f(v); f(0); // 1, 2
}
```

[std::enable_if cppreference](https://en.cppreference.com/w/cpp/types/enable_if)
реализация `std::enable_if`:
```C++
template <bool B, typename T = void>
struct enable_if {};

template <typename T>
struct enable_if<true, T> {
	using type = T;
};

template <bool B, typename T = void>
using enable_if_t = enable_if<B, T>::type;
```

Например, сделаем перегрузку которая бы работала только с integral-types используя `SFINAE` и `std::enable_if`:
```C++
// так перегрузить не получится, так как для компилятора
// это будет считаться redefenition (обу функции отличаются
// только аргументами по умалчанию)
template <typename T,
          typename = std::enable_if_t< std::is_integral_v<T> >>
auto f(T) {
	std::cout << "1\n";
}

template <typename T,
          typename = std::enable_if_t< !std::is_integral_v<T> >>
auto f(T) {
	std::cout << "2\n";
}
```

А вот так можно. Второй тип будет либо невалидным либо `bool`, причем неважно укажем ли мы `true` или `false` значением аргумента по умолчанию, это нужно лишь для того, чтобы не приходилось явно передавать каждый раз значение второго параметра:
```C++
// будет работать только для интегральных типов
template <typename T,
          std::enable_if_t< std::is_integral_v<T>, bool> = true>
auto f(T) {
	std::cout << "1\n";
}

// будет работать только для неинтегральных типов
template <typename T,
          std::enable_if_t< !std::is_integral_v<T>, bool> = true>
auto f(T) {
	std::cout << "2\n";
}
```

---
## Реализация метафункций используя SFINAE
#has_method_construct #declval

Проверка наличия методов в классе:
```C++
// Имеется проблема в том, чтобы получить объекты типа T и Args
// но при этом у них может не быть конструктора по умолчанию.
// Решить эту проблему можно при помощи std::declval()
namespace detail {
	template <typename T, typename... Args>
	std::true_type test( decltype(T().construct(Args()...))* );

	template <typename...>
	std::false_type test(...);
}

template <typename T, typename... Args>
struct has_method_construct: decltype(detail::test<T, Args...>(nullptr)) {};

template <typename T, typename... Args>
constexpr bool has_method_construct_v = has_method_construct<T, Args...>::value;
```

18:00