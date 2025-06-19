```bash
$ sudo apt install libgraphviz-dev
$ git clone https://github.com/moking/ndctl.git
$ cd ndctl
$ git checkout patch-gen
$ meson setup build
$ meson compile -C build
$ ./build/cxl/cxl graph
$ ls cxl-topology-graph.png
cxl-topology-graph.png
```
