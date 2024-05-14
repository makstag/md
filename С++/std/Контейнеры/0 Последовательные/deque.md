#deque #not_complited

https://www.youtube.com/watch?v=8629XIx3WQY&list=PLmSYEYYGhnBviRYhIDty-CSTDS16a3whl&index=29

https://www.youtube.com/watch?v=N2-hjzXqZ_Y&t=2026s

![[deque.png]]

Отличие std::deque от std::vector:
 - умеет делать pushfront и popfront
 - не инвалидирует указатели и ссылки при добавлении чего-то
 - отсутствуют методы shrink_to_fit() reserve() capacity() 

>В deque при добавления элемента в конец или начало указатели и ссылки на элементы не инвалидируются в отличие от vector'а. Но все еще инвалидируются итераторы.

std::stack std::queue std::priority_queue - адаптеры над std::deque (по дефолту, но можно сделать на основе std::vector и т.д). Пример объявления адаптера stack:

```C++
template <typename T, typename std::deque<T>> std::stack
```

***

Наивная реализация deque:
```C++

```
