# Makefile for RPI image.

# set image file name
FILE=`date +%Y-%m-%d`-netbsd-evbarm.img

# iso file information
IMAGE=/usr/release/images/NetBSD-6.99.28-evbarm-arm-201312050420Z.iso
PKGSRC=pkgsrc-2013-12-28.tar.gz

# mount cdrom
MNT_CDROM=/mnt1
VND_CDROM=vnd1

SETS=${MNT_CDROM}/evbarm/binary/sets
RPI=${MNT_CDROM}/evbarm/binary/gzimg/rpi.img

# extract sets.
#LIST=man misc modules tests text xbase xcomp xetc xfont xserver
LIST=man modules text xbase xcomp xetc xfont xserver

# mount point
MNT=/mnt
VND=vnd0

# skelton
RPIDIR=../../RPI/RPIimage
KOBO=..



all: file cdmount rpi extract cdunmount

# make image file. 512*3813376=2GB
file: 
	echo Create ${FILE}.
	dd if=/dev/zero of=${FILE} bs=512 count=3813376 progress=50000
	sleep 3

# copy rpi.img.gz to image file and resize partition.
rpi:
	echo Copy rpi.img.gz
	vnconfig ${VND} ${FILE}
	gunzip < ${RPI}.gz |dd of=/dev/r${VND}d bs=1m progress=20
	disklabel -R ${VND} DISKLABEL.proto
	fsck -fy /dev/r${VND}a
	resize_ffs -y /dev/r${VND}a
	fsck -fy /dev/r${VND}a
	vnconfig -u ${VND}

# extract sets to image file.
extract:
	echo extract sets.
	vnconfig ${VND} ${FILE}
	fsck -fy /dev/r${VND}a
	mount /dev/${VND}a ${MNT}
	for i in ${LIST} ; do (tar xzpf ${SETS}/$${i}.tgz -C ${MNT};); done
	tar xzpf ${PKGSRC} -C ${MNT}/usr
	umount ${MNT}
	vnconfig -u ${VND}


# stop sshd for package install
pkg:
	vnconfig ${VND} ${FILE}
	fsck -fy /dev/r${VND}a
	mount /dev/${VND}a ${MNT}
	echo rc_configured=YES >> ${MNT}/etc/rc.conf
	echo sshd=NO           >> ${MNT}/etc/rc.conf
	echo syslogd=NO           >> ${MNT}/etc/rc.conf
	echo dhcpcd=NO           >> ${MNT}/etc/rc.conf
	umount ${MNT}
	vnconfig -u ${VND}

# copy default setupfile
release:
	vnconfig ${VND} ${FILE}
	fsck /dev/r${VND}a
	mount /dev/${VND}a ${MNT}
	mount /dev/${VND}e ${MNT}/boot
	(cd ${RPIDIR};tar cf - etc root) |(cd ${MNT};tar xf -)
	(cd ${RPIDIR};tar cf - Oshimaya) |(cd ${MNT}/root;tar xpf -)
	cp ${RPIDIR}/boot/*     ${MNT}/boot
	cp u-boot.bin ${MNT}/boot
	cp ${KOBO}/netbsd-ld0.bin   ${MNT}/boot/kobo-ld0.bin
	cp ${KOBO}/netbsd-sd0.bin   ${MNT}/boot/kobo-sd0.bin
	umount ${MNT}/boot
	umount ${MNT}
	vnconfig -u ${VND}

# mount cdrom image
cdmount:
	echo mount cdrom image.
	vnconfig ${VND_CDROM} ${IMAGE}
	mount_cd9660 /dev/${VND_CDROM}a ${MNT_CDROM}

# unmount cdrom image
cdunmount:
	echo mount cdrom image.
	umount ${MNT_CDROM}
	vnconfig -u ${VND_CDROM}

copy:
	dd if=${FILE} of=/dev/rsd0d bs=4m progress=20

firm:
	cp /usr/local/NetBSD/RPI/Firmware/firmware/boot/bootcode.bin /mnt
	cp /usr/local/NetBSD/RPI/Firmware/firmware/boot/fixup.dat /mnt
	cp /usr/local/NetBSD/RPI/Firmware/firmware/boot/fixup_cd.dat /mnt
	cp /usr/local/NetBSD/RPI/Firmware/firmware/boot/start.elf /mnt
	cp /usr/local/NetBSD/RPI/Firmware/firmware/boot/start_cd.elf /mnt
	cp /usr/local/NetBSD/RPI/Firmware/firmware/boot/LICENCE.broadcom /mnt

# fetch newest isofileimage
fetch:  
	./NewestISO |tee Fetch
	sh Fetch
	rm Fetch
