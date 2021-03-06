## 创建软件包

这里，以helloworld为例在系统中创建helloworld程序.

步骤：

* 创建helloworld项目，包括源程序helloworld.c和相应的Makefile;
* 创建上层Makefile，描述helloworld信息；
* 修改编译过程中出现的错误；
* helloworld在开发板运行。

## 创建helloworld项目

在Package目录下创建src目录，src目录下创建源程序helloworld.c和相应的Makefile。

源程序helloworld.c
```ruby
#include <stdio.h>
int main()
{
    printf("This is my hello word!\n");

    return 0;
}
```
Makefile
```ruby
helloworld : helloworld.o
    $(CC) $(LDFLAGS) helloworld.o -o helloworld

helloworld.o : helloworld.c
    $(CC) $(CFLAGS) -c helloworld.c

clean :
    rm *.o helloworld
```

## 创建上层Makefile
上层Makefile，描述helloworld信息，信息包括：软件包配置、编译规则和打包方式。

这里是我照框架修改的Makefile,力求用最简单的方式来写，缺少哪里再补充，这样能更加了解这个Makefile。

```ruby
include $(TOPDIR)/rules.mk

PKG_NAME:=helloworld

PKG_BUILD_DIR:=$(COMPILE_DIR)/$(PKG_NAME)

include $(BUILD_DIR)/package.mk

define Package/helloworld
	CATEGORY:=Allwinner
	TITLE:=Helloworld
endef

define Package/helloworld/install
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/$(PKG_NAME) $(1)/bin/
endef

$(eval $(call BuildPackage,helloworld))
```
## 修改编译过程中出现的问题

* 缺少版本号（VERSION）
PKG_VERSION描述软件包的版本，最终生成的包为$(PKG_NAME)$(PKG_VERSION).ipk文件。
例如helloworld0.0.1.ipk

```ruby
make[4]: Entering directory `/home/alanzhou/workspace/tina/out/azalea-m2ultra/compile_dir/target/helloworld-0.0.1'
make[4]: *** No targets specified and no makefile found.  Stop.

```
在编译过程中产生的target/helloworld-0.0.1目录下没有源文件和Makefile,联想到package文件夹下的helloworld.c和相应的Makefile，以及Build/Prepare所做的工作就是创建tina/out/azalea-m2ultra/compile_dir/target/helloworld-0.0.1这个文件夹，还有将src下的文件拷贝过去。

所以，这里加入
```ruby
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef
```

```ruby
Package helloworld is missing dependencies for the following libraries:
libc.so.6
make[3]: *** [/home/alanzhou/workspace/tina/out/sitar-perf1/packages/base/helloworld_0.0.1_sunxi.ipk] Error 
```
最初的想法是去找相关的依赖库，然后
```ruby
find -name libc.so.6
```
发现不同编译器的很多libc.so.6库文件

于是找其他答案，发现[OpenWRT - package missing dependencies when recompiling](https://stackoverflow.com/questions/20190030/openwrt-package-missing-dependencies-when-recompiling)
提到
```ruby
Is your src/ directory actually clean? I suspect it contains an "amldmonitor" executable already which was built for your host. Make sure your src/ directory does not contain any junk like *.o files or final executables.
```

在写底层源文件和Makefile时编译身材

