.. _rest:

Orthanc的REST API
===================

.. contents::
   :depth: 3

Orthanc的一个主要优势在于它内置 `RESTful API
<https://en.wikipedia.org/wiki/Representational_state_transfer>`__,
可以用来从外部应用中驱动Orthanc，自主选择开发的编程语言开发这些应用程序。
Orthanc的REST API对Orthanc的所有核心特性进行了全面的编程访问。

重要的是, Orthanc Explorer (Orthanc的嵌入式管理界面)完全依赖于这个REST API的所有特性。
这意味着任何可以通过Orthanc Explorer完成的事情，也可以通过REST查询完成。

*注意:* 所有的例子都用 `cURL command-line tool <https://curl.haxx.se/>`__,
但同样的也很容易转换到任何同时支持HTTP和JSON的编程语言。


.. _sending-dicom-images:

发送DICOM影像
---------------

.. highlight:: bash

通过查询REST API，可以上传DICOM文件，语法如下::

    $ curl -X POST http://localhost:8042/instances --data-binary @CT.X.1.2.276.0.7230010.dcm

.. highlight:: json

Orthanc将使用一个JSON文件进行响应，该文件包含有关存储实例位置的信息,例如::

    {
      "ID" : "e87da270-c52b-4f2a-b8c6-bae25928d0b0",
      "Path" : "/instances/e87da270-c52b-4f2a-b8c6-bae25928d0b0",
      "Status" : "Success"
    }

.. highlight:: bash

注意，对于curl，设置“Expect”HTTP报头就可以了显著 `减少POST请求的执行时间
<http://stackoverflow.com/questions/463144/php-http-post-fails-when-curl-data-1024/463277#463277>`__::

    $ curl -X POST -H "Expect:" http://localhost:8042/instances --data-binary @CT.X.1.2.276.0.7230010.dcm

Orthanc的代码分发版包含一个 `Python脚本示例
<https://bitbucket.org/sjodogne/orthanc/src/default/Resources/Samples/ImportDicomFiles/ImportDicomFiles.py>`__
，它展示了递归地将某些文件夹的内容通过REST API上载到Orthanc::

    $ python ImportDicomFiles.py localhost 8042 ~/DICOM/


.. _rest-access:

访问Orthanc的内容
------------------

Orthanc使用"Patient,Study, Series, Instance"DICOM标准模型存储DICOM资源。每个DICOM
资源与一个 :ref:`unique identifier <orthanc-ids>` 相关联。

列出所有DICOM资源
^^^^^^^^^^^^^^^^^^^

下面是如何列出存储在你的本地Orthanc实例里的所有DICOM资源的方法::

    $ curl http://localhost:8042/patients
    $ curl http://localhost:8042/studies
    $ curl http://localhost:8042/series
    $ curl http://localhost:8042/instances

注意，这个命令的结果是一个包含一个资源标识符数组的`JSON 文件
<https://en.wikipedia.org/wiki/Json>`__ JSON文件格式是轻量级的，可以是
几乎由任何计算机语言解析。

访问一个patient
^^^^^^^^^^^^^^^^

.. highlight:: bash

To access a single resource, add its identifier to the `URI
<https://en.wikipedia.org/wiki/Uniform_resource_identifier>`__. You
would for instance retrieve the main information about one patient as
follows::

    $ curl http://localhost:8042/patients/dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94

.. highlight:: json

Here is a possible answer from Orthanc::

 {
   "ID" : "07a6ec1c-1be5920b-18ef5358-d24441f3-10e926ea",
   "MainDicomTags" : {
      "OtherPatientIDs" : "(null)",
      "PatientBirthDate" : "0",
      "PatientID" : "000000185",
      "PatientName" : "Anonymous^Unknown",
      "PatientSex" : "O"
   },
   "Studies" : [ "9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15" ],
   "Type" : "Patient"
 }

