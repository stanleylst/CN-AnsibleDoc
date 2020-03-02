.. _intro_inventory:
.. _inventory:

***************************
Inventory 使用进阶
***************************

Ansible 从 Inventory 读取列表或组，可同时并发操作这些受控节点或主机。 一旦 inventory 被定义，你就可以使用正则匹配主机或者组来指定要运行的主机列表 :ref:`patterns <intro_patterns>` 。

Inventory 主机清单存放在 ``/etc/ansible/hosts``。 你可以在命令行使用 ``-i <path>`` 指定特定的 inventory 清单。 当然，你可以一次指定多个 inventory 清单，也可以使用 pull inventory 的动态获取或者从云主机获取。具体参考  :ref:`intro_dynamic_inventory`.

Ansible 从 2.4 开始， 使用 :ref:`inventory_plugins` 的方式使 inventory 清单更灵活

.. contents::
   :local:

.. _inventoryformat:

Inventory 基础: formats, hosts, and groups
============================================

Inventory 文件可以有多种格式，取决于你使用什么插件，最常用的格式是 YAML 和 INI。 下面是救命:

.. code-block:: text

    mail.example.com

    [webservers]
    foo.example.com
    bar.example.com

    [dbservers]
    one.example.com
    two.example.com
    three.example.com



括号中的标题是组名，用于对主机进行分类，用于确定什么时间、什么目的、相对哪些主机做什么事情

如下为 YAML 格式的示例:

.. code-block:: yaml

  all:
    hosts:
      mail.example.com:
    children:
      webservers:
        hosts:
          foo.example.com:
          bar.example.com:
      dbservers:
        hosts:
          one.example.com:
          two.example.com:
          three.example.com:

.. _default_groups:

默认组 ( Default groups )
--------------------------

默认有两个分组： ``all`` and ``ungrouped`` 。 ``all`` 组顾名思义包括所有主机。 ``ungrouped`` 则是 ``all`` 组之外所有主机。所有的主机要不属于 ``all`` 组，要不就属于 ``ungrouped`` 组。

尽管 ``all`` 和 ``ungrouped`` 始终存在，但它们以隐式的方式出现，而不出现在诸如 ``group_names`` 的组列表中。


.. _host_multiple_groups:

多主机组
----------

你可以把一台主机放在多个组中。 比如： 亚特兰大 (Atlanta) 数据中心中的生产(prod) 的 webserver 服务器可能包含在 [prod] 和 [atlanta]和 [webservers] 组中。你可以参考如下思路创建组:

* What - 是什么？一个应用，栈，还是微服务. (例如: database servers, web servers, etc).
* Where - 在哪里？ 数据中心？某个地区？本地 DNS? 还是一个存储  (例如, east, west).
* When - 何时？ 开发阶段、生产阶段？(例如： prod, test).

扩展以前的 YAML 清单，加入如上示例中 (what, when, and where) 的示例：

.. code-block:: yaml

  all:
    hosts:
      mail.example.com:
    children:
      webservers:
        hosts:
          foo.example.com:
          bar.example.com:
      dbservers:
        hosts:
          one.example.com:
          two.example.com:
          three.example.com:
      east:
        hosts:
          foo.example.com:
          one.example.com:
          two.example.com:
      west:
        hosts:
          bar.example.com:
          three.example.com:
      prod:
        hosts:
          foo.example.com:
          one.example.com:
          two.example.com:
      test:
        hosts:
          bar.example.com:
          three.example.com:

可以看到 ``one.example.com`` 同时存在 ``dbservers``, ``east``, and ``prod`` 组中
您还可以使用嵌套组来简化此清单中的 ``prod`` and ``test`` 组，优化后结果如下：


.. code-block:: yaml

  all:
    hosts:
      mail.example.com:
    children:
      webservers:
        hosts:
          foo.example.com:
          bar.example.com:
      dbservers:
        hosts:
          one.example.com:
          two.example.com:
          three.example.com:
      east:
        hosts:
          foo.example.com:
          one.example.com:
          two.example.com:
      west:
        hosts:
          bar.example.com:
          three.example.com:
      prod:
        children:
          east:
      test:
        children:
          west:

你可以找到更多编排 inventory 的安全 :ref:`inventory_setup_examples`.

增加主机段
--------------

如果您有许多具有相似模式的主机，则可以将它们添加为一个范围，而不必分别列出每个主机名：


In INI:

.. code-block:: text

    [webservers]
    www[01:50].example.com

