#lambda #лямбда 

```C++
#include <iostream>
#include <algorithm>
#include <vector>

int main()
{

  // [&] 1 1
  // inside: 2 2
  // 2 2
  {
    int x = 1, y = 1;
    std::cout << "[&] ";
    std::cout << x << " " << y << std::endl;
    auto foo = [&]() { ++x; ++y; std::cout << "inside: " << x << " " <<
                                 y << std::endl;};
    foo();
    std::cout << x << " " << y << std::endl;
  }
  std::cout << std::endl;

  // [&x] 1
  // inside: 2
  // 2
  {
    int x = 1;
    std::cout << "[&x] ";
    std::cout << x << std::endl;
    auto foo = [&x]() { ++x;  std::cout << "inside: " << x << std::endl; };
    foo();
    std::cout << x << std::endl;
  }
  std::cout << std::endl;

  // [&x, y] mutable 1 1
  // inside: 2 2
  // 2 1
  {
    int x = 1, y = 1;
    std::cout << "[&x, y] mutable ";
    std::cout << x << " " << y << std::endl;
    auto foo = [&x, y]() mutable { ++x; ++y; std::cout << "inside: " <<
                                             x << " " << y << std::endl;};
    foo();
    std::cout << x << " " << y << std::endl;
  }
  std::cout << std::endl;

  // [&, y] mutable 1 1
  // inside: 2 2
  // 2 1
  {
    int x = 1, y = 1;
    std::cout << "[&, y] mutable ";
    std::cout << x << " " << y << std::endl;
    auto foo = [&x, y]() mutable { ++x; ++y; std::cout << "inside: " <<
                                             x << " " << y << std::endl;};
    foo();
    std::cout << x << " " << y << std::endl;
  }
  std::cout << std::endl;

  // [=] mutable 1 1
  // inside: 2 2
  // 1 1
  // inside: 3 3
  // 1 1
  {
    int x = 1, y = 1;
    std::cout << "[=] mutable ";  
    std::cout << x << " " << y << std::endl;
    auto foo = [=]() mutable { ++x; ++y; std::cout << "inside: " <<
                                         x << " " << y << std::endl;};
    foo();
    std::cout << x << " " << y << std::endl;
    foo();
    std::cout << x << " " << y << std::endl;
  }
  std::cout << std::endl;

  // [=, &y] mutable 1 1
  // inside: 2 2
  // 1 2
  // inside: 3 3
  // 1 3
  {
    int x = 1, y = 1;
    std::cout << "[=, &y] mutable ";  
    std::cout << x << " " << y << std::endl;
    auto foo = [=, &y]() mutable { ++x; ++y; std::cout << "inside: " <<
                                             x << " " << y << std::endl;};
    foo();
    std::cout << x << " " << y << std::endl;
    foo();
    std::cout << x << " " << y << std::endl;
  }   
  std::cout << std::endl;

  // [x] mutable 1
  // inside: 2
  // 1
  // inside: 3
  // 1
  {
    int x = 1;
    std::cout << "[x] mutable ";  
    std::cout << x << std::endl;
    auto foo = [x]() mutable { ++x; std::cout << "inside: " <<
                                    x << std::endl; };
    foo();
    std::cout << x << std::endl;
    foo();
    std::cout << x << std::endl;
  }
  std::cout << std::endl;

}
```