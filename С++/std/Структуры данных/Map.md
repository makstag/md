#map #not_complited

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
template<typename Key, typename Value, typename Comparator = std::less<Key>>
class map {
	BaseNode* fakeNode;
	Node* begin;
	size_t sz;
	Comparator comp

	struct BaseNode {
		Node* left;
		Node* right;
		Node* parent; 	
	}

	struct Node: BaseNode {
		pair<const Key, Value> kv;
		bool red;
	}; 
};
```

Узел хранит пару _**const**_ Key, Value, что важно помнить, иначе можно получить ошибки. Например в следующем случае будет каждый раз происходить копирование узла и программа будет работать меденно:
```C++
// const ссылка сделана не на тот тип, которым она является
// из-за этого приходится делать лишнее копирование
// и дальше будем работать с ссылкой на временную копию, а не
// реальным объектом
for (const pair<int std::string>& p : m){
}

// ОК, лишнего копирования нет
for (const pair<const int std::string>& p : m){
}

// ОК, лишнего копирования нет
for (const auto& p : m){
}
```

***

https://cplusplus.com/reference/algorithm/lower_bound/?kw=lower_bound

std::lower_bound(Key) - возвращает итератор на наименьший элемент больше или равный вашего
std::upper_bound(Key) - возвращает итаратор на наименьший элемент строго больше вашего

***

