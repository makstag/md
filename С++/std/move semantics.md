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

Если компилятор видит `rvalue` - он вызовет `move` операцию, а если `lvalue` копирующую, но можно принудительно заставить его вызвать move операцию для `lvalue`. Для этого используется функция `std::move` .

```C++
// версия для l-value
S( const std::string& data ): data(data) {}
// версия для r-value
S( std::string&& data ): data(std::move(data)) {}
```

Наивная реализация std::move :
```C++
template <typename T>
T&& move (T& x) {
	 return static_cast<T&&>(x);
 }
```

