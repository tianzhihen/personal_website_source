---
title: source Insight unknown command or macro 解决办法
tags: [source Insight,编辑器]
categories: 编辑器
---

### 0x00 前言

>
在接手一个新项目的时候由于别人的代码对于自己来说太过于陌生，因此使用一个代码组织良好的编辑器来看代码至关重要，毫无例外，SI是比较好的选择，但是老外的东西对中文支持就是不好。大多是情况下的中文乱码可以通过将原文件另存为编码格式为ANSI解决。但是今天碰到了使用backspace和左右移动光标时的问题（unknown command or macro：XXX）。


### 0x01 解决办法
1.在sourceinsight的安装目录下创建新以下文件：

|序号|文件名|
|---|-----|
|1|ShiftCursorLeft.em|
|2|ShiftCursorRight.em|
|3|SuperCursorLeft.em|
|4|SuperCursorRight.em|
|5|superbackspace.em|
|6|superdelete.em|

每个文件的代码如下：
**ShiftCursorLeft.em 文件**
```C++
macro IsShiftLeftComplexCharacter()
{
hwnd = GetCurrentWnd();
hbuf = GetCurrentBuf();

if (hbuf == 0)
   return 0;

selRec = GetWndSel(hwnd);
pos = selRec.ichFirst;
ln = selRec.lnFirst;
text = GetBufLine(hbuf, ln);
len = strlen(text);

if(len == 0 || len < pos)
   return 1;

//Msg("@len@;@pos@;");
if(pos > 0)
{
   i=pos;
   count=0;
   while(AsciiFromChar(text[i-1]) >= 160)
   {
    i = i - 1;
    count = count+1;
    if(i == 0)
     break;
   }

   if((count/2)*2==count|| count==0)
    return 0;
   else
    return 1;
}

return 0;
}

macro shiftMoveLeft()
{
hwnd = GetCurrentWnd();
hbuf = GetCurrentBuf();
if (hbuf == 0)
        stop;

ln = GetBufLnCur(hbuf);
ipos = GetWndSelIchFirst(hwnd);
totalLn = GetBufLineCount(hbuf);
text = GetBufLine(hbuf, ln);
selRec = GetWndSel(hwnd);

//curLen = GetBufLineLength(hbuf, selRec.lnFirst);
//Msg("@curLen@;@selRec@");
if(selRec.ichFirst == 0)
{
   if(selRec.lnFirst == 0)
    stop;

   selRec.lnFirst = selRec.lnFirst - 1;
   selRec.ichFirst = GetBufLineLength(hbuf, selRec.lnFirst)-1;
   SetWndSel(hwnd, selRec);
   if(IsShiftLeftComplexCharacter())
    shiftMoveLeft();
   stop;
}

selRec.ichFirst = selRec.ichFirst-1;
SetWndSel(hwnd, selRec);
}

macro SuperShiftCursorLeft()
{
if(IsComplexCharacter())
   SuperCursorLeft();

shiftMoveLeft();
if(IsShiftLeftComplexCharacter())
   shiftMoveLeft();
}
```
**ShiftCursorRight.em 文件**
```C++

macro IsShiftRightComplexCharacter()
{
hwnd = GetCurrentWnd();
hbuf = GetCurrentBuf();

if (hbuf == 0)
   return 0;

selRec = GetWndSel(hwnd);
pos = selRec.ichLim;
ln = selRec.lnLast;
text = GetBufLine(hbuf, ln);
len = strlen(text);

if(len == 0 || len < pos)
   return 1;

//Msg("@len@;@pos@;");
if(pos > 0)
{
   i=pos;
   count=0;
   while(AsciiFromChar(text[i-1]) >= 160)
   {
    i = i - 1;
    count = count+1;
    if(i == 0)
     break;
   }

   if((count/2)*2==count|| count==0)
    return 0;
   else
    return 1;
}

return 0;
}

macro shiftMoveRight()
{
hwnd = GetCurrentWnd();
hbuf = GetCurrentBuf();
if (hbuf == 0)
        stop;

ln = GetBufLnCur(hbuf);
ipos = GetWndSelIchFirst(hwnd);
totalLn = GetBufLineCount(hbuf);
text = GetBufLine(hbuf, ln);
selRec = GetWndSel(hwnd);

curLen = GetBufLineLength(hbuf, selRec.lnLast);
if(selRec.ichLim == curLen+1 || curLen == 0)
{
   if(selRec.lnLast == totalLn -1)
    stop;

   selRec.lnLast = selRec.lnLast + 1;
   selRec.ichLim = 1;
   SetWndSel(hwnd, selRec);
   if(IsShiftRightComplexCharacter())
    shiftMoveRight();
   stop;
}

selRec.ichLim = selRec.ichLim+1;
SetWndSel(hwnd, selRec);
}

macro SuperShiftCursorRight()
{
if(IsComplexCharacter())
   SuperCursorRight();

shiftMoveRight();
if(IsShiftRightComplexCharacter())
   shiftMoveRight();
}
```

