# 雷蛇灵刃15"(2018)标准版黑苹果驱动

本仓库内容是适用于雷蛇灵刃15"(2018)标准版的黑苹果CLOVER目录，已经过精心调教，将其替换你的EFI分区的CLOVER目录即可100% function（需更换部分原装硬件）。本文也可做为雷蛇灵刃系列或其他相近的Intel 8代Core平台笔记本的黑苹果驱动教程，这种食用方式需要对黑苹果概念有基本的认识，对CLOVER的运行机制有初步的了解。黑苹果取得今日的发展成就，离不开各位先驱者的智慧与无私，吃水不忘挖井人，向各位开疆拓土的巨人致敬，本文的参考文献在文末一并列出。

目录
---
1. 环境概况
2. 解锁BIOS
3. 显卡
4. 有线网卡
5. 无线网卡
6. 亮度调节记忆
7. 声卡
8. 定制USB
9. 触摸板
10. 电池状态显示
11. 蓝牙
12. 传感器
13. 常见问题
14. 参考文献

环境概况
---
本教程已证实可行的软硬件配置，其中更换的原装硬件用\*标注：

操作系统：macOS Catalina 10.15.3

引导：CLOVER 2.5k(r5100)

BIOS\*：AMI（需重新刷写以解锁部分配置项）

CPU：Intel Core i7-8750H 6 Cores/12 Threads, up to 4.1GHz

GPU：Intel UHD Graphics 630 （独显屏蔽）

显示器：15.6" 全高清 60Hz

硬盘\*：Samsung SSD 970 EVO 500G (NVMe) + HDD 5400转 1T

内存：16GB双通道 DDR4-2667MHz

电池：65Wh

触摸板：I2C Microsoft Precision精准触摸板

无线\*：BCM94352z（DW1560）

声卡：ALC256 3.5mm二合一

解锁BIOS
---
官方BIOS隐藏了很多配置项，其中包含一些使得macOS可以启动所必须启用或禁用的功能，当然通过一些软件层面的手段可以进行修补，但刷写BIOS是一劳永逸的，即使不再玩黑果，也能发掘机器更深的潜力。所有未解锁BIOS的灵刃系列笔记本在安装黑苹果前都应该手动解锁和刷写BIOS。

> 在刷写之前务必确保当前的BIOS、EC、ME等均已是官方最新版！

* **BIOS备份**

在Windows下，打开AFUWINGUIx64，点击储存按钮，将当前BIOS文件导出并备份到某个安全的目录下。

* **BIOS修改**

打开AMIBCP64，点击File - Open，打开刚才导出的BIOS文件。在左边的导航栏展开根目录下的Setup目录，选择某条目后，右边会出现对应的配置项，每一个配置项记录都有一个Access/Use字段，将该字段值从Default改为USER即可启用该配置，即开机进入BIOS后可以看到该配置。需要开启的配置项有：

*
    - Advanced/Power & Performance/
        * Power & Performance（第二行）
        * CPU - Power Management Control
        * Intel(R) Speed Shift Technology
    - Advanced/Power & Performance/CPU - Power Management Control/
        * CPU - Power Management Control（第二行）
        * Intel(R) SpeedStep(tm)
        * Intel(R) Speed Shift Technology
        * C states
        * IO MWAIT Redirection
        * Package C State Limit
        * Timed MWAIT
        * CPU Lock Configuration
    - Advanced/Power & Performance/CPU - Power Management Control/View/Configure CPU Lock Options/
        * CFG Lock
        * Overclocking Lock
    - Chipset/
        * System Agent (SA) Configuration
    - Chipset/System Agent (SA) Configuration/
        * System Agent (SA) Configuration（第二行）
        * VT-d（第五行）
        * Graphics Configuration
        * VT-d（另一个）
    - Chipset/System Agent (SA) Configuration/Graphics Configuration/
        * Graphics Configuration（第二行）
        * Primary Display
        * Internal Graphics
        * DVMT Pre-Allocated（Optimal改为64M或128M）
        * DVMT Total Gfx Mem
	
将BIOS另存为另一个文件。
	
* **BIOS刷写**

打开AFUWINGUIx64，打开刚才修改后的BIOS文件，点击刷新开始刷写，其间尽量不要进行其他操作。

* **BIOS配置**

重启进入BIOS菜单，进行如下配置：

