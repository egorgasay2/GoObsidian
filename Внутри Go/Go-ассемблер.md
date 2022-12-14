В качестве примера рассмотрим код ассемблера для программы goEnv.go, описанной в предыдущем разделе этой главы. Для этого мы выполним следующую команду:

```bash
$ GOOS=darwin GOARCH=amd64 go tool compile -S goEnv.go
```

Значение переменной GOOS определяет имя операционной системы, для которой  создается программа, а значение переменной GOARCH — архитектуру компиляции.  Эта команда выполнена на компьютере с macOS Mojave, поэтому переменной GOOS  было присвоено значение darwin.  Даже для такой небольшой программы, как goEnv.go, эта команда выводит довольно много данных. Часть ее результатов выглядит так:

```
"".main STEXT size=859 args=0x0 locals=0x118  
0x0000 00000 (goEnv.go:8) TEXT "".main(SB), $280-0  
0x00be 00190 (goEnv.go:9) PCDATA $0, $1  
0x0308 00776 (goEnv.go:13) PCDATA $0, $5  
0x0308 00776 (goEnv.go:13) CALL runtime.convT2E64(SB)  
"".init STEXT size=96 args=0x0 locals=0x8  
0x0000 00000 (<autogenerated>:1) TEXT "".init(SB), $8-0  
0x0000 00000 (<autogenerated>:1) MOVQ (TLS), CX  
0x001d 00029 (<autogenerated>:1) FUNCDATA $0,  
gclocals d4dc2f11db048877dbc0f60a22b4adb3(SB)  
0x001d 00029 (<autogenerated>:1) FUNCDATA $1,  
gclocals 33cdeccccebe80329f1fdbee7f5874cb(SB)
```

Переменная GOOS может принимать значения android, darwin, dragonfly, freebsd, linux, nacl, netbsd, openbsd, plan9, solaris, windows и zos; переменная GOARCH — значения 386, amd64, amd64p32, arm, armbe, arm64, arm64be, ppc64, ppc64le, mips, mipsle, mips64, mips64le, mips64p32, mips64p32le, ppc, s390, s390x, spar, spar и sparc64.

	Если вас действительно заинтересовал Go-ассемблер и вы хотите полу-  
	чить дополнительную информацию, посетите сайт https://golang.org/  
	doc/asm.

[[Внутри Go]]