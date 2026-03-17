---
title: "多人聊天室小项目学习笔记"
date: 2025-07-27
categories:
  - 杂
tags:
  - 笔记
---

# 多人聊天室小项目学习笔记

本项目是使用python实现多人聊天的功能，会使用到第三方模块wxPython，这个模块是用来实现图形化的，运行项目需要安装wxPython，在项目目录中安装即可

```cmd
pip install wxPython
```

此外，项目用到的其他模块都是Python里面内置的模块的。

## 第一步:界面设计

### 服务端界面：

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/967b64eaba284ebfa5637074d7de3fda.png)
具体代码：

```python
import wx # wxPython Python的图形化界面库
# 继承wx.Frame窗体
class Hhtserver(wx.Frame):
    def __init__(self,name):
        # 依次是父组件，id,标题，位置，大小
        wx.Frame.__init__(self,None,id=1002,title=name+"的服务器",pos=wx.DefaultPosition,size=(600,550))
        # 生成一个面板
        pl = wx.Panel(self)
        # 创建盒子 参数：垂直布局 vertical:垂直
        box = wx.BoxSizer(wx.VERTICAL)
        # 可伸缩的网格布局 水平布局
        fgz1 = wx.FlexGridSizer(wx.HSCROLL)
        # 创建三个按钮
        start_server_btn = wx.Button(pl, size=(200, 40), label='启动服务')
        save_btn = wx.Button(pl, size=(200, 40), label='保存聊天记录')
        stop_btn = wx.Button(pl, size=(200, 40), label='停止服务')
        #添加进网格布局中
        fgz1.Add(start_server_btn)
        fgz1.Add(save_btn)
        fgz1.Add(stop_btn)
        box.Add(fgz1)
        # 创建文本框，参数含义:父组件，大小，样式(多行文本框输入，只读)
        self.Chat_text = wx.TextCtrl(pl, size=(600, 220), style=wx.TE_MULTILINE | wx.TE_READONLY)
        # 将聊天框放进box中
        box.Add(self.Chat_text, 1, wx.ALIGN_CENTER)
        # 将box放进面板中
        pl.SetSizer(box)
        '''以上就是服务器的图形界面设计'''
if __name__ == '__main__':
    # 创建应用程序对象（必需）
    app=wx.App()
    server = Hhtserver('hht')
    server.Show()
    # 启动主事件循环（必需）
    app.MainLoop()


```

### 客户端界面：

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/c0919908ad184516bcd4c66553257179.png)

具体代码：

```python
import wx
class HhtClient(wx.Frame):
    def __init__(self,name):
        # None表示没有父窗体
        # id 表示窗体的编号
        # title表示窗体的标题
        # pos 表示窗体的位置
        # size表示窗体的大小 宽和长
        wx.Frame.__init__(self,None,id=1001,title=name+'的客户端',pos=wx.DefaultPosition,size=(400,500))
        #创建面板
        pl = wx.Panel(self)
        # 创建盒子 参数的垂直布局 vertical:垂直
        box = wx.BoxSizer(wx.VERTICAL)
        # 可伸缩的网格布局 水平布局
        fgz1 = wx.FlexGridSizer(wx.HSCROLL)

        #创建两个按钮
        conn_btn = wx.Button(pl,size=(200,40),label='连接')
        dis_conn_btn = wx.Button(pl,size=(200,40),label='断开')

        #将两个按钮放入网格布局中
        fgz1.Add(conn_btn,1,wx.Top|wx.LEFT)
        fgz1.Add(dis_conn_btn,1,wx.Top|wx.RIGHT)

        # 将网格布局添加到box中
        box.Add(fgz1,wx.ALIGN_CENTER)

        # 只读文本框
        self.show_text= wx.TextCtrl(pl,size=(400,280),style=wx.TE_MULTILINE|wx.TE_READONLY)
        box.Add(self.show_text,1,wx.ALIGN_CENTER)

        # 聊天框
        self.Chat_text = wx.TextCtrl(pl,size=(400,120),style=wx.TE_MULTILINE)
        box.Add(self.Chat_text,1,wx.ALIGN_CENTER)

        # 第二个网格布局
        fgz2 = wx.FlexGridSizer(wx.HSCROLL)

        # 创建两个按钮
        reset_btn = wx.Button(pl, size=(200, 40), label='重置')
        send_btn = wx.Button(pl, size=(200, 40), label='发送')

        # 将两个按钮放入网格布局中
        fgz2.Add(reset_btn, 1, wx.Top | wx.LEFT)
        fgz2.Add(send_btn, 1, wx.Top | wx.RIGHT)

        # fgz1放入box中
        box.Add(fgz2)
        pl.SetSizer(box)


if __name__ == '__main__':
    # 初始化App()
    app = wx.App()
    # 创建自己的客户端
    client = HhtClient('hht')
    client.Show()
    app.MainLoop()
```

