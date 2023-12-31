添加 Family
=============

上一节仅生成 024 时效的 GRIB2 数据产品，本节中我们为 CMA-GFS 00 时次生成的 10 天 240 小时预报都生成 GRIB2 产品。

下面将为每个时效创建一个 Family 节点。

修改工作流定义
--------------

更新 ``${TUTORIAL_HOME}/def`` 中的工作流定义文件 **cma_gfs_post.py**：

.. code-block:: py
    :linenos:
    :emphasize-lines: 47-48,52-62

    import os

    import ecflow


    def slurm_serial(class_name, wckey):
        variables = {
            "ECF_JOB_CMD": "slsubmit6 %ECF_JOB% %ECF_NAME% %ECF_TRIES% %ECF_TRYNO% %ECF_HOST% %ECF_PORT%",
            "ECF_KILL_CMD": "slcancel4 %ECF_RID% %ECF_NAME% %ECF_HOST% %ECF_PORT%",
        	"CLASS": class_name,
            "WCKEY": wckey,
        }
        return variables


    def slurm_parallel(nodes, tasks_per_node, class_name, wckey):
        variables = {
            "ECF_JOB_CMD": "slsubmit6 %ECF_JOB% %ECF_NAME% %ECF_TRIES% %ECF_TRYNO% %ECF_HOST% %ECF_PORT%",
            "ECF_KILL_CMD": "slcancel4 %ECF_RID% %ECF_NAME% %ECF_HOST% %ECF_PORT%",
            "NODES": nodes,
            "TASKS_PER_NODE": tasks_per_node,
        	"CLASS": class_name,
            "WCKEY": wckey,
        }
        return variables


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

        suite.add_limit("total_tasks", 10)
        suite.add_inlimit("total_tasks")

        forecast_hour_list = [ f"{hour:03}" for hour in range(0, 241, 3)]

        for forecast_hour in forecast_hour_list:
            with suite.add_family(forecast_hour) as fm_hour:
                fm_hour.add_variable("FHOUR", forecast_hour)

                with fm_hour.add_task("pre_data2grib2") as tk_pre_data2grib2:
                    tk_pre_data2grib2.add_variable(slurm_serial("serial", "105-09"))

                with fm_hour.add_task("data2grib2") as tk_data2grib2:
                    tk_data2grib2.add_variable(slurm_parallel(4, 64, "normal", "105-09"))
                    tk_data2grib2.add_trigger("./pre_data2grib2 == complete")

    print(defs)
    def_output_path = str(os.path.join(def_path, "cma_gfs_post.def"))
    defs.save_as_defs(def_output_path)

47-48 行创建一个 Limit，并将该 Limit 应用到 cma_gfs_post 节点上，限制同时运行的任务不能超过 10 个。

.. warning::

    对于需要运行大量任务的工作流，一定要限制同时运行的任务数，避免同一时间大量提交作业导致 ecFlow 服务卡死。

50-54 行为每个时效创建创建 Family，逐三小时创建以三位时效数字作为名称的任务。

更新工作流
-----------

与前一节一样，需要重新生成 def 文件并更新到 ecFlow 服务中。
在更新之前，我们先在 ecFlow UI 中将 cma_gfs_post 节点挂起，避免更新后系统提交大量作业。

更新工作流：

.. code-block:: bash

    cd ${TUTORIAL_HOME}/def
    python3 cma_gfs_post.py
    ecflow_client --port 43083 --replace /cma_gfs_post cma_gfs_post.def

更新任务脚本
------------

在 ``${TUTORIAL_HOME}/def/ecffiles`` 目录下更新 ecf 脚本 **pre_data2grib2.ecf** 和 **data2grib2.ecf**。
使用 ``FHOUR`` 表示时效，将 ``forecast_hour=024`` 替换为 ``forecast_hour=%FHOUR%``。

运行任务
---------

创建脚本后，可以恢复 cma_gfs_post 自动运行。
右键单击 cma_gfs_post，选择 Resume，恢复工作流运行。
可以看到前 10 个作业开始运行：

.. image:: image/ecflow-ui-run-limit.png

因为我们创建的 Limit (total_tasks) 限制最多运行 10 个任务，所以有 1 个任务运行结束后第 11 个任务才会自动运行。
