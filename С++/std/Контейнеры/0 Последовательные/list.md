#list #not_complited 

 insert
erase
push_back
pop_back

```C++
template <typename T>
class list {
	struct BaseNode {
		BaseNode* next;
		BaseNode* prev;
	}
	struct Node : BaseNode {
		T value;
	};
	BaseNode fakeNode;
	size_t sz;

public:
	list(): fakeNode {&fakeNode, &fakeNode}, sz(0) {}

};
```
