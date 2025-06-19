```bash
$ git clone https://github.com/torvalds/linux.git
$ cd linux
$ git tag
$ git checkout v6.15
$ git log --oneline -1
$ make olddefconfig
$ make menuconfig
$ make -j`nproc`
$ sudo make INSTALL_MOD_STRIP=1 modules_install -j`nproc`
$ sudo make install
```
