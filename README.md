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

## 三、修复启动黑屏

使用 Whatevergreen.kext，设置启动参数： agdpmod=pikera

还不行，可尝试添加在启动项添加 igfxonln=1 参数，还可与尝试启动项添加gfxrst=1 参数

## 四、参考资料

- [OC-little](https://github.com/daliansky/OC-little/tree/master/00-%E6%80%BB%E8%BF%B0/00-3-ACPI%E8%A1%A8%E5%8D%95)
- [ OpenCore Post-Install  Fixing Sleep](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html)

## 五、版本

2023-06-30  已修复睡眠及崩溃问题，目前完美
