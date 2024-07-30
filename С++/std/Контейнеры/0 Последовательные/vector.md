#vector #not_complited
[cppreference vector](https://en.cppreference.com/w/cpp/container/vector)
[metanit vector](https://metanit.com/cpp/tutorial/7.2.php)
[deduction guides for std::vector](https://en.cppreference.com/w/cpp/container/vector/deduction_guides)

> Vector является динамическим массивом и его данные хранятся в динамической памяти, поэтому sizeof(vector) - 24 байта на x64. На стеке хранится только следующее:
    • _M_start - начало выделенной памяти;
    • _M_finish - последний вставленный элемент;
    • _M_end_of_storage - конец выделенной памяти.
	• alloc_ - аллокатор (может не занимать места на стеке из-за empty-base optimization)
	
> [!WARNING]
> В vector при добавления элемента в конец указатели на элементы, а также ссылки и итераторы инвалидируются

***
## Наивная реализация вектора

```C++
#include <iostream>

template <typename T, typename Alloc = std::allocator<T>>
class vector { // : private Alloc {
			   // возможно унаследование ради  empty-base optimization
	T* arr_;
	size_t sz_;
	size_t cap_;
	Alloc alloc_;

	using AllocTraits = std::allocator_traints<Alloc>;

public:
	void reserve(size_t newcap) {
		if (newcap <= cap) {
			return;
		}

		// T* newarr = reinterpret_cast<T*>(new char[newcap * sizeof(T)]);
		T* newarr = AllocTraits::allocate(alloc_, newcap);

		size_t index = 0;
		try {
			for (; index < sz; ++index) {
				// placement new
				// не запрашивает память, только вызывает конструктор
				// new(newarr + index) T(arr_[index]);

				// тут важно использовать std::move_if_noexcept
				// если move-конструктор не noexcect - мы не будем
				// мувать, а будем копировать
				AllocTraits::construct(alloc_, newarr+index,
						std::move_if_noexcept(arr_[index]));
			}
		} catch (...) {
			for (size_t newindex = 0; newindex < index; ++newindex) {
				// (newarr + newindex)->~T();
				AllocTraits::destroy(alloc_, newarr+oldindex);
			}
			// delete[] reinterpret_cast<char*>(newarr);
			AllocTraits::deallocate(alloc_, newarr, newcap);
			throw;
		}

		for (size_t index = 0; index < sz_; ++index) {
			// (arr_ + index)->~T();
			AllocTraits::destroy(alloc_, arr_+index);
		}
		// delete[] reinterpret_cast<char*>(arr_);
		AllocTraits::deallocate(alloc_, arr_, cap_);

		arr_ = newarr;
		cap_ = newcap;
	}

	// push_back можно выразить через emplace_back
	void push_back(const T& value) {
		emplace_back(value);
	}

	// push_back можно выразить через emplace_back
	void push_back(T&& value) {
		emplace_back(std::move(value));
	}

	void emplace_back(auto&&... args) {
	    if (sz_ == cap_) {
	        reserve(cap_ > 0 ? cap_ * 2 : 1);
	    }
	    AllocTraits::construct(alloc_, arr_ + sz_,
	            std::forward<decltype(args)>(args)...);
	    ++sz_;
	}

//	template <typename... Args>
//	void emplace_back(Args&&... args) {
//		if (sz_ == cap_) {
//			reserve(cap_ > 0 ? cap_ * 2 : 1);
//		}
//		AllocTraits::construct(alloc_, arr_ + sz_,
//				std::forward<Args>(args)...);
//		++sz_;
//	}

	// мы должны старые объекты удалить старым аллокатором, а новые объекты
	// выделить новым аллокатором, при этом мы все еще должны старые объекты
	// удалить позже, чем выделим новые
	vector& operator=(const vector& other) {

		Alloc newalloc =
			AllocTraints::propagate_on_ontainer_copy_assignmet::value
			& other.alloc_ : alloc_;

		T* newarr = AllocTraits::allocate(newalloc, other.cap_);
		size_t i = 0;
		try {
			for (; i < other.sz_; ++i) {
				AllocTraits::construct(newalloc, newarr + i, other[i]);
			}
		} catch (...) {
			for (size_t j = 0; j < i; ++j) {
				AllocaTraits::destroy(newalloc, newarr + j);
			}
			AllocTraits::deallocate(newalloc, newarr, other.cap_);
			throw;
		}

		for (size_t i = 0; i < sz_; ++i) {
			AllocTraints::destroy(alloc_, arr_ + i);
		}
		AllocTraints:deallocate(alloc_, arr_, cap_);

		alloc_ = newalloc; // nothrow
		arr_ = newarr;
		sz_ = other.sz_;
		cap_ = other.cap_;
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
