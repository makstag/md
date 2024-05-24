
bool test_and_set( std::memory_order order =
                       std::memory_order_seq_cst ) volatile noexcept;

Атомарно устанавливает значение флага в true и возвращает предыдущее значение.

***

T exchange( T desired, std::memory_order order =
                           std::memory_order_seq_cst ) noexcept;


Позволяет атомарно заменить ранее сохраненное значение новым (true or false) и вернуть прежнее значение.

***

bool compare_exchange_weak( T& expected, T desired,
                            std::memory_order success,
                            std::memory_order failure ) noexcept;

bool compare_exchange_strong( T& expected, T desired,
                              std::memory_order success,
                              std::memory_order failure ) noexcept;

Атомарно сравнивает *this с expected. Если они побитово равны - заменяет *this на desired и возвращает true. В противном случае загружает *this в expected и возвращает false.

***

fetch_add() , fetch_sub() , fetch_and() , fetch_or() , fetch_xor() ,

Выполняют свои операции атомарно и возвращают старое значение.

( += , -= , &= , |= , ^= ) 

Выполняют свои операции атомарно и возвращают новое значение.

( ++x , x++ , --x, x-- )

Операторы пред- и постинкремента и декремента работают как обычно: ++x увеличивает значение переменной на единицу и возвращает новое значение, а x++ увеличивает значение переменной на единицу и возвращает старое значение.

***