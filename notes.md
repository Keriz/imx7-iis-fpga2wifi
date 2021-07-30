## Installing crosstool-ng (to build up our own cross-compiling toolchain)

Needed packages 

sudo apt install build-essential git autoconf bison flex texinfo help2man gawk \
libtool-bin libncurses5-dev unzip

Configure git 

git config user.name ""
git config user.email ""

git clone https://github.com/crosstool-ng/crosstool-ng.git
cd crosstool-ng/
git checkout 79fcfa17

./bootstrap

./configure --enable-local
make

To get help

./ct-ng help 
./ct-ng menuconfig

once config is done, run
./ct-ng build

export toolchain (default build output path is $HOME/x-tools/) in your parth

export PATH=$HOME/x-tools/arm-imx7ulp-linux-uclibcgnueabihf/bin/:$PATH