## 第二步：实现多人通信的思路

从两个人的通信拓展到多人通信，如果只是两个人通信的话，那么两个程序就可以了，直接使用网络编程，引入包socket，你一句我一句的接受和发送就可以了，那多个人之间如何才相互通信，我们不可能让多个人之间一个对一个的相互通信，我们需要一个中间人，这个中间人是独立于其他人的存在，只是负责接送每个人的数据，并发送给每个人。这个中间人就是服务器，服务会接受所有人的信息，然后将所有的信息发送到每个人的客户端，这样一个多人在线群聊就实现了。

上面只是实现多人通信最粗略的思路，接下来的就是如何让这个服务器可以接受所有人的信息，我们采用的是TCP网络协议，一对一的通信，就是服务器与每个人都是一对一的通信。具体图解如下：

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/1211155b0e1647948e26ff3b51c27cf7.png)

因为TCP是一对一的，所以每个客户可服务器之间通信应该在一个独立的线程里面，这样就相当于每个人在和服务器通信，但服务器会告知所有人信息，这样我们的线程设计就是，有一个主线程用来创建用户和服务器的通信线程，多个子线程让用户和服务器保持通信，最会每个用户会接受从服务器发送的所有信息，并显示到只读聊天框中。

### 客户端代码：

在\_\_init\_\_()中

```python
		'''---------------服务器端的连接--------------'''
    	# 绑定事件
        self.Bind(wx.EVT_BUTTON,self.connect_to_server,conn_btn)
        self.client_name = name
        self.isConnect = False # 连接服务器的状态
        self.client_socket = None # 设置客户端的socket对象为空
```

连接函数connect_to_server（）

```python
    def connect_to_server(self,event):
        if self.isConnect:
            return
        # 创建套接字
        self.client_socket = socket.socket(AF_INET,SOCK_STREAM)
        # 试图连接
        server_host_port = ('127.0.0.1',8888)
        self.client_socket.connect(server_host_port)
        # 连接成功就发送一条数据
        self.client_socket.send(self.client_name.encode('utf-8'))
        # 启动一个线程与服务器的会话线程进行对话
        client_thread = threading.Thread(target=self.recv_data)
        # 设置守护线程
        client_thread.daemon = True
        #修改连接状态
        self.isConnect = True
        # 启动线程
        client_thread.start()
        print(f'客户端{self.client_name}已经连接成功')

    # 接受服务器的信息
    def recv_data(self):
        while self.isConnect:
            # 接受信息
            data = self.client_socket.recv(1024).decode('utf-8')
            # 显示到文本框中
            self.show_text.AppendText('-'*40+'\n'+data+'\n')
        self.client_socket.close()
```

### 服务端代码：

为开启服务的按钮绑定事件，在\_\_init\_\_()中绑定

```python
'''-----------------------服务器启动的必要属性-----------------------'''
        self.isOn= False
        self.host_port =('127.0.0.1',8888)
        #服务器套接字
        self.server_socket = socket(AF_INET,SOCK_STREAM)
        #绑定端口
        self.server_socket.bind(self.host_port)
        # 监听
        self.server_socket.listen(5)
        # 创建一个字典存储与客户端对话的会话线程
        self.session_thread_dict={} # key_value {客户端的名称key:会话线程value}
        # 为事件绑定函数
		self.Bind(wx.EVT_BUTTON,self.start_server,start_server_btn)
```

