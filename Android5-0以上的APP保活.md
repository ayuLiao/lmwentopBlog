---
title: Android5.0以上的APP保活
date: 2017-02-23 16:33:15
tags: android
---

## 简介
APP进程保活一直是Android开发中的重要话题，谁都不想被用户干掉，所以会出现很多不见光的保活手法，这些手法在Android5.0以后大部分都被Google干死了，因为APP一直占用后台资源，这样就会导致Android手机越用越卡，Google当然不希望这种情况的发生，这里单纯的从技术角度来研究一下Android5.0以上的保活

## 进程优先级
为了让进程在Android系统中获得更久，就需要有更高的进程优先级，提高进程的优先级可以减少被Android系统杀掉的概率

1.前台进程(Forgroud process)
a.顶层Activity
b.跟顶层Activity绑定的Service
c.正在执行onReceive()函数的BroadCastReceiver

2.可见进程(Visible process)
a.被弹出对话框遮挡的Activity
b.跟被弹出对话框遮挡的Activity绑定的Service

3.服务进程(Service process)
a.正在运行的Service

4.后台进程(Background process)
a.不可见的Activity
b.没有正在运行的Service

5.空进程(Empty Process)
a.不包含Android中4大组件的进程
空进程的目的是快速启动一个进程

理解的进程优先级后，就可以想出一些保活的方法，防止进程被Android系统给杀掉

我可以想到的保活方法：
1.双Service层互相保活
2.前台透明一像素保活（提高进程优先级）
3.Android 5.0以上使用JobService保活
4.监听系统广播（网络变更广播、拍照广播、短信广播）来保活
5.开启软件无障碍功能来保活，该功能底层就有保活机制
6.利用系统的漏洞来启动一个前台的Service进程（该Service不会像正常前台进程那样显示出来）
7.多应用互相保活（微信、QQ都是这样做的，启动腾讯中的一款应用，其他产品应用的后台Service就会被唤醒）
8.开启软件自启动，防止一键被杀
9.使用Native进程保活，Android5.0以下可用（通过C/C++，fork出多个进程来保活）

本篇博客主要讲
Android 5.0以上使用JobService保活

## JobService
Google在Android5.0中引入了JobScheduler来执行满足特定条件且不紧急的后台任务，使用JobScheduler来执行后台任务可以减少电量的消耗。

要使用JobScheduler就必须构建出JobService来配置使用，JobService继承自Service，是一个抽象类，必须使用其中的两个抽象方法，onStartJob(JobParameters params)方法和onStopJob(JobParameters params)方法

onStartJob()方法在开始执行jobScheduler时被调用，可以将需要被执行的任务都写到该方法中，需要注意的是，在该方法中不能执行耗时任务，如果需要执行耗时任务可以开启一个新的线程（new Thread）来执行

onStopJob()方法在停止JobScheduler时被调用，在该方法中可以做一些扫尾工作，如释放一些资源

直接来看一下具体的代码
```java
package com.ayu.ayusaveapp;

import android.app.job.JobParameters;
import android.app.job.JobService;
import android.content.Intent;
import android.os.Build;
import android.os.Handler;
import android.os.Message;
import android.support.annotation.RequiresApi;
import android.util.Log;

//使用JobService必须在Android5.0以后
@RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
public class SaveService extends JobService {
    /**
     * Callback则是由工作线程内部传出接收到的消息的回调接口，其他线程通过Handler的sendMessage
     * 发送消息给工作线程后，工作线程就会通过Callback将接收到的消息通知给监听者。
     */
    private Handler handler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            JobParameters param = (JobParameters) msg.obj;
            jobFinished(param, true);
            Intent intent = new Intent(getApplicationContext(), MainActivity.class);
            //在新的栈中打开
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            startActivity(intent);
            return true;
        }
    });

    /**
     *需要重写，开始jobScheduler时被调用
     */
    @Override
    public boolean onStartJob(JobParameters params) {
        Log.e("ayuLiao", "onStartJob is running");
        Message msg = Message.obtain();
        msg.obj = params;
        handler.sendMessage(msg);
        return true;
    }

    /**
     * 停止JobScheduler时被调用
     */
    @Override
    public boolean onStopJob(JobParameters params) {
        //清除回调线程和消息
        handler.removeCallbacksAndMessages(null);
        return false;
    }

}
```
代码的逻辑比较简单，首先看到onStartJob()方法，在该方法中，先通过Message.obtain()方法构建出一个Message类对象，然后为Message类对象的obj字段赋值，将JobParameters类对象传递给它，JobParameters类对象包含了一些配置参数，这个类是不需要自己创建的，最后通过Handler的sendMessage()方法将Message类对象发送出去

接着看到相应的handleMessage()方法，使用该方法来接收sendMessage()方法发送的内容，在handleMessage()方法中，首先获得Message中的JobParameters类对象，然后调用jobFinished()方法，用于在任务执行完成后，通知系统释放相关的资源，接着构建Intent类对象，并通过addFlags()方法让Activity在新的任务栈中开启

最后就在AndroidManifest文件中对该Service进行声明
```xml
<service
    android:name=".SaveService"
    android:permission="android.permission.BIND_JOB_SERVICE">
</service>
```
注意要添加上**BIND_JOB_SERVICE权限**

