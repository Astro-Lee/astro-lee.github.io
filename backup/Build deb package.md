Build deb package.
本文介绍deb的构建方式

```bash
DEBMAIL="xxxx@gamil.com"
DEBFULLNAME="XXX"
DEBSIGN_KEYID="xyrtyfhfghgfhfg"
export DEBMAIL DEBFULLNAME DEBSIGN_KEYID
```


```bash
gpg --full-generate-key --expert
gpg --armor --export id
dpkg-source --commit
debuild -S -sa -k$DEBSIGN_KEYID | tee /tmp/debuild.log 2>&1
dupload XXX.changes
dput ppa:ruizhi-li/ppa XXX.changes
```

# 参考
- [Debian 新维护者手册](https://www.debian.org/doc/manuals/maint-guide/index.zh-cn.html)
- [Astropy Packaging Tutorial](https://wiki.debian.org/DebianAstro/AstropyPackagingTutorial/Preparation)
- [如何发布自己的"个人软件包存档](https://briteming.blogspot.com/2018/06/blog-post_58.html)
- [发布你的开源软件到 Ubuntu PPA](https://hedzr.com/packaging/deb/publishing-and-hosting-your-own-dev-at-ppa/)