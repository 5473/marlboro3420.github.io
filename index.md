#前言
在上一篇[树莓派Android Things使用超声波测距模块HC-SR04](http://www.jianshu.com/p/9a6b059e0d79)文章中，简单的介绍了超声波模块的使用，本篇介绍下人体红外感应模块HC-SR501在刷入了Android Things的树莓派中的使用方法。由于上一篇已经介绍了树莓派和Android Things的基本信息，本篇将不再赘述，直接提供接线方法和代码，如果你没有基本的树莓派运行环境，建议看一下上一篇了解下。
#操作步骤
##准备以下物品
######硬件
- 树莓派 * 1（假定是一个可以启动的树莓派，包含电源线、8G以上的TF卡，已经刷入了Android Things。没有刷入的看[官方操作方法，包含（Linux、Mac、Windows）](https://developer.android.google.cn/things/hardware/raspberrypi.html)刷入即可）
- 杜邦线 若干
- HC-SR501人体感应模块 * 1
**HC-SR501人体感应模块简单介绍**
某度的文档[HC-SR501人体感应模块](https://wenku.baidu.com/view/4503a2409b6648d7c1c746fd.html?re=view)
![HC-SR501红外感应模块](http://upload-images.jianshu.io/upload_images/6753590-28cf4cde42e498e8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![HC-SR501红外感应模块针脚图](http://upload-images.jianshu.io/upload_images/6753590-b5a1948a7c51fd90.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

HC-SR501人体感应模块一共有三个针脚，比超声波测距少了一个，看一下三个针脚分别什么用途。
VCC：电源（Volt Current Condenser） 接树莓派5v电源即可。
OUT：输出针脚，当传感器收到感应时，从此针脚输出数据。
GND：接地线，接树莓派的Ground口即可。
######软件环境
- Android Things 系统镜像（[官网下载](https://developer.android.google.cn/things/preview/download.html)，选择Raspberry Pi的镜像）
- Android Studio
#将HC-SR501和树莓派连接起来
![树莓派3针脚图](http://upload-images.jianshu.io/upload_images/6753590-e0a5285b4a87901a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

VCC：我接入了树莓派4口。
OUT：我接入了树莓派7口，即BCM4。（PS：DEMO代码接收针脚也是BCM4，如果此处你换针脚，注意修改代码）
GND：接入树莓派39口。（PS：本次例子我买了一包电阻，所以GND多接了个10K阻值的电阻。）
![接线图1](http://upload-images.jianshu.io/upload_images/6753590-9874a6d6a624c122.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![接线图2](http://upload-images.jianshu.io/upload_images/6753590-94d3ee6dd5634eef.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![接线图3](http://upload-images.jianshu.io/upload_images/6753590-49bca1404796c228.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![总预览](http://upload-images.jianshu.io/upload_images/6753590-87ccdffbb73593cd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![外观图0.0](http://upload-images.jianshu.io/upload_images/6753590-538da24dffce98ab.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可能接线有点乱，再梳理一下。VCC（蓝线）、OUT（红线）都是直接接树莓派的，GND（黑线）接面包板接电阻后（白线）再接入的树莓派。多说一句，本次例子我多用了个盒子，是因为我在查资料的时候，有一篇文章说HC-SR501如果直接暴露在外可能会误报，所以找了个盒子装了起来。
#编译工程到树莓派上
将我写的[Android Things HC-SR04超声波模块测距Demo](https://github.com/marlboro3420/UltrasonicDome)克隆到本地，用的上一篇的工程，我在里面新建了一个Module。用Android Studio打开，将已经连接好传感器的树莓派电源打开，在Andorid Studio的Terminal中输入
`adb connect <ip-address>`
连接到你的树莓派，Run **hongwai Module**即可。如果成功当有人从传感器前经过的时候可以从LOG中看到结果。

![运行结果](http://upload-images.jianshu.io/upload_images/6753590-e1a84f324d1921de.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#关键代码
```package com.yan.hongwai;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;

import com.google.android.things.pio.Gpio;
import com.google.android.things.pio.GpioCallback;
import com.google.android.things.pio.PeripheralManagerService;

import java.io.IOException;

public class MainActivity extends Activity {
    private static final String TAG = MainActivity.class.getSimpleName();
    private Gpio mInputGpio;
    private final String pinInput = "BCM4";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.i(TAG, "Starting MainActivity");

        PeripheralManagerService peripheralManagerService = new PeripheralManagerService();
        try {
            mInputGpio = peripheralManagerService.openGpio(pinInput);//Echo针脚
            mInputGpio.setDirection(Gpio.DIRECTION_IN);//将引脚初始化为输入
            mInputGpio.setActiveType(Gpio.ACTIVE_HIGH);//设置收到高电压是有效的结果
            //注册状态更改监听类型 EDGE_NONE（无更改，默认）EDGE_RISING（从低到高）EDGE_FALLING（从高到低）
            mInputGpio.setEdgeTriggerType(Gpio.EDGE_BOTH);
            mInputGpio.registerGpioCallback(mGpioCallback);//注册回调
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    private GpioCallback mGpioCallback = new GpioCallback() {
        @Override
        public boolean onGpioEdge(Gpio gpio) {
            try {
                if (gpio.getValue()) {
                    Log.e("有人来了", gpio.getValue() + ":1111111111111");
                } else {
                    Log.e("没有人", gpio.getValue() + ":222222222222");
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            return true;
        }

        @Override
        public void onGpioError(Gpio gpio, int error) {
            super.onGpioError(gpio, error);
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mInputGpio != null) {
            try {
                mInputGpio.close();
            } catch (IOException e) {
                Log.w(TAG, "Unable to close mEchoGpio", e);
            } finally {
                mInputGpio = null;
            }
        }
        mInputGpio.unregisterGpioCallback(mGpioCallback);
    }
}
```