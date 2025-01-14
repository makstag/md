## std istream_iterator
#istream_iterator

```C++
#include <iostream>
#include <iterator>
#include <algorithm>

struct nullopt_t {};
nullopt_t nullopt;

// наивная реализация istream_iterator
template <typename T>
class istream_iterator {
	std::isteam* in_ = nullptr;
	T value_;
public:
	// чтобы компилировалось
	using iterator_category = std::input_iterator_tag;
	using pointer = T*;
	using value_type = T;
	using reference = T&;
	using difference type = int;

	istream_iterator(std::istream& in): in_(&in) {
		in >> value_;
	}
	istream_iterator() {}

	istream_iterator& operator++() {
		*in_ >> value_;
		if (!(*in_ >> value_)) {
			*this = istream_iterator();
		}
		return *this;
	}

	T& operator*() {
		return value_;
	}
}
```

### std ostream_iterator

[возможная реализация](https://cplusplus.com/reference/iterator/ostream_iterator/)
```C++
template <class T, class charT=char, class traits=char_traits<charT> >
  class ostream_iterator :
    public iterator<output_iterator_tag, void, void, void, void>
{
  basic_ostream<charT,traits>* out_stream;
  const charT* delim;

public:
  typedef charT char_type;
  typedef traits traits_type;
  typedef basic_ostream<charT,traits> ostream_type;
  ostream_iterator(ostream_type& s) : out_stream(&s), delim(0) {}
  ostream_iterator(ostream_type& s, const charT* delimiter)
    : out_stream(&s), delim(delimiter) { }
  ostream_iterator(const ostream_iterator<T,charT,traits>& x)
    : out_stream(x.out_stream), delim(x.delim) {}
  ~ostream_iterator() {}

  ostream_iterator<T,charT,traits>& operator= (const T& value) {
    *out_stream << value;
    if (delim!=0) *out_stream << delim;
    return *this;
  }

  ostream_iterator<T,charT,traits>& operator*() { return *this; }
  ostream_iterator<T,charT,traits>& operator++() { return *this; }
  ostream_iterator<T,charT,traits>& operator++(int) { return *this; }
};
```
