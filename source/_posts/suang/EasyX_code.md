---
title: 使用EasyX制作游戏需要读写文件时遇到编码问题的解决方法
date: 2024-05-28
update: 2024-06-04
permalink: articles/suang/EasyX_code/
categories: suang
tags: [c++, 编码, EasyX]
---

痛苦之子suang又遇到了什么问题呢？
噢！原来是编码问题！
快来帮他解决吧！
痛苦之子~~~乔戈亚~~~~~

<!-- More -->

### 一、编码问题
例如我们需要从file.txt中读取文字，再使用`outtextxy()`函数向窗口绘制文字。
查找EasyX的官方文档可知，该函数有两个重载，分别为：`void outtextxy(int x, int y, LPCTSTR str)`和`void outtextxy(int x, int y, TCHAR c)`。
如果我们的file.txt文件使用GBK或者GB2312编码的话，会导致VS编译器混合utf-8编码和GBK编码，导致程序不能正确绘制文字。
编码问题一直是令人头痛的问题，这里给出通用的方法论，希望能够带来一些帮助。

### 二、解决方法

#### 1. 重新编码txt文件

首先使用vscode打开file.txt文件，确保文件编码为utf-8。

如果不是utf-8编码，点击选择编码，通过编码重新打开，选择utf-8编码，这时原来的内容会变成乱码，将原来的内容删除，重新输入，保存即可。

#### 2. 代码部分

首先，我们需要用到`std::wstring_convert`，这个标准库需要头文件`#include<codecvt>`，我们定义`string str`，用`static std::wstring_convert<std::codecvt_utf8<wchar_t>, wchar_t> converter;`定义我们所需要的从utf-8编码的字符串到宽字符串的转换器。

打开文件，读取文件内容到str内，使用`from_bytes(str)`函数即可实现字符串的转换，又由于outtextxy没有使用wstring的重载，使用wstring的成员函数`c_str()`即可转换成`wchar_t`字符串，最终，我们用这样的代码将文字绘制在窗口上`outtextxy(10, 10, converter.from_bytes(str).c_str());`。

完整代码如下：

```cpp
#include<fstream>
#include<codecvt>
#include<string>
#include<iostream>
#include<graphics.h>

std::string str;

int main()
{
	initgraph(500, 500);

	static std::wstring_convert<std::codecvt_utf8<wchar_t>, wchar_t> converter;

	std::ifstream infile("file.txt");
	if (!infile)
	{
		std::cerr << "无法打开文件" << std::endl;
		return 0;
	}
	std::getline(infile, str);
	infile.close();

	while (true)
	{
		outtextxy(10, 10, converter.from_bytes(str).c_str());
	}
}
```

### 三、问题延伸

如果我们需要在程序内使用`InputBox`对话框输入，并将输入的内容正确保存在文件里，该如何操作呢。
首先查看`InputBox`的参数列表`bool InputBox(LPTSTR pString, int nMaxCount, LPCTSTR pPrompt = NULL, LPCTSTR pTitle = NULL, LPCTSTR pDefault = NULL, int width = 0, int height = 0, bool bOnlyOK = true);`，我们关注第一个参数，`InputBox`只接受`&wchar_t`的参数，因此，假设输入的字符串最大长度为256，我们定义`TCHAR buffer[256];`数组来接收输入，写下这样的代码来弹出对话框`InputBox(buffer, 256, _T("请输入："), _T("输入框"), NULL, 0, 0, TRUE);`打开文件，定义`std::string str;`，仍然使用我们刚才定义的转换器，使用`to_bytes()`函数，即可将输入的内容转换为utf-8字符串。再进行输入即可。

完整代码如下：

```cpp
#include<fstream>
#include<codecvt>
#include<string>
#include<iostream>
#include<graphics.h>

int main()
{
	initgraph(500, 500);

	static std::wstring_convert<std::codecvt_utf8<wchar_t>, wchar_t> converter;

	TCHAR buffer[256];
	InputBox(buffer, 256, _T("请输入："), _T("输入框"), NULL, 0, 0, TRUE);

	std::ofstream outfile("file.txt");
	if (!outfile)
	{
		std::cerr << "无法打开文件" << std::endl;
		return 0;
	}
	std::string str = converter.to_bytes(buffer);
	outfile << str;
	outfile.close();
}
```

---

说些题外话，suang心事很重，虽然表面看上去什么都不在乎，但是其实内心很脆弱很敏感。
时常会独自emo，我的朋友跟我说啊，要把不开心的事情说出去，不能憋着。
其实我是不想憋的，但是我说不出口，或者说我已经不知道怎么去说了，每次难受的时候，就什么都说不出来了，也许发生在我身上的事情真的那么那么复杂，也许只有亲身经历过，才能感同身受，才能知道怎么解开我的心结罢。
可我没什么文采，写不来文章，又没那么专业，拍不了电影。所以，我想做游戏，在另一个世界里，说我想说但说不出口的，做我想做却没勇气做的，弥补我曾经的遗憾。也许这样，我就真的释然了。

---

做游戏的想法埋下了，我却一直都没去做，一是我确实没什么实力，二是我并不知道如何开始，直到我发现了大V老师......

我真的没想到，大V老师能帮我远程解决问题，当时真的震惊到我了，我跟我的室友炫耀了好久，我真的没想到，在那些卖课的占据b站首页大部分的今天，还有大V老师这样的一股清流。来到群里之后我发现，真的有乐于分享，愿意帮助我的人，suang真的真的很感动。
