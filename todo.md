
https://stackoverflow.com/questions/42011458/static-cast-from-base-class-pointer-to-derived-class-pointer-is-invalid

```C++
#include <iostream>

struct Base {
    int a = 0;
    Base(){
        std::cout << "default" << std::endl;
    }
    Base(const Base&){
        std::cout << "copy" << std::endl;
    }
    virtual void foo(){};
};

struct Derived: public Base {
    int a = 1;
    int b = 2;
    Derived() = default;
    void foo() override {};
};

void f(Base b){
    std::cout << b.a << std::endl;  // 0
}

void f_ptr(Base* b){
    std::cout << b->a << std::endl; // 0
}

void f_ref(Base& b){
    std::cout << b.a << std::endl;  // 0
}

int main() {
    Derived d; // Выведется "default"
    Base& b = d;

//  Derived  new_d1 = b;
//  Derived& new_d2 = b;
    Derived& new_d3 =  static_cast<Derived&>(b);
    Derived& new_d4 = dynamic_cast<Derived&>(b);

    d.a = 77;

    std::cout << new_d3.a << std::endl;
    std::cout << new_d3.a << std::endl;

    Base bb;
    Derived& new_d5 = static_cast<Derived&>(bb);
    std::cout << new_d5.b << std::endl;

    Derived& new_d6 = dynamic_cast<Derived&>(bb);

    return 0;
}
```



кастование классов разобраться
Полиморфизм и виртуальные функции - дополнить код 1
перенести статью про потоки
перенести про числа с плавающей точкой


разное
- https://cppinsights.io/
- http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-make_shared
- https://www.tutorialspoint.com/compile_cpp_online.php
- https://godbolt.org/
- http://cppstudio.com/cat/309/


1) числа с плавающей точкой https://www.h-schmidt.net/FloatConverter/IEEE754.html https://www.learncpp.com/cpp-tutorial/relational-operators-and-floating-point-comparisons/

2) Дебаг vscode https://www.geeksforgeeks.org/vs-code-build-run-and-debug-in-c/

3) shared_ptr https://www.geeksforgeeks.org/shared_ptr-in-cpp/

4) способы передачи переменных в функцию https://ru.stackoverflow.com/questions/265785/%D0%A1%D0%BA%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE-%D0%B5%D1%81%D1%82%D1%8C-%D1%81%D0%BF%D0%BE%D1%81%D0%BE%D0%B1%D0%BE%D0%B2-%D0%BF%D0%B5%D1%80%D0%B5%D0%B4%D0%B0%D1%87%D0%B8-%D0%B0%D1%80%D0%B3%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D0%BE%D0%B2-%D0%B2-%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8E

5) особенности работы со строками https://younglinux.info/c/strings-pointers

6) адресная арифметика https://pvs-studio.ru/ru/blog/terms/0005/ https://pvs-studio.ru/ru/blog/lessons/0013/ https://pvs-studio.ru/ru/blog/posts/cpp/0576/ https://www.open-std.org/JTC1/SC22/WG14/www/docs/n2012.htm

7) спинлок https://rigtorp.se/spinlock/