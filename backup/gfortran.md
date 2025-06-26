```
env -i HOME=$HOME PATH=/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin \
TERM=$TERM SHELL=/bin/bash \
DYLD_LIBRARY_PATH=/opt/homebrew/lib \
bash --noprofile --norc
```

```
which gfortran
# → 应该是 /opt/homebrew/bin/gfortran

gfortran --version
# → 应该是 Homebrew gcc 版本

otool -L /opt/homebrew/bin/gfortran | grep fortran
# → 应该只看到 /opt/homebrew/lib，不能有 mambaforge
```

```
cd ~/Downloads/MultiNest
rm -rf build
mkdir build && cd build

cmake .. -DCMAKE_Fortran_COMPILER=/opt/homebrew/bin/gfortran
make -j4
```