```bash
$ git cclone https://github.com/intel/pcm.git
$ cd pcm
$ mkdir bulid
$ cd build
$ cmake ..
$ cmake --build .
$ sudo ./pcm-memory 1 -csv=test.log -- <your program>
```
