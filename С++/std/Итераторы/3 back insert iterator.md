#back_insert_iterator #back_insert_iterator 

```C++
#include <iostream>
#include <algorithm>
#include <vector>

bool even(int x) {
	return x % 2 == 0;
}

// Наивная реализация
template <typename Container>
class back_insert_iterator {
	Container& container;
public:
	back_insert_iterator(Container& container): container(container) {}
	// оператор разыменования *
	back_insert_iterator& operator=(const typename Container::value_type& value){
		container.push_back(value);
		return *this;
	}

	back_insert_iterator& operator++(){
		return *this;
	}

	back_insert_iterator operator++(int){
		return *this;
	}

	back_insert_iterator& operator*(){
		return *this;
	}
};

// Синтаксический сахар для back_insert_iterator
template <typename Container>
back_insert_iterator<Container> back_inserter(Container& container) {
	return {container};
}

int main() {
	int a[10] = {1, 2, 3, 4, 5};
	std::vector<int> v;
	
//	std::copy_if(a, a + 10, v.begin(), &even); // UB
	std::copy_if(a, a + 10,
	             std::back_insert_iterator<std::vector<int>>(v), &even); // OK
	std::copy_if(a, a + 10, std::back_inserter(v), &even); // also OK
}
```