*   
    - Advanced
        - Power & Performance
            - CPU - Power Management Control
                - CPU Lock Configuration
                    * 禁用CFG Lock
                    * 禁用Overclocking Lock
    - Advanced
        - Thunderbolt(TM) Configuration
            * 将Security Level设置为No Security
    - Chipset
        - System Agent (SA) Configuration
            - 禁用VT-d
            - Graphics Configuration
                * 设置DVMT Pre-Allocated为64或128
                * 设置DVMT Total Gfx Mem为MAX
    - Security
        * 禁用Secure Boot
    - Boot
        * 禁用Fast Boot

显卡
---
* **驱动集显**

待补充

* **屏蔽N卡独显**

使用[Rehabman](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/tree/master/hotpatch)大神的hotpatch，将其中的SSDT-DDGPU.dsl文件修改为只保留自己的独显设备路径，灵刃15"(2018)标准版的为`_SB.PCI0.PEG0.PEGP`，修改后的完整内容如下，然后编译放到CLOVER/ACPI/patched。

```
// For disabling the discrete GPU

#ifndef NO_DEFINITIONBLOCK
DefinitionBlock("", "SSDT", 2, "hack", "_DDGPU", 0)
{
#endif
    // Note: The _OFF path should be customized to correspond to your native ACPI
    // the two paths provided here should be considered examples only
    // it is best to edit the code such that only the single _OFF path that your ACPI
    // uses is included.
    External(_SB.PCI0.PEG0.PEGP._OFF, MethodObj)

    Device(RMD1)
    {
        Name(_HID, "RMD10000")
        Method(_INI)
        {
            // disable discrete graphics (Nvidia/Radeon) if it is present
            If (CondRefOf(\_SB.PCI0.PEG0.PEGP._OFF)) { \_SB.PCI0.PEG0.PEGP._OFF() }
        }
    }
#ifndef NO_DEFINITIONBLOCK
}
#endif
//EOF
```

有线网卡
---
这个是最容易驱动的，没有之一，直接下载最新的RealtekRTL8111.kext放到CLOVER/kexts/Other并重建缓存。

无线网卡
---
原装的Intel无线网卡全球无解，建议更换成BCM94352z，然后下载最新的AirportBrcmFixup.kext放到CLOVER/kexts/Other并重建缓存。

亮度调节记忆
---
亮度调节使用RehabMan大神的hotpatch，将其中的SSDT-PNLF.dsl编译后放到CLOVER/ACPI/patched，并将最新的WhateverGreen.kext放到CLOVER/kexts/Other并重建缓存；

亮度记忆使用RehabMan大神的hotpatch，将其中的SSDT-ALS0.dsl编译后放到CLOVER/ACPI/patched。

声卡
---
声卡驱动是比较棘手的一个部分，基本有两种方式：VoodooHDA和AppleALC。VoodooHDA的优势是操作简单，拖拉拽即可；劣势也很突出，表现为随机或固定时机的爆音、破音、电流声，不建议强迫症患者使用。AppleALC是个开源项目，旨在辅助驱动苹果原生的声卡驱动，从而达到白果原汁原味的体验，且经过深度定制后可以保证系统升级后不失效，它的支持列表日趋完善，几乎能覆盖当下所有常见的消费级声卡。但直接使用最新的AppleALC.kext可能存在的问题是，试遍了所有的layout-id，要么直接无法驱动，要么喇叭响麦克无电平，要么麦克有电平喇叭不响，又或者有声但丢失频段，这个时候，就需要自己动手定制专属的AppleALC。

* **提取声卡codec**
  
下载Ubuntu镜像。在已装好的macOS下，插入一个USB3.0的U盘，确定其设备文件名diskX。
  
使用如下命令卸载U盘，准备写入Ubuntu镜像：
  
```
diskutil umountDisk diskX
```
  
将镜像恢复到U盘。写入速度取决于设备体质。
  
```
sudo dd if=path/to/image/Ubuntu-xx.xx-desktop-amd64.iso of=/dev/diskX bs=1m
```
  
写入完成后，重启选择U盘引导，然后选择Try Ubuntu without installing。
  
进入系统后，按<kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>t</kbd>打开终端，键入命令：
  
```
cd ~/Desktop
cp /proc/asound/card0/codec* .
sudo cp -R /sys/firmware/acpi/tables .    // 这个命令用于提取原始ACPI表，在这里可以不用
sudo chown -R ubuntu:ubuntu *
```
  
codec文件要么在card0，要么在card1，复制出来的文件有一个是我们需要的声卡codec。将codec文件（和tables目录）复制到其他可被macOS读取的目录。
  
* **格式化有效节点**

进入macOS，使用脚本输出声卡的有效节点。如果产生文件错误，需要删除codec中的AFG Function Id: 0x1 (unsol 1)和空行。
  
```
verbit.sh codec#0 > ALC256_dump.txt
```
  
