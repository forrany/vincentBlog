---
title: Juypter Notebook 前端二次开发
date: 2019-12-09 20:15:24
author:  "Vinecnt Ko"
tags:
    - jupyter notebook
    - 前端
    - 二次开发
---
# Jupiter Notebook 前端二次开发与记录

## 一、 环境准备

1. 使用Anaconda进行版本控制、包管理，其管理工具`conda` 与`npm`一样方便的存在

```bash
brew cask install Anaconda
```

2. 安装Node.js与npm
3. 创建虚拟环境

```bash
conda create -n envName # envName 是环境名
```

这里顺便提供环境管理的几个常用命令

- 激活环境

```bash
activate envName
```

- 推出环境

```bash
deactivate envName
```

- 删除环境

```bash
conda remove -n flowers --all
```

- 查看所有环境

```bash
conda info --envs
```



## 二、 Jupyter Notebook使用

使用Anaconda安装python后，就已经集成Jupyter nodebook了，如果notebook与conda的环境和包没有关联，可以执行以下命令进行关联

### 1. 安装

```
conda install nb_conda
```

### 2.使用

- 可以在Conda类目下对conda环境和包进行一系列操作。
- 可以在笔记本内的“Kernel”类目里的“Change
  kernel”切换内核。

详细参考，见:

