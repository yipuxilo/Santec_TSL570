#  TSL570
  
  
###  导入库函数
  
 ```python
 import tkinter as tk
 import tkinter.ttk as ttk
 import pyvisa as visa
 ```
###  定义仪器各功能的大类，面向对象
  
  
```python
`class TSL570():`
  
    def __init__(self):
```
###  初始化，包括ip，后缀，资源库等
  
  
        self.visa_dll = 'c:/windows/system32/visa32.dll'
        self.rm = visa.ResourceManager()
        self.ip='192.168.101.100'
        self.resource='TCPIP0::{}::5000::SOCKET'.format(self.ip)
        self.inst = self.rm.open_resource(self.resource)
        self.inst.read_termination = '\r'
        self.inst.write_termination = '\r'
        self.identity={}
###  写命令
  
    def write(self, command):
  
        self.inst.write(command)
###  询问命令
  
    def query(self,command):
  
        output = self.inst.query(command)
        print(output)
        return output
###  连接设备，检验师傅连接成功
  
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
###  设置波长
  
    def set_wavelength(self, par):
  
        self.write(':WAV {}'.format(par))
        return self.query(':WAV?')
###  设置波长单位
  
    def set_wave_unit(self,par):
  
        if par=='thz' or par =='THz':
            temp=1
        elif par=='nm':
            temp=0
        else:
            return 'error'
        self.write(':WAV:UNIT {}'.format(temp))
        #   1:thz   ;   0:nm
        tem=self.query(':WAV:UNIT?')
        if tem==1:
            return 'THz'
        elif tem==0:
            return 'nm'
        else:
            return 'error'
###  设置微调值
  
    def set_fine(self,par):
  
        self.write(':WAV:FIN {}'.format(par))
        return self.query(':WAV:FIN?')
###  无法调微值
  
    def fineturning_disable(self):
  
        self.write(':WAV:FIN:DIS')
        print('Hane Finished')
###  设置频率
  
    def set_frequency(self, par):
  
Tune the laser to a new wavelength. 
If a value is not specified, return the current one. Units: THz.
  
        self.write(':FREQ {}'.format(par))
        return self.query(':FREQ?')
###  相干控制状态
  
0: Coherence control OFF
1: Coherence control ON
  
    def set_coherence_control(self,par):
  
        self.write(':COHC {}'.format(par))
        if par=='1':
            print('Coherence control ON')
        elif par=='0':
            print('Coherence control OFF')
        else:
            print('error')
###  激光输出状态
  
0: optical output OFF
1: optical output ON
  
    def set_power_status(self, par):
  
        self.write(':POW:STAT {}'.format(par))
        temp=self.query(':POW:STAT?')
        if temp==1 :
            print('optical output ON')
        else:
            print('optical output OFF')
###  激光单位
  
0: dBm
1: mW
  
    def set_power_unit(self,par):
  
        self.write(':POW:UNIT {}'.format(par))
        temp=self.query(':POW:UNIT?')
        if temp==1 :
            return 'mW'
        else:
            return 'dBm'
###  设置衰减值
  
Range: 0 〜30 (dB)
Step: 0.01 (dB)
  
    def set_attenuation(self,par):
  
        self.write(':POW:ATT {}'.format(par))
        return self.query(':POW:ATT?')
###  设置控制模式
  
0: Manual mode
1: Auto mode
  
    def set_control_mode(self,par):
  
        self.write(':POW:ATT:AUT {}'.format(par))
        temp=self.query(':POW:ATT:AUT?')
        if temp==1 :
            print('Auto mode')
        else:
            print('Manual mode')
###  设置激光强度
  
Range: -15dBm to +13dBm
Step: 0.01dB (0.01mW)
  
    def set_power_level(self, par):
  
        self.write(':POW {}'.format(par))
        return self.query(':POW?')
###  获取监测激光强度
  
Range: -15dBm to peak power
Step: 0.01dB (0.01mW)
  
    def read_monitored_power_level(self):
  
        return self.query(':POW:ACT?')