In YAML:

.. code-block:: yaml

    ...
      webservers:
        hosts:
          www[01:50].example.com:



对于数字匹配 [0-9], 也支持字母正则 [a-z]：


.. code-block:: text

    [databases]
    db-[a:f].example.com

Inventory 变量定义
=============================

你可以直接在 Inventory 清单中定义的 host 或 group 变量。刚开始的时候，你可以直接添加 host 或 group 到 Inventory 文件中。当你越加越多的时候，你可能会考虑将变量和 host group 分离成独立的文件。

.. _host_variables:

给单台主机设置变量 : host variables
======================================

在 Playbook 中的示例 INI 文件的写法：

.. code-block:: text

   [atlanta]
   host1 http_port=80 maxRequestsPerChild=808
   host2 http_port=303 maxRequestsPerChild=909

YAML:

.. code-block:: yaml

    atlanta:
      host1:
        http_port: 80
        maxRequestsPerChild: 808
      host2:
        http_port: 303
        maxRequestsPerChild: 909

比如给 host 添加非标准 SSH 端口，把端口直接添加到主机名后，心冒号分隔即可:

.. code-block:: text

    badwolf.example.com:5309

connection 连接远程主机的方式:

.. code-block:: text

   [targets]

   localhost              ansible_connection=local
   other1.example.com     ansible_connection=ssh        ansible_user=myuser
   other2.example.com     ansible_connection=ssh        ansible_user=myotheruser

.. note::  如果你在 SSH 配置文件中定义了非标端口，使用 ``openssh`` 连接主机可以默认读取不用指定特定端口， 但 ``paramiko`` 就不能自动发现了。

.. _inventory_aliases:

Inventory aliases
-----------------

在 Inventory 中定义别名：

In INI 中定义:

.. code-block:: text

    jumper ansible_port=5555 ansible_host=192.0.2.50

在 YAML 中定义:

.. code-block:: yaml

    ...
      hosts:
        jumper:
          ansible_port: 5555
          ansible_host: 192.0.2.50

在如上的示例中， 执行 Ansible 对 ""jumper"" 主机执行命令时，会连接 192.0.2.50 的 5555 端口。 这种方式仅适用于通过静态 IP 的主机，或者通过隧道连接的主机。

.. note::

   INI 格式定义的  ``key=value`` 声明的位置不同所代表的含意也不同:
   
   * 当在主机同行声明时， INI 值将会被解释了 Python 内置结构，（ strings, numbers, tuples, lists, dicts, booleans, None ）。 在同一行中主机可以接受多个 ``key=value`` 参数， 因此，他们需要一种方法来指示空格是值的一部分而不是分隔符。

   * 当声明 ``:vars`` 段落声明时， INI 的值会被解析为字符串。 比如 ``var=FALSE`` 的 'FALSE' 就是字符串。 和 host lines 定义变量的方式不同， ``:vars`` 段落只接受一个条目，所以 ``=`` 后面所有的内容都是变量的值。

   * 如果 INI Inventory 的变量定义必须是一个确切的类型 ( a string or a boolean value )，请在 task 中使用过滤器指定类型，请勿依赖 INI Inventory 中定义的类型。

   * 建议使用 YAML 格式定义 Inventory 清单， YAML 格式可以很好的避免变量混淆的情况。 YAML 插件可以保证变量的一致性和正确性。


通常来讲，使用这种方式定义系统策略变量并不完美。 在主配置文件 Inventory 中设置变量只是一种速记的临时方式。 可以参考 :ref:`splitting_out_vars` 如果在 'host_vars' 目录中定义使用独立的变量文件设置变量。

.. _group_variables:

给多台主机设置变量 : group variables
======================================================

如果组中的所有主机共享一个变量值，则可以一次将该变量应用于整个组， INI 格式：

.. code-block:: text

   [atlanta]
   host1
   host2

   [atlanta:vars]
   ntp_server=ntp.atlanta.example.com
   proxy=proxy.atlanta.example.com

In YAML:

.. code-block:: yaml

    atlanta:
      hosts:
        host1:
        host2:
      vars:
        ntp_server: ntp.atlanta.example.com
        proxy: proxy.atlanta.example.com


