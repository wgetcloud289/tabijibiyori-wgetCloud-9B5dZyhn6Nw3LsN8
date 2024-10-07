
可以定期对 VMware Cloud Foundation 环境中的相关核心组件（如 SDDC Manager、NSX Manager 以及 vCenter Server 等）创建配置备份，以防止当意外故障或数据丢失时，能够进行恢复。默认情况下，NSX Manager 组件的备份将创建并存储在 SDDC Manager 设备中内置的 SFTP 服务器上，建议单独创建外部备份服务器以代替默认的备份位置。


下面以 SFTP 备份服务器为例，演示 [VCF 核心组件备份配置](https://github.com)的过程。


 



## 一、SFTP 备份服务器



环境中准备了一台 SFTP 备份服务器来作为这些组件的备份位置，基于 CentOS 发行版。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006112423565-989305083.png)](https://github.com)


SFTP 备份服务器上创建一个专用于备份的用户。



```
useradd -m vcf-backup
passwd vcf-backup
chage -M 99999 vcf-backup
```

[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006112754635-1094489743.png)](https://github.com)


SFTP 备份服务器上创建备份的目录，由于 SDDC Manager 和 NSX Manager 组件共享了同一个配置目录，所以只创建了 vcf 和 vcsa 目录。注意，请确定用于备份的目录具有足够的空间。



```
mkdir -p /backup/vcf
mkdir -p /backup/vcsa
chown -R vcf-backup /backup
```

[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006113820589-1975528983.png)](https://github.com)


本地测试一下 SFTP 备份服务器的连接、上传和下载是否正常。



```
sftp vcf-backup@vcf-backup.mulab.local
```

[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006114232477-1385463186.png)](https://github.com)


 



## 二、SDDC Manager 备份



登录 SDDC Manager UI，导航到管理\-\>备份\-\>站点设置。默认情况下，SDDC Manager 中配置了内部 SFTP 服务器，SDDC Manager 和 NSX Manager 的配置文件备份在 SDDC Manager 当中。通过配置为外部 SFTP 服务器，将默认的备份位置调整为上面准备的备份服务器。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006114500292-1457199838.png)](https://github.com)


可以使用以下命令在 SDDC Manager 或者 SFTP 服务器上获取 SFTP 服务器的 SSH 指纹。



```
ssh-keygen -lf <(ssh-keyscan -p 22 -t rsa vcf-backup.mulab.local 2> /dev/null) | cut -d' ' -f2
ssh-keygen -lf /etc/ssh/ssh_host_rsa_key.pub
```

[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006115418103-370051353.png)](https://github.com)


配置 SFTP 服务器地址、备份目录等信息。注意，加密密码短语需注意保存，执行还原过程的时候需要加密密码。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006115812263-1443191463.png)](https://github.com)


保存配置后，SDDC Manager 和 NSX Manager 组件开始配置。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006115859835-2088130609.png)](https://github.com)


可以在任务栏中查看配置状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006120035285-2091389553.png)](https://github.com)


转到 SDDC Manager 配置，点击“立即备份”，开始 SDDC Manager 的配置备份。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006120347137-908067231.png)](https://github.com)


可以在“备份计划”后面点击编辑，创建 SDDC Manager 的备份任务计划。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006120654138-2024773878.png)](https://github.com):[milou加速器](https://xinminxuehui.org)


 



## 三、NSX Manager 备份



登录 NSX Manager UI（VIP），导航到生命周期管理\-\>备份和还原，可以在这里管理 NSX Manager 组件的备份配置。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006120738601-1111813067.png)](https://github.com)


在 SFTP 服务器处点击编辑，能够看到 NSX Manager 默认遵循了 SDDC Manager 中的备份配置，你可以自定义备份配置修改为其他的 SFTP 服务器。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006120811388-686699328.png)](https://github.com)


在计划处点击编辑，可以为 NSX Manager 配置备份创建备份调度任务。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006120829598-1558803413.png)](https://github.com)


SFTP 服务器上查看备份的配置文件。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006121134360-1170365865.png)](https://github.com)


 



## 四、vCenter Server 备份



登录 vCenter Server VAMI（https://vcenter\-ip\-or\-fqdn:5480）管理后台，导航到备份，在这里设置 vCenter Server 组件的备份配置。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006121313034-1580068001.png)](https://github.com)


点击“配置”，填入 SFTP 服务器的参数信息，创建备份调度任务。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006122641184-1361582937.png)](https://github.com)


已成功激活备份调度任务，不过不会立即开始备份，而是会在调度任务设置的时间进行备份。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006122718726-696203735.png)](https://github.com)


可以手动点击“立即备份”，立刻开始 vCenter Server 的配置备份。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006122750266-982520470.png)](https://github.com)


vCenter Server 配置备份完成。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006122954557-1435684427.png)](https://github.com)


SFTP 服务器上查看备份的配置文件。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241006123108648-1473792394.png)](https://github.com)


