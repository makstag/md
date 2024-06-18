#move

> [!TIP]
> Если не вводить для класса операторы move-копирования и move-присваивания, move семантика будет работать как обычные копирующие аналоги.

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