函数start_server()

```python
    def start_server(self,event):
        if not self.isOn: # 等价于self.isOn==False
            # 启动服务器
            self.isOn =True
            # 创建主线程，
            main_thread = threading.Thread(target=self.do_work)
            # 设置为守护线程
            main_thread.daemon =True
            # 启动主线程
            main_thread.start()

    #创建子线程
    def do_work(self):
        while self.isOn:
            print(self.isOn)
            # 接受用户的连接请求，新的套接字对象（用来与客户端通信），  客户端的地址信息
            session_socket,client_addr = self.server_socket.accept()
            # 第一次传输的数据是用户的名称
            user_name = session_socket.recv(1024).decode('utf-8')
            # 创建会话线程
            session_thread = Session_thread(session_socket,user_name,self)
            # 添加进字典里面去
            self.session_thread_dict[user_name] = session_thread
            # 开启会话线程
            session_thread.start()
            # 显示信息并将所有的信息发送给所有的用户
            self.show_info_and_send_client('服务器信息：',f'欢迎{user_name}来到聊天室',time.strftime('%Y-%m-%d:%H:%M:%S',localtime()))
        # 如果sion为false，就要关闭server的套接字了
        self.server_socket.close()
```

Session_thread类

```python

class Session_thread(threading.Thread):
    def __init__(self,session_socket,user_name,server):
        threading.Thread.__init__(self)
        self.session_socket = session_socket
        self.user_name = user_name
        self.server = server
        self.isOn = True

    def run(self):
        print(f'客户端：{self.user_name}已经与服务器连接成功了，服务器启动了一个会话线程')
        while self.isOn:
            data = self.session_socket.recv(1024).decode('utf-8')
            if data =='hhtpassword':
                # 接受到这个的数据表示关闭会话
                self.isOn = False
                self.server.show_info_and_send_client('服务器通知',f'{self.user_name}离开聊天室',time.strftime('%Y-%m-%d:%H:%M:%S',localtime()))
            else:
                # 如果是其他的数据发送给服务器
                self.server.show_info_and_send_client(self.user_name,data,time.strftime('%Y-%m-%d:%H:%M:%S',localtime()))
        # 关闭会话线程的套接字
        self.session_socket.close()
        #清除字典里面的数据
        del self.server.session_thread_dict[self.user_name]
        print(f'{self.user_name}已经被清理')

```

## 第三步：完善功能

### 服务端：

将服务端接受到的信息显示到服务端的只读文本框并发送给每个用户

```python
	'''
        向服务器显示和向客户端发送信息
    '''
    def show_info_and_send_client(self,data_source,data,data_time):
        # 字符串拼接操作
        send_data = f'{data_source}:{data}\n时间：{data_time}\n'
        # 显示到服务端
        self.Chat_text.AppendText(send_data)
        # 向客户端发送信息
        for client in self.session_thread_dict.values():
            # 这个会话线程是打开的
            if client.isOn:
                # 给服务器发信息
                client.session_socket.send(send_data.encode('utf-8'))
```

为其他按钮绑定事件

```python
        self.Bind(wx.EVT_BUTTON,self.save_log,save_btn)
        self.Bind(wx.EVT_BUTTON,self.stop_server,stop_btn)
```

函数

```python
    def stop_server(self,event):
        print('服务器已经停止')
        self.isOn=False
        self.server_socket.close()
        #清理字典
        self.session_thread_dict.clear()
        print('服务器已完全停止')


    def save_log(self,event):
        data = self.Chat_text.GetValue()
        with open('log.text','a',encoding='utf-8') as file:
            file.write(data)
```

### 客户端：

为其他按钮绑定事件

