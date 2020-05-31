---
title: 小米5sPlus安装原生系统
date: 2020-05-31 21:20:42
tags: Android 
summary: 小米5s Plus安装原生系统
---

<!-- toc -->


>手机型号：***MI 5s Plus***&emsp; &emsp; codeName：**natrium**

## 第一步：先解锁手机 

1. 登录网站[http://www.miui.com/unlock/index.html](http://www.miui.com/unlock/index.html)，点击立即解锁。

2. 提交申请成功后，等待审核通过。(一般申请后马上通过)

3. 审核通过后即获解锁资格，然后点击解决工具下载。

4. 将下载好的工具压缩包解压，并点击miflash_unlock.exe运行解锁工具。

5. 然后设置手机进入Bootloader模式（关机后，同时按住开机键和音量下键）。

6. 通过USB连接手机，点击 “解锁”按钮，过程中需要验证小米账号(保证手机登录了有解锁资格的账号)

7. 解锁成功后，点击重启手机。

   *详细解锁教程可以参考[http://www.shuajizhijia.net/news/18550.html](http://www.shuajizhijia.net/news/18550.html)*

## 第二步：刷一个第三方recovery

1. 下载Recovery工具包，链接：[https://pan.baidu.com/s/1wARkj7nArWUlzrI7iLKPBA](https://pan.baidu.com/s/1wARkj7nArWUlzrI7iLKPBA) 密码：**ovkz**

2. 手机要进入FastBoot模式，进入的方法：在手机关机状态下，按住音量键-和电源键

3. 保证手机正常连接电脑

4. 然后把上面下载下来的recovery工具包在电脑上进行解压，然后在文件夹里找到【recovery-twrp一键刷入工具.bat】文件直接双击打开运行。

5. 过几秒安装完成后会自动打开recovery界面。

6. 这个版本是3.2.3，可以自动解密data分区

   *详细刷recovery教程可以参考[http://bbs.xiaomi.cn/t-25351609-2-o1#comment_top](http://bbs.xiaomi.cn/t-25351609-2-o1#comment_top)*

## 第三步： 获取Root权限

### 方法一、刷root包

1. 下载root包，链接：[https://pan.baidu.com/s/13PogRZX2ZUivdb_wOjuzug](https://pan.baidu.com/s/13PogRZX2ZUivdb_wOjuzug) 密码：**rl7v**

2. 将下载下来的压缩包解压，并将Root-Supersu_Pro_2.74.zip放到手机根目录下

3. 进入Recovery界面，点击安装，在根目录下选择压缩包。

4. 安装完成后，点重新启动。

### 方法二、 MIUI系统中获取

   1. 安全中心-->应用管理-->权限-->Root权限管理
   2. 点击获取root权限。

   

## 第四步： 获取ROM包

AospExtended：[https://downloads.aospextended.com/](https://downloads.aospextended.com/)

Lineageos：[https://download.lineageos.org/](https://download.lineageos.org/)

## 第五步： 安装ROM包

1. 将下好的ROM包放到手机根目录下
2. 打开手机的Recovery界面，点击安装，选择ROM包。
3. 手机卡在开机界面时，先进入Recovery界面再格式化data即可。

## 第六步：获取Root权限

### 方法一、使用Lineageos自带的补丁包

1. Lineageos的Root补丁包地址：[https://download.lineageos.org/extras](https://download.lineageos.org/extras)

2. 下载好对应CPU位数及系统版本的Root包后，将压缩包放到手机根目录下

3. 打开手机的Recovery界面，选择安装Root包。

   参考网址：[https://tieba.baidu.com/p/4961319803?share=9105&fr=share&red_tag=2572190043](https://tieba.baidu.com/p/4961319803?share=9105&fr=share&red_tag=2572190043)
   
   结果：**失败**

### 方法二、使用SuperSu

1. SuperSu的卡刷包地址：[https://www.lineageosrom.com/2018/09/lineage-os-16-root-android-pie-90-super.html](https://www.lineageosrom.com/2018/09/lineage-os-16-root-android-pie-90-super.html)

2. 下载好对应CPU位数及系统版本的Root包后，将压缩包放到手机根目录下

3. 打开手机的Recovery界面，选择安装Root包。

4. 下载SuperSu的app应用并安装。

   结果：**失败**

### 方法三、使用Magisk

1. magisk网址：[https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445)

2. 两种方法：

   * 支持 [TWRP](https://twrp.me/Devices/) ，直接进入Recovery界面，安装Magisk包。

   * 不支持TWRP，使用以下方法：

     1. 下载一个magisk manager安装包，并安装。
     2. 从你的刷机包中提取当前固件的 **boot.img** 文件，将它传入到安装了 Magisk Manager 的手机中
     3. 进入 Magisk Manager —— 安装（install）—— install —— 修补 boot 镜像文件
     4. 然后选择传入的 boot.img 文件进行生成，并将生成后的 Patchedboot.img （姑且这么命名） 传输到电脑上。
     5. 随后我们使用 Magisk 应用对 boot.img 进行重新打包：
     6. 打开命令行窗口
     7. 执行 `adb reboot bootloader` 进入 Bootloader 界面
     8. 执行 `fastboot boot Patchedboot.img` 来加载生成后的 boot 分区文件获取**临时 root**

     10. 此时进入系统，你会发现你已经成功安装了 Magisk（如果显示没有安装则为获取失败，请检查操作过程重新尝试），但这还不够，我们还得进入 Magisk Manager，选择安装（install）——install——Direct Install（直接安装）才能将临时 root 转换为永久 root。

***注意：安装Magisk之前确保没有安装过Super su，否则无法安装成功。若是之前安装过了，需要格式化data并重新刷系统，才能安装成功。***

参考网址：[https://sspai.com/post/53043](https://sspai.com/post/53043)

结果：**成功**

## 遇到问题

   1. WIFI显示叉号，提示无法联网，但是可以上网

      解决方法：

      * Android 7.0之前版本的系统，执行以下命令：

        `adb shell "settings put global captive_portal_server captive.lineageos.org.cn"`

      * **Android 7.0之后**的版本需要执行下面的两条命令：

        `adb shell "settings put global captive_portal_http_url http://captive.lineageos.org.cn/generate_204";`

        `adb shell "settings put global captive_portal_https_url https://captive.lineageos.org.cn/generate_204"`

      参考网址：[https://www.lineageos.org.cn/thread-118-1-1.html](https://www.lineageos.org.cn/thread-118-1-1.html)

   2. ***一定要科学上网，否则网速会让你绝望***

