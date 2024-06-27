#### Работа с субмодулями
```bash
git submodule init
git submodule update
```
Добавить субмодуль в проект
```bash
git submodule add https://gitlab.com/kovachilya/ros.git
```
Проверка на наличие изменений в субмодуле при push
```bash
git push --recurse-submodules=check
```
Для отслеживания нужной ветки в субмодуле
```bash
git config -f .gitmodules submodule.ros.branch master  
```