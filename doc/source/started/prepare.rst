准备环境
============

创建目录
--------

为教程创建目录，比如在用户临时空间中创建目录 ``/g7/JOB_TMP/wangdp/tutorial/ecflow``，并将该目录设为环境变量 ``TUTORIAL_HOME``。

.. code-block:: shell

    export TUTORIAL_HOME=/g7/JOB_TMP/wangdp/tutorial/ecflow
    mkdir -p ${TUTORIAL_HOME}
    cd ${TUTORIAL_HOME}

.. note::

    HPC2023 个人账户注销时间太短，注意重新登录进行后续操作前需要再次设置环境变量 ``TUTORIAL_HOME``

在该目录中创建以下几个子目录：

* def：保存 ecFlow 的工作流定义文件和任务的 ecf 脚本文件
* ecfout：ecFlow 服务运行的目录，ecFlow 服务的日志会保存在该目录下
* program：保存模式程序、配置文件和脚本
* workdir：模式运行的目录

.. code-block:: shell

    mkdir -p def
    mkdir -p ecfout
    mkdir -p program
    mkdir -p workdir

拷贝程序
--------

将 CMA-GFS 后处理程序包拷贝到 ``program`` 目录下：

.. note::

    CMA-GFS 后处理序包来自 HPC2023 最新测试版本，但经过删减，去掉本教程用不到的一些程序。

.. code-block:: shell

    cd ${TUTORIAL_HOME}/program
    cp -r ${TUTORIAL_PACKAGE}/cma-gfs-post-program .