组变量是一次将变量同时应用于多个主机的便捷方法。 但是，在执行之前，Ansible始终将变量（包括 Inventory 清单变量）展平到主机级别。 如果该主机是多个组的成员，则 Ansible 将从所有这些组中读取变量值。 如果同一主机在不同的组中被赋予不同的变量值，则 Ansible 会根据内部规则来选择要使用的值。 具体合并值的方式请参考 :ref:`rules for merging <how_we_merge>` 。        


.. _subgroups:

继承变量值：嵌套组的组变量设置

----------------------------------------------------------------

您可以设置一个 children 的组变量， ``:children`` INI 格式或 ``children:``  YAML 格式，
您可以分别使用 ``:vars`` or ``vars:`` 给组定义变量

INI 格式:

.. code-block:: text

   [atlanta]
   host1
   host2

   [raleigh]
   host2
   host3

   [southeast:children]
   atlanta
   raleigh

   [southeast:vars]
   some_server=foo.southeast.example.com
   halon_system_timeout=30
   self_destruct_countdown=60
   escape_pods=2

   [usa:children]
   southeast
   northeast
   southwest
   northwest

YAML 格式:

.. code-block:: yaml

  all:
    children:
      usa:
        children:
          southeast:
            children:
              atlanta:
                hosts:
                  host1:
                  host2:
              raleigh:
                hosts:
                  host2:
                  host3:
            vars:
              some_server: foo.southeast.example.com
              halon_system_timeout: 30
              self_destruct_countdown: 60
              escape_pods: 2
          northeast:
          northwest:
          southwest:


如果您需要存储列表或哈希数据，或者更喜欢将主机和组特定变量与清单文件分开, 请参考 :ref:`splitting_out_vars`
子组有几个要注意的属性：


 - 子组成员默认属于父组成员
 - 子组的变量比父组的变量优先级高，即值会覆盖父组的变量。
 - 组可以有多个父组或孩子，但不能循环关系。
 - 主机也可以隶属于多个组中，但是只有 **一个** 主机实例，可以合并多个组中的数据。

.. _splitting_out_vars:

编排主机和组变量
=====================

尽管你可以将变量存储在 Inventory 主清单文件中，但是将变量存储在单独的主机变量和组变量文件中，可以帮助您更轻松地组织变量值。 主机和组变量文件必须使用YAML语法。 文件可以以'.yml', '.yaml', '.json' 结尾，甚至没有扩展结尾。更多请参考 :ref:`yaml_syntax`。

Ansible 通过搜索对应的 Inventory 清单文件或 Playbook 剧本文件来加载主机和组变量文件。如果你的清单文件 ``/etc/ansible/hosts`` 主机 'foosball' 同时属于 'raleigh' 和 'webservers' 组， 该主机使用的变量可能包含在如下这些路径: 

.. code-block:: bash

    /etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
    /etc/ansible/group_vars/webservers
    /etc/ansible/host_vars/foosball


举例， 如果您按数据中心分组主机，并且每个数据中心使用其自己的NTP服务器和数据库服务器，则可以创建一个名为 ``/etc/ansible/group_vars/raleigh`` 的文件来存储 ``raleigh`` 的变量组: 

.. code-block:: yaml

    ---
    ntp_server: acme.example.org
    database_server: storage.example.org

你可以在 hosts 或者 groups 后创建 *directories* . Ansible 将按顺读取这些目录中的所有文件。举例 'raleigh' 组:

.. code-block:: bash

    /etc/ansible/group_vars/raleigh/db_settings
    /etc/ansible/group_vars/raleigh/cluster_settings

'raleigh' 组中所有主机的变量在这些文件中都有效。 这种方式非常适合组织变量比较多的场景或者原来的文件非常大，更多组变量请参考 :ref:`Ansible Vault<playbooks_vault>` 。

你也可以新建 ``group_vars/``  和 ``host_vars/`` 目录到 plyabook 同级目录下。 ``ansible-playbook`` 会默认查找当前工作目录。 其它类似 ``ansible``, ``ansible-console`` 等命令默认从查找 ``group_vars/`` and ``host_vars/`` 目录。如果希望其他命令从 playbook 目录中加载组和主机变量，则必须在命令行上提供 ``--playbook-dir`` 选项。

如果您同时从 Playbook 目录和 Inventory 清单目录中加载清单文件，则 Playbook 目录中的变量将覆盖在清单目录中设置的变量。

将 Inventory 清单文件和变量保存在git repo（或其他版本控制系统）中是跟踪库存和主机变量更改的绝佳方法。

.. _how_we_merge:

变量合并
=============

