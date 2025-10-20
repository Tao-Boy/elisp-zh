
# 34 非ASCII字符

本章介绍与字符有关的特殊问题以及它们如何存储在字符串和缓冲区中。


<a id="org473f2ed"></a>

## 34.1 文本表示

Emacs 缓冲区和字符串支持来自许多不同脚本的大量字符，允许用户以几乎任何已知的书面语言键入和显示文本。

为了支持大量字符和脚本，Emacs 严格遵循 Unicode 标准。Unicode 标准为每个字符分配一个唯一编号，称为代码点。由 Unicode 或 Unicode 代码空间定义的代码点范围是 0..#x10FFFF（十六进制表示法），包括 0..#x10FFFF。Emacs 使用 #x110000..#x3FFFFF 范围内的代码点扩展了这个范围，它用于表示与 Unicode 不统一的字符和无法解释为字符的原始 8 位字节。因此，Emacs 中的字符代码点是一个 22 位整数。

为了节省内存，Emacs 不保存固定长度的 22 位数字，这些数字是缓冲区和字符串中文本字符的代码点。相反，Emacs 使用可变长度的字符内部表示，将每个字符存储为 1 到 5 个 8 位字节的序列，具体取决于其代码点的大小19。例如，任何 ASCII 字符只占用 1 个字节，一个 Latin-1 字符占用 2 个字节，等等。我们称这种文本表示为多字节。

在 Emacs 之外，字符可以用许多不同的编码表示，例如 ISO-8859-1、GB-2312、Big-5 等。当 Emacs 将文本读入一个缓冲区或字符串，或者当它将文本写入磁盘文件或将其传递给其他进程时。

有时，Emacs 需要在其缓冲区或字符串中保存和操作编码文本或二进制非文本数据。例如，当 Emacs 访问一个文件时，它首先将文件的文本逐字读取到缓冲区中，然后才将其转换为内部表示。在转换之前，缓冲区保存编码文本。

就 Emacs 而言，编码文本并不是真正的文本，而是一个原始的 8 位字节序列。我们将保存编码文本单字节缓冲区和字符串的缓冲区和字符串称为缓冲区和字符串，因为 Emacs 将它们视为单个字节的序列。通常，Emacs 将单字节缓冲区和字符串显示为八进制代码，例如 \\237。我们建议您不要使用单字节缓冲区和字符串，除非是处理编码文本或二进制非文本数据。

在缓冲区中，变量 enable-multibyte-characters 的缓冲区本地值指定使用的表示。字符串的表示是在构造字符串时确定并记录在字符串中的。

    Variable: enable-multibyte-characters ¶

此变量指定当前缓冲区的文本表示。如果为非零，则缓冲区包含多字节文本；  否则，它包含单字节编码文本或二进制非文本数据。

您不能直接设置此变量；  相反，使用函数 set-buffer-multibyte 来更改缓冲区的表示。

    Function: position-bytes position ¶

缓冲区位置以字符为单位测量。该函数返回当前缓冲区中缓冲区位置位置对应的字节位置。这是缓冲区开始时的 1，并以字节为单位向上计数。如果位置超出范围，则值为  `nil` 。

    Function: byte-to-position byte-position ¶

返回与当前缓冲区中给定字节位置相对应的缓冲区位置（以字符为单位）。如果字节位置超出范围，则值为  `nil` 。在多字节缓冲区中，字节位置的任意值可以不在字符边界，而是在表示单个字符的多字节序列内；在这种情况下，该函数返回多字节序列中包含字节位置的字符的缓冲区位置。换句话说，对于属于同一字符的所有字节位置，该值不会改变。

当 Lisp 程序需要将缓冲区位置映射到缓冲区访问的文件中的字节偏移量时，以下两个函数很有用。

    Function: bufferpos-to-filepos position &optional quality coding-system ¶

此函数类似于 position-bytes，但它不是返回当前缓冲区中的字节位置，而是返回与缓冲区中给定字符位置相对应的字节从当前缓冲区文件开头的偏移量。转换需要知道文本在缓冲区文件中的编码方式；这就是 coding-system 参数的用途，默认为 buffer-file-coding-system 的值。可选参数 quality 指定结果应该有多准确；它应该是以下之一：

    exact

结果必须准确。该函数可能需要对大部分缓冲区进行编码和解码，这很昂贵并且可能很慢。

    approximate

该值可以是近似值。该函数可以避免昂贵的处理并返回不精确的结果。

    nil

如果确切的结果需要昂贵的处理，该函数将返回  `nil`  而不是近似值。如果参数被省略，这是默认值。

    Function: filepos-to-bufferpos byte &optional quality coding-system ¶

此函数返回与 byte 指定的文件位置相对应的缓冲区位置，该位置是从文件开头的零基字节偏移量。该函数执行与 bufferpos-to-filepos 所做的相反的转换。可选参数 quality 和 coding-system 与 bufferpos-to-filepos 具有相同的含义和值。

    Function: multibyte-string-p string ¶

如果 string 是多字节字符串，则返回 t，否则返回  `nil` 。如果 string 是字符串以外的某个对象，此函数也返回  `nil` 。

    Function: string-bytes string ¶

此函数返回字符串中的字节数。如果 string 是多字节字符串，则可以大于 (length string)。

    Function: unibyte-string &rest bytes ¶

此函数连接其所有参数字节并使结果成为单字节字符串。

脚注
(19)

此内部表示基于 Unicode 标准定义的编码之一，称为 UTF-8，用于表示任何 Unicode 代码点，但 Emacs 扩展了 UTF-8 以表示它用于原始 8 位字节和未统一字符的附加代码点与 Unicode。


<a id="orgccdf771"></a>

## 34.2 禁用多字节字符

默认情况下，Emacs 以多字节模式启动：它使用内部编码存储缓冲区和字符串的内容，该内部编码使用多字节序列表示非 ASCII 字符。多字节模式允许您不受限制地使用所有支持的语言和脚本。

在非常特殊的情况下，您可能希望禁用特定缓冲区的多字节字符支持。当缓冲区中禁用多字节字符时，我们称之为单字节模式。在单字节模式下，缓冲区中的每个字符都有一个字符代码，范围从 0 到 255（八进制 0377）；0 到 127（八进制 0177）代表 ASCII 字符，128（八进制 0200）到 255（八进制 0377）代表非 ASCII 字符。

要以单字节表示编辑特定文件，请使用 find-file-literally 访问它。请参阅访问文件的函数。您可以通过将多字节缓冲区保存到文件、终止缓冲区并再次使用 find-file-literally 访问文件来将多字节缓冲区转换为单字节。或者，您可以使用 Cx RET c（通用编码系统参数）并指定 `原始文本` 作为访问或保存文件的编码系统。请参阅 GNU Emacs 手册中的为文件文本指定编码系统。与 find-file-literally 不同，以 `原始文本` 形式查找文件不会禁用格式转换、解压缩或自动模式选择。

缓冲区局部变量 enable-multibyte-characters 在多字节缓冲区中为非  `nil` ，在单字节缓冲区中为  `nil` 。模式行还指示缓冲区是否为多字节。对于图形显示，在多字节缓冲区中，模式行中指示字符集的部分有一个工具提示（除其他外），说明该缓冲区是多字节的。在单字节缓冲区中，不存在字符集指示符。因此，在单字节缓冲区中（当使用图形显示时），在访问文件的行尾约定（冒号、反斜杠等）的指示之前通常没有任何内容，除非您使用的是输入法。

您可以通过在该缓冲区中调用命令 toggle-enable-multibyte-characters 来关闭特定缓冲区中的多字节支持。


<a id="orgfc8cf8b"></a>

## 34.3 转换文本表示

Emacs 可以将单字节文本转换为多字节；它还可以将多字节文本转换为单字节，前提是多字节文本仅包含 ASCII 和 8 位原始字节。通常，这些转换发生在将文本插入缓冲区或将多个字符串中的文本放在一个字符串中时。您还可以将字符串的内容显式转换为任一表示形式。

Emacs 根据构造字符串的文本选择字符串的表示形式。一般规则是在将单字节文本与其他多字节文本组合时将其转换为多字节文本，因为多字节表示更通用并且可以容纳单字节文本具有的任何字符。

将文本插入缓冲区时，Emacs 将文本转换为缓冲区的表示形式，由该缓冲区中的 enable-multibyte-characters 指定。特别是，当您将多字节文本插入单字节缓冲区时，Emacs 会将文本转换为单字节，即使这种转换通常不能保留多字节文本中可能存在的所有字符。另一种自然的选择是将缓冲区内容转换为多字节，这是不可接受的，因为缓冲区的表示是用户做出的无法自动覆盖的选择。

将单字节文本转换为多字节文本保持 ASCII 字符不变，并将代码 128 到 255 的字节转换为原始 8 位字节的多字节表示。

将多字节文本转换为单字节会将所有 ASCII 和 8 位字符转换为其单字节形式，但会丢失非 ASCII 字符的信息，因为它会丢弃每个字符代码点的低 8 位以外的所有信息。将单字节文本转换为多字节并转换回单字节会再现原始单字节文本。

