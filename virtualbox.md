# VirtualBox

## Ubuntu下安装VirtualBox

  参考文档：

>



```bash
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
echo deb https://mirrors.tuna.tsinghua.edu.cn/virtualbox/apt/ xenial contrib|sudo tee /etc/apt/sources.list.d/virtualbox.list
sudo apt-get install virtualbox virtualbox-ext-pack -y 
```



{% hint style="info" %}
注意：这里需要安装`virtualbox-ext-pack`，否则无法虚拟机开启远程桌面，就无法安装系统
{% endhint %}

## 命令行安装Ubuntu系统

参考文档：

>

### 下载Ubuntu安装盘ISO镜像

```bash
mkdir -p ~/VirtualBox\ VMs/
curl --progress-bar -o ~/VirtualBox\ VMs/ubuntu-16.04.4-server-amd64.iso https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/16.04/ubuntu-16.04.4-server-amd64.iso
```

### 创建虚拟机

```bash
# 定义个变量，要创建的虚拟机名称
vm=node1
vboxmanage createvm -name ${vm} -register
```

{% code-tabs %}
{% code-tabs-item title="输出内容：" %}
```bash
Virtual machine 'node1' is created and registered.
UUID: 3d26d492-880a-43c2-ac47-6764abc5ef1d
Settings file: '/home/xiaohui/VirtualBox VMs/node1/node1.vbox'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

###  配置虚拟机硬件信息

```bash
# 查看支持的操作系统，这里我们将记录下Ubuntu_64是我们需要安装的操作系统类型
vboxmanage list ostypes
```

{% code-tabs %}
{% code-tabs-item title="输出内容：" %}
```bash
...
ID:          Ubuntu
Description: Ubuntu (32-bit)
Family ID:   Linux
Family Desc: Linux
64 bit:      false

ID:          Ubuntu_64
Description: Ubuntu (64-bit)
Family ID:   Linux
Family Desc: Linux
64 bit:      true
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
# 查看主机上的IP地址，得到上网网卡的接口名称，这里是 enp8s0
ip addr
```

{% code-tabs %}
{% code-tabs-item title="输出内容：" %}
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp8s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:57:00:f1:7d:c6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.1.16/24 brd 10.0.1.255 scope global enp8s0
       valid_lft forever preferred_lft forever
    inet6 2002:b695:9fa8:0:a57:ff:fef1:7dc6/64 scope global mngtmpaddr dynamic
       valid_lft 14300sec preferred_lft 3500sec
    inet6 fe80::a57:ff:fef1:7dc6/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:fe:ba:2f:f6 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:feff:feba:2ff6/64 scope link
       valid_lft forever preferred_lft forever
494: vboxnet0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 0a:00:27:00:00:00 brd ff:ff:ff:ff:ff:ff
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
# 修改虚拟机的硬件配置
vboxmanage modifyvm ${vm} \
  --memory 3072 \
  --ostype Ubuntu_64 \
  --acpi on \
  --cpus 2 \
  --boot1 dvd \
  --nic1 bridged \
  --bridgeadapter1 enp8s0
```

{% code-tabs %}
{% code-tabs-item title="参数说明" %}
```bash
--memory 3072 # 内存为3G
--ostype Ubuntu_64 # 操作系统类型为Ubuntu 64
--acpi on # 开启ACPI电源控制
--cpus 2 # CPU核心数为双核
--boot1 dvd # 第一启动设备为DVD光驱
--nic1 bridged # 网路类型为桥接模式
--bridgeadapter1 enp8s0 # 网络接口为 enp8s0
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```bash
# 创建一个硬盘, 磁盘大小为64G
vboxmanage createhd --filename ~/VirtualBox\ VMs/${vm}/os.vdi --size 64000
# 添加一个SATA控制器
vboxmanage storagectl ${vm} --name "SATA Controller" --add sata --controller IntelAHCI
# 把创建的硬盘挂载到虚拟机
vboxmanage storageattach ${vm} --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium ~/VirtualBox\ VMs/${vm}/os.vdi
# 创建一个IDE控制器
vboxmanage storagectl ${vm} --name "IDE Controller" --add ide
# 把下载的Ubuntu镜像挂载到虚拟机
vboxmanage storageattach ${vm} --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium ~/VirtualBox\ VMs/ubuntu-16.04.4-server-amd64.iso
```

### 启动虚拟机

```bash
# 开启远程桌面管理
vboxmanage modifyvm ${vm} --vrdeport 3390 --vrde on
# 启动
vboxmanage startvm ${vm} --type headless

