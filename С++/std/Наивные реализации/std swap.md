#swap

```C++
template <typename T>
void swap(T& x, T& y) {
	T tmp = std::move(x);
	x = std::move(y);
	y = std::move(tmp);
}
```