得到的ALC256_dump.txt文件的示例内容如下，这也是我的灵刃上的实际结果：
  
```
Verbs from Linux Codec Dump File: codecFromUbuntu/card0/codec#0
Codec: Realtek ALC256   Address: 0   DevID: 283902550 (0x10ec0256)

 Jack   Color  Description                  Node     PinDefault             Original Verbs
--------------------------------------------------------------------------------------------------------
Digital Unknown  Mic at Oth Mobile-In        18 0x12   0xb7a60140   01271c40 01271d01 01271ea6 01271fb7
Unknown Unknown  Line Out at Ext N/A         19 0x13   0x40000000   01371c00 01371d00 01371e00 01371f40
 Analog Unknown  Speaker at Int N/A          20 0x14   0x90170120   01471c20 01471d01 01471e17 01471f90
    1/8   Black  Speaker at Ext Rear         24 0x18   0x411111f0   01871cf0 01871d11 01871e11 01871f41
    1/8   Black  Mic at Ext Right            25 0x19   0x04a11030   01971c30 01971d10 01971ea1 01971f04
    1/8   Black  Speaker at Ext Rear         26 0x1a   0x411111f0   01a71cf0 01a71d11 01a71e11 01a71f41
Speaker at Ext Rear    0x1b 0x1b                        1091637744 01b71cf0 01b71d11     01b71e11 01b71f41  
 Analog    Pink  Modem Line at Ext N/A       29 0x1d   0x40679a2d   01d71c2d 01d71d9a 01d71e67 01d71f40
    1/8   Black  Speaker at Ext Rear         30 0x1e   0x411111f0   01e71cf0 01e71d11 01e71e11 01e71f41
    1/8   Black  HP Out at Ext Right         33 0x21   0x04211010   02171c10 02171d10 02171e21 02171f04
--------------------------------------------------------------------------------------------------------

   Jack   Color  Description                  Node     PinDefault             Modified Verbs
--------------------------------------------------------------------------------------------------------
Digital Unknown  Mic at Oth Mobile-In        18 0x12   0xb7a60140   01271c40 01271d00 01271ea6 01271fb0
Unknown Unknown  Line Out at Ext N/A         19 0x13   0x40000000   01371c00 01371d00 01371e00 01371f40
 Analog Unknown  Speaker at Int N/A          20 0x14   0x90170120   01471c20 01471d00 01471e17 01471f90
    1/8   Black  Mic at Ext Right            25 0x19   0x04a11030   01971c30 01971d10 01971ea1 01971f00
 Analog    Pink  Modem Line at Ext N/A       29 0x1d   0x40679a2d   01d71c50 01d71d90 01d71e67 01d71f40
    1/8   Black  HP Out at Ext Right         33 0x21   0x04211010   02171c60 02171d10 02171e21 02171f00
--------------------------------------------------------------------------------------------------------
Modified Verbs in One Line: 01271c40 01271d00 01271ea6 01271fb0 01371c00 01371d00 01371e00 01371f40 01471c20 
01471d00 01471e17 01471f90 01971c30 01971d10 01971ea1 01971f00 01d71c50 01d71d90 01d71e67 01d71f40 02171c60 
02171d10 02171e21 02171f00
--------------------------------------------------------------------------------------------------------
```
  
> 名词解释：`Codec: Realtek ALC256`指的是声卡型号是ALC256；`Address: 0`指的是生成ConfigData的数据前缀是0，在Modified Verbs in One Line后面的每组数据的第一位就是这个0；`DevID: 283902550 (0x10ec0256)`指的是vendorID（0x10ec）和设备型号（0256），283902550是对应的十进制形式。
  
脚本中的Modified Verbs就是为我们整理出的有效节点，但是仍然不准确，将所有PinDefault最高位为4的节点去掉，剩下的就是最终的有效节点：
  
|节点ID（十进制）|节点ID|节点描述|
|:--------------|:-----|:-----|
|18|0x12|Mic at Int|
|20|0x14|Speaker at Int|
|25|0x19|Mic at Ext Right|
|33|0x21|HP Out at Ext Right|
  
* **整理ConfigData**

将ALC256_dump.txt的所有节点或只将上一步整理出的有效节点按如下样式整理，并对PinDefault做小端变换：
  
[Fixed]是内部设备；[Jack]是通过插孔进行连接的外部设备；[N/A]是其它未知设备

