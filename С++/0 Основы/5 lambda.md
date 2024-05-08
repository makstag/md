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

## Сравнение скорости выполнения

```C++
#include <algorithm>
#include <iostream>
#include <iterator>
#include <random>
#include <vector>
#include <chrono>

typedef std::chrono::high_resolution_clock Clock;

/*
g++ -O3 -std=c++2a -Wall -Wextra -Wpedantic -Wconversion lambda_comporation.cpp -o lc.out && ./lc.out
*/

int main() {

    std::random_device rd;
    std::mt19937 mt(rd());
    std::uniform_int_distribution<> dis(0, 100);
 
    std::vector<int> v1(1000000000);
    std::generate(v1.begin(), v1.end(), std::bind(dis, std::ref(mt)));

    size_t count{0};

//  -------------------------------------------------------------------------  //
    auto t1 = Clock::now();
    for (auto& it : v1) {
        if ( it > 20 && it < 50 ) {
            ++count;
        }
    }
    auto t2 = Clock::now();

    // 287127994 217'605'496 for
    std::cout << count << " ";
    std::cout << std::chrono::duration_cast<std::chrono::nanoseconds>(t2 - t1).count()
              << " for" << std::endl;
//  -------------------------------------------------------------------------  //

    t1 = Clock::now();
    count = std::count_if(v1.begin(), v1.end(), [](int i) { return i > 20 && i < 50; } );
    t2 = Clock::now();

    // 287127994 236'528'866 lambda int
    std::cout << count << " ";
    std::cout << std::chrono::duration_cast<std::chrono::nanoseconds>(t2 - t1).count()
              << " lambda int" << std::endl;

//  -------------------------------------------------------------------------  //

    t1 = Clock::now();
    count = std::count_if(v1.begin(), v1.end(), [](int& i) { return i > 20 && i < 50; } );
    t2 = Clock::now();

    // 287127994 237'925'078 lambda int&
    std::cout << count << " ";
    std::cout << std::chrono::duration_cast<std::chrono::nanoseconds>(t2 - t1).count()
              << " lambda int&" << std::endl;
//  -------------------------------------------------------------------------  //
}
```