.. cemc-ecflow-tutorial documentation master file, created by
   sphinx-quickstart on Sun Jul 24 19:53:09 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

CEMC ecFlow 实战教程
================================================

本教程用于 CEMC 2023 培训的 ecFlow 上机实习环节。

本教程以 **简化版** CMA-GFS 后处理程序为例说明如何使用 ecFlow 运行串行和并行程序。

教程源码：https://github.com/perillaroc/cemc-ecflow-tutorial-2023

在线访问：https://cemc-ecflow-tutorial-2023.readthedocs.io

为什么不用虚拟工作流介绍？
--------------------------

ecFlow 官网教程使用虚拟工作流，用简单的脚本模拟工作流运行情况，比如下面的工作流来自 ecFlow 教程的事件 (event) 章节：

.. code-block::

   |- test
      |- f1
         |- t1
         |- t2
            trigger t1 == complete
            event a
            event b
         |- t3
            trigger t2:a
         |- t4
            trigger t2:b == set

考虑到虚拟工作流与中心使用 ecFlow 的实际情况有所脱节，本次上机实习尝试选择一个简单的实际任务介绍 ecFlow 的用法，期望能给大家留下更直观的印象。
实际上我们不需要使用 ecFlow 提供的全部功能，就可以构建出可以实时运行的流程。

如果大家对使用 ecFlow 感兴趣，可以访问 ECMWF 的官网查看最新教程（介绍最新的 v5 版本）：

https://confluence.ecmwf.int/display/ECFLOW/Tutorial

也可以访问我之前翻译的中文版教程（仅介绍 v4 版本）：

https://perillaroc.github.io/ecflow-tutorial-cn/

为什么选择后处理程序？
--------------------------

目前 CEMC 有 5 个主要数值天气预报模式业务系统，均使用 ecFlow 实时运行并维护：

- CMA-GFS
- CMA-MESO
- CMA-TYM
- CMA-GEPS
- CMA-REPS

因为本轮培训有专题介绍全球同化预报系统 CMA-GFS 和区域预报系统 CMA-MESO 的运行脚本，上机实战环节不再重复介绍。
区域台风预报系统 CMA-TYM 与 CMA-MESO 类似，也不再重复介绍。
集合预报系统 CMA-GEPS 和 CMA-REPS 都有众多集合成员，系统流程比较复杂，不适合初学者。

后处理程序包括了串行程序和并行程序，对于同一个时次，每个时效的后处理任务都可以独立运行，可以充分体现 ecFlow 相对于 SHELL 脚本在流程并发上的优势。

因此，本次上机培训选择 CMA-GFS 后处理任务作为示例讲解如何使用 ecFlow 搭建业务流程。

教程内容
---------

本教程由三部分组成：

- **开始使用**：介绍如何启动 ecFlow 服务，并创建只有一个串行任务的工作流，生成后处理需要的参数文件。
- **构建单个流程**：添加一个并行任务，运行后处理程序，介绍如何使用触发器设定运行流程，以及如何将作业提交到 Slurm 队列中。
- **构建多时效流程**：为一个时次的全部时效创建后处理任务，介绍如何组织多个任务并限制同时运行的任务数量。
- **进阶**：添加文件检测任务和日期循环变量，形成一个可以滚动运行的后处理工作流。

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: 开始使用

   started/introduction
   started/prepare
   started/start-ecflow-server
   started/create-a-suite
   started/load-the-suite
   started/create-includes
   started/create-task-script

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: 构建单个流程

   single-hour/add-another-task
   single-hour/add-family
   single-hour/add-more-tasks
   single-hour/use-script-to-ignore-task
   single-hour/create-parallel-task

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: 进阶

   multi-hour/create-model-task
   multi-hour/create-post-tasks
   multi-hour/create-archive-task
   multi-hour/run-the-flow