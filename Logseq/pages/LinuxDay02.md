- # 一、文件子系统
	- ## ls -a -l -h -i #card
	  collapsed:: true
		- 1.**`ls - list directory contents`**
		- 展示当前工作目录下的内容
		- `ls [OPTION]... [FILE]...`后面加目录可以展示该目录的内容
		-
		-
		- **在Linux中以`.`开头的文件或者文件夹就是隐藏文件。**
		- `-a, --all`
		                `do not ignore entries starting with .`
			- 显示所有文件，包括隐藏文件
		- `-l     use a long listing format`：long：展示详细信息
		- ```
		  total 8
		  drwxrwxr-x 9 ubuntu ubuntu 4096 May 19 22:45 Chapter01
		  drwxrwxr-x 3 ubuntu ubuntu 4096 Jun 22 07:56 wangdao
		  ```
			- drwxrwxr-x：文件类型+权限
				- d：directory：目录
					- -：普通文件
					- l：symbolic link：符号链接，软连接
					- c：character：字符设备文件
						- 输入和输出字符的设备：键盘、显示器
					- b：block，块设备文件
						- 输入输出一块一块的传，磁盘
					- p：pipe：管道文件
						- 进程间通信的一种机制
					- s：socket：套接字
						- 用于网络通信的一种机制
				- rwx
					- r：read，读权限
					- w：write，写权限
					- x：execute：执行权限
				- 第一组：用户的权限
				- 第二组：用户所在的组内其他成员的权限
				- 第三组：其他人的权限。
			- 第一个数字：表示硬链接数目。
			- 第一个ubuntu：文件所有者的名字
			- 第二个：文件所有者所在的组
			- 4096：文件的大小
			- `May 19 22:45`：上一次修改的时间
			- `Chapter01`：文件名
		- `-h, --human-readable`
		                `with -l and -s, print sizes like 1K 234M 2G etc.`
			- 以人类可读的方式显式。因为有的文件太大了，显示数字不好看，如下面显示4k，就好看了，一目了然有多大
			- ```
			  ubuntu@VM-16-2-ubuntu:~/My_Code/wangdao$ ls -l -h
			  total 4.0K
			  drwxrwxr-x 2 ubuntu ubuntu 4.0K Jun 22 07:53 LinuxDay02
			  ```
		- `-i, --inode`
		                `print the index number of each file`
			- 打印inode结点
			- ```
			  ubuntu@VM-16-2-ubuntu:~/My_Code/wangdao$ ls -i
			  789762 LinuxDay02
			  
			  ubuntu@VM-16-2-ubuntu:~/My_Code/wangdao$ ls -li
			  total 4
			  789762 drwxrwxr-x 2 ubuntu ubuntu 4096 Jun 22 07:53 LinuxDay02
			  ```
		-
	- ## cp -r -i -v -u #card
	  collapsed:: true
		- `cp - copy files and directories`复制文件或目录
		- ```
		  cp file1 file2
		  将file1复制到file2
		  ```
			- 如果file2不存在，则创建，如果file2存在，则会覆盖原有的内容
		- ```
		  -i, --interactive
		                prompt before overwrite (overrides a previous -n option)
		                
		  ubuntu@VM-16-2-ubuntu:~$ cp dir1 dir2 -rvi
		  cp: overwrite 'dir2/dir1/c.txt'? y
		  'dir1/c.txt' -> 'dir2/dir1/c.txt'
		  cp: overwrite 'dir2/dir1/a.txt'? y
		  'dir1/a.txt' -> 'dir2/dir1/a.txt'
		  //y表示确认，其他字符表示不覆盖
		  ```
			- 在cp的时候给予提示，交互的方式
		- `cp file dir`
			- 把文件复制到dir目录，dir一定要存在，不然就会把他当成文件，默认创建了一个文件，就不对了。
			- 如：
				- ```
				  ubuntu@VM-16-2-ubuntu:~$ cp a.txt dir
				  -rw-rw-r--  1 ubuntu ubuntu   13 Jun 22 10:54 dir
				  dir是个普通文件
				  ```
		- `cp dir1 di2 -r`    -r表示递归复制
			- ```
			  -R, -r, --recursive
			                copy directories recursively
			  ```
			- ```
			  不加r，会提示
			  ubuntu@VM-16-2-ubuntu:~$ cp dir1 dir2
			  cp: -r not specified; omitting directory 'dir1'
			  
			  ubuntu@VM-16-2-ubuntu:~$ cp dir1 dir2 -r
			  ubuntu@VM-16-2-ubuntu:~$ ls dir2
			  dir1
			  //将整个目录复制到了dir2
			  ```
			- 如果dir2存在，则会将dir1以及dir1中所有内容复制到dir2中。
			- 如果dir2不存在，则首先创建dir2，并复制dir1里面的内容到dir2中，不包括dir1了。
		- 打印一些详细信息  -v
			- ```
			  -v, --verbose
			                explain what is being done
			  
			  ubuntu@VM-16-2-ubuntu:~$ cp dir1 dir2 -rv
			  'dir1/a.txt' -> 'dir2/dir1/a.txt'
			  'dir1/c.txt' -> 'dir2/dir1/c.txt'
			  ```
		- -u。只复制最新的数据或者dir2没有的数据。
			- ```
			  -u, --update
			                copy only when the SOURCE file is newer than the destination file or when the  desti‐
			                nation file is missing
			  
			  ubuntu@VM-16-2-ubuntu:~$ cp dir1 dir2 -rvu
			  'dir1/a.txt' -> 'dir2/dir1/a.txt'
			  ```
		-
	- ## mv -i -v -u #card
	  collapsed:: true
		- `mv - move (rename) files`
		- `mv file1 file2`
			- 把file1移动到file2里去
			- 如果file2不存在，则会创建文件，然后移动到file2
			- 如果文件存在，则会覆盖原来的文件的内容
		- 交互的方式进行mv，就像cp的-i
			- ```
			  -i, --interactive
			                prompt before overwrite
			  ```
		- `mv file dir`
			- dir一定要存在，不然也会被当作文件
			- ```
			  //文件不存在
			  ubuntu@VM-16-2-ubuntu:~$ mv c.txt d
			  -rw-rw-r--  1 ubuntu ubuntu     0 Jun 22 11:15 d
			  
			  ubuntu@VM-16-2-ubuntu:~$ mv c.txt b
			  ubuntu@VM-16-2-ubuntu:~$ ls b
			  a.txt  c.txt
			  ```
		- `mv dir1 dir2`
			- 把一个目录移动到另外一个目录里
			- 如果dir2不存在，作用就相当于重命名dir1
			- 如果存在，将dir1整个移动到dir2目录下面去
		- -v：显示详细信息
			- ```
			  -v, --verbose
			                explain what is being done
			  ```
		- -u：同cp -u
			- ```
			  -u, --update
			                move  only when the SOURCE file is newer than the destination file or when the desti‐
			                nation file is missing
			  ```
	- ## rm -i、-f、-r、-v #card
	  collapsed:: true
		- `rm - remove files or directories`
			- 删除文件和文件夹
		- `-i     prompt before every removal`
			- 每次删除之前都给予提示
			- ```
			  ubuntu@VM-16-2-ubuntu:~$ rm c.txt -i
			  rm: remove regular empty file 'c.txt'? p
			  //只有y是确认删除，其他都不是
			  ```
		- 递归删除目录-r
			- ```
			  -r, -R, --recursive
			                remove directories and their contents recursively
			  ```
		- ```
		  -v, --verbose
		                explain what is being done
		  ```
		- -f：强制删除，不会有任何提示
			- ```
			  -f, --force
			                ignore nonexistent files and arguments, never prompt
			  ```
		- ## alias 别名
			- ```
			  ubuntu@VM-16-2-ubuntu:~$ alias
			  alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
			  alias egrep='egrep --color=auto'
			  alias fgrep='fgrep --color=auto'
			  alias grep='grep --color=auto'
			  alias l='ls -CF'
			  alias la='ls -A'
			  alias ll='ls -alF'
			  alias ls='ls --color=auto'
			  
			  
			  alias his='history' 可以起别名，但只是临时设置，重连就没了。
			  
			  ```
			- `history`可以查看前面执行过那些命令
				- `ubuntu@VM-16-2-ubuntu:~$ history > h.txt`可以保存执行过的命令信息
			-
			-
		-
	-
	-
	-