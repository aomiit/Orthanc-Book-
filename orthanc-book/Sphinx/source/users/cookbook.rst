.. highlight:: bash
.. _cookbook:

快速入门
========

.. contents::
   :depth: 2


.. _binaries:

获得预编译程序
--------------

要获得预编译程序，有几个选择:

* :ref:`自己动手编译Orthanc <compiling>`。
* `下载预编译包 <http://www.orthanc-server.com/download.php>`__。
* :ref:`使用Docker <docker>`.
* 外部贡献者也在维护 `Vagrant VM for Orthanc
  <https://github.com/jodogne/OrthancContributed/blob/master/Links.md#vagrant>`__。


.. _orthanc-explorer:

打开Orthanc Explorer
--------------------

使用Orthanc最直接的方式是通过Web浏览器打开Orthanc的嵌入式管理界面**Orthanc Explorer**。
一旦Orthanc开始运行, 打开下面的地址:http://localhost:8042/app/explorer.html。
请注意：

* 端口号 8042是依据你的设置 :ref:`configuration <configuration>`。
* Orthanc Explorer不能很好的在Microsoft Internet Explorer下工作。请使用Mozilla Firefox,Google Chrome，Apple Safari，或者 `任何基于WebKit的Web浏览器 <https://en.wikipedia.org/wiki/WebKit>`__ 上传DICOM文件

-------------

Orthanc Explorer界面包含一个用户友好的页面来上传DICOM文件。
你可以访问如下链接到达该页面。
http://localhost:8042/app/explorer.html#upload.
这样，你就可以通过拖放方式选择DICOM文件并通过点击Upload按钮上传。

你可以 `观看视频教程
<https://www.youtube.com/watch?v=4dOcXGMlcFo&hd=1>`__ 演示如何通过在Chromium浏览器中利用
Orthanc Explorer上传文件到Orthanc。

**重要提示:** 这里有一些 `已知问题
<https://bitbucket.org/sjodogne/orthanc/issues/21/dicom-files-missing-after-uploading-with>`__
这使得Mozilla Firefox无法正确地上传所有DICOM文件。


通过DICOM协议上传
------------------

一旦Orthanc启动并运行，任何成像模式都可以通过DICOM协议（实现了C-Store命令）发送实例到Orthanc。

您可以使用标准命令行工具 ``storescu`` 来自
`DCMTK 软件 <http://dicom.offis.de/dcmtk.php.en>`__ 来手动发送DICOM图像到Orthanc, 例如::

    $ storescu -aec ORTHANC localhost 4242 *.dcm

这会发送所有".dcm"扩展名的文件到``localhost``运行的Orthanc实例，它的AET（application entity
title）是``ORTHANC``, DICOM 端口是``4242``。 显然，所有这些参数都依赖于您的
:ref:`configuration <configuration>`。请检查 :ref:`FAQ <dicom>` 如果你遇到任何问题。


接下来的步骤
------------

1. 阅读一般性的介绍 ":ref:`dicom-guide`".
2. 看看你的 :ref:`configuration file <configuration>`.
3. 驱动Orthanc通过其 :ref:`REST API <rest>`.
4. 自动化DICOM的任务 :ref:`Lua scripts <lua>`.

