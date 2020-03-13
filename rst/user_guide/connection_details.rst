.. _connections:

******************************
连接方法和详细信息
******************************

本节我们来介绍如何扩展和完善 `Ansible` 的连接方法。

持久连接和 paramiko 模块
---------------------------

默认情况下， `Ansible` 使用本地 `OpenSSh`，因为 `OpenSSH` 支持长久连接(一种性能特征)， Kerberos 和 ``~/.ssh/config`` 的一些选项配置，比如：主机跳转设置。 如果您的管理机使用的是不支持 ``ControlPersist`` 的较旧版本 ``OpenSSH``，``Ansible`` 将回退使用 ``OpenSSH`` ``paramiko``连接模块。

.. _connection_set_user:

设置远程连接用户
---------------------

默认情况下，Ansible 使用控制节点当前使用的用户连接远程设备。 如果该用户名在远程设备上不存在，则可以为连接设置其他用户名。设置远程连接用户请参考 :ref:`become`。 同样，你也可以在 playbook 中设置:

.. code-block:: yaml

   ---
   - name: update webservers
     hosts: webservers
     remote_user: admin

     tasks:
     - name: thing to do first in this playbook
     . . .

在 Inventory 中设置主机变量的方式也可以实现:

.. code-block:: text

   other1.example.com     ansible_connection=ssh        ansible_user=myuser
   other2.example.com     ansible_connection=ssh        ansible_user=myotheruser

在 Inventory 中设置组变量:

.. code-block:: yaml

    cloud:
      hosts:
        cloud1: my_backup.cloud.com
        cloud2: my_backup2.cloud.com
      vars:
        ansible_user: admin

设置 SSH 公私钥
----------------------

默认， Ansible 假定你使用 SSH 密钥连接远程主机。 推荐使用 SSH key的方式，但如果你想使用密码认证的话，请使用 ``--ask-pass`` 选项。 如果需要输出密码，使用 ``--ask-become-pass`` ，详见 :ref:`privilege escalation <become>` (sudo, pbrun, etc.)

.. include:: shared_snippets/SSH_password_prompt.txt


添加互信，可以避免每次 SSH 连接时输入密码:

.. code-block:: bash

   $ ssh-agent bash
   $ ssh-add ~/.ssh/id_rsa

具体视你的配置情况，你可以使用 ``--private-key`` 在命令行指定 pem 私钥文件。 好可以添加私钥文件:

.. code-block:: bash

   $ ssh-agent bash
   $ ssh-add ~/.ssh/keypair.pem

另外一种不使用 ``ssh-agent`` 添加私钥认证的方式是编辑 inventory 文件，指定 ``ansible_ssh_private_key_file`` ，具体参考 :ref:`intro_inventory`.

在本机 localhost 运行
-------------------------

你可以直接使用 "localhost" or "127.0.0.1"  指定在本机运行:

.. code-block:: bash

    $ ansible localhost -m ping -e 'ansible_python_interpreter="/usr/bin/env python"'

你可以将 localhost 添加到 Inventory 文件中显示指定:


.. code-block:: bash

    localhost ansible_connection=local ansible_python_interpreter="/usr/bin/env python"

.. _host_key_checking_on:

首次互信认证提示
--------------------------

默认 Ansible 开启认证，即首次认证时会有提示。 检查主机密钥可以防止服务器欺骗和中间人攻击，但管理起来确实会比较麻烦。

如果一台主机被重装并且和原来的私钥不一样，那将无法连接。如果是一台新主机，连接时会提醒你验证密钥，这种交互的方式在使用 Ansible 时非常不方便，尤其是配合 cron 使用时。你可能不会喜欢.

如果你是专业人士明白关掉其它提醒背后的意义及安全风险，你可以编辑 ``/etc/ansible/ansible.cfg`` or ``~/.ansible.cfg``:

.. code-block:: text

    [defaults]
    host_key_checking = False


或者设置变量 :envvar:`ANSIBLE_HOST_KEY_CHECKING` :

.. code-block:: bash

    $ export ANSIBLE_HOST_KEY_CHECKING=False

另请注意，使用 paramiko 模式检查密钥的速度很慢。因此，使用此功能时也建议切换到 `ssh` 模式。

其它连接方式
------------------------

Ansible 除了 SSH 外还有很多其它连接方法。 我们可以选择任何连接插件，包括在本地管理以及管理chroot，lxc 和jail容器。 Ansible 还有一种反转系统的连接方式: 'ansible-pull'。 该方式通过调度 git checkout 从中央仓库拉取配置来使系统处于'phone home' 「随时待命？」的状态。
