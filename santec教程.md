# Python版本
1.登录Python官网，下载（或者应用商店，如果有的话）
2.下载Windows 64 bit 3.11或3.10（注意3.11不支持wxpython，不过我用的tkinter库，所以建议大家都用tkinter库做gui界面，到时候好整合）
3.注意不要下载多个版本的Python，有些库不能共存

# Python编程环境
**1.默认环境：IDLE**
没有报错和提示，很不舒服，因此要下个编译环境，但是也建议下个IDLE（即Python官网的软件），它的优势是可以互动，打一行代码马上运行，马上反馈，而环境的Python得打完代码运行才能反馈

**2.pycharm（极力推荐）**
十分适合Python新手，打代码过程中就能提示你可能的问题，各板块用颜色区分很明显，软件包也可直接下载，还有各种插件和终端，我就是用的pycharm

**3.VScode**
虽然没有pycharm那么优秀，但可以支持任何你能想到的编程语言（如c++，Java），更适合开发者，里面的各种插件能让你节省不少时间，而且markdown也很适合以这个为环境,但注意要下载Python的插件

**4.Anaconda**
支持Windows，Mac，Linux。包含各种科学计算，数据分析的Python包，很适合科研

# Python包的安装方法
**1.用Anaconda安装**
```python
conda install jupyter notebook
```

**2.用pip安装（终端或cmd）**
`pip3 install （包名）`

**3.官网安装**
找到想装的包的官网，下载后放到Python下载路径的==site-package==包里

# 设备接口
1.usb接口，买USB转USB数据线接上就成功连接

2.串行接口，买USB转串口（com接口），但注意看针数

3.RJ45接口，通常叫网线，设备连接路由器，电脑联网即可控制

==建议统一网线方便==

4.光纤接口，连接光纤

5.GPIB接口，通用数据总线，注意接口公母区分

# 连接协议
**1.SCPI协议**
可编程仪器标准命令，如果用NI-VISA测试时可使用，适用大部分仪器

**2.LEGACY协议**
有些设备可能不支持，具体得看手册，Santec是支持的，而且我的gui界面就是用的LEGACY协议

# 调试方法
1.可先用NI-VISA连接后输入代码测试仪器是否连接成功，命令是否正常执行

2.测试程序时可以用断点一步步试哪一步出错，再找错误原因

# 示例代码

## **1.函数示例**

####  导入库函数
  
 ```python
 import tkinter as tk
 import tkinter.ttk as ttk
 import pyvisa as visa
 ```
####  定义仪器各功能的大类，面向对象
  
  
```python
`class TSL570():`
  
    def __init__(self):
```
####  初始化，包括ip，后缀，资源库等
  
  
        self.visa_dll = 'c:/windows/system32/visa32.dll'
        self.rm = visa.ResourceManager()
        self.ip='192.168.101.100'
        self.resource='TCPIP0::{}::5000::SOCKET'.format(self.ip)
        self.inst = self.rm.open_resource(self.resource)
        self.inst.read_termination = '\r'
        self.inst.write_termination = '\r'
        self.identity={}
####  写命令
  
    def write(self, command):
  
        self.inst.write(command)
####  询问命令
  
    def query(self,command):
  
        output = self.inst.query(command)
        print(output)
        return output
####  连接设备，检验师傅连接成功
  
    def connect(self):
  
        if self.inst=={}:
            a=input('please write an instance:')
            self.inst=self.rm.open_resource(a)
            output = self.query("*IDN?")[:-1].split(',')
            self.identity = {'name': output[0], 'model': output[1], 'SN': output[2]}
            return self.identity
        else:
            output = self.query("*IDN?")[:-1].split(',')
            self.identity = {'name': output[0], 'model': output[1], 'SN': output[2]}
            return self.identity
####  设置波长
  
    def set_wavelength(self, par):
  
        self.write(':WAV {}'.format(par))
        return self.query(':WAV?')

## **2.gui界面示例

####  调整窗口大小
  
        self.master.geometry("400x500")  
####  更改窗口标题
  
        self.master.title("TSL-570")  
####  设置主题（可有可无）
  
        style=ttk.Style()
        style.theme_create('my_theme',parent='alt',settings={'TButton':{'configure':{'background':'#BBEOE3','foreground':'black'}},
                                                             'TLabel':{'configure':{'background':'#BBE0E3'}}})
        style.theme_use('my_theme')
  
####  创建界面
  
        self.create_widgets()
####  创建界面的类方法
  
    def create_widgets(self):
####  添加一个菜单栏
  
        menubar = tk.Menu(self.master)
        self.master.config(menu=menubar)
####  添加 初始化 菜单
  
*添加菜单用 tk.Menu
  
        file_menu = tk.Menu(menubar, tearoff=0)
        menubar.add_cascade(label="INITIALIZE", menu=file_menu)
        file_menu.add_command(label="Connect", command=self.connect)
菜单栏分隔线
  
        file_menu.add_separator()
增加菜单栏

        file_menu.add_command(label="Device Shut Down", command=TSL.device_shut_down)
        file_menu.add_command(label="Device Restart", command=TSL.device_restart)
        file_menu.add_separator()
        file_menu.add_command(label="Exit", command=self.master.quit)
  
添加一个画布
  
        self.canvas = tk.Canvas(self.master, width=400, height=300)
        self.canvas.pack()
  
添加一个标签框
  
        self.result_frame = ttk.LabelFrame(self.master, text='Results')
        self.result_frame.pack(fill='both', expand=300)
        self.result_label = ttk.Label(self.result_frame, text='')
        self.result_label.grid(row=0,column=0,padx=10,pady=10)
        self.result_label.pack(fill='both', expand=300)
  
####  各菜单栏生成套娃小窗口的函数，然后进一步添加控件
  
*用tk.Toplevel生成子窗口

创建一个新窗口
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
在新窗口中添加一些控件用于进一步的操作
  
        label = tk.Label(settings_window, text="Wavelength")
        label.pack(pady=10)
添加文本框，供用户输入
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
添加按钮
''lambda:''用于实现有参数依赖的函数
  
        button1 = tk.Button(settings_window, text="Save", command=lambda:TSL.set_wavelength(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)

  