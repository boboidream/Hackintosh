# Hackintosh

主机配置为：
```
主板：技嘉 H97M-D3H，属于9系主板。声卡 Realtek ALC 892，板载 Realtek 千兆网卡
CPU：Intel E3-1231 v3，Haswell 架构
显卡：蓝宝石 RX 6600XT，仙后座 RDNA 2.0 Navi 核心（macOS 12.1beta 起免驱）
```

详情见文章：[使用OpenCore为H97M-D3H主板安装黑苹果Monterey系统](https://www.wenboz.com/p/9770.html)

## 一、修复睡眠问题

详情见文章：[解决几个电脑小问题](https://www.wenboz.com/p/6ce.html)

> Mac的sleep mode在os x系统里有一个准确的叫法是HibernateMode，它有三个值：0、1、3
> **Mode: 0**
> 当 HibernateMode 的值为 0 时 ，设备里除了 RAM（内存）外，键盘，显示器，鼠标等所有内外工作模块都会断开电源（或电池供应），此时系统不会将内存的数据写入硬盘，如果到设备被再次唤醒之前，电源线一直接入或者电池电量足够，那么用户在开盖后可立刻唤醒 Mac。
> 这种模式的优点明显，就是她不会向硬盘写数据，也就是设备在深度睡眠（一般成为休眠）时不会产生内存镜像，即能减少硬盘的占用率，也能让唤醒操作立刻完成。但请注意，许多事情有优点也有缺点，那就是当设备处于睡眠过程中时，电源线没插上，电池电量耗尽，那内存的供电就会自动中断，内存里保存的数据也会自动清除。OS X系统自身默认没有选择它。
> **Mode: 1**
> 当 HibernateMode 的值为 1 时，设备里所有模块均断电，内存数据被全部写入硬盘，硬盘里有一个专门负责“休眠”的内存镜像文件，当设备从“休眠”中恢复时，会自动调用保存好的内存镜像文件，将数据重新写回内存中，受硬盘的输入输出速率影响，这个过程会很漫长，所以许多朋友会在唤醒时看到屏幕里有进度条，千万别认为你的设备硬件不够用了，该换电脑了，别听奸商的忽悠，Mac 的产品寿命可比手机长多了，回到正题，在数据被完全写回内存后，Mac 才能被完全唤醒。这种模式优点明显，无需单独为内存供电，内存的数据不容易被丢失，缺点就是唤醒时间较长。
> **Mode: 3**
> 第三项值就是 OS X 默认选择的，这种状态下也叫：“**Safe Sleep**”，人们还叫她是“混合休眠模式”，这种模式结合了前两种模式的长处，设备进入睡眠后，内存仍然保持供电，但仍然会将数据写入硬盘，这样内存的数据就同时被保存在两个硬件模块里，如果在唤醒时，设备电量充足（或够用），那 Mac 就会像 Mode 0 一样快速被唤醒，如果唤醒操作前，设备电量已不足了或者已经被耗尽，此时插上电源线后，系统会自动从硬盘里的内存镜像文件中恢复内存，而唤醒过程和 Mode 1 一样慢。这种模式优缺点就不用再用我说了吧，非常灵活。

```
# 关闭小憩
sudo pmset restoredefaults
sudo pmset -a tcpkeepalive 0
pmset -a powernap 0
sudo pmset -a ttyskeepawake 0
# pmset -a dwlinterval 0

# 关闭定时任务：
pmset -g sched
sudo pmset schedule cancelall
```

定位主板

```
我的主板
 Method (_PRW, 0, NotSerialized)  // _PRW: Power Resources for Wake
            {
                Return (GPRW (0x0D, 0x04))
            }
```

修复内核崩溃问题： [Disabled the real EC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix.html#disabling-real-ec-desktops-only), [patched SMBus support](https://dortania.github.io/Getting-Started-With-ACPI/Universal/smbus.html).

## 二、修复 RTC 问题

Issue with AppleRTC, quite a simple fix:

- config.plist -> Kernel -> Quirks -> DisableRtcChecksum -> true

**Note**: If you still have issues, you'll need to use [RTCMemoryFixup](https://github.com/acidanthera/RTCMemoryFixup/releases) and exclude ranges. See [here for more info](https://github.com/acidanthera/bugtracker/issues/788#issuecomment-604608329)

The following boot-arg should handle 99% of cases(pair this with RTCMemoryFixup):

```text
rtcfx_exclude=00-FF
```

If this works, slowly shorten the excluded area until you find the part macOS is getting fussy on

### 修复总有计划任务

``` bash
sudo pmset sched cancelall
sudo chflags schg /Library/Preferences/SystemConfiguration/com.apple.AutoWake.plist
```
This prevents the system from updating the file and my MacBook no longer wakes to reminder events. Needles to mention that it prevents all scheduled system events like sleep, power-on and such. I recommend removing the flag before a system update:
``` bash
sudo chflags noschg /Library/Preferences/SystemConfiguration/com.apple.AutoWake.plist
```

## 三、修复启动黑屏

使用 Whatevergreen.kext，设置启动参数： agdpmod=pikera

还不行，可尝试添加在启动项添加 igfxonln=1 参数，还可与尝试启动项添加gfxrst=1 参数

## 四、参考资料

- [OC-little](https://github.com/daliansky/OC-little/tree/master/00-%E6%80%BB%E8%BF%B0/00-3-ACPI%E8%A1%A8%E5%8D%95)
- [ OpenCore Post-Install  Fixing Sleep](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html)

## 五、版本
2023-07-30  多次睡眠、关机、重启，再未出现报错问题，终于可以聚焦电脑使用而不是修复
2023-06-30  已修复睡眠及崩溃问题，目前完美
