# yum-upgrade-CentOS7-kernel
用标准yum方式升级CentOS 7内核并启用BBR及同步时钟

之前有玩家看完用标准apt方式升级Ubuntu内核并启用BBR的说明,就抱怨说他用的是CentOS,不能用这个apt的方法,逼着我要写一个几键升级CentOS的官方方法,我这个小白虚汗都出来了,的确CentOS有一大票服务器玩家说它稳定,嚷嚷着一定非它不用,但是这个CentOS也的确难用,小白我虽然有些心得,但不到万不得已是不愿写它的几键的,因为太费劲    
        
下面是针对CentOS 7的内核升级步骤,CentOS 6没有试过,玩家有兴趣可自行探索,CentOS 8应该不用升,至于下面的命令,玩家也是有兴趣可以深究,没有兴趣可以猛敲即可       
       
内核小版本升级(这个可跳过直奔大版本升级):      
         
查看当前和可升级版本命令     
     
转用root用户           
    
$su          
       
然后输入     
       
#yum list kernel      
      
输出大概是下面这个样子:       
Loaded plugins: fastestmirror       
Loading mirror speeds from cached hostfile        
 * base: mirror.it.xxx.xx         
 * extras: mirror.it.xxx.xx         
 * updates: mirror.esecuredata.com          
Installed Packages       
kernel.x86_64                        3.10.0-1127.el7                             @anaconda             
kernel.x86_64                        3.10.0-1127.19.1.el7                        @updates            
       
打入升级命令             
        
#yum update kernel -y          
             
重启并检查         
        
#reboot        
       
#uname -a     
     
内核大版本升级         
   
转用root用户         
         
$su     
       
然后载入公钥命令     
    
#rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org          
    
升级安装ELRepo命令         
     
#rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm        
      
载入elrepo-kernel元数据命令      
       
#yum --disablerepo=\* --enablerepo=elrepo-kernel repolist         
     
查看可用的rpm包命令       
         
#yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel*              
          
输出大概是下面这个样子:         
Loaded plugins: fastestmirror          
Loading mirror speeds from cached hostfile             
 * elrepo-kernel: muug.xx             
Installed Packages          
kernel.x86_64                                3.10.0-1127.el7                 @anaconda                  
kernel.x86_64                                3.10.0-1127.19.1.el7            @updates             
kernel-tools.x86_64                          3.10.0-1127.19.1.el7            @updates               
kernel-tools-libs.x86_64                     3.10.0-1127.19.1.el7            @updates                
Available Packages              
kernel-lt.x86_64                             4.4.241-1.el7.elrepo            elrepo-kernel           
kernel-lt-devel.x86_64                       4.4.241-1.el7.elrepo            elrepo-kernel          
kernel-lt-doc.noarch                         4.4.241-1.el7.elrepo            elrepo-kernel              
kernel-lt-headers.x86_64                     4.4.241-1.el7.elrepo            elrepo-kernel                
kernel-lt-tools.x86_64                       4.4.241-1.el7.elrepo            elrepo-kernel                 
kernel-lt-tools-libs.x86_64                  4.4.241-1.el7.elrepo            elrepo-kernel           
kernel-lt-tools-libs-devel.x86_64            4.4.241-1.el7.elrepo            elrepo-kernel              
kernel-ml.x86_64                             5.9.6-1.el7.elrepo              elrepo-kernel                
kernel-ml-devel.x86_64                       5.9.6-1.el7.elrepo              elrepo-kernel              
kernel-ml-doc.noarch                         5.9.6-1.el7.elrepo              elrepo-kernel                   
kernel-ml-headers.x86_64                     5.9.6-1.el7.elrepo              elrepo-kernel                
kernel-ml-tools.x86_64                       5.9.6-1.el7.elrepo              elrepo-kernel             
kernel-ml-tools-libs.x86_64                  5.9.6-1.el7.elrepo              elrepo-kernel              
kernel-ml-tools-libs-devel.x86_64            5.9.6-1.el7.elrepo              elrepo-kernel              
        
说明：          
     
lt  ：long term support，长期支持版本；        
   
ml：mainline，主线版本；    
      
安装最新版本的kernel命令(一般选用主线版本,因为lt版只有4.4.241,不符合BBR支持的最低内核要求4.9,没有升的意义)        
      
