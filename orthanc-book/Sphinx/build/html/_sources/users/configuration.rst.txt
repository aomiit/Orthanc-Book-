.. _configuration:
.. highlight:: bash

Orthanc的配置
==============

配置Orthanc仅仅是复制和修改
`默认配置文件
<https://bitbucket.org/sjodogne/orthanc/raw/Orthanc-1.3.2/Resources/Configuration.json>`_。 这个
文件是 `JSON <https://en.wikipedia.org/wiki/JSON>`_ 格式。您可以通过如下的调用生成一个示例配置文件
::

    $ Orthanc --config=Configuration.json

然后，就可以开始运行Orthanc，通过传递一个修改后的Configuration.json路径作为命令行参数::

    $ Orthanc ./Configuration.json

默认的配置文件会:

* 创建一个DICOM服务器，DICOM AET (Application Entity Title)为
  ``ORTHANC`` ，并侦听4242端口.
* 为侦听端口8042的REST API创建一个HTTP服务器。
* 将Orthanc数据库存储在一个名为“OrthancStorage”的文件夹中。

*注意:* 在微软Windows下指定路径时，反斜杠(i.e. ``\``) 应该改为双反斜杠(像这样 ``\\``),
或者用正斜杠代替(像这样 ``/``)。

要获得更多的诊断信息，您可以使用``--verbose``或``--trace`` 选项::

    $ Orthanc ./Configuration.json --verbose
    $ Orthanc ./Configuration.json --trace

从Orthanc 0.9.1开始，您还可以使用指向一个目录的路径启动Orthanc。在这种情况下，Orthanc会加载该目
录下所有``.json``扩展名的文件，合并它们来构建配置文件。这允许将全局配置分解为几个文件。
