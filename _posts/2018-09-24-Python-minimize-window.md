---

layout: post

title:  "Python中对窗口的操作"

categories: Python

tags:  pypiwin32 win32gui win32con win32api

---
在Windows下写了一个Python脚本用来启动一系列程序，在启动之后，桌面会显示这些打开的窗口，显得很乱，给人的感觉很不好，是否可以把这些窗口自动最小化呢？答案是可以的。需要用到win32 API，先找到对应的窗口，再设置窗口的显示属性

- ## 安装pypiwin32，管理员权限运行cmd
```
 pip install pypiwin32
```
安装完pypiwin32，就可以调用win32 API了，
```
import win32gui, win32api, win32con
```
- ## 找到对应窗口，以Notepad++为例
```python
notepad_win = win32gui.FindWindow("notepad++", None)
```
其中FindWindow函数原型（C++）为：
```c++
HWND FindWindowA(
  LPCSTR lpClassName,//类名，可利用Spy++查看，Spy++可在VS的工具栏打开
  LPCSTR lpWindowName//即窗口title，可为空
);
```
Python版本函数和C++一样
- ## 最小化窗口
```python
win32gui.ShowWindow(notepad_win , win32con.SW_MINIMIZE)
```
其中ShowWindow函数原型（C++）为：
```c++
BOOL ShowWindow(
  HWND hWnd,//FindWindow的返回值
  int  nCmdShow//窗口属性，属性比较多，具体可参考msdn
);
```