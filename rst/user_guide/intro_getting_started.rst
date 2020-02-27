.. _intro_getting_started:

***************
入门
***************


假设你已经阅读了 :ref:`安装指南<installation_guide>` 安装好了管理节点并且了解了 Ansible 是如何工作的，那么你可以开始基本的Ansbile 入门操作了: 
  * 从 inventory 仓库中选择要执行命令的对象
  * 连接测试这些节点 ( 或者网络设备，或者其它受控节点)connects to those machines (or network devices, or other managed nodes)。 通常使用 SSH 的方式
  * 复制一个或多个模块到远程计算机并尝试执行

Ansible 的实际功能更强大，但开始之前你需要了解他的基础用法才能发掘其它更强大的功能，比如配置、部署、编排等。本页内容会通过简单的 Inventory 配置和 ad-hoc 命令来演示其基本用法。inventory 参考: :ref:`inventory<intro_inventory>`, 更充分了解 Ansible 点击这里 :ref:`playbooks<playbooks_intro>`.

.. contents::
   :local:

选择指定机器执行
====================

Ansible 是通过读取 Inventory 中的配置知道我们要对哪些机器变更。 虽然你可以在命令行使用 ad-hoc 临时命令时指定 IP 地址的方式来控制要操作的对象，但如果想充分使用 Ansible 的灵活性和或扩展性，你必须掌握 Inventory 的配置

行动： 创建基础清单
--------------------------------


创建 ``/etc/ansible/hosts`` 并添加一些主机列表.  使用 IP 地址或者主机名均可: 

.. code-block:: text

   192.0.2.50
   aserver.example.org
   bserver.example.org

基础进阶
-----------------


Inventory 不权可以存放 IPs 和主机名. 也可以创建别名，点击查看 :ref:`aliases<inventory_aliases>`, set variable values for a single host with :ref:`host vars<host_variables>`, or set variable values for multiple hosts with :ref:`group vars<group_variables>`.

.. _remote_connection_information:

连接远程受控节点
==========================

Ansible 和远程主机的通信是通过 `SSH protocol <https://www.ssh.com/ssh/protocol/>`_. 默认 Ansible 使用开源软件 OpenSSh 通过当前用户连接远程主机。


行动： 检查 SSH 连接
----------------------------------


确认通过相同的用户可以连接到所有的受控节点，有必要的话，你可以手动添加公钥到对应主机的``authorized_keys`` 。

基础进阶
------------

有如下几种方式指定用户连接远程受控节点:
  * 在命令行使用 ``-u`` 指定用户
  * 在 Inventory 是指定连接用户
  * 在配置文件中设置连接用户
  * 设置环境变量

点击 :ref:`general_precedence_rules`  查看优化级管理规则。 connection 连接模块，查看更多请点击 :ref:`connections`.


复制和执行模块
=============================

一旦建立连接后， Ansible 会将命令或者 playbook 剧本需要的模块传输到远程主机后，在远程主机上执行命令。

行动： Ansible 初体验
---------------------

使用 ping 模块测试主机存活

.. code-block:: bash

   $ ansible all -m ping

在所有节点上执行一条实时命令:

.. code-block:: bash

   $ ansible all -a "/bin/echo hello"

你运行命令的每台主机应该有类似如下的输出:

.. code-block:: ansible-output

   aserver.example.org | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python"
       },
       "changed": false,
       "ping": "pong"
   }

基础进阶
-----------

Ansbile 默认使用 sftp 传输文件。 如果受控节点不支持 SFTP ，你可以根据文档 :ref:`intro_configuration` 切换成成 SCP 模式。这些文件会临时存放在指定目录下，并在该目录执行这些文件。

如果需要超级权限或者特殊权限，类似 sudo ， 使用 ``become`` 参数指定:

.. code-block:: bash

    # as bruce
    $ ansible all -m ping -u bruce
    # as bruce, sudoing to root (sudo is default method)
    $ ansible all -m ping -u bruce --become
    # as bruce, sudoing to batman
    $ ansible all -m ping -u bruce --become --become-user batman

更新请参考 in :ref:`become`.


恭喜！ 你已经使用 Ansible 打通了所有的主机的奇经八脉。您使用了一个基本清单文件和一个 ad-hoc 临时命令来操作 Ansible 连接到特定的远程节点，并在这个过程中复制模块文件然后执行它，最后返回输出。 您已经拥有一个可以正常运行的 Ansible 基础架构了。

接下来
=====

接下来，你可以了解更多关于 ad-hoc 的使用 :ref:`intro_adhoc`, 探索更多其它模块，更多请参考 :ref:`working_with_playbooks` .  Ansible不仅能运行命令，还包括强大的配置管理和部署功能。


.. seealso::

   :ref:`intro_inventory`
       inventory 详解
   :ref:`intro_adhoc`
       基础命令示范案例
   :ref:`working_with_playbooks`
       深入学习 Ansible 配置管理
   `Mailing List <https://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