接着我们编写一个类来使用JobService，具体代码如下

MainActivity.class
```java
package com.ayu.ayusaveapp;

import android.app.job.JobInfo;
import android.app.job.JobScheduler;
import android.content.ComponentName;
import android.content.Context;
import android.os.Build;
import android.support.annotation.RequiresApi;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {

    private Button btnClick;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        btnClick = (Button) findViewById(R.id.btnClick);
        btnClick.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.e("ayuLiao", "btnClick is click");
                // jobId每个Job任务的id
                int jobId = 1;
                // 指定你需要执行的JobService
                ComponentName name = new ComponentName(getPackageName(), SaveService.class.getName());

                /**
                 *  满足下面任意一个条件，就会执行JobService中onStartJob()方法中的代码
                 */
                //设置一些条件
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    JobScheduler jobScheduler = (JobScheduler) getSystemService(JOB_SCHEDULER_SERVICE);
                    JobInfo jobInfo = new JobInfo.Builder(jobId, name)
                            .setPeriodic(1000)//设置间隔时间
                            .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)//设置需要的网络条件，默认NETWORK_TYPE_NONE
							//.setMinimumLatency(2000)// 设置任务运行最少延迟时间
							//.setOverrideDeadline(50000)// 设置deadline，若到期还没有达到规定的条件则会开始执行
                            .setRequiresDeviceIdle(false)// 设置手机是否空闲的条件,默认false
                            .setPersisted(true)//设备重启之后你的任务是否还要继续执行
                            .build();
                    jobScheduler.schedule(jobInfo);
                }
            }
        });
    }
}
```
MainActivity类中代码也十分简单，看到onClick()方法，在该方法中首先构建出ComponentName类对象，用于指定需要执行的JobService，这里当然是我们前面编写的SaveService类，接着通过getSystemService()方法构建JobScheduler类，该类是Job的调度类，负责任务的执行、取消等逻辑，然后就再构建JobInfo类对象，通过该对象设置一下条件，只有满足其中的一个条件或多个条件，JobService中的逻辑才会被执行，如果没有设置条件，程序会抛出IllegalArgumentException，这些约束条件决定了JobService会在什么被执行，如果没有条件，Android系统就不知道什么时候执行它，当然会抛出错误，这里使用了setPersisted()方法设置设备重启之后继续执行任务，所以需要添加相应的权限，避免报错
```xml
 <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

这里要注意setPeriodic()设置了间隔时间，这里间隔1000毫秒调用一次onStartJob()方法，但是在Service的handleMessage()方法我们使用了jobFinished()方法，所以onStartJob()不会一直被调用

还需要注意，setOverrideDeadline()方法、setMinimumLatency()方法和setPeriodic()方法不能同时调用，不然程序会崩溃，所以这里将setMinimumLatency()方法和setOverrideDeadline()方法注释掉，

接着调用build()方法完成JobInfo类对象的构建，最后通过JobScheduler类的schedule()方法执行任务，若该方法返回RESULT_SUCCESS表示执行成功，返回RESULT_FAILURE表示执行失败


这样我们就通过JobService完成了进程的保活了，回顾一下JobService的用法：

1.继承JobService，从新onStartJob()方法和onStopJob()方法

2.在AndroidManifest文件中声明JobService，需要加上BIND_JOB_SERVICE权限

3.JobService不能执行耗时任务，所以通过Handler，让主线程来执行任务，或者可以开启一个新的Thread来执行任务

4.通过JobInfo设置约束条件，执行要设置一条约束条件

5.通过JobScheduler类的schedule()方法执行任务

## JobService杀不死的原因
JObService会在满足特定的约束条件后就会执行其中的onStartJob()方法，就算相应的进程被系统杀死，onStartJob()方法也会被执行，这里我们都知道了，可以是为什么呢？

通过Google，知道了简单的原理
在Android中会有很多系统级的进程，在启动系统级的进程时会相应的启动Android中关键性的服务，如Activity管理服务（ActivityManagerService，AMS）、包管理服务（PackageManagerServer，PMS）和这里需要用到的JobSchedulerService，系统级服务是不会被轻易杀死的，除非你自己用ROOT权限来作死，JobSchedulerService会使用bindServiceAsUser()的方法把实现了JobService的子类服务启动起来，并执行它的onStartJob()方法，这样onStartJob()方法中的逻辑就被执行了
源码层面的理解可以看下面这篇博客
[Android之JobScheduler运行机制源码分析](http://blog.csdn.net/zhangyongfeiyong/article/details/52130413)

## 结尾
这样就讲解完如何通过JobService来保活APP了，来看一下具体的效果

使用ADM来关闭程序，一关闭该进程，马上又会被开启
![保活JobService.gif](http://obfs4iize.bkt.clouddn.com/%E4%BF%9D%E6%B4%BBJobService.gif)

直接使用Android手机，将后台进程关闭，关闭该进程后，因为模拟器比较卡，等待一会，该进程又会被开启
![保活2.gif](http://obfs4iize.bkt.clouddn.com/%E4%BF%9D%E6%B4%BB2.gif)

<p align="center">用代码创造乐趣---ayuLiao</p>

![懒写作](http://obfs4iize.bkt.clouddn.com/%E6%87%92%E5%86%99%E4%BD%9C.jpg)