**SuperCursorLeft.em 文件**
```C++
macro IsComplexCharacter()
{
hwnd = GetCurrentWnd();
hbuf = GetCurrentBuf();

if (hbuf == 0)
   return 0;
//当前位置
pos = GetWndSelIchFirst(hwnd);

//当前行数
ln = GetBufLnCur(hbuf);

//得到当前行
text = GetBufLine(hbuf, ln);

//得到当前行长度
len = strlen(text);

//从头计算汉字字符的个数
if(pos > 0)
{
   i=pos;
   count=0;
   while(AsciiFromChar(text[i-1]) >= 160)
   {
    i = i - 1;
    count = count+1;
    if(i == 0)
     break;
   }

   if((count/2)*2==count|| count==0)
    return 0;
   else
    return 1;
}

return 0;
}

macro moveleft()
{
hwnd = GetCurrentWnd();
hbuf = GetCurrentBuf();
if (hbuf == 0)
        stop;   // empty buffer

ln = GetBufLnCur(hbuf);
ipos = GetWndSelIchFirst(hwnd);

if(GetBufSelText(hbuf) != "" || (ipos == 0 && ln == 0))   // 第0行或者是选中文字,则不移动
{
   SetBufIns(hbuf, ln, ipos);
   stop;
}

if(ipos == 0)
{
   preLine = GetBufLine(hbuf, ln-1);
   SetBufIns(hBuf, ln-1, strlen(preLine)-1);
}
else
{
   SetBufIns(hBuf, ln, ipos-1);
}
}

macro SuperCursorLeft()
{
moveleft();
if(IsComplexCharacter())
   moveleft();
}
```
**SuperCursorRight.em 文件**
```C++
macro moveRight()
{
hwnd = GetCurrentWnd();
hbuf = GetCurrentBuf();
if (hbuf == 0)
        stop;   // empty buffer
ln = GetBufLnCur(hbuf);
ipos = GetWndSelIchFirst(hwnd);
totalLn = GetBufLineCount(hbuf);
text = GetBufLine(hbuf, ln);

if(GetBufSelText(hbuf) != "")   //选中文字
{
   ipos = GetWndSelIchLim(hwnd);
   ln = GetWndSelLnLast(hwnd);
   SetBufIns(hbuf, ln, ipos);
   stop;
}

if(ipos == strlen(text)-1 && ln == totalLn-1) // 末行
   stop;

if(ipos == strlen(text))
{
   SetBufIns(hBuf, ln+1, 0);
}
else
{
   SetBufIns(hBuf, ln, ipos+1);
}
}

macro SuperCursorRight()
{
moveRight();
if(IsComplexCharacter()) // defined in SuperCursorLeft.em
   moveRight();
}
```
**superbackspace.em 文件**
```C++
macro SuperBackspace()
{
    hwnd = GetCurrentWnd();
    hbuf = GetCurrentBuf();

    if (hbuf == 0)
        stop;   // empty buffer

    // get current cursor postion
    ipos = GetWndSelIchFirst(hwnd);

    // get current line number
    ln = GetBufLnCur(hbuf);

    if ((GetBufSelText(hbuf) != "") || (GetWndSelLnFirst(hwnd) != GetWndSelLnLast(hwnd))) {
        // sth. was selected, del selection
        SetBufSelText(hbuf, " "); // stupid & buggy sourceinsight :(
        // del the " "
        SuperBackspace(1);
        stop;
    }

    // copy current line
    text = GetBufLine(hbuf, ln);

    // get string length
    len = strlen(text);

    // if the cursor is at the start of line, combine with prev line
    if (ipos == 0 || len == 0) {
        if (ln <= 0)
            stop;   // top of file
        ln = ln - 1;    // do not use "ln--" for compatibility with older versions
        prevline = GetBufLine(hbuf, ln);
        prevlen = strlen(prevline);
        // combine two lines
        text = cat(prevline, text);
        // del two lines
        DelBufLine(hbuf, ln);
        DelBufLine(hbuf, ln);
        // insert the combined one
        InsBufLine(hbuf, ln, text);
        // set the cursor position
        SetBufIns(hbuf, ln, prevlen);
        stop;
    }

    num = 1; // del one char
    if (ipos >= 1) {
        // process Chinese character
        i = ipos;
        count = 0;
        while (AsciiFromChar(text[i - 1]) >= 160) {
            i = i - 1;
            count = count + 1;
            if (i == 0)
                break;
        }
        if (count > 0) {
            // I think it might be a two-byte character
            num = 2;
            // This idiot does not support mod and bitwise operators
            if ((count / 2 * 2 != count) && (ipos < len))
                ipos = ipos + 1;    // adjust cursor position
        }
    }

    // keeping safe
    if (ipos - num < 0)
        num = ipos;

    // del char(s)
    text = cat(strmid(text, 0, ipos - num), strmid(text, ipos, len));
    DelBufLine(hbuf, ln);
    InsBufLine(hbuf, ln, text);
    SetBufIns(hbuf, ln, ipos - num);
    stop;
}
```