接下来的两个函数要么返回参数字符串，要么返回一个新创建的没有文本属性的字符串。

    Function: string-to-multibyte string ¶

此函数返回一个多字节字符串，其中包含与字符串相同的字符序列。如果 string 是多字节字符串，则原样返回。该函数假定字符串仅包含 ASCII 字符和原始 8 位字节；后者被转换为对应于代码点 #x3FFF80 到 #x3FFFFF 的多字节表示，包括（参见代码点）。

    Function: string-to-unibyte string ¶

此函数返回一个包含与字符串相同的字符序列的单字节字符串。如果字符串包含非 ASCII 字符，它会发出错误信号。如果 string 是单字节字符串，则原样返回。将此函数用于仅包含 ASCII 和八位字符的字符串参数。

    Function: byte-to-string byte ¶

此函数返回一个单字节字符串，其中包含单个字节的字符数据字节。如果 byte 不是 0 到 255 之间的整数，则会发出错误信号。

    Function: multibyte-char-to-unibyte char ¶

这会将多字节字符 char 转换为单字节字符，并返回该字符。如果 char 既不是 ASCII 也不是八位，则函数返回 -1。

    Function: unibyte-char-to-multibyte char ¶

这会将单字节字符 char 转换为多字节字符，假设 char 是 ASCII 或原始 8 位字节。


<a id="orgce76778"></a>

## 34.4 选择表示

有时，将现有缓冲区或字符串检查为多字节（当它是单字节时）很有用，反之亦然。

    Function: set-buffer-multibyte multibyte ¶

设置当前缓冲区的表示类型。如果多字节为非零，则缓冲区变为多字节。如果多字节为  `nil` ，则缓冲区变为单字节。

当被视为字节序列时，此函数使缓冲区内容保持不变。因此，它可以改变被视为字符的内容；例如，在多字节表示中被视为一个字符的三个字节序列在单字节表示中将计为三个字符。表示原始字节的八位字符是一个例外。它们由单字节缓冲区中的一个字节表示，但是当缓冲区设置为多字节时，它们将转换为两字节序列，反之亦然。

此函数设置 enable-multibyte-characters 以记录正在使用的表示形式。它还调整缓冲区中的各种数据（包括覆盖、文本属性和标记），以便它们覆盖与以前相同的文本。

如果缓冲区变窄，此函数会发出错误信号，因为变窄可能发生在多字节字符序列的中间。

如果缓冲区是间接缓冲区，此函数也会发出错误信号。间接缓冲区始终继承其基本缓冲区的表示。

    Function: string-as-unibyte string ¶

如果 string 已经是单字节字符串，则此函数返回 string 本身。否则，它返回一个与字符串具有相同字节的新字符串，但将每个字节视为一个单独的字符（这样该值可能包含比字符串更多的字符）；作为一个例外，代表原始字节的每个八位字符都被转换为单个字节。新创建的字符串不包含文本属性。

    Function: string-as-multibyte string ¶

如果 string 是多字节字符串，则此函数返回 string 本身。否则，它返回一个与字符串具有相同字节的新字符串，但将每个多字节序列视为一个字符。这意味着该值的字符数可能少于字符串的字符数。如果字符串中的字节序列作为单个字符的多字节表示无效，则序列中的每个字节都被视为原始 8 位字节。新创建的字符串不包含文本属性。


<a id="org20c5fd5"></a>

## 34.5 字符代码

单字节和多字节文本表示使用不同的字符代码。单字节表示的有效字符代码范围从 0 到 #xFF (255) — 可以容纳在一个字节中的值。多字节表示的有效字符代码范围从 0 到 #x3FFFFF。在此代码空间中，值 0 到 #x7F (127) 用于 ASCII 字符，值 #x80 (128) 到 #x3FFF7F (4194175) 用于非 ASCII 字符。

Emacs 字符代码是 Unicode 标准的超集。值 0 到 #x10FFFF (1114111) 对应于同一代码点的 Unicode 字符；值 #x110000 (1114112) 到 #x3FFF7F (4194175) 表示未与 Unicode 统一的字符；并且值 #x3FFF80 (4194176) 到 #x3FFFFF (4194303) 表示 8 位原始字节。

    Function: characterp charcode ¶

如果 charcode 是有效字符，则返回 t，否则返回  `nil` 。

    
    
    (characterp 65)
         ⇒ t
    
    (characterp 4194303)
         ⇒ t
    
    (characterp 4194304)
         ⇒ nil

    Function: max-char ¶

此函数返回有效字符代码点可以具有的最大值。

    
    
    (characterp (max-char))
         ⇒ t
    
    (characterp (1+ (max-char)))
         ⇒ nil

    Function: char-from-name string &optional ignore-case ¶

