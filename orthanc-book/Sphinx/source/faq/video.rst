Orthanc支持视频吗?
============================

这个问题的答案取决于你的怎么认识"支持" 和 "视频":

* 在DICOM的世界里, 视频"video"可以指2D+t 二维影像+时间 (又叫做. "cine" 电影式独立帧序列)
  或者真正的视频 (使用H.264 或 MPEG2压缩的). 例如，超声设备产生的是电影式独立序列帧，然而最新
  的内窥镜产生真正的视频。
* 根据上下文,"支持"可以是指"*Orthanc能够 query/receive/transfer DICOM 文件吗?*",
  或者 "*Orthanc能够 render/play DICOM文件吗?*".

假如你恰恰指的是**query/receive/transfer** DICOM视频, Orthanc能够很好的工作不管
是2D+t 或者真正视频(因为 Orthanc 是一个 `VNA厂商中立的存档 <https://en.wikipedia.org/wiki/Vendor_Neutral_Archive>`__ )。
这个特性也在 :ref:`另一个FAQ入口<supported-images>` 里讨论。

假如你指的是 **play**视频, 据我们所知，现在还没有支持H.264 或 MPEG2 的Orthanc插件 (虽然
这样的插件将来应该会开发出来)。 然而，如果你的视频是2D+t(cine)序列，Orthanc已经可以在Web浏览器
以至少有两种不同的方式显示:

1. 内置的管理接口叫 :ref:`Orthanc Explorer <orthanc-explorer>` 能够显示独立序列帧并手工通过
   键盘导航。
2. 官方的 `Web viewer插件 <http://www.orthanc-server.com/static.php?page=web-viewer>`__
   允许您使用鼠标滚轮来显示连续的视频的帧。

总结一下，如果你的视频没有使用MPEG2或H264编码，或者如果你不需要在浏览器中播放视频，Orthanc
实际上支持视频。
