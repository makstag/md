 insert
erase
push_back
pop_back

```C++
template <typename T>
class list {
	struct BaseNode {
		Node* next;
		Node* prev;
	}
	struct Node : BaseNode {
		T value;
	};
	BaseNode fakeNode;
	size_t sz;

public:
	list(): fakeNode {&fakeNode, &fakeNode}, sz(0) {};

};
```

## map

pair<iterator, bool> insrt(const pair<Key, Value>&)
size_t erase(Key)
Value& at(Key)
Value operator[] (Key)
iterator erase(iterator)
iterator find(Key)
size_t count(Key)

Все это работает за O(log n)

> от const map нельзя вызывать метод  [] - так как этот метод потенциально может поменять map


```C++
// неэффективно
if (m.const(1)) {
	m[1] = 2;
}

// эффективнее - всего один спуск по дереву
if (auto it = m.find(1), it != m.end(), ++it) {
	it->second = 2;
}
```

```C++
template<typename Key, typename Value, typename Comparator>
class map {

	struct Node {
		pair<Key, Value> kv;
		Node* left;
		Node* right;
		Node* parent;
		bool red;
	}; 
};
```

https://www.youtube.com/watch?v=6aMaWipXWYo&list=PLmSYEYYGhnBviRYhIDty-CSTDS16a3whl&index=33
1:10