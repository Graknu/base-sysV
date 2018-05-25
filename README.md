## Yaolinux-cross-Systemd, please following these commands to install the base

## on a normal Nutyx in root
```bash
cards install cards.devel git 
```
wget http://rems.techozor.fr/sources/install-nutyx{,.md5sum} \
md5sum -c install-nutyx.md5sum

## if the commands says "install-nutyx: OK", you can continue
chmod -v 755 install-nutyx \
mv -v install-nutyx /usr/bin/install-nutyx

## If you've already make the installation process, you have to remove the LFS user from the nutyx base by
userdel clfs \
rm -r /home/clfs \
rm -r /mnt/clfs \
groupdel clfs

## After that or if you didn't make an installation process, you have to run these following commands
export CLFS=/mnt/clfs \
mkdir -pv $CLFS

## Please note that you have to choose your partition in the next command, here /dev/sda2
mount /dev/sda2 $CLFS

## It's time to begin pass1
## make sure the LFS variable is set by
echo $CLFS 

## If nothing is in the output, make
export CLFS=/mnt/clfs 

## Now, create the directories
mkdir -vp $CLFS/{sources,tools,cross-tools} \
ln -svf $CLFS/tools / \
ln -svf $CLFS/cross-tools / \
ln -svf $CLFS/sources / 

## Create the CLFS user
groupadd clfs \
useradd -s /bin/bash -g clfs -d /home/clfs clfs \
mkdir -pv /home/clfs \
chown -v clfs:clfs /home/clfs \
chown -v clfs ${CLFS}/tools \
chown -v clfs ${CLFS}/cross-tools \
chown -v clfs ${CLFS}/sources \
chown -v clfs $CLFS

## Now go in the LFS user
su - clfs

## Create bashrc and bash-profile for clfs user
cat > /home/clfs/.bash_profile << "EOF" \
exec env -i HOME=${HOME} TERM=${TERM} PS1='\u:\w\$ ' /bin/bash \
EOF 

set +h \
umask 022 \
CLFS=/mnt/clfs \
LC_ALL=POSIX \
PATH=/home/clfs/bin:/cross-tools/bin:/bin:/usr/bin \
export CLFS LC_ALL PATH \
unset CFLAGS CXXFLAGS PKG_CONFIG_PATH > /home/clfs/.bashrc

## You are in the LFS user, now continue the installation with
git clone https://github.com/Graknu/cross-base_sysD.git development \
cd development \
scripts/runmebeforepass1 

## Normally, all will be good with the message above
"====> Successfull configured"

## initializing variable for cross-tools
export CLFS_HOST=$(echo ${MACHTYPE} | sed -e 's/-[^-]*/-cross/') \
export CLFS_TARGET="x86_64-unknown-linux-gnu" \
export CLFS_TARGET32="i686-pc-linux-gnu" \
export BUILD32="-m32" \
export BUILD64="-m64" 

cat >> ~/.bashrc << EOF \
export CLFS_HOST="${CLFS_HOST}" \
export CLFS_TARGET="${CLFS_TARGET}" \
export CLFS_TARGET32="${CLFS_TARGET32}" \
export BUILD32="${BUILD32}" \
export BUILD64="${BUILD64}" \
EOF

## Go to compile cross-tools
cd cross-tools
pass

## initializing variable for chroot
export CC="${CLFS_TARGET}-gcc ${BUILD64}" \
export CXX="${CLFS_TARGET}-g++ ${BUILD64}" \
export AR="${CLFS_TARGET}-ar" \
export AS="${CLFS_TARGET}-as" \
export RANLIB="${CLFS_TARGET}-ranlib" \
export LD="${CLFS_TARGET}-ld" \
export STRIP="${CLFS_TARGET}-strip" 

echo export CC=\""${CC}\"" >> ~/.bashrc \
echo export CXX=\""${CXX}\"" >> ~/.bashrc \
echo export AR=\""${AR}\"" >> ~/.bashrc \
echo export AS=\""${AS}\"" >> ~/.bashrc \
echo export RANLIB=\""${RANLIB}\"" >> ~/.bashrc \
echo export LD=\""${LD}\"" >> ~/.bashrc \
echo export STRIP=\""${STRIP}\"" >> ~/.bashrc

## Do the first pass
cd chroot \
pass