```python
 		#为发送按钮绑定事件
        self.Bind(wx.EVT_BUTTON,self.send_to_server,send_btn)
        # 为断开连接的按钮绑定事件
        self.Bind(wx.EVT_BUTTON,self.dis_conn_to_server,dis_conn_btn)
        # 为重置按钮提供事件
        self.Bind(wx.EVT_BUTTON,self.reset_client,reset_btn)
```

函数

```python
	#重置
    def reset_client(self,event):
        self.Chat_text.Clear()
	# 断开
    def dis_conn_to_server(self,event):
        self.client_socket.send('hhtpassword'.encode('utf-8'))
        self.isConnect =False
        # 直接关闭套接字
        self.client_socket.close()
	# 发送
    def send_to_server(self,event):
        # 是否连接状态
        if self.isConnect:
            # 获取聊天框的数据
            input_data = self.Chat_text.GetValue()
            if input_data!='':
                # 发送到服务器，服务器会将数据显示到各个客户端
                self.client_socket.send(input_data.encode('utf-8'))
                # 将输入框清空
                self.Chat_text.SetValue('')
```

## 总结：

虽然这个小项目功能比较单一，就是一个聊天室的功能，但是这个项目对于巩固网络编程和多线程的知识帮助还是挺大的。可以帮助你初步了解搭建多人聊天室的基本思路。

## 总的代码：

### 服务端代码：

