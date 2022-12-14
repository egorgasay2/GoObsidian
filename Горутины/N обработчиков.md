Помните, как мы запускали болтушки в горутинах — по одной на каждую фразу?

```go
func main() {
    phrases := []string{
        // ...
    }
    for idx, phrase := range phrases {
        go say(idx+1, phrase)
    }
}
```

Горутины — легковесные объекты. Вполне можно запустить одновременно 10, 100 или 1000 штук. Но что, если исходных фраз будет 100 тысяч или миллион? Понятно, что реальная многозадачность все равно ограничена количеством ядер. Так что нет смысла впустую расходовать память на сотни тысяч горутин, если параллельно выполняться все равно будут только восемь (или сколько у вас там CPU).

Скажем, мы хотим, чтобы одновременно существовали только N say-горутин. Добиться этого поможет буферизованный канал. Идея следующая:

-   создаем канал с буфером размера N и заполняем его «токенами» (любыми значениями);
-   перед запуском горутина забирает токен из канала;
-   по завершении работы горутина возвращает токен в канал.

Таким образом, если в канале не осталось токенов, то очередная горутина не запустится и будет ждать, пока кто-нибудь вернет токен в канал. В результате одновременно запущены будут не более N горутин.

Вот как это может выглядеть при N = 2:

```go
func main() {
    phrases := []string{
        // ...
    }

    // пул идентификаторов для 2 горутин
    pool := make(chan int, 2)
    pool <- 1
    pool <- 2

    for _, phrase := range phrases {
        // получаем идентификатор из пула,
        // если есть свободные
        id := <-pool
        go say(pool, id, phrase)
    }

    // дожидаемся, пока все горутины закончат работу
    // (то есть все идентификаторы вернутся в пул)
    <-pool
    <-pool
}
```

```go
func say(pool chan<- int, id int, phrase string) {
    for _, word := range strings.Fields(phrase) {
        fmt.Printf("Worker #%d says: %s...\n", id, word)
        dur := time.Duration(rand.Intn(100)) * time.Millisecond
        time.Sleep(dur)
    }
    // возвращаем идентификатор в пул
    pool <- id
}
```

`main()` проходит по исходным фразам, для каждой фразы получает токен из пула и запускает say-горутину. Say-горутина печатает фразу и возвращает токен в пул. Таким образом, фразы обрабатываются одновременно, причем каждая фраза печатается только один раз:

```http
Worker #2 says: go...
Worker #1 says: cats...
Worker #2 says: is...
Worker #1 says: are...
Worker #2 says: awesome...
Worker #1 says: cute...
Worker #1 says: rain...
Worker #1 says: is...
...
```

В качестве токенов используются идентификаторы (порядковые номера 1, 2), но это исключительно для наглядности вывода в `say()`. Токенами могут быть пустые структуры или любые другие значения.

[песочница](https://go.dev/play/p/8ibrtZEI2Fh)

### Альтернативный вариант

Решить задачу можно было и без пула, как мы это делали на шаге [Четыре счетовода](https://stepik.org/lesson/740355/step/2?unit=742025). Бросаем исходные данные в канал и запускаем N горутин, которые его разгребают:

```go
func main() {
    phrases := []string{
        // ...
    }

    pending := make(chan string)

    go func() {
        for _, phrase := range phrases {
            pending <- phrase
        }
        close(pending)
    }()

    done := make(chan struct{})

    go say(done, pending, 1)
    go say(done, pending, 2)

    <-done
    <-done
}
```

```go
func say(done chan<- struct{}, pending <-chan string, id int) {
    for phrase := range pending {
        for _, word := range strings.Fields(phrase) {
            fmt.Printf("Worker #%d says: %s...\n", id, word)
            dur := time.Duration(rand.Intn(100)) * time.Millisecond
            time.Sleep(dur)
        }
    }
    done <- struct{}{}
}
```

В таком варианте мы зависим от поставщика данных в канал `pending`, который в нужный момент закрывает канал. Если исходные данные заранее неизвестны и приходят с непонятной частотой — вариант с пулом токенов может быть удобнее.

Разница еще в том, что в первом подходе (с пулом токенов) мы запускаем множество короткоживущих горутин, а во втором (с разгребанием канала) — две долгоживущие.