## All will be ok with the message after a long time, which depends of your machine
"=======> Building '/home/clfs/development/chroot/cards/Pkgfile' succeeded. \
/home/clfs/development/chroot"

## Go to pass2 :

## The rest of installation will be done in root
exit

## check the LFS variable
echo $CLFS

## will return 
"/mnt/clfs"

## if the result is correct continue with
chown -R root:root $CLFS \
install -dv -m0750  $CLFS/root \
ln -sv development/scripts $CLFS/root/bin \
mv /home/clfs/development $CLFS/root/ \
cd $CLFS/root/development/base/nutyx

## make the first package
/tools/bin/pkgmk -cf ../../../bin/pkgmk.conf.passes

## install it
/tools/bin/pkgadd -r $LFS nutyx1* \
/tools/bin/pkgadd -r $LFS nutyx.man1*

## check if it's present
/tools/bin/pkginfo -r $LFS -i

## It's have to return 
"(base) nutyx 8.2-1... \
(base) nutyx.man 8.2-1..."

## make the configuration
VERSION="development" install-nutyx -ic

## We mount the folders 
mkdir -pv ${CLFS}/{dev,proc,run,sys} \
mknod -m 600 ${CLFS}/dev/console c 5 1 \
mknod -m 666 ${CLFS}/dev/null c 1 3 \
mount -v -o bind /dev ${CLFS}/dev \
mount -vt devpts -o gid=5,mode=620 devpts ${CLFS}/dev/pts \
mount -vt proc proc ${CLFS}/proc \
mount -vt tmpfs tmpfs ${CLFS}/run \
mount -vt sysfs sysfs ${CLFS}/sys \
[ -h ${CLFS}/dev/shm ] && mkdir -pv ${CLFS}/$(readlink ${CLFS}/dev/shm) \
cp -v /etc/resolv.conf $LFS/etc

## We check the correct mount
mount|grep $CLFS

## will normally return if it's on /dev/sda2
"/dev/sda2 on /mnt/lfs type ext4 (rw) \
/devtmpfs on /mnt/lfs/dev type devtmpfs (rw,nosuid,relatime,size=16300988k,nr_inodes=4075247,mode=755) \
devpts on /mnt/lfs/dev/pts type devpts (rw,relatime,gid=5,mode=620,ptmxmode=000) \
proc on /mnt/lfs/proc type proc (rw,relatime) \
sysfs on /mnt/lfs/sys type sysfs (rw,relatime) \
tmpfs on /mnt/lfs/run type tmpfs (rw,relatime)"

## continue in chroot
chroot "${CLFS}" /tools/bin/env -i \
    HOME=/root TERM="${TERM}" PS1='\u:\w\$ ' \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin \
    /tools/bin/bash --login +h

## Some "command not found" will appears, but not important here

## continue with
export PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin:/root/bin \
cd /root/development/base \
pass

## After a moment the scripts says you have to install bash manually, go with the commands
exit \
cd $LFS/root/development/base/bash \
for PACK in *.xz; do /tools/bin/pkgadd -r $LFS $PACK;done \
/tools/bin/pkginfo -r $LFS -i|grep bash

## The last command will return that if succeeds
"(base) bash 4.4-1 \
(base) bash.da 4.4-1 \
(base) bash.de 4.4-1 \
(base) bash.devel 4.4-1 \
(base) bash.doc 4.4-1 \
..."

## return in chroot
chroot "$LFS" /usr/bin/env -i HOME=/root TERM="$TERM" PS1='\u:\w\$ ' \ \
/bin/bash --login +h \
export PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin:/root/bin \
cd /root/development/base \
pass

## The last pass will return at end
"ADD: ca-certificates-20150725, 1282 files: 100% \
=======> Installing 'ca-certificates1418739487x86_64.cards.tar' succeeded. \
=======> compress ca-certificates1418739487x86_64.cards.tar" 

## Now, follow few commands to configure your nutyx-systemd
exit \
echo $LFS

## The ouptput have to be "/mnt/lfs "

## Go back in chroot
chroot $LFS /usr/bin/env -i HOME=/root TERM="$TERM" PS1='\u: \w\$' \ \
/bin/bash --login

## No "command not found" have to appears here

cat > /etc/cards.conf << "EOF" \
dir /usr/ports/base \
locale fr \
base /usr/ports/base \
EOF \
ports -u

