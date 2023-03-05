# CrackMe 001 - Acid burn

作为“适合破解新手的160个crackme练手”系列的第一个 CrackMe，难度很小，唯一的难点在于关键代码处函数调用的作用分析，然而这一点可以通过加载 IDA 签名+[x64dbgida](https://github.com/x64dbg/x64dbgida#installation) 插件轻松解决。另外还有一个问题就是关键函数里面有无效代码，可能会因此误判。

## 分析

<img src="images/1.png" style="zoom: 70%;" />

首先对 Acid burn.exe 检测。可以看到这是一个用 Delphi 3.x 写的无壳程序，所以可以直接用 Delphi Decompiler 来打开它。

<img src="images/2.png" style="zoom: 60%;" />

在 Delphi Decompiler 中打开后查看 **Forms** 和 **Procedures** 选项卡，可以清晰的看到窗口对应的过程。

从窗口名字可以推断出 Tindex 应该是启动时弹出的窗口，TNS 是要求输出 name 和 serial 的窗口，Tserial 是仅要求输入 serial 的窗口。

## 去除 NAG 弹窗

<img src="images/3.png" style="zoom:80%;" />

打开 Acid burn.exe 后可以看到一个 NAG 弹窗并且要求去除它，这个弹窗是在主窗口显示之前出现的，所以可以推断出主窗口应该会有一个窗口创建的事件来显示这个弹窗。

<img src="images/4.png" style="zoom:80%;" />

从图中可以看到，Tindex 有个窗口创建的事件可能用来弹出弹窗。转到 **Procedures** 选项卡并点击 **Tindex** 窗口后双击 **FormCreate** 事件后不难看出这里就是弹出弹窗的关键代码。

```asm
0042F784   6A00                   push    $00

* Possible String Reference to: 'hello you have to kill me!'
|
0042F786   B9A0F74200             mov     ecx, $0042F7A0

* Possible String Reference to: 'Welcome to this Newbies Crackme mad
|                                e by ACiD BuRN [CracKerWoRlD]'
|
0042F78B   BABCF74200             mov     edx, $0042F7BC

* Reference to TApplication instance
|
0042F790   A1480A4300             mov     eax, dword ptr [$00430A48]
0042F795   8B00                   mov     eax, [eax]

* Reference to: Forms.Proc_0042A170
|
0042F797   E8D4A9FFFF             call    0042A170
0042F79C   C3                     ret
```

<img src="images/5.png" style="zoom:80%;" />

在 IDA 中打开 Acid burn.exe 后点击 **View**->**Open subviews**->**Signatures** 右键并点击 **Apply new signature...**，并导入 delphi 和 d3vcl 两个签名。导入签名后可以看到 IDA 正确识别了函数。

<img src="images/6.png" style="zoom:80%;" />

在 **Procedures**->**Events** 中右击 **FormCreate** 后点击 **Copy current RVA to clipboard** 复制这个事件的 RVA 0042F784，并在 IDA 中按下 **G** 键（或点击 **Jump**->**Jump to address...**）输入这个地址后可以转到这个事件的函数。

<img src="images/7.png" style="zoom:80%;" />

转到 **Options**->**General...**->**Disassembly**->**Display disassembly line parts** 中开启 **Line prefixes (graph)** 和 **Stack Pointer** 后可以看到，0042F784 和 0042F79C 的栈指针是相同的，所以选中 0042F784 并点击 **Edit**->**Patch program**->**Assemble...** 来修改汇编代码。

<img src="images/8.png" style="zoom:80%;" />

将 `push 0` 修改为 `jmp 0x42F79C` 并按下回车修改后点击 **Edit**->**Patch Program**->**Apply patches to...** 并点击 **Apply patches**将修改保存到文件即可跳过弹窗。

<img src="images/9.png" style="zoom:80%;" />

可以点击 **Edit**->**x64dbgida**->**Export x64dbg database** 来导出之前应用的签名来给接下来的 x64dbg 来使用，保存到 x64dbg 目录下的 **x32\db\Acid burn.exe.dd32** 文件即可（如果没有 x64dbgida 插件可以在此处[下载 Acid burn.exe.dd32 文件](misc/Acid%20burn.exe.dd32)）。

## Serial / Name 分析

<img src="images/10.png" style="zoom:80%;" />

在 Delphi Decompiler 中转到 **Forms** 选项卡后点击 **TNS** 窗口可以看到 “Check it Baby !”按钮的控件名叫 **BitBtn1**，同时它还有一个单击事件，这样不难猜出 **BitBtn1Click** 是注册的关键函数。

<img src="images/11.png" style="zoom:80%;" />

转到 **Procedures** 选项卡后点击 **TNS**，右击 **BitBtn1Click** 并点击 **Copy current RVA to clipboard**。

在 x64dbg 打开 Acid burn.exe 后按 **Ctrl**+**G** 将 0042F998 粘贴到输入框中按下回车。

<img src="images/12.png" style="zoom:80%;" />

在函数起始处按下 **F2** 再按下 **F9** 开始调试。

### 第一处关键点

运行程序后点击“Serial / Name”，在两个输入框中输入假码（这里输入“123456”和“qwert”）并点击“Check it Baby !”。

在 0042F998 处断下后 F8 单步步过可以看到有一处关键代码。

<img src="images/13.png" style="zoom:80%;" />

## Name 分析