该函数返回 Unicode 名称为字符串的字符。如果 ignore-case 不为零，则字符串中的大小写将被忽略。如果字符串没有命名字符，则此函数返回  `nil` 。

    ;; U+03A3
    (= (char-from-name "GREEK CAPITAL LETTER SIGMA") #x03A3)
         ⇒ t

    Function: get-byte &optional pos string ¶

此函数返回当前缓冲区中字符位置 pos 处的字节。如果当前缓冲区是单字节的，那么这就是该位置的字节。如果缓冲区是多字节的，则 ASCII 字符的字节值与字符代码点相同，而 8 位原始字节将转换为它们的 8 位代码。如果 pos 处的字符不是 ASCII，则该函数会发出错误信号。

可选参数字符串意味着从该字符串而不是当前缓冲区中获取一个字节值。


<a id="org6ba37a9"></a>

## 34.6 字符属性

字符属性是字符的命名属性，它指定字符的行为方式以及在文本处理和显示期间应如何处理。因此，字符属性是指定字符语义的重要部分。

总的来说，Emacs 在字符属性的实现上遵循 Unicode 标准。特别是，Emacs 支持 Unicode Character Property Model，而 Emacs 字符属性数据库是从 Unicode Character Database (UCD) 衍生而来的。有关 Unicode 字符属性及其含义的详细说明，请参阅 Unicode 标准的字符属性一章。本节假定您已经熟悉 Unicode 标准的那一章，并希望将这些知识应用到 Emacs Lisp 程序中。

在 Emacs 中，每个属性都有一个名称，它是一个符号，以及一组可能的值，其类型取决于属性；如果一个字符没有特定属性，则值为  `nil` 。作为一般规则，Emacs 中字符属性的名称是从相应的 Unicode 属性生成的，方法是将它们向下转换并用破折号 '-' 替换每个 '\_' 字符。例如，Canonical<sub>Combining</sub><sub>Class</sub> 变为 canonical-combining-class。但是，有时我们会缩短名称以使其更易于使用。

一些代码点未被 UCD 分配——它们不对应于任何字符。Unicode 标准为此类代码点定义了属性的默认值；它们在下面针对每个属性进行了提及。

这是 Emacs 知道的所有字符属性的值类型的完整列表：

    name

对应于 Name Unicode 属性。该值是一个字符串，由大写拉丁字母 A 到 Z、数字、空格和连字符 `-` 字符组成。对于未分配的代码点，该值为  `nil` 。

    general-category

对应于 General<sub>Category</sub> Unicode 属性。该值是一个符号，其名称是字符分类的 2 个字母缩写。对于未分配的代码点，该值为 Cn。

    canonical-combining-class

对应于 Canonical<sub>Combining</sub><sub>Class</sub> Unicode 属性。该值是一个整数。对于未分配的代码点，该值为零。

    bidi-class

对应于 Unicode Bidi<sub>Class</sub> 属性。该值是一个符号，其名称是字符的 Unicode 方向类型。Emacs 在重新排序双向文本以进行显示时使用此属性（请参阅双向显示）。对于未分配的代码点，该值取决于代码点所属的代码块：大多数未分配的代码点获得 L（强 L）的值，但有些获得 AL（阿拉伯字母）或 R（强 R）的值。

    decomposition

对应于 Unicode 属性 Decomposition<sub>Type</sub> 和 Decomposition<sub>Value</sub>。该值是一个列表，其第一个元素可以是表示兼容性格式标记的符号，例如small20；其他元素是给出这个字符的兼容性分解序列的字符。对于没有分解序列的字符和未分配的代码点，该值是具有单个成员的列表，即字符本身。

    decimal-digit-value

对应于 Numeric<sub>Type</sub> 为 `十进制` 的字符的 Unicode Numeric<sub>Value</sub> 属性。该值是一个整数，如果字符没有十进制数字值，则为  `nil` 。对于未分配的代码点，该值为  `nil` ，表示 NaN，或 `不是数字` 。

    digit-value

对应于 Numeric<sub>Type</sub> 为 `Digit` 的字符的 Unicode Numeric<sub>Value</sub> 属性。该值是一个整数。此类字符的示例包括兼容性下标和上标数字，其值为对应的数字。对于没有任何数值的字符和未分配的代码点，该值为  `nil` ，表示 NaN。

    numeric-value

对应于 Numeric<sub>Type</sub> 为 `Numeric` 的字符的 Unicode Numeric<sub>Value</sub> 属性。此属性的值是一个数字。具有此属性的字符示例包括分数、下标、上标、罗马数字、货币分子和带圆圈的数字。例如，字符 U+2155 VULGAR FRACTION ONE FIFTH 的此属性的值为 0.2。对于没有任何数值的字符和未分配的代码点，该值为  `nil` ，表示 NaN。

    mirrored

对应于 Unicode Bidi<sub>Mirrored</sub> 属性。此属性的值是一个符号，可以是 Y 或 N。对于未分配的代码点，该值为 N。

    mirroring

对应于 Unicode Bidi<sub>Mirroring</sub><sub>Glyph</sub> 属性。该属性的值是一个字符，其字形表示该字符字形的镜像，如果没有定义镜像字形，则为  `nil` 。所有镜像属性为 N 的字符都以  `nil`  作为其镜像属性；然而，一些镜像属性为 Y 的字符也有  `nil`  用于镜像，因为镜像字形不存在合适的字符。Emacs 使用此属性在适当的时候显示字符的镜像（请参阅双向显示）。对于未分配的代码点，该值为  `nil` 。

    paired-bracket

对应于 Unicode Bidi<sub>Paired</sub><sub>Bracket</sub> 属性。此属性的值是字符配对括号的代码点，如果字符不是括号字符，则为  `nil` 。这在被 Unicode 双向算法视为括号对的字符之间建立了映射；Emacs 在决定如何重新排序显示括号、大括号和其他类似字符时使用此属性（请参阅双向显示）。

    bracket-type

对应于 Unicode Bidi<sub>Paired</sub><sub>Bracket</sub><sub>Type</sub> 属性。对于双括号属性为非  `nil`  的字符，此属性的值是一个符号，o（用于左括号字符）或 c（用于右括号字符）。对于双括号属性为  `nil`  的字符，其值为符号 n（无）。与双括号一样，此属性用于双向显示。

    old-name

对应于 Unicode Unicode<sub>1</sub><sub>Name</sub> 属性。该值是一个字符串。对于未分配的代码点和对此属性没有值的字符，该值为  `nil` 。

    iso-10646-comment

对应于 Unicode ISO<sub>Comment</sub> 属性。该值是字符串或  `nil` 。对于未分配的代码点，该值为  `nil` 。

    uppercase

对应于 Unicode Simple<sub>Uppercase</sub><sub>Mapping</sub> 属性。此属性的值是单个字符。对于未分配的代码点，该值为  `nil` ，表示字符本身。

    lowercase

对应于 Unicode Simple<sub>Lowercase</sub><sub>Mapping</sub> 属性。此属性的值是单个字符。对于未分配的代码点，该值为  `nil` ，表示字符本身。

    titlecase

对应于 Unicode Simple<sub>Titlecase</sub><sub>Mapping</sub> 属性。标题大小写是当单词的第一个字符需要大写时使用的一种特殊形式的字符。此属性的值是单个字符。对于未分配的代码点，该值为  `nil` ，表示字符本身。

    special-uppercase

对应于 Unicode 语言和上下文无关的特殊大写规则。此属性的值是一个字符串（可能为空）。例如，U+00DF LATIN SMALL LETTER SHARP S 的映射是 `SS` 。对于没有特殊映射的字符，该值为  `nil` ，这意味着需要查询大写属性。

    special-lowercase

对应于 Unicode 语言和上下文无关的特殊小写规则。此属性的值是一个字符串（可能为空）。例如，U+0130 LATIN CAPITAL LETTER I WITH DOT ABOVE 的映射值为 `i\u0307` （即由拉丁小写字母 I 后跟 U+0307 COMBINING DOT ABOVE 组成的 2 个字符的字符串）。对于没有特殊映射的字符，该值为  `nil` ，这意味着需要查询小写属性。

    special-titlecase

对应于 Unicode 无条件特殊标题大小写规则。此属性的值是一个字符串（可能为空）。例如 U+FB01 LATIN SMALL LIGATURE FI 的映射，其值为 `Fi` 。对于没有特殊映射的字符，该值为  `nil` ，这意味着需要查询 titlecase 属性。

    Function: get-char-code-property char propname ¶

此函数返回 char 的 propname 属性的值。

    
    
    (get-char-code-property ?\s 'general-category)
         ⇒ Zs
    
    (get-char-code-property ?1 'general-category)
         ⇒ Nd
    
    ;; U+2084
    (get-char-code-property ?\N{SUBSCRIPT FOUR}
    			'digit-value)
         ⇒ 4
    
    ;; U+2155
    (get-char-code-property ?\N{VULGAR FRACTION ONE FIFTH}
    			'numeric-value)
         ⇒ 0.2
    
    ;; U+2163
    (get-char-code-property ?\N{ROMAN NUMERAL FOUR}
    			'numeric-value)
         ⇒ 4
    
    (get-char-code-property ?\( 'paired-bracket)
         ⇒ 41  ; closing parenthesis
    
    (get-char-code-property ?\) 'bracket-type)
         ⇒ c

    Function: char-code-property-description prop value ¶

此函数返回属性 prop 的值的描述字符串，如果 value 没有描述，则返回  `nil` 。

    
    
    (char-code-property-description 'general-category 'Zs)
         ⇒ "Separator, Space"
    
    (char-code-property-description 'general-category 'Nd)
         ⇒ "Number, Decimal Digit"
    
    (char-code-property-description 'numeric-value '1/5)
         ⇒ nil

    Function: put-char-code-property char propname value ¶

此函数将 value 存储为字符 char 的属性 propname 的值。

    Variable: unicode-category-table ¶

此变量的值是一个字符表（请参阅 Char-Tables），它为每个字符指定其 Unicode General<sub>Category</sub> 属性作为符号。

    Variable: char-script-table ¶

该变量的值是一个字符表，它为每个字符指定一个符号，其名称是该字符所属的脚本，根据 Unicode 标准将 Unicode 代码空间分类为特定于脚本的块。这个字符表有一个额外的槽，其值是所有脚本符号的列表。请注意，Emacs 将字符分类为脚本并不是 Unicode 标准的一对一反映，例如，Unicode 中没有 `符号` 脚本。

    Variable: char-width-table ¶

这个变量的值是一个字符表，它指定每个字符在屏幕上占据的列的宽度。

    Variable: printable-chars ¶

这个变量的值是一个字符表，它为每个字符指定它是否可打印。也就是说，如果计算 (aref printable-chars char) 结果为 t，则该字符是可打印的，如果结果为  `nil` ，则不是。

脚注
(20)

Unicode 规范将这些标签名称写在 '<..>' 括号内，但 Emacs 中的标签名称不包括括号；例如，Unicode 指定 '<small>'，而 Emacs 使用 'small'。


<a id="org06a668e"></a>

## 34.7 字符集

Emacs 字符集或 charset 是一组字符，其中每个字符都分配有一个数字代码点。（Unicode 标准称之为编码字符集。）每个 Emacs 字符集都有一个名称，它是一个符号。单个字符可以属于任意数量的不同字符集，但它通常在每个字符集中具有不同的代码点。字符集的示例包括 ascii、iso-8859-1、greek-iso8859-7 和 windows-1255。分配给字符集中字符的代码点通常不同于其在 Emacs 缓冲区和字符串中使用的代码点。

Emacs 定义了几个特殊字符集。字符集 unicode 包括 Emacs 代码点在 0..#x10FFFF 范围内的所有字符。字符集 emacs 包括所有 ASCII 和非 ASCII 字符。最后，8 位字符集包括 8 位原始字节；Emacs 使用它来表示文本中遇到的原始字节。

    Function: charsetp object ¶

如果 object 是命名字符集的符号，则返回 t，否则返回  `nil` 。

    Variable: charset-list ¶

该值是所有已定义字符集名称的列表。

    Function: charset-priority-list &optional highestp ¶

此函数返回按优先级排序的所有已定义字符集的列表。如果highestp 不为 `nil` ，则函数返回最高优先级的单个字符集。

    Function: set-charset-priority &rest charsets ¶

此函数使字符集成为最高优先级的字符集。

    Function: char-charset character &optional restriction ¶

该函数返回该字符所属的最高优先级字符集的名称。ASCII 字符是一个例外：对于它们，此函数始终返回 ascii。

如果限制不为零，则它应该是要搜索的字符集列表。或者，它可以是编码系统，在这种情况下，返回的字符集必须由该编码系统支持（请参阅编码系统）。

    Function: charset-plist charset ¶

该函数返回字符集字符集的属性列表。虽然 charset 是一个符号，但这与该符号的属性列表不同。字符集属性包括有关字符集的重要信息，例如其文档字符串、短名称等。

    Function: put-charset-property charset propname value ¶

此函数将 charset 的 propname 属性设置为给定值。

    Function: get-charset-property charset propname ¶

此函数返回 charsets 属性 propname 的值。

    Command: list-charset-chars charset ¶

此命令显示字符集 charset 中的字符列表。

Emacs 可以在字符的内部表示和特定字符集中的字符代码点之间进行转换。以下两个函数支持这些转换。

    Function: decode-char charset code-point ¶

此函数将在 charset 中分配了代码点的字符解码为相应的 Emacs 字符，并返回它。如果 charset 不包含该代码点的字符，则值为  `nil` 。

为了向后兼容，如果代码点不适合 Lisp fixnum（请参阅 most-positive-fixnum），可以将其指定为 cons 单元格（high . low），其中 low 是值的低 16 位， high 是高 16 位。这种用法已经过时了。

    Function: encode-char char charset ¶

此函数返回分配给 charset 中字符 char 的代码点。如果 charset 没有 char 的代码点，则值为  `nil` 。

以下函数可用于将某个函数应用于 charset 中的全部或部分字符：

    Function: map-charset-chars function charset &optional arg from-code to-code ¶

字符集中字符的调用函数。使用两个参数调用函数。第一个是一个 cons 单元格（从 .to），其中 from 和 to 表示 charset 中包含的字符范围。传递给函数的第二个参数是 arg，如果省略 arg，则为  `nil` 。

默认情况下，传递给函数的代码点范围包括 charset 中的所有字符，但可选参数 from-code 和 to-code 将其限制为这两个 charset 代码点之间的字符范围。如果其中任何一个为  `nil` ，则分别默认为 charset 的第一个或最后一个代码点。注意 from-code 和 to-code 是 charset 的代码点，而不是 Emacs 的字符代码；相反，传递给函数的 cons 单元格中的 from 和 to 值是 Emacs 字符代码。这些 Emacs 字符代码要么是 Unicode 代码点，要么是扩展 Unicode 并且超出 Unicode 字符范围 0..#x10FFFF 的 Emacs 内部代码点（请参阅文本表示）。后者很少发生，传统的 CJK 字符集用于字符集的代码点，这些字符集指定尚未与 Unicode 统一的字符。


<a id="org4502682"></a>

## 34.8 扫描字符集

有时找出特定字符属于哪个字符集很有用。这样做的一个用途是确定哪些编码系统（参见编码系统）能够表示所有相关文本；另一个是确定显示该文本的字体。

    Function: charset-after &optional pos ¶

此函数返回最高优先级的字符集，其中包含当前缓冲区中位置 pos 处的字符。如果 pos 被省略或为零，则默认为 point 的当前值。如果 pos 超出范围，则值为  `nil` 。

    Function: find-charset-region beg end &optional translation ¶

此函数返回最高优先级的字符集列表，其中包含当前缓冲区中位置 beg 和 end 之间的字符。

可选参数 translation 指定用于扫描文本的转换表（请参阅字符转换）。如果它不为  `nil` ，则区域中的每个字符都通过此表进行翻译，返回的值描述了翻译后的字符，而不是缓冲区中实际存在的字符。

    Function: find-charset-string string &optional translation ¶

此函数返回包含字符串中字符的最高优先级字符集列表。它就像 find-charset-region 一样，只是它适用于字符串的内容而不是当前缓冲区的一部分。


<a id="org2a86aad"></a>

## 34.9 字符翻译

转换表是一个字符表（参见 Char-Tables），它指定了字符到字符的映射。这些表用于编码和解码，以及用于其他目的。一些编码系统指定了自己的特定翻译表；还有适用于所有其他编码系统的默认翻译表。

一个翻译表有两个额外的插槽。第一个是  `nil`  或执行反向翻译的翻译表；第二个是查找翻译字符序列的最大字符数（请参阅下面的 make-translation-table-from-alist 的描述）。

    Function: make-translation-table &rest translations ¶

此函数根据参数翻译返回一个翻译表。翻译的每个元素都应该是表单元素的列表（从.到）；这表示将字符从转换为到。

每个参数中的参数和形式按顺序处理，如果先前的形式已经转换为其他字符，例如 to-alt，from 也将转换为 to-alt。

在解码期间，翻译表的翻译应用于普通解码产生的字符。如果编码系统具有属性 :decode-translation-table，它指定要使用的转换表，或按顺序应用的转换表列表。（这是编码系统的属性，由 coding-system-get 返回，而不是作为编码系统名称的符号的属性。请参阅编码系统的基本概念。）最后，如果 standard-translation-table-for -decode 不为零，结果字符由该表翻译。

在编码过程中，翻译表的翻译应用于缓冲区中的字符，翻译的结果实际上是编码的。如果编码系统具有属性 :encode-translation-table，则指定要使用的翻译表，或者要按顺序应用的翻译表列表。此外，如果变量standard-translation-table-for-encode 不为 `nil` ，它指定用于翻译结果的翻译表。

    Variable: standard-translation-table-for-decode ¶

这是解码的默认转换表。如果编码系统指定了它自己的转换表，那么作为该变量值的表（如果非零）将应用在它们之后。

    Variable: standard-translation-table-for-encode ¶

这是编码的默认转换表。如果编码系统指定了它自己的转换表，那么作为该变量值的表（如果非零）将应用在它们之后。

    Variable: translation-table-for-input ¶

自插入字符在插入之前通过此翻译表进行翻译。搜索命令还通过此表转换其输入，因此它们可以更可靠地与缓冲区中的内容进行比较。

此变量在设置时自动变为缓冲区本地。

    Function: make-translation-table-from-vector vec ¶

此函数返回一个由 vec 制成的转换表，它是一个包含 256 个元素的数组，用于将字节（值 0 到 #xFF）映射到字符。对于未翻译的字节，元素可能为零。返回的表在第一个额外槽中有一个用于反向映射的转换表，在第二个额外槽中具有值 1。

此函数提供了一种简单的方法来创建将每个字节映射到特定字符的私有编码系统。您可以使用属性 :decode-translation-table 和 :encode-translation-table 分别在 define-coding-system 的 props 参数中指定返回表和反向转换表。

    Function: make-translation-table-from-alist alist ¶

此函数类似于 make-translation-table 但返回一个复杂的转换表而不是简单的一对一映射。alist 的每个元素都采用 (from . to) 形式，其中 from 和 to 是指定字符序列的字符或向量。如果 from 是一个字符，则该字符被转换为 to（即，转换为一个字符或一个字符序列）。如果 from 是字符向量，则将该序列转换为 to。返回的表在第一个额外槽中有一个用于反向映射的转换表，以及第二个额外槽中所有 from 字符序列的最大长度。


<a id="org107be85"></a>

## 34.10 编码系统

当 Emacs 读取或写入文件时，以及当 Emacs 向子进程发送文本或从子进程接收文本时，它通常会执行特定编码系统指定的字符代码转换和行尾转换。

如何定义编码系统是一个神秘的问题，这里没有记录。


<a id="org92925ed"></a>

### 34.10.1 编码系统的基本概念

字符代码转换涉及在 Emacs 中使用的字符的内部表示与其他一些编码之间的转换。Emacs 支持许多不同的编码，因为它可以相互转换。例如，它可以将文本转换为拉丁语 1、拉丁语 2、拉丁语 3、拉丁语 4、拉丁语 5 和 ISO 2022 的几种变体等编码或从编码转换。在某些情况下，Emacs 支持相同字符的多种替代编码；例如，西里尔字母（俄语）有三种编码系统：ISO、Alternativnyj 和 KOI8。

每个编码系统都指定了一组特定的字符代码转换，但未决定的编码系统是特殊的：它没有指定选择，而是在解码或编码时根据文件或字符串的数据为每个文件或字符串进行启发式选择.  编码系统prefer-utf-8 就像未定，但它更喜欢尽可能选择utf-8。

一般来说，编码系统不能保证往返身份：使用编码系统对字节序列进行解码，然后在同一编码系统中对生成的文本进行编码，可以产生不同的字节序列。但是一些编码系统确实保证字节序列与您最初解码的相同。这里有一些例子：

iso-8859-1、utf-8、big5、shift<sub>jis</sub>、euc-jp

编码缓冲区文本然后解码结果也可能无法重现原始文本。例如，如果您使用不支持该字符的编码系统对字符进行编码，则结果是不可预测的，因此使用相同的编码系统对其进行解码可能会产生不同的文本。目前，Emacs 无法报告因编码不受支持的字符而导致的错误。

行尾转换处理各种系统上用于表示文件中行尾的三种不同约定。用于 GNU 和 Unix 系统的 Unix 约定是使用换行符（也称为换行符）。在 MS-Windows 和 MS-DOS 系统上使用的 DOS 约定是在行尾使用回车符和换行符。Mac 的惯例是只使用回车。（这是 Classic Mac OS 中使用的约定。）

诸如 latin-1 之类的基本编码系统未指定行尾转换，以根据数据进行选择。诸如 latin-1-unix、latin-1-dos 和 latin-1-mac 等变体编码系统也明确指定了行尾转换。大多数基本编码系统都有三个相应的变体，它们的名称是通过添加 `-unix` 、 `-dos` 和 `-mac` 形成的。

编码系统原始文本的特殊之处在于它阻止了字符代码转换，并使使用该编码系统访问的缓冲区成为单字节缓冲区。由于历史原因，您可以使用此编码系统保存单字节和多字节文本。当您使用原始文本对多字节文本进行编码时，它会执行一种字符代码转换：它将八位字符转换为其单字节外部表示。raw-text 不指定行尾转换，允许像往常一样由数据确定，并且具有指定行尾转换的通常的三个变体。

no-conversion（及其别名二进制）等价于 raw-text-unix：它指定不转换字符代码或行尾。

编码系统 utf-8-emacs 指定数据以内部 Emacs 编码表示（请参阅文本表示）。这就像原始文本一样，没有发生代码转换，但不同之处在于结果是多字节数据。名称 emacs-internal 是 utf-8-emacs-unix 的别名（因此它不强制转换行尾，不像 utf-8-emacs 可以解码所有 3 种行尾约定） .

    Function: coding-system-get coding-system property ¶

该函数返回编码系统编码系统的指定属性。大多数编码系统属性都是出于内部目的而存在的，但您可能会发现一个有用的属性是 :mime-charset。该属性的值是 MIME 中用于该编码系统可以读写的字符编码的名称。例子：

    (coding-system-get 'iso-latin-1 :mime-charset)
         ⇒ iso-8859-1
    (coding-system-get 'iso-2022-cn :mime-charset)
         ⇒ iso-2022-cn
    (coding-system-get 'cyrillic-koi8 :mime-charset)
         ⇒ koi8-r

:mime-charset 属性的值也被定义为编码系统的别名。

    Function: coding-system-aliases coding-system ¶

该函数返回编码系统的别名列表。


<a id="org16b09dc"></a>

### 34.10.2 编码和 I/O

编码系统的主要目的是用于读取和写入文件。函数 insert-file-contents 使用编码系统对文件数据进行解码，而 write-region 使用编码系统对缓冲区内容进行编码。

您可以指定要显式使用的编码系统（请参阅为一个操作指定编码系统），或隐式使用默认机制（请参阅默认编码系统）。但这些方法可能无法完全指定要做什么。例如，他们可以选择一种编码系统，例如未决定的，这使得字符代码转换由数据来确定。在这些情况下，I/O 操作完成了选择编码系统的工作。很多时候你会想知道选择了哪种编码系统。

    Variable: buffer-file-coding-system ¶

这个缓冲区局部变量记录了用于保存缓冲区和用 write-region 写入部分缓冲区的编码系统。如果无法使用此变量指定的编码系统对要写入的文本进行安全编码，则这些操作会通过调用函数 select-safe-coding-system 来选择替代编码（请参阅用户选择的编码系统）。如果选择不同的编码需要要求用户指定编码系统，则将缓冲区文件编码系统更新为新选择的编码系统。

buffer-file-coding-system 不会影响将文本发送到子进程。

    Variable: save-buffer-coding-system ¶

此变量指定用于保存缓冲区的编码系统（通过覆盖缓冲区文件编码系统）。请注意，它不用于写入区域。

当保存缓冲区的命令开始使用缓冲区文件编码系统（或保存缓冲区编码系统）时，该编码系统无法处理缓冲区中的实际文本，该命令要求用户选择另一个编码系统（通过调用 select-safe-coding-system）。之后，该命令还会更新 buffer-file-coding-system 以表示用户指定的编码系统。

    Variable: last-coding-system-used ¶

文件和子进程的 I/O 操作将此变量设置为所使用的编码系统名称。显式编码和解码功能（请参阅显式编码和解码）也设置了它。

警告：由于接收子进程输出设置了这个变量，它可以在 Emacs 等待时改变；因此，您应该在存储您感兴趣的值的函数调用之后立即复制该值。

变量 selection-coding-system 指定如何对窗口系统的选择进行编码。请参阅窗口系统选择。

    Variable: file-name-coding-system ¶

变量 file-name-coding-system 指定用于编码文件名的编码系统。Emacs 使用该编码系统对所有文件操作的文件名进行编码。如果 file-name-coding-system 为  `nil` ，则 Emacs 使用由所选语言环境确定的默认编码系统。在默认语言环境下，文件名中的任何非 ASCII 字符都不会进行特殊编码；它们使用内部 Emacs 表示出现在文件系统中。

警告：如果您在 Emacs 会话中更改文件名编码系统（或语言环境），如果您已经访问过名称使用早期编码系统编码的文件，并且在新的编码系统。如果您尝试将这些缓冲区之一保存在访问的文件名下，则保存可能会使用错误的文件名，或者可能会出错。如果发生此类问题，请使用 Cx Cw 为该缓冲区指定新文件名。

在 Windows 2000 及更高版本上，Emacs 默认使用 Unicode API 将文件名传递给操作系统，因此 file-name-coding-system 的值在很大程度上被忽略了。当系统类型为 windows-nt 时，需要在 Lisp 级别对文件名进行编码或解码的 Lisp 应用程序应使用 utf-8 编码系统；将 UTF-8 编码的文件名转换为适合与操作系统通信的编码是由 Emacs 内部执行的。


<a id="org5f74144"></a>

### 34.10.3 Lisp 中的编码系统

以下是使用编码系统的 Lisp 工具：

    Function: coding-system-list &optional base-only ¶

此函数返回所有编码系统名称（符号）的列表。如果 base-only 不为零，则该值仅包括基本编码系统。否则，它还包括别名和变体编码系统。

    Function: coding-system-p object ¶

如果 object 是编码系统名称或  `nil` ，则此函数返回 t。

    Function: check-coding-system coding-system ¶

此功能检查编码系统的有效性。如果有效，则返回编码系统。如果 coding-system 为  `nil` ，则函数返回  `nil` 。对于任何其他值，它表示一个错误，其错误符号是编码系统错误（见信号）。

    Function: coding-system-eol-type coding-system ¶

此函数返回编码系统使用的行尾（又名 eol）转换类型。如果coding-system指定了某种eol转换，则返回值为整数0、1、2，分别代表unix、dos、mac。如果 coding-system 没有明确指定 eol 转换，则返回值是编码系统的向量，每个编码系统都有一种可能的 eol 转换类型，如下所示：

    (coding-system-eol-type 'latin-1)
         ⇒ [latin-1-unix latin-1-dos latin-1-mac]

如果这个函数返回一个向量，作为文本编码或解码过程的一部分，Ema​​cs 将决定使用什么 eol 转换。对于解码，自动检测文本的行尾格式，并设置 eol 转换以匹配它（例如，DOS 样式的 CRLF 格式将暗示 dos eol 转换）。对于编码，eol 转换取自适当的默认编码系统（例如，缓冲区文件编码系统的缓冲区文件编码系统的默认值），或来自适用于底层平台的默认 eol 转换。

    Function: coding-system-change-eol-conversion coding-system eol-type ¶

该函数返回一个编码系统，除了它的 eol 转换由 eol-type 指定外，它类似于coding-system。eol-type 应该是 unix、dos、mac 或  `nil` 。如果为  `nil` ，则返回的编码系统确定数据的行尾转换。

eol-type 也可以是 0、1 或 2，分别代表 unix、dos 和 mac。

    Function: coding-system-change-text-conversion eol-coding text-coding ¶

该函数返回使用eol-coding 的行尾转换和text-coding 的文本转换的编码系统。如果 text-coding 为  `nil` ，则根据 eol-coding 返回未决定的或其变体之一。

    Function: find-coding-systems-region from to ¶

此函数返回一个编码系统列表，可用于对 from 和 to 之间的文本进行编码。列表中的所有编码系统都可以安全地编码该部分文本中的任何多字节字符。

如果文本不包含多字节字符，则函数返回列表（未确定）。

    Function: find-coding-systems-string string ¶

此函数返回可用于对字符串文本进行编码的编码系统列表。列表中的所有编码系统都可以安全地编码字符串中的任何多字节字符。如果文本不包含多字节字符，则返回列表（未确定）。

    Function: find-coding-systems-for-charsets charsets ¶

此函数返回可用于对列表字符集中的所有字符集进行编码的编码系统列表。

    Function: check-coding-systems-region start end coding-system-list ¶

该函数检查列表 coding-system-list 中的编码系统是否可以对 start 和 end 之间区域中的所有字符进行编码。如果列表中的所有编码系统都可以对指定文本进行编码，则该函数返回  `nil` 。如果某些编码系统无法对某些字符进行编码，则该值为 alist，其中每个元素的形式为 (coding-system1 pos1 pos2 &#x2026;)，这意味着 coding-system1 无法对缓冲区位置 pos1、pos2、..的字符进行编码。 ..

start 可能是一个字符串，在这种情况下 end 被忽略并且返回值引用字符串索引而不是缓冲区位置。

    Function: detect-coding-region start end &optional highest ¶

此函数选择一个合理的编码系统来从头到尾对文本进行解码。该文本应该是一个字节序列，即只有 ASCII 和八位字符的单字节文本或多字节文本（参见显式编码和解码）。

通常，此函数返回可以处理对扫描文本进行解码的编码系统列表。它们按优先级降序排列。但是如果最高是非零，则返回值只是一种编码系统，即优先级最高的一种。

如果该区域仅包含 ASCII 字符，除了诸如 ESC 等 ISO-2022 控制字符 ISO-2022 之外，该值是未确定的或（未确定的），或者指定行尾转换的变体，如果可以从文本中推断出的话。

如果该区域包含空字节，则该值为无转换，即使该区域包含在某些编码系统中编码的文本。

    Function: detect-coding-string string &optional highest ¶

此函数类似于检测编码区域，只是它对字符串的内容而不是缓冲区中的字节进行操作。

    Variable: inhibit-null-byte-detection ¶

如果此变量具有非零值，则在检测区域或字符串的编码时会忽略空字节。这允许正确检测包含空字节的文本编码，例如具有索引节点的信息文件。

    Variable: inhibit-iso-escape-detection ¶

如果此变量具有非零值，则在检测区域或字符串的编码时会忽略 ISO-2022 转义序列。结果是从未检测到任何文本以某种 ISO-2022 编码进行编码，并且所有转义序列在缓冲区中变得可见。警告：使用此变量时要格外小心，因为 Emacs 发行版中的许多文件都使用 ISO-2022 编码。

    Function: coding-system-charset-list coding-system ¶

此函数返回编码系统支持的字符集列表（请参阅字符集）。一些支持太多字符集以列出它们的编码系统都会产生特殊值：

如果 coding-system 支持所有 Emacs 字符，则值为 (emacs)。
如果coding-system 支持所有Unicode 字符，则值为(unicode)。
如果 coding-system 支持所有 ISO-2022 字符集，则值为 iso-2022。
如果 coding-system 支持 Emacs 版本 21 使用的内部编码系统中的所有字符（在实现内部 Unicode 支持之前），则值为 emacs-mule。

请参阅进程信息，特别是对函数 process-coding-system 和 set-process-coding-system 的描述，了解如何检查或设置用于子进程 I/O 的编码系统。


<a id="org6925787"></a>

### 34.10.4 用户选择的编码系统

    Function: select-safe-coding-system from to &optional default-coding-system accept-default-p file ¶

这个函数选择一个编码系统来对指定的文本进行编码，如果需要的话要求用户选择。通常，指定的文本是当前缓冲区中 from 和 to 之间的文本。如果 from 是一个字符串，则该字符串指定要编码的文本，to 被忽略。

如果指定的文本包含原始字节（请参阅文本表示），则 select-safe-coding-system 建议使用原始文本进行编码。

如果 default-coding-system 是非零，那就是第一个尝试的编码系统；如果可以处理文本，则 select-safe-coding-system 返回该编码系统。它也可以是编码系统的列表；然后该函数一一尝试它们中的每一个。在尝试了所有这些之后，它接下来尝试当前缓冲区的缓冲区文件编码系统的值（如果它不是未决定的），然后是缓冲区文件编码系统的默认值，最后是用户最喜欢的编码系统，用户可以使用命令 prefer-coding-system 进行设置（参见 GNU Emacs 手册中的识别编码系统）。

如果其中一个编码系统可以安全地编码所有指定的文本，则 select-safe-coding-system 选择它并返回它。否则，它会要求用户从可以对所有文本进行编码的编码系统列表中进行选择，并返回用户的选择。

default-coding-system 也可以是一个列表，其第一个元素是 t，其他元素是编码系统。然后，如果列表中没有编码系统可以处理文本，则 select-safe-coding-system 会立即查询用户，而无需尝试上述三种替代方法中的任何一种。这对于仅检查列表中的编码系统很方便。

可选参数 accept-default-p 确定在没有用户交互的情况下选择的编码系统是否可接受。如果它被省略或为零，那么这种静默选择总是可以接受的。如果它是非零，它应该是一个函数；select-safe-coding-system 使用一个参数调用此函数，即所选编码系统的基本编码系统。如果函数返回  `nil` ，则 select-safe-coding-system 拒绝静默选择的编码系统，并要求用户从可能的候选列表中选择一个编码系统。

如果变量 select-safe-coding-system-accept-default-p 不为零，它应该是一个采用单个参数的函数。它用于代替accept-default-p，覆盖为此参数提供的任何值。

作为最后一步，在返回所选编码系统之前，select-safe-coding-system 检查该编码系统是否与从文件中读取区域内容时将选择的编码系统一致。（如果不是，这可能会导致随后重新访问和编辑的文件中的数据损坏。）通常， select-safe-coding-system 为此目的使用缓冲区文件名作为文件，但如果文件是非零，它使用该文件代替（这可能与写入区域和类似功能相关）。如果它检测到明显的不一致，则 select-safe-coding-system 在选择编码系统之前会询问用户。

    Variable: select-safe-coding-system-function ¶

当输出操作的默认编码系统无法安全地编码该文本时，此变量命名要调用的函数以请求用户选择适当的编码系统来编码文本。此变量的默认值为 select-safe-coding-system。将文本写入文件（例如 write-region）或将文本发送到其他进程（例如 process-send-region）的 Emacs 原语通常调用此变量的值，除非coding-system-for-write 绑定到非- `nil`  值（请参阅为一项操作指定编码系统）。

这里有两个函数可以用来让用户指定一个编码系统，并完成。请参阅完成。

    Function: read-coding-system prompt &optional default ¶

此函数使用 minibuffer 读取编码系统，以字符串提示进行提示，并将编码系统名称作为符号返回。如果用户输入空输入，默认指定返回哪个编码系统。它应该是一个符号或字符串。

    Function: read-non-nil-coding-system prompt ¶

此函数使用 minibuffer 读取编码系统，以字符串提示进行提示，并将编码系统名称作为符号返回。如果用户尝试输入空输入，它会要求用户再试一次。请参阅编码系统。


<a id="orgc239f7b"></a>

### 34.10.5 默认编码系统

本节描述了为某些文件或运行某些子程序时指定默认编码系统的变量，以及 I/O 操作用来访问它们的函数。

这些变量的想法是您将它们一劳永逸地设置为您想要的默认值，然后不再更改它们。要为 Lisp 程序中的特定操作指定特定的编码系统，请不要更改这些变量；相反，使用 coding-system-for-read 和 coding-system-for-write 覆盖它们（请参阅为一个操作指定编码系统）。

    User Option: auto-coding-regexp-alist ¶

该变量是文本模式和相应编码系统的列表。每个元素都有形式 (regexp . coding-system);  前几千字节与正则表达式匹配的文件在其内容被读入缓冲区时使用编码系统进行解码。此 alist 中的设置优先于编码：文件中的标签和 file-coding-system-alist 的内容（见下文）。设置默认值是为了让 Emacs 自动识别 Babyl 格式的邮件文件并在不进行代码转换的情况下读取它们。

    User Option: file-coding-system-alist ¶

此变量是一个列表，它指定用于读取和写入特定文件的编码系统。每个元素都有格式（pattern .coding），其中pattern 是匹配特定文件名的正则表达式。该元素适用于匹配模式的文件名。

元素的 CDR，coding，应该是一个编码系统，一个包含两个编码系统的 cons 单元，或者一个函数名（一个带有函数定义的符号）。如果编码是一个编码系统，则该编码系统用于读取文件和写入文件。如果coding是一个包含两个编码系统的cons cell，它的CAR指定解码的编码系统，它的CDR指定编码的编码系统。

如果 coding 是一个函数名，该函数应该有一个参数，即传递给 find-operation-coding-system 的所有参数的列表。它必须返回一个编码系统或一个包含两个编码系统的 cons 单元。该值与上述含义相同。

如果编码（或上述函数返回的内容）未确定，则执行正常的代码检测。

    User Option: auto-coding-alist ¶

此变量是一个列表，它指定用于读取和写入特定文件的编码系统。它的形式类似于 file-coding-system-alist，但与后者不同的是，此变量优先于文件中的任何编码：标签。

    Variable: process-coding-system-alist ¶

此变量是一个列表，指定子进程使用哪些编码系统，具体取决于子进程中运行的程序。它的工作方式类似于 file-coding-system-alist，除了该模式与用于启动子进程的程序名称相匹配。此列表中指定的编码系统用于初始化用于子进程 I/O 的编码系统，但您可以稍后使用 set-process-coding-system 指定其他编码系统。

警告：编码系统，如 undecided，根据数据确定编码系统，不能完全可靠地处理异步子进程输出。这是因为 Emacs 在异步子进程输出到达时分批处理它。如果编码系统未指定字符代码转换，或未指定行尾转换，Emacs 必须尝试一次从一批中检测正确的转换，但这并不总是有效。

因此，对于异步子进程，如果可能的话，请使用既确定字符代码转换又确定行尾转换的编码系统——即类似于 latin-1-unix 的编码系统，而不是未定或 latin-1。

    Variable: network-coding-system-alist ¶

此变量是一个列表，指定用于网络流的编码系统。它的工作原理很像 file-coding-system-alist，不同之处在于元素中的模式可以是端口号或正则表达式。如果是正则表达式，则匹配用于打开网络流的网络服务名称。

    Variable: default-process-coding-system ¶

此变量指定用于子进程（和网络流）输入和输出的编码系统，而没有其他指定要做什么。

该值应该是形式的 cons 单元格（输入编码。输出编码）。这里输入编码适用于子流程的输入，输出编码适用于它的输出。

    User Option: auto-coding-functions ¶

该变量包含一个函数列表，这些函数试图根据文件的未解码内容确定文件的编码系统。

此列表中的每个函数都应编写为查看当前缓冲区中的文本，但不应以任何方式修改它。缓冲区将包含文件部分的文本。每个函数都应该有一个参数 size，它告诉它从 point 开始要查看多少个字符。如果函数成功确定文件的编码系统，它应该返回该编码系统。否则，它应该返回  `nil` 。

当文件被访问并且 Emacs 想要解码其内容时，和/或当文件的缓冲区即将被保存并且 Emacs 想要确定如何对其内容进行编码时，可以调用此列表中的函数。

如果一个文件有一个 'coding:' 标记，它优先，所以这些函数不会被调用。

    Function: find-auto-coding filename size ¶

此函数尝试为文件名确定合适的编码系统。它检查访问指定文件的缓冲区，按顺序使用上面记录的变量，直到找到这些变量指定的规则之一的匹配项。然后它返回一个形式为 (coding.source) 的 cons 单元格，其中 coding 是要使用的编码系统， source 是一个符号，是 auto-coding-alist、auto-coding-regexp-alist、:coding 或 auto 之一-coding-functions，指明是哪一个提供了匹配规则。值 :coding 表示编码系统由文件中的 coding: 标记指定（参见 GNU Emacs 手册中的编码标记）。查找匹配规则的顺序是先auto-coding-alist，然后是auto-coding-regexp-alist，然后是coding: tag，最后是auto-coding-functions。如果没有找到匹配的规则，该函数返回  `nil` 。

第二个参数 size 是文本的大小，以字符为单位，如下点。该函数仅检查点后大小字符内的文本。正常情况下，调用这个函数时缓冲区应该定位在开头，因为编码的地方之一： tag 是文件的前一两行；在这种情况下， size 应该是缓冲区的大小。

    Function: set-auto-coding filename size ¶

此函数为文件文件名返回一个合适的编码系统。它使用 find-auto-coding 来查找编码系统。如果无法确定编码系统，则函数返回  `nil` 。参数大小的含义类似于 find-auto-coding。

    Function: find-operation-coding-system operation &rest arguments ¶

此函数返回编码系统以使用（默认情况下）执行带参数的操作。该值具有以下形式：

    (decoding-system . encoding-system)

第一个元素，解码系统，是用于解码的编码系统（如果操作进行解码），编码系统是用于编码的编码系统（如果操作进行编码）。

参数操作是一个符号；它应该是 write-region、start-process、call-process、call-process-region、insert-file-contents 或 open-network-stream 之一。这些是可以进行字符代码和 eol 转换的 Emacs I/O 原语的名称。

剩余的参数应该与可能提供给相应 I/O 原语的参数相同。根据原语，选择这些参数之一作为目标。例如，如果操作执行文件 I/O，则指定文件名的任何一个参数都是目标。对于子流程原语，流程名称是目标。对于 open-network-stream，目标是服务名称或端口号。

根据操作，此函数在 file-coding-system-alist、process-coding-system-alist 或 network-coding-system-alist 中查找目标。如果在 alist 中找到目标，则 find-operation-coding-system 返回其在 alist 中的关联；否则返回零。

如果 operation 是 insert-file-contents，则对应于目标的参数可能是形式的 cons 单元格 (filename.buffer)。在这种情况下，filename 是要在 file-coding-system-alist 中查找的文件名，而 buffer 是包含文件内容（尚未解码）的缓冲区。如果 file-coding-system-alist 指定了一个函数来调用这个文件，并且该函数需要检查文件的内容（就像它通常那样），它应该检查缓冲区的内容而不是读取文件。


<a id="org36a89d3"></a>

### 34.10.6 为一个操作指定编码系统

您可以通过绑定变量 coding-system-for-read 和/或 coding-system-for-write 来指定特定操作的编码系统。

    Variable: coding-system-for-read ¶

如果此变量不为  `nil` ，则它指定用于读取文件或来自同步子进程的输入的编码系统。

它也适用于任何异步子进程或网络流，但方式不同：当您启动子进程或打开网络流时，coding-system-for-read 的值指定该子进程或网络流的输入解码方法。除非被覆盖，否则它会一直用于该子进程或网络流。

使用此变量的正确方法是将其与 let 绑定以进行特定的 I/O 操作。它的全局值通常为  `nil` ，您不应将其全局设置为任何其他值。这是使用变量的正确方法的示例：

    ;; Read the file with no character code conversion.
    (let ((coding-system-for-read 'no-conversion))
      (insert-file-contents filename))

当其值为非零时，此变量优先于所有其他指定用于输入的编码系统的方法，包括 file-coding-system-alist、process-coding-system-alist 和 network-coding-system-alist .

    Variable: coding-system-for-write ¶

这很像coding-system-for-read，除了它适用于输出而不是输入。它影响对文件的写入，以及将输出发送到子进程和网络连接。它也适用于编码 Emacs 调用子进程的命令行参数。

当单个操作同时进行输入和输出时，就像 call-process-region 和 start-process 一样，coding-system-for-read 和 coding-system-for-write 都会影响它。

    Variable: coding-system-require-warning ¶

将 coding-system-for-write 绑定到非  `nil`  值可防止输出原语调用 select-safe-coding-system-function 指定的函数（请参阅用户选择的编码系统）。这是因为 Cx RET c (universal-coding-system-argument) 通过绑定 coding-system-for-write 工作，Emacs 应该服从用户选择。如果 Lisp 程序将 coding-system-for-write 绑定到对要写入的文本进行编码可能不安全的值，它还可以将 coding-system-require-warning 绑定到非  `nil`  值，这将强制输出原语以通过调用 select-safe-coding-system-function 的值来检查编码，即使 coding-system-for-write 不为零。或者，在使用指定的编码之前显式调用 select-safe-coding-system。

    User Option: inhibit-eol-conversion ¶

当此变量为非零时，无论指定哪种编码系统，都不会进行行尾转换。这适用于所有 Emacs I/O 和子进程原语，以及显式编码和解码功能（请参阅显式编码和解码）。

有时，您需要为某些操作选择多个编码系统，而不是修复一个。Emacs 允许您指定使用编码系统的优先顺序。这种排序会影响由 find-coding-systems-region 等函数返回的编码系统列表的排序（请参阅 Lisp 中的编码系统）。

    Function: coding-system-priority-list &optional highestp ¶

此函数按当前优先级的顺序返回编码系统列表。可选参数highestp，如果非零，表示只返回最高优先级的编码系统。

    Function: set-coding-system-priority &rest coding-systems ¶

此功能将编码系统置于编码系统优先级列表的开头，从而使其优先级高于其他所有系统。

    Macro: with-coding-priority coding-systems &rest body ¶

这个宏执行主体，就像 progn 一样（参见 progn），编码系统位于编码系统的优先级列表的前面。encoding-systems 应该是在执行主体期间首选的编码系统列表。


<a id="org4186dbb"></a>

### 34.10.7 显式编码和解码

所有将文本传入和传出 Emacs 的操作都能够使用编码系统对文本进行编码或解码。您还可以使用本节中的函数显式编码和解码文本。

编码的结果和解码的输入，不是普通的文本。它们在逻辑上由一系列字节值组成；即一系列 ASCII 和八位字符。在单字节缓冲区和字符串中，这些字符的代码范围为 0 到 #xFF (255)。在多字节缓冲区或字符串中，八位字符的字符代码高于 #xFF（请参阅文本表示），但是当您对此类文本进行编码或解码时，Emacs 会透明地将它们转换为单字节值。

将文件作为字节序列读入缓冲区的常用方法是使用 insert-file-contents-literally （请参阅从文件中读取）；或者，在使用 find-file-noselect 访问文件时指定非  `nil`  原始文件参数。这些方法产生一个单字节缓冲区。

使用显式编码文本产生的字节序列的常用方法是将其复制到文件或进程中——例如，使用 write-region 写入它（请参阅写入文件），并通过绑定 coding-system- 来抑制编码for-write 到 no-conversion。

以下是执行显式编码或解码的函数。编码函数产生字节序列；解码功能旨在对字节序列进行操作。所有这些函数都会丢弃文本属性。他们还将 last-coding-system-used 设置为他们使用的精确编码系统。

    Command: encode-coding-region start end coding-system &optional destination ¶

该命令根据编码系统编码系统从头到尾对文本进行编码。通常，编码文本会替换缓冲区中的原始文本，但可选参数目标可以改变它。如果destination 是一个缓冲区，则将编码文本插入到该缓冲区中的点之后（点不移动）；如果是 t，则该命令将编码文本作为单字节字符串返回，而不插入它。

如果将编码文本插入某个缓冲区，则此命令返回编码文本的长度。

编码的结果在逻辑上是一个字节序列，但如果缓冲区之前是多字节，则缓冲区仍然是多字节的，并且任何 8 位字节都将转换为它们的多字节表示（参见文本表示）。

编码文本时不要使用 undecided 作为编码系统，因为这可能会导致意想不到的结果。相反，如果编码系统没有明显的相关价值，请使用 select-safe-coding-system（请参阅 select-safe-coding-system）建议合适的编码。

    Function: encode-coding-string string coding-system &optional nocopy buffer ¶

该函数根据编码系统编码系统对字符串中的文本进行编码。它返回一个包含编码文本的新字符串，除非 nocopy 为非零，在这种情况下，如果编码操作很简单，函数可能会返回字符串本身。编码的结果是一个单字节字符串。

    Command: decode-coding-region start end coding-system &optional destination ¶

该命令根据编码系统编码系统从头到尾对文本进行解码。为了使显式解码有用，解码前的文本应该是字节值序列，但多字节和单字节缓冲区都是可接受的（在多字节情况下，原始字节值应表示为八位字符）。通常，解码后的文本会替换缓冲区中的原始文本，但可选参数目标可以改变这一点。如果destination是一个缓冲区，则解码后的文本将插入到该缓冲区中的点之后（点不移动）；如果是 t，则命令将解码后的文本作为多字节字符串返回，而不插入它。

如果解码文本插入某个缓冲区，则此命令返回解码文本的长度。如果该缓冲区是单字节缓冲区（请参阅选择表示），则解码文本的内部表示（请参阅文本表示）作为单个字节插入缓冲区。

此命令将字符集文本属性放在解码的文本上。该属性的值表示用于解码原始文本的字符集。

如有必要，此命令会检测文本的编码。如果 encoding-system 未确定，该命令会根据它在文本中找到的字节序列检测文本的编码，并且还会检测文本使用的行尾约定的类型（参见 eol 类型）。如果 coding-system 是 undecided-eol-type，其中 eol-type 是 unix、dos 或 mac，则该命令仅检测文本的编码。任何未指定 eol-type 的编码系统，如 utf-8，都会导致命令检测行尾约定；完全指定编码，如在 utf-8-unix 中，如果预先知道文本使用的 EOL 约定，以防止任何自动检测。

    Function: decode-coding-string string coding-system &optional nocopy buffer ¶

该函数根据编码系统对字符串中的文本进行解码。它返回一个包含解码文本的新字符串，除非 nocopy 为非零，在这种情况下，如果解码操作很简单，函数可能会返回字符串本身。为了使显式解码有用，字符串的内容应该是具有一系列字节值的单字节字符串，但多字节字符串也是可以接受的（假设它包含多字节形式的 8 位字节）。

如果需要，此函数会检测字符串的编码，就像 decode-coding-region 一样。

如果可选参数 buffer 指定一个缓冲区，则解码后的文本将插入到该缓冲区中的点之后（点不移动）。在这种情况下，返回值是解码文本的长度。如果该缓冲区是单字节缓冲区，则解码文本的内部表示将作为单个字节插入其中。

此函数在解码后的文本上放置一个字符集文本属性。该属性的值表示用于解码原始文本的字符集：

    (decode-coding-string "Gr\374ss Gott" 'latin-1)
         ⇒ #("Grüss Gott" 0 9 (charset iso-8859-1))

    Function: decode-coding-inserted-region from to filename &optional visit beg end replace ¶

此函数将文本从 from 解码到 to ，就好像它是使用 insert-file-contents 使用提供的其余参数从文件文件名中读取的一样。

使用此功能的正常方法是从文件中读取文本而不进行解码，如果您决定更愿意对其进行解码。而不是删除文本并再次阅读它，这次解码，您可以调用此函数。


<a id="org573a644"></a>

### 34.10.8 终端 I/O 编码

Emacs 可以使用编码系统来解码键盘输入和编码终端输出。这对于使用特定编码（例如 Latin-1）传输或显示文本的终端很有用。Emacs 在编码或解码终端 I/O 时没有设置 last-coding-system-used。

    Function: keyboard-coding-system &optional terminal ¶

此函数返回用于解码终端键盘输入的编码系统。no-conversion 值表示不进行解码。如果终端省略或为零，则表示所选帧的终端。请参阅多个终端。

    Command: set-keyboard-coding-system coding-system &optional terminal ¶

该命令将 coding-system 指定为用于解码来自终端的键盘输入的编码系统。如果 coding-system 为  `nil` ，则意味着不对键盘输入进行解码。如果终端是一个框架，则表示该框架的终端；如果为  `nil` ，则表示当前选择的帧的终端。请参阅多个终端。请注意，在现代 MS-Windows 系统上，Emacs 在解码键盘输入时始终使用 Unicode 输入，因此该命令设置的编码对 Windows 没有影响。

    Function: terminal-coding-system &optional terminal ¶

此函数返回用于对终端的终端输出进行编码的编码系统。no-conversion 值表示不进行编码。如果终端是一个框架，则表示该框架的终端；如果为  `nil` ，则表示当前选择的帧的终端。

    Command: set-terminal-coding-system coding-system &optional terminal ¶

此命令将 coding-system 指定为用于对终端输出进行编码的编码系统。如果 coding-system 为  `nil` ，则意味着不对终端输出进行编码。如果终端是一个框架，则表示该框架的终端；如果为  `nil` ，则表示当前选择的帧的终端。


<a id="orge0495ba"></a>

## 34.11 输入法

输入法提供了从键盘输入非 ASCII 字符的便捷方式。与将非 ASCII 字符与程序读取的编码进行转换的编码系统不同，输入法提供了人性化的命令。（有关用户如何使用输入法输入文本的信息，请参阅 GNU Emacs 手册中的输入法。）本手册中尚未记录如何定义输入法，但我们在此描述如何使用它们。

每个输入法都有一个名字，目前是一个字符串；将来，符号也可以用作输入法名称。

    Variable: current-input-method ¶

此变量保存当前缓冲区中现在处于活动状态的输入法的名称。（当以任何方式设置时，它会在每个缓冲区中自动变为本地。）如果现在缓冲区中没有输入法处于活动状态，则为  `nil` 。

    User Option: default-input-method ¶

此变量保存选择输入法的命令的默认输入法。与 current-input-method 不同，此变量通常是全局的。

    Command: set-input-method input-method ¶

此命令激活当前缓冲区的输入法输入法。它还将默认输入方法设置为输入方法。如果 input-method 为  `nil` ，则此命令停用当前缓冲区的任何输入法。

    Function: read-input-method-name prompt &optional default inhibit-null ¶

该函数使用 minibuffer 读取输入法名称，以 prompt 提示。如果默认为非零，则默认返回，如果用户输入空输入。但是，如果 inhibitor-null 不为零，则空输入表示错误。

返回值是一个字符串。

    Variable: input-method-alist ¶

这个变量定义了所有支持的输入法。每个元素定义一个输入法，并且应该具有以下形式：

    (input-method language-env activate-func
     title description args...)

这里的input-method是输入法名称，一个字符串；language-env 是另一个字符串，建议使用此输入法的语言环境的名称。（这仅用于文档目的。）

activate-func 是调用以激活此方法的函数。args（如果有）作为参数传递给 activate-func。总而言之，activate-func 的参数是输入方法和参数。

标题是此方法处于活动状态时在模式行中显示的字符串。description 是描述此方法及其用途的字符串。

输入法的基本接口是通过变量输入法函数。请参阅读取一个事件和调用输入法。


<a id="org77c3b4f"></a>

## 34.12 语言环境

在 POSIX 中，语言环境控制在与语言相关的功能中使用哪种语言。这些 Emacs 变量控制 Emacs 如何与这些功能交互。

    Variable: locale-coding-system ¶

此变量指定用于解码系统错误消息和（仅在 X Window 系统上）键盘输入、将批处理输出发送到标准输出和错误流、将格式参数编码为 format-time-string 以及解码格式时间字符串的返回值。

    Variable: system-messages-locale ¶

此变量指定用于生成系统错误消息的语言环境。更改区域设置可能会导致消息以不同的语言或不同的拼写形式出现。如果变量为  `nil` ，则区域设置由环境变量以通常的 POSIX 方式指定。

    Variable: system-time-locale ¶

此变量指定用于格式化时间值的语言环境。更改区域设置可能会导致消息根据不同语言的约定显示。如果变量为  `nil` ，则区域设置由环境变量以通常的 POSIX 方式指定。

    Function: locale-info item ¶

如果可用，此函数返回当前 POSIX 语言环境的语言环境数据项。item 应该是以下符号之一：

    codeset

将字符集作为字符串返回（语言环境项 CODESET）。

    days

返回日期名称的 7 元素向量（区域设置项 DAY<sub>1</sub> 到 DAY<sub>7</sub>）；

    months

返回包含月份名称的 12 元素向量（区域设置项目 MON<sub>1</sub> 到 MON<sub>12</sub>）。

    paper

返回一个包含 2 个整数的列表（宽度高度），默认纸张尺寸以毫米为单位（语言环境项目 \_NL<sub>PAPER</sub><sub>WIDTH</sub> 和 \_NL<sub>PAPER</sub><sub>HEIGHT</sub>）。

如果系统无法提供请求的信息，或者 item 不是这些符号之一，则值为  `nil` 。返回值中的所有字符串都使用语言环境编码系统进行解码。有关语言环境和语言环境项目的更多信息，请参阅 GNU Libc 手册中的语言环境。

