记录进度：选择接口应用程序，构建并安装 exif 命令行实用程序，该步骤报错<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/eaf9c90a-a627-41cb-8004-a0f8e7764376)

```
cd exif-exif-0_6_15-release/
autoreconf -fvi
./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/" PKG_CONFIG_PATH=$HOME/fuzzing_libexif/install/lib/pkgconfig
make
make install
```

检查pkgconfig的安装
