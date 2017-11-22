# Go 语言核心文件调试

程序调对于检查和理解程序运行过程和状态是非常有用的。

一个【核心转储文件】(译注：core dump file)中包含程序进程运行时的内存信息和进程状态。它主要用于程序的问题调试，以及在运行过程中理解程序的状态。这些对于我们诊断程序问题原因和分析生产环境中的服务问题有非常大的帮助。

在本文中，我会用一个非常简单的hello world 网页应用服务举例，实际情况，我们的程序会更加复杂。对核心转储文件的分析意义在于可以帮助我们查看程序当时的运行情况，并可能让我们有机会重现当时的程序问题。

**注意**: 接下来的操作都是在Linux系统终端中执行，我不确定其它类Unix系统是否可以工作正常，mac OS 和 Windows 应该都不支持。

在开始之前，你需要确定已经打开了操作系统对核心转储文件的支持。`ulimit` 的默认值为0意思是说核心转储文件最大容量只能是零。我通常在开发机上设置为 `unlimited` 命令如下：

    $ ulimit -c unlimited

然后，确定你的机器上已经安装了[delve](https://github.com/derekparker/delve) 。

这是一个 `main.go` 文件，包含一个HTTP启动服务和一个处理函数。

    $ cat main.go
    package main

    import (
    	"fmt"
    	"log"
    	"net/http"
    )

    func main() {
    	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    		fmt.Fprint(w, "hello world\n")
    	})
    	log.Fatal(http.ListenAndServe("localhost:7777", nil))
    }

我们把它编译成二进制文件。

    $ go build .

我们假设下，将来这个服务可能会出现问题，但是你不知道会出现什么样的问题。你可能已经用了很多方法测试程序但仍然找不到程序异常退出的原因。

基本上，这种情况下，最好能够有当时程序进程的快照，然后用你的调试工具对快照进行调试。

有很多种方式可以获得程序的核心转储文件。你可能已经熟悉程序崩溃转储方式，当程序崩溃时会将崩溃时的程序内核信息写入磁盘文件。Go 默认是不开启程序崩溃转储的，但是你可以设置 `GOTRACEBACK` 为 `crash` 来开启 Ctrl + backslash 生成崩溃转储文件。

    $ GOTRACEBACK=crash ./hello
    (Ctrl+\)

这样就可以使程序崩溃并将堆栈跟踪打印写入核心转储文件。

另一种方法是从正在运行的进程中生成核心转储文件，而不必杀死进程。使用 `gcore` 选项就可以在不崩溃的情况下生成核心转储文件。我们重新启动程序：

    $ ./hello &
    $ gcore 546 # 546 is the PID of hello.

我们已经可以在程序不崩溃的情况下拿到核心转储文件。下一步通过 delve 加载内核转储文件进行分析。

    $ dlv core ./hello core.546

好了，就是这样。这和 delve 的普通用法没有什么不同。你可以回放，查看代码，查看变量等。有些功能会被禁用，毕竟核心转储文件只是快照，而不是真实的进程情况，但执行过程和进程状态是完全可以访问的。

    (dlv) bt
     0  0x0000000000457774 in runtime.raise
        at /usr/lib/go/src/runtime/sys_linux_amd64.s:110
     1  0x000000000043f7fb in runtime.dieFromSignal
        at /usr/lib/go/src/runtime/signal_unix.go:323
     2  0x000000000043f9a1 in runtime.crash
        at /usr/lib/go/src/runtime/signal_unix.go:409
     3  0x000000000043e982 in runtime.sighandler
        at /usr/lib/go/src/runtime/signal_sighandler.go:129
     4  0x000000000043f2d1 in runtime.sigtrampgo
        at /usr/lib/go/src/runtime/signal_unix.go:257
     5  0x00000000004579d3 in runtime.sigtramp
        at /usr/lib/go/src/runtime/sys_linux_amd64.s:262
     6  0x00007ff68afec330 in (nil)
        at :0
     7  0x000000000040f2d6 in runtime.notetsleep
        at /usr/lib/go/src/runtime/lock_futex.go:209
     8  0x0000000000435be5 in runtime.sysmon
        at /usr/lib/go/src/runtime/proc.go:3866
     9  0x000000000042ee2e in runtime.mstart1
        at /usr/lib/go/src/runtime/proc.go:1182
    10  0x000000000042ed04 in runtime.mstart
        at /usr/lib/go/src/runtime/proc.go:1152

    (dlv) ls
    > runtime.raise() /usr/lib/go/src/runtime/sys_linux_amd64.s:110 (PC: 0x457774)
       105:		SYSCALL
       106:		MOVL	AX, DI	// arg 1 tid
       107:		MOVL	sig+0(FP), SI	// arg 2
       108:		MOVL	$200, AX	// syscall - tkill
       109:		SYSCALL
    => 110:		RET
       111:
       112:	TEXT runtime·raiseproc(SB),NOSPLIT,$0
       113:		MOVL	$39, AX	// syscall - getpid
       114:		SYSCALL
       115:		MOVL	AX, DI	// arg 1 pid

----------------

via: https://rakyll.org/coredumps/

作者：[rakyll](https://rakyll.org/about/)
译者：[j.zhongming](https://github.com/jzhongming)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go中文网](https://studygolang.com/) 荣誉推出