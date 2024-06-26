#lvalue #rvalue

>[!IMPORTANT]
>lvalue and rvalue are categoty of expressions, not types

> [!INFO]
> Вид `value` тернарного оператора зависит от других его частей  . Если обе его части `lvalue` - он сам `lvalue`, но если хоть одна из частей `rvalue` - тернарный оператор будет `rvalue`.
#### lvalue:
- Вызов функции (или каста) считается lvalue если возвращаемый тип - T&
- идентификатор;
- строковый литерал `"abc"`;
- результат выражений `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `<<=`, `>>=`, `&=`, `|=`, `^=`;
- `++expr`, `--expr`;
- `*ptr`;
- `a[i]`;
- `,` if rhs is lvalue;

#### rvalue:
- Вызов функции (или каста) считается Rvalue если возвращаемый тип - T или T&&
- литерал `5`, `2.0f`, `true`;
- результат вырвжений `+` `-` `*` `/` `%` `<<` `>>` `&` `|` `&&` `||` `~` `<` `>` `<=` `>=` `==` `!=`;
- `expr++`, `expr--`;
- `&a`;
- `,` if rhs is rvalue;

***
## Rvalue-ссылки и их свойства

1) `rvalue`-ссылку можно проинициализировать только `rvalue`-выражением
2) Если `rvalue`-ссылку вернуть из функции, это будет `rvalue-expression`
3) `rvalue`-ссылки, как и `const lvalue`-ссылки, умеют продлевать жизнь объекту
4) `const` для `rvalue`-ссылок работает так же как для обычных (чтобы сделать `move` для `const T&&` придется снять константность через `const_cast`)

> [!WARNING]
> Если `rvalue`-ссылка `T&&` является типом аргумента в данной функции, для которой `T` является шаблонным параметром - это все работает по другим правилам. Это сделано для того, что бы таким синтаксисом (`T&&`) можно было принимать как `rvalue` так и `lvalue`. При этом `const T&&` уже не является исключением из правил.

Правила работы для типа аргумента в функции`T&&` (имеет название `forwarding-references` или `universal-references`), для которой `T` является шаблонным параметром:
1) Если передавать в `T&&` `lvalue` - меняются правила вывода шаблонов
2) Если явно указать тип `T` при вызове шаблонной функции, правила вывода типов в принципе не применяются
```C++
template<typename T>
void h(T&& x){};

template<typename T>
void g(T& x){};

template<typename T>
void f(T x){};

int x = 0;
f(x); // T - int
g(x); // T - int
h(x); // T - int&

int& y = x;
f(y); // T - int
g(y); // T - int
h(y); // T - int&

int&& z = std::move(x);
f(z); // T - int
g(z); // T - int
h(z); // T - int& (передаем lvalue)

{
int x = 0;
f(1); // T - int
g(2); // T - int
h(3); // T - int (передаем rvalue)
h(std::move(x)); // T - int (передаем rvalue)
}
```
 
***

#todo описать что происходит
```C++
#include <iostream>
#include <string>

int main() {
    {
    const std::string x = "abc";
    std::string y = std::move(x); // произойдет copy, а не move

    std::cout << x << " " << y << std::endl;
    y = "abc2";
    std::cout << x << " " << y << std::endl;
    }

    {
    std::string x = "abc";
    std::string &&y = std::move(x); // будет "abc"
    std::string &&z = std::move(y); // будет "abc"

    std::cout << x << " " << y << " " << z << std::endl;
    z = "heh";
    std::cout << x << " " << y << " " << z << std::endl;

    std::string t = std::move(x); // теперь x, y, z - пустые строки
    std::cout << std::boolalpha << x.empty() << " " << y.empty()
              << " " << z.empty() << " " << t << std::endl;
    }

    {
    int x = 5;
    // int&& y = x; // CE
    int&& y = std::move(x); // OK
    int&& z = std::move(y); // OK, std::move возвращает rvalue
    int i = std::move(x); // ОК, копирование для тривиальных типов
    std::cout << x << " " << y << " " << z << " " << i << std::endl;
    y = 7; // OK
    std::cout << x << " " << y << " " << z << " " << i << std::endl;
    }
}
```

> [!INFO]
> const type&   <- lvalue
> const type&   <- rvalue
> type&& <- rvalue
> type&& <- std::move <-lvalue

***
## Ссылочные квалификаторы
#квалификаторы #reference #qualifiers

Можно сделать перегрузку между rvalue и lvalue объектами (начиная с C++11)

```C++
stuct S {
	string str;

	// (1)
	// можно вызываться как от lvalue, так и от rvalue
	string getData() const {
		return str;
	}
	// (2)
	// то же самое, что и (1), можно вызываться как от lvalue так и от rvalue
	string getData() const & { // приняли this по константной ссылке
		return str;
	}
	// (3)
	// разрешим вызывать getData ТОЛЬКО для lvalue
	// т.к. неконстантная lvalue-ссылка не может быть проинициализированна
	// через rvalue
	string getData() & {
		return str;
	}
	// (4)
	// если this - rvalue будет выбрана эта перегрузка
	string getData() && { // приняли this по rvalue ссылке
		return std::move(str);
	}
};

// lvalue
S s = S{"abc"};
s.getData();        // строка str будет создана и скопирована

// rvalue
S{"cba"}.getData(); // строка будет создана, а затем произойдет move
```

***
