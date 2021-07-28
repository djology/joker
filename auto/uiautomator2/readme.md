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
        
    push a file to the device
        # push to a folder
        d.push("foo.txt", "/sdcard/")
        # push and rename
        d.push("foo.txt", "/sdcard/bar.txt")
        # push fileobj
        with open("foo.txt", 'rb') as f:
            d.push(f, "/sdcard/")
        # push and change file access mode
        d.push("foo.sh", "/data/local/tmp", mode=0o755)
    
    pull a file to the device
        d.pull("/sdcard/tmp.txt", "tmp.txt")
        
        # FileNotFoundError will raise if the file is not found on the device
        d.pull("/sdcard/some-file-not-exists.txt", "tmp.txt")

检查并维持设备端守护进程处于运行状态（已弃用）

        d.healthcheck()
        # DeprecationWarning: Call to deprecated method healthcheck. (method: healthcheck is useless now) -- Deprecated since version 3.0.0.

Auto click permission dialogs （已弃用）
    
    disable_popups 函数不稳定，暂不要使用，等候通知
        d.disable_popups() # automatic skip popups
        d.disable_popups(False) # disable automatic skip popups
    
    popup
    If this method is not working on your device, you can make a pull request or create an issue to 
    enhance this function: 
            1. Open uiautomatorviewer.bat
            2. Get popup hierarchy
    
    hierarchy
    Now you know the button text and current package name. Make a pull request by update function 
    disable_popups or create an issue if you are not familar with git and python. 

Open Scheme
    
    You can do it wire adb: adb shell am start -a android.intent.action.VIEW -d "appname://appnamehost"
    Also you can do it with python code
        d.open_url("https://www,baidu.com")
        d.open_url("taobao://taobao.com") # open Taobao app
        d.open_url("appname://appnamehost")
```
## Basic API Usages
```
This part showcases how to perform common device operations
Shell commands

    Run a short-lived shell command with a timeout protection (Default timeout 60s)
    Note: timeout support require atx-agent >= 0.3.3
    adb_shell function is deprecated. Use shell instead. 
        output, exit_code = d.shell("pwd", timeout=60) # timeout 60s (Default)
        # output: "/\n", exit_code: 0
        # Similar to command: adb shell pwd
        
        # Since `shell` function return type is `namedtuple("ShellResponse", ("output", "exit_code"))`
        # so we can do some tricks
        output = d.shell("pwd").output
        exit_code = d.shell("pwd").exit_code

    Run a long-running shell command
    add stream=True will return requests.models.Response object
        r = d.shell("logcat", stream=True)
        # r: requests.models.Response
        deadline = time.time() + 10 # run maxium 10s
        try: 
            for line in r.iter_lines(): # r.iter_lines(chunk_size=512, decode_unicode=None, delimiter=None)
                if time.time() > deadline:
                    break
                print("Read:", line.decode('utf-8'))
        finally:
            r.close() # this method must be called

    Command will be terminated when r.close() called

Session
    Session represent an app lifecycle. Can be used to start app, detect app crash
    Launch and close app
        sess = d.session("com.netease.cloudmusic") # start 网易云音乐
        sess.close() # 停止网易云音乐
        sess.restart() # 冷启动网易云音乐
    
    Use python with to launch and close app
        with d.session("com.netease.cloudmusic") as sess:
            sess(text="Play").click()
    
    Attack to the running app
        # launch app if not running, skip launch if already running
        sess = d.session("com.netease.cloudmusic", attack=True)
        
        # raise SessionBrokenError if not running
        sess = d.session("com.netease.cloudmusic", attack=True, strict=True)
    
    Detect app crash
        # When app is still running
        sess(text="Music").click() # operation goes normal
        
        # If app crash or quit
        sess(text="Music").click() # raise SessionBrokenError
        # other function calls under session will raise SessionBrokenError too
        
        # check if session is ok
        # Warning: function name may change in the future
        session.running() # True or False

Retrieve the device info

    Get basic information
        d.info
    Below is a possible output:
        {
            'currentPackageName': 'com.android.launcher3', 
            'displayHeight': 1560, 
            'displayRotation': 0, 
            'displaySizeDpX': 411, 
            'displaySizeDpY': 937, 
            'displayWidth': 720, 
            'productName': 'Punisher_00WW', 
            'screenOn': True, 
            'sdkInt': 30, 
            'naturalOrientation': True
        }
    
    Get window size
        print(d.window_size())
        # device upright output example: (1080, 1920)
        # device horizontal output example: (1920, 1080)
    
    Get current app info. For some android devices, the output could be empty
        print(d.app_current())
        # Output example 1: {'activity': '.Client', 'package': 'com.netease.example', 'pid': 23710}
        # Output example 2: {'activity': '.Client', 'package': 'com.netease.example'}
        # Output example 3: {'activity': None, 'package': None}
    
    Wait activity
        d.wait_activity(".ApiDemos", timeout=10) # default timeoput 10.0 seconds
        # Output: true or false
    
    Get device serial number
        print(d.serial)
        # output example: 74aAEDR428Z9
    
    Get WLAN ip
        print(d.wlan_ip)
        # output example: 10.0.0.1
    
    Get detailed device info
        print(d.device_info)
    
    Below is a possible output:
        {
            'udid': 'A00000P660152700116-5e:ce:73:a1:74:88-Nokia_G50', 
            'version': '11', 
            'serial': 'A00000P660152700116', 
            'brand': 'Nokia', 
            'model': 'Nokia G50', 
            'hwaddr': '5e:ce:73:a1:74:88', 
            'sdk': 30, 
            'agentVersion': '0.10.0', 
            'display': {'width': 720, 'height': 1640}, 
            'battery': {'acPowered': False, 
            'usbPowered': True, 
            'wirelessPowered': False, 
            'status': 5, 
            'health': 2, 
            'present': True, 
            'level': 100, 
            'scale': 100, 
            'voltage': 4348, 
            'temperature': 279, 
            'technology': 'Li-ion'}, 
            'memory': {'total': 5597172, 'around': '5 GB'}, 
            'cpu': {'cores': 7, 'hardware': ''}, 
            'arch': '', 
            'owner': None, 
            'presenceChangedAt': '0001-01-01T00:00:00Z', 
            'usingBeganAt': '0001-01-01T00:00:00Z', 
            'product': None, 
            'provider': None
        }

Clipboard

    Get of set clipboard content
    设置粘贴板内容或获取内容
        d.set_clipboard('text', 'label')
        print(d.clipboard)

Key Events

    Turn on/of screen
        d.screen_on() # turn on the screen
        d.screen_off() # turn off the screen

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