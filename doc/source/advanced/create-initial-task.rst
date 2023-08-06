添加数据文件检测任务
=====================

从本节起，将介绍 ecFlow 的一些进阶功能。
我们将朝着实时运行的业务系统流程更前进一步，我们将为 CMA-GFS 后处理流程添加数据文件检测任务，并通过设置时间依赖和日期循环参数实现一个可以实时滚动运行的流程。

前面的 pre_data2grib 任务默认 CMA-GFS 已生成了对应时次时效的 modelvar 文件。
实际上，实时运行的后处理系统为了保证产品的时效性，不能等全部时效 modelvar 都生成后再运行后处理任务，
而是要在模式积分逐渐输出文件的过程中就开始运行后处理流程，只要有一个时效的文件输出，就运行对应时效的后处理流程。

后处理流程通常运行一个程序顺序检测模式是否生成各个时效的 modelvar 文件，通过事件 (event) 触发后续的流程。

本节将创建数据文件检测任务，为每个时效设置一个事件，用来触发后续的流程。

更新工作流定义
--------------

更新 ``${TUTORIAL_HOME}/def`` 中的工作流定义文件 **cma_tym.py**：

.. code-block:: py
    :linenos:
    :emphasize-lines: 54-56,61

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

        suite.add_variable(slurm_serial("serial", "105-09"))

        forecast_hour_list = [ f"{hour:03}" for hour in range(0, 241, 3)]

        with suite.add_task("initial") as tk_initail:
            for forecast_hour in forecast_hour_list:
                tk_initail.add_event(f"modvar_{forecast_hour}")

        for forecast_hour in forecast_hour_list:
            with suite.add_family(forecast_hour) as fm_hour:
                fm_hour.add_variable("FHOUR", forecast_hour)
                fm_hour.add_trigger(f"./initial:modvar_{forecast_hour}")

                with fm_hour.add_task("pre_data2grib2") as tk_pre_data2grib2:
                    pass

                with fm_hour.add_task("data2grib2") as tk_data2grib2:
                    tk_data2grib2.add_variable(slurm_parallel(4, 64, "normal", "105-09"))
                    tk_data2grib2.add_trigger("./pre_data2grib2 == complete")

    print(defs)
    def_output_path = str(os.path.join(def_path, "cma_gfs_post.def"))
    defs.save_as_defs(def_output_path)

新增代码解析：

- 54-56 行：添加 initial 任务，设置 000 到 240 的事件
- 61 行：为时效节点增加触发器，等待 initial 相应事件设置后才运行

更新工作流
----------

挂起 cma_gfs_post 节点，更新 ecFlow 上的工作流：

.. code-block:: bash

    cd ${TUTORIAL_HOME}/def
    python3 cma_gfs_post.py
    ecflow_client --host login_a13 --port 43083 --replace /cma_gfs_post cma_gfs_post.def

创建 initial 任务脚本
-----------------------

在 ``${TUTORIAL_HOME}/def/ecffiles`` 中创建 ecf 脚本 **initial.ecf**：

.. code-block:: bash
    :linenos:
    :emphasize-lines: 10-12,29-69

    #!/bin/ksh
    %include <slurm_serial.h>
    %include <head.h>
    %include <configure.h>

    date

    #=======================

    max_check_count=1200
    sleep_seconds_for_check=1
    sleep_seconds_for_next_time=1

    #=======================
    bin_dir=${PROGRAM_BIN_DIR}
    condat_dir=${PROGRAM_CON_DIR}

    run_dir=${CYCLE_RUN_BASE_DIR}

    #------------------------
    start_time=${START_TIME}

    #--------------
    test -d ${run_dir} || mkdir -p ${run_dir}
    cd ${run_dir}

    #------------------

    check_file_without_size_change()
    {
        start_time=$1
        f_time=$2
        file_path=$3

        count=0
        get_data=0
        while [[ ${count} -lt ${max_check_count} ]] && [ ${get_data} -ne 1 ]
        do
            echo "check...${start_time} ${f_time}...${count}/${max_check_count}"
            file_path=/g2/op_gfs/OPER_ARCH_TEST/GRAPES_GFS_GMF/Fcst-long/${start_time}/modelvar${start_time}_${f_time}

            if [ -e ${file_path} ]; then
                echo "check...${file_path}...${count}/${max_check_count}: check size change"
                last_file_size=-1
                while [[ ${count} -lt ${max_check_count} ]] && [[ $get_data -eq 0 ]]
                do
                    echo "check...${file_path}...${count}/${max_check_count}: check size chagne"
                    current_file_size=$(stat -c %%s ${file_path})
                    if [[ ${last_file_size} -eq ${current_file_size} ]]; then
                        get_data=1
                        echo "check...${file_path}...${count}/${max_check_count}: check size chagne success"
                    else
                        last_file_size=${current_file_size}
                        sleep ${sleep_seconds_for_check}
                    fi
                    count=$(($count+1))
                done
            else
                sleep ${sleep_seconds_for_check}
            fi
            count=$(($count+1))
        done
        if [[ $get_data -eq 0 ]];then
            echo "check...${start_time} ${f_time}...failed (too many times)"
            return 1;
        fi
        echo "check...${file_path}...found"
        return 0;
    }

    #===================================================
    # check modelvar file one by one, and set event.
    forecast_array="$(seq -f %%03g 0 3 240)"

    for ftime in ${forecast_array}; do
        echo "checking for ${ftime}..."
        file_path="NOTFOUND"
        if ! check_file_without_size_change \
            ${start_time} \
            ${ftime} \
            ${file_path}; then
            we_got_an_error
        fi
        sleep ${sleep_seconds_for_next_time}
        echo "file path: ${file_path}"
        ecflow_client --event modvar_${ftime}
    done

    #------------------
    %include <tail.h>

说明：

- 10-12 行：设置每次检查的间隔时间（``sleep_seconds_for_check``），检查次数上限（``max_check_count``）
- 29-69 行：``check_file_without_size_change`` 函数检测文件是否存在，如果存在检测文件大小是否有变化，如果无变化则返回
- 73-87 行：逐时效检测文件是否存在，如果存在，则设置相应的事件 ``modvar_${ftime}``

运行任务
---------

恢复挂起的 cma_gfs_post 节点，观察任务的启动顺序。