```
Node     c  d  e  f     Description
12	40 01 a6 b7     [Fixed] Mic at Oth Mobile-In
13	00 00 00 40     [N/A]Line Out at Ext N/A
14	20 01 17 90     [Fixed] Speaker at Int N/A	
18	f0 11 11 41     [N/A]Speaker at Ext Rear
19	30 10 a1 04     [Jack] Mic at Ext Right
1a	f0 11 11 41	[N/A]Speaker at Ext Rear
1b	f0 11 11 41     [N/A]Speaker at Ext Rear
1d	2d 9a 67 40	[N/A]Modem Line at Ext N/A
1e	f0 11 11 41	[N/A]Speaker at Ext Rear
21	10 10 21 04	[Jack] HP Out at Ext Right
```
  
（1）修正PinDefault
  
将小端转换后的PinDefault的8个位按如下参考规则做修正：
  
第一位：数值越小优先级越高，耳机优先级一定要低于内置扬声器，外置麦克风一定要低于内置麦克风。默认开启内置扬声器和麦克风。外置麦克如果设置为7不可用，则应设置成2。耳机可能需要设置成3；
  
第二位：一般设置为0。Line Out设置为f；
  
第三位：无需修改；
  
第四位：耳机（耳麦）设置为0，表示检测到设备插入时使用。内置麦克风和内置扬声器设置为1；
  
第五位：按接口功能设置。外置麦克如果设置为A不可用，则应设置成8；
  
第六位：仅为描述，对驱动本质无影响，无需修改；
  
第七位：0是外置设备，9是笔记本内建设备，4是被屏蔽的无用端口；
  
第八位：仅为描述，对驱动本质无影响，无需修改。
  
前面整理出的0x19的设备为[Jack] Mic at Ext Right，PinDefault值为30 10 a1 04，现在希望它能正常工作，将PinDefault修改为20 10 81 02（笔记本声卡需要将外置麦克Mic Ext设置为Line In），同时，因为灵刃15"(2018)标准版只有一个耳机插孔，即二合一插孔，所以需要将耳机输出和耳麦定义成组合插孔，即最终的修改：

Mic at Ext : 30 10 a1 04 -> 20 10 8b 02
  
HP Out Ext : 10 10 21 04 -> 30 10 2b 02

使用用f0 00 00 40屏蔽无效设备节点，以避免杂音和底噪。修正后的最终数据为：
  
```
Node     c  d  e  f     Description
12	10 01 a6 90	[Fixed] Mic at Oth Mobile-In
13	f0 00 00 40	屏蔽
14	40 01 17 90	[Fixed] Speaker at Int N/A	
18	f0 00 00 40	屏蔽
19	20 10 8b 02	[Jack] Mic at Ext Right	
1a	f0 00 00 40	屏蔽
1b	f0 00 00 40	屏蔽
1d	f0 00 00 40	屏蔽
1e	f0 00 00 40	屏蔽
21	30 10 2b 02	[Jack] HP Out at Ext Right
```
  
（2）生成ConfigData
  
每个设备有4条数据，计算公式：
  
Address + Node + 71c +【c】
  
Address + Node + 71d +【d】
  
Address + Node + 71e +【e】
  
Address + Node + 71f +【f】
  
c、d、e、f为刚才整理出的2位数值。按此计算出的各个设备的数据如下，其中，还需要在ALC256_dump.txt中搜索EAPD，存在EAPD字样的设备节点的数据后需要再加一段Address+Node+70c+EAPD值：

```
01271c10 01271d01 01271ea6 01271f90
01371cf0 01371d00 01371e00 01371f40
01471c40 01471d01 01471e17 01471f90 01470c02
01971c20 01971d10 01971e8b 01971f02
01d71cf0 01d71d00 01d71e00 01d71f40
02171c30 02171d10 02171e2b 02171f02 02170c02
```
  
将上面的数据去掉换行，整理出来就是最终的ConfigData：
  
```
01271c10 01271d01 01271ea6 01271f90 01371cf0 01371d00 01371e00 01371f40 01471c40 01471d01 01471e17 01471f90 01470c02 01871cf0 01871d00 01871e00 01871f40 01971c20 01971d10 01971e8b 01971f02 01a71cf0 01a71d00 01a71e00 01a71f40 01b71cf0 01b71d00 01b71e00 01b71f40 01d71cf0 01d71d00 01d71e00 01d71f40 01e71cf0 01e71d00 01e71e00 01e71f40 02171c30 02171d10 02171e2b 02171f02 02170c02
```
  
* **整理有效路径**
  
一般3个节点为1条路径，实现一个设备接口功能，且节点一般不重复。对于输入设备（麦克风），为反向推导，即设备节点是最后一个节点；对输出设备（耳机，扬声器），设备节点是第一个节点。具体方法：
  
