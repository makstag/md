[Магистерский курс C++ (МФТИ, 2022-2023). Лекция 19. Многопоточность, часть 1](https://www.youtube.com/watch?v=xTpAJWe7ZD4&list=PL3BR09unfgcgf7R88ZQRQqWOdLy4pRW2h&index=20&t=2753s)

Правила гонки -  гонка происходит если сколько угодно потоков читают область памяти и хотя бы один одновременно пишет в нее же.

```C++
unsigned x = 0, i = 0, j = 0; // области памяти
void readerf() { while (i++ < 'g') x += 0x1; }
void writerf() { while (j++ < 'g') x += 0x10000; }
std::thread t1{readerf}, t2{writerf};
// Все равно не определено, что будет на экране, несмотря на то, что запись и
// чтение в разнах потоках 'трогают' разные байты
t1.join(); t2.join();
```

Если заменить `unsigned x = 0` на `char x[2] = {0, 0}` - это уже не будет UB так как теперь x это не сказярный объект, а агрегат, который состоит из скалярных объектов.

> [!TIP]
>`std::cout`, `std::cin`, `std::cerr` по стандарту - **thread safe**. `std::ostream`'ы **не thread safe**

Интерфейс std::mutex:
- lock() - Попытка вызвать повторно в том же потоке это UB. Также может кинуть исключение std::system_error
- try_lock() - То же, что и для lock()
- unlock() - Попытка разблокировать не захваченый mutex это тоже UB

std::lock_guard<__T__> это RAII обертка над любым классом, поддерживающим интерфейс из методов lock() и unlock(). lock_guard нельзя копировать и перемещать. Использование lock_guard не дороже, чем использование mutex.

***

Лучше нейтральности

Безопасным относительно многопоточного окружения являтеся объект, никакие операции с которым в этом окружении не приводят к data race Шире говоря: не приводят к неконсистентному состоянию.


Нулевой уровень безопасности

у нас есть нулевой уровень, это нейтральные объекты, например int

Если int защищен синхронизацией - он безопасен, если нет, то нет


Хуже нейтральности

Указатель ведет себя хуже нейтрального уровня безопасности и даже mutex не защитит

***

>[!WARNING]
>`lock_guard<mutex>{buf_mut_};` - типичная ошибка, если не дать имя lock_guard он сразу же уничтожится и отпустит mutex

## API races

Представим себе следующий кусок кода, исполняемый сразу двумя потоками на одном глобальном объекте S. В данном случае может дважды произойти вызов pop() даже если `S.size() == 1`. Или же может дважды произойти удаление одного и того же объекта, что приведет к ошибке. 

```C++
if (!S.empty()) {
	auto elem = S.top();
	S.pop();
	use(elem);
}
```

## std::lock()

Решение проблемы обедающих философов:

```C++
void take(mutex &left, mutex &right) {
	for (;;) {
		left.lock();
		if (right.try_lock())
			break;
		left.unlock();
		std::this_thread:: yield();
	}
}
```

std::lock() решает эту задачу в обзем случае и позволяет безопасно захватить произвольное количество мьютексов. Но придется делать ручной unlock() в конце критической секции. Можно этого избещать и использовать идиому RAII и std::adopt_lock():

```C++
template <typename T>
void MyBuffer::swap(MyBuffer<T> &rth) noexcept {
	if (this == &rth)
		return;
	std::lock(bufmut_, rts.bufnut_);
	// std::adopt_lock не блокирует мьютекс, но приминает уже заблокированный
	// мьютекс, чтобы правильно его отпустить в деструкторе
	std::lock_guard<mutex> lk1(bufmut_, std::adopt_lock);
	std::lock_guard<mutex> lk2(rhs.bufmut_, std::adopt_lock)
	;
	swap(arr_, rhs.arr_);
	swap(size_, rhs.size_);
	swap(used_, rhs.used_);
}
```

Начиная с C++17 можно использовать RAII оболочку `std::scoped_lock` - она внутри использует алгоритм std::lock, чтобы правильно заблокировать несколько мьютексов и правильно их отпускает в деструкторе.

```C++
template <typename T>
void MyBuffer::swap(MyBuffer<T> &rth) noexcept {
	if (this == &rth)
		return;
	std::scoped_lock sl(bufmut_, rts.bufnut_);
	swap(arr_, rhs.arr_);
	swap(size_, rhs.size_);
	swap(used_, rhs.used_);
}
```

>[!TIP]
>`std::scoped_lock` тяжелее и сложнее, чем `std::lock_guard`. Если нужно взять один мьютекс - лучше использовать `std::lock_guard`

***
лекция 2

new работает с кучей, которая общая, поэтому внутри него есть mutex. В высоконагруженных системах внутри него начинают сериализоваться потоки.


В данном коде есть проблема в том, что mutex захватывается в любом случае даже если не попадем в услование if,  
```C++
{
	lock_guard<mutex> lk{resmut};
	if (!resptr)
		resptr = new Resource(); // создание требует синхронизации
}
resptr->use(); // use это const функция, синхронизация не требуется
```

Можно было бы попробовать сделать проверку - double-check lock, но тогда мы получаем UB (хотя практически не проявляется)
```C++
if (!resptr) { // UB, чтение может быть одновременно с записью в другом потоке
	lock_guard<mutex> lk{resmut};
	// компилятор не будет оптимизировать два if'a так как между ними есть
	// side effect - конструктор lock_guard'a который идет в POSIX
	if (!resptr)
		resptr = new Resource(); // создание требует синхронизации
}
resptr->use(); // use это const функция, синхронизация не требуется
```

Специальный примитив, вместе с std::call_once защищающий однократное создание. Расходы на это решение существенно меньше, чем на постоянную сериализацию вокруг мьютекса.
```C++
resource *resptr;
std::once_flag resflag;
void init_resource() { resptr = new resource();}

// И где-то далее в коде
std::call_once(resflag, init_resource);
resptr->use();
```

Наивная попытка передать состояние в другой поток:
```C++
volatile int resready = 0; resource *resptr;

void foo() { // will be called by thread 1
	resptr = new resource();
	resready = 1; // data race
}

void bar() {
	while (!resready) { std::this_thread::yield(); }
	resptr->use();
}
```

```C++
resource *resptr = nullptr;
std::mutex resmut;
std::condition_variable data_cond;

// thread 1
{
	lock_guard<mutex> lk{resmut};
	resptr = new resource();
}
data_cond.notify_one();

// thread 2
data_cond.wait(resmut);
resptr->use();
resmut.unlock(); // теряем RAII, чего не хочется
```

Класс std::unique_lock предоставляет уникальное владение блокировкой.
```C++
{
std::unique_lock<mutex> ul{resmut}; // locked by ctor
res->use();
ul.unlock();
// something on unlocked
ul.lock();
res->use();
} // unlocked by dtor
```

И вот теперь да, в языке могут существовать условные переменные. Всё ещё есть небольшая проблема - ожидание data_cond.wait(lk) может закончиться само по себе (spuriously)
```C++
resource *resptr = nullptr;
std::mutex resmut;
std::condition_variable data_cond;

// thread 1
{
	lock_guard<mutex> lk{resmut};
	resptr = new resource();
}
data_cond.notify_one();

// thread 2
std::unique_lock<mutex> lk{resmut};
data_cond.wait(lk); // unlock & wait
resptr->use(); // lock obtained
```

> [!TIP]
> spuriously wekeup . Для того, чтобы снова взять блокировку при пробуждении поток вызывает sys call. Время на вызов sys call'a  может привести к изменению обстоятельств пробуждения.

Чтобы избежать spuriously wekeup в `data_cond.wait`  мы передаем не только mutex, но и проверку на ложные пробуждения.
```C++
data_cond.wait(lk, []{
	return (resptr != nullptr);
});
```

## strace

-f (follow)
```bash
strace -f ./a.out
```

## perf

```bash
sudo perf record -e intel_pt/mtc,mtc_period=0/u ./a.out

perf script -F comm,tid,time,cpu,ip,sym,symoff,srcline --itrace=c --ns > dump.txt

grep -E 'lock|unlock|wait|notify|std::cout' -B2 dump.txt
```

## Разделяемые блокировки

shared_lock_bench.cc

Решение: расшарить мьютекс для чтения и уникально захватить на запись. Если `unique_lock()` захвачен - захвати mutex, если нет - `shared_lock()` "проскальзывает" и не захватывает mutex.
```C++
mutable std::shared_mutex m_; T value_;

T get() const {
	std::shared_lock<std::shared_mutex> lock{m_};
	return value_;
}

void modify(const T &newval) {
	std::unique_lock<std::shared_mutex> lock{m_};
	value_ = newval;
}
```

1:15
https://www.youtube.com/watch?v=vVRNJjf1MCE&list=PL3BR09unfgcgf7R88ZQRQqWOdLy4pRW2h&index=21