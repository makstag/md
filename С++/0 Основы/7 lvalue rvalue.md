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

```C++
int x = 5;

int&& y = x; // CE

int&& y = 6; // OK
y = 7; // OK

int&& z = y; // CE, имя переменной всегда lvalue
int&& z = std::move(y); // OK, std::move возвращает rvalue
int&& t = static_cast<int&&>(x); // ОК, каст к T&& считается rvalue
t = 1; // OK, 'x' станет 1

z = y;
int&& r = 1; //
```



черновик: https://godbolt.org/z/14joMEhKs

> [!INFO]
> const type&   <- lvalue
> const type&   <- rvalue
> type&& <- rvalue
> type&& <- std::move <-lvalue

27