对于输入设备，在codec中搜索设备节点，如0x19，得到一个是节点本身，一个在关联节点下（如0x22）；然后再搜索0x22，得到一个是节点本身，一个在关联节点（如0x09）下。则路径为0x09 -> 0x22 -> 0x19，转为十进制为9 -> 34 -> 25
  
对于输出设备，在codec中搜索设备节点，如0x14，只需要关注节点本身的Connection中的结点，如0x02；然后再搜索0x02，也是只关注节点本身的Connection中的结点，如果有多个，在已选节点不重复的前提下，优选数值小的。则路径为0x14 -> 0x02，即20 –> 2
  
由此推导出所有设备的路径，如果有共同关联的节点，则应更换上个节点重新推导，直到无重复。
  
也可以通过findPath.py脚本整理。
  
* **编译相关文件**

AppleALC-master – Resources – ALC256 - Platforms13.xml
  
AppleALC-master – Resources – ALC256 - Layout13.xml
  
AppleALC-master – Resources – ALC256 - Info.Plist
  
AppleALC-master – Resources - PinConfigs.kext – Contents - Info.Plist

现在，要将先前整理出的有效节点路径和ConfigData数据放进AppleALC的相应位置。下载AppleALC源码，进入Resources目录，删除其他型号声卡的目录，进入ALC256目录，该目录结构：
  
ALC256
> Info.plist  // 定义声卡驱动所需的数据
>> CodecID  // 声卡型号的十进制形式  
>> CodecName  // 声卡名称描述  
>> Files  
>>> Layouts  // 定义声卡设备布局  
>>> Platforms  // 定义声卡平台注入，包括有效节点和路径
>
> layoutXX.xml  // 需要注入的layout-id  
> Platforms*.xml  // 路径定义

由于全自主定制，所以可使用任意layout-id，比如13，只需保持统一即可。
  
（1）修改Platforms.xml
  
PathMap
> 0  // 第一输入项
>> 0  // 内置mic
>>> 0
>>>> 0  // 将路径按顺序输入到0、1、2的NodeID中  
>>>> 1  
>>>> 2
>
>> 1  // Line In
>>> 0
>>>> 0  
>>>> 1  
>>>> 2
>
>>
>> 2  // 外置mic（笔记本可能需要配置在Line In下面，从而可以删除该项）
>>> ...
>
> 1  // 第一输出项（或第二输入项。当外置mic无法自动切换时，此时也需要删除第一输入项的1和2）
>> 0  // 内置speaker
>>> ...
>
>> 1  // 耳机
>>> ... 
>
>> 2  // Line Out
>>> ...
  
推导的所有路径修改完毕后，在PathMapID中输入声卡型号256。
  
（2）修改AppleALC-master – Resources – ALC256 - Info.plist
  
> CodecID  // 283902550
> Files  
>> Layouts
>>> Comment  // 注释信息  
>>> Id  // 13  
>>> Path  // layout13.xml  
>
>>
>> Platforms  
>>> Comment  // 注释信息  
>>> Id  // 13  
>>> Path  // Platforms13.xml
  
（3）修改AppleALC-master – Resources - PinConfigs.kext – Contents - Info.plist
  
IOKitPersonalities  
> HDA Hardware Config Resource  
>> HDAConfigDefault  
>>> 0  
>>>> Codec  // ALC256  
>>>> CodecID  // 283902550  
>>>> ConfigData  // 填入整理后的不带换行的ConfigData数据  
>>>> LayoutID  // 13
  
（4）修改AppleALC-master – Resources – ALC256 - Layout13.xml
  
> LayoutID  // 13
> PathMapRef  
>> 0  
>>> CodecID  // 283902550  
>>> PathMapID  // 256

部分声卡需要删除LineOut项。
  
外置mic需要修改电压控制值来实现驱动。搜索codec中外置mic下的vref值，vref含义为初始电压基础上增加的百分比。当vref不为Hiz时，muteGPIO={（vref转换为16进制）+"0100"+node id}转换为10进制，codec中vref表示的是十进制，计算时转为16进制。如果vref为Hiz，则muteGPIO=0
  
* **实现声卡驱动**

将DEBUG版本的Lilu放入AppleALC-master根目录，使用Xcode编译AppleALC-master中的AppleALC.xcodeproj，并将构建好的驱动文件放入CLOVER驱动目录。在CLOVER – Devices – Audio – Inject注入正确的layout-id值，如13。并在Clover Configurator中加入hotpatch重命名补丁：
  
Comment: change HDAS to HDEF
	
   Find: 48444153
  
Replace: 48444546
  