## To boot, you have to compile the kernel with 
cd /usr/ports/base-systemd/base/kernel-lts \
pkgmk -d -i

## make a grub, if you don't have a working linux on an other partition or harddrive, with 
cards depcreate grub \
grub-install /dev/sda \
cat > /boot/grub/grub.cfg << "EOF" \
#Begin /boot/grub/grub.cfg \
set default=0 \
set timeout=5 \
insmod ext2 \
set root=(hd0,2) \
menuentry "GNU/Linux, Linux 4.16.8-lfs-20180511-systemd" { \
        linux   /boot/vmlinuz-4.16.8-lfs-20180511-systemd root=/dev/sda2 ro \
} \
EOF

## make a link for /etc/resolv.conf
ln -sfv /run/systemd/resolve/resolv.conf /etc/resolv.conf

## make a /etc/hostname
echo "nutyx-systemd" > /etc/hostname

## make a /etc/hosts like that if you have a network with DHCP
cat > /etc/hosts << "EOF" \
#Begin /etc/hosts \
\
127.0.0.1 localhost \
127.0.1.1 nutyx-systemd.home nutyx-systemd \
::1       localhost ip6-localhost ip6-loopback \
ff02::1   ip6-allnodes \
ff02::2   ip6-allrouters \
\
#End /etc/hosts \
EOF
  
## make a working network with dhcpcd
cards depcreate dhcpcd

## after compiling dhcpcd, you have to enable a systemd service. you have to know your network interface with
ip a \
systemctl enable dhcpcd@"your network interface"

## make a text editor
cards depcreate vim \

## make a fstab like that if you have your partition in /dev/sda2
cat > /etc/fstab << "EOF" \
#Begin /etc/fstab \
\
#file system  mount-point  type     options             dump  fsck order \
\
/dev/sda2     /            ext4    defaults            1     1 \
#/dev/<yyy>     swap         swap     pri=1               0     0 \
\
#End /etc/fstab \
EOF 

## make a file for time
cat > /etc/adjtime << "EOF" \
0.0 0 0.0 \
0 \
LOCAL \
EOF

## make a /etc/vconsole.conf like that il you're french
cat > /etc/vconsole.conf << "EOF" \
KEYMAP=fr-latin9 \
FONT=Lat2-Terminus16 \
EOF 

## make a /etc/locale.conf for example in french
cat > /etc/locale.conf << "EOF" \
LANG=fr_FR.utf8 \
EOF

## make a /etc/inputrc

cat > /etc/inputrc << "EOF" \
#Début de /etc/inputrc \
#Modifié par Chris Lynn <roryo@roryo.dynup.net> \
\
#Permettre à l'invite de commande d'aller à la ligne \
set horizontal-scroll-mode Off \
\
#Activer l'entrée sur 8 bits \
set meta-flag On \
set input-meta On \
\
#Ne pas supprimer le 8ème bit \
set convert-meta Off \
\
#Conserver le 8ème bit à l'affichage \
set output-meta On \
\
#none, visible ou audible \
set bell-style none \
\
#Toutes les indications qui suivent font correspondre la séquence \
#d'échappement contenue dans le 1er argument à la fonction \
#spécifique de readline \
"\eOd": backward-word \
"\eOc": forward-word \
\
#Pour la console linux \
"\e[1~": beginning-of-line \
"\e[4~": end-of-line \
"\e[5~": beginning-of-history \
"\e[6~": end-of-history \
"\e[3~": delete-char \
"\e[2~": quoted-insert \
\
#pour xterm \
"\eOH": beginning-of-line \
"\eOF": end-of-line \
\
#pour Konsole \
"\e[H": beginning-of-line \
"\e[F": end-of-line \
\
#Fin de /etc/inputrc \
EOF \
]
## make a /etc/shells
cat > /etc/shells << "EOF" \
#Begin /etc/shells \
\
/bin/sh \
/bin/bash \
\
#End /etc/shells \
EOF

## make a /etc/os-release
cat > /etc/os-release << "EOF" \
NAME="Linux From Scratch" \
VERSION="nutyx-systemd" \
ID=lfs \
PRETTY_NAME="nutyx-systemd" \
VERSION_CODENAME="<your name here>" \
EOF 