This is once again a JSON file. Note how Orthanc gives you a summary
of the main DICOM tags that correspond to the patient level.

从病人到实例的浏览
^^^^^^^^^^^^^^^^^^^^^

.. highlight:: bash

The field ``Studies`` list all the DICOM studies that are associated
with the patient. So, considering the patient above, we would go down
in her DICOM hierarchy as follows::

    $ curl http://localhost:8042/studies/9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15

.. highlight:: json

And Orthanc could answer::

 {
   "ID" : "9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15",
   "MainDicomTags" : {
      "AccessionNumber" : "(null)",
      "StudyDate" : "20120716",
      "StudyDescription" : "TestSUVce-TF",
      "StudyID" : "23848",
      "StudyInstanceUID" : "1.2.840.113704.1.111.7016.1342451220.40",
      "StudyTime" : "170728"
   },
   "ParentPatient" : "07a6ec1c-1be5920b-18ef5358-d24441f3-10e926ea",
   "Series" : [
      "6821d761-31fb55a9-031ebecb-ba7f9aae-ffe4ddc0",
      "2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35",
      "7384c47e-6398f2a8-901846ef-da1e2e0b-6c50d598"
   ],
   "Type" : "Study"
 }

.. highlight:: bash

The main DICOM tags are now those that are related to the study
level. It is possible to retrieve the identifier of the patient in the
``ParentPatient`` field, which can be used to go upward the DICOM
hierarchy. But let us rather go down to the series level by using the
``Series`` array. The next command would return information about one
of the three series that have just been reported::

    $ curl http://localhost:8042/series/2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35

.. highlight:: json

Here is a possible answer::

 {
   "ExpectedNumberOfInstances" : 45,
   "ID" : "2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35",
   "Instances" : [
      "41bc3f74-360f9d10-6ae9ffa4-01ea2045-cbd457dd",
      "1d3de868-6c4f0494-709fd140-7ccc4c94-a6daa3a8",
      <...>
      "1010f80b-161b71c0-897ec01b-c85cd206-e669a3ea",
      "e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4"
   ],
   "MainDicomTags" : {
      "Manufacturer" : "Philips Medical Systems",
      "Modality" : "PT",
      "NumberOfSlices" : "45",
      "ProtocolName" : "CHU/Body_PET/CT___50",
      "SeriesDate" : "20120716",
      "SeriesDescription" : "[WB_CTAC] Body",
      "SeriesInstanceUID" : "1.3.46.670589.28.2.12.30.26407.37145.2.2516.0.1342458737",
      "SeriesNumber" : "587370",
      "SeriesTime" : "171121",
      "StationName" : "r054-svr"
   },
   "ParentStudy" : "9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15",
   "Status" : "Complete",
   "Type" : "Series"
 }

It can be seen that this series comes from a PET modality. Orthanc has
computed that this series should contain 45 instances.

.. highlight:: bash

So far, we have navigated from the patient level, to the study level,
and finally to the series level. There only remains the instance
level. Let us dump the content of one of the instances::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4

.. highlight:: json

The instance contains the following information::

 {
   "FileSize" : 70356,
   "FileUuid" : "3fd265f0-c2b6-41a2-ace8-ae332db63e06",
   "ID" : "e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4",
   "IndexInSeries" : 6,
   "MainDicomTags" : {
      "ImageIndex" : "6",
      "InstanceCreationDate" : "20120716",
      "InstanceCreationTime" : "171344",
      "InstanceNumber" : "6",
      "SOPInstanceUID" : "1.3.46.670589.28.2.15.30.26407.37145.3.2116.39.1342458737"
   },
   "ParentSeries" : "2cc6336f-2d4ae733-537b3ca3-e98184b1-ba494b35",
   "Type" : "Instance"
 }

.. highlight:: bash

The instance has the index 6 in the parent series. The instance is
stored as a raw DICOM file of 70356 bytes. You would download this
DICOM file using the following command::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/file > Instance.dcm


