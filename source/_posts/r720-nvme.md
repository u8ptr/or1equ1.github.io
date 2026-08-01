---
title: 「 家里云」R720从NVME设备引导
date: 2026-08-02
tags: 家里云
---

> 本文所载乃数月前之技术问题，现时信息可能有所变更，敬请 阁下留意
# 前言
我一直以来在用一台Poweredge R720跑PVE作为Homelab使用，至于存储则是便宜大碗的二手sas硬盘，自认为非常有必要组一个RAID来保证可靠性。但pve的安装向导似乎并不支持直接将系统安装在raid池中；如果单独划一块盘给pve系统用又略显奢侈，只得采取一些曲线救国的措施。
![alt text](/images/IMG_6495.JPG)

# 曲线救国
首先想到方案就是搞一块小容量的硬盘插在机器上，可是略有强迫症的我并不想把系统盘也插到前面板来。朋友给我推荐了Intel的傲腾固态，在此之前对这条产品线只是略有耳闻，不过这里也用不到傲腾特有的功能，便宜大碗+宣称的高寿命似乎就足以支持我的选择了。
![alt text](/images/optane.png)
我收到硬盘以后第一时间装上去，并且重装了系统，发现机器并不能正常启动。查阅资料才得知，由于Xeon 12代平台的固件没有集成nvme驱动，导致无法直接从nvme硬盘中引导系统，并且有相当一部份资料建议在U盘中通过Clover引导系统。不过Clover似乎是为了黑苹果设计的引导，虽说可以这里同样能用，但我查到有价值的指引并不多，我也同样没有跑通…… 只得另寻他法

# rEFInd
[rEFInd](https://www.rodsbooks.com/refind/)是我了解到另一个与Clover类似启动管理器，二者同样是基于rEFIt的产物，但前者通用性更强，~~官方但文档也更详尽~~。

> 此步骤需要在安装完成进系统后进行，否则重启后无法引导需要借助`Live CD`工作；不过也可以选择在电脑上装完在插过去

我找了一个小容量的U盘插在R720上，划分了1GB的空间给rEFInd，并给了一个挂载点方便日后维护（不过我从来没有动过）。
``` bash
root@GrayWolf:~# lsblk /dev/sdc
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdc      8:32   1  3.8G  0 disk 
└─sdc1   8:33   1    1G  0 part /rEFInd
root@GrayWolf:~# 
```
而rEFInd可以直接按[文档](https://www.rodsbooks.com/refind/installing.html#linux)将必要的文件添加到u盘中，**并且需要找一个`nvmexpressdxe.efi`（似乎是一个通用文件）追加到`drivers_x64`目录中才能认到盘**。就我的经历而言，安装完成后直接重启进入rEFInd便可以识别到装有PVE的硬盘，不过稳妥起见也可以设置一下`refind.conf`。
``` bash
root@GrayWolf:~# ls /rEFInd/EFI/BOOT/
bootx64.efi  drivers_x64  icons  refind.conf  refind.conf~  tools_x64  vars
root@GrayWolf:~# 
```
```
menuentry "Proxmox VE" {
    icon     /EFI/refind/icons/os_debian.png
    volume   "pve-root"
    loader   /boot/vmlinuz-6.17.4-1-pve
    initrd   /boot/initrd.img-6.17.4-1-pve
    options  "root=YOUR_PVE_ROOT_UUID rw add_efi_memmap"
    disabled
}
```

# 总结
至此，本次的升级便告一段段落，现在看来我在Clover上跑不通的原因或许也是缺少nvmeexpressdxe.efi。而随着~~AI发展的时代潮流~~存储价格暴涨，现在似乎也买不到25元的傲腾固态了，但傲腾的容量和价格大概仍是一个可考量的选项。
话说回来，这台Poweredge R720主要以折腾为主，iDRAC可以让我不用走到服务器面前、甚至出门在外都可以进行几乎全部不涉及硬件更换的操作，~~而身边朋友还需要单独准备一台ip_kvm甚至委托家人帮忙远程操作~~。但如果追求一个稳定、便捷的Homelab，私以为购买合适的家用或商用产品是一个更好的选择。