.. _intro_configuration:

*******************
Configuring Ansible
*******************

.. contents:: Topics


本页内容介绍如何配置 Ansible .


.. _the_configuration_file:

配置文件
==================

主要的核心配置都在 (ansible.cfg)， 其中大多数默认配置满足大部分用户的需求，参考文档中列出了搜索配置文件的路径 :ref:`reference documentation<ansible_configuration_settings_locations>`。

.. _getting_the_latest_configuration:

获取最新配置
--------------------------------

如果通过包管理工具安装的，那最新的 ``ansible.cfg`` 默认存放在 ``/etc/ansible`` 中，也有可能因为重复安装或升级，文件是以 ``.rpmnew`` 结尾

如果是通过 pip 或者源码包编译安装 Ansible，你需要手动创建或者覆盖原有配置。

案例参考： `example file is available on GitHub <https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg>`_.

更多更全配置信息可参考：:ref:`configuration_settings<ansible_configuration_settings>`. 从 2.4 版本开始，你可以使用 :ref:`ansible-config` 命令行列出可用选项和变量信息，并且可以检查当前配置。



更详细信息请参考 :ref:`ansible_configuration_settings`.

.. _environmental_configuration:

环境配置
===========================

Ansible 配置同样支持使用系统变量。如果系统环境设置了，会覆盖 Ansible 的配置。

更详细信息请参考 :ref:`ansible_configuration_settings`.


.. _command_line_configuration:

命令行选项
====================

Ansible 并没有把所有的配置都展示在命令中，只是列出最有用或最通用的部分。在命令行指定的配置优先级比从配置文件中读取的优化级要高，也即会覆盖从配置文件中读取的环境变量。

更详细信息请参考 :ref:`ansible-playbook` and :ref:`ansible`.