以JSON文件的形式访问实例的DICOM字段
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. highlight:: bash

When one gets to the instance level, you can retrieve the hierarchy of
all the DICOM tags of this instance as a JSON file::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/simplified-tags

.. highlight:: json

Here is a excerpt of the Orthanc answer::

 {
   "ACR_NEMA_2C_VariablePixelDataGroupLength" : "57130",
   "AccessionNumber" : null,
   "AcquisitionDate" : "20120716",
   "AcquisitionDateTime" : "20120716171219",
   "AcquisitionTime" : "171219",
   "ActualFrameDuration" : "3597793",
   "AttenuationCorrectionMethod" : "CTAC-SG",
   <...>
   "PatientID" : "000000185",
   "PatientName" : "Anonymous^Unknown",
   "PatientOrientationCodeSequence" : [
      {
         "CodeMeaning" : "recumbent",
         "CodeValue" : "F-10450",
         "CodingSchemeDesignator" : "99SDM",
         "PatientOrientationModifierCodeSequence" : [
            {
               "CodeMeaning" : "supine",
               "CodeValue" : "F-10340",
               "CodingSchemeDesignator" : "99SDM"
            }
         ]
      }
   ],
   <...>
   "StudyDescription" : "TestSUVce-TF",
   "StudyID" : "23848",
   "StudyInstanceUID" : "1.2.840.113704.1.111.7016.1342451220.40",
   "StudyTime" : "171117",
   "TypeOfDetectorMotion" : "NONE",
   "Units" : "BQML",
   "Unknown" : null,
   "WindowCenter" : "1.496995e+04",
   "WindowWidth" : "2.993990e+04"
 }

.. highlight:: bash

If you need more detailed information about the type of the variables
or if you wish to use the hexadecimal indexes of DICOM tags, you are
free to use the following URL::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/tags

访问实例的原始DICOM字段
^^^^^^^^^^^^^^^^^^^^^^^^^

.. highlight:: bash

You also have the opportunity to access the raw value of the DICOM
tags of an instance, without going through a JSON file. Here is how
you would find the Patient Name of the instance::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/content/0010-0010
    Anonymous^Unknown

The list of all the available tags for this instance can also be retrieved
easily::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/content

It is also possible to recursively explore the sequences of tags::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/content/0008-1250/0/0040-a170/0/0008-0104
    For Attenuation Correction

The command above has opened the "0008-1250" tag that is a DICOM
sequence, taken its first child, opened its "0040-a170" tag that is
also a sequence, taken the first child of this child, and returned the
"0008-0104" DICOM tag.

下载影像
^^^^^^^^^

.. highlight:: bash

It is also possible to download a preview PNG image that corresponds to some DICOM instance::

    $ curl http://localhost:8042/instances/e668dcbf-8829a100-c0bd203b-41e404d9-c533f3d4/preview > Preview.png

The resulting image will be a standard graylevel PNG image that can be opened by any
 painting software.


.. _changes:



发送资源到远程模态
--------------------

Orthanc can send its DICOM instances to remote DICOM modalities (C-Store SCU).
 This process can be triggered by the REST API.

配置
^^^^^^^^

.. highlight:: json

You first have to declare the AET, the IP address and the port number
of the remote modality inside the :ref:`configuration file
<configuration>`. For instance, here is how to declare a remote
modality::

    ...
    "DicomModalities" : {
      "sample" : [ "STORESCP", "127.0.0.1", 2000 ]
    },
    ...

.. highlight:: bash

Such a configuration would enable Orthanc to connect to another DICOM
store (for instance, another Orthanc instance) that listens on the
localhost on the port 2000. The modalities that are known to Orthanc
can be queried::

    $ curl http://localhost:8042/modalities


发送一个资源
^^^^^^^^^^^^^

.. highlight:: bash

