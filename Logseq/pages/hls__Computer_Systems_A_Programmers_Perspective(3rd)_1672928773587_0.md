file:: [Computer_Systems_A_Programmers_Perspective(3rd)_1672928773587_0.pdf](../assets/Computer_Systems_A_Programmers_Perspective(3rd)_1672928773587_0.pdf)
file-path:: ../assets/Computer_Systems_A_Programmers_Perspective(3rd)_1672928773587_0.pdf

- # A Tour of Computer Systems
  ls-type:: annotation
  hl-page:: 38
  hl-color:: red
  id:: 63b6de46-e2dc-463b-8cdc-87e246cb7495
	- ## 1.1 Information Is Bits + Context
	  hl-page:: 40
	  ls-type:: annotation
	  id:: 63b6de53-a262-4b62-8926-2325d38964fb
	  hl-color:: yellow
	  collapsed:: true
		- ```c
		  //hello.c
		  #include <stdio.h>
		  
		  int main()
		  {
		  	printf("hello world\n");
		      return 0;
		  }
		  ```
		- hello.c文件在计算机中用ASCⅡ码表示每一个字符，每一个字符用一个ASCⅡ码表示，用一个字节存储（一个字节就是八个bit的0、1数据）。叫做`text files`。其他类型的文件就是`binary fules`
		- `hello.c`反映了一个最基本的事实，那就是计算机中所有的文件都是以一串 `bit` 来存储的。唯一区别不同数据的方式就是根据数据的上下文（`contexts`）来区分。在不同的contexts中，相同的数据可能表示整数、浮点数、字符或者机器指令
		- 总结：计算机中的数据都是用二进制bit进行存储，使用上下文contexts进行区分的。
		  background-color:: purple
	- ## 1.2 Programs Are Translated by Other Programs into Different Forms
	  hl-page:: 41
	  ls-type:: annotation
	  id:: 63b90b69-7520-425a-90bc-062acbf073f1
	  hl-color:: yellow
	  collapsed:: true
		- 上述C程序是人类可阅读的高级程序，为了在计算机系统上运行上述程序，需要将其转化为低级的机器指令，这些指令被打包成可执行程序，并以二进制形式存储在磁盘文件中。
		- On a Unix system, the translation from source file to object file is performed by a compiler driver:
		  ls-type:: annotation
		  hl-page:: 41
		  hl-color:: yellow
		  id:: 63b90c8d-2676-4fd6-ac16-ef153dedc35e
		- [:span]
		  ls-type:: annotation
		  hl-page:: 42
		  hl-color:: yellow
		  id:: 63b90ce5-f07b-4fad-aefe-e88164b80eae
		  hl-type:: area
		  hl-stamp:: 1673071841502
			- Preprocessing phase. The preprocessor (cpp) modifies the original C program according to directives that begin with the ‘\#’ character. For example, the\#include <stdio.h> command in line 1 of hello.c tells the preprocessor to read the contents of the system header file stdio.h and insert it directly into the program text. The result is another C program, typically with the .i suffix.
			  hl-page:: 42
			  ls-type:: annotation
			  id:: 63b912f1-2536-45d1-9fca-77a020404a75
			  hl-color:: yellow
			- Compilation phase.
			  ls-type:: annotation
			  hl-page:: 42
			  hl-color:: yellow
			  id:: 63b91392-c9ce-46fb-b020-36065c38ae20
				- 编译将`.i`文件转化为`.s`汇编语言文件。汇编语言对不同的高级语言在不同的编译器下提供了相同的输出。
			- Assembly phase.
			  ls-type:: annotation
			  hl-page:: 42
			  hl-color:: yellow
			  id:: 63b91401-c8f2-4999-b4de-4603495bf5ca
				- the assembler (as) translates hello.s into machinelanguage instructions, packages them in a form known as a relocatable object program, and stores the result in the object file hello.o
				  ls-type:: annotation
				  hl-page:: 42
				  hl-color:: yellow
				  id:: 63b91436-d404-4253-a67d-1b03b5e4a143
			- Linking phase.
			  ls-type:: annotation
			  hl-page:: 43
			  hl-color:: yellow
			  id:: 63b91495-c21b-46a9-b355-4a67eac8099f
				- Notice that our hello program calls the printf function, which is part of the standard C library provided by every C compiler. The printf function resides in a separate precompiled object file called printf.o, which must somehow be merged with our hello.o program. The linker (ld) handles this merging. The result is the hello file, which is an executable object file (or simply executable) that is ready to be loaded into memory and executed by the system.
				  ls-type:: annotation
				  hl-page:: 43
				  hl-color:: yellow
				  id:: 63b9180c-ccea-4f57-a616-41a1c4851025
	- ## 1.3 It Pays to Understand How Compilation Systems Work
	  ls-type:: annotation
	  hl-page:: 43
	  hl-color:: yellow
	  id:: 63b91817-90e9-49c4-8d0a-8e0d4e9bc515
	  hl-stamp:: 1673154611028
	  collapsed:: true
		- why programmers need to understand how compilation systems work
		  ls-type:: annotation
		  hl-page:: 43
		  hl-color:: purple
		  id:: 63b954d7-9283-49a2-845f-9cae8b1b05df
		  background-color:: purple
			- Optimizing program performance.
			  ls-type:: annotation
			  hl-page:: 43
			  hl-color:: yellow
			  id:: 63b95505-21ad-409e-bf15-40e35f3b7d08
			- Understanding link-time errors.
			  ls-type:: annotation
			  hl-page:: 44
			  hl-color:: yellow
			  id:: 63b954f2-1eb0-41df-a960-3a3bda496393
			- Avoiding security holes.
			  ls-type:: annotation
			  hl-page:: 44
			  hl-color:: yellow
			  id:: 63b95512-bc3c-43b3-9191-34ac86f78f9c
	- ## 1.4 Processors Read and Interpret Instructions Stored in Memory
	  ls-type:: annotation
	  hl-page:: 44
	  hl-color:: yellow
	  id:: 63b95519-215e-4d17-aab6-0546f4ddbefb
		- ### 1.4.1 Hardware Organization of a System
		  ls-type:: annotation
		  hl-page:: 45
		  hl-color:: yellow
		  id:: 63b95638-1ccf-4d0f-bc33-cf377e04d957
			- Buses
			  ls-type:: annotation
			  hl-page:: 45
			  hl-color:: yellow
			  id:: 63b95676-03c4-460f-8d99-16318192ed6a
		- ### 1.4.2 Running the hello Program
		  ls-type:: annotation
		  hl-page:: 47
		  hl-color:: yellow
		  id:: 63b96233-9fef-4ab7-a731-fe33fbab3c44
	- ## 1.5 Caches Matter
	  ls-type:: annotation
	  hl-page:: 48
	  hl-color:: yellow
	  id:: 63babc56-8f16-4564-97a0-7862d837bd01