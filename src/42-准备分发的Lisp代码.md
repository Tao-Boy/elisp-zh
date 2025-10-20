# 42 准备分发的Lisp代码

Emacs 提供了一种将 Emacs Lisp 代码分发给用户的标准方法。包是一个或多个文件的集合，其格式和捆绑方式使用户可以轻松下载、安装、卸载和升级它。

以下部分描述了如何创建一个包，以及如何将它放在一个包存档中以供其他人下载。请参阅 The GNU Emacs Manual 中的 Packages，了解对打包系统的用户级功能的描述。

这些部分主要针对包存档维护者——其中大部分信息与包作者（即编写将通过这些存档分发的代码的人）无关。


<a id="orgcc32633"></a>

## 42.1 包装基础

包可以是简单包，也可以是多文件包。一个简单的包以单个 Emacs Lisp 文件的形式存储在包存档中，而多文件包则以 tar 文件的形式存储（包含多个 Lisp 文件，可能还有非 Lisp 文件，例如手册）。

平时使用中，简单包和多文件包的区别相对不重要；  Package Menu 界面不区分它们。但是，创建它们的过程有所不同，如以下部分所述。

每个包（无论是简单的还是多文件的）都有一定的属性：

    Name

一个简短的词（例如，'auctex'）。这通常也是程序中使用的符号前缀（参见 Emacs Lisp 编码约定）。

    Version

版本号，采用函数version-to-list 可以理解的形式（例如，'11.86'）。软件包的每次发布都应伴随版本号的增加，以便用户查询软件包存档时将其识别为升级。

    Brief description

当包在包菜单中列出时显示。它应该占一行，最好是 36 个字符或更少。

    Long description

这显示在由 Ch P (describe-package) 创建的缓冲区中，遵循包的简要描述和安装状态。它通常跨越多行，并且应该完整地描述包的功能以及安装后如何开始使用它。

    Dependencies

此包所依赖的其他包的列表（可能包括可接受的最低版本号）。该列表可能为空，这意味着此包没有依赖项。否则，安装这个包也会自动递归地安装它的依赖；  如果找不到任何依赖项，则无法安装该软件包。

通过命令 package-install-file 或通过 Package Menu 安装包，会在 package-user-dir 的名为 name-version 的子目录中创建一个子目录，其中 name 是包的名称，版本是其版本（例如，~/. emacs.d/elpa/auctex-11.86/）。我们称之为包的内容目录。它是 Emacs 放置包内容的地方（简单包的单个 Lisp 文件，或从多文件包中提取的文件）。

然后 Emacs 在内容目录中的每个 Lisp 文件中搜索自动加载魔术注释（参见 Autoload）。这些自动加载定义保存到内容目录中名为 name-autoloads.el 的文件中。它们通常用于自动加载包中定义的主要用户命令，但它们也可以执行其他任务，例如向 auto-mode-alist 添加元素（请参阅 Emacs 如何选择主要模式）。请注意，包通常不会自动加载其中定义的每个函数和变量——只有通常调用的少数命令才能开始使用包。Emacs 然后对包中的每个 Lisp 文件进行字节编译。

安装后，已安装的包被加载：Emacs 将包的内容目录添加到 load-path，并评估 name-autoloads.el 中的自动加载定义。

每当 Emacs 启动时，它会自动调用函数 package-activate-all 以使已安装的包可用于当前会话。这是在加载早期初始化文件之后但在加载常规初始化文件之前完成的（请参阅摘要：启动时的操作序列）。如果在 early init 文件中将用户选项 package-enable-at-startup 设置为  `nil` ，则不会自动使包可用。

    Function: package-activate-all ¶

此功能使包可用于当前会话。用户选项 package-load-list 指定哪些包可用；  默认情况下，所有已安装的软件包都可用。请参阅 GNU Emacs 手册中的软件包安装。

在大多数情况下，您不需要调用 package-activate-all，因为这是在启动期间自动完成的。只需确保将任何应该在 package-activate-all 之前运行的代码放在早期的 init 文件中，并将任何应该在它之后运行的代码放在主 init 文件中（参见 GNU Emacs 手册中的 Init 文件）。

    Command: package-initialize &optional no-activate ¶