```python

# 多人聊天室小项目
import threading #引入线程的包，因为每个客户端发消息就代表了一个线程
import time # 引入时间的包，需要记录发送消息的时间，和时间类的操作
from time import localtime

import wx # wxPython Python的图形化界面库
from socket import socket,AF_INET,SOCK_STREAM # 网络编程的套接字，网络协议，TCP协议
# 继承wx.Frame窗体
class Hhtserver(wx.Frame):
    def __init__(self,name):
        # 依次是父组件，id,标题，位置，大小
        wx.Frame.__init__(self,None,id=1002,title=name+"的服务器",pos=wx.DefaultPosition,size=(600,550))
        # 生成一个面板
        pl = wx.Panel(self)
        # 创建盒子 参数的垂直布局 vertical:垂直
        box = wx.BoxSizer(wx.VERTICAL)
        # 可伸缩的网格布局 水平布局
        fgz1 = wx.FlexGridSizer(wx.HSCROLL)
        # 创建三个按钮
        start_server_btn = wx.Button(pl, size=(200, 40), label='启动服务')
        save_btn = wx.Button(pl, size=(200, 40), label='保存聊天记录')
        stop_btn = wx.Button(pl, size=(200, 40), label='停止服务')
        #添加进网格布局中
        fgz1.Add(start_server_btn)
        fgz1.Add(save_btn)
        fgz1.Add(stop_btn)
        box.Add(fgz1)
        # 创建文本框，父组件，大小，样式：多行文本框输入，只读
        self.Chat_text = wx.TextCtrl(pl, size=(600, 220), style=wx.TE_MULTILINE | wx.TE_READONLY)
        # 将聊天框放进box中
        box.Add(self.Chat_text, 1, wx.ALIGN_CENTER)
        # 将box放进面板中
        pl.SetSizer(box)
        '''-----------------------服务器启动的必要属性-----------------------'''
        self.isOn= False
        self.host_port =('127.0.0.1',8888)
        #服务器套接字
        self.server_socket = socket(AF_INET,SOCK_STREAM)
        #绑定端口
        self.server_socket.bind(self.host_port)
        # 监听
        self.server_socket.listen(5)
        # 创建一个字典存储与客户端对话的会话线程
        self.session_thread_dict={} # key_value {客户端的名称key:会话线程value}
        # 为事件绑定函数
        self.Bind(wx.EVT_BUTTON,self.start_server,start_server_btn)
        self.Bind(wx.EVT_BUTTON,self.save_log,save_btn)
        self.Bind(wx.EVT_BUTTON,self.stop_server,stop_btn)

    def stop_server(self,event):
        print('服务器已经停止')
        self.isOn=False
        self.server_socket.close()
        #清理字典
        self.session_thread_dict.clear()
        print('服务器已完全停止')


    def save_log(self,event):
        data = self.Chat_text.GetValue()
        with open('log.text','a',encoding='utf-8') as file:
            file.write(data)

    def start_server(self,event):
        if not self.isOn: # 等价于self.isOn==False
            # 启动服务器
            self.isOn =True
            # 创建主线程，
            main_thread = threading.Thread(target=self.do_work)
            # 设置为守护线程
            main_thread.daemon =True
            # 启动主线程
            main_thread.start()

    def do_work(self):
        while self.isOn:
            print(self.isOn)
            # 接受用户的连接请求，新的套接字对象（用来与客户端通信），  客户端的地址信息
            session_socket,client_addr = self.server_socket.accept()
            # 第一次传输的数据是用户的名称
            user_name = session_socket.recv(1024).decode('utf-8')
            # 创建会话线程
            session_thread = Session_thread(session_socket,user_name,self)
            # 添加进字典里面去
            self.session_thread_dict[user_name] = session_thread
            # 开启会话线程
            session_thread.start()
            # 显示信息
            self.show_info_and_send_client('服务器信息：',f'欢迎{user_name}来到聊天室',time.strftime('%Y-%m-%d:%H:%M:%S',localtime()))
        # 如果sion为false，就要关闭server的套接字了
        self.server_socket.close()


    '''
        向服务器显示和向客户端发送信息
    '''
    def show_info_and_send_client(self,data_source,data,data_time):
        # 字符串拼接操作
        send_data = f'{data_source}:{data}\n时间：{data_time}\n'
        # 显示到服务端
        self.Chat_text.AppendText(send_data)
        # 向客户端发送信息
        for client in self.session_thread_dict.values():
            # 这个会话线程是打开的
            if client.isOn:
                # 给服务器法信息
                client.session_socket.send(send_data.encode('utf-8'))



class Session_thread(threading.Thread):
    def __init__(self,session_socket,user_name,server):
        threading.Thread.__init__(self)
        self.session_socket = session_socket
        self.user_name = user_name
        self.server = server
        self.isOn = True

    def run(self):
        print(f'客户端：{self.user_name}已经与服务器连接成功了，服务器启动了一个会话线程')
        while self.isOn:
            data = self.session_socket.recv(1024).decode('utf-8')
            if data =='hhtpassword':
                # 接受到这个的数据表示关闭会话
                self.isOn = False
                self.server.show_info_and_send_client('服务器通知',f'{self.user_name}离开聊天室',time.strftime('%Y-%m-%d:%H:%M:%S',localtime()))
            else:
                # 如果是其他的数据发送给服务器
                self.server.show_info_and_send_client(self.user_name,data,time.strftime('%Y-%m-%d:%H:%M:%S',localtime()))
        # 关闭会话线程的套接字
        self.session_socket.close()
        #清除字典里面的数据
        del self.server.session_thread_dict[self.user_name]
        print(f'{self.user_name}已经被清理')




if __name__ == '__main__':
    app=wx.App()
    server = Hhtserver('hht')
    server.Show()
    app.MainLoop()

```

### 客户端代码：

