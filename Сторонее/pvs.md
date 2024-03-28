# Контроль качества кода
Для контроля качества кода можно использовать статические анализаторы, а также средства автоматического форматирования, для следования code style.

# Статический анализатор PVS-Studio

[https://pvs-studio.com/ru/](https://pvs-studio.com/ru/)

Анализатор предназначен для автоматического выявления типичных ошибок, уязвимостей и способов увеличения производительности, поддерживает стандарты MISRA, OWASP, AUTOSAR ([ссылка](https://pvs-studio.com/ru/blog/posts/0774/)).

Анализатор работает с языками С, С++, С#, Java и поддерживает интеграцию в среды разработки VIsual Studio, CLion, Rider, Qt Creator. Интеграции с VS Code на текущий момент нет. Но анализатор может генерировать полноценные интерактивные отчеты в формате HTML.

Анализатор является платным, но для пробных целей можно воспользоваться [https://pvs-studio.com/ru/order/for-students/](https://pvs-studio.com/ru/order/for-students/)

## Использование с catkin проектами внутри docker контейнера

Скачать дистрибутив [https://pvs-studio.com/ru/pvs-studio/download/](https://pvs-studio.com/ru/pvs-studio/download/)

Далее необходимо закинуть .deb файл внутрь контейнера и установить его:

`sudo` `dpkg -i pvs-studio-7.14.50353.142-amd64.deb`

После этого активировать бесплатную лицензию:

`pvs-studio-analyzer credentials` `"PVS-Studio Free"` `FREE-FREE-FREE-FREE`

Чтобы анализатор отработал, необходимо во все файлы, используемые для формирования единиц трансляции (*.c, *.cpp) добавить в начало специальные комментарии:

`// This is a personal academic project. Dear PVS-Studio, please check it.`

`// PVS-Studio Static Code Analyzer for C, C++, C#, and Java: [http://www.viva64.com](http://www.viva64.com)`

Далее запустить сборку через catkin_make(_isolated) с дополнительным флагом CMAKE_EXPORT_COMPILE_COMMANDS=On, например:

`catkin_make_isolated -DCMAKE_BUILD_TYPE=Release --only-pkg-with-deps lol -j8 -DCMAKE_EXPORT_COMPILE_COMMANDS=On --merge`

При этом в папке build_isolated/lol будет создан файл compile_commands.json, необходимый для работы анализатора.

После этого нужно создать папку, которая будет видна на хосте и в которой будут размещены отчеты анализатора, например:

`mkdir` `src``/PVS-Studio/lol` `-p`

Далее запустить сам анализатор:

`pvs-studio-analyzer analyze -j8 -l` `/home/docker_lol/``.config``/PVS-Studio/PVS-Studio``.lic -o src``/PVS-Studio/lol/PVS-Studio``.log -f build_isolated``/lol/compile_commands``.json -a` `"GA;OP;CS;MISRA;AUTOSAR;OWASP"` `-e` `/usr/include/` `-e` `/usr/local/include/` `-e` `/opt/ros/noetic/include/`

После этого сформировать отчет в виде HTML:

`plog-converter -a` `"GA:1,2,3;OP:1,2,3;CS:1,2,3;MISRA:1,2,3;AUTOSAR:1,2,3;OWASP:1,2,3"` `-t fullhtml -o src/PVS-Studio_lol/ -d V009 src/PVS-Studio/lol/PVS-Studio.log`

Отчет будет размещен в src/PVS-Studio/lol/fullhtml. Для его просмотра достаточно открыть в браузере файл index.html
![](attachments/8b63bbb680322d0d8026123cc41ae99b.png)