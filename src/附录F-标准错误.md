# 附录 F 标准错误

这是标准 `Emacs` 中更重要的错误符号列表，按概念分组。该列表包括每个符号的消息和对如何发生错误的描述的交叉引用。

每个错误符号都有一组父错误条件，即符号列表。通常这个列表包括错误符号本身和错误符号。有时它包括附加符号，它们是中间分类，比错误更窄但比单个错误符号更宽。例如，所有访问文件的错误都有条件file-error。如果我们在这里不说某个错误符号有额外的错误条件，那就意味着它没有。

作为一个特殊的例外，错误符号 `quit` 和 `minibuffer-quit` 没有条件错误，因为退出不被视为错误。

这些错误符号大部分是在 `C` 中定义的（主要是 `data.c` ），但也有一些是在 `Lisp` 中定义的。例如，文件 `userlock.el` 定义了文件锁定和文件取代错误。与 `Emacs` 一起分发的几个专用 `Lisp` 库定义了它们自己的错误符号。我们不打算在这里列出所有这些。

有关如何生成和处理错误的说明，请参阅错误。

\#+begin<sub>src</sub>~ emacs-lisp
 error
\#+end<sub>src</sub>

  消息是 `错误` 。请参阅错误。
\#+begin<sub>src</sub>~ emacs-lisp
 quit
\#+end<sub>src</sub>

  消息是 `退出` 。请参阅退出。
\#+begin<sub>src</sub>~ emacs-lisp
 minibuffer-quit
\#+end<sub>src</sub>

  消息是 `退出` 。这是退出的子类别。请参阅退出。
\#+begin<sub>src</sub>~ emacs-lisp
 args-out-of-range
\#+end<sub>src</sub>

  消息是Args `超出范围` 。当试图访问超出序列、缓冲区或其他容器类对象范围的元素时，就会发生这种情况。请参阅序列、数组和向量，并参阅文本。
\#+begin<sub>src</sub>~ emacs-lisp
 arith-error
\#+end<sub>src</sub>

  消息是 `算术错误` 。尝试执行整数除以零时会发生这种情况。请参阅数值转换和算术运算。
\#+begin<sub>src</sub>~ emacs-lisp
 beginning-of-buffer
\#+end<sub>src</sub>

  消息是 `缓冲区开始` 。请参阅按字符移动。
\#+begin<sub>src</sub>~ emacs-lisp
 buffer-read-only
\#+end<sub>src</sub>

  消息是 `缓冲区是只读的` 。请参阅只读缓冲区。
\#+begin<sub>src</sub>~ emacs-lisp
 circular-list
\#+end<sub>src</sub>

  消息是 `列表包含一个循环` 。当遇到圆形结构时会发生这种情况。请参阅循环对象的读取语法。
\#+begin<sub>src</sub>~ emacs-lisp
 cl-assertion-failed
\#+end<sub>src</sub>

  消息是 `断言失败` 。当 `cl-assert` 宏未通过测试时会发生这种情况。请参阅 `Common Lisp Extensions` 中的断言。
\#+begin<sub>src</sub>~ emacs-lisp
 coding-system-error
\#+end<sub>src</sub>

 消息是无效的编码系统 。请参阅 `Lisp` 中的编码系统。
\#+begin<sub>src</sub>~ emacs-lisp
 cyclic-function-indirection
\#+end<sub>src</sub>

 消息是符号的函数间接链包含一个循环。请参阅符号函数间接。
\#+begin<sub>src</sub>~ emacs-lisp
 cyclic-variable-indirection
\#+end<sub>src</sub>

 消息是符号的变量间接链包含一个循环。请参见变量别名。
\#+begin<sub>src</sub>~ emacs-lisp
 dbus-error
\#+end<sub>src</sub>

 消息是D-Bus `错误` 。请参阅 `Emacs` 中 `D-Bus` 集成中的错误和事件。
\#+begin<sub>src</sub>~ emacs-lisp
 end-of-buffer
\#+end<sub>src</sub>

 消息是缓冲区结束。请参阅按字符移动。
\#+begin<sub>src</sub>~ emacs-lisp
 end-of-file
\#+end<sub>src</sub>

 消息是 `解析期间文件结束` 。请注意，这不是文件错误的子类别，因为它与 `Lisp` 阅读器有关，而不是文件 `I/O`  。请参阅输入函数。
\#+begin<sub>src</sub>~ emacs-lisp
 file-already-exists
\#+end<sub>src</sub>

 这是文件错误的子类别。请参阅写入文件。
\#+begin<sub>src</sub>~ emacs-lisp
 file-date-error