#yum --disablerepo=\* --enablerepo=elrepo-kernel install  kernel-ml.x86_64  -y              
        
删除旧版本工具包命令           
        
#yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64  -y         
     
安装新版本工具包命令      
          
#yum --disablerepo=\* --enablerepo=elrepo-kernel install kernel-ml-tools.x86_64  -y      
     
查看内核插入顺序命令       
        
#awk -F \' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg          
          
输出大概是下面这个样子:     
0 : CentOS Linux (5.9.6-1.el7.elrepo.x86_64) 7 (Core)           
1 : CentOS Linux (3.10.0-1127.19.1.el7.x86_64) 7 (Core)          
2 : CentOS Linux (3.10.0-1127.el7.x86_64) 7 (Core)         
3 : CentOS Linux (0-rescue-a819de6cf20247929196a0f137d5799c) 7 (Core)         
      
说明：默认新内核是从头插入，默认启动顺序也是从0开始（当前顺序还未生效），或者使用以下命令查看：            
        
#grep "^menuentry" /boot/grub2/grub.cfg | cut -d "'" -f2               
        
输出大概是下面这个样子(输出没有前面的命令直观显示顺序号码,所以不建议):          
CentOS Linux (5.9.6-1.el7.elrepo.x86_64) 7 (Core)          
CentOS Linux (3.10.0-1127.19.1.el7.x86_64) 7 (Core)          
CentOS Linux (3.10.0-1127.el7.x86_64) 7 (Core)             
CentOS Linux (0-rescue-a819de6cf20247929196a0f137d5799c) 7 (Core)         
          
补充说明上面两个命令的文件 /etc/grub2.cfg 和 /boot/grub2/grub.cfg 内容一致。        
       
查看当前实际启动顺序命令          
         
#grub2-editenv list           
     
输出大概是下面这个样子:       
saved_entry=CentOS Linux (3.10.0-1127.19.1.el7.x86_64) 7 (Core)         
       
设置默认启动命令(注意:内核名字要原封不动拷贝前面命令输出的第一行那个名字)          
       
#grub2-set-default 'CentOS Linux (5.9.6-1.el7.elrepo.x86_64) 7 (Core)'           
       
再查看当前实际启动顺序命令           
         
#grub2-editenv list         
     
输出大概改变成下面这个样子:        
saved_entry=CentOS Linux (5.9.6-1.el7.elrepo.x86_64) 7 (Core)          
         
或者直接设置启动数值命令(这个比拷贝粘贴名字简单多了!!!)         
       
#grub2-set-default 0　　// 0代表当前第一行，也就是上面的5.9.6版本那一行内容          
        
再查看当前实际启动顺序命令         
      
#grub2-editenv list         
      
输出大概改变成下面这个样子(这时就只显示数字了,不过实际效果是一样的,所以看个人喜欢,只选一种设置默认启动命令即可):         
saved_entry=0       
    
重启并检查内核版本是否已经升级    
   
#reboot    
        
#uname -a         
      
------------启用 BBR------------     
虽然内核已经升级好了，但还没有正式装载 BBR 模块，还无法在可用拥塞算法中查到。运行以下命令装载 BBR     
     
改用root用户     
$su     
#modprobe tcp_bbr    
#echo "tcp_bbr" | sudo tee -a /etc/modules-load.d/modules.conf          
      
然后输入命令，就能看到 bbr 了        
#sysctl net.ipv4.tcp_available_congestion_control            
         
输出大概是这个样子        
net.ipv4.tcp_available_congestion_control = reno cubic bbr           
     
看到这个输出后就可以按照漫谈BBR一键加速脚本repo的方式开启BBR了           
      
------------CentOS 7 时钟同步------------         
改用root用户        
$su         
      
查看系统时间命令      
#date       
        
查看BIOS时间命令        
#hwclock        
   
安装ntpdate工具命令  
#yum -y install ntp ntpdate  
  
停止NTP服务命令  
#service ntpd stop  

设置系统时间与网络时间同步命令  
#ntpdate time-a.nist.gov  
  
启动NTP服务命令  
#service ntpd start  
  
将系统时间写入BIOS时间  
#hwclock --systohc  