**superdelete.em文件**
```C++
macro SuperDelete()
{
    hwnd = GetCurrentWnd();
    hbuf = GetCurrentBuf();

    if (hbuf == 0)
        stop;   // empty buffer

    // get current cursor postion
    ipos = GetWndSelIchFirst(hwnd);

    // get current line number
    ln = GetBufLnCur(hbuf);

    if ((GetBufSelText(hbuf) != "") || (GetWndSelLnFirst(hwnd) != GetWndSelLnLast(hwnd))) {
        // sth. was selected, del selection
        SetBufSelText(hbuf, " "); // stupid & buggy sourceinsight :(
        // del the " "
        SuperDelete(1);
        stop;
    }

    // copy current line
    text = GetBufLine(hbuf, ln);

    // get string length
    len = strlen(text);

    if (ipos == len || len == 0) {
totalLn = GetBufLineCount (hbuf);
lastText = GetBufLine(hBuf, totalLn-1);
lastLen = strlen(lastText);

        if (ipos == lastLen)// end of file
   stop;

        ln = ln + 1;    // do not use "ln--" for compatibility with older versions
        nextline = GetBufLine(hbuf, ln);
        nextlen = strlen(nextline);
        // combine two lines
        text = cat(text, nextline);
        // del two lines
        DelBufLine(hbuf, ln-1);
        DelBufLine(hbuf, ln-1);
        // insert the combined one
        InsBufLine(hbuf, ln-1, text);
        // set the cursor position
        SetBufIns(hbuf, ln-1, len);
        stop;
    }

    num = 1; // del one char
    if (ipos > 0) {
        // process Chinese character
        i = ipos;
        count = 0;
        while (AsciiFromChar(text[i-1]) >= 160) {
            i = i - 1;
            count = count + 1;
            if (i == 0)
                break;
        }
        if (count > 0) {
            // I think it might be a two-byte character
            num = 2;
            // This idiot does not support mod and bitwise operators
            if (((count / 2 * 2 != count) || count == 0) && (ipos < len-1))
                ipos = ipos + 1;    // adjust cursor position
        }

// keeping safe
if (ipos - num < 0)
            num = ipos;
    }
    else {
i = ipos;
count = 0;
while(AsciiFromChar(text[i]) >= 160) {
     i = i + 1;
     count = count + 1;
     if(i == len-1)
   break;
}

if(count > 0) {
     num = 2;
}
    }

    text = cat(strmid(text, 0, ipos), strmid(text, ipos+num, len));
    DelBufLine(hbuf, ln);
    InsBufLine(hbuf, ln, text);
    SetBufIns(hbuf, ln, ipos);
    stop;
}
```

2.将这些文件复制到SI的base工程目录下（base工程在你的SI统一的工程目录下）。
3.用SI打开base工程，然后通过Project->Add and remove project files
添加刚刚复制过来的文件
![这里写图片描述](http://img.blog.csdn.net/20160301110427351)

然后添加Add all。
4.打开options->key assignments，在command中过滤出 macro开头的（宏）。
![这里写图片描述](http://img.blog.csdn.net/20160301111010510)

找到super开头的几个宏，分别绑定对应的按键。如superbackspace -> assign new key 然后按下 键盘退格键（backspace）。完成。

### 总结
上述方法将me分别进行创建，方便管理，但是也可以放入一个me文件中。道理都是一样的。

### THE END