该函数初始化 Emacs 内部记录安装了哪些包，然后调用 package-activate-all。

可选参数 no-activate，如果非  `nil` ，会导致 Emacs 更新其已安装包的记录，但实际上并不使它们可用。


<a id="org50606ae"></a>

## 42.2 简单包

一个简单的包由一个 Emacs Lisp 源文件组成。该文件必须符合 Emacs Lisp 库头文件约定（请参阅 Emacs 库的约定头文件）。包的属性取自各种标头，如下例所示：

    ;;; superfrobnicator.el --- Frobnicate and bifurcate flanges
    
    ;; Copyright (C) 2011 Free Software Foundation, Inc.
    
    
    ;; Author: J. R. Hacker <jrh@example.com>
    ;; Version: 1.3
    ;; Package-Requires: ((flange "1.0"))
    ;; Keywords: multimedia, hypermedia
    ;; URL: https://example.com/jrhacker/superfrobnicate
    
    …
    
    ;;; Commentary:
    
    ;; This package provides a minor mode to frobnicate and/or
    ;; bifurcate any flanges you desire.  To activate it, just type
    …
    
    ;;;###autoload
    (define-minor-mode superfrobnicator-mode
      …

包的名称与文件的基本名称相同，如第一行所写。在这里，它是'superfrobnicator'。

简要说明也取自第一行。在这里，它是 `Frobnicate 和分叉法兰` 。

版本号来自 `Package-Version` 标头（如果存在），否则来自 `Version` 标头。一个或另一个必须存在。这里，版本号是 1.3。

如果文件有 ';;;  Commentary:' 部分，此部分用作长描述。（当显示描述时，Emacs 省略了 ';;; Commentary:' 行，以及注释本身中的前导注释字符。）

如果文件具有 `Package-Requires` 标头，则将其用作包依赖项。在上面的示例中，包依赖于 'flange' 包，版本 1.0 或更高版本。有关 `Package-Requires` 标头的描述，请参阅 Emacs 库的常规标头。如果省略标头，则包没有依赖项。

'Keywords' 和 'URL' 标头是可选的，但建议使用。命令 describe-package 使用这些将链接添加到其输出。 `关键字` 标题应包含至少一个来自 finder-known-keywords 列表的标准关键字。

该文件还应该包含一个或多个自动加载魔术注释，如 Packaging Basics 中所述。在上面的示例中，魔术注释会自动加载 superfrobnicator-mode。

有关如何将单文件包添加到包存档的说明，请参阅创建和维护包存档。


<a id="org8bddab4"></a>

## 42.3 多文件包

创建多文件包不如创建单文件包方便，但它提供了更多功能：它可以包含多个 Emacs Lisp 文件、一个 Info 手册和其他文件类型（如图像）。

在安装之前，多文件包作为 tar 文件存储在包存档中。tar 文件必须命名为 name-version.tar，其中 name 是包名，version 是版本号。它的内容一旦被提取，必须全部出现在名为 name-version 的目录中，即内容目录（参见 Packaging Basics）。文件也可以提取到内容目录的子目录中。

内容目录中的文件之一必须命名为 name-pkg.el。它必须包含一个单一的 Lisp 形式，包括对函数 define-package 的调用，如下所述。这定义了包的属性：版本、简要描述和要求。

例如，如果我们将 1.3 版的 superfrobnicator 分发为多文件包，则 tar 文件将为 superfrobnicator-1.3.tar。它的内容将提取到目录 superfrobnicator-1.3 中，其中之一是文件 superfrobnicator-pkg.el。

    Function: define-package name version &optional docstring requirements ¶

这个函数定义了一个包。name 是包名，一个字符串。version 是版本，作为一个可以被函数 version-to-list 理解的形式的字符串。docstring 是简要说明。

requirements 是所需软件包及其版本的列表。此列表中的每个元素都应具有 (dep-name dep-version) 形式，其中 dep-name 是一个符号，其名称是依赖项的包名称，dep-version 是依赖项的版本（一个字符串）。

如果内容目录包含名为 README 的文件，则该文件用作长描述（覆盖任何 ';;; Commentary:' 部分）。

如果内容目录包含一个名为 dir 的文件，则假定这是一个使用 install-info 创建的 Info 目录文件。请参阅在 Texinfo 中调用 install-info。相关的信息文件也应该存在于内容目录中。在这种情况下，Emacs 会在激活包时自动将内容目录添加到 Info-directory-list 中。

不要在包中包含任何 .elc 文件。这些是在安装软件包时创建的。请注意，无法控制文件字节编译的顺序。

不要包含任何名为 name-autoloads.el 的文件。该文件是为包的自动加载定义保留的（参见 Packaging Basics）。它是在安装包时自动创建的，方法是在包中的所有 Lisp 文件中搜索自动加载魔术注释。

如果多文件包包含辅助数据文件（例如图像），则包的 Lisp 代码可以通过变量 load-file-name 引用这些文件（请参阅加载）。这是一个例子：

    (defconst superfrobnicator-base (file-name-directory load-file-name))
    
    (defun superfrobnicator-fetch-image (file)
      (expand-file-name file superfrobnicator-base))


<a id="orgcc8312d"></a>

## 42.4 创建和维护包档案

通过包菜单，用户可以从包档案中下载包。此类档案由变量 package-archives 指定，其默认值列出了托管在 GNU ELPA 和非 GNU ELPA 上的档案。本节介绍如何设置和维护包存档。

    User Option: package-archives ¶

此变量的值是 Emacs 包管理器识别的包存档列表。

每个 alist 元素对应一个档案，并且应该具有格式 (id . location)，其中 id 是档案的名称（一个字符串），而 location 是它的基本位置（一个字符串）。

如果基本位置以 `http:` 或 `https:` 开头，则将其视为 HTTP(S) URL，并通过 HTTP(S) 从该存档下载包（默认 GNU 存档就是这种情况） .

否则，基本位置应该是目录名称。在这种情况下，Emacs 通过普通文件访问从这个归档中检索包。这样的本地档案主要用于测试。

包存档只是一个目录，其中存储了包文件和相关文件。如果您希望存档可通过 HTTP 访问，则此目录必须可供 Web 服务器访问；  请参阅与存档 Web 服务器的接口。

设置和更新包存档的一种便捷方法是通过 package-x 库。这包含在 Emacs 中，但默认情况下不加载；  输入 Mx load-library RET package-x RET 来加载它，或者添加 (require 'package-x) 到你的 init 文件。请参阅 GNU Emacs 手册中的 Lisp 库。

创建存档后，请记住，除非它位于 package-archives 中，否则无法在 Package Menu 界面中访问它。

维护公共包档案需要一定程度的责任。当 Emacs 用户从您的存档安装包时，这些包可能会导致 Emacs 以安装用户的权限运行任意代码。（这对于一般的 Emacs 代码来说是正确的，而不仅仅是对于包。）所以你应该确保你的归档得到很好的维护并保持托管系统的安全。

提高包安全性的一种方法是使用加密密钥对其进行签名。如果您生成了一个私有/公共 gpg 密钥对，您可以使用 gpg 对包进行签名，如下所示：

    gpg -ba -o file.sig file

对于单文件包，file 是包 Lisp 文件；  对于多文件包，它是包 tar 文件。您也可以以相同的方式签署存档的内容文件。使 .sig 文件在与包相同的位置可用。您还应该使您的公钥可供人们下载；  例如，通过将其上传到密钥服务器，例如 <https://pgp.mit.edu/。当人们从您的档案中安装软件包时，他们可以使用您的公钥来验证签名>。

对这些事项的完整解释超出了本手册的范围。有关加密密钥和签名的更多信息，请参阅 The GNU Privacy Guard Manual 中的 GnuPG。Emacs 带有一个到 GNU Privacy Guard 的接口，请参阅 Emacs EasyPG 助手手册中的 EasyPG。


<a id="org9e6dfc7"></a>

## 42.5 与存档 Web 服务器的接口

提供对包存档的访问的 Web 服务器必须支持以下查询：

    archive-contents

返回描述存档内容的 lisp 表单。该表单是一个 `package-desc` 结构的列表（参见 package.el），除了列表的第一个元素是存档版本。

    <package name>-readme.txt

返回包的详细描述。

    <file name>.sig

返回文件的签名。

    <file name>

返回文件。这将是多文件包的 tarball，或简单包的单个文件。

