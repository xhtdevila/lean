1、配置Git SSH密钥

由于本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以必须要让github仓库认证你SSH key，在操作之前，需要先在服务器上生成SSH key。
我们先去根目录下使用命令：

cd ~
ssh-keygen -t rsa
这里会要你命名密匙名称(这里建议使用默认名称)，然后连续按几次Enter，这时候会在/root/.ssh文件夹生成2个ssh密钥，然后我们查看公钥id_rsa.pub。

cat ~/.ssh/id_rsa.pub
查看后，再复制下公钥，然后打开Github官网，进入https://github.com/settings/ssh/new，Title随便填，然后Key填入刚刚复制的密匙，最后点击Add SSH Key添加即可。
![1](1.png)

2、建立私人仓库
我们需要先访问https://github.com/new，新建一个仓库用来存放备份文件，名称自己随意，记得下面一定要勾选Private，也就是私人仓库。不然你辛辛苦苦备份的小姐姐就要被别人偷走了。
![1](2.png)

3、配置本地仓库
由于博主是用来备份网站，所以需要备份文件夹为/home/www.moerats.com，也就是把该文件夹定为本地仓库，使用命令：

#进入需要备份的文件夹
cd /home/www.moerats.com
#初始化你的github仓库
git init
#关联到远程github仓库
git remote add origin git@github.com:iiiiiii1/MOERATS.git
关联仓库的时候，后面可以用HTTPS链接也可以用SSH，这里强烈建议选择SSH，安全性很高。
![1](3.png)
4、初次备份

#进入备份的文件夹
cd /home/www.moerats.com
#把目录下所有文件更改状况提交到暂存区，包括增，删，改。
git add -A
#提交更改的说明，说明随意了，这里为backsite
git commit -m "backsite"
#开始推送到Github
git push -u origin master
推送的时候可能会提示The authenticity of host 'github.com' can't be established.信息，直进yes即可。

然后可以看到仓库的备份文件了。
![1](4.png)

5、设置定时备份
在根目录先新建一个bash脚本：

nano ~/gitback.sh
代码如下：

#!/bin/bash
#进入到网站根目录，记得修改为自己的站点
cd /home/xxx.com
#将数据库导入到该目录，这里以mysql为例，passwd为数据库密码，typecho为数据库名称，typecho.sql为备份的数据库文件
mysqldump -uroot -ppasswd typecho > typecho.sql
git add -A
git commit -m "backsite"
git push -u origin master
然后编辑好了后，使用ctrl+x，y保存退出。再测试下脚本，使用命令：

bash ~/gitback.sh
脚本没问题的话，再设置为每天05:15执行一次：

#并将运行日志输出到根目录的siteback.log文件
echo "15 05 * * * bash ~/gitback.sh > ~/siteback.log 2>&1 &" > bt.cron
crontab bt.cron
rm -rf bt.cron
最后使用crontab -l命令查看添加成功没。成功的话，就基本上算完成了。

推送失败
如果你将本地文件夹推送到Github失败的话，常见原因有2种，具体如下。

1、邮件问题

报错提示：Your push would publish a private email address.
这里可能是你将你的邮件地址私密了，解决方法如下：

#方法一，如果你想一直保持私密，可以选择方法二
访问https://github.com/settings/emails，将Keep my email address private的勾去掉。

#方法二
1、访问https://github.com/settings/emails，将Block command line pushes that expose my email的勾去掉。
2、设置你的github邮箱，修改成自己的再运行命令：
git config --global user.email "admin@moerats.com"
2、密匙问题

报错提示：Permission denied (publickey).
大概的原因就是，你设置密匙的时候改成了其它名称，而ssh默认只读取id_rsa，所以会显示没权限。解决方法如下：

#方法一
进入根目录的.ssh文件夹，将你的github密匙文件，重新更名为id_rsa。