定制USB
---
* 将USBInjectAll.kext（用于端口发现）和XHCI-unsupported.kext放入CLOVER/kexts/Other；

* hotpatch重命名补丁

Comment: change XHCI to XHC

   Find: 58484349

Replace: 5848435F

* 重启，打开Hackintool转到工具栏 – USB，删掉所有端口并刷新，确认本机全部的USB端口情况；

* 在CLOVER启动参数中加入-uia_exclude_ss后重启，再进入Hackintool看到的就是屏蔽了所有usb3.0端口后剩下的端口。用一个usb2.0以上的设备插拔所有外部物理usb接口，记录使用到的端口（标绿色的）；

* 将上述启动参数改为-uia_exclude_hs uia_include=HS08（HS08为内置键盘使用的端口）后重启，再进入Hackintool看到的就是屏蔽了所有usb2.0端口后剩下的端口。用一个usb3.0设备插拔所有外部物理usb接口，记录使用到的端口（标绿色的）；

* 删掉上述启动参数后重启，进入Hackintool，刚才所有标绿色的端口必须保留，其他端口酌情删除，最后剩下不超过15个端口即可；

* 设置连接器类型

永远连接设备的端口设为Internal

外部设备使用的端口设为USB3

Type-C接口使用的端口，如果正反插用同一个端口，设为TypeC+Sw；否则，设为TypeC

* 点击“导出”生成USBPort.kext、SSDT-EC.aml、SSDT-UIAC.aml和SSDT-USBX.aml（可能没有该文件），复制SSDT-EC.aml到 CLOVER/ACPI/patched，然后以下方式选其一：

方式一：复制USBPorts.kext到CLOVER/kexts/Other，删除USBInjectAll.kext和XHCI-unsupported.kext

方式二：复制SSDT-UIAC.aml和SSDT-USBX.aml（如果有）到CLOVER/ACPI/patched，删除XHCI-unsupported.kext

触摸板
---
* 运行模式：轮询（polling mode）和中断（interrupts mode）

轮询模式是软件层驱动，占用更多系统资源；中断模式是硬件层驱动，是触摸板最佳的运行模式，分APIC和GPIO两种类型。

GPIO 中断：DSDT在SBFG中存在有效 GPIO Pin，CRS方法中返回 (SBFB,SBFG)，并应用启用GPIO控制器的补丁。

APIC 中断：当IOInterruptSpecifiers值没有或者小于0x2F时，此时就是APIC中断，无需对DSDT进行大量修改，只需应用Windows(20XX)补丁即可。此时使用的是APIC控制器而不是GPIO控制器。

轮询：DSDT中除了Windows(20XX)补丁以外，不应用其他补丁，CRS方法中只返回SBFB或者(SBFB,SBFI)。

* 确认设备类型

打开设备管理器，找到人体学输入设备，如果是i2c设备，那么应该能看到一个i2c hid设备。接下来需要右击“鼠标和其他指针设备”查看触摸设备的属性，如果显示为“在I2C HID上”，那么就是I2C HID设备；如果是“在I2C USB上”，那么就是I2C USB设备。但是如果找不到，那么就说明触摸板不是I2C类型的而是PS2类型的，这类触摸板的驱动文件一般是Applesmarttouchpad或者VoodooPS2controller。

如何确认设备是否被VoodooI2CHID支持？IORegExplorer搜索ACPI ID，如果Compatible属性值为PNP0C50，则支持。

* 确认BIOS设备名称（即ACPI ID）

在Windows设备管理器中找到I2C HID设备，打开 属性 – 详细信息 – BIOS设备名称。一般情况的对应关系如下，X为某个数字：

触摸板 - TPDX, ELAN, SYNA, CYPR
  
触摸屏 - TPLX, ELAN, ETPD, SYNA, ATML
  
> 这些ID可能在DSDT中出现多次，需要自己确认哪一个才是我们要的设备。

* 确认当前中断模式的情况

在没有安装VoodooI2C及各种目标驱动的情况下，在IORegExplorer搜索ACPI ID，如果没有，则需要打Windows Patch。如果在右侧没有IOInterruptSpecifiers或者有但它的value值前两位小于等于0x2F，则直接安装驱动即可；否则，记录下该值，即hexadecimal APIC pin number，在后面的GPIO pin环节会用到。

* hotpatch的SSDT制作

（1）在原始DSDT中搜素ACPI ID，将相应的Device代码块复制到样本SSDT-GPI0.dsl的DefinitionBlock代码块中（放在最后面），并将复制后的设备名改为全路径名，同时也要修改DefinitionBlock行的设备名和注释中的OEM Table ID值设备名。后续代码修改均在SSDT-GPI0.dsl中做;