Once you have identified the Orthanc identifier of the DICOM resource
that would like to send :ref:`as explained above <rest-access>`, you
would use the following command to send it::

    $ curl -X POST http://localhost:8042/modalities/sample/store -d c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f

The ``/sample/`` component of the URI corresponds to the identifier of
the remote modality, as specified above in the configuration file.

Note that you can send isolated DICOM instances with this command,
 but also entire patients, studies or series.

Bulk Store SCU
^^^^^^^^^^^^^^

.. highlight:: bash

Each time a POST request is made to ``/modalities/.../store``, a new
DICOM connection is possibly established. This may lead to a large
communication overhead if sending multiple isolated instances.

To circumvent this problem, you have 2 possibilities:

1. Set the ``DicomAssociationCloseDelay`` option in the
   :ref:`configuration file <configuration>` to a non-zero value. This
   will keep the DICOM connection open for a certain amount of time,
   waiting for new instances to be routed.

2. If you do not want to keep the connection open but inactive, it is
   possible to send multiple instances with a single POST request
   (so-called "Bulk Store SCU", available from Orthanc 0.5.2)::

    $ curl -X POST http://localhost:8042/modalities/sample/store -d '["d4b46c8e-74b16992-b0f5ca11-f04a60fa-8eb13a88","d5604121-7d613ce6-c315a5-a77b3cf3-9c253b23","cb855110-5f4da420-ec9dc9cb-2af6a9bb-dcbd180e"]'

   The list of the resources to be sent are given as a JSON array. In
   this case, a single DICOM connection is used. `Sample code is
   available
   <https://bitbucket.org/sjodogne/orthanc/src/default/Resources/Samples/Python/HighPerformanceAutoRouting.py>`__.


使用REST执行查Query/Retrieve 和 Find
------------------------------------

*Section contributed by Bryan Dearlove*

Orthanc can be used to perform queries on the local Orthanc instance,
or on remote modalities through the REST API.

To perform a query of a remote modality you must define the modality
within the :ref:`configuration file <configuration>` (See
Configuration section under Sending resources to remote modalities).


执行查询
^^^^^^^^^^^

.. highlight:: bash

To initiate a query you perform a POST command against the Modality
with the identifiers you are looking for. The the example below we are
performing a study level query against the modality sample for any
study descriptions with the word chest within it. This search is case
insensitive unless configured otherwise within the Orthanc
configuration file::

     $ curl --request POST \
       --url http://localhost:8042/modalities/sample/query \
       --data '{"Level":"Study","Query": {"PatientID":"","StudyDescription":"*Chest*","PatientName":""}}'


.. highlight:: json

You will receive back an ID which can be used to retrieve more
information with GET commands or C-Move requests with a POST Command::

     {
        "ID": "5af318ac-78fb-47ff-b0b0-0df18b0588e0",
        "Path": "/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0"
     }


附加选项
^^^^^^^^^^^

.. highlight:: json

You can use patient identifiers by including the `*` within your
search. For example if you were searching for a name beginning with
`Jones` you can do::

  "PatientName":"Jones*"

If you wanted to search for a name with the words `Jo` anywhere within
it you can do::

  "PatientName":"*Jo*"

To perform date searches you can specify within StudyDate a starting
date and/or a before date. For example ``"StudyDate":"20180323-"``
would search for all study dates after the specified date to
now. Doing ``"StudyDate":"20180323-20180325"`` would search for all
study dates between the specified date.


检验Level
^^^^^^^^^^^^^^^

.. highlight:: bash

::

   $ curl --request GET --url http://localhost:8042/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0/level

Will retrieve the level with which the query was performed, Study,
Series or Instance.


检验Modality
^^^^^^^^^^^^^^^^^^

.. highlight:: bash

::

   $ curl --request GET --url http://localhost:8042/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0/modality

Will provide the modality name which the original query was performed against.


检验Query
^^^^^^^^^^^^^^^

.. highlight:: bash

