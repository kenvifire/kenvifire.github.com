---
layout: page
title: OS02-Boot Loader
tagline: OS
---
###软盘格式
可以通过定义一些文件来说明磁盘的格式，相当于系统的格式化功能，这里我们把磁盘的格式设置为FAT12
<pre><code>
	DB		0x90
	DB		"HARIBOTE"	
	DW		512				
	DB		1			
	DW		1				
	DB		2			
	DW		224				
	DW		2880			
	DB		0xf0			
	DW		9				
	DW		18			
	DW		2				
	DD		0				
	DD		2880			
	DB		0,0,0x29		
	DD		0xffffffff	
	DB		"HARIBOTEOS "
	DB		"FAT12   "
</code></pre>

###Boot Loader
前面讲过，启动扇区的大小是512字节，出去末尾的0xaa55就只有510字节，操作系统的内核是不可能在这510字节的空间里完成的，所以内核需要用单独的程序来编写，但是需要有程序在计算机启动的时候加载内核，这个程序就是Boot Loader。

Boot Loader的功能很简单，就是在运行的时候加载内核到内存，然后跳到内核的开始处执行即可。
<pre><code>
	CYLS	EQU	2	
	ORG	0x7c00
	JMP	entry
	DB		0x90
	DB		&quot;HARIBOTE&quot;
	DW		512				
	DB		1			
	DW		1				
	DB		2			
	DW		224				
	DW		2880			
	DB		0xf0			
	DW		9				
	DW		18			
	DW		2				
	DD		0				
	DD		2880			
	DB		0,0,0x29		
	DD		0xffffffff	
	DB		&quot;HARIBOTEOS &quot;
	DB		&quot;FAT12   &quot;
	times	18	DB 0

entry:
	MOV	AX,0
	MOV	SS,AX
	MOV	SP,0x7c00
	MOV	DS,AX

	MOV	AX,0x0820
	MOV	ES,AX
	MOV	CH,0
	MOV	DH,0
	MOV	CL,2

readloop:
	MOV	SI,0
retry:
	MOV	AH,0x02
	MOV	AL,1
	MOV	BX,0
	MOV	DL,0x00
	INT	0x13
	JNC	next
	ADD	SI,1
	CMP	SI,5
	JAE	error
	MOV	AH,0x00
	MOV	DL,0x00
	INT	0x13
	JMP	retry

next:
	MOV	AX,ES
	ADD	AX,0X0020
	MOV	ES,AX
	ADD	CL,1
	CMP	CL,18
	JBE	readloop
	MOV	CL,1
	ADD	DH,1
	CMP	DH,2
	JB	readloop
	MOV	DH,0
	ADD	CH,1
	CMP	CH,CYLS
	JB	readloop

	MOV	[0x0ff0],CH
	JMP	0xc200

error:

	MOV	SI,msg


putloop:
	MOV	AL,[SI]
	ADD	SI,1
	CMP	AL,0
	JE	fin
	MOV	AH,0x0e
	MOV	BX,15
	INT	0x10
	JMP	putloop

vedio:
	MOV	AL,0x13
	MOV	AH,0x00
	INT	0x10
fin:
	HLT
	JMP	fin

msg:
	DB	0x0a,0x0a
	DB	&quot;hello,error&quot;
	DB	0x0a
	DB	0

	times	510-($-$$)	db 0
	DB	0x55,0xaa
	times  1474048 db 0
</code></pre>
以上就是Boot Loader的代码，它的主要作用是读取整个软盘的10个柱面（保证能读取到kernel的内容），加载到内存从0X0200开始的地方。然后跳转到内核的开始（`JMP	0xc200`）处执行。

###kernel
kerlne也就是系统的内核，它是系统的核心。我们这里主要是为了演示Boot Loader，所以只有一个很简陋的内核。
<pre><code>
	ORG 0xc200

	MOV AL,0x13
	MOV AH,0x00
	INT 0x10

fin:
	HLT
	JMP fin
</code></pre>


这个就是我们的内核，它的主要功能就是黑屏（￣□￣｜｜），然后无限等待。

###执行

* 生成镜像文件myOS.img
* 生成kernel.sys内核
* 将kernel.sys文件存入到myOS.img软驱中
	* 用VFD软件加载myOS.img到虚拟软驱
	* 把kernel.sys拷贝到myOS.img中
* 用Qemu或者bochs加载myOS.img

###几个常量
* **0x8000** 这个是我们放软盘数据的地方，但是为什么是这里呢？这是因为，系统启动的时候，内存是不能乱用的，有些地方是存有系统数据的，我们只能在指定的内存出进行操作。查看资料可以知道0x00007c00-0x00007dff的地方是供启动程序使用的。由于计算机在启动的时候会自动把第一个扇区加载到0x7c00开始的512个字节处，所以启动扇区加载完后，剩下的内存是0x00007e00-0x00007dff，这里为了方便，我们取0x8000。

* **0x004200** 这个常量代码里没有出现，但是隐含在代码里。这个是文件内容在FAT12里存储的起始地址。也就是说我们的kernel是在软盘的0x004200处开始存储的。

* **0xc200** 这个是kernel的起始地址，它等于0x8000+0x4200，也就是说软盘的内容加载到0x8000处后，那么相对于软盘起始处距离为0x4200的kernel文件在内存中的位置就是0xc200。

###替换内核
有了Boot Loader 那么我们就可以随时替换我们的内核了，只需要修改kernel文件即可。

###需要注意的地方
* 用Qemu来加载光驱的时候，需要修改`-drive file=myOS.img,media=disk`为`-fda myOS.img`，否则，默认会把镜像当作光盘加载，导致读取失败。







