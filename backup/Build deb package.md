本文介绍 deb 的构建方式

# 前期准备
## 创建GPG公钥和私钥
```bash
gpg --full-generate-key --expert
# 记住私钥的密码
```
![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/a9ba85af-5f40-41c8-b814-a96de04927aa)

```bash
gpg --armor --export GPG密钥ID
```
复制以 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 开头并以 `-----END PGP PUBLIC KEY BLOCK-----` 结尾的 GPG 密钥。将内容提交到 [keyserver](https://keyserver.ubuntu.com/)

![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/b2a628d4-dc39-4d12-adc3-dfa642f7a822)

等待10分钟左右，进入 [launchpad](https://launchpad.net/) 的`OpenPGP keys`

![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/0069a056-8742-4723-9f45-a5c7451d2e49)

运行一下命令获得fingerprint
```bash
gpg --fingerprint
```
![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/ea238455-1d1a-4420-ae04-7a1dec8a19f9)

将fingerprint填入`OpenPGP keys`

![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/e475b928-78cd-43cd-8cc2-3ecf806b9c89)

将以下内容写入到一个临时文件`temp.txt`中，并执行`gpg -d temp.txt`

![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/a522c7f5-38ed-4a17-ae6c-1c70c3cdca86)
![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/bec8e4df-fe55-4b08-8b3c-d35b5b6d8e8e)
![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/7b9995de-9da0-4521-8af0-8ff7c476fe96)

## 设置环境变量
```bash
DEBMAIL="xxxx@gamil.com"
DEBFULLNAME="XXX"
DEBSIGN_KEYID="GPG密钥ID"
export DEBMAIL DEBFULLNAME DEBSIGN_KEYID
#DEBUILD_DPKG_BUILDPACKAGE_OPTS="-i -I -us -uc"
#DEBUILD_LINTIAN_OPTS="-i -I --show-overrides"
```
# 从项目源码构建
`psfex-3.24.2.tar.gz` 和 `psfex-3.24.2`

```bash
cd psfex-3.24.2
./autogen.sh && ./configure
dh_make -f ../psfex-3.24.2.tar.gz
```

修改debian目录中`control`, `changelog`和其他文件的内容

其中编辑`debian/changelog`
![image](https://github.com/Astro-Lee/astro-lee.github.io/assets/61745903/431d2007-3b7c-42ea-bc9f-83b47ca23b06)

```bash
dpkg-source --commit #修改过上游版本的源码，需要commit 
```
```
debuild -S -sa -k$DEBSIGN_KEYID | tee /tmp/debuild.log 2>&1 # 并输入签名的密码
```
```
dupload XXX.changes
dput -f ppa:ruizhi-li/XXXsoftware XXX.changes
```

# 参考
- [Debian 新维护者手册](https://www.debian.org/doc/manuals/maint-guide/index.zh-cn.html)
- [Astropy Packaging Tutorial](https://wiki.debian.org/DebianAstro/AstropyPackagingTutorial/Preparation)
- [如何发布自己的"个人软件包存档](https://briteming.blogspot.com/2018/06/blog-post_58.html)
- [发布你的开源软件到 Ubuntu PPA](https://hedzr.com/packaging/deb/publishing-and-hosting-your-own-dev-at-ppa/)