[1. Jupyter Notebook介绍、安装及使用教程](https://zhuanlan.zhihu.com/p/33105153)

[2. 给初学者的 Jupyter Notebook 教程](https://juejin.im/post/5af8d3776fb9a07ab7744dd0#heading-10)



## 三、二次开发

Jupyter Notebook的[项目地址](https://github.com/jupyter/notebook)

在创建的虚拟环境中，运行一下操作

```bash
git clone https://github.com/jupyter/notebook # clone project
cd notebook
pip install -e . # install packages under the workspace
```

Notebook后端使用tornado框架，分为多个模块。对应的于templates不同的页面。

### 配置文件

执行`jupyter notebook --generate-config`  

在用户目录生成配置文件，mac中，进入finder， 选择`~/.jupyter/jupyter_notebook_config.py` 进行配置
常用配置如下：

```bash
# 解决跨域问题
c.NotebookApp.tornado_settings = {
      'headers': {
            'Content-Security-Policy': "frame-ancestors self *; report-uri /api/security/csp-report",
      }
}
# 可访问的IP地址
c.NotebookApp.ip = '*'
# 端口
c.NotebookApp.port = 9123
# 启动服务端时是否打开浏览器
c.NotebookApp.open_browser = False
# 去掉密码验证
c.NotebookApp.token = ""
# 是否开启新建终端
c.NotebookApp.terminals_enabled = False
# 是否可以通过前端修改密码
c.NotebookApp.allow_password_change = False
# 前端是否展示退出按钮
c.NotebookApp.quit_button = False
# 默认打开的目录路径
c.NotebookApp.notebook_dir = "workspace"
```

### 前端二次开发

项目采用Tornado的自带系统模版，该模板本身是HTML文件，包含有python的反控制结构和表达式。所以，对于前端页面的二次开发，主要包含两个部分： 

- 模板文件及CSS样式的修改
- 控制模块修改（python和js）



其中，jupyter的核心，cell及在线编辑的功能是基于`Code Mirror`, 该项目是网页实现的代码编辑器，支持语法高亮、自动缩紧等，因此，对于一些cell相关的二次开发，需要对`Code Mirror` 熟悉。

`Code Mirror` 的简介如下： 

[1. 在线代码编辑器CodeMirror](https://zhuanlan.zhihu.com/p/22163474)

[2.官方文档](https://codemirror.net/doc/manual.html#usage)

#### 目录结构

项目按照模块划分，包括`notebook`、`edit`、`nbconvert`等，前端比较关注的是

- `notebook->static->templates` 各页面的模板文件
- `notebook->static->static` 各页面的样式文件CSS和渲染控制文件JS

项目的其他文件和目录暂不做分析，先从简单的修改开始

#### 本地启动

经过以上配置，进入项目文件夹，执行以下代码: 

```bash
jupyter notebook
```

这里需要注意的是，因为这里关注前端的二次开发，因此可以运行`npm run build:watch`用来监听js的修改和构建。

#### toolbar的修改

工具栏在notebook的主页面中，打开`templates->notebook.html` 查看其模板结构，可以看到其工具栏的模板代码非常简单

```html
<div id="maintoolbar" class="navbar">
  <div class="toolbar-inner navbar-inner navbar-nobg">
    <div id="maintoolbar-container" class="container"></div>
  </div>
</div>
```

即完全是静态页面，也就说，工具栏的按钮和标签是通过脚本动态添加的。

同样在`static`目录下，可以看到名为`maintoolbar.js`，这里定了`maintoolbar`对象，在该对象的原型上绑定了一些方法，比如： 

```js
MainToolBar.prototype._pseudo_actions.add_celltype_list = function () {
        var that = this;
        var multiselect = $('<option/>').attr('value','multiselect').attr('disabled','').text('-');
        var sel = $('<select/>')
            .attr('id','cell_type')
            .attr('aria-label', i18n.msg._('combobox, select cell type'))
            .attr('role', 'combobox')
            .addClass('form-control select-xs')
            .append($('<option/>').attr('value','code').text(i18n.msg._('Code')))
            .append($('<option/>').attr('value','markdown').text(i18n.msg._('Markdown')))
            .append($('<option/>').attr('value','raw').text(i18n.msg._('Raw NBConvert')))
            .append($('<option/>').attr('value','heading').text(i18n.msg._('Heading')))
            .append(multiselect);
        this.notebook.keyboard_manager.register_events(sel);
      
      ...
```

这里即动态添加工具栏内容的代码，也就可以从这里入手，根据自己的实际需求，修改相应的前端展示内容。

##### 下拉选项修改

![image-20191209171941402](/images/jupyter-toolbar.png)



比如一个简单需求：修改工具栏下拉的内容，并能通过与父级通讯，实现在下拉切换时，调用外部的方法。

- 修改下拉的内容

直接修改动态加载页面的js文件，去除不需要的下拉内容，这里不过多说明

- 与父组件进行

notebook在项目中会作为iframe嵌在页面中，可考虑iframe父子通讯的方法。在本系统中，因为页面存在跨域问题，因此无法直接使用`window.parent.fn();` 或者 `window.top.fn()` 。 

这里使用`window.postMessage`方法，实现跨域的通讯，在下拉的change事件中，添加如下代码

```js
/**
* 与父级通讯，调用外部方法
*/
window.top.postMessage({
  selected: data.cell_type,
  eventType: 'languageChanged'
}, '*')
```

这样，在父级页面，只需要添加监听，即可实现通讯，具体如下:

```js
window.addEventListener('message', event => {
  if (event.data.eventType === 'languageChanged') {
    console.log(event.data.selected) // 修改为具体的需要执行的函数即可
  }
})
```

对iframe跨域同学的两种方案: 

[1. postMessage方法]([https://gjincai.github.io/2017/05/22/%E5%AD%90%E9%A1%B5%E9%9D%A2iframe%E8%B7%A8%E5%9F%9F%E6%89%A7%E8%A1%8C%E7%88%B6%E9%A1%B5%E9%9D%A2%E5%AE%9A%E4%B9%89%E7%9A%84JS%E6%96%B9%E6%B3%95/](https://gjincai.github.io/2017/05/22/子页面iframe跨域执行父页面定义的JS方法/))

[2. iframe代理方法](https://www.jianshu.com/p/9d90d3333215)



其他的模块、内容类似，等之后深入研究后，继续补充前端二次开发的踩坑经历。



## 参考资料

[SQL Interface within JupyterLab](https://www.datacamp.com/community/tutorials/sql-interface-within-jupyterlab)

[Jupyter Notebook Tutorial: The Definitive Guide](https://www.datacamp.com/community/tutorials/tutorial-jupyter-notebook)

[jupyter notebook 中文文档](https://www.osgeo.cn/jupyter/extending/frontend_extensions.html)

[jupyter notebook document](https://jupyter-notebook.readthedocs.io/en/latest/extending/frontend_extensions.html#installing-and-enabling-extensions)

[tornado web 服务器框架](http://www.ttlsa.com/docs/tornado/)

[jupyter notebook 二次开发经验](https://blog.csdn.net/weixin_39198406/article/details/90812334)



