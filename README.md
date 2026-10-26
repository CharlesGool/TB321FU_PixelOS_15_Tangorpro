# TB321FU_PixelOS_15_Tangorpro

修改自 [Mystic-GSI](https://sourceforge.net/projects/mystic-gsi-updates/), 原机型为 Pixel Tablet(代号 tangorpro),
底包 `Pixel-tangorpro-15-12366759-AB-20241019-MysticGSI`(Android 15, AP3A.241005.015)。

和 [TB321FU_PixelOS_15_Komodo](https://github.com/CharlesGool/TB321FU_PixelOS_15_Komodo) 相比: 底包换成了平板机型,
**修好了自动亮度和蓝牙音频**, 并且深度精简, 使用 EROFS 打包。

Bug:

1. 蓝牙: LE 音频不可用(已关闭, 双模耳机走经典蓝牙); 微信语音通话不会使用蓝牙耳机的麦克风(Telegram 等正常)

我修改了什么:

1. 已签名; 可直接启动, 无需关闭 vbmeta 验证
2. 原 GSI 开机后为 SELinux 宽容, 我修复了这点, 现在开机 10s 后自动设置为 SELinux 严格
3. 修复 `build.prop`, 可被直接识别为正确的机型, 默认关闭 ADB 等调试用的功能
4. 深度精简: 桌面只保留 设置、Play 商店、Chrome、文件极客, 另外保留 Gboard。Google 相机、时钟、Google App、YouTube、
   Gmail、地图、相册等都删了, 需要的话从 Play 商店装回来(删了 Google App 后桌面搜索栏和圈定即搜要装回 Google 才能用)
5. 默认显示 WLAN 热点(Pixel Tablet 原本把热点入口藏了)
6. **修复自动亮度**: 加入 TB321FU 专用 overlay, 亮度曲线取自原厂 ZUI
7. **修复蓝牙耳机**: 部分耳机没声音、声音从平板扬声器出来的问题。A2DP 不再走高通硬件 offload, 改走原厂 vendor 的软件编码
8. **修复蓝牙通话**: 戴蓝牙耳机通话时一开始双方听不到、麦克风用的是平板麦克风的问题
9. 使用 EROFS(lz4hc)打包, 镜像只有 1.9 GB

工作的硬件:

1. 几乎所有! 包括:
2. **CPU / GPU**
3. **蓝牙 / Wi-Fi / WLAN 热点**
4. **自动亮度**
5. **高刷新率**(165Hz)
6. **扬声器 / 蓝牙耳机 / 蓝牙通话**
7. **前后相机**(需要自己装相机应用)
8. **自动旋转**
9. **X 轴线性马达**
10. **USB-C 接口**(两个接口均可全速正常工作)

如何刷入:

1. 刷入官方系统 **ZUI 17.5.10.170**(蓝牙通话修复依赖这个版本的 vendor, 其他版本会自动跳过这一项)
2. 重启到 fastbootd
3. 像一个正常 GSI 一样刷入(从其他系统换过来建议清数据)
4. 刷入 KernelSU(可选)
5. 享受它!

```
fastboot flash system Pixel-tangorpro-15_TB321FU_V2_erofs_raw_Signed.img
fastboot -w
fastboot reboot
```

刷机时如果报 `Write to device failed`, 多半是 USB 连接不稳, 重新插线再刷即可。
