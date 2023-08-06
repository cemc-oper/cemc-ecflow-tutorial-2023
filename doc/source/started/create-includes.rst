创建头文件
==========

在编写任务脚本之前，我们先创建一些头文件 (include files)，包括每个 ecFlow 任务都必须使用的两个头文件 **head.h** 和 **tail.h**。

在 ``${TUTORIAL_HOME}/def`` 目录下创建两个子目录：

.. code-block:: bash

    cd ${TUTORIAL_HOME}/def
    mkdir -p include
    mkdir -p ecffiles
    cd include

head.h
-----------

**head.h** 放在 ecf 脚本开头，完成如下任务：

- 准备与 ecFlow 服务通信的环境变量
- 定义脚本错误处理函数。当错误发生，通知服务该任务进入 *aborted* 状态
- 使用 ``ecflow_client`` 命令通知服务该任务已经开始

在 ``${TUTORIAL_HOME}/def/include`` 中创建头文件 **head.h**：

.. code-block:: bash

    #!/bin/ksh

    set -e          # stop the shell on first error
    set -u          # fail when using an undefined variable
    set -x          # echo script lines as they are executed
    # set -o pipefail # fail if last(rightmost) command exits with a non-zero status

    date

    # Defines the three variables that are needed for any
    # communication with ecFlow
    export ECF_PORT=%ECF_PORT%    # The server port number
    export ECF_HOST=%ECF_HOST%
    export ECF_NODE=%ECF_HOST%    # The host name where the server is running
    export ECF_NAME=%ECF_NAME%    # The name of this current task
    export ECF_PASS=%ECF_PASS%    # A unique password
    export ECF_TRYNO=%ECF_TRYNO%  # Current try number of the task

    . /etc/profile.d/modules.sh
    PATH=$PATH:$HOME/bin

    # Tell ecFlow we have stated
    # The ecFlow variable ECF_RID will be set to parameter of smsinit
    # Here we give the current PID.
    RID=$( echo ${SLURM_JOB_ID:-0.0} )
    if [ $RID -eq 0 ] ; then
      RID=$$
    fi
    export ECF_RID=$RID

    # Tell ecFlow we have started
    ecflow_client --init=$RID

    # Define a error handler
    ERROR() {
       set +e                      # Clear -e flag, so we don't fail
       wait                        # wait for background process to stop
       ecflow_client --abort=trap  # Notify ecFlow that something went wrong, using 'trap' as the reason
       trap 0                      # Remove the trap
       exit 0                      # End the script
    }
    # Trap any calls to exit and errors caught by the -e flag

    trap ERROR 0

    # Trap any signal that may cause the script to fail
    trap '{ echo "Killed by a signal";trap 0;ERROR; }' 1 2 3 4 5 6 7 8 10 12 13 15
    echo "exec on hostname:" $(hostname)



tail.h
-----------

**tail.h** 放在 ecf 脚本结尾，通知服务该任务已经完成。

在 ``${TUTORIAL_HOME}/def/include`` 中创建头文件 **tail.h**：

.. code-block::

    date
    wait                      # wait for background process to stop
    ecflow_client --complete  # Notify ecFlow of a normal end
    trap 0                    # Remove all traps
    exit 0                    # End the shell

    # slsubmit6 %ECF_JOB% %ECF_NAME% %ECF_TRIES% %ECF_TRYNO% %ECF_HOST% %ECF_PORT%
    # slcancel4 %ECF_RID% %ECF_NAME% %ECF_HOST% %ECF_PORT%


configure.h
----------------

**configure.h** 用于设置模式参数和目录。

在 ``${TUTORIAL_HOME}/def/include`` 中创建头文件 **configure.h**：

.. code-block:: bash

    #------------
    # START: configure.h
    #------------

    PROGRAM_BASE_DIR=%PROGRAM_BASE_DIR%
    RUN_BASE_DIR=%RUN_BASE_DIR%
    ECF_DATE=%ECF_DATE%
    HH=%HH%

    START_TIME=${ECF_DATE}${HH}

    COMPONENT_PROJECT_BASE=${PROGRAM_BASE_DIR}
    CYCLE_RUN_BASE_DIR=${RUN_BASE_DIR}/${START_TIME}

    PROGRAM_BIN_DIR=${COMPONENT_PROJECT_BASE}/bin
    PROGRAM_CON_DIR=${COMPONENT_PROJECT_BASE}/condat
    PROGRAM_SCRIPT_DIR=${COMPONENT_PROJECT_BASE}/script

    export PATH=${PROGRAM_BIN_DIR}:${PROGRAM_SCRIPT_DIR}:${PATH}

    #------------
    # END: configure.h
    #------------
