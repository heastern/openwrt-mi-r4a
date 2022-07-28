## 小米 R4A 千兆版本 刷 openwrt(22.7.7)  源码编译/breed直刷   (最新芯片 EON EN25QX64 1c7117)

## 成果

先上图

旧版breed看到的是1c7117，我不恢复来看了。

![](https://github.com/heastern/openwrt-mi-r4a/blob/main/img/1.png?raw=true)
![](https://github.com/heastern/openwrt-mi-r4a/blob/main/img/2.png?raw=true)

## 编译前准备：

1、系统 ——`ubuntu-20.04.3-desktop-amd64.iso`  （自行百度下载）

之前使用的是 `ubuntu-16.04.7-desktop-amd64.iso`  但是对编译源码需要使用到的依赖安装很麻烦，可以说是安不上，`bebian11`也安不上

2、openwrt 源码 ——   [openwrt 源码](https://github.com/coolsnowwolf/lede)

     使用的是此源码，里面有编译教程

3、最好有梯子，否则听说会很慢，我全程用的梯子，没有做任何的换源操作。

## 编译步骤：

1、安装编译`openwrt`需要的依赖

    [openwrt 源码](https://github.com/coolsnowwolf/lede)  此源码地址里面有教程如何安装，就几个命令

2、下载源码

3、添加一些 `openwrt` 需要使用到的第三方包（比如梯子什么的，自己在里面找）——  如果不需要任何三方包，请直接跳到 第4步

  [openwrt 第三方包](https://github.com/kenzok8/openwrt-packages)这个里面有

> ```
> sed -i '$a src-git kenzo https://github.com/kenzok8/openwrt-packages' feeds.conf.default  
> sed -i '$a src-git small https://github.com/kenzok8/small' feeds.conf.default  
> git pull
> ```

4、执行安装 `feeds   `

> ```
> ./scripts/feeds update -a  
> ./scripts/feeds install -a
> ```

5、复制R4A需要的一些配置为`.config`文件

    [R4A 简单配置](https://github.com/unkaer/Actions-OpenWrt-Xiaomi-R4A/blob/main/ra4.config)

> ```
> CONFIG_TARGET_ramips=y  
> CONFIG_TARGET_ramips_mt7621=y  
> CONFIG_TARGET_ramips_mt7621_DEVICE_xiaomi_mi-router-4a-gigabit=y  
> CONFIG_PACKAGE_6in4=y  
> CONFIG_PACKAGE_ip6tables=y  
> CONFIG_PACKAGE_ipv6helper=y  
> CONFIG_PACKAGE_kmod-ipt-nat6=y  
> CONFIG_PACKAGE_kmod-iptunnel=y  
> CONFIG_PACKAGE_kmod-iptunnel4=y  
> CONFIG_PACKAGE_kmod-nf-nat6=y  
> CONFIG_PACKAGE_kmod-sit=y  
> CONFIG_PACKAGE_luci-proto-ipv6=y  
> CONFIG_PACKAGE_odhcp6c=y  
> CONFIG_PACKAGE_odhcp6c_ext_cer_id=0  
> CONFIG_PACKAGE_odhcpd-ipv6only=y  
> CONFIG_PACKAGE_odhcpd_ipv6only_ext_cer_id=0
> ```

5.1、如果要下载为`breed`直刷版本，请往后看。

6、`make menuconfig`

   如果需要安装图形界面，请把这个勾上。

   ![](https://github.com/heastern/openwrt-mi-r4a/blob/main/img/3.png?raw=true)

7、`make download -j8`

   先下载需要的包，8代表8线程

    [检查是否下载完毕的方法](https://github.com/xiaorouji/openwrt-passwall/discussions/1603)

> ```
> find dl -size -1024c -exec ls -l {} \;  
> #此命令可以列出下载不完整的文件（根据我多次编译的经验得出小于1k的文件属于下载不完整），如果存在这样的文件可以使用find dl -size -1024c -exec rm -f {} ;命令将它们删除，然后重新执行make download下载并反复检查，确认所有文件完整可大大提高编译成功率，避免浪费时间。（如果有问题再次下载）
> ```

8、一切就绪，开始编译

   `make -j1 V=s -> log.txt`   第一次编译就使用j1，否则会出现一些你想不到的问题

   为何要使用V=s：          为了看详细信息

   为何要重定向到log.txt：看日志，个人习惯，日志多的时候看着方便

9、坐等，我是编译1-2小时才完成的。第一次编译很慢，以后编译就快了

10、编译完成会有这几个文件（`222.bin`和`3333.bin`不是）

       ![](https://github.com/heastern/openwrt-mi-r4a/blob/main/img/4.png?raw=true)

## 问题

### 问题1：

`openwrt` 官方说不能给 `EN25QX 128`安装，但是`issue`里面有人解决了。

[xiaomi-4a-gigabit-edition has a new flash which is EN25QX128@44Mhz cause a endless reboot · Issue #9442 · openwrt/openwrt · GitHub](https://github.com/openwrt/openwrt/issues/9442#issuecomment-1153677878)

解决方案：

添加此文件：`target/linux/generic/pending-5.4/477-mtd-spi-nor-add-eon-en25qx128a-en25qh16-k51x.patch`

内容：

> ```
> --- a/drivers/mtd/spi-nor/spi-nor.c  
> +++ b/drivers/mtd/spi-nor/spi-nor.c  
> @@ -2240,8 +2240,10 @@ static const struct flash_info spi_nor_i  
> { "en25p64", INFO(0x1c2017, 0, 64 * 1024, 128, 0) },  
> { "en25q64", INFO(0x1c3017, 0, 64 * 1024, 128, SECT_4K) },  
> { "en25q128", INFO(0x1c3018, 0, 64 * 1024, 256, SECT_4K) },  
> 
> + { "en25qx128a", INFO(0x1c7118, 0, 64 * 1024, 256, SECT_4K) },  
>   { "en25q80a", INFO(0x1c3014, 0, 64 * 1024, 16,  
>   SECT_4K | SPI_NOR_DUAL_READ) },  
> + { "en25qh16", INFO(0x1c7015, 0, 64 * 1024, 32, 0) },  
>   { "en25qh32", INFO(0x1c7016, 0, 64 * 1024, 64, 0) },  
>   { "en25qh64", INFO(0x1c7017, 0, 64 * 1024, 128,  
>   SECT_4K | SPI_NOR_DUAL_READ) },  
> 
> ```

注：

1、`issue`中会给他编译好的，需要梯子才能下载，而且不支持`breed`。可通过如下方法操作

文件放入到`/tmp`文件中，之后执行   `mtd -e OS1 -r write 文件名 OS1`即可刷写成功

### 问题2：

如何在源码基础上修改为breed直刷。

解决方案：

`openwrt` 源码修改`breed`直刷教程

[GitHub - unkaer/Actions-OpenWrt-Xiaomi-R4A: 自己编译小米R4A千兆breed直刷版用](https://github.com/unkaer/Actions-OpenWrt-Xiaomi-R4A)

执行命令即可：

> ```shell
> ### 修改为R4A千兆版Breed直刷版
> ## mt7621_xiaomi_mir3g-v2.dts 好像被改成了 mt7621_xiaomi_mi-router-4a-3g-v2.dtsi  测试一下
> ## 1.修改 mt7621_xiaomi_mir3g-v2.dts
> export shanchu1=$(grep  -a -n -e '&spi0 {' target/linux/ramips/dts/mt7621_xiaomi_mi-router-4a-3g-v2.dtsi|cut -d ":" -f 1)
> export shanchu2=$(grep  -a -n -e '&pcie {' target/linux/ramips/dts/mt7621_xiaomi_mi-router-4a-3g-v2.dtsi|cut -d ":" -f 1)
> export shanchu2=$(expr $shanchu2 - 1)
> export shanchu2=$(echo $shanchu2"d")
> sed -i $shanchu1,$shanchu2 target/linux/ramips/dts/mt7621_xiaomi_mi-router-4a-3g-v2.dtsi
> grep  -Pzo '&spi0[\s\S]*};[\s]*};[\s]*};[\s]*};' target/linux/ramips/dts/mt7621_youhua_wr1200js.dts > youhua.txt
> echo "" >> youhua.txt
> echo "" >> youhua.txt
> export shanchu1=$(expr $shanchu1 - 1)
> export shanchu1=$(echo $shanchu1"r")
> sed -i "$shanchu1 youhua.txt" target/linux/ramips/dts/mt7621_xiaomi_mi-router-4a-3g-v2.dtsi
> rm -rf youhua.txt
> ## 2.修改mt7621.mk
> export imsize1=$(grep  -a -n -e 'define Device/xiaomi_mir3g-v2' target/linux/ramips/image/mt7621.mk|cut -d ":" -f 1)
> export imsize1=$(expr $imsize1 + 2)
> export imsize1=$(echo $imsize1"s")
> sed -i "$imsize1/IMAGE_SIZE := .*/IMAGE_SIZE := 16064k/" target/linux/ramips/image/mt7621.mk
> ```

### 问题3：

刷入最新版之后WIFI性能极差，网速差，信号差

解决方案：

可通过修改mac解决，但是我不会。

使用breed刷入的话WIFI无影响。

### 问题4：

openwrt第三方包如何进行选择

> ```shell
> make menuconfig
> ```

之后就可以通过方向键选择了

![](https://github.com/heastern/openwrt-mi-r4a/blob/main/img/5.png?raw=true)

注：使用空格进行是否需要，*：添加到固件   M：只是编译，固件里面不会有

### 问题5：

我想要编译好的，有没有。

1.不能使用breed直刷的，请自行把文件后缀改成【.bin】： 

    https://wwz.lanzout.com/iIisA08k00va

    密码:ddh4

2.使用breed直刷的，请自行把文件后缀改成【.bin】：

   https://wwz.lanzout.com/iZgjA08k02hi

   密码:b8be

## 参考

1、[xiaomi-4a-gigabit-edition has a new flash which is EN25QX128@44Mhz cause a endless reboot · Issue #9442 · openwrt/openwrt · GitHub](https://github.com/openwrt/openwrt/issues/9442#issuecomment-1153677878)

2、[GitHub - coolsnowwolf/lede: Lean&#39;s OpenWrt source](https://github.com/coolsnowwolf/lede)

> 我是在这个commit编译的  
> commit 3211a973de1351e8387eced77526190947e39598

3、[GitHub - unkaer/Actions-OpenWrt-Xiaomi-R4A: 自己编译小米R4A千兆breed直刷版用](https://github.com/unkaer/Actions-OpenWrt-Xiaomi-R4A)

4、[Xiaomi 4A 3-Gigaport cannot flash OpenWrt 21.02 - #6 by 123serge123 - Installing and Using OpenWrt - OpenWrt Forum](https://forum.openwrt.org/t/xiaomi-4a-3-gigaport-cannot-flash-openwrt-21-02/122878/6)

5、[GitHub - kenzok8/openwrt-packages: openwrt常用软件包](https://github.com/kenzok8/openwrt-packages)