To retrieve information on what identifiers the query was originally
performed using you can use the query filter::

  $ curl --request GET --url http://localhost:8042/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0/query


检验Query应答
^^^^^^^^^^^^^^^^^^^^^^^

.. highlight:: bash

You are able to individually review each answer returned by performing
a GET with the answers parameter::

  $ curl --request GET --url http://localhost:8042/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0/answers

You will get a JSON back with numbered identifiers for each answer you
received back. For example because we performed a Study level query we
received back 5 studies answers back. We are able to query each answer
for content details::

  $ curl --request GET --url http://localhost:8042/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0/answers/0/content

If there are content items missing, you may add them by adding that
identifier to the original query. For example if we wanted Modalities
listed in this JSON answer in the initial query we would add to the
POST body: `"ModalitiesInStudy":""`


执行Retrieve (C-Move)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. highlight:: bash

You can perform a C-Move to retrieve all studies within the original
query using a post command and identifying the Modality to be one to
in the POST contents::

  $ curl --request POST --url http://localhost:8042/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0/retrieve --data Orthanc

You are also able to perform individual C-Moves for a content item by
specifying that individual content item::

  $ curl --request POST --url http://localhost:8042/queries/5af318ac-78fb-47ff-b0b0-0df18b0588e0/answers/0/retrieve --data Orthanc


在Orthanc上执行Finds
^^^^^^^^^^^^^^^^^^^^^
.. highlight:: bash

Performing a find within Orthanc is very similar to using Queries
against DICOM modalities and the additional options listed above work
with find also.  When performing a find, you will receive the Orthanc
ID's of all the matched items within your find. For example if you
perform a study level find and 5 Studies match you will receive 5
study level Orthanc ID's in JSON format as a response::

  $ curl --request POST --url http://localhost:8042/tools/find --data '{"Level":"Instance","Query":{"Modality":"CR","StudyDate":"20180323-","PatientID":"*"}}'


附加选项
^^^^^^^^^^
.. highlight:: json

You also have the ability to limit the responses by specifying a limit within the body of
 the POST message. For example::

  "Limit":4


跟踪变化
--------

.. highlight:: bash

Whenever Orthanc receives a new DICOM instance, this event is recorded
in the so-called "Changes Log". This enables remote scripts to react
to the arrival of new DICOM resources. A typical application is
**auto-routing**, where an external script waits for a new DICOM
instance to arrive into Orthanc, then forward this instance to another
modality.

The Changes Log can be accessed by the following command::

    $ curl http://localhost:8042/changes

.. highlight:: json

Here is a typical output::

 {
   "Changes" : [
      {
         "ChangeType" : "NewInstance",
         "Date" : "20130507T143902",
         "ID" : "8e289db9-0e1437e1-3ecf395f-d8aae463-f4bb49fe",
         "Path" : "/instances/8e289db9-0e1437e1-3ecf395f-d8aae463-f4bb49fe",
         "ResourceType" : "Instance",
         "Seq" : 921
      },
      {
         "ChangeType" : "NewSeries",
         "Date" : "20130507T143902",
         "ID" : "cceb768f-e0f8df71-511b0277-07e55743-9ef8890d",
         "Path" : "/series/cceb768f-e0f8df71-511b0277-07e55743-9ef8890d",
         "ResourceType" : "Series",
         "Seq" : 922
      },
      {
         "ChangeType" : "NewStudy",
         "Date" : "20130507T143902",
         "ID" : "c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f",
         "Path" : "/studies/c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f",
         "ResourceType" : "Study",
         "Seq" : 923
      },
      {
         "ChangeType" : "NewPatient",
         "Date" : "20130507T143902",
         "ID" : "dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94",
         "Path" : "/patients/dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94",
         "ResourceType" : "Patient",
         "Seq" : 924
      }
   ],
   "Done" : true,
   "Last" : 924
 }

