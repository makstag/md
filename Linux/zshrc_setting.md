##### Тема
```console
ZSH_THEME="robbyrussell"
```
***
##### Стандартные плагины

```console
plugins=(git docker docker-compose)
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
RPROMPT='%{$fg[yellow]%}[%*]%{${reset_color}'
```
***
##### Стандартные алиасы

```console
alias cnd='source ~/anaconda3/bin/activate'
alias noetic='source /opt/ros/noetic/setup.zsh && source ~/catkin_ws/install/setup.zsh'
alias humble='source ~/ros2_humble/install/local_setup.zsh'
```
***
#### Алиасы для билда и clang'а

```console
alias debugme='cmake -G Ninja -B debug -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=Debug .'

alias reldebme='cmake -G Ninja -B reldebinfo -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=RelWithDebInfo .'

alias releaseme='cmake -G Ninja -B build -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=Release .'

alias tidyme='clang-tidy --format-style=file'

alias formatme='clang-format -i -style=file'


alias help='echo "$fg[yellow]debugme$reset_color && cmake --build debug"

echo "cmake -G Ninja -B debug -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=Debug .\n"

echo "$fg[yellow]reldebme$reset_color && cmake --build reldebinfo"

echo "cmake -G Ninja -B reldebinfo -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=RelWithDebInfo .\n"

echo "$fg[yellow]releaseme$reset_color && cmake --build build"

echo "cmake -G Ninja -B build -DCMAKE_CXX_COMPILER=g++-10 -DCMAKE_BUILD_TYPE=Release .\n"

echo "$fg[yellow]tidyme$reset_color src/*.cpp -- "

echo "clang-tidy --format-style=file\n"

echo "$fg[yellow]formatme$reset_color src/*.cpp "

echo "clang-format -i -style=file"
'

```
***