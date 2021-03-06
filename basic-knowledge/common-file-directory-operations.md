本文讲了一些文件和目录本身的基础知识和操作，如新建、复制、移动等，并不涉及文件内容的查看、编辑、搜索等操作。

### 文件归属

文件的归属只有三种关系：

* **所有者u：**即文件的拥有者，并且一个文件同时只能有一个所有者，一般是谁创建的文件，这个文件的所有者就是谁。
* **所属组g：**文件归属的组，并且一个文件同时只能归属于一个组，只有组中的成员才能操作此文件。
* **其他人o：**非前两者关系的用户就是其他人。

### 文件权限

Linux中权限的表示由四部分组成，如-rw-r--r--，第一部分为第一个字符，表示文件类型，常用文件类型有-（二进制文件）、d（目录）和l（软连接文件，相当于Windows中的快捷方式）。接下来每三个字符为一组的三个部分，分别代表该文件所有者u、所属组g和其他人o所拥有的权限，而每个部分的权限由三个字符表示其拥有的r读、w写和x执行权限，如果有该权限则用对应的字母表示，如果没有该权限则用-表示。

目录的rwx权限如下（文件的rwx权限很好理解，但是目录的rwx权限是有些差别的）：

* **r：**可以列出目录中的内容。
* **w：**可以在目录中创建和删除文件。
* **x：**可以进入目录。

### ls命令

**ls \[选项\] \[目录\]：**列出目录下的所有文件及子目录。

选项：

* **-a：**显示目录下的所有内容，包括隐藏文件（Linux中以点“.”开头的文件为隐藏文件）。
* **-l：**显示目录下内容的详细信息，分别为权限、引用系数（相当于引用计数）、文件所有者、文件所属组、文件大小（单位Byte）、文件最后一次修改时间（Linux中没有创建时间的概念）、文件名。
* **-h：**人性化显示，将文件等内容的大小以较为人性化的方式显示，如M、G等，而不是默认的单位字节。
* **-d：**显示当前所在目录或指定目录本身。
* **-i：**显示文件或目录的id号（也称为i节点号）。

### mkdir命令

**mkdir \[-p\] 目录 \[目录1 目录2 ...\]：**用于创建一个或多个空白目录。-p选项是用于递归创建目录。

### cd命令

**cd \[目录\]：**切换到指定目录，目录还以使用一个点“.”表示当前目录，两个点“..”表示上一级目录。

### pwd命令

显示当前目录的绝对路径。

### rmdir命令

**rmdir 目录：**删除一个空目录。

### rm命令

**rm \[-rf\] 文件或目录：**删除文件或目录，不加选项则默认删除文件。

* **-r：**删除目录。
* **-f：**强制执行。

### cp命令

**cp \[-rp\] 一个或多个原文件或目录 目标文件或目录：**复制文件或目录，如果不加选项，则默认复制文件。当目标文件或目录不存在时，相当于复制并重命名。

* **-r：**复制目录。
* **-p：**保留文件属性，如最后一次修改时间等。

### mv命令

**mv 原文件或目录 目标文件或目录：**移动（剪切）文件或目录。当目标文件或目录不存在时，则相当于剪切并重命名。

**文件或目录重命名：**Linux中没有直接的重命名命令，但是一般使用mv来实现文件或目录的重命名，即将文件或目录移动到“原位置”，但是名称却变了，如“mv /tmp/test.txt /tmp/linux\_test.txt”就可以将test.txt重命名为linux\_test.txt。

### touch命令

**touch 一个或多个文件：**创建一个或多个空白文件（多个空白文件使用空格隔开，如果文件名中含有空格，文件名需要使用双引号括起来，但建议不要使用空格来命名文件）。如果文件或目录已经存在，则会根据指定的选项修改文件的时间属性，如最后一次修改时间（这里没有列出相应的选项，需要时可自行查看）。

**注意：**新建的文件是没有执行权限的，所以如果新建的文件是脚本，则需要先赋予它执行权限才能执行这个脚本。

### ln命令

**ln \[-s\] 原文件 目标文件：**生成链接文件（即目标文件，它指向原文件），默认生成硬链接文件。-s选项指定生成软链接文件。

**软链接：**类似Windows中的快捷方式，只是一个指向另一个文件的链接而已，并且Linux中的软链接文件的权限永远都是lrwxrwxrwx，以及大小都是固定的很小的字节数。

**硬链接：**硬链接的信息与原文件的信息都是一样的，并且其中一个文件更新后，硬链接文件也会同步更新，相当于cp -p命令再加上同步跟新的功能。之所以它能同步更新，是因为硬链接的i节点和原文件的i节点是相同的（而Linux就是通过i节点来识别不同的文件）。

**软链接与硬链接的差别（或者说硬链接的特点）：  **

* 硬链接是不能跨分区的。
* 硬链接是不能指向目录的。



### locate命令

**locate -i 文件名：**在文件资料库中查找文件。（默认区分大小写，-i选项表示不区分大小写）

这个命令搜索速度非常快，几乎秒搜，find命令是去硬盘上搜索，而find是在自己维护的一个文件资料库中查找，这个文件资料库会定期自动更新。但是文件资料库是不会收录/tmp目录下的文件的。

**updatedb：**手动更新文件资料库。



### find命令

**find 搜索范围 匹配条件：**文件搜索。

find是直接在硬盘上搜索，所以它的消耗是非常大的，所以使用的时候应该尽量缩小搜索范围，匹配条件也越精确越好。而且最好不要在系统负载较高时使用这个命令。

常用选项：

* **-name：**根据文件名搜索。如“find / -name init”表示在根目录的范围内搜索文件名为init的文件（精确匹配）。如果想要模糊匹配，可以使用通配符，如find / -name \*init\*（星号\*匹配任意字符，问号?匹配单个字符）。
* **-iname：**不区分大小写进行文件查找。
* **-size：**根据文件大小进行搜索，+表示大于，-表示小于，=表示等于。查找大小的单位为一个数据块，Linux中一个数据块的大小为512字节，即0.5KB，所以搜索的时候需要自己转换以下。如“find / -size +204800”表示在根目录下查找大于100MB的文件（100MB=102400KB=204800个数据块）。
* **-user：**根据所有者来进行搜索。
* **-group：**根据所属组来进行搜索。
* **-amin：**根据文件访问时间查找，时间单位为分钟。
* **-cmin：**根据文件属性（属性即ls -l能查看到的内容）的改变时间查找，时间单位为分钟。如“find /etc -cmin -5”表示在/etc目录下查找5分钟内文件属性被修改过的文件。
* **-mmin：**根据文件内容的改变时间查找，时间单位为分钟。
* **-a：**连接选项，逻辑与，表示需要两个条件同时满足。如find /etc -size +1638840 -a -size -204800。
* **-o：**连接选项，逻辑或，表示两个条件满足其中任意一个即可。
* **-type：**根据文件类型查找，f表示文件，d表示目录，l表示软链接。
* **-exec/-ok 命令 {}\;：**使用-exec（直接执行，不会询问）或-ok（执行命令时会进行询问）对查找结果执行某个命令，{}\;是固定的写法，{}表示查找结果，\只是对后面分号;的转义而已。如“find /etc -name init -exec ls -l {}\;”。
* **-inum：**根据i节点查找。



