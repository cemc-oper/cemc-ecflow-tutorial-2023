定义一个工作流
===============

下面首先创建只有一个任务的工作流，在 ecFlow 中工作流被称为 suite。

创建 python 文件
-----------------

在 ``${TUTORIAL_HOME}/def`` 目录中创建文件 **cma_gfs_post.py**：

.. code-block:: py
    :linenos:

    import os

    import ecflow


    current_path = os.path.dirname(__file__)
    tutorial_base = os.path.abspath(os.path.join(current_path, "../"))
    def_path = os.path.join(tutorial_base, "def")
    ecfout_path = os.path.join(tutorial_base, "ecfout")
    program_base_dir = os.path.join(tutorial_base, "program/cma-gfs-post-program")
    run_base_dir = os.path.join(tutorial_base, "workdir")

    defs = ecflow.Defs()

    with defs.add_suite("cma_gfs_post") as suite:
        suite.add_variable("PROGRAM_BASE_DIR", program_base_dir)
        suite.add_variable("RUN_BASE_DIR", run_base_dir)

        suite.add_variable("ECF_INCLUDE", os.path.join(def_path, "include"))
        suite.add_variable("ECF_FILES", os.path.join(def_path, "ecffiles"))

        suite.add_variable("ECF_DATE", "20230806")
        suite.add_variable("HH", "00")

        with suite.add_task("pre_data2grib2") as tk_pre_data2grib2:
            pass

    print(defs)
    def_output_path = str(os.path.join(def_path, "cma_gfs_post.def"))
    defs.save_as_defs(def_output_path)

上述脚本主要完成如下操作：

- 15 行：定义名为 ``cma_tym`` 的工作流 (suite)
- 16-23 行：为 suite 定义多个变量 (variable)，包括目录等
- 25-26 行：定义名为 ``pre_data2grib2`` 的任务 (task)
- 29-30 行：将工作流定义写入到文件 **cma_gfs_post.def** 中

生成 def 文件
-------------

运行 Python 脚本 **cma_gfs_post.py**，生成工作流定义文件 **cma_gfs_post.def**：

.. code-block:: bash

    cd ${TUTORIAL_HOME}/def
    python3 cma_gfs_post.py

**cma_gfs_post.def** 文件是纯文本格式描述的工作流定义，文件内容如下：

.. code-block::

    #5.10.0
    suite cma_gfs_post
      edit PROGRAM_BASE_DIR '/g7/JOB_TMP/wangdp/tutorial/ecflow/program/cma-gfs-post-program'
      edit RUN_BASE_DIR '/g7/JOB_TMP/wangdp/tutorial/ecflow/workdir'
      edit ECF_INCLUDE '/g7/JOB_TMP/wangdp/tutorial/ecflow/def/include'
      edit ECF_FILES '/g7/JOB_TMP/wangdp/tutorial/ecflow/def/ecffiles'
      edit ECF_DATE '20230806'
      edit HH '00'
      task pre_data2grib2
    endsuite
    # enddef
