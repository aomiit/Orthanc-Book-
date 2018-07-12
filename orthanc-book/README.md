基本信息
===================

Orthanc是轻量级的，DICOM厂商中立的存档的RESTful。 

关于Orthanc的基本信息可以浏览它的 [官方网站](http://www.orthanc-server.com/)。

本仓库包含 [Orthanc Book](http://book.orthanc-server.com/) 的源码, 介绍Orthanc如何使用。 


安装
-----

从源码构建 Orthanc Book, 你需要安装 [Sphinx](http://www.sphinx-doc.org/), 它是一个 Python 文档生成器。


### Ubuntu 14.04 LTS下安装Sphinx ###

    # sudo pip install sphinx sphinx_bootstrap_theme


生成文档
----------------------------

### Linux 下 ###

    # cd ./Sphinx
    # make html

HTML文档可以在`./build/html`文件夹中找到。您可以使用Mozilla Firefox打开它,例如:

    # firefox ./build/html/index.html


如何做出贡献
-----------------

 * 确认已经理解了 [reStructuredText文件格式](https://en.wikipedia.org/wiki/ReStructuredText)。
 * Fork仓库到你的 BitBucket 账户。
 * 编辑内容 [`./Sphinx/source/` 目录](./Sphinx/source/)。
 * 提交 [pull request](https://confluence.atlassian.com/bitbucket/create-a-pull-request-945541466.html) 供Orthanc项目审查。
 * 一旦审查通过，Orthanc项目的持续集成服务器将自动发布新版本 [online](http://book.orthanc-server.com/)。