默认情况下，在运行之前变量会合并/展开到特定目标主机。这种方式使 Ansible 始终专注于主机和任务，因此组的定义是完全依赖 Inventory 和 host。 默认， Ansible 会按序覆盖重复变量，无论是 组变量还是主机变量。 ( 请参考 :ref:`DEFAULT_HASH_BEHAVIOUR<DEFAULT_HASH_BEHAVIOUR>` )。 优先顺序是（从最低到最高）：

- all group (because it is the 'parent' of all other groups)
- parent group
- child group
- host


默认，Ansible 按字母顺序合并处于相同父/子级别的组，后加载的组将覆盖先前的组。 在下面的示例中， a_group 将与 b_group 组合并，并且匹配的 b_group 组覆盖 a_group 。

您可以通过设置组变量 ``ansible_group_priority`` 来更改此行为，以更改同一级别的组的合并顺序（在解决父/子顺序之后）。 数字越大，合并的时间越晚，优先级越高。 如果未设置，则此变量默认为“ 1”。 示例：

.. code-block:: yaml

    a_group:
        testvar: a
        ansible_group_priority: 10
    b_group:
        testvar: b
 
在这个示例中，如果两个组的优先级相同，则最终的结果是 ``testvar == b``， 但是如果我们设置 ``a_group`` 的优先级为 10 ，则最终结果为 ``testvar == a``

.. note:: ``ansible_group_priority`` 只在 Inventory 中有效，在 group_vars/ 无效， 因为该变量用于加载 group_vars 来设置变量变量优先级。

.. _using_multiple_inventory_sources:

使用多个 Inventory
================================

你可以命令行提供多个 Inventory 选项或者配置 :envvar:`ANSIBLE_INVENTORY` 的方式，同时使用多个 Inventory 源 ( 目录， 动态 Inveory 脚本 或者 Inventory 插件提供的文件 )。 该功能针对相互独立的环境非常有帮助，比如你想同时对测试环境和生产环境执行某操作。

同时使用两个源的命令执行方式如下:


.. code-block:: bash

    ansible-playbook get_logs.yml -i staging -i production


注意，如果变量冲突，冲突解决方案请参考 :ref:`how_we_merge` and :ref:`ansible_variable_precedence`。 变量合并顺序由 Inventory 输入顺序决定。
如果 ``[all:vars]``  在 staging inventory 定义为 ``myvar = 1``, 但是在 production inventory 定义为 ``myvar = 2``,
最终的结果为 ``myvar = 2``.  同样，结果会反转如果改变参数输入顺序 ``-i production -i staging``。

**合并目录下的所有 Inventory**

您还可以合并组合目录下的多个 Inventory 清单和不同类型的 Inventory 来创建新清单。这对于组合静态和动态主机并将它们作为一个 Inventory 清单进行管理很有用。以下 Inventory 清单结合了清单插件源，动态清单脚本，和带有静态主机的文件: 

.. code-block:: text

    inventory/
      openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
      dynamic-inventory.py   # add additional hosts with dynamic inventory script
      static-inventory       # add static hosts and groups
      group_vars/
        all.yml              # assign variables to all hosts

您可以像下面这样指定一个 Inventory 清单目录:

.. code-block:: bash

    ansible-playbook example.yml -i inventory

如果存在与其他库存来源之间的变量冲突或组依赖关系，则控制库存来源的合并顺序可能很有用。 合并根据文件名按字母顺序合并，因此可以通过在文件前添加数字前缀来控制结果: 

.. code-block:: text

    inventory/
      01-openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
      02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
      03-static-inventory       # add static hosts
      group_vars/
        all.yml                 # assign variables to all hosts


如果 ``01-openstack.yml`` 在  ``all`` 中声明 ``myvar = 1`` , ``02-dynamic-inventory.py`` 声明 ``myvar = 2``,
 ``03-static-inventory`` 声明 ``myvar = 3``, playbook 最终会返回结果  ``myvar = 3``.

关于 Inventory 插件和动态 Inventory 脚本的更多信息请参考 :ref:`inventory_plugins` and :ref:`intro_dynamic_inventory`.

.. _behavioral_parameters:

主机连接: Inventory 参数设置
====================================================

综上所述, 设置以下变量可控制Ansible与远程主机的交互方式。

connection 模式:

.. include:: shared_snippets/SSH_password_prompt.txt

ansible_connection
    连接受控主机的方式. 填写 ansible 连接的插件名字，如 ``smart``, ``ssh`` or ``paramiko``.  默认： smart. 下一节将介绍基于非SSH的类型。