This output corresponds to the receiving of one single DICOM instance
by Orthanc. It records that a new instance, a new series, a new study
and a new patient has been created inside Orthanc. Note that each
changes is labeled by a ``ChangeType``, a ``Date`` (in the `ISO format
<https://en.wikipedia.org/wiki/ISO_8601>`__), the location of the
resource inside Orthanc, and a sequence number (``Seq``).

Note that this call is non-blocking. It is up to the calling program
to wait for the occurrence of a new event (by implementing a polling
loop).

.. highlight:: bash

This call only returns a fixed number of events, that can be changed
by using the ``limit`` option::

    $ curl http://localhost:8042/changes?limit=100

The flag ``Last`` records the sequence number of the lastly returned
event. The flag ``Done`` is set to ``true`` if no further event has
occurred after this lastly returned event. If ``Done`` is set to
``false``, further events are available and can be retrieved. This is
done by setting the ``since`` option that specifies from which
sequence number the changes must be returned::

    $ curl 'http://localhost:8042/changes?limit=100&since=922'

A `sample code in the source distribution
<https://bitbucket.org/sjodogne/orthanc/src/default/Resources/Samples/Python/ChangesLoop.py>`__
shows how to use this Changes API to implement a polling loop.


从Orthanc上删除资源
-------------------------------

.. highlight:: bash

删除 patients, studies, series 或 instances
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Deleting DICOM resources (i.e. patients, studies, series or instances)
from Orthanc is as simple as using a HTTP DELETE on the URI of this
resource.

Concretely, you would first explore the resources that are stored in
Orthanc :ref:`as explained above <rest-access>`::

    $ curl http://localhost:8042/patients
    $ curl http://localhost:8042/studies
    $ curl http://localhost:8042/series
    $ curl http://localhost:8042/instances

Secondly, once you have spotted the resources to be removed, you would
use the following command-line syntax to delete them::

    $ curl -X DELETE http://localhost:8042/patients/dc65762c-f476e8b9-898834f4-2f8a5014-2599bc94
    $ curl -X DELETE http://localhost:8042/studies/c4ec7f68-9b162055-2c8c5888-5bf5752f-155ab19f
    $ curl -X DELETE http://localhost:8042/series/cceb768f-e0f8df71-511b0277-07e55743-9ef8890d
    $ curl -X DELETE http://localhost:8042/instances/8e289db9-0e1437e1-3ecf395f-d8aae463-f4bb49fe


清除变化的log日志
^^^^^^^^^^^^^^^^^^

:ref:`As described above <changes>`, Orthanc keeps track of all the
changes that occur in the DICOM store. This so-called "Changes Log"
is accessible at the following URI::

    $ curl http://localhost:8042/changes

To clear the content of the Changes Log, simply DELETE this URI::

    $ curl -X DELETE http://localhost:8042/changes

清除导出资源的log日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For medical traceability, Orthanc stores a log of all the resources
that have been exported to remote modalities::

    $ curl http://localhost:8042/exports

In auto-routing scenarios, it is important to prevent this log to grow
indefinitely as incoming instances are routed. You can either disable
this logging by setting the option ``LogExportedResources`` to ``false``
in the :ref:`configuration file <configuration>`, or periodically
clear this log by DELETE-ing this URI::

    $ curl -X DELETE http://localhost:8042/exports


匿名化和修改
------------------

对DICOM资源进行匿名化和修改的过程是在 :ref:`单独的文档 <anonymization>`.


延伸阅读
----------

The examples above have shown you the basic principles for driving an
instance of Orthanc through its REST API. All the possibilities of the
API have not been described:

* A :ref:`FAQ entry <rest-samples>` lists where you can find more
  advanced samples of the REST API of Orthanc.
* The full documentation of the REST API is maintained as an online
  spreadsheet accessible from the `documentation part of the official
  Web site
  <http://www.orthanc-server.com/static.php?page=documentation>`__
  (click on the *Reference of the REST API* button).
