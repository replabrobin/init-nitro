This is a preliminary attempt at a way to use nitro see

https://github.com/leahneukirchen/nitro

as an init system like runit.

This package contains nitro, nitroctl, nsm & shutdown binaries and also
/usr/lib/rc, /etc/rc system setup code.

Some sample services are installed in /etc/nitro/sv.

Currently this package can be installed alogside runit if the PKGBUILD
conflicts are commented.

I started by removing all runit related packages using

pacman -Rdd runit runit-rc

then makepkg of this package and installing with pacman -U.

To allow for a pre-existing init I then symbolically linked init using

ln -sf nitro /usr/bin/init

To force a shutdown 
	sync && sudo sh -c "echo o > /proc/sysrq-trigger"

or a restart
	sync && sudo sh -c "echo b > /proc/sysrq-trigger"

I use nsm to control the services. Try nsm -h to see what should/might work.
