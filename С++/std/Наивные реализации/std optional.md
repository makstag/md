#optional

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
	T& value_or(T& other) {
		return initialized ? reinterpter_cast<T&>(*value) : over;
	}
};

struct nullopt_t {};
nullopt_t nullopt;
```
