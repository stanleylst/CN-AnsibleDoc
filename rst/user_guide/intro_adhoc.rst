.. _intro_adhoc:

********************
ad-hoc 命令操作指引
********************

Ansible ad-hoc 临时命令工具使用命令行工具 `/usr/bin/ansible` 来对一台或多台受控节点变更。 Ad-hoc 命令快捷方便，但不可复用。那为何还要学习它？ Ad-hoc 能很好的展示 Ansible 的简单和强大。Ansible 的语法和 Ansible-playbook 的兼容。阅读和运行这些案例前，请详细阅读 :ref:`intro_inventory`.

.. contents::
   :local:

为什么使用 ad-hoc 临时命令?
==========================

Ad-hoc 非常适合临时命令，即很少重复执行的命令。比如：如果您想在圣诞节假期关闭实验室中所有机器的电源，您无需编写剧本，简单一条 Ansible 命令即可。 Ad-hoc 的命令形式如下：

.. code-block:: bash

    $ ansible [pattern] -m [module] -a "[module options]"


更多请查看 :ref:`patterns<intro_patterns>` and :ref:`modules<working_with_modules>` 。

Ad-hoc 命令案例
===============

`Ad-hoc` 命令可以被用作重启服务器、复制文件、包管理和用户管理等等。可以使用任何 `Ansible module`。 `Ad-hoc` 和 `playbook `一样，使用声明式事务模型，该模式具有冥等特性，即在任务开始执行前会先检查，确认执行前后的状态是否有变化，有不同才会执行，否则不做变更。

重启服务器
----------

 ``ansible`` 命令行工具的默认模块是 :ref:`command module<command_module>`. 你可以使用 `Ad-hoc` 命令调用 `command` 一次10台重启 Atlanta 机房的所有服务器。 开始之前，你需要把 Atlanta 的所有服务器添加到`Inventory` 组  [atlanta] 中。 并且你已经添加 SSH 信任。 重启 `[atlanta]` 中的所有服务器：

.. code-block:: bash

    $ ansible atlanta -a "/sbin/reboot"

`Ansible` 默认开启 5 个并发线程。如果你有机器多于你开启的线程数，时间会长一点。 开启 10 个并发线程的命令如下：

.. code-block:: bash

    $ ansible atlanta -a "/sbin/reboot" -f 10

默认使用你当前用户连接远程主机。使用不同用户连接：

.. code-block:: bash

    $ ansible atlanta -a "/sbin/reboot" -f 10 -u username

重启可能需要提权到更高权限，你可以连接服务器使用  ``username`` ，执行命令使用 ``root`` 权限，但需要使用模块 :ref:`become <become>` :

.. code-block:: bash

    $ ansible atlanta -a "/sbin/reboot" -f 10 -u username --become [--ask-become-pass]

如果你添加 ``--ask-become-pass`` or ``-K``, Ansible 会提示你使用密码提权 (sudo/su/pfexec/doas/etc).

.. note::

   :ref:`command module <command_module>`  不支持 ``shell`` 扩展语法，比如管道和重定向（但是 shell 变量支持的）。如果你执行的命令中需要用于该情况，建议你使用 `shell` 模块。更多请查看 :ref:`working_with_modules` 

目前为止我们介绍的模块都是 'command' ，使用其它模块 ``-m`` 指定，比如使用 :ref:`shell module <shell_module>`:

.. code-block:: bash

    $ ansible raleigh -m shell -a 'echo $TERM'

当执行 `Ansible` *ad hoc* CLI 的任何命令时，请尤其要注意 shell 的引用规则，本地 shell 会保留变量并传递给 Ansible。 比如，如上案例如果使用的是双引号，则会输出 ``$TERM`` 的变量值，而非 `$TERM` 字符串本身。

.. _file_transfer:

文件管理
--------

``Ad-hoc`` 可以使用 ``Ansible`` 和 ``SCP`` 并发传输多个文件至多台主机。传输文件至 ``[atlanta]`` 组的所主机:

.. code-block:: bash

    $ ansible atlanta -m copy -a "src=/etc/hosts dest=/tmp/hosts"


如果这条命令以后会经常用到, 请使用 ``playbook`` 模块  :ref:`template<template_module>` 。

The :ref:`file<file_module>` 模块允许更新文件属性及权限。也可以在运行 ``copy`` 模块时指定:

.. code-block:: bash

    $ ansible webservers -m file -a "dest=/srv/foo/a.txt mode=600"
    $ ansible webservers -m file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"


``file`` 模块也可以创建目录，类似 ``mkdir -p``:

.. code-block:: bash

    $ ansible webservers -m file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"

删除和递归删除好同样：

.. code-block:: bash

    $ ansible webservers -m file -a "dest=/path/to/c state=absent"

.. _managing_packages:

包管理
-------

``ad-hoc``  像``yum`` 一样进行包安装、升级、删除包。确保包已经安装但不升级:

.. code-block:: bash

    $ ansible webservers -m yum -a "name=acme state=present"

指定版本安装包:

.. code-block:: bash

    $ ansible webservers -m yum -a "name=acme-1.5 state=present"

确保安装的版本是最新版:

.. code-block:: bash

    $ ansible webservers -m yum -a "name=acme state=latest"

删除包:

.. code-block:: bash

    $ ansible webservers -m yum -a "name=acme state=absent"

Ansible 包管理模块可以在多个平台工作。如果合适你平台的包管理模块，你可以使用 ``command`` 模块或写一个新的包管理模块。

.. _users_and_groups:

用户和组管理
------------

使用 ``ad-hoc``创建、管理、删除用户:

.. code-block:: bash

    $ ansible all -m user -a "name=foo password=<crypted password here>"

    $ ansible all -m user -a "name=foo state=absent"


查看更多 :ref:`user <user_module>` , 还有 group 组和 group 成员管理。

.. _managing_services:

服务进程管理
------------

确保服务已安装:

.. code-block:: bash

    $ ansible webservers -m service -a "name=httpd state=started"


或者，在所有Web服务器上重新启动服务：

.. code-block:: bash

    $ ansible webservers -m service -a "name=httpd state=restarted"

确保服务被关闭:

.. code-block:: bash

    $ ansible webservers -m service -a "name=httpd state=stopped"

.. _gathering_facts:

收集主机信息
------------

``Facts`` 是收集的所有系统变量。你可以使用 ``Facts`` 有条件地执行任务，也可以仅获取有关系统的临时信息。

.. code-block:: bash

    $ ansible all -m setup

同样，你可以指定某个选项的变量。具体参考 :ref:`setup <setup_module>` 。

如上内容为 ``Ansible`` 执行的基本要素，现在的你已经具备学习自动化重复性任务的条件了 :ref:`Ansible Playbooks <playbooks_intro>`

.. seealso::

   :ref:`intro_configuration`
       关于 ``Ansible`` 配置文件的所有信息
   :ref:`all_modules`
       可用模块列表
   :ref:`working_with_playbooks`
       使用 ``Ansible`` 进行配置管理和部署
   `Mailing List <https://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
