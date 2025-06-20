```bash
$ file cpu2017-1.1.9.ios
cpu2017-1.1.9.iso: Zip archive data, at least v2.0 to extract, compression method=store
$ unzip cpu2017-1.1.9.iso
$ cd cpu2017-1.1.9
$ chmod +x install.sh
$ chmod -R +x bin/*
$ chmod +x tools/bin/linux-x86_64/spectar
$ chmod +x tools/bin/linux-x86_64/specxz
$ chmod +x tools/bin/linux-x86_64/specsha512sum
$ ./install.sh
SPEC CPU2017 Installation

[...snip...]

Installation successful.  Source the shrc or cshrcc in
/path/to/cpu2017-1.1.9
to set up your environment for the benchmark.
```
