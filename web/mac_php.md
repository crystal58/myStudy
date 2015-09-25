#mac ox 10.10.2 自带PHP 不支持 imagettfbbox()

##报错：Call to undefined function imagettfbbox()

此函数 需要gd库和freetype的支持

gd已经支持，需要安装freetype: brew install freetype

如果mac 没有安装brew,安装brew ：ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

安装好freetype,

重新编辑 php

一切ok
