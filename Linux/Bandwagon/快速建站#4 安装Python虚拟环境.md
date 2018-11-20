[TOC]

> 服务器系统为Ubuntu 18

## 虚拟环境的选择

Python的虚拟环境有很多工具选择，常用的大概是这三个：  

1.  virtualenv
2. virtualenvwrapper
3. pyenv&pyenv-virtualenv

这些工具在之前的文章中介绍得很详细，可以看看我的相关文章。 `virtualenv` 是用起来极其不方便，也不方便管理；`virtualenvwrapper` 对虚拟环境的使用和管理都非常方便，但是呢它仅仅作用于某个python版本这个层面；`pyenv` 非常强大，不但能够管理并隔离多个python版本，还能搭配 `pyenv-virtualenv` 插件，去管理并隔离各个python版本的虚拟环境，并且能自动激活/失效虚拟环境。  

因此选择使用pyenv。  

## pyenv安装

```bash
cd ~
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc
source ~/.zshrc
```

安装依赖：  

```bash
sudo apt-get update
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev
```

## pyenv-virtualenv安装

```bash
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
# 添加虚拟环境自动激活
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
source ~/.zshrc
```

至此，虚拟环境安装完毕，在 `.zshrc` 中添加的所有内容如下：  

```bash
# pyenv
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi
eval "$(pyenv virtualenv-init -)"
```

## 安装项目对应的Python版本

```bash
pyenv install 3.7.1
pyenv rehash
```

## 拉取Django项目

```bash
mkdir -p ~/work/deploy/webapps/
cd ~/work/deploy/webapps/
git clone git@github.com:CloudSen/RedQueen.git
```

## 创建特定python版本的虚拟环境

```bash
pyenv virtualenv 3.7.1 red_queen_blog_env
```

## 给Django项目指定创建的虚拟环境

```bash
cd ~/work/deploy/webapps/RedQueen
pyenv local red_queen_env 
```

## 安装项目依赖

```bash
cd ~/work/deploy/webapps/RedQueen
pip install -r requirements.txt
```

## 生成项目的静态资源

```bash
cd ~/work/deploy/webapps/RedQueen
python manage.py collectstatic
```

  

至此，项目拉取完毕，虚拟环境搭建完毕，项目所需要的第三方包也安装完毕，静态文件生成完毕。



