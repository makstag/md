#base_iterator #iterator #const_iterator #reverse_iterator #const_reverse_iterator

```C++
#include <iostream>

template <typename T>
class vector {
	T* arr_;
	size_t sz_;
	size_t cap_;

private:

	template <bool IsConst>
	class base_iterator {
	public:
		using pointer_type = std::conditional_t<IsConst, const T*, T*>;
		using reference_type = std::conditional_t<IsConst, const T&, T&>;
		using value_type = T;
	private:
		pointer_type ptr;
		base_iterator(T* ptr): ptr(ptr) {}
		friend class vector<T>;
	public:
		base_iterator(const base_iterator&) = default;
		base_iterator& operator=(const base_iterator&) = default;

		sreference_type operator*() const { return *ptr; }

		pointer_type operator->() const {	return ptr; }

		base_iterator& operator++() {
			++ptr;
			return *this;
		}

		base_iterator operator++(int) {
			base_iterator copy = *this;
			++ptr;
			return copy;
		}

		// неявное приведение обычного итератора в константный
		operator base_iterator<true>() const {
			return {ptr};
		}
	};

public:
	using iterator = base_iterator<false>;
	using const_iterator = base_iterator<true>;
	using reverse_iterator = std::reverse_iterator<iterator>;
	using const_reverse_iterator = std::reverse_iterator<const_iterator>;

	iterator begin() {
		return {arr_};
	}

	iterator end() {
		return {arr_ + sz_};
	}

	const_iterator begin() const {
		return {arr_};
	}

	const_iterator end() const {
		return {arr_ + sz_};
	}

	const_iterator cbegin() const {
		return {arr_};
	}

	const_iterator cend() const {
		return {arr_ + sz_};
	}

	/*
	void reserve(size_t newcap) {
		/**
	}

	void push_back(const T& value) {
		/**
	}
	*/
}

int main() {}
```
