## 安装svn和git

## 配置git
```
$ git config --global user.name "yunshandi"
$ git config --global user.email yunshandi@tencent.com
```

## 源svn库地址

```
http://svn-cd1.tencent.com/qcloudgp/greeplum_proj
```

## 从该仓库导出用户映射信息

```
svn log --xml http://svn-cd1.tencent.com/qcloudgp/greeplum_proj | grep author | sed 's#<author>##' | sed 's#</author>##' | sort -u | sed 's/.*/& = & <&@tencent.com>/' > authors.txt
```

## 开始迁移
```
# -s，即--stdlayout，意思是按标准目录结构（也就是branches/tags/trunk），等同于参数：--trunk="trunk" --branches="branches" --tags="tags"（也可以简写为：-T trunk -b branches -t tags），如果你只迁移trunk，则需要把-s参数移除
# --prefix=svn/，该参数是指将svn的branches和tags下的内容迁移到.git/refs/remotes/svn/下，这样与将来工蜂remote区分开，但是如果你只想迁移一个分支或目录，--prefix参数就没太大必要了。
# --no-minimize-url，默认git svn会回溯到仓库的根目录（对于一些老仓库来说，就是xx_xxx_rep级别），由于当前用户可能没有rep级别的权限，就可能出现迁移异常，而加上该参数后，git svn则不会回溯仓库根目录。


git svn clone http://svn-cd1.tencent.com/qcloudgp/greeplum_proj -s --prefix=svn/ --no-metadata --no-minimize-url --authors-file=authors.txt beowulf

git svn clone http://svn-cd1.tencent.com/qcloudgp/greeplum_proj --no-metadata --no-minimize-url --authors-file=authors.txt greenplum-proj

git svn clone http://svn-cd1.tencent.com/qcloudgp/greeplum_proj -T trunk --no-metadata --no-minimize-url --authors-file=authors.txt snova
```

## svn:ignore迁移
```
# 进入刚才通过git svn迁移生成的本地git仓库
$ cd proj

# 将svn:ignore设置导入到.gitignore
$ git svn show-ignore --id=origin/trunk > .gitignore

# 提交.gitignore
$ git add .gitignore
$ git commit -m "convert svn ignore to git."
```

## git push 

```
git remote add origin http://git.code.oa.com/Hadoop/snova.git
git push -u origin --all
git push --tags
```
