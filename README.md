# TB321FU_PixelOS_15_Tangorpro · faceunlock 分支

> **这是人脸解锁分支, 不是 main 分支。** main 分支是普通版(V2–V6), 没有人脸解锁。
> 想要人脸解锁刷这个分支发布的 `V6-faceunlock` 镜像; 不要装人脸解锁就刷 main 分支的 V2–V6。
> 两份不能同时用: V6-faceunlock 已经把人脸解锁的改动 bake 进 system 镜像, 不需要装任何额外 KSU 模块。

修改自 [Mystic-GSI](https://sourceforge.net/projects/mystic-gsi-updates/), 原机型为 Pixel Tablet(代号 tangorpro),
底包 `Pixel-tangorpro-15-12366759-AB-20241019-MysticGSI`(Android 15, AP3A.241005.015)。

和 [TB321FU_PixelOS_15_Komodo](https://github.com/CharlesGool/TB321FU_PixelOS_15_Komodo) 相比: 底包是平板专用的, 不是手机的 GSI,
所以 UI 错位几乎没有; 这台机器不支持但系统仍然暴露在设置/状态栏里的功能也更少(V6 又进一步删掉了 Hub 底座、侧边指纹界面、Pixel 屏幕强光模式、UV 统计、Pixel 基带服务)。
另外**修好了自动亮度和蓝牙音频**, 用 EROFS 打包。

和本仓库 main 分支的 V6 相比: 加了人脸解锁(ParanoidSense + Megvii 旷视 SDK, WEAK/Class 2 生物识别, 走前置摄像头)。
其它(V2 的深度精简、V3 的两个崩溃修复、V4 的相机库挂载清理、V5 的横屏方向修复、V6 的硬件不支持组件删除、自动亮度、蓝牙耳机、
蓝牙通话修复)都在。

## 它和原版 ParanoidSense 的区别

PixelOS 原版的 ParanoidSense 要配 ROM 一起编, 用 Google 的 platform 签名, 以 `android.uid.system` 跑。
这个仓库里的是在不换 ROM 的前提下把它移植进 GSI, 做法:

- ParanoidSense 应用自签名、跑在普通 priv-app uid, 不再要 platform 签名
- 底包 `USE_BIOMETRIC_INTERNAL` 和 `MANAGE_BIOMETRIC` 两个权限都是 signature protection(不是 signature\|privileged),
  privapp 白名单给不了。改过的 `services.jar` 把生物识别服务里 57 处权限校验改走一个 helper:
  调用方是系统 priv-app `co.aospa.sense` 就放行
- `SenseService` 用 `BIND_FOREGROUND_SERVICE | BIND_INCLUDE_CAPABILITIES` 绑, 避开 CAMERA foreground-only 限制
- 录入界面去掉了 Google 的 setupdesign/setupcompat 依赖, 换成标准 Android 组件

源码、构建脚本和每一步的具体改法单独开源在 [paranoidsense-gsi-port](https://github.com/CharlesGool/paranoidsense-gsi-port), 可以移植到别的 Android 15 GSI。

## Bug

1. 和 main 分支一样: 蓝牙 LE 音频不可用(已关闭, 双模耳机走经典蓝牙); 微信语音通话不会使用蓝牙耳机的麦克风(Telegram 等正常)
2. 人脸解锁是 WEAK 生物识别(Class 2), 不如指纹/人脸 3D 强: 可以用来解锁和弱校验, **不能**用来授权支付或打开 Keystore 里带 StrongBox 的凭据

## 我修改了什么

1. 已签名; 可直接启动, 无需关闭 vbmeta 验证
2. 原 GSI 开机后为 SELinux 宽容, 我修复了这点, 现在开机 10s 后自动设置为 SELinux 严格
3. 修复 `build.prop`, 可被直接识别为正确的机型, 默认关闭 ADB 等调试用的功能
4. 深度精简: 桌面只保留 设置、Play 商店、Chrome、文件极客, 另外保留 Gboard。Google 相机、时钟、Google App、YouTube、
   Gmail、地图、相册等都删了, 需要的话从 Play 商店装回来(删了 Google App 后桌面搜索栏和圈定即搜要装回 Google 才能用)
5. 默认显示 WLAN 热点(Pixel Tablet 原本把热点入口藏了)
6. **修复自动亮度**: 加入 TB321FU 专用 overlay, 亮度曲线取自原厂 ZUI

   原因和改法见 [tb321fu-brightness-overlay](https://github.com/CharlesGool/tb321fu-brightness-overlay), 刷别的 GSI 也可以装它的 KSU 模块
7. **修复蓝牙耳机**: 部分耳机没声音、声音从平板扬声器出来的问题。A2DP 不再走高通硬件 offload, 改走原厂 vendor 的软件编码
8. **修复蓝牙通话**: 戴蓝牙耳机通话时一开始双方听不到、麦克风用的是平板麦克风的问题

   7、8 两项的原因和改法见 [tb321fu-bt-audio-fix](https://github.com/CharlesGool/tb321fu-bt-audio-fix), 刷别的 GSI 也可以装它的 KSU 模块
9. 使用 EROFS(lz4hc)打包, 镜像只有 1.8 GB
10. 修复 Pixel Tablet 专属服务在这台机器上反复停止运行的问题(设备连接服务; UWB 服务直接删掉, 这台机器没有 UWB)
11. 去掉 GSI 把相机库复制到可写 tmpfs 再挂到 /vendor 上的多余挂载(这个库在这台机器上不需要修改)
12. 修只能横屏的应用(B 站全屏等)画面方向反的问题
13. 删掉 TB321FU 硬件不支持的 Pixel 组件: Hub 模式/底座、侧边指纹界面、Pixel 屏幕强光模式和 UV 统计、Pixel 基带服务
14. **人脸解锁**(本分支独有): 内置 ParanoidSense + Megvii 旷视 SDK, 走前置摄像头做 WEAK 生物识别。详见上面"它和原版 ParanoidSense 的区别"

## 工作的硬件

1. 几乎所有! 包括:
2. **CPU / GPU**
3. **蓝牙 / Wi-Fi / WLAN 热点**
4. **自动亮度**
5. **高刷新率**(165Hz)
6. **扬声器 / 蓝牙耳机 / 蓝牙通话**
7. **前后相机**(需要自己装相机应用)
8. **自动旋转**(横屏和反向横屏方向正确)
9. **X 轴线性马达**
10. **USB-C 接口**(两个接口均可全速正常工作)
11. **人脸解锁**(走前置摄像头, 本分支独有)

## 如何刷入

1. 刷入官方系统 **ZUI 17.5.10.170**(蓝牙通话修复依赖这个版本的 vendor, 其他版本会自动跳过这一项)
2. 重启到 fastbootd
3. 像一个正常 GSI 一样刷入(从其他系统换过来建议清数据)
4. 刷入 KernelSU(可选, 本版本内置人脸解锁, 不需要再装人脸解锁 KSU 模块)
5. 享受它!

```
fastboot flash system Pixel-tangorpro-15_TB321FU_V6-faceunlock_erofs_raw_Signed.img
fastboot -w
fastboot reboot
```

刷机时如果报 `Write to device failed`, 多半是 USB 连接不稳, 重新插线再刷即可。
如果之前在 main 分支的 V5/V6 上装过人脸解锁的 KSU 测试模块 `tb321fu_faceunlock_v*.zip`, 刷本镜像前先卸载它(两份 services.jar 会冲突):
```
adb shell su -c "ksud module uninstall tb321fu_faceunlock"
adb reboot
```

## 怎么用人脸解锁

设置 → 安全与隐私 → 设备解锁 → 人脸解锁 → 输 PIN → 走完录入 → 以后锁屏对准前置摄像头就能解。
删除: 设置里 "人脸模型" → 删除。

## 更新日志

- V6-faceunlock(首发): 把人脸解锁 bake 进系统镜像(改过的 services.jar + ParanoidSense priv-app + 设置 overlay + 权限 xml), 不需要 KSU 模块。基础修复和 main 分支的 V6 一致: 删掉 TB321FU 硬件不支持的 Pixel 组件、横屏方向、相机库挂载清理、崩溃循环、深度精简、自动亮度、蓝牙耳机/通话修复
