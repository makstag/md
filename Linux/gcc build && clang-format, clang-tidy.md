
clang-tidy --format-style=file src/*.cpp --
clang-format -i -style=file src/*.cpp

***

cmake -G Ninja -B debug -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=Debug .

cmake --build debug

***

cmake -G Ninja -B reldebinfo -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=RelWithDebInfo .

cmake --build reldebinfo

***

cmake -G Ninja -B build -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=Release .

cmake --build build

***