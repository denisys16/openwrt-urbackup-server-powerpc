
# How to build custom OpenWrt packages:
#      UrBackup Server
#      UrBackup Client
#      miniDLNA (with thumbnails creation)
# for WD My Book Live NAS (with PowerPC cpu)

Package UrBackup server was created by me.
Package UrBackup client was created by me.
Package miniDLNA was copied from Entware repository and modified (https://github.com/Entware/entware-packages).

miniDLNA from Entware is patched for thumbnails creation support.


1. Download Openwrt SDK
[https://downloads.openwrt.org/releases/22.03.3/targets/apm821xx/sata/openwrt-sdk-22.03.3-apm821xx-sata_gcc-11.2.0_musl.Linux-x86_64.tar.xz](https://downloads.openwrt.org/releases/22.03.3/targets/apm821xx/sata/openwrt-sdk-22.03.3-apm821xx-sata_gcc-11.2.0_musl.Linux-x86_64.tar.xz "https://downloads.openwrt.org/releases/22.03.3/targets/apm821xx/sata/openwrt-sdk-22.03.3-apm821xx-sata_gcc-11.2.0_musl.Linux-x86_64.tar.xz")


2. Extract downloaded archive:
```shell
tar xf openwrt-sdk-22.03.3-apm821xx-sata_gcc-11.2.0_musl.Linux-x86_64.tar.xz
```

3. Go to sdk directory:
```shell
cd openwrt-sdk-22.03.3-apm821xx-sata_gcc-11.2.0_musl.Linux-x86_64
```
4 Search optimization flags in Config-build.in:
    "-Os -pipe -mcpu=464fp"
and replace by
    "-O2 -pipe -mcpu=464fp"
in all places (DEFAULT_TARGET_OPTIMIZATION and TARGET_OPTIMIZATION)


5 Insert a line with custompackages feed in top of feeds.conf.default:
```shell
src-git-full custompackages https://github.com/denisys16/openwrt-custompackages-powerpc.git
```
NOTE: if you want to override packages coming from an existing feed, you must write your custom feed ABOVE the line of the package feed containing the packages you want to override.


6. Inside SDK directory run:
```shell
./scripts/feeds update -a
./scripts/feeds uninstall -a                        (optional for minidlna (to override existing openwrt package))
./scripts/feeds install -f -p custompackages -a     (optional for minidlna (to override existing openwrt package))
./scripts/feeds install -a
make menuconfig
```

7. In menuconfig GUI:

>     Enter Global Build Settings and in the submenu, deselect/exclude the following options:
>             Select all target specific packages by default
>             Select all kernel module packages by default
>             Select all userspace packages by default
>             Unselect  cryptographical signing. Its enabled by default. Turn it off
>     Enter Network
>             Select urbackup-server      (for UrBackup Server only)
>             ### Select urbackup-client      (for UrBackup Client only)
>     Enter Multimedia
>             Select minidlna             (for miniDLNA only )
>     Enter LuCI->Applications
>             Select luci-app-minidlna    (for miniDLNA only)
>     Save
>     Exit

10. Build packages:
```shell
make -j $(nproc) world                                          (для сборки всех отмеченых пакетов)
make package/urbackup-server/{clean,compile}
make package/urbackup-client/{clean,compile}
make package/minidlna/{clean,compile}
```
if you have errors you can try compile with flag V=s to see what happens. For example:
```shell
make package/urbackup-client/{clean,compile} V=s
```


11. You can find results in PATH_2_SDK/bin/package/powerpc_464fp/custompackages/



If you want to see verbose log when package is configured:
```shell
make package/urbackup-server/configure V=s
```



