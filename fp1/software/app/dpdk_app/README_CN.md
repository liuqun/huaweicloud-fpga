### *`该部分所有的命令必须在虚拟机上以根用户的方式运行`* ###

[Switch to the English version](./README.md)

*`目录列表`*

* **bin/**: 存储编译后的应用程序和用于启动/停止日志的脚本；  
* **example1/**: 存储用例一的源代码；  
* **example2/**: 存储用例二的源代码；  
* **example3/**: 存储用例三的源代码； 
* **execute_objs/**: 存储**example1/**和**example2/**目录下源代码编译后的目标文件；  
* **func/**: 存储用于主函数调用的源代码；  
* **func_objs/**: 存储**func/**目录下源代码编译后的目标文件；  
* **include/**: 存储头文件；  
* **lib/**: 存储动态链接库文件；  
* **tools/**: 存储工具的源码（读写寄存器）； 

# 1. 编译DPDK源代码和应用程序  
我们提供了用于编译DPDK源代码和用例程序的自动化编译脚本，用户可以通过下面的命令来使用该脚本：

`source build_dpdk_app.sh`

# 2. 运行用例一

`cd bin/`

## 2.1 运行 `ul_get_version` ，打印IP的版本信息
`./ul_get_version -s XXX`

下文中的XXX为选择运行用例的设备槽位号，默认为0。  

## 2.2 运行反相器

`./ul_set_data_test -s XXX -i 0xaa55`  
`./ul_get_data_test -s XXX`

如果运行成功，将会打印0xffff55aa。

## 2.3 运行加法器

`./ul_set_data_add -s XXX -i 0x11111111 -i 0x5a5a5a5a`  
`./ul_get_data_add_result -s XXX`  

如果运行成功，将会打印0x6b6b6b6b。

## 2.4 打印DFX寄存器信息

`./dump_dfx_regs -s XXX`  

如果运行成功，将会打印DFX寄存器的值。

## 2.5 读取UL逻辑寄存器

`./ul_read_bar2_data -s XXX -a AAA`  
 
AAA是需要读取的寄存器地址。

## 2.6 写入UL逻辑寄存器

`./ul_write_bar2_data -s XXX -a AAA -d DDD`  

AAA是需要写入的寄存器的地址，DDD是实际要写入的数据。

# 3. 运行用例二
## 3.1 配置大页

`sysctl -w vm.nr_hugepages=8192`

## 3.2 运行`packet_process`
### 3.2.1 准备

`cd bin/`

### 3.2.2 开启/停止日志

运行下面的命令，将日志保存到**/var/log/fpga/dpdk.log**：

`sh start_dpdk_log.sh`

运行下面的命令，停止日志：

`sh shut_down_dpdk_log.sh`

### 3.2.3 收发包测试

`./packet_process -s XXX -q 0,1,2,7 -l 512 -n 102400099`  

选择`slot XXX`设备的`0, 1, 2, 和 7`队列发送和接收`102400099`个长度为`512`的包，  
上述命令执行成功后，发包和收包线程分别会发送和接收`102400099`个包。

|    参数   |        描述                              |  
| --------- | ---------------------------------------- |  
| -d xxx    | xxx: 队列深度，数值必须是 **1024**, **2048**, **4096**, 或者 **8192**，默认为 **8192** |  
| -s xxx    | xxx: 端口号，数值范围必须是[0, 7]，默认为 **0** |  
| -q xxx    | xxx: 队列号，数值范围必须是[0, 7]，默认为 **0** |  
| -l xxx    | xxx: 收发包测试每个包的长度(取值范围为[64, 1048576]，默认为 **64**) |  
| -n xxx    | xxx: 收发包测试包的个数(默认为**128**，最大为**4294966271**) |  
| -x xxx    | xxx: 收发包测试的次数(取值范围为[1, 64511]，默认为 **1**) |  
| -h        | 该参数打印帮助信息  |  

## 3.3 运行 DDR Checker

该命令仅支持用例二，打印0x5a5a5a5a。

**DDR读写测试中相关参数说明**  

| Parameter | Description                              |
| --------- | ---------------------------------------- |
| **-s**    | slot ID. (The scope is [0, 7]) The default value is 0. |
| **-n**    | DDR ID. The default value is 0.  |
| **-a**    | DDR address |
| **-d**    | DDR write data |

`./ul_write_ddr_data -s XXX -n 0 -a 0x1000 -d 0x5a5a5a5a`  
`./ul_read_ddr_data -s XXX -n 0 -a 0x1000`  

# 4. 运行用例三
## 4.1 配置大页

`sysctl -w vm.nr_hugepages=8192`  

## 4.2 运行`fpga_ddr_rw`
### 4.2.1 准备

`cd bin/`  

### 4.2.2 读写FPGA DDR内存测试

`./fpga_ddr_rw -s 0 -t 1 -p 1000 -m 1 -l 131072 -r 0`  

发起一个线程，从槽位0设备的FPGA DDR读地址为0处读取1000个长度为131072 byte的数据包。

`./fpga_ddr_rw -s 0 -t 1 -p 1000 -m 2 -l 131072 -w 0`  

发起一个线程，从槽位0设备的FPGA DDR写地址为0处写入1000个长度为131072 byte的数据包。

`./fpga_ddr_rw -s 0 -t 1 -p 1000 -m 0 -l 131072 -w 0 -r 0`  

发起一个线程，从槽位0设备的FPGA DDR写地址和FPGA DDR读地址均为0处环回1000个长度为131072 byte的数据包。

|    参数   |        描述                              |  
| --------- | ---------------------------------------- |  
| -t xxx    | xxx: 线程个数. 数值范围必须是 [1, 10]. 默认为 **1**. |  
| -s xxx    | xxx: 槽位号. 默认为 **0**. |  
| -p xxx    | xxx: 包的个数. 默认为 **1000**. |  
| -m xxx    | xxx: 模式. 数值范围必须是 [0(环回模式), 1(读模式), 2(写模式)]. 默认为**1**. |  
| -l xxx    | xxx: 包长. 默认为 **64**. 数值范围必须是 [1, 4*1024*1024]|  
| -r xxx    | xxx: FPGA DDR读地址. 数值范围必须是 [0, 64*1024*1024*1024). 最大为 **64*1024*1024*1024**. |   
| -w xxx    | xxx: FPGA DDR写地址. 数值范围必须是 [0, 64*1024*1024*1024). 最大为 **64*1024*1024*1024**. |   
| -h        | 该参数打印帮助信息.  |  