常用的连接参数有如下这些:

ansible_host
    要连接的主机名，如果与您要给它提供的别名不同。
ansible_port
    连接端口，默认22
ansible_user
    连接远程主机的用户
ansible_password
    认证密码( 切勿明文保存在文本中，使用加密的方式保存。 详见 :ref:`best_practices_for_variables_and_vaults`)


SSH 连接选项:

ansible_ssh_private_key_file
    指定 ssh 私钥。如果没有私钥或者有多个私钥时有用
ansible_ssh_common_args
    这个设置通常添加在默认命令行 :command:`sftp`, :command:`scp` and :command:`ssh` 之后。 当为一台主机或组配置 ``ProxyCommand`` 时有用。
ansible_sftp_extra_args
    此设置始终附加在默认的 `sftp` 命令行中。
ansible_scp_extra_args
    此设置始终附加在默认的 `scp` 命令行中。
ansible_ssh_extra_args
    This setting is always appended to the default :command:`ssh` command line.
    此设置始终附加在默认的 `ssh` 命令行中。
ansible_ssh_pipelining
    设置是否使用 SSH 管道，可以在  :file:`ansible.cfg` 设置
ansible_ssh_executable (added in version 2.2)
    此设置将覆盖默认行为以使用系统 :command:`ssh`。 这样会覆盖 :file:`ansible.cfg` 文件中的 ``ssh_executable`` 设置



提权设置 ( 更多请参考 :ref:`Ansible Privilege Escalation<become>` ):

ansible_become
    等同 ``ansible_sudo`` or ``ansible_su``,  允许强制特权升级
ansible_become_method
    允许设置权限提升方法
ansible_become_user
    等同 ``ansible_sudo_user`` or ``ansible_su_user``, 允许设置通过特权升级成为的用户
ansible_become_password
    等同 ``ansible_sudo_password`` or ``ansible_su_password``, 允许您设置特权升级密码。 ( 切勿存储明文密码，请务必使用加密的方式 。详见 :ref:`best_practices_for_variables_and_vaults`)
ansible_become_exe
    等同 to ``ansible_sudo_exe`` or ``ansible_su_exe``, 允许您为所选的升级方法设置可执行文件
ansible_become_flags
    等同 ``ansible_sudo_flags`` or ``ansible_su_flags``, 允许您设置传递给所选升级方法的标志。 也可以在中 :file:`ansible.cfg` 全局设置 ``sudo_flags`` 

远程主机环境变量选项:

.. _ansible_shell_type:

ansible_shell_type
    指定远程主机使用的 Shell。 在使用该选项前一定要先将 :ref:`ansible_shell_executable<ansible_shell_executable>` 设置为 non-Bourne (sh) 。 默认命令使用 ``sh``. 设置 ``csh`` or ``fish`` 将会在远程主机上使用 ``csh`` ``fish``，而非默认的 ``sh``

.. _ansible_python_interpreter:

ansible_python_interpreter
    目标主机 python 目录。 对于一台主机上有多个 Python 环境或者默认路径不是 :command:`/usr/bin/python` 的 \*BSD 环境，或者  where :command:`/usr/bin/python` 的不是 2.X 系统的 Python 情况有用。我们不使用:command:`/usr/bin/env` 命令机制，因为这需要设置远程用户的路径，并且假定 python 可执行文件名为 python ，其中可执行文件可能命名为像 python2.6 一样的程序。

ansible_*_interpreter
    适用于 ruby or perl 等类型 :ref:`ansible_python_interpreter<ansible_python_interpreter>` 环境。这将替换运行模块在远程主机上的 shabang.

.. 2.1 版本开始的新功能:: 2.1

.. _ansible_shell_executable:

ansible_shell_executable
    设置远程主机使用何种 shell，默认  :command:`/bin/sh`，会覆盖 ``executable`` in :file:`ansible.cfg`。 如果远程主机没有安装 :command:`/bin/sh` ，则需要修改下了。 ( 比如： :command:`/bin/sh` 在远程主机没有安装或者无法 sudo 运行  )

Ansible-INI 示例:

.. code-block:: text

  some_host         ansible_port=2222     ansible_user=manager
  aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
  freebsd_host      ansible_python_interpreter=/usr/local/bin/python
  ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3

Non-SSH 连接类型
--------------


如上一节所述，Ansible 通过 SSH 执行剧本，但不限于此连接类型。