（2）修改_CRS方法。删除方法中直接的If语句，删除所有Return语句，只留这一句Return语句（其他语句不动）：

```
Return (ConcatenateResTemplate (SBFB, SBFG))
```

（3）删除_CRS方法中的SBFI Name。但是如果它不只有Interrupt代码块，则重命名SBFI为SBFB，并删除Interrupt代码块：

```
Method (_CRS, 0, Serialized)  // _CRS: Current Resource Settings
    {
         Name (SBFI, ResourceTemplate ()
             {
                  I2cSerialBusV2 (0x0015, ControllerInitiated, 0x00061A80,
                      AddressingMode7Bit, "\\_SB.PCI0.I2C1",
                      0x00, ResourceConsumer, , Exclusive,
                  )
                  Interrupt (ResourceConsumer, Level, ActiveLow, Exclusive, ,, )
                  {
                      0x0000006D,
                  }
              })
              Return (SBFI)
    }
```

（4）GPIO pinning

根据表一[Cannon Point-H Labels](https://github.com/coreboot/coreboot/blob/master/src/soc/intel/cannonlake/include/soc/gpio_defs_cnp_h.h#L42)和表二[Cannon Point-H Decimal Pin Numbers](https://github.com/coreboot/coreboot/blob/master/src/soc/intel/cannonlake/include/soc/gpio_soc_defs_cnp_h.h#L40)做映射hexadecimal APIC pin number => decimal hardware pin number
（如果该值的十六进制介于5c到77之间，则可用；否则，应该存在APIC pin到GPIO pin的多个映射，尝试其他映射）

再根据表三[Cannon Point-H Chipset GPP](https://github.com/coolstar/VoodooGPIO/blob/master/VoodooGPIO/CannonLake-H/VoodooGPIOCannonLakeH.hpp#L414)做映射decimal hardware pin number => CNL_GPP(num, base, end, gpio_base)

计算decimal GPIO pin number = decimal GPIO pin number – base + gpio_base

将上述结果值转为16进制填入SBFG Name块的Pin list：

```
Name (SBFG, ResourceTemplate ()
    {
        GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000, "\\_SB.PCI0.GPI0", 0x00, ResourceConsumer, ,
        )
        {   // Pin list
            0x00
        }
    })
```

> 如果最终的十六进制计算值不在5c到77之间，可能驱动无效，此时可能存在多种映射，尝试其他。如果都不行，则尝试常规值：0x17, 0x1B, 0x34 and 0x55

（5）SSDT排错

编译如果报错，则去原始DSDT中定位相应的数值、方法、设备或引用。

对于数值，将其构造区域复制到SSDT：

```
OperationRegion (COMP, SystemMemory, 0x7AF56018, 0x0200)
    Field (COMP, AnyAcc, Lock, Preserve)
    {
       // 数据,   值,
    }
```

对于设备或方法，在原始DSDT中找到其全路径，复制到SSDT：

```
External (xxxx.xxxx.xxxx, DeviceObj) // 如果是方法，就使用MethodObj；如果是区域则使用FieldUnitObj
```

* hotpatch重命名补丁

（1）将触摸板设备改名，比如TPD0改为TPDX，对应上述自定义的SSDT-GPI0.aml
  
Comment: change TPD0 to TPDX
  
Find: 54504430
     
Replace: 54504458
  
（2）Windows Patch，对应RehabMan的SSDT-XOSI.aml
  
Comment: change _OSI to XOSI
  
Find: 5F4F5349
     
Replace: 584F5349
  
（3）GPIO Controller Enabling，对应上述自定义的SSDT-GPI0.aml
  
Comment: change _STA to XSTA in GPI0
  
Find: 5F535441
       
Replace: 58535441
    
TgtBridge: 47504930
  
7. 安装驱动

VoodooI2C.kext

VoodooI2CHID.kext


电池状态显示
---
直接给原始DSDT打入MaciASL的补丁 [bat] Razer Blade (2014)

```
into method label B1B2 remove_entry;

into definitionblock code_regex . insert

begin

Method (B1B2, 2, NotSerialized) { Return(Or(Arg0, ShiftLeft(Arg1, 8))) }\n

end;


into device label EC0 code_regex BIF1,\s+16, replace_matched begin IF10,8,IF11,8, end;

into device label EC0 code_regex BIF2,\s+16, replace_matched begin IF20,8,IF21,8, end;

into device label EC0 code_regex BIF3,\s+16, replace_matched begin IF30,8,IF31,8, end;

into device label EC0 code_regex BIF4,\s+16, replace_matched begin IF40,8,IF41,8, end;


into device label EC0 code_regex BST0,\s+16, replace_matched begin ST00,8,ST01,8, end;

into device label EC0 code_regex BST1,\s+16, replace_matched begin ST10,8,ST11,8, end;

into device label EC0 code_regex BST2,\s+16, replace_matched begin ST20,8,ST21,8, end;

into device label EC0 code_regex BST3,\s+16, replace_matched begin ST30,8,ST31,8, end;


into method label _BIF code_regex \^\^EC0\.BIF1, replaceall_matched begin B1B2(^^EC0.IF10,^^EC0.IF11), end;

into method label _BIF code_regex \^\^EC0\.BIF2, replaceall_matched begin B1B2(^^EC0.IF20,^^EC0.IF21), end;

into method label _BIF code_regex \^\^EC0\.BIF3, replaceall_matched begin B1B2(^^EC0.IF30,^^EC0.IF31), end;

into method label _BIF code_regex \^\^EC0\.BIF4, replaceall_matched begin B1B2(^^EC0.IF40,^^EC0.IF41), end;


into method label _BST code_regex \^\^EC0\.BST0, replaceall_matched begin B1B2(^^EC0.ST00,^^EC0.ST01), end;

into method label _BST code_regex \^\^EC0\.BST1, replaceall_matched begin B1B2(^^EC0.ST10,^^EC0.ST11), end;

into method label _BST code_regex \^\^EC0\.BST2, replaceall_matched begin B1B2(^^EC0.ST20,^^EC0.ST21), end;

into method label _BST code_regex \^\^EC0\.BST3, replaceall_matched begin B1B2(^^EC0.ST30,^^EC0.ST31), end;


# added for Razer Blade 15 (2018), per JomanJi

into device label EC0 code_regex BIF0,\s+16, replace_matched begin IF00,8,IF01,8, end;

into method label _BIF code_regex \(\^\^EC0.BIF0, replaceall_matched begin (B1B2(\^\^EC0.IF00,\^\^EC0.IF01), end;


# utility methods to read/write buffers from/to EC

into method label RE1B parent_label EC0 remove_entry;

into method label RECB parent_label EC0 remove_entry;

into device label EC0 insert

begin

Method (RE1B, 1, NotSerialized)\n

{\n

OperationRegion(ERAM, EmbeddedControl, Arg0, 1)\n

Field(ERAM, ByteAcc, NoLock, Preserve) { BYTE, 8 }\n

Return(BYTE)\n

}\n

Method (RECB, 2, Serialized)\n

// Arg0 - offset in bytes from zero-based EC\n

// Arg1 - size of buffer in bits\n

{\n

ShiftRight(Add(Arg1,7), 3, Arg1)\n

Name(TEMP, Buffer(Arg1) { })\n

Add(Arg0, Arg1, Arg1)\n

Store(0, Local0)\n

While (LLess(Arg0, Arg1))\n

{\n

Store(RE1B(Arg0), Index(TEMP, Local0))\n

Increment(Arg0)\n

Increment(Local0)\n

}\n

Return(TEMP)\n

}\n

end;


# buffer fields

into device label EC0 code_regex (ECCM,)\s+(256) replace_matched begin ECCX,%2,//%1%2 end;

into method label _BIF code_regex \(\^\^EC0.ECCM, replaceall_matched begin (^^EC0.RECB(0x60,256), end;
```

蓝牙
---
直接下载BrcmBluetoothInject.kext + BrcmFirmwareData.kext + BrcmPatchRAM2.kext放到CLOVER/kexts/Other。查看系统信息里面的蓝牙固件版本号，如果没有安装成功，那个固件版本号是固定的4096；如果安装成功，版本号>4096

传感器
---
FakeSMC.kext是黑苹果必备驱动，用来告诉系统这是苹果的硬件。现在除了FakeSMC.kext还有VirtualSMC.kext来替代，它需要配合CLOVER/drivers/UEFI/VirtualSmc.efi，且和FakeSMC.kext、ACPIBatteryManager.kext、drivers/UEFI/SMCHelper.efi不兼容。使用VirtualSMC用iStat Menus能显示电池生产日期。但检查CPU频率需要额外安装Intel Power Gadget。

常见问题
---
* 如果某个功能在睡眠唤醒或系统升级后失效，可以尝试重建缓存：

```
sudo kextcache -i /
```

* 10.15安装时卡最后2分钟，尝试加入CLOVER/drivers/UEFI/AptioMemoryFix.efi

* 如果关机变成重启，尝试加入CLOVER/drivers/UEFI/EmuVariableUefi-64.efi
