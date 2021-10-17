# 基于Linux-docker环境下软件部署问题解决的分享

## 一、本地（设备/linux下）问题解决

1.安装matplotlib库（绘制各类可视化图形的命令字字库）

问题点：执行python xy_repro.py时，提示没有matplotlib库

**解决办法**：终端执行命令”pip3 install  -U matplotlib“（参考方法：https://blog.csdn.net/toby54king/article/details/86743463）

**解决过程：**

1.试过通过apt的安装方式、pip的安装方式问题都卡在build状态就不动了（找过卡住不动的原因，有通过降低python版本解决的，中间也试过把设备上的python3.6给删了，最后还是一样的效果）

2.把python3.6删掉后，再次重新安装，找到了另外的解决办法，即终端执行命令“终端执行命令“pip3 install  -U matplotlib”进行安装。

2.安装"numpy"库（开源的Python科学计算基础库，完成基础数值计算）

**问题点**：安装完"matplotlib库"之后再次执行"python xy_repro.py"时,提示需要安装numpy库。

**解决办法**：终端执行命令："pip3 install numpy"（参考方法：https://blog.csdn.net/guotianqing/article/details/114883421）

![](E:\Desktop\problem\1\1.png)

3.安装"six"库（专门用来兼容python2和python3的库）

**问题点：**安装完"numpy库"之后再次执行"python xy_repro.py"时,提示需要安装six库。

**解决过程：**终端执行命令 pip install six -i http://pypi.douban.com/simple --trusted-host pypi.douban.com（参考方法：https://blog.csdn.net/p1279030826/article/details/110234104?ops_request_misc=&request_id=&biz_id=102&utm_term=ModuleNotFoundError:%20No%20module&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-110234104.nonecase&spm=1018.2226.3001.4187）

4.设置环境变量

**问题点：**安装完"six库"之后再次执行"python xy_repro.py"时,终端提示"Illegal instruction (core dumped)"（非法指令集）。

**解决办法：**

1.终端执行命令：sudo vim ~/.bashrc。

2.末尾添加环境变量export OPENBLAS_CORETYPE=ARMV8，保存并关闭文件。

3.终端运行source ~/.bashrc

**解决过程：**

1.网上找解决办法，都说是cpu没有avx2指令集，后来用命令"cat /proc/cpuinfo"查了下，确实是没有。

![](E:\Desktop\problem\1\2.png)

2.再次查找解决办法，最后找到修改环境变量的解决办法，问题最终解决，python 程序得以执行，本地（设备）上，软件能各个功能能正常使用。

## 二、docker上问题解决

1.显示问题

问题背景：由于标定工具执行可执行程序几乎每一步都需要图形化显示，而docker本身是类似于一台服务器，不具备显示功能。通过查阅资料，需要在docker上进行显示，总共有两种方法，即"宿主机方式"和启动容器时"配置参数方式"（方法参考：https://blog.csdn.net/liujiandu101/article/details/80027381）。

解决方法：

1.在本地（设备）终端执行命令 "sudo docker run -it --env="DISPLAYF" -v /tmp/.X11-unix:/tmp/.unix [镜像ID]"，执行命令后会创建一个新的容器。

2.创建容器后，这时就可以把软件拷贝到容器里面，进行验证，同时也可以通过在容器内，终端执行命令：apt-get install xarclock（安装时钟插件（在linux下是xclock）），安装完成后，终端输入xarclock则终端会显示一个实时时钟界面，说明方法正确，docker实现了可视化。

解决过程：

1.首先采用的是宿主机的方式，也就是通过ip地址映射来显示（具体操作方法见参考链接），结果在执行"sudo systemctl restart lightdm"命令重启lightdm服务的时候报错，后来通过执行命令"systemctl status lightdm.service"发现lightdm已经挂了，接着继续查挂的原因，发现网上没有合理的解决办法，最后决定采用启动容器时，配置参数的方式进行实现。

2.按照参考链接上的方法，执行，可以创建容器，但是发现可视化效果并未生效，后来经过查找，问题最终得以解决（参考链接：https://www.jianshu.com/p/a31a7eeee291）

![](E:\Desktop\problem\3.png)



![](E:\Desktop\problem\4.png)

注意点：

1.执行xarclock命令，如果提示终端提示"No protocal specific"，则说明主机（设备）中没有执行"xhost+命令"，同时需要执行命令"sudo apt-get install x11-xserver-utils"安装"x11"服务。

2.python问题

2.1 numpy库安装问题

**问题点：**执行 python 程序，提示没有"numpy库"

**解决方法：**

1.命令终端执行命令"apt-get install freetds-devel.x86_64"

2.命令终端执行命令"python -m pip install  numpy --upgrade --force pip"，成功安装

**解决过程：**

1.根据报错点从前到后依次查方法解决。

2.2 非法指令集问题

**问题点：**安装完"numpy库"之后，再次执行python程序，报错"Illegal instruction （core dumped）"非法指令集

**解决方法：**

1.终端执行命令：sudo vim ~/.bashrc。

2.末尾添加环境变量export OPENBLAS_CORETYPE=ARMV8，保存并关闭文件。

3.终端运行source ~/.bashrc

2.3 matplotlib库安装问题

**问题点：**解决非法指令集问题后，再次执行python程序，报错，需要安装"matplotlib库"

**解决方法：**命令终端执行命令：pip install matplotlib==1.5.1（指定版本安装方式）

**解决过程：**

1.首先采用在本地尝试过的方法进行安装，发现都安装不成功。

2.重新找方法，得以解决，安装成功。

![](E:\Desktop\problem\5.jpg)

2.4 opencv库问题

**问题点：**再次执行python程序，报错，"User .........Try installing cairociff"。

**解决方法：**命令终端执行命令：pip3 install --user -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python。

**解决过程：**

1.刚开始找到的方法，执行 pip install opencv-contrib-python命令安装，执行结束后，还是报同样错误。

2.再次查找，找到解决方法，问题解决。

2.5 格式不支持问题

问题点：执行python程序，报不支持jpg格式

![](E:\Desktop\problem\6.jpg)

解决方法：

1.执行pip install pillow -i https://pypi.tuna.tsinghua.edu.cn/simple，安装pillow（图像处理）库成功。

2.再次执行，则报错，不支持JPG格式图片，后来把python程序里的jpg格式改成了png，问题解决。

注意点：

1.在执行python程序过程中，可能会有警告：QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to ‘/tmp/runtime-root，解决方法（参考链接：https://blog.csdn.net/yhjahjj1314/article/details/118090251）



备注：

1.docker中执行"apt-get install net-tools"安装ifconfig命令后，发现在执行ping命令时候，报没有这个命令，此时，执行"apt-get install -y inetutils-ping"命令方式进行安装（参考链接：https://blog.csdn.net/whatday/article/details/103062396)）