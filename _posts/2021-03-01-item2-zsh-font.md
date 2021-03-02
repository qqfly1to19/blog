---
layout: post
title: mac中配置一个漂亮的编程环境— item2 + zsh + agnoster Theme
date: 2021-03-02
Author: 乌卡 
tags: [配置, item2, zsh, IDE]
comments: true
---


预先善其事，必先利其器。舒服的编程环境和便捷的使用工具是变成的开始。

**程序员最大的优点那就是——懒惰**

## 下载item2

用mac自带的shell和item2比起来还是差距很大。主要是item2各方面的生态支持的比较好，想配置个啥可以随时配置。下载item2也比较简单。[item2下载](https://iterm2.com/)

## ZSH

ZSH是一个增强版的shell。配置完成后，用起来感觉不是在玩一个枯燥的命令行。大家感受一下这个画面。

![image-20210302120303847](https://tva1.sinaimg.cn/large/e6c9d24ely1go5fgbyffcj20gc07htbd.jpg)

mac的配置比linux稍微复杂一点。方法上就是用[github-ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)上的就可以。如果远程的来不了（大概率被墙了），可以把整个项目git下来

```shell
cd $PROJECTS/tools/ && sh install.sh
```

这个脚本会自动创建一个~/.zshrc的文件，用于配置zsh的。

安装完之后重启item2，这时候可能会弹出一大堆文本，意思就是让你修改两个文件的权限。跟着修改就好了.

```shell
chmod 777 $PROFILE
```



我用的这个主题叫“agnoster”。 要用这个主题，必须下载相关的字体。

```
$ echo "\ue0b0 \u00b1 \ue0a0 \u27a6 \u2718 \u26a1 \u2699"
```

如果输出有问好，那说明你的字体是不完整的。根据github上面的提示去下载[powerline字体](https://github.com/powerline/fonts)。

我这里也是直接git下来字体。

```
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
```

下载完之后，也不能代表一定会好用。这时候，需要在item2内配置一下字体才可以使用。

Item2 -> preference -> Profiles -> Text

![image-20210302122127487](https://tva1.sinaimg.cn/large/e6c9d24ely1go5fzewfuzj20pd0fbmzo.jpg)

选择之后，才会真正的使用zsh的相关主题。

这时候修改~/.zshrc文件,添加如下代码 ZSH_THEME="agnoster" 。注释掉其他的theme

![image-20210302122232045](https://tva1.sinaimg.cn/large/e6c9d24ely1go5g0jdmctj20di03dq3b.jpg)
