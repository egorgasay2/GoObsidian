Обычно файлы считывают не одним куском, а _буфером_ фиксированного размера. Для этого подойдет метод `os.File.Read()`.

Допустим, есть файл `awesome.txt` с таким содержимым:

```no-highlight
go is awesome
```

Прочитаем его буфером размером в 5 байт:

```go
file, err := os.Open("awesome.txt")     // (1)
if err != nil {
    panic(err)
}
defer file.Close()                      // (2)

buf := make([]byte, 5)                  // (3)
for {
    n, err := file.Read(buf)            // (4)
    fmt.Println(n, err)
    if err == io.EOF {                  // (5)
        break
    }
    if err != nil {
        panic(err)
    }
    fmt.Printf("read %d bytes: %q\n", n, buf[:n])  // (6)
}
```

`os.Open()` открывает файл для чтения и возвращает объект типа `File` ➊. Когда закончим работать с файлом, его необходимо закрыть, чтобы освободить занятые ресурсы ➋.

Мы используем буфер фиксированного размера в 5 байт ➌. Метод `File.Read()` считывает очередную порцию данных из файла в буфер ➍. Он возвращает количество считанных байт (обычно оно равно размеру буфера) и ошибку.

Если данные в файле закончились, `File.Read()` возвращает особое значение ошибки — `io.EOF` ➎. Мы ориентируемся на него, чтобы выйти из цикла.

Наконец, мы выводим `n` прочитанных байт на экран ➏.

Результат работы программы:

```go
5 <nil>
read 5 bytes: "go is"
5 <nil>
read 5 bytes: " awes"
3 <nil>
read 3 bytes: "ome"
0 EOF
```

Как видите, n равно размеру буфера (5 байт) до тех пор, пока в файле есть по крайней мере 5 непрочитанных байт. В предпоследнем вызове `File.Read()` осталось только 3 байта (`"ome"`), поэтому n = 3. Последний вызов `File.Read()` возвращает n = 0 и ошибку `io.EOF`, поэтому буфер мы не печатаем, а выходим из цикла.

`File.Read()` демонстрирует распространенный подход: читаем данные в цикле буфером фиксированного размера, пока не получим `io.EOF`. Но на практике чаще используют не его, а другой похожий инструмент.

[[Стандартная библиотека]]