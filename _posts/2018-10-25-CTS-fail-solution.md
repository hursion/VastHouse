---
layout: post
title: GMS认证问题分析收集
category: Android CTS|GTS|STS|VTS
comments: true

---

#CTS篇

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

# GTS篇
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
	at com.google.android.pm.gts.PackageManagerHostTest.testMediaProjection(PackageManagerHostTest.java:286)```
```
# VTS篇

# STS篇

# 豁免项
[unisoc 咨询问题列表](http://eip.unisoc.com/sites/pld/_layouts/15/WopiFrame2.aspx?sourcedoc=/sites/pld/CTSGoogle/CTS_google%E5%92%A8%E8%AF%A2%E9%97%AE%E9%A2%98%E5%88%97%E8%A1%A8.xlsx&action=default)

[google git-TR check](https://android-review.googlesource.com/q/project:platform/cts)
