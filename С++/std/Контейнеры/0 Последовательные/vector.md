#vector #not_complited
[cppreference vector](https://en.cppreference.com/w/cpp/container/vector)
[metanit vector](https://metanit.com/cpp/tutorial/7.2.php)

> Vector является динамическим массивом и его данные хранятся в динамической памяти, поэтому sizeof(vector) - 24 байта на x64. На стеке хранится только следующее:
    • _M_start - начало выделенной памяти;
    • _M_finish - последний вставленный элемент;
    • _M_end_of_storage - конец выделенной памяти.
> Иногда (в stl из visual studio) на стеке также будет храниться аллокатор.

> [!WARNING]
> В vector при добавления элемента в конец указатели на элементы, а также ссылки и итераторы инвалидируются

***
## Наивная реализация вектора

```C++
#include <iostream>

template <typename T, typename Alloc = std::allocator<T>>
class vector {
	T* arr_;
	size_t sz_;
	size_t cap_;
	Alloc alloc_;

public:
	void reserve(size_t newcap) {
		if (newcap <= cap) {
			return;
		}

		// T* newarr = reinterpret_cast<T*>(new char[newcap * sizeof(T)]);
		T* newarr = alloc_.allocate(newcap);

		size_t index = 0;
		try {
			for (; index < sz; ++index) {
				// placement new
				// не запрашивает память, только вызывает конструктор
				// new(newarr + index) T(arr_[index]);
				alloc_.construct(newarr+index, arr_[index]);
			}
		} catch (...) {
			for (size_t newindex = 0; newindex < index; ++newindex) {
				// (newarr + newindex)->~T();
				alloc_.destroy(newarr+oldindex);
			}
			// delete[] reinterpret_cast<char*>(newarr);
			alloc_.deallocate(newarr, newcap);
			throw;
		}

		for (size_t index = 0; index < sz_; ++index) {
			// (arr_ + index)->~T();
			alloc_.destroy(arr_+index);
		}
		// delete[] reinterpret_cast<char*>(arr_);
		alloc_.deallocate(arr_, cap_);

		arr_ = newarr;
		cap_ = newcap;
	}

	void push_back(const T& value) {
		if (sz_ == cap_) {
			reserve(cap_ > 0 ? cap_ * 2 : 1);
		}
	}
}

template <>
class vector<bool> {
	char* arr_;
	size_t sz_;
	size_t cap_;
	
	struct BitReference {
		char* cell;
		uint8_t index;

		BitReference(char* cell2, uint8_t index2)
			: cell(cell2), index(index2) {}

		BitReference operator=(bool b) {
			if (b) {
				*cell |= (1 << index);
			} else {
				*cell &= ~(1 << index);
			}
			return *this;
		}
		
		operator bool() const {
			return *cell & (1 << index);
		}
	};
public:

	BitReference operator[](size_t index) {
		return BitReference(arr + (index >> 3), index % 8);
	}
}

int main() {
	std::vector<bool> v(10);
	v[5] = true;
}
```

***
## Тестирую placement new
#placementnew
```C++
#include <iostream>
#include <vector> 

struct Test {
    Test() {std::cout << "T";}
    explicit Test(const Test& t) {std::cout << " copy constructor ";}
    ~Test() {std::cout << "~T";}
    void operator=(const Test& t) { std::cout << "="; }
};
 
int main()
{
	// TT
    Test* t = new Test[2];

    Test* newarr = reinterpret_cast<Test*>(malloc(sizeof(Test)*2));
    // copy constructor
    new(newarr)     Test(t[0]);
    // copy constructor
    new(newarr + 1) Test(t[1]);

	// ~T~T
    delete[] t;
}
```