###  设置扫描范围
  
    def set_sweep_limits(self,cha):
        l = cha.split(',')
        i = l[0]
        uniti = l[1]
        f = l[2]
        unitf = l[3]
        if uniti=='nm' or uniti=='NM':
            self.write(':WAV:SWE:STARt {}'.format(i))
            start = self.query(':WAV:SWE:STARt?')
        elif uniti=='THz' or uniti=='thz':
            self.write(':FREQ:SWE:STAR {}'.format(i))
            start = self.query(':FREQ:SWE:STAR?')
        else:
            print('error')
        if unitf=='nm' or unitf=='NM':
            self.write(':WAV:SWE:STOP {}'.format(f))
            stop = self.query(':WAV:SWE:STOP?')
        elif unitf=='THz' or unitf=='thz':
            self.write(':FREQ:SWE:STOP {}'.format(f))
            stop = self.query(':FREQ:SWE:STOP?')
        else:
            print('error')
        print(start,uniti,'~',stop,unitf)
###  读取波长和频率范围
  
    def read_wave_range(self):
        LMIN=self.query(':WAV:SWE:RANG:MIN?')
        LMAX=self.query(':WAV:SWE:RANG:MAX?')
        FMIN=self.query(':FREQ:SWE:RANG:MIN?')
        FMAX=self.query(':FREQ:SWE:RANG:MAX?')
        return 'Frequency:{}~{}'.format(FMIN,FMAX),'Wavelength:{}~{}'.format(LMIN,LMAX)
