
#todo find right place
```C++
#include <iostream>

// пример из 42 лекции
template <typename T>
class Optional {
public:
	Optional() = default;

	Optional(const Optional&) {
		std::cout << "A" << std::endl;
	}

	template<typename U>
	Optional(Optional<U>&&) {
		std::cout << "B" << std::endl;
	}

	template<typename U>
	Optional(U&&) {
		std::cout << "C" << std::endl;
	}
};

int main() {
	Optional<int> a;
	Optional<int> b(a);
}
```

***

#todo find firgt place
#exchange

Возможная реализация:
```C++
template<class T, class U = T>
constexpr
T exchange(T& obj, U&& new_value)
	noexcept(
		std::is_nothrow_move_constuctibla<T>::value &&
		std::is_nothrow_assignable<T&, U>::value
	)
{
	T old_value = std::move(obj);
	obj = std::forward<U>(new_value);
	return old_value;
}
```

***

