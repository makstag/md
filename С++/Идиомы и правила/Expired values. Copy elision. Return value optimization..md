[cppreference copy elision](https://en.cppreference.com/w/cpp/language/copy_elision)
[habr RVO NRVO](https://habr.com/ru/companies/vk/articles/666330/)
## Copy elision

> [!IMPORTANT]
> Если объект инициализируется посредством `prvalue` выражения - промежуточный объект можно в принципе не создавать

#todo проверить это
```C++
// std::string("abc") - prvalue, промежуточный объект не будет создан
std::string s1 = std::string("abc");

// std::move(std::string("abc")) - xvalue, промежуточный объект будет создан
// после чего произойдет move, а это лишние операции
std::string s2 = std::move(std::string("abc"));

// когда нужно применять std::move
// если мы приняли объект по rvalue-ссылке и возвращаем его же
// то следует применять move. Компилятор не сможет оптимизировать
// такой код и применить RVO (переменная `a` не локальная).
// Но написав move мы сможем применить move-конструктор вместо
// copy-конструктора на возврате
S f(S&& a) {
	return std::move(a);
}
```

***
## Return value optimization

RVO - частный случай `copy elision`.

> [!IMPORTANT]
> RVO работает только для локальных переменных и не работает для аргументов функции. Даже если компилятору при возврате не удается применить RVO, но переменная локальная - он ее мувает, а не копирует на возврат

```C++
std::string f() {
	std::string s = "abc"; // NRVO
	return s;
}

// при создани строки `s` будет вызван только один конструктор
std::strin s = f(); // copy elision
```

***
## Temporary materialization
[cppreference implicit conversions, temproraly materialization](https://en.cppreference.com/w/cpp/language/implicit_conversion)

A [prvalue](https://en.cppreference.com/w/cpp/language/value_category#prvalue "cpp/language/value category") of any complete type `T` can be converted to an xvalue of the same type `T`. This conversion initializes a [temporary object](https://en.cppreference.com/w/cpp/language/lifetime#Temporary_object_lifetime "cpp/language/lifetime") of type T from the prvalue by evaluating the prvalue with the temporary object as its result object, and produces an xvalue denoting the temporary object.

If `T` is a class or array of class type, it must have an accessible and non-deleted destructor.
```C++
struct S { int m; };
int i = S().m; // member access expects glvalue as of C++17;
               // S() prvalue is converted to xvalue
```

Temporary materialization occurs in the following situations:
- when [binding a reference](https://en.cppreference.com/w/cpp/language/reference_initialization "cpp/language/reference initialization") to a prvalue;
- when [accessing](https://en.cppreference.com/w/cpp/language/operator_member_access "cpp/language/operator member access") a non-static [data member](https://en.cppreference.com/w/cpp/language/data_members "cpp/language/data members") of a class prvalue;
- when [invoking](https://en.cppreference.com/w/cpp/language/operator_other#Built-in_function_call_operator "cpp/language/operator other") an [implicit object member function](https://en.cppreference.com/w/cpp/language/member_functions "cpp/language/member functions") of a class prvalue;
- when performing an array-to-pointer conversion (see above) or [subscripting](https://en.cppreference.com/w/cpp/language/operator_member_access#Built-in_subscript_operator "cpp/language/operator member access") on an array prvalue;
- when initializing an object of type `std::initializer_list<T>` from a braced-enclosed initializer list;
- when a prvalue appears as a discarded-value expression.

Note that temporary materialization does **not** occur when initializing an object from a prvalue of the same type (by [direct-initialization](https://en.cppreference.com/w/cpp/language/direct_initialization "cpp/language/direct initialization") or [copy-initialization](https://en.cppreference.com/w/cpp/language/copy_initialization "cpp/language/copy initialization")): such object is initialized directly from the initializer. This ensures “guaranteed copy elision”.