###  设置扫描模式
  
    def set_sweep_mode(self, mode):
        MODE={'0': 'Step sweep mode and One way',
              '1': 'Continuous sweep mode and One way',
              '2'`: 'Step sweep mode and Two way',
              '3': 'Continuous sweep mode and Two way'}
        self.write(':WAV:SWE:MOD {}'.format(mode))
        temp=self.query(':WAV:SWE:MOD?')
        print(MODE[temp])
###  设置扫描速度
  
    def set_sweep_speeds(self, par):
        self.write(':WAV:SWE:SPE {}'.format(par))
        print(self.query(':WAV:SWE:SPE?'))
        return self.query(':WAV:SWE:SPE?')
###  设置扫描歩长
  
    def set_sweep_step(self,str):
        l=str.split(',')
        par=l[0]
        unit=l[1]
        if unit == 'nm' or unit == 'NM':
            self.write(':WAV:SWE:STEP {}'.format(par))
            num= self.query(':WAV:SWE:STEP?')
        elif unit == 'THz' or unit=='thz':
            self.write(':FREQ:SWE:STEP {}'.format(par))
            num = self.query(':FREQ:SWE:STEP?')
        else:
            return 'error'
        if unit=='nm' or unit=='NM':
            target='wavelength'
        else:
            target='frequency'
        print(target,num,unit)
###  设置停留时间
  
Range: 0 to 999.9 sec
Step: 0.1 sec
  
    def set_dwell_time(self, par):
  
        if int(par) > 999.9 or int(par) < 0:
            return 'error'
        self.write(':WAV:SWE:DWEL {}'.format(par))
        print(self.query(':WAV:SWE:DWEL?'))
###  设置扫描周期
  
 Range: 0 to 999
Step: 1
  
    def set_sweep_cycles(self, par):
  
        if int(par) > 999 or int(par) < 1:
            print('error')
            return 0
        self.write(':WAV:SWE:CYCL {}'.format(par))
        print(self.query(':WAV:SWE:CYCL?'))
###  读取扫描周期（进行时）
  
    def read_sweep_count(self):
  
        return self.query(':WAV:SWE:COUN?')
###  设置扫描延时
  
Range: 0 to 999.9 sec
Step: 0.1 sec
  
    def set_sweep_delay(self, par):
  
        if par > 999.9 or par < 0:
            print('error')
            return 0
        self.write(':WAV:SWE:DEL {}'.format(par))
        return self.query(':WAV:SWE:DEL?')
###  扫描
  
    def sweep(self, signal=1):
signal is true or false
  
        if signal:
            self.write(':WAV:SWE 1')
            return self.query(':WAV:SWE?')
        else:
            self.write(':WAV:SWE 0')
            return self.query(':WAV:SWE?')
###  再次扫描
  
    def sweep_repeat(self):
  
        self.write(":WAV:SWE:REP")
        print('Have Repeat')
###  激光是否能用
  
    def laser_output(self,signal=1):
1:ON
0:OFF
  
        if signal:
            self.write(':AM:STAT 1')
            print('ON')
            return self.query(':AM:STAT?')
        else:
            self.write(':AM:STAT 0')
            print('OFF')
            return self.query(':AM:STAT?')
###  波长偏移
  
Add the constant offset to the output wavelength
    def wavelength_offset(self,par):
Range: -0.1000 ~ 0.1000 (nm)
Step: 0.0001（nm）
  
        self.write(':WAV:OFFS {}'.format(par))
        return self.query(':WAV:OFFS?')
###  关闭仪器
  
    def device_shut_down(self):
        self.laser_output(0)
        self.write(':SPEC:SHUT')
###  重启仪器
  
    def device_restart(self):
        self.write(':SPEC:REBoot')
`TSL = TSL570()`
###  定义APP这个类，面向对象写方法
  
class MyApplication(tk.Frame):
  
    def __init__(self, master=None):
        super().__init__(master)
        self.master = master
###  调整窗口大小
  
        self.master.geometry("400x500")  
###  更改窗口标题
  
        self.master.title("TSL-570")  
###  设置主题（可有可无）
  
        style=ttk.Style()
        style.theme_create('my_theme',parent='alt',settings={'TButton':{'configure':{'background':'#BBEOE3','foreground':'black'}},
                                                             'TLabel':{'configure':{'background':'#BBE0E3'}}})
        style.theme_use('my_theme')
  
###  创建界面
  
        self.create_widgets()
###  创建界面的类方法
  
    def create_widgets(self):
####  添加一个菜单栏
  
        menubar = tk.Menu(self.master)
        self.master.config(menu=menubar)
####  添加 初始化 菜单
  
添加菜单用 tk.Menu
  
        file_menu = tk.Menu(menubar, tearoff=0)
        menubar.add_cascade(label="INITIALIZE", menu=file_menu)
        file_menu.add_command(label="Connect", command=self.connect)
菜单栏分隔线
  
        file_menu.add_separator()
        file_menu.add_command(label="Device Shut Down", command=TSL.device_shut_down)
        file_menu.add_command(label="Device Restart", command=TSL.device_restart)
        file_menu.add_separator()
        file_menu.add_command(label="Exit", command=self.master.quit)
  
####  主菜单栏
  
        file_menu = tk.Menu(menubar, tearoff=0)
        menubar.add_cascade(label="MAIN", menu=file_menu)
        file_menu.add_command(label="Set Wavelength", command=self.set_wavelength)
        file_menu.add_command(label="Set Wavelength Unit", command=self.set_wave_unit)
        file_menu.add_command(label="Set Frequency", command=self.set_frequency)
        file_menu.add_command(label="Set Coherence Control", command=self.set_coherence_control)
        file_menu.add_command(label="Set Power Unit", command=self.set_power_unit)
        file_menu.add_command(label="Set Attenuation", command=self.set_attenuation)
        file_menu.add_command(label="Set Control Mode", command=self.set_control_mode)
        file_menu.add_command(label="Set Power Level", command=self.set_power_level)
        file_menu.add_separator()
        file_menu.add_command(label="Set Sweep Limits", command=self.set_sweep_limits)
        file_menu.add_command(label="Set Sweep Mode", command=self.set_sweep_mode)
        file_menu.add_command(label="Set Sweep Speeds", command=self.set_sweep_speeds)
        file_menu.add_command(label="Set Sweep Step", command=self.set_sweep_step)
        file_menu.add_command(label="Set Dwell Time", command=self.set_dwell_time)
        file_menu.add_command(label="Set Sweep Delay", command=self.set_sweep_delay)
        file_menu.add_command(label="Set Sweep Speeds", command=self.set_sweep_speeds)
  
####  读取菜单栏
  
        file_menu = tk.Menu(menubar, tearoff=0)
        menubar.add_cascade(label="READ", menu=file_menu)
        file_menu.add_command(label="Read Monitored Power Level", command=self.read_monitored_power_level)
        file_menu.add_command(label="Read Wave Range", command=self.read_wave_range)
        file_menu.add_command(label="Read Sweep Count", command=self.read_sweep_count)
        file_menu.add_command(label="Read Sweep Speed", command=self.read_sweep_speed)
        file_menu.add_command(label="Read Frequency Sweep Step", command=self.read_frequency_sweep_step)
        file_menu.add_command(label="Read Sweep State", command=self.read_sweep_state)
  
####  操作菜单栏
  
        file_menu = tk.Menu(menubar, tearoff=0)
        menubar.add_cascade(label="OPERATION", menu=file_menu)
        file_menu.add_command(label="Set Wavelength Fine", command=self.set_wavelength_fine)
        file_menu.add_command(label="Fine turning Disable", command=TSL.fineturning_disable)
        file_menu.add_command(label="Sweep Repeat", command=TSL.sweep_repeat)
        file_menu.add_command(label="Wavelength Offset", command=self.wavelength_offset)
        file_menu.add_separator()
        file_menu.add_command(label="Power Output", command=self.power_output)
        file_menu.add_command(label="Sweep", command=self.sweep)
        file_menu.add_separator()
        file_menu.add_command(label="Query", command=self.query)
        file_menu.add_command(label="Write", command=self.write)
  
####  trigger菜单栏
  
        file_menu = tk.Menu(menubar, tearoff=0)
        menubar.add_cascade(label="TRIGGER", menu=file_menu)
        file_menu.add_command(label="Tigger State", command=self.trigger_state)
        file_menu.add_command(label="Trigger Input Active", command=self.trigger_input_active)
        file_menu.add_command(label="Trigger Standby Mode", command=self.trigger_standby_mode)
        file_menu.add_command(label="Soft Trigger", command=lambda :TSL.write(':TRIG:INP:SOFTr'))
        file_menu.add_command(label="Set Trigger Output State", command=self.set_trigger_output_state)
        file_menu.add_command(label="Read Trigger Output State", command=self.read_trigger_output_state)
        file_menu.add_command(label="Trigger Output Active", command=self.trigger_output_active)
        file_menu.add_command(label="Set Trigger Output Step", command=self.set_trigger_output_step)
        file_menu.add_command(label="Read Trigger Output Step", command=self.read_trigger_output_step)
        file_menu.add_command(label="Trigger Output Period Mode", command=self.trigger_output_period_mode)
        file_menu.add_command(label="Trigger Through Mode", command=self.trigger_through_mode)
  
添加一个画布
  
        self.canvas = tk.Canvas(self.master, width=400, height=300)
        self.canvas.pack()
  
添加一个标签框
  
        self.result_frame = ttk.LabelFrame(self.master, text='Results')
        self.result_frame.pack(fill='both', expand=300)
        self.result_label = ttk.Label(self.result_frame, text='')
        self.result_label.grid(row=0,column=0,padx=10,pady=10)
        self.result_label.pack(fill='both', expand=300)
  
###  各菜单栏生成套娃小窗口的函数，然后进一步添加控件
  
用tk.Toplevel生成子窗口
  
    def connect(self):
        re=TSL.connect()
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def set_wavelength(self):
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
同理生成如下套娃小窗口函数
  
    ```python
    def set_wave_unit(self):
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="unit")
        label1.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="nm", command=lambda:TSL.set_wave_unit('nm'))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="THz", command=TSL.set_wave_unit('THz'))
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_frequency(self):
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label = tk.Label(settings_window, text="frequency")
        label.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda:TSL.set_frequency(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_wavelength_fine(self):
  
        # 创建一个新窗口
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="wavelength fine")
        label1.pack(pady=10)
        label2 = tk.Label(settings_window, text=" -100.00 to +100.00")
        label2.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda:TSL.set_fine(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_coherence_control(self):
  
        # 创建一个新窗口
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="coherence_control")
        label1.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Coherence control OFF", command=lambda:TSL.set_coherence_control('0'))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Coherence control ON", command=lambda :TSL.set_coherence_control('1'))
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def power_output(self):
  
        # 创建一个新窗口
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="power status")
        label2 = tk.Label(settings_window, text="0: optical output OFF  1: optical output ON")
        label1.pack(pady=10)
        label2.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="laser on", command=lambda:TSL.set_power_status('1'))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="laser off", command=lambda :TSL.set_power_status('0'))
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_power_unit(self):
  
        # 创建一个新窗口
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="power unit")
        label1.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="dBm", command=lambda:TSL.set_power_unit('0'))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="mW", command=lambda :TSL.set_power_unit('1'))
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_attenuation(self):
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="attenuation")
        label2 = tk.Label(settings_window, text="Range: 0 〜30 (dB)  Step: 0.01 (dB)")
        label1.pack(pady=10)
        label2.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda:TSL.set_attenuation(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_control_mode(self):
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="control mode")
        label1.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Manual mode", command=lambda:TSL.set_control_mode('0'))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Auto mode", command=lambda :TSL.set_control_mode('1'))
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_power_level(self):
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="power level")
        label2 = tk.Label(settings_window, text="Range:-15dBm ~ +13dBm   Step:0.01dB (0.01mW)")
        label1.pack(pady=10)
        label2.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda:TSL.set_power_level(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_sweep_limits(self):
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="sweep_limits")
        label2 = tk.Label(settings_window, text="填写顺序:开始数值,单位,结束数值,单位(英文逗号隔开)")
        label1.pack(pady=10)
        label2.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda:TSL.set_sweep_limits(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_sweep_mode(self):
  
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x300")
  
        label1 = tk.Label(settings_window, text="sweep_mode")
        label2 = tk.Label(settings_window, text='0:Step sweep mode and One way' )
        label4 = tk.Label(settings_window, text="1:Continuous sweep mode and One way")
        label1.pack(pady=2)
        label2.pack(pady=0.5)
        label4.pack(pady=0.5)
        label3 = tk.Label(settings_window, text="2: Step sweep mode and Two way")
        label3.pack(pady=0.5)
        label5 = tk.Label(settings_window, text="3: Continuous sweep mode and Two way")
        label5.pack(pady=0.5)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda:TSL.set_sweep_mode(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def set_sweep_speeds(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x200")
  
        label1 = tk.Label(settings_window, text="sweep speeds")
        label1.pack(pady=10)
        label1 = tk.Label(settings_window, text="Range: 1 to 200 nm/s    Selection: 1,2,5,10,20,50,100,200 (nm/s)")
        label1.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.set_sweep_speeds(entry.get()))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def set_sweep_step(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x250")
  
        label1 = tk.Label(settings_window, text="sweep step")
        label1.pack(pady=10)
        label2 = tk.Label(settings_window, text="Range: 0.1pm to specified wavelength span   Step: 0.1 pm")
        label2.pack(pady=10)
        label3 = tk.Label(settings_window, text="fill parameter and unit('nm' or 'THz',splited with ',' in English)")
        label3.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.set_sweep_step(entry.get()))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def set_dwell_time(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="dwell time")
        label1.pack(pady=10)
        label2 = tk.Label(settings_window, text="Range: 0 to 999.9 sec    Step: 0.1 sec")
        label2.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.set_dwell_time(entry.get()))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def set_sweep_cycles(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="sweep cycles")
        label1.pack(pady=10)
        label1 = tk.Label(settings_window, text="Range: 0 to 999    Step:1")
        label1.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.set_sweep_cycles(entry.get()))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def set_sweep_delay(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="sweep delay")
        label1.pack(pady=10)
        label2 = tk.Label(settings_window, text="Range: 0 to 999.9 sec    Step: 0.1 sec")
        label2.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.set_sweep_delay(entry.get()))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def read_monitored_power_level(self):
        re = TSL.read_monitored_power_level
        self.result_label = tk.Label(self.result_label, text='{}'.format(re))
        self.result_label.pack(fill='both', expand=300)
  
    def read_wave_range(self):
        re=TSL.read_wave_range()
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def read_sweep_count(self):
        re = TSL.read_sweep_count()
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def read_sweep_speed(self):
        re = TSL.query(':WAV:SWE:SPE?')
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def read_frequency_sweep_step(self):
        re = TSL.query(':WAV:SWE:STEP?')
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def sweep(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="sweep")
        label1.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="sweep start", command=lambda: TSL.sweep(1))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="sweep stop", command=TSL.sweep(0))
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def read_sweep_state(self):
        state={'0': 'Stopped','1': 'Running','3': 'Standing by trigger','4': 'Preparation for sweep start'}
        re = state[TSL.query(':WAV:SWE?')]
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def wavelength_offset(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        label1 = tk.Label(settings_window, text="wavelength offset")
        label1.pack(pady=10)
        label2 = tk.Label(settings_window, text="constant offset to the output wavelength")
        label2.pack(pady=10)
        label3 = tk.Label(settings_window, text="Step: 0.0001（nm）")
        label3.pack(pady=10)
        label4 = tk.Label(settings_window, text="Range: -0.1000 ~ 0.1000 (nm)）")
        label4.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.wavelength_offset(entry.get()))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def query(self):
        # 创建一个新窗口
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label = tk.Label(settings_window, text="query command:")
        label.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Query", command=lambda:TSL.query(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def write(self):
        # 创建一个新窗口
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label = tk.Label(settings_window, text="write command:")
        label.pack(pady=10)
  
        entry=tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Write", command=lambda:TSL.write(entry.get()))
        button1.pack(side=tk.LEFT,padx=10,pady=10)
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT,padx=10,pady=10)
  
    def trigger_state(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("300x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="trigger")
        label1.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="enable", command=lambda: TSL.write(':TRIG:INP:EXT 1'))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="disable", command=lambda :TSL.write(':TRIG:INP:EXT 0'))
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def trigger_input_active(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x200")
  
        label1 = tk.Label(settings_window, text="trigger active")
        label1.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="high active/ Triggers at rising edge", command=lambda: TSL.write(':TRIG:INP:ACT 0'))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="low active/ Triggers at falling edge", command=lambda :TSL.write(':TRIG:INP:ACT 1'))
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def trigger_standby_mode(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="trigger_standby_mode")
        label1.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Normal operation mode", command=lambda: TSL.write(':TRIG:INP:STAN 0'))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Trigger standby mode", command=lambda :TSL.write(':TRIG:INP:STAN 1'))
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def set_trigger_output_state(self):
        state=['0: None','1: Stop','3: Start','4: Step']
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x250")
  
        label1 = tk.Label(settings_window, text="trigger output state")
        label1.pack(pady=10)
        label2 = tk.Label(settings_window, text=state)
        label2.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.write(':TRIG:OUTP {}'.format(entry.get())))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def read_trigger_output_state(self):
        state={'0': 'None','1': 'Stop','3': 'Start','4': 'Step'}
        re = state[TSL.query(':TRIG:OUTP?')]
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def trigger_output_active(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="trigger active")
        label1.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="high active/ Triggers at rising edge", command=lambda: TSL.write(':TRIG:OUP:ACT 0'))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="low active/ Triggers at falling edge", command=lambda :TSL.write(':TRIG:OUP:ACT 1'))
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def set_trigger_output_step(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x250")
  
        label1 = tk.Label(settings_window, text="trigger output step")
        label1.pack(pady=10)
        label2 = tk.Label(settings_window, text='Range：0.0001 〜 Maximum specified wavelength range (nm)')
        label2.pack(pady=10)
  
        entry = tk.Entry(settings_window)
        entry.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="Save", command=lambda: TSL.write(':TRIG:OUTP:STEP {}'.format(entry.get())))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="Exit", command=settings_window.destroy)
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def read_trigger_output_step(self):
        re = TSL.query(':TRIG:OUTP:STEP?')
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
  
    def trigger_output_period_mode(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="trigger output period mode")
        label1.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="periodic in wavelength", command=lambda: TSL.write(':TRIG:OUTP:SETT 0'))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="periodic in time", command=lambda :TSL.write(':TRIG:OUTP:SETT 1'))
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def trigger_through_mode(self):
        settings_window = tk.Toplevel(self.master)
        settings_window.title("Settings")
        settings_window.geometry("400x200")
  
        # 在新窗口中添加一些控件用于进一步的操作
        label1 = tk.Label(settings_window, text="trigger through mode")
        label1.pack(pady=10)
  
        button1 = tk.Button(settings_window, text="through mode off",
                            command=lambda: TSL.write(':TRIG:THR 0'))
        button1.pack(side=tk.LEFT, padx=10, pady=10)
  
        button2 = tk.Button(settings_window, text="through mode on", command=lambda: TSL.write(':TRIG:THR 1'))
        button2.pack(side=tk.RIGHT, padx=10, pady=10)
  
    def read_ip(self):
        re=TSL.query(':SYST:COMM:ETH:IPAD?')
        self.result_label = tk.Label(self.result_label, text=re)
        self.result_label.pack(fill='both', expand=300)
        print(re)
    ```
  
```python
root = tk.Tk()
app = MyApplication(master=root)
 app.mainloop()
```
  