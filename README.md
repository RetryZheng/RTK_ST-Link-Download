# RTK_ST-Link-Download
LabVIEW实现ST-Link自动烧录
还记得刚毕业那会弄过一台测试设备，测试空调主板功能，测试前需要进行固件烧录，其中用到的主控芯片就是STM32Fxxx，具体型号不记得了，当时是哪种方式去实现的也不太记得了，现在又需要解决STM32F4xx的芯片自动烧录问题，索性整理成档便于以后查阅。

用ST官方提供的一个工具即可实现，那就是ST-Link utility，使用简单下载方便。

官方下载链接：https://www.st.com/en/development-tools/stsw-link004.html

百度网盘：https://pan.baidu.com/s/1pAwqGaxrtG1zRB4pqWQszg  提取码：jxyw

安装完成后打开界面如下图所示，具体操作请自行查看帮助文档或网络相关资源。
![image](https://user-images.githubusercontent.com/29085487/229339330-246b82e2-c68a-47d1-82bf-e075c5cfb932.png)


该工具提供了CLI(Command Line Interface)，帮助文档也详细介绍了相关指令如何使用，这里我就是通过CLI去实现的，其中的命令有很多，我只介绍几个有关烧录的，其它的自行查阅文档。

![image](https://user-images.githubusercontent.com/29085487/229339289-ece11234-13bb-482e-b577-12c68b4a6b69.png)

![image](https://user-images.githubusercontent.com/29085487/229339245-2cd6ba5e-ec47-44c1-8cdc-d340170f2671.png)


接下来看看具体的实现步骤：

### 1.添加CLI到系统环境变量中

将ST-Link_CLI.exe所在目录的路径(如D:\Program Files (x86)\STM32 ST-LINK Utility v4.6.0\ST-LINK Utility)添加到系统环境变量中，如下图所示：
![image](https://user-images.githubusercontent.com/29085487/229339338-6259ba4b-0784-4966-ba6e-4740a4a48299.png)


### 2.查询ST-Link/V2烧录器信息
先安装烧录器的USB驱动以确保能够正确识别到该烧录器硬件，驱动下载链接：

官方：https://www.st.com/content/my_st_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-utilities/stsw-link009.html

网盘：https://pan.baidu.com/s/1UXQ4X4MtjhjSGUzpEUdhDA    提取码：amrn

一切正常可以在设备管理器中找到它，如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339350-d1904180-3029-4147-bce8-2e579f1dd6e2.png)

在命令行中输入：ST-LINK_CLI -List ，即可获取烧录器的SN和固件版本，如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339361-88d770b6-0bca-494b-bb73-02ffa865d413.png)

如果同时连接了多个，这里将展示出多个烧录器信息，SN信息在连接芯片时需要用到。

### 3.连接待烧录的MCU芯片

烧录前请确保硬件连接正常，使用 -c [ID=<id>/SN=<sn>] [JTAG/SWD] [FREQ=<frequency>] [UR/HOTPLUG] [LPM] 命令进行连接，其中包含了很多参数，简单说明如下：

参数1(ID/SN)：提供烧录器的ID或SN信息，ID从[0..9]，根据连接的烧录器数量递增，SN信息可以通过-List命令获取；

参数2(JTAG/SWD)：选择使用的接口协议类别，是用JTAG还是SWD，默认使用的是JTAG，这里我选用SWD；

参数3(FREQ)：设置不同协议的频率，JTAG和SWD支持的各不相同，JTAG默认使用的是9.0MHz, SWD默认使用的是4.0MHz，通常使用默认即可，也可以通过索引去设置 FREQ=x，SWD(x=0~10), JTAG(x=0~6)，分别代表了不同的频率，具体请查看文档；

参数4：设置复位模式，UR（Connect to the target under reset）， HOTPLUG（Connect to the target without halt or reset），这里我选用UR；

参数5：激活在低功耗模式下调试；

详细内容请阅读帮助文档，如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339424-0c3cdb5d-5969-49e1-b01b-49a7b4a37b1c.png)

使用ID连接如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339426-31312d95-c77a-4303-9f2c-ca59456f9317.png)

使用SN连接如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339430-d1f83881-3e5f-4bf8-a67c-5f009e93b4ae.png)

### 4.下载固件到Flash
使用 -P <File_Path> [\<Address\>] 命令进行操作，其中地址是可选的，如果没有特定要求可以不指定，STM32的Flash映射地址是从0x08000000开始的，固件文件格式支持3种：.bin, .hex, .srec；如果文件路径中有空格，需要包含在双引号中，演示如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339462-04a09624-3345-467f-9913-4f8022b4f1a4.png)

如果需要验证烧录是否成功，需要使用 -V [while_programming/after_programming] 命令，一种是在烧录中进行验证，另一种是在烧录完后进行；如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339470-23af9175-6f8c-449d-a343-79fb59178be0.png)

到这里烧录功能就已经实现了，接下来说几个可能会用到的命令。

### 5.可能会使用的命令
#### 5.1 -Rst
复位MCU，如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339475-d460dc79-c3ba-4913-b81b-65478c9f8374.png)

#### 5.2 -ME
擦除整个芯片，如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339480-7e0dbfa8-0164-4c4b-b9dd-85a8df5a0d71.png)

#### 5.3 -SE
擦除指定的扇区，-SE  <Start_Sector> [<End_Sector>]，如果只指定起始扇区号，就只擦除这个扇区，如 -SE 0 （擦除扇区0）；如果指定了起始和结束扇区号，那么会擦除指定区间范围内的所有扇区，如 -SE 2 12 （擦除扇区2~扇区12），如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339483-65912e0c-3081-4483-9f31-32339693fb24.png)


关于指令就介绍这么多，其它的功能需要用到的话请自行查阅帮助文档。

### 6.封装好的LabVIEW库
以上指令都是在命令行中输入的，用来手动验证还是不错的，为了更方便地使用，我把它们封装好了，如下图所示：

![image](https://user-images.githubusercontent.com/29085487/229339588-2ad38e8d-6c42-499e-808d-829b22d178d0.png)

![image](https://user-images.githubusercontent.com/29085487/229339592-1c7bfe1a-2a1b-42be-abad-b7e1a21919b5.png)

![image](https://user-images.githubusercontent.com/29085487/229339594-504ad297-7252-40e8-a8be-a791181f8c70.png)

![image](https://user-images.githubusercontent.com/29085487/229339598-ddec8a29-2279-4ce1-a86c-5f14d2517c1a.png)

![image](https://user-images.githubusercontent.com/29085487/229339602-b5c27ff2-5e46-4e6a-af00-e0cb6f43c751.png)




