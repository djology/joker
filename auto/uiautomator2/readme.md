```
UiAutomator是Google提供用来做Android自动化测试的一个Java库，基于Accessibility服务  
openatx/uiautomator2并不支持iOS测试，需要iOS自动化测试，可以转到openatx/facebook-wda
```
## Quick Start
```
准备一台Android测试机开启开发者选项，连接电脑
运行 pip3 install -U uiautomator2 安装uiautomator2
运行 python3 -m uiautomator2 init 安装包含httprpc服务的apk到手机上，1.3.0以后的版本，运行python代码
 u2.connect() 时就会自动推送这些文件
    
    import uiautomator2 as u2
    d = u2.connect() # connect to device
    print(d.info)

相关项目：
    设备管理平台 atxserver2 
    与adb进行交互的库 adbutils 
    运行在设备上的驻守程序，go开发，用于保活设备上相关服务 atx-agent 
    类似与uiautomatorviewer，辅助编辑器 weditor
```
## Installation
```
1. Install uiautomator2

        # Since uiautomator2 is still under development, you have to add --pre to install the development version
        pip install --upgrade --pre uiautomator2
        
        # Or you can install directly from github source
        git clone https://github.com/openatx/uiautomator2
        pip install -e uiautomator2

2. Install weditor (UI Inspector)

    uiautomator是独占资源，所以当atx运行的时候uiautomatorviewer是不可用的，为了减少atx频繁启停，开发了基于浏览器
    技术的weditor UI查看器  https://github.com/openatx/weditor
        
        pip install -U weditor
    
    命令行输入 weditor 会自动打开浏览器，输入设备的ip或序列号，点击connect即可
    
3. Install daemons to a device (Optional)

    执行下面的命令会自动安装所需要的设备端程序：uiautomator-server、atx-agent、openstf/minicap、openstf/minitouch
        
        # init 所有已经连接到电脑的设备
        python -m uiautomator2 init
        
        # 高阶用法
        # init and set atx-agent listen in all address
        python -m uiautomator2 init --addr :7912

4. AppetizerIO 脚本编辑器

    AppetizerIO 提供对uiautomator2的深度集成，可以图形化管理ATX设备
```        
## Connect to a device
```
1. Through WiFi
        import uimatomator2 as u2
        d = u2.connect('10.0.0.1') # alias for u2.connect_wifi('10.0.0.1')
        print(d.info)
2. Through USB
        import uiautomator2 as u2
        d = u2.connect('123456f') # alias for u2.conect_usb('123456f')
        print(d.info)
3. Through ADB WiFi
        import uiautomator2 as u2
        d = u2.connect_adb_wifi("10.0.0.1:5555")
        
        # Equals to 
        # + Shell: adb connect 10.0.0.1:5555
        # + Python: u2.connect_usb("10.0.0.1:5555")
```
## Command line
```
$device_ip 表示设备的ip地址
如需指定设备需要传入 --serial 
        如 python3 -m uiautomator2 --serial bff1234 <SubCommand>，SubCommand为子命令

1.0.3 Added：python3 -m uiautomator2 可以简写为 uiautomator2

screenshot：截图
        $ uiautomator2 screenshot screenshot.jpg
current：获取当前包名和activity
        $ uiautomator2 current
        {
            "package": "com.android.browser", 
            "activity": "com.uc.browser.InnerUCMobile",
            "pid": 28478            
        }
uninstall：卸载
        $ uiautomator2 uninstall <package-name> # 卸载一个包
        $ uiautomator2 uninstall <package-name-1> <package-name-2> # 卸载多个包
        $ uiautomator2 uninstall --all # 全部卸载
stop：停止应用
        $ uiautomator2 stop com.example.app # 停止一个app
        $ uiautomator2 stop --all # 停止所有的app
install：安装apk，apk通过URL给出（赞不能用）
healthcheck：健康检查（暂不能用）
doctor：检查uiautomator2无法使用的原因
        $ uiautomator2 doctor
```
## API Documents
```
New command timeout
    How long (in seconds) will wait for a new command from the client before assuming the client quit 
    and ending the uiautomator service (Default 3 minutes)
    配置Accessibility服务的最大空闲时间，超时将自动释放，默认3分钟
    
        d.set_new_command_timeout(300) # change to 5 minutes, unit seconds

Debug HTTP requests
    Trace HTTP requests and response to find out how it works
        
        >>> d.debug = True
        >>> d.info
        16:50:59.630 $ curl -X POST -d '{"jsonrpc": "2.0", "id": "a3d49565ca381239fb75afd89a038231", "method": "deviceInfo", "params": {}}' 'http://127.0.0.1:56295/jsonrpc/0'
        16:50:59.704 Response (73 ms) >>>{"jsonrpc":"2.0","id":"a3d49565ca381239fb75afd89a038231","result":{"currentPackageName":"com.android.systemui","displayHeight":1560,"displayRotation":0,"displaySizeDpX":411,"displaySizeDpY":937,"displayWidth":720,"productName":"Punisher_00WW","screenOn":false,"sdkInt":30,"naturalOrientation":true}}
        <<< END

Implicit wait
    Set default element wait time, unit seconds
    设置元素查找等待时间，默认20s
    
        d.imlicitly_wait(10.0) # 也可以通过d.settings['wait_timeout'] = 10.0 修改 
        d(text="settings").click() # if Settings button not show in 10s, UiObjectNotFoundError will raised
        print("wait timeout", d.implicity_wait()) # get default implicit wait
    
    This function will have influence on click, long_click, drag_to, set_text, clear_text, etc
```
## App management
```
This part showcases how to perform app management
Install an app

    We only support installing an APK from a URL
        
        d.app_install('http://some-domain.com/some.apk')
        
Launch an app

        # 默认是先通过atx-agent解析apk包的mainActivity，然后调用am start -n $package/$activity启动
        d.app_start("com.example.hello_world")
        
        # 使用monkey -p com.example.hello_world -c android.intent.category.LAUNCHER 1 启动
        # 该方法有副作用，会自动将手机的旋转锁定关掉
        d.app_start("com.example.hello_world", use_monkey=True) # start with package name

        # 通过指定main activity的方式启动应用，等价于调用am start -n com.example.hello_world/.MainActivity
        d.app_start("com.example.hello_world", ".MainActivity")

Stop an app
        
        # equivalent to `am force_stop`, thus you could lose data
        d.app_stop("com.example.hello_world")
        # equivalent to `pm clear`
        d.app_clear('com.example.hello_world')

Stop all running apps
        
        # stop all
        d.app_stop_all()
        # stop all app except for com.examples.demo
        d.app_stop_all(excludes=['com.example.demo'])

Get app info
    
        d.app_info("com.examples.demo")
        # expect output
        #{
        #        "mainActivty": "com.github.uiautomator.MainActivity", 
        #        "label": "ATX", 
        #        "versionName": "1.1.7", 
        #        "versionCode": 1001007, 
        #        "size": 1760809
        #}
        
        # save app icon
        img = d.app_icon("com.examples.demo")
        img.save("icon.png")

List all running apps
        
        d.app_list_running()
        # expect output
        # ["com.xxxx.xxxx", "com.github.uiautomator", "xxxx"]

Wait until app running
        
        pid = d.app_wait("com.example.android") # 等待应用运行，return pid(int)
        if not pid:
            print("com.example.android is not running")
        else: 
            print("com.example.android pid is %d" % pid)
        
        d.app_wait("com.example.android", front=True) # 等待应用前台运行
        d.app_wait("com.example.android", timeout=20.0) # 最长等待时间20s（默认）

Push and pull files
        
        

检查并维持设备端守护进程处于运行状态

Auto click permission dialogs （已弃用）

Open Scheme

Basic API Usages
    Shell commands
    Session
    Retrieve the device info
    Clipboard
    Key Events
    Gesture interaction with the device
    Scree-related
    Selector
    WatchContext
    Watcher
    Screenrecord
    Image match

Common Issues
    Stop UiAutomator

Article Recommended


```



## Global settings
```
```
##     Debug HTTP requests
##     Implicit wait
## App management
```


```
##     Install an app
##     Launch an app
##     Stop an app
##     Stop all running apps
##     Push and pull files
##     Auto click permission dialogs
##     Open Scheme
## UI automation
```


```
##     Shell commands
##     Session
##     Retrieve the device info
##     Key Events
##     Gesture interaction with the device
##     Screen-related
##     Selector
##     Watcher
##     Global settings
##     Input method
##     Toast
##     XPath
##     Screenrecord
##     Image match