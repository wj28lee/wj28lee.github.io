```bash
$ git clone https://github.com/acpica/acpica.git
$ cd acpica
$ make -j
$ sudo make install
```
```bash
$ sudo acpidump -o acpidump.out
$ acpixtract -a acpidump.out
$ iasl -d cedt.dat
$ iasl -d srat.dat
```