\#+end<sub>src</sub>

  这是文件错误的子类别。当 `copy-file` 尝试设置输出文件的最后修改时间但失败时会发生这种情况。请参阅更改文件名和属性。
\#+begin<sub>src</sub>~ emacs-lisp
 file-error
\#+end<sub>src</sub>

  我们没有列出此错误及其子类别的错误字符串，因为当错误条件 `file-error` 存在时，错误消息通常仅由数据项构成。因此，错误字符串不是很相关。但是，这些错误符号确实具有错误消息属性，如果未提供数据，则使用错误消息属性。请参阅文件。
\#+begin<sub>src</sub>~ emacs-lisp
 file-missing
\#+end<sub>src</sub>

  这是文件错误的子类别。当操作尝试对丢失的文件执行操作时会发生这种情况。请参阅更改文件名和属性。
\#+begin<sub>src</sub>~ emacs-lisp
 compression-error
\#+end<sub>src</sub>

这是文件错误的一个子类别，它是由处理压缩文件的问题引起的。请参阅程序如何加载。

    file-locked

这是文件错误的子类别。请参阅文件锁。

    file-supersession

这是文件错误的子类别。请参阅缓冲区修改时间。

    file-notify-error

这是文件错误的子类别。当无法监视文件的更改时，就会发生这种情况。请参阅文件更改通知。

    remote-file-error

这是文件错误的一个子类别，它是由于访问远程文件时出现问题而导致的。请参阅 GNU Emacs 手册中的远程文件。通常，当计时器、进程过滤器、进程标记或特殊事件通常尝试访问远程文件并与另一个远程文件操作发生冲突时，就会出现此错误。一般来说，写一个错误报告是个好主意。请参阅 GNU Emacs 手册中的错误。

    ftp-error

这是 remote-file-error 的一个子类别，它是由于使用 ftp 访问远程文件时出现问题而导致的。请参阅 GNU Emacs 手册中的远程文件。

    invalid-function

消息是 `无效功能` 。请参阅符号函数间接。

    invalid-read-syntax

该消息通常是 `无效的读取语法` 。请参阅打印表示和读取语法。当表达式后面有文本时，类似 eval-expression 的命令也会引发此错误。在这种情况下，消息是 `尾随垃圾表达式` 。

    invalid-regexp

消息是 `无效的正则表达式` 。请参阅正则表达式。

    mark-inactive

消息是 `标记现在未激活` 。见标记。

    no-catch

消息是 `没有捕获标记` 。请参阅显式非本地退出：catch and throw。

    range-error

消息是算术范围错误。

    overflow-error

消息是 `算术溢出错误` 。这是范围误差的一个子类别。这可能发生在整数超过整数宽度限制的情况下。请参阅整数基础。

    scan-error

消息是 `扫描错误` 。当某些语法解析函数发现无效的语法或不匹配的括号时，就会发生这种情况。通常使用三个参数提出：人类可读的错误消息，无法移动的障碍物的开始，以及障碍物的结束。请参阅移动平衡表达式，并参阅解析表达式。

    search-failed

消息是 `搜索失败` 。请参阅搜索和匹配。

    setting-constant

消息是 `尝试设置一个常量符号` 。当尝试将值分配给  `nil` 、t、most-positive-fixnum、most-negative-fixnum 和关键字符号时，会发生这种情况。当尝试将值分配给启用多字节字符和由于某种原因不允许直接分配的其他一些符号时，也会发生这种情况。请参阅永不改变的变量。

    text-read-only

消息是 `文本是只读的` 。这是缓冲区只读的子类别。请参阅具有特殊含义的属性。

    undefined-color

消息是 `未定义的颜色` 。请参阅颜色名称。

    user-error

消息是空字符串。请参阅如何发出错误信号。

    user-search-failed

这就像 `搜索失败` ，但不会触发调试器，如 `用户错误` 。请参阅如何发出错误信号，并参阅搜索和匹配。这用于在 Info 文件中搜索，请参阅在 Info 中搜索文本。

    void-function

消息是 `符号的函数定义无效` 。请参阅访问函数单元格内容。

    void-variable

消息是 `符号的值作为变量是无效的` 。请参阅访问变量值。

    wrong-number-of-arguments

消息是 `参数数量错误` 。请参阅参数列表的功能。

    wrong-type-argument

消息是 `错误类型参数` 。请参阅类型谓词。

    unknown-image-type

消息是 `无法确定图像类型` 。见图像。

    inhibited-interaction

消息是 `用户交互被禁止` 。当禁止交互为非零并且调用用户交互函数（如从迷你缓冲区读取）时，会发出此错误信号。