#方法二
将github密匙添加到ssh agent，比如密匙名称为github，使用命令：
ssh-agent bash
ssh-add ~/.ssh/github

## rclone备份

配置完成后，手工备份一次，测试一下效果。这里我要将服务器/hi_ktsee_com/attachments/201705目录下的所有文件，备份到网盘中的/hi_ktsee_com/attachments/201705中，执行：

rclone copy --ignore-existing /hi_ktsee_com/attachments/201705 gdrive:hi_ktsee_com/attachments/201705
这里使用了copy命令，主要是由于备份是单向的。--ignore-existing则是忽略掉网盘中已经备份过的文件，相当于增量备份了。

稍等片刻，如果没有任何错误信息返回，那么这次备份就完成了，可以在网盘中看到对应备份文件。用rclone备份真的的是很简单。

设置按月备份脚本
由于我这服务器的文件是按月归档到不同文件夹，文件夹命名格式为"年月"，如201705，那么每个月只需要对当月目录进行增量备份即可，避免了每次备份rclone都要重新检查所有目录。

比如现在是2017年5月，那么今天备份脚本就应该执行：

rclone copy --ignore-existing /hi_ktsee_com/attachments/201705 gdrive:hi_ktsee_com/attachments/201705
而到了6月，那么我希望脚本自动执行：

rclone copy --ignore-existing /hi_ktsee_com/attachments/201706 gdrive:hi_ktsee_com/attachments/201706
这时可以编写一个简单的脚本，自动获取当前时间，对应不同备份指令，执行：

vi sync2gdrive.sh
写入脚本内容：

#!/bin/bash
cur_month=$(date +%Y%m)
if [ -d "/hi_ktsee_com/attachments/$cur_month" ]; then
  /usr/local/sbin/rclone copy --ignore-existing /hi_ktsee_com/attachments/$cur_month gdrive:hi_ktsee_com/attachments/$cur_month >> ~/sync2gdrive_$cur_month.log
fi
这里定义了一个变量cur_month获取当前时间，组合成我们需要的目录形式201705，接着调用备份命令对指定目录备份，最后输出到log文件，以便于备份出错时查看错误记录。

设置系统定时器（计划任务）
定时执行脚本有几种方案，包括AT，crontab和systemd.timer。

其中AT常用于只执行一次的任务，虽然结合守护进程atd也可以实现定时效果。crontab之前非常常用，是不错的选择，但是这里对于CentOS7上新的systemd的用法，还是要学习一下，因此这次使用了systemd.timer定时器。

配置定时器
首先进入systemd服务文件存放目录，新建一个sync2gdrive.service文件：

cd /etc/systemd/system
vi sync2gdrive.service
填入内容：

[Unit]
Description=Sync local files to google drive
[Service]
Type=simple
ExecStart=/root/sync2gdrive.sh
User=root
这里/root/sync2gdrive.sh是上一步编写的脚本的路径。另外这里指定了用户为root，因为测试脚本执行环境中，需要调用的配置文件是以root用户的身份生成的，因此这里指定定时脚本也同样以root身份执行。如果你需要指定其他用户身份来执行这个脚本，更改User配置即可。

然后新建sync2gdrive.timer文件：

vi sync2gdrive.timer
填入内容：

[Unit]
Description=Daily sync local files to google drive
[Timer]
OnBootSec=5min
OnUnitActiveSec=1d
TimeoutStartSec=1h
Unit=sync2gdrive.service
[Install]
WantedBy=multi-user.target
OnBootSec表示开机后五分钟后启动
OnUnitActiveSec表示每隔1天执行一次
TimeoutStartSec表示脚本执行后1小时后检查结果，防止备份时间过长，脚本认为没有响应，认为任务失败而中断任务
启用定时器并加入开机启动
这里启用以及加入开机启动，就与其他服务一样，只是注意末位是timer：

systemctl enable sync2gdrive.timer
systemctl start sync2gdrive.timer
这样自动备份就基本配置完成了。