通过设置 ``ansible_connection=<connector>`` 选项来修改连接类型。如下 non-SSH  选项可用。

**local**

部署应用到管理机自身。

**docker**

该连接器使用本地Docker客户端将 Playbook 直接部署到 Docker 容器中。 此连接器处理以下参数:

ansible_host
    要连接的 Docker 容器名字
ansible_user
    在容器内操作的用户名，用户必须存在于容器内。

ansible_become
如果设置为 ``true``  ，则 ``become_user`` 将用于在容器内进行操作。
ansible_docker_extra_args
    可以是一个字符串，该字符串由 Docker 可识别的命令选项成，不限定命令任何其他参数。这个选项主要用于配置远程主机 Docker daemon 服务。

如下为立即部署容器的示例:

.. code-block:: yaml

   - name: create jenkins container
     docker_container:
       docker_host: myserver.net:4243
       name: my_jenkins
       image: jenkins

   - name: add container to inventory
     add_host:
       name: my_jenkins
       ansible_connection: docker
       ansible_docker_extra_args: "--tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/client-cert.pem --tlskey=/path/to/client-key.pem -H=tcp://myserver.net:4243"
       ansible_user: jenkins
     changed_when: false

   - name: create directory for ssh keys
     delegate_to: my_jenkins
     file:
       path: "/var/jenkins_home/.ssh/jupiter"
       state: directory

可用插件及更多案例请参考: :ref:`connection_plugin_list`.

.. note:: 如果你是从头开始阅读文档，这个案例应该是你看到的第一个 ansible playbook 案例。 这不是一个 Inventory 文件。 Playbooks 将在后面的章节更详细的介绍。

.. _inventory_setup_examples:

Inventory 设置示例
========================

点击这里查看 playbooks 清单剧本和 Ansible 其它组件示例 :ref:`sample_setup`

.. _inventory_setup-per_environment:

示例：一个环境一个 Inventory 清单
--------------------------------------

如果你需要管控多套环境，明智的做法是每套环境对应一个独立的 Inventory 配置。 这种方式很难有误操作。比如，你很难实际要修改 "staging" 的服务器却修改了 "test" 环境的。

对于上述示例，你可以使用 :file:`inventory_test` file:

.. code-block:: ini

  [dbservers]
  db01.test.example.com
  db02.test.example.com

  [appservers]
  app01.test.example.com
  app02.test.example.com
  app03.test.example.com


该文件仅包含属于 “test” 的主机环境。 在另外一个文件定义 "staging" 环境的服务器 :file:`inventory_staging`:

.. code-block:: ini

  [dbservers]
  db01.staging.example.com
  db02.staging.example.com

  [appservers]
  app01.staging.example.com
  app02.staging.example.com
  app03.staging.example.com

使用 :file:`site.yml` 
对 "test" 环境的所有服务器执行变更，使用如下命令::

  ansible-playbook -i inventory_test site.yml -l appservers

.. _inventory_setup-per_function:

示例： 按功能分组
-----------------

在上一节中，您已经看到具有相同功能的主机分组在一个群集。 这种方式可以方便我们在不影响 DB 服务器的前提下，在 Playbook 或 Role 中定义防火墙规则:


.. code-block:: yaml

  - hosts: dbservers
    tasks:
    - name: allow access from 10.0.0.1
      iptables:
        chain: INPUT
        jump: ACCEPT
        source: 10.0.0.1

.. _inventory_setup-per_location:

示例: 按位置分组
--------------------------

有些任务可能更在意服务器所在的位置。 如 ``db01.test.example.com`` 和 ``app01.test.example.com`` 在 DC1, 而 ``db02.test.example.com`` 位于DC2:


.. code-block:: ini

  [dc1]
  db01.test.example.com
  app01.test.example.com

  [dc2]
  db02.test.example.com

在实践中，您甚至可能需要混合所有这些设置，因为有一天可能需要更新特定数据中心中的所有节点，而另一天则不管应用程序服务器位置在哪里，所有服务器都更新。

.. seealso::

   :ref:`inventory_plugins`
       从动态或静态源中拉取 Inventory
   :ref:`intro_dynamic_inventory`
       从动态源中拉取 Inventory, 例如 云厂商
   :ref:`intro_adhoc`
       基础命令示例
   :ref:`working_with_playbooks`
       学习 Ansible 的配置、部署和语法编排
   `Mailing List <https://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