# 查看启动虚拟机列表
xiaohui@tanmer-dev:~$ vboxmanage list runningvms
"node1" {3d26d492-880a-43c2-ac47-6764abc5ef1d}

# 查看虚拟机信息
xiaohui@tanmer-dev:~$ vboxmanage showvminfo $vm
Name:            node1
Groups:          /
Guest OS:        Ubuntu (64-bit)
UUID:            3d26d492-880a-43c2-ac47-6764abc5ef1d
Config file:     /home/xiaohui/VirtualBox VMs/node1/node1.vbox
Snapshot folder: /home/xiaohui/VirtualBox VMs/node1/Snapshots
Log folder:      /home/xiaohui/VirtualBox VMs/node1/Logs
Hardware UUID:   3d26d492-880a-43c2-ac47-6764abc5ef1d
Memory size:     3072MB
Page Fusion:     off
VRAM size:       8MB
CPU exec cap:    100%
HPET:            off
Chipset:         piix3
Firmware:        BIOS
Number of CPUs:  2
PAE:             on
Long Mode:       on
Triple Fault Reset: off
APIC:            on
X2APIC:          off
CPUID Portability Level: 0
CPUID overrides: None
Boot menu mode:  message and menu
Boot Device (1): DVD
Boot Device (2): DVD
Boot Device (3): HardDisk
Boot Device (4): Not Assigned
ACPI:            on
IOAPIC:          off
BIOS APIC mode:  APIC
Time offset:     0ms
RTC:             local time
Hardw. virt.ext: on
Nested Paging:   on
Large Pages:     off
VT-x VPID:       on
VT-x unr. exec.: on
Paravirt. Provider: Default
Effective Paravirt. Provider: KVM
State:           running (since 2018-04-18T16:11:32.153000000)
Monitor count:   1
3D Acceleration: off
2D Video Acceleration: off
Teleporter Enabled: off
Teleporter Port: 0
Teleporter Address:
Teleporter Password:
Tracing Enabled: off
Allow Tracing to Access VM: off
Tracing Configuration:
Autostart Enabled: off
Autostart Delay: 0
Default Frontend:
Storage Controller Name (0):            SATA Controller
Storage Controller Type (0):            IntelAhci
Storage Controller Instance Number (0): 0
Storage Controller Max Port Count (0):  30
Storage Controller Port Count (0):      30
Storage Controller Bootable (0):        on
Storage Controller Name (1):            IDE Controller
Storage Controller Type (1):            PIIX4
Storage Controller Instance Number (1): 0
Storage Controller Max Port Count (1):  2
Storage Controller Port Count (1):      2
Storage Controller Bootable (1):        on
SATA Controller (0, 0): /home/xiaohui/VirtualBox VMs/node1/os.vdi (UUID: 76208d22-3257-4496-898a-2bd17e930b43)
IDE Controller (0, 0): /home/xiaohui/VirtualBox VMs/ubuntu-16.04.4-server-amd64.iso (UUID: d724e290-e032-42d6-91eb-03b8e4a18004)
NIC 1:           MAC: 0800276585BE, Attachment: Bridged Interface 'enp8s0', Cable connected: on, Trace: off (file: none), Type: Am79C973, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 2:           disabled
NIC 3:           disabled
NIC 4:           disabled
NIC 5:           disabled
NIC 6:           disabled
NIC 7:           disabled
NIC 8:           disabled
Pointing Device: PS/2 Mouse
Keyboard Device: PS/2 Keyboard
UART 1:          disabled
UART 2:          disabled
UART 3:          disabled
UART 4:          disabled
LPT 1:           disabled
LPT 2:           disabled
Audio:           enabled (Driver: ALSA, Controller: AC97, Codec: STAC9700)
Audio playback:  disabled
Audio capture: disabled
Clipboard Mode:  disabled
Drag and drop Mode: disabled
Session name:    headless
Video mode:      640x480x16 at 0,0 enabled
VRDE:            enabled (Address 0.0.0.0, Ports 3390, MultiConn: off, ReuseSingleConn: off, Authentication type: null)
Video redirection: disabled
USB:             disabled
EHCI:            disabled
XHCI:            disabled

