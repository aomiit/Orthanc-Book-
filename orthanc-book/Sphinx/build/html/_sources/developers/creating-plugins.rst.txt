.. _creating-plugins:

创建新插件
====================

:ref:`为Orthanc贡献代码 <contributing>` 的推荐方法包括通过创建新的 :ref:`插件
<plugins>` 来扩展它。

概述
--------

Orthanc插件必须使用 `plugin SDK <http://sdk.orthanc-server.com/>`__　并且必须使用 C 或 C++ 编写。
还必须履行Orthanc核心遵从的 `GPLv3 license <http://www.gnu.org/licenses/quick-guide-gplv3.en.html>`__ 条款。
插件的例子可以在 `官方Orthanc仓库 <https://bitbucket.org/sjodogne/orthanc/src/default/Plugins/Samples/>`__ 找到
（＂Plugins/Samples＂目录下）。演示如何实现基本WADO服务器的教程在 `CodeProject <http://www.codeproject.com/Articles/797118/Implementing-a-WADO-Server-using-Orthanc>`__　可获得。

我们建议开发人员采用 :ref:`Orthanc核心编码风格 <coding-style>`, 虽然这当然不是必需的。

如果你希望你的插件被 **indexed** 在 :ref:`Orthanc之书的专有部分 <plugins-contributed>`　
请毫不犹豫的 `联系我们 <http://www.orthanc-server.com/static.php?page=contact>`__ !


插件的结构
------------------------

插件采用共享库的形式 (在Windows下是``.DLL``,
在GNU/Linux是``.so``, 在Apple OS X是``.dylib`` ...)，这是使用 `C语言的ABI
<https://en.wikipedia.org/wiki/Application_binary_interface>`__ 去声明4个
public 函数/标志:

* ``int32_t OrthancPluginInitialize(OrthancPluginContext* context)``.
这是一个回调函数负责初始化插件。＂context＂参数提供访问Orthanc的API的功能。

* ``void OrthancPluginFinalize()``. 这个函数负责为完成插件，释放所有分配的资源。

* ``const char* OrthancPluginGetName()``. 这个函数必须给出插件名称。

* ``const char* OrthancPluginGetVersion()``. 这个函数必须提供插件的版本。

*Remark:* 在之间交换的内存缓冲区的大小Orthanc核心和插件必须低于4GB。 
这是Orthanc插件SDK使用``uint32_t``对内存缓冲区的大小进行编码的结果。
我们可能会在将来扩展SDK来处理大小超过4GB的缓冲区。
