---
title: Windows下生成sphinx开源文档
abbrlink: 46f35077
date: 2019-03-17 14:28:18
categories:
  - Document
  - Sphinx
tags: [Document, Sphinx]
---

# 介绍
在Windows在生成Sphinx文档,便于团队交流,项目介绍


# 环境安装
## Python环境
略过,百度一大堆.官网安装,下载即可.


## sphinx环境
``` python
pip install sphinx sphinx-autobuild
```

## 启动,创建
使用windows自带的命令行窗口,执行命令
新建示例项目
`sphinx-quickstart`
根据命令提示,创建项目,介绍,作者等信息.

在创建后的项目目录下,有`make.bat`文件,通过命令
``` 
make clean
make html
```
清除并创建html
创建后的html在`..\_build\html`目录下.

## 目录结构
在项目目录下,依次存有以下文件

- `_build`  渲染后的html文件存放处
- `_static` 静态不做处理的文件
- `_templates` 模板文件
- `conf.py` 配置文件
- `index.rst` 主页,及主页跳转
- `make.bat` Windows环境下,启动,渲染脚本
- `Makefile` sphinx文件

## 配置文件信息
配置信息在文档项目,`conf.py`文件中.

### 支持 markdown语法
下载解析器
`pip install recommonmark sphinx-markdown-tables`

在`conf.py`配置文件中,新增.
``` python
source_parsers = {
    '.md': 'recommonmark.parser.CommonMarkParser',
}

source_suffix = ['.rst', '.md']

extensions = [
    'sphinx.ext.autodoc',
	'sphinx_markdown_tables'
]
```

### 修改主题风格
下载主题风格
`pip install sphinx_rtd_theme`

修改配置文件`conf.py`
`html_theme = 'sphinx_rtd_theme'`


# 编写

## 创建子文件
在项目目录下,创建`page`文件,新建文件`语法介绍.rst`文件.

## 子文件添加到主页面
修改主页面`index.rst`.添加以下.

```
.. toctree::
   :maxdepth: 2
   :caption: Contents:

```


