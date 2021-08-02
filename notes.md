## Installing crosstool-ng (to build up our own cross-compiling toolchain)

### Needed packages 

`sudo apt install build-essential git autoconf bison flex texinfo help2man gawk \
libtool-bin libncurses5-dev unzip`

- Configure git 

```bash
git config user.name "NAME"
git config user.email "EMAIL"
```

- Copy crosstool-ng locally 

```bash
git clone https://github.com/crosstool-ng/crosstool-ng.git
cd crosstool-ng/
```

- Cherry-pick a working release

```bash
git checkout 79fcfa17`
```

```bash
./bootstrap
```

```bash
./configure --enable-local
make
```

- To get help and then configure crossng for your architecture

```bash 
./ct-ng help 
./ct-ng menuconfig
```

-Once config is done, build the cross compilation toolchain

```bash
./ct-ng build
```
- Export toolchain (default build output path is `$HOME/x-tools/`) in your path

```bash
export PATH=$HOME/x-tools/arm-imx7ulp-linux-uclibcgnueabihf/bin/:$PATH`
```
## Yocto Project, toolchain installation

TODO: cleanup apt-get command to remove unused packages

```bash
sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
build-essential chrpath socat cpio python python3 python3-pip python3-pexpect \
xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \
pylint3 xterm rsync curl
```

(Extracted from bootlin yocto labs, this step may not be needed if the bin folder already exists)

```bash
mkdir ~/bin 
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

- Create a working directory to build your image

```bash
mkdir imx-yocto-bsp
cd imx-yocto-bsp
```

- Configure git on your host machine beforehand
- From Yocto User Guide provided by NXP (YUG)

```bash
repo init -u https://source.codeaurora.org/external/imx/imx-manifest \
-b imx-linux-hardknott -m imx-5.10.35-2.0.0.xml
repo sync
DISTRO=fsl-imx-fb MACHINE=imx7ulpevk source imx-setup-release.sh
```

- Accept EULA

Make sure that the `MACHINE??="<yourmachine>"`  in `./build/conf/local.conf` is `imx7ulpevk`.
I also commented out the unused packages in `./build/conf/bblayers.conf`. Here's my file:

```bash
LCONF_VERSION = "7"

BBPATH = "${TOPDIR}"
BSPDIR := "${@os.path.abspath(os.path.dirname(d.getVar('FILE', True)) + '/../..')}"

BBFILES ?= ""
BBLAYERS = " \
  ${BSPDIR}/sources/poky/meta \
  ${BSPDIR}/sources/poky/meta-poky \
  \
  ${BSPDIR}/sources/meta-openembedded/meta-oe \
  ${BSPDIR}/sources/meta-openembedded/meta-multimedia \
  ${BSPDIR}/sources/meta-openembedded/meta-python \
  \
  ${BSPDIR}/sources/meta-freescale \
  ${BSPDIR}/sources/meta-freescale-3rdparty \
  ${BSPDIR}/sources/meta-freescale-distro \
"

# i.MX Yocto Project Release layers
BBLAYERS += "${BSPDIR}/sources/meta-imx/meta-bsp"
BBLAYERS += "${BSPDIR}/sources/meta-imx/meta-sdk"
#BBLAYERS += "${BSPDIR}/sources/meta-imx/meta-ml"
#BBLAYERS += "${BSPDIR}/sources/meta-imx/meta-v2x"
#BBLAYERS += "${BSPDIR}/sources/meta-nxp-demo-experience"

BBLAYERS += "${BSPDIR}/sources/meta-browser/meta-chromium"
BBLAYERS += "${BSPDIR}/sources/meta-clang"
BBLAYERS += "${BSPDIR}/sources/meta-openembedded/meta-gnome"
BBLAYERS += "${BSPDIR}/sources/meta-openembedded/meta-networking"
BBLAYERS += "${BSPDIR}/sources/meta-openembedded/meta-filesystems"
#BBLAYERS += "${BSPDIR}/sources/meta-qt5"
BBLAYERS += "${BSPDIR}/sources/meta-python2"
```

- Now build the image using

```bash
bitbake fsl-image-machine-test
"WARNING: You are running bitbake under WSLv2, this works properly but you should optimize your VHDX file eventually to avoid running out of storage pace"
```



- Grab a coffee and wait :)

## Flash the SD Card (my setup is using WSL)

I recommend using dd for linux users or Balena Etcher for the Windows fellows.

The image can be found in `./build/tmp/deploy/images/imx7ulpevk/fsl-image-machine-test-imx7ulpevk.wic.bz2`.     

- Copy the image on your windows mounted directory using `scp` (WINDOWS WSL ONLY) and then use Balena Etcher.

- Linux users

```bash
bzcat <image_name>.wic.bz2 | sudo dd of=/dev/sd<partition> bs=1M conv=fsync
```

## Set up the Ethernet communication and NFS on the board

Use PuTTY, Tera Term, minicom or picocom to start a UART communication (baudrate 115200) through USB.

Setup U-boot to mount the NFS (it will pass thoses arguments to the Linux Kernel)

```bash
setenv bootargs 'console=ttyS0,115200 root=/dev/nfs rw
nfsroot=192.168.0.1:/nfs,nfsvers=3,tcp
ip=192.168.0.100:::::usb0 g_ether.dev_addr=f8:dc:7a:00:00:02
g_ether.host_addr=f8:dc:7a:00:00:01'
setenv bootcmd 'mmc dev 0; devnum=${mmcdev}; setenv devtype mmc;
mmc rescan; run loadimage; run findfdt; run mmcloados'
saveenv
```
