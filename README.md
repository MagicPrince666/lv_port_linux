# LVGL for frame buffer device

LVGL configured to work with /dev/fb0 on Linux.

When cloning this repository, also make sure to download submodules (`git submodule update --init --recursive`) otherwise you will be missing key components.

Check out this blog post for a step by step tutorial:
https://blog.lvgl.io/2018-01-03/linux_fb

## 同步子仓库
```bash
git submodule update --init --recursive
```

## 交叉编译
```bash
cmake -DCMAKE_TOOLCHAIN_FILE=cmake/build_for_rv1126.cmake ..
```
