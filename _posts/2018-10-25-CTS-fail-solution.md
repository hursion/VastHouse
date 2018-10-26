---
layout: post
title: GMS认证问题分析收集
category: Android CTS|GTS|STS|VTS
comments: true

---

|Author|Hursion|
|---|---
|Email|private now|

目录
===== 
* [CTS篇](#CTS篇)
* [GTS篇](#GTS篇)
* [VTS篇](#VTS篇)

## CTS篇

**1.android.media.cts.MediaPlayerDrmTest#testCAR_CLEARKEY_AUDIO_DOWNLOADED_V0_SYNC	
和android.media.cts.RingtoneManagerTest#testAccessMethods fail**

root cause： 
测试项会往sd卡下载mp4文件，因客户persist.storage.type属性修改默认sd卡存储，导致无法正常写入，下载测试资源失败

solution： 
还原persist.storage.type属性

key log：
```
testCAR_CLEARKEY_AUDIO_DOWNLOADED_V0_SYNC报的是java.lang.NullPointerException: uri param can not be null.
testAccessMethods 报的是java.lang.NullPointerException: Attempt to invoke virtual method 'boolean java.lang.String.equals(java.lang.Object)' on a null object reference
```
```
I MediaPlayerDrmTestBase: Downloading file:file:///storage/D465-1BEB/Download/car_cenc-20120827-8c.mp4
I MediaDownloadManager: uri:http://storage.googleapis.com/wvmedia/cenc/clearkey/car_cenc-20120827-8c-pssh.mp4file:file:///storage/D465-1BEB/Download/car_cenc-20120827-8c.mp4 wait:600 Secs
W DownloadManager: Path appears to be invalid: /storage/D465-1BEB/Download/car_cenc-20120827-8c.mp4
E DownloadManager: [6] Failed: java.lang.IllegalArgumentException: Invalid UUID string: D465-1BEB
E DownloadManager: java.lang.IllegalArgumentException: Invalid UUID string: D465-1BEB
I MediaPlayerDrmTestBase: Downloaded file:file:///storage/D465-1BEB/Download/car_cenc-20120827-8c.mp4 id:6 uri:null
```
**2.CTS 7.0_r24 android.dpi.cts.ConfigurationScreenLayoutTest#testScreenLayout fail**

root cause：
testScreenLayout()要求四个方向旋转时都要返回相同的Configuration，因为客户机器的wm size为：720×1440，并且只修改了竖屏状态下的nav高度（"navigation_bar_height"），导致横竖屏report不一致。

solution： 
修改横屏状态的nav高度（"navigation_bar_width"）和竖屏相等

key log：

```
junit.framework.AssertionFailedError: Expected screen long value of 16 but got 32 for orientation 1 expected:<16> but was:<32>
at junit.framework.Assert.fail(Assert.java:50)
at junit.framework.Assert.failNotEquals(Assert.java:287)
at junit.framework.Assert.assertEquals(Assert.java:67)
at junit.framework.Assert.assertEquals(Assert.java:199)
at android.dpi.cts.ConfigurationScreenLayoutTest.testScreenLayout(ConfigurationScreenLayoutTest.java:58)
```
**3. com.android.cts.devicepolicy.MixedDeviceOwnerTest#testAudioRestriction fail和com.android.cts.devicepolicy.MixedProfileOwnerTest#testAudioRestriction  fail**
root cause: 
未设置默认铃声

solution： 
设置默认铃声

key log：
```
10-15 21:05:11.860 22257 22276 W MediaPlayer: Couldn't open content://0@settings/system/ringtone_cachesprd_ringtone: java.io.FileNotFoundException: Direct file access no longer supported; ringtone playback is available through android.media.Ringtone
10-15 21:05:11.861 22257 22276 W MediaPlayer: Couldn't open null: java.lang.NullPointerException: uri
10-15 21:05:11.883   415   631 E f: Couldn't open fd for content://settings/system/ringtone
10-15 21:05:11.883 22257 22276 E MediaPlayerNative: Unable to create media player
...
10-15 21:05:11.909 22257 22276 I TestRunner: ----- begin exception -----
10-15 21:05:11.910 22257 22276 I TestRunner: java.io.IOException: setDataSource failed.: status=0x80000000
10-15 21:05:11.910 22257 22276 I TestRunner: 	at android.media.MediaPlayer.nativeSetDataSource(Native Method)
```

CTS代码：
```
public void testDisallowAdjustVolume() throws Exception {
...
            mAudioManager.setStreamVolume(AudioManager.STREAM_RING, 1, /* flag= */ 0);
```
**4.android.security.cts.StagefrightTest#testStagefright_bug_68953854 ——fail**
root cause：
未合入google安全patch

solution：
合入google安全patch

key log：
```
10-24 12:50:45 I/ConsoleReporter: [1/1 armeabi-v7a CtsSecurityTestCases S5319A6xxxxx1] android.security.cts.StagefrightTest#testStagefright_bug_68953854 fail: junit.framework.AssertionFailedError: operation not completed within timeout of 60000ms
at junit.framework.Assert.fail(Assert.java:50)
at android.security.cts.StagefrightTest.runWithTimeout(StagefrightTest.java:962)
```
ps：
如何检查是否合入对应patch，首先确认fail的bug id，如此例中id为68953854，然后去google[安全公告网站](https://source.android.com/security/bulletin)搜索以上bug,然后根据参考内容找到patch，和问题代码做比较即可确认。

**5.android.jobscheduler.cts.TimingConstraintsTest#testJobParameters_unexpiredDeadline
**
root cause:
和wifi翻墙有关

solution:

建议可否较少人用vpn重新测试.
如果还是测试不过，请申请到sprd的实验室测试测试，以排除网路问题

key log:
```
10-11 15:06:05.804  1240  1240 I Finsky  : [2] com.google.android.finsky.scheduler.JobSchedulerEngine$PhoneskyJobSchedulerJobService.onStartJob(7): onJobSchedulerWakeup with jobId 9002
10-11 15:06:06.062   557  1898 I JobScheduler.Conn: Connectivity CHANGED for JobStatus{807ddbb #u0a33/9000 com.android.vending/com.google.android.finsky.scheduler.JobSchedulerEngine$PhoneskyJobSchedulerJobService u=0 s=10033 TIME=+16h58m50s422ms:+1d16h58m50s422ms NET=1 WAIT:APP_NOT_IDLE WAIT:DEV_NOT_DOZING}: usable=true connected=true validated=true metered=false unmetered=true notRoaming=true
10-11 15:06:06.065   557  1898 I JobScheduler.Conn: Connectivity CHANGED for JobStatus{a8a92d8 #u0a33/9001 com.android.vending/com.google.android.finsky.scheduler.JobSchedulerEngine$PhoneskyJobSchedulerJobService u=0 s=10033 TIME=+30s0ms:+22h52m19s684ms NET=2 CHARGING IDLE WAIT:APP_NOT_IDLE WAIT:DEV_NOT_DOZING}: usable=true connected=true validated=true metered=false unmetered=true notRoaming=true
10-11 15:06:19.122  4712  4730 I TestRunner: failed: testJobParameters_unexpiredDeadline(android.jobscheduler.cts.TimingConstraintsTest)
10-11 15:06:19.123  4712  4730 I TestRunner: ----- begin exception -----
10-11 15:06:19.124  4712  4730 I TestRunner: junit.framework.AssertionFailedError: Failed to execute non-deadline job
10-11 15:06:19.124  4712  4730 I TestRunner: 	at junit.framework.Assert.fail(Assert.java:50)
10-11 15:06:19.124  4712  4730 I TestRunner: 	at junit.framework.Assert.assertTrue(Assert.java:20)
10-11 15:06:19.124  4712  4730 I TestRunner: 	at android.jobscheduler.cts.TimingConstraintsTest.testJobParameters_unexpiredDeadline(TimingConstraintsTest.java:93)
10-11 15:06:19.124  4712  4730 I TestRunner: 	at java.lang.reflect.Method.invoke(Native Method)
```

## GTS篇

**1. com.google.android.pm.gts.PackageManagerHostTest#testMediaProjection fail**
root cause:
Instant App should be able to access Media Projection APIs

solution: 
合入以下google patch
[https://android-review.googlesource.com/c/platform/frameworks/base/+/732269](https://android-review.googlesource.com/c/platform/frameworks/base/+/732269)

key log:
```
10-24 18:04:00 I/ModuleListener: [1/1] com.google.android.pm.gts.PackageManagerHostTest#testMediaProjection fail:
junit.framework.AssertionFailedError: Instant App should be able to access Media Projection APIs.
Please apply patch r.android.com/732269
	at junit.framework.Assert.fail(Assert.java:57)
	at junit.framework.Assert.assertTrue(Assert.java:22)
	at junit.framework.TestCase.assertTrue(TestCase.java:192)
	at com.google.android.pm.gts.PackageManagerHostTest.testMediaProjection(PackageManagerHostTest.java:286)
```

**2.GtsMemoryHostTestCases - com.google.android.memory.gts.AllAppsMemoryHostTest#testPeakPssOfAllApps fail**

root cause：
此测项主要会启动项目内置的各个app, 并计算memory 的使用量, app 使用的memory 量不符规范即fail.
如单测此项发现fail, 建议多测数次确定fail 的原因皆相同.

常见的fail 有以下

1. 某app memory 使用量超过规范导致fail.
```
D/ModuleListener: ModuleListener.testFailed(com.google.android.memory.gts.AllAppsMemoryHostTest#testPeakPssOfAllApps, java.lang.AssertionError: com.aa.bbb/.MainActivity 99548, failed to keep to the max pss of 76800
```

2. 每次执行此测项总是在启动某app 后出现断开现象.
```
D/ModuleListener: ModuleListener.testFailed(com.google.android.memory.gts.AllAppsMemoryHostTest#testPeakPssOfAllApps, com.android.tradefed.device.DeviceUnresponsiveException: Attempted shell am start -W -S 'com.aa.bbb/.MainActivity' multiple times on device 0180531144859 without communication success. Aborting.
```

3. 每次执行此测项启动某app 总是失败.
```
D/ModuleListener: ModuleListener.testFailed(com.google.android.memory.gts.AllAppsMemoryHostTest#testPeakPssOfAllApps, java.lang.AssertionError: com.aa.bbb/.MainActivity failed to start
```

上述情形如果是第三方app 导致, 可以先将第三方app 取消内置再进行单测以确定是否为第三方app 问题.
如果确定为第三方app 问题则需由第三方修改或是协调取消内置.
如果是平台app 可提交请相关模块协助.

ps:
关于检查启动某app的memory状态判断，可由以下命令手动模拟获取
```
输入
adb shell am start -n com.clarodrive.android/com.owncloud.android.ui.splash.SplashActivity

启动后输入
adb shell dumpsys -t 30 meminfo --package com.clarodrive.android

检测反馈结果 TOTAL 的memory 量.
```
举个例子，可以看出如下TOTAL的memory为99415
```
$ am start -n com.clarodrive.android/com.owncloud.android.ui.splash.SplashActivity
clarodrive.android/com.owncloud.android.ui.splash.SplashActivity              <
Starting: Intent { cmp=com.clarodrive.android/com.owncloud.android.ui.splash.SplashActivity }
$ dumpsys -t 30 meminfo --package com.clarodrive.android
dumpsys -t 30 meminfo --package com.clarodrive.android
Applications Memory Usage (in Kilobytes):
Uptime: 5932737 Realtime: 15671620
** MEMINFO in pid 12706 [com.clarodrive.android] **
                   Pss  Private  Private  SwapPss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    13199    13052        0        0    19968    14463     5504
  Dalvik Heap     2798     2552      200       25     3481     2611      870
 Dalvik Other     1501     1500        0        1
        Stack       40       40        0        0
       Ashmem     7222     7064        0        0
    Other dev       23        0       20        0
     .so mmap     3310      160      940       47
    .apk mmap    38500      104    36284        0
    .ttf mmap      537        0      188        0
    .dex mmap    20129       16    18508        0
    .oat mmap     2465        0      504        0
    .art mmap     1218      708      176        1
   Other mmap     2275        4     2060        0
      Unknown     5412     5284      120      712
        TOTAL    99415    30484    59000      786    23449    17074     6374
 App Summary
                       Pss(KB)
                        ------
           Java Heap:     3436
         Native Heap:    13052
                Code:    56704
               Stack:       40
            Graphics:        0
       Private Other:    16252
              System:     9931
               TOTAL:    99415       TOTAL SWAP PSS:      786
 Objects
               Views:       19         ViewRootImpl:        2
         AppContexts:       11           Activities:        2
              Assets:        6        AssetManagers:        6
       Local Binders:       51        Proxy Binders:       48
       Parcel memory:       23         Parcel count:       92
    Death Recipients:        2      OpenSSL Sockets:        1
            WebViews:        1
 SQL
         MEMORY_USED:      109
  PAGECACHE_OVERFLOW:       66          MALLOC_SIZE:      117
 DATABASES
      pgsz     dbsz   Lookaside(b)          cache  Dbname
         4       16                        6/31/5  /data/user/0/com.clarodrive.android/databases/evernote_jobs.db
         4       24                       16/39/9  /data/user/0/com.clarodrive.android/databases/crash_reports
```



## VTS篇

## STS篇

## 豁免项

[unisoc 咨询问题列表](http://eip.unisoc.com/sites/pld/_layouts/15/WopiFrame2.aspx?sourcedoc=/sites/pld/CTSGoogle/CTS_google%E5%92%A8%E8%AF%A2%E9%97%AE%E9%A2%98%E5%88%97%E8%A1%A8.xlsx&action=default)

[google git-TR check](https://android-review.googlesource.com/q/project:platform/cts)

## 其他帮助
[CTSv2 最佳做法](https://docs.google.com/document/d/1HNf9dpDLrd82u6PabvrfFmpjmOlaysK1P2YoiFXjmEo/view#)