```python
import socket
import threading
from socket import AF_INET,SOCK_STREAM
import wx
class HhtClient(wx.Frame):
    def __init__(self,name):
        # None表示没有父窗体
        # id 表示窗体的编号
        # title表示窗体的标题
        # pos 表示窗体的位置
        # size表示窗体的大小 宽和长
        wx.Frame.__init__(self,None,id=1001,title=name+'的客户端',pos=wx.DefaultPosition,size=(400,500))
        #创建面板
        pl = wx.Panel(self)
        # 创建盒子 参数的垂直布局 vertical:垂直
        box = wx.BoxSizer(wx.VERTICAL)
        # 可伸缩的网格布局 水平布局
        fgz1 = wx.FlexGridSizer(wx.HSCROLL)

        #创建两个按钮
        conn_btn = wx.Button(pl,size=(200,40),label='连接')
        dis_conn_btn = wx.Button(pl,size=(200,40),label='断开')

        #将两个按钮放入网格布局中
        fgz1.Add(conn_btn,1,wx.Top|wx.LEFT)
        fgz1.Add(dis_conn_btn,1,wx.Top|wx.RIGHT)

        # 将网格布局添加到box中
        box.Add(fgz1,wx.ALIGN_CENTER)

        # 只读文本框
        self.show_text= wx.TextCtrl(pl,size=(400,280),style=wx.TE_MULTILINE|wx.TE_READONLY)
        box.Add(self.show_text,1,wx.ALIGN_CENTER)

        # 聊天框
        self.Chat_text = wx.TextCtrl(pl,size=(400,120),style=wx.TE_MULTILINE)
        box.Add(self.Chat_text,1,wx.ALIGN_CENTER)

        # 第二个网格布局
        fgz2 = wx.FlexGridSizer(wx.HSCROLL)

        # 创建两个按钮
        reset_btn = wx.Button(pl, size=(200, 40), label='重置')
        send_btn = wx.Button(pl, size=(200, 40), label='发送')

        # 将两个按钮放入网格布局中
        fgz2.Add(reset_btn, 1, wx.Top | wx.LEFT)
        fgz2.Add(send_btn, 1, wx.Top | wx.RIGHT)

        # fgz1放入box中
        box.Add(fgz2)
        pl.SetSizer(box)

        #为发送按钮绑定事件
        self.Bind(wx.EVT_BUTTON,self.send_to_server,send_btn)
        # 为断开连接的按钮绑定事件
        self.Bind(wx.EVT_BUTTON,self.dis_conn_to_server,dis_conn_btn)
        # 为重置按钮提供事件
        self.Bind(wx.EVT_BUTTON,self.reset_client,reset_btn)

        '''---------------服务器端的连接--------------'''
        self.Bind(wx.EVT_BUTTON,self.connect_to_server,conn_btn)
        self.client_name = name
        self.isConnect = False # 连接服务器的状态
        self.client_socket = None # 设置客户端的socket对象为空

    def reset_client(self,event):
        self.Chat_text.Clear()

    def dis_conn_to_server(self,event):
        self.client_socket.send('hhtpassword'.encode('utf-8'))
        self.isConnect =False
        # 直接关闭套接字
        self.client_socket.close()

    def send_to_server(self,event):
        # 是否连接状态
        if self.isConnect:
            # 获取聊天框的数据
            input_data = self.Chat_text.GetValue()
            if input_data!='':
                # 发送到服务器，服务器会将数据显示到各个客户端
                self.client_socket.send(input_data.encode('utf-8'))
                # 将输入框清空
                self.Chat_text.SetValue('')

    def connect_to_server(self,event):
        if self.isConnect:
            return
        # 创建套接字
        self.client_socket = socket.socket(AF_INET,SOCK_STREAM)
        # 试图连接
        server_host_port = ('127.0.0.1',8888)
        self.client_socket.connect(server_host_port)
        # 连接成功就发送一条数据
        self.client_socket.send(self.client_name.encode('utf-8'))
        # 启动一个线程与服务器的会话线程进行对话
        client_thread = threading.Thread(target=self.recv_data)
        # 设置守护线程
        client_thread.daemon = True
        #修改连接状态
        self.isConnect = True
        # 启动线程
        client_thread.start()
        print(f'客户端{self.client_name}已经连接成功')

    # 接受服务器的信息
    def recv_data(self):
        while self.isConnect:
            # 接受信息
            data = self.client_socket.recv(1024).decode('utf-8')
            # 显示到文本框中
            self.show_text.AppendText('-'*40+'\n'+data+'\n')
        self.client_socket.close()

if __name__ == '__main__':
    # 初始化App()
    app = wx.App()
    # 创建自己的客户端
    client = HhtClient('hht')
    client.Show()

    #响应式刷新
    app.MainLoop()
```