USB Device Filters:

<none>

Available remote USB devices:

<none>

Currently Attached USB Devices:

<none>

Bandwidth groups:  <none>

Shared folders:  <none>

VRDE Connection:    not active
Clients so far:     0

Capturing:          not active
Capture audio:      not active
Capture screens:    0
Capture file:       /home/xiaohui/VirtualBox VMs/node1/node1.webm
Capture dimensions: 1024x768
Capture rate:       512 kbps
Capture FPS:        25
Capture options:

Guest:

Configured memory balloon size:      0 MB
OS type:                             Ubuntu_64
Additions run level:                 0

Guest Facilities:

No active facilities.
```

### 连接虚拟机远程桌面

映射主机的3390端口到本地电脑，然后用 RDP 远程桌面连接工具连接虚拟机远程桌面，Mac 系统推荐使用 [Remotix](http://www.ifunmac.com/2017/10/remotix-5/)



![](.gitbook/assets/image%20%2824%29.png)



![](.gitbook/assets/image%20%285%29.png)

如何装Ubuntu，这里略过...

### 卸载Ubuntu安装ISO镜像

```bash
vboxmanage storageattach $vm --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium none
```

### 关闭远程桌面

```bash
# 运行状态，关闭远程桌面
vboxmanage controlvm $vm vrde off
# 运行状态，开启远程桌面
vboxmanage controlvm $vm vrde on
# 修改配置，关闭远程桌面（需要关闭虚拟机）
vboxmanage modifyvm ${vm} --vrde off
# 修改配置，开启远程桌面（需要关闭虚拟机）
vboxmanage modifyvm ${vm} --vrde on

```

### 备份虚拟机系统\(建立快照\)

```bash
vboxmanage snapshot $vm take backup-20180418-001
```

### 还原虚拟机系统\(恢复快照\)

```bash
vboxmanage snapshot $vm restore backup-20180418-001
```

### 查看快照列表

```bash
xiaohui@tanmer-dev:~$ vboxmanage snapshot node1 list
   Name: init-ubuntu-20180418 (UUID: 1960cff6-e827-4a8f-a0df-8bba2b580f23) *
```

### 关闭虚拟机

```bash
vboxmanage controlvm $vm poweroff
```

### 注销虚拟机

```bash
# 注销，不删除数据
vboxmanage unregister $vm
# 注销，并删除数据
vboxmanage unregister $vm --delete
```

### 克隆创建的Ubuntu系统

当我们需要创建多个虚拟机Ubuntu系统时，我们没有必要每次都重复上面的安装步骤。我们可以用VirtualBox的克隆功能，快速复制个Ubuntu系统

通过快照克隆系统

```bash
# 通过node1的init-ubuntu-20180814快照克隆系统
vboxmanage clonevm node1 --snapshot init-ubuntu-20180418 --name node2 --register
# 启动node2
vboxmanage startvm node2 --type headless
```

从`node1`克隆为`node2`之后，她的hostname没有改变，这里需要修改：

```bash
sudo sed -i 's/node1/node2/' /etc/hosts
sudo sed -i 's/node1/node2/' /etc/hostname
sudo hostname node2
```

IP地址的设置也需要修改

```bash
sudo vi /etc/networking/interfaces
```

启动克隆后的虚拟机，如果是Bridged网路可能会无法上网。可以通过一下方式解决：

```bash
vboxmanage controlvm node2 poweroff
vboxmanage modifyvm node2 --nic1 none
vboxmanage modifyvm node2 --nic1 bridged
vboxmanage startvm node2 --type headless
```



