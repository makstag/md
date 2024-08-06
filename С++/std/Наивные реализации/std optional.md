#optional #deducingthis

[cppreference optional](https://en.cppreference.com/w/cpp/utility/optional)
[habr optional](https://habr.com/ru/articles/372103/)

наивная реализация optional:
```C++
// начиная с C++17
template <typename T>
class optional {
	char value[sizeof(T)];
	bool initialized = false;
public:
	optional(const T& newvalue): initialized(true) {
		new (reinterpret_cast<T*>(value)) T(newvalue);
	}
	optional() {}
	~optional() {
		if (initialized) {
			reinterpret_cast<T*>(value)->~T();
		}
	}
	bool has_value() const {
		return initialized;
	}
	operator bool() const {
		return initialized;
	}
	T& operator*() {
		return reinterpter_cast<T&>(*value);
	}
	T& value() & {
    if (!initialized)
        throw std::bad_optional_access();
	    return reinterpret_cast<T&>(*value);
	}
	T& value_or(T& other) {
		return initialized ? reinterpter_cast<T&>(*value) : other;
	}
};

struct nullopt_t {};
nullopt_t nullopt;
```

Для того, чтобы поддержать корректное поведение, всю константность и все виды value нам нужно писать 4 перегрузки:
```C++
T& operator*() & {
    return reinterpret_cast<T&>(*val);
}
const T& operator*() const & {
    return reinterpret_cast<const T&>(*val);
}
T&& operator*() && {
    return reinterpret_cast<T&&>(*val);
}
const T&& operator*() const && {
    return reinterpter_cast<const T&&>(*val);
}
```

Начиная с C++23 появился [Deducing this](https://habr.com/ru/articles/722668/) и теперь можно явно передавать `this` и избегать дублирования кода:
```C++
// Deducing this, since c++23
template <typename Self>
decltype(auto) value(this Self&& self) {
    if (!initialized)
        throw std::bad_optional_access();
    using DesiredType = decltype(std::forward_like<decltype(self)>(std::declval<T>()));
    return reinterpret_cast<DesiredType>(*self.value);
}

// или даже так
decltype(auto) value(this auto&& self) {
    if (!initialized)
        throw std::bad_optional_access();
    using DesiredType = decltype(std::forward_like<decltype(self)>(std::declval<T>()));
    return reinterpret_cast<DesiredType>(*self.value);
}
```

> [!INFO]
> `Deducing this` может заменить собой `CRTP`

