## Debugging and Profiling

[MIT 6.NULL - Debugging and Profiling](https://missing.csail.mit.edu/2020/debugging-profiling/)

### Debugging

#### Printf Debugging and Logging
脑力劳动debug，借助打印信息思考和推断问题所在

信息除了Printf，还可以Logging，更灵活（可以输出到文件、sockets、remote servers），可复用

[Here](https://missing.csail.mit.edu/static/files/logger.py) is an example code that logs messages:

```shell
$ python logger.py
# Raw output as with just prints
python logger.py log
# Log formatted output
$ python logger.py log ERROR
# Print only ERROR levels and above
$ python logger.py color
# Color formatted output
```
利用颜色信息： [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code)

```shell
#!/usr/bin/env bash
for R in $(seq 0 20 255); do
    for G in $(seq 0 20 255); do
        for B in $(seq 0 20 255); do
            printf "\e[38;2;${R};${G};${B}m█\e[0m";
        done
    done
done
```

#### Third party logs

* UNIX系统中第三方库的log常存在`/var/log`
  * the [NGINX](https://www.nginx.com/) webserver places its logs under `/var/log/nginx`
* `systemd`, a system daemon that controls many things in your system such as which services are enabled and running 
  * `/var/log/journal`，可用`journalctl`显示
  * macOS上`var/log/system.log`，但更多人用system log，用 [`log show`](https://www.manpagez.com/man/1/log/) 显示
  * 参考[data wrangling]()，需要对log信息处理，也可以用[lvav](http://lnav.org/)，体验更好

```shell
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

#### Debuggers
共同命令：c(continue), l(ist), s(tep), n(ext), b(reak), p(rint), r(eturn), run, q(uit), watch

* `watch -l `同时监视表达式本身和表达式指向的内容

**Python**: [`ipdb`](https://pypi.org/project/ipdb/) is an improved `pdb` that uses the [`IPython`](https://ipython.org) REPL enabling tab completion, syntax highlighting, better tracebacks,  and better introspection while retaining the same interface as the `pdb` module.
* ipdb命令: `p locals()`, j(ump), pp([`pprint`](https://docs.python.org/3/library/pprint.html)), restart
* [pdb turorial](https://github.com/spiside/pdb-tutorial), [pdb depth tutorial](https://realpython.com/python-debugging-pdb)

**C++**: [`gdb`](https://www.gnu.org/software/gdb/) (and its quality of life modification [`pwndbg`](https://github.com/pwndbg/pwndbg)) and [`lldb`](https://lldb.llvm.org/)
* gdb命令: start, finish, cond, disable, where
* `cond 3 this==0xXXX`
* `gdb --args sleep 20`

[CS107 GDB and Debugging教程](https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/resources/gdb)

[CS107 Software Testing Strategies](https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/testing.html)

[lldb的使用](https://www.jianshu.com/p/9a71329d5c4d)

[macOS上配置VSCode的gdb调试环境](https://zhuanlan.zhihu.com/p/106935263?utm_source=wechat_session)  

#### Specialized Tools

Even if what you are trying to debug is a black box binary there are tools that can help you with that. Whenever programs need to perform actions that only the kernel can, they use [System Calls](https://en.wikipedia.org/wiki/System_call). There are commands that let you trace the syscalls your program makes. In Linux there’s [`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) and macOS and BSD have [`dtrace`](http://dtrace.org/blogs/about/). `dtrace` can be tricky to use because it uses its own `D` language, but there is a wrapper called [`dtruss`](https://www.manpagez.com/man/1/dtruss/) that provides an interface more similar to `strace` (more details [here](https://8thlight.com/blog/colin-jones/2015/11/06/dtrace-even-better-than-strace-for-osx.html)).

* [strace入门](https://blogs.oracle.com/linux/strace-the-sysadmins-microscope-v2)

```shell
# On Linux
sudo strace (-e lstat) ls -l > /dev/null
# On macOS
sudo dtruss -t lstat64_extended ls -l > /dev/null
```
Under some circumstances, you may need to look at the network packets to figure out the issue in your program. Tools like [`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) and [Wireshark](https://www.wireshark.org/) are network packet analyzers that let you read the contents of network packets and filter them based on different criteria.

For web development, the Chrome/Firefox developer tools are quite handy. They feature a large number of tools, including:

- Source code - Inspect the HTML/CSS/JS source code of any website.
- Live HTML, CSS, JS modification - Change the website content,  styles and behavior to test (you can see for yourself that website  screenshots are not valid proofs).
- Javascript shell - Execute commands in the JS REPL.
- Network - Analyze the requests timeline.
- Storage - Look into the Cookies and local application storage.

#### Static Analysis
* [Static Analysis介绍](https://en.wikipedia.org/wiki/Static_program_analysis)
  
  * formal methods 
  * Python: [`pyflakes`](https://pypi.org/project/pyflakes) , [`mypy`](http://mypy-lang.org/), [`shellcheck`](https://www.shellcheck.net/)
  * English也有静态分析！
  * 静态分析可以融入编辑器, vim:[`ale`](https://vimawesome.com/plugin/ale) or [`syntastic`](https://vimawesome.com/plugin/syntastic) 
* [Static Analysis仓库整理](https://github.com/analysis-tools-dev/static-analysis#go)
* [awesome linters整理](https://github.com/caramelomartins/awesome-linters#go)
  * Python:  [`pylint`](https://github.com/PyCQA/pylint) and [`pep8`](https://pypi.org/project/pep8/) 是stylistic linters，[`bandit`](https://pypi.org/project/bandit/) 可查security问题
* A complementary tool to stylistic linting are code formatters such as [`black`](https://github.com/psf/black) for Python, `gofmt` for Go, `rustfmt` for Rust or [`prettier`](https://prettier.io/) for JavaScript, HTML and CSS.


### Profiling

profilers和monitoring tools的意义：[premature optimization is the root of all evil](http://wiki.c2.com/?PrematureOptimization)

[时间概念](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)：real/user/system time

* real: 真实时间, user: 用户态耗时, sys: 内核态耗时, user+sys: 实际用时
* time指令

#### Profilers

**CPU**: [两种CPU profilers](https://jvns.ca/blog/2017/12/17/how-do-ruby---python-profilers-work-)，tracing and sampling profilers

* Python
  * cProfile: `python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py`
  *  [`line_profiler`](https://github.com/pyutils/line_profiler) 可逐行输出，用`@prifile`decorator标注函数, `kernprof -l -v a.py`

```Python
b = [2] * (2 * 10 ** 7)
del b

kernprof -l -v sorts.py
python -m line_profiler sorts.py.lprof
```

**Event Profiling** 

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) 

- `perf list` - List the events that can be traced with perf
- `perf stat COMMAND ARG1 ARG2` - Gets counts of different events related a process or command
- `perf record COMMAND ARG1 ARG2` - Records the run of a command and saves the statistical data into a file called `perf.data`
- `perf report` - Formats and prints the data collected in `perf.data`

```shell
sudo perf record stress -c 1 # record->stat
sudo perf report
```

**内存泄露问题**

* [Valgrind](https://valgrind.org/)
* `gdb`

**Visualization**

* [Flame Graph](http://www.brendangregg.com/flamegraphs.html)
*  [`pycallgraph`](http://pycallgraph.slowchop.com/en/master/) 

#### Resource Monitoring

- **General Monitoring** - Probably the most popular is [`htop`](https://hisham.hm/htop/index.php), which is an improved version of [`top`](https://www.man7.org/linux/man-pages/man1/top.1.html). `htop` presents various statistics for the currently running processes on the system. `htop` has a myriad of options and keybinds, some useful ones  are: `<F6>` to sort processes, `t` to show tree hierarchy and `h` to toggle threads.  See also [`glances`](https://nicolargo.github.io/glances/) for similar implementation with a great UI. For getting aggregate measures across all processes, [`dstat`](http://dag.wiee.rs/home-made/dstat/) is another nifty tool that computes real-time resource metrics for lots of different subsystems like I/O, networking, CPU utilization, context  switches, &c.
- **I/O operations** - [`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) displays live I/O usage information and is handy to check if a process is doing heavy I/O disk operations
- **Disk Usage** - [`df`](https://www.man7.org/linux/man-pages/man1/df.1.html) displays metrics per partitions and [`du`](http://man7.org/linux/man-pages/man1/du.1.html) displays **d**isk **u**sage per file for the current directory. In these tools the `-h` flag tells the program to print with **h**uman readable format. A more interactive version of `du` is [`ncdu`](https://dev.yorhel.nl/ncdu) which lets you navigate folders and delete files and folders as you navigate.
- **Memory Usage** - [`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) displays the total amount of free and used memory in the system. Memory is also displayed in tools like `htop`.
- **Open Files** - [`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html)  lists file information about files opened by processes. It can be  quite useful for checking which process has opened a specific file.
- **Network Connections and Config** - [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) lets you monitor incoming and outgoing network packets statistics as well as interface statistics. A common use case of `ss` is figuring out what process is using a given port in a machine. For  displaying routing, network devices and interfaces you can use [`ip`](http://man7.org/linux/man-pages/man8/ip.8.html). Note that `netstat` and `ifconfig` have been deprecated in favor of the former tools respectively.
- **Network Usage** -  [`nethogs`](https://github.com/raboof/nethogs) and [`iftop`](http://www.ex-parrot.com/pdw/iftop/) are good interactive CLI tools for monitoring network usage.

If you want to test these tools you can also artificially impose loads on the machine using the [`stress`](https://linux.die.net/man/1/stress) command.

##### Specialized tools

Sometimes, black box benchmarking is all you need to determine what software to use. Tools like [`hyperfine`](https://github.com/sharkdp/hyperfine) let you quickly benchmark command line programs. For instance, in the shell tools and scripting lecture we recommended `fd` over `find`. We can use `hyperfine` to compare them in tasks we run often. E.g. in the example below `fd` was 20x faster than `find` in my machine.

```
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

As it was the case for debugging, browsers also come with a fantastic set of tools for profiling webpage loading, letting you figure out  where time is being spent (loading, rendering, scripting, &c). More info for [Firefox](https://developer.mozilla.org/en-US/docs/Mozilla/Performance/Profiling_with_the_Built-in_Profiler) and [Chrome](https://developers.google.com/web/tools/chrome-devtools/rendering-tools).


### Exercises

(Advanced) Read about [reversible debugging](https://undo.io/resources/reverse-debugging-whitepaper/) and get a simple example working using [`rr`](https://rr-project.org/) or [`RevPDB`](https://morepypy.blogspot.com/2016/07/reverse-debugging-for-python.html).    

rr:

```
watch -l XX
reverse-cont
```

[memory-profiler](https://pypi.org/project/memory-profiler/)

[pycallgraph](http://pycallgraph.slowchop.com/en/master/)



**限制进程资源**

taskset --cpu-list 0,2 stress -c 3

[利用cgroup控制内存等资源](https://segmentfault.com/a/1190000008125359)

