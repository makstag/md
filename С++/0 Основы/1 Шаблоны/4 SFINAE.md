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
[std::declval cppreference](https://en.cppreference.com/w/cpp/utility/declval)

>[!INFO]
>`std::declval` используется только на этапе компиляции, например в выражениях  `std::decltype`, `std::sizeof`, `std::alignof`. Вызов `std::declval` в рантайме является ошибкой.

Возможная реализация `std::declval`:
```C++
// add_rvalue_reference в отличие от && работает также и для void
template<typename T>
typename std::add_rvalue_reference<T>::type declval() noexcept
{
    static_assert(false, "declval not allowed in an evaluated context");
}
```

Реализация проверки наличия методов в классе:
```C++
// Имеется проблема в том, чтобы получить объекты типа T и Args
// но при этом у них может не быть конструктора по умолчанию.
// Решить эту проблему можно при помощи std::declval()
namespace detail {
	// comma trick
	template <typename T, typename... Args>
	std::true_type test( decltype(std::declval<T>()
				.construct(std::declval<Args>()...), nullptr));

	template <typename...>
	std::false_type test(...);
}

template <typename T, typename... Args>
struct has_method_construct: decltype(detail::test<T, Args...>(nullptr)) {};

template <typename T, typename... Args>
constexpr bool has_method_construct_v = has_method_construct<T, Args...>::value;
```

Реализация проверки наличия конструктора копирования:
```C++
namespace detail {
	// comma trick
	template <typename T>
	std::true_type test(decltype(T(std::declval<T&>()), nullptr));

	template <typename...>
	std::false_type test(...);
}

template <typename T>
struct is_copy_constructible: decltype(detail::test<T>(nullptr)) {};

template <typename T, typename... Args>
constexpr bool is_copy_constructible_v = is_copy_constructible<T>::value;
```

Реализация проверки наличия `noexcept` конструктора копирования:
```C++
namespace detail {
	template <typename T>
	std::true_type test(
			std::enable_if_t<noexcept(T(std::declval<T>())), decltype(nullptr)>
	);

	template <typename...>
	std::false_type test(...);
}

// -----

struct Good {
	Good(Good&&) noexcept {}
};

struct Bad {
	Bad(Bad&&) {}
};

struct VeryBad {
	VeryBad(VeryBad&&) = delete;
};

int main() {
	static_assert(is_nothrow_move_constructible_v<Good>);
	static_assert(!is_nothrow_move_constructible_v<Bad>);
	static_assert(!is_nothrow_move_constructible_v<VeryBad>);
}
```

[std::is_base_of cppreference](https://en.cppreference.com/w/cpp/types/is_base_of)
[std::conjunction cppreference](https://en.cppreference.com/w/cpp/types/conjunction)
Реализация проверки является ли один тип наследником другого:
```C++
#include <iostream>

namespace details
{
    template<typename B>
    std::true_type test_ptr_conv(const volatile B*);
    template<typename>
    std::false_type test_ptr_conv(const volatile void*);
 
    template<typename B, typename D>
    auto test_is_base_of(int) -> decltype(test_ptr_conv<B>(static_cast<D*>(nullptr)));
    template<typename, typename>
    auto test_is_base_of(...) -> std::true_type; // private or ambiguous base
}
 
template<typename Base, typename Derived>
struct is_base_of :
    std::integral_constant<
        bool,
        std::is_class<Base>::value &&
        std::is_class<Derived>::value &&
        decltype(details::test_is_base_of<Base, Derived>(0))::value
    > {};

template <typename B, typename D>
const bool is_base_of_v = is_base_of<B, D>::value;

// -----

struct Base {};
struct Derived: Base {};
struct Derived_private: private Base {};

int main() {
	static_assert(is_base_of_v<Base, Derived>);
	static_assert(!is_base_of_v<Derived, Base>);
	static_assert(is_base_of_v<Base, Derived_private>);
}
```

Наивная реализация `std::common_type` (В C++11 такая реализация и была):
```C++
template <typename... Types>
struct common_type;

template <typename T>
struct common_type<T>: std::type_identity<T> {};

template <typename T, typename U>
struct common_type<T, U>
	: std::type_identity<
		decltype(true ? std::declval<T>() : std::declval<U>())
    > {};

template <typename T, typename.. Types>
struct common_type<T, Types...>
	: common_type<T, common_type<Types...>::type> {};
```

Лекция 50  33:00