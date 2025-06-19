```bash
$ git clone https://github.com/radii/devmem2.git
$ cd devmem2
$ gcc -o devmem2 devmem2.c
$ ./devmem2

Usage:  ./devmem2 { address } [ type [ data ] ]
        address : memory address to act upon
        type    : access operation type : [b]yte, [h]alfword, [w]ord
        data    : data to be written
```
