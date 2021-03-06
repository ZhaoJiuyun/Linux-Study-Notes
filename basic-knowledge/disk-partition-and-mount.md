Linux中的所有设备都必须挂载之后才能使用，即必须给它分配“盘符”（即挂载点，就是系统中的某个空目录）给它才能使用，Windows系统中一般是自动分配盘符，所以挂载的概念在Windows中是没有的，比如你插入了一个U盘，Windows就会自动给它一个盘符（名称）来连接U盘，但是Linux中必须手动挂载某个设备，并给它分配一个挂载点（目录名），即连接设备，挂载（连接）成功后才能使用。

### 一、磁盘分区

**理解分区：**磁盘的各个分区，可以理解为一个柜子的不同抽屉，而设备文件名则代表不同的抽屉，如/dev/sda1表示此分区的设备文件名，通常我们讲某个设备文件，其实就是说它代表的分区或者它代表的硬盘，而不会将它作为一个单纯的文件来看待。其中/dev目录下存放的是对应的硬件设备，sd为设备类型，表示SATA磁盘，a表示磁盘编号，1表示此磁盘的分区号。

**Windows分区：**Windows中磁盘的使用一般是经过分区（将一个大的硬盘分成多个小的逻辑分区），格式化（指定文件系统，而此时会清空磁盘内的数据，注意格式化的目的是重新指定文件系统而不是清空数据），然后给分区指定盘符（如C盘、D盘等），在Linux中，没有盘符的说法，从分区到给此分区指定盘符的过程称之为挂载，而对应的盘符则称之为挂载点，比如上述的/dev/sda1我可以指定它的挂载点为/test，就相当于在Windows中指定了这块分区为test盘（就像C盘、D盘一样）。

**Linux分区：**在Linux中，相比于Windows的三个步骤，磁盘分区这一步会比Windows多一个内容，就是分区时，需要给对应分区指定一个设备文件名，如/dev/sda1、/dev/sda2、/dev/sda3等，表示磁盘/dev/sda下的各个分区由不同的文件来管理，然后接下来也是格式化和分配挂载点（即盘符，就是对应的目录名）。

**Linux常用分区：**

* **/（根分区）：**此为必须分区，即必须给这个目录一个分区。
* **swap分区（交换分区）：**这也是一个必须分区，大小为内存的2倍，但是也不能超过2GB。此分区可以理解为虚拟内存，即内存不够时，可以使用此分区作为内存使用。
* **/boot：**系统启动目录，建议此目录单独分一个区作为启动分区（就像Windows中的C盘为系统启动盘一样），一般为200MB。

Linux的文件系统虽然是由根目录到一级目录，然后到二级目录，然后一直往下扩散延伸（不像Windows那样C盘和D盘等盘是平级的），但是目录中的某一个空目录是可以单独拿出来给它分区的，比如根目录下的/boot通常就单独分一个区出来作为系统启动运行的专用分区，如/dev/sda1（对应的设备文件），而根目录/则使用另外一个分区/dev/sda2（对应的设备文件）。

### 二、手动分区新硬盘

注意使用命令手动挂载的方式，在系统重启之后就会失效，想要挂载永久生效，还需要将对应的挂载配置写入/etc/fstab文件（见之后的“分区自动挂载”）。以下步骤按顺序执行。

**fdisk -l：**查看硬盘信息。每个硬盘都会单独显示一个“Disk”，然后在下面列出已分区的“Device”信息，如果列出的信息只有“Disk”而没有对应的“Device”信息，则表示该硬盘还没有进行分区。

**fdisk /dev/sdb：**给硬盘分区，参数/dev/sdb就是上一步查询出来的硬盘设备的名称（注意不要加编号，只有分区之后系统才会自动分配编号，没有分区之前是没有编号的）。执行此命令后会要求按顺序执行以下的命令：

* **m：**表示查看分区命令的帮助信息（这一步一般不用）。
* **n：**新建一个分区。
* **p：**新建一个主分区。（显示的提示为p primary partition \(1-4\)。）
* **1：**设置分区号（建议根据已存在分区号从低到高按顺序指定，此时显示的提示为Partition number \(1-4\)）。
* **【回车】：**指定从哪个柱面开始分区，默认从第一个。建议就从第1个开始，不要从其他位置指定，此时显示的提示为First cylinder \(1-1305, default 1\)，表示此硬盘有1305个柱面可用于分区，默认从第1个柱面开始分区。
* **【回车】：**指定此分区的结束柱面，默认为全部柱面，也可以根据提示指定固定的大小。
* **p：**查看下分区结果。（这一步也可以不用执行。）
* **w：**保存退出。
* **理解柱面：**柱面可以理解为五子棋或围棋的棋盘上的格子，整个棋盘就是硬盘，一个硬盘可以按固定大小分为若干个格子（柱面），分区时，你需要指定哪部分的格子来作为你要建立的分区，但这些格子你只能按照编号连续地来使用，如1到500作为分区，而不能使用1、4和9等不连续格子合在一起作为一个分区。

