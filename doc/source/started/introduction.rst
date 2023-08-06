简介
===========

首先介绍使用的预报数据和实验环境。

预报数据
---------

使用 CMA-GFS 预报子系统生成 modelvar 二进制数据。

实验环境
-----------

下面介绍如何使用 HPC2023 高性能计算机搭建 ecFlow 流程。

使用账户用户（例如 *wangdp*）登录 HPC2023 业务分区 *login_a13* 节点。

在使用 ecFlow 前需要加载 ecFlow 软件包：

.. code-block::

    module load ecflow/5.9.0/gnu

查看 ecFlow 版本：

.. code-block:: bash

    ecflow_client --version

.. code-block::

    Ecflow version(5.10.0) boost(1.71.0) compiler(gcc 8.4.1) protocol(JSON cereal 1.3.0) openssl(enabled) Compiled on May 10 2023 04:07:20

将本教程程序包目录设为环境变量 ``TUTORIAL_PACKAGE``：

.. code-block::

    export TUTORIAL_PACKAGE=/g4/wangdp/project/course/ecflow/2023/package

在开始之前，请检查 ``$HOME/bin`` 目录是否存在 ``slsubmit6`` 和 ``slcancel4`` 两个脚本。
如果脚本不存在或目录不存在，请创建 ``$HOME/bin`` 目录并拷贝这两个脚本：

.. warning::

    上述两个脚本为简化版，如果当前用户已有这两个脚本，请 **不要** 覆盖。

.. code-block:: bash

    mkdir -p ${HOME}/bin
    cd ${HOME}/bin
    cp ${TUTORIAL_PACKAGE}/slcancel4 .
    cp ${TUTORIAL_PACKAGE}/slsubmit6 .

检查 ``slsubmit6`` 是否可以被找到：

.. code-block:: bash

    which slsubmit6

如果输出类似

.. code-block::

    ~/bin/slsubmit6

则表示 ``slsubmit6`` 在环境变量 ``PATH`` 目录中。

如果没有输出结果，则需要检查环境变量 ``PATH`` 是否包含 ``~/bin`` 目录。

.. note::

    **如果使用 CMA-PI 高性能计算机**：

    系统默认已将 ecFlow 程序包加入到环境变量 ``PATH`` 中，默认版本为 4.11.1。

    程序包默认路径 ``TUTORIAL_PACKAGE`` 为：

    .. code-block:: bash

        export TUTORIAL_PACKAGE=/g11/wangdp/project/course/ecflow/2023/package