**partprobe：**重新读取分区表。如果上一步w之后提示需要重启才能生效，就可以执行这一步。

**mkfs -t ext4 /dev/sdb1：**根据指定的文件系统格式化分区。

**mkdir /disk1：**创建挂载点，即创建一个空目录，也可以使用已有的任何一个空目录。

**mount /dev/sdb1 /disk1/：**挂载设备/dev/sdb1到指定挂载点/disk1/。

**mount或df：**查看是否挂载成功。（注意fdisk命令只能查看是否分区成功分配，但是不能查看挂载结果。）

### 三、分区自动挂载

系统在启动时，会依据/etc/fstab文件中的配置信息进行自动挂载，所以可以选择手动挂载之后将挂载信息配置在此文件中，也可以配置好此文件后重启系统。但需要注意的是，如果此文件写错了就可能会影响到系统的启动，所以出了挂载报错的问题需要查看和修复此文件。

其中/、/boot、/home、swap、/dev/shm（tmpfs\)、/dev/pts\(devpts\)、/sys\(sysfs\)、/proc\(proc\)等是系统默认的一些分区和挂载点，不能修改它们。

**配置文件中需要配置的六个字段：**

* **第一字段：**分区设备文件名（这种方式就不能改变分区设备的顺序，即该设备文件名不能在某次重启或其他操作后映射到了别的分区）或UUID（硬盘通用唯一识别码，使用UUID就不用担心映射错分区的问题了），    UUID可以通过“dumpe2fs -h 设备文件名”查看Filesystem UUID的值。
* **第二字段：**挂载点。
* **第三字段：**默认的文件系统。
* **第四字段：**-o挂载参数，使用默认defaults即可。
* **第五字段：**指定分区是否被dump备份，0代表不备份，1代表每天备份，2代表不定期备份。分区的备份都保存在分区目录下的lost+found文件中。
* **第六字段：**指定分区是否被fsck检测，0代表不检测，其他数字代表检测的优先级，1的优先级是高于2的，并且我们自己添加的分区应该是大于等于2的。

**mount -a：**配置完成后应该执行这个命令挂载一遍，如果有报错信息，就解决了再重启，不然以重启的方式来挂载，导致系统崩溃了之后就不容易定位问题了，至少解决的时候会比较麻烦。

**mount -o remount,rw /：**这个命令用于/etc/fstab的修复，当系统因为挂载报错后，且/etc/fstab这个文件是只读的，无法去修改文件以修复问题，此时可以使用这个命令重新挂载一次，并让它具有读写的权限。

### 四、相关命令

#### df/du命令

**df \[选项\] \[挂载点\]：**查看系统分区的占用情况。

**选项：**

* **-a：**显示所有的文件系统信息，包括特殊文件系统，如/proc、/sysfs等。
* **-h：**使用习惯单位显示容量，如KB/MB/GB等。
* **-T：**显示文件系统类型。
* **-m：**以MB为单位显示容量。
* **-k：**以KB为单位显示容量，也是默认的显示单位。

**du \[选项\] \[目录名或文件名\]：**查看文件或目录的空间占用大小。最常用的命令为“du -sh 目录名”，用于查看某个目录的占用空间总大小。但是du命令一般用来查看目录的占用大小，文件的占用大小直接使用ll -h命令即可查看。

**选项：**

* **-a：**显示每个子目录和子文件的磁盘占用量，默认只统计子目录本身的磁盘占用量。
* **-h：**使用习惯单位显示磁盘占用量，如KB/MB/GB等。
* **-s：**统计总占用量，而不列出子目录和子文件的占用量。

**df和du命令的区别：**

* df命令是从文件系统角度考虑，不光要考虑文件占用的空间，还要统计被命令或程序占用的空间（最常见的情况就是删除的文件并没有得到释放，所以服务器应该定期进行重启）。
* du命令是面向文件的，只会计算文件或目录占用的空间。

#### mount/umount命令

**mount \[-l\]：**查询系统中已经挂载的设备，-l会显示卷标名称。

**mount -a：**依据配置文件/etc/fstab的内容进行自动挂载。

**mount \[-t 文件系统\] \[-L 卷标名\] \[-o 特殊选项\] 设备文件名 挂载点：**连接设备到指定的挂载点。

**选项：**

* **-t：**指定文件系统类型，可以是ext3/ext4/iso9660等。
* **-L：**挂载指定卷标的分区，而不是安装设备文件名挂载。
* **-o：**可以指定挂载的额外选项。选项内容较多，可以查看相关资料。需要注意这个mount特殊选项都是针对的分区。

**umount 设备文件名或者挂载点：**卸载设备。只有手动卸载设备后，这个设备才能“弹出”，否则就会一直是“使用中”的状态。

