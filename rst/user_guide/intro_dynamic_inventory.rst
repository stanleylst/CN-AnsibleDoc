.. _intro_dynamic_inventory:
.. _dynamic_inventory:

******************************
动态 Inventory 清单配置
******************************

.. contents::
   :local:

如果你的 Ansible Inventory 清单会随需求变化，主机会随业务需求更换或关闭，上面的这些需求， 静态 Inventory :ref:`inventory`则无法满足你的需求了。 你可能需要同时使用多个来源的主机清单： 云厂商， LDAP, `Cobbler <https://cobbler.github.io>`_, 或者企业 CMDB 系统。

Inventory 插件利用了 Ansible 最新的核心代码。我们建议使用插件而非脚本动态获取 Inventory 清单。你可以参考 :ref:`write your own plugin <developing_inventory>`  连接动态 Inventory 源。

当然了，如果你坚持的话依然可以使用 Inventory 脚本的方式调用。但是我们 Inventory 插件的实现确保了向后兼容性，更加灵活健壮。如下是如何使用 Inventory 脚本的示例。

如果你需要一个 GUI 页面来处理动态 Inventory， :ref:`ansible_tower` 可以同步你所有的动态 Inventory 源，并提供 web 接口和 REST api 查看结果，同时提供图形化界面的编辑器。 tower 会把所有的 hosts 存储到数据库，你可以关联历史事件，你可以查看哪些主机运行了什么命令，遇到过什么故障。

.. _cobbler_example:

Inventory 脚本示例: Cobbler
=================================

Ansible 和 Cobbler `Cobbler <https://cobbler.github.io>`_ 无缝衔接， Cobbler 是一套 Linux 服务器部署安装的自动化工具，创始人是 Michael DeHaan， 现在由 James Cammarata 主导，而这两个人现在都在 Ansible 工作。

Cobbler 虽然主要用于启动安装 OS， 并管理 DHCP 和 DNS ，但 Cobbler 具有通用层，可以表示多个（甚至同时）配置管理系统的数据，并充当 “轻量级 CMDB” 。



将 Ansible 的 inventory 清单关联至 Cobbler, 复制  `this script <https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/cobbler.py>`_ 到 ``/etc/ansible`` 并且添加可执行权限 ``chmod +x`` . 每当使用 ``-i`` 选项(e.g. ``-i /etc/ansible/cobbler.py``)运行 ``cobblerd``时， 都会都可以使用 Cobbler 的 XMLRPC API 与 Cobbler 通信。

为了让 Ansible 知道 Coobler 服务器在哪里，并且可以使用缓存功能提升性能，需要添加 ``cobbler.ini`` 到 ``/etc/ansible`` 目录。

.. code-block:: text

    [cobbler]

    # Set Cobbler's hostname or IP address
    host = http://127.0.0.1/cobbler_api

    # API calls to Cobbler can be slow. For this reason, we cache the results of an API
    # call. Set this to the path you want cache files to be written to. Two files
    # will be written to this directory:
    #   - ansible-cobbler.cache
    #   - ansible-cobbler.index

    cache_path = /tmp

    # The number of seconds a cache file is considered valid. After this many
    # seconds, a new API call will be made, and the cache file will be updated.

    cache_max_age = 900

首先直接运行  ``/etc/ansible/cobbler.py`` 脚本，正常情况下你可以看到 JSON 格式的数据输出，但也有可能因为还没有配置好而没有任何输出。

让我们探索下他的使用场景. Cobbler 中, 假设有一个类似如下的使用场景:

.. code-block:: bash

    cobbler profile add --name=webserver --distro=CentOS6-x86_64
    cobbler profile edit --name=webserver --mgmt-classes="webserver" --ksmeta="a=2 b=3"
    cobbler system edit --name=foo --dns-name="foo.example.com" --mgmt-classes="atlanta" --ksmeta="c=4"
    cobbler system edit --name=bar --dns-name="bar.example.com" --mgmt-classes="atlanta" --ksmeta="c=5"

在上面的示例中， 'foo.example.com' 主机即可以直接在 Ansible 中找到，但在使用组 'webserver' or 'atlanta' 时也可以被找到。 由于 Ansible 使用 SSH ，因此他只能通过 'foo.example.com' 找到对应关系，  'foo' 是不行的。 类似的， "ansible foo"  Ansible 也是找不到对应主机的。 但是 "ansible 'foo*'" 是可以的，因为系统 DNS 的名字是以 'foo' 开始的。

该脚本不仅提供主机和组信息。 此外，作为奖励，当运行 “setup” 模块（ 使用 playbook 剧本时会自动发生 ）时，变量 “a”，“b” 和 “ c” 都将自动填充到模板中

.. code-block:: text

    # file: /srv/motd.j2
    Welcome, I am templated with a value of a={{ a }}, b={{ b }}, and c={{ c }}

可以像这样执行:

.. code-block:: bash

    ansible webserver -m setup
    ansible webserver -m template -a "src=/tmp/motd.j2 dest=/etc/motd"

.. note::
   
   webserver 和配置文件的变量来自 Cobbler ，您仍可以像在 Ansible 中一样传递自己的变量，但是外部 Inventory 清单脚本中的变量将覆盖具有相同名称的任何变量。

因此，使用上面的模板（ ``motd.j2`` ），这将修改 'foo' 系统的 ``/etc/motd``:

.. code-block:: text

    Welcome, I am templated with a value of a=2, b=3, and c=4

And on system 'bar' (bar.example.com):

.. code-block:: text

    Welcome, I am templated with a value of a=2, b=3, and c=5

从技术上讲, 尽管没有充分的理由这样做，但这也适用:

.. code-block:: bash

    ansible webserver -m shell -a "echo {{ a }}"

换句话讲，你可以在参数/命令使用这些变量。

.. _aws_example:

Inventory 清单脚本示例: AWS EC2
=================================

如果您使用 Amazon Web Services EC2，则维护一份静态 Inventory 清单文件可能不是最佳方法，因为主机可能会随着时间的流逝而移动，或者由外部应用程序进行管理，甚至你可能正在使用AWS自动缩放。
这些情况建议你使用 `EC2 external inventory  <https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py>`_ .

您可以通过如下两种方式之一使用此脚本。 最简单的方法是使用 Ansible 的 ``-i`` 命令行选项，并在将脚本标记为可执行文件后指定脚本的路径：

.. code-block:: bash

    ansible -i ec2.py -u ubuntu us-east-1d -m ping

第二种方法是复制脚本到  `/etc/ansible/hosts` 并 `chmod +x` 。 同时，你需要复制 `ec2.ini  <https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini>`_ file to `/etc/ansible/ec2.ini`。 然后，您可以像往常一样运行 Ansible。

要成功的调用 AWS API ，您必须配置 Boto（ AWS 的 Python 接口）。 您可以通过这里查看 `several ways <http://docs.pythonboto.org/en/latest/boto_config_tut.html>`_ ，但是最简单的方法是导出两个环境变量：


.. code-block:: bash

    export AWS_ACCESS_KEY_ID='AK123'
    export AWS_SECRET_ACCESS_KEY='abc123'

你运行脚本可以自测配置是否成功:

.. code-block:: bash

    cd contrib/inventory
    ./ec2.py --list

片刻后，你会看到 JSON 格式输出的所有区域的 EC2 Inventory 清单。

如果你使用 Boto 配置文件管理多个 AWS 账户，则可以传递 ``--profile PROFILE`` 参数给 ``ec2.py`` 脚本。 示例配置文件可能是：

.. code-block:: text

    [profile dev]
    aws_access_key_id = <dev access key>
    aws_secret_access_key = <dev secret key>

    [profile prod]
    aws_access_key_id = <prod access key>
    aws_secret_access_key = <prod secret key>

运行 ``ec2.py --profile prod`` 获取 prod 账户的 Inventory， 但是 ``ansible-playbook`` 不支持该选项。
你同样也可以使用 ``AWS_PROFILE``  变量，比如 ``AWS_PROFILE=prod ansible-playbook -i ec2.py myplaybook.yml``

由于每个区域都需要自己独立的API调用，因此，如果你只希望调用部分区域，则可以编辑 ``ec2.ini`` 文件，注释掉不需要的区域。


``ec2.ini`` 中还有其他配置选项，包括缓存控制和目标变量。 默认情况下，``ec2.ini`` 的配置是为 “所有Amazon云服务” 生效。 ，但是您可以注释掉所有不适用的功能。

.. code-block:: text

    [ec2]
    ...

    # To exclude RDS instances from the inventory, uncomment and set to False.
    rds = False

    # To exclude ElastiCache instances from the inventory, uncomment and set to False.
    elasticache = False
    ...

从本质上讲，Inventory 清单文件只是从名称到目标的映射。默认的 ``ec2.ini`` 设置为从外部配置 EC2（例如从笔记本电脑）运行Ansible， 显然这不是管理EC2的最有效方法。

如果你从 EC2，内部 DNS 和 IP地址运行 Ansible，这种方式比使用公共 DNS 更高效。这种情况下，你可以修改实例的 ``ec2.ini`` 中的 ``destination_variable`` 为私有 DNS 名称。这种方式在私有网段的 VPC 中运行 Ansible 的场景下非常重要，这种方式也是唯一的办法访问私有 IP 地址的实例。 VPN 实例，``ec2.ini`` 的配置项 `vpc_destination_variable` 提供了

如果从 EC2 内部运行 Ansible ，则内部 DNS 名称和 IP 地址可能比公共 DNS 名称更有意义。在这种情况下，你可以将 ``ec2.ini`` 中的 ``destination_variable`` 为修改实例的私有 DNS 名称。 在 VPC 内的私有子网中运行 Ansible 时，这尤其重要，在该子网中，访问实例的唯一方法是通过其私有 IP 地址。 对于 VPC 实例，对于VPC实例，``ec2.ini`` 中的 `` vpc_destination_variable`` 如何选择最合适您用例的案例  `boto.ec2.instance variable <http://docs.pythonboto.org/en/latest/ref/ec2.html#module-boto.ec2.instance>`_ 。

EC2 外部 Inventory 清单提供了从多个组到实例的映射:

Global
  ``ec2`` 组中的所有实例

Instance ID
  These are groups of one since instance IDs are unique.

  e.g.
  ``i-00112233``
  ``i-a1b1c1d1``

Region
  AWS 一个区域中的所有实例
  e.g.
  ``us-east-1``
  ``us-west-2``

Availability Zone
  一组可用性区域中所有实例

  e.g.
  ``us-east-1a``
  ``us-east-1b``

Security Group
  
  实例可以属于一个或多个安全组。为每个安全组创建一个组，组名由字母和数字组成，除字母和数字外的所有字符都转换为下划线 (_)。 所有的案例组都以 ``security_group_`` 开头。 当下， dashed (-) 也使用下划线 (_) 表示。  您可以使用 ``ec2.ini`` 中的 ``replace_dash_in_groups`` 设置进行更改（此设置在多个版本中已更改，因此请检查 ``ec2.ini``以获取详细信息）。

  e.g.
  ``security_group_default``
  ``security_group_webservers``
  ``security_group_Pete_s_Fancy_Group``

Tags
  每个实例可以具有与其关联的各种键/值对，称为“标签”。虽然任何字符串都可以用来表示键值，但最常见的标准名是 'Name',。每个键值对都是其实例组的名字，但是要将特殊字符转换为下划线，格式如 ``tag_KEY_VALUE``
  e.g.
  ``tag_Name_Web`` can be used as is
  ``tag_Name_redis-master-001`` becomes ``tag_Name_redis_master_001``
  ``tag_aws_cloudformation_logical-id_WebServerGroup`` becomes ``tag_aws_cloudformation_logical_id_WebServerGroup``


当 Ansible 和特定服务器交互时， EC2 Inventory 脚本将再次调用 ``--host HOST`` 选项。 这将在索引缓存中查找 HOST 以获取实例 ID，然后对 AWS 进行 ``API`` 调用以获取该特定实例的信息。 然后，它将有关该实例的信息作为变量提供给您的 Playbook 。 每个变量都以``ec2_`` 为前缀。

- ec2_architecture
- ec2_description
- ec2_dns_name
- ec2_id
- ec2_image_id
- ec2_instance_type
- ec2_ip_address
- ec2_kernel
- ec2_key_name
- ec2_launch_time
- ec2_monitored
- ec2_ownerId
- ec2_placement
- ec2_platform
- ec2_previous_state
- ec2_private_dns_name
- ec2_private_ip_address
- ec2_public_dns_name
- ec2_ramdisk
- ec2_region
- ec2_root_device_name
- ec2_root_device_type
- ec2_security_group_ids
- ec2_security_group_names
- ec2_spot_instance_request_id
- ec2_state
- ec2_state_code
- ec2_state_reason
- ec2_status
- ec2_subnet_id
- ec2_tag_Name
- ec2_tenancy
- ec2_virtualization_type
- ec2_vpc_id


``ec2_security_group_ids`` 和 ``ec2_security_group_names`` 都是逗号分隔的所有安全组。每个 EC2 的 tag 格式如 ``ec2_tag_KEY``。

查看实例支持的完整变量列表，执行如下脚本:

.. code-block:: bash

    cd contrib/inventory
    ./ec2.py --host ec2-12-12-12-12.compute-1.amazonaws.com


需要注意的 AWS Inventory 脚本会缓存 API 的调用结果，缓存配置可以在 ec2.ini 修改。如果想清除缓存，带上 ``--refresh-cache`` 参数:


.. code-block:: bash

    ./ec2.py --refresh-cache

.. _openstack_example:


Inventory 脚本示例: OpenStack
===================================

如果您使用 OpenStack 云，则无需手动维护 Inventory 清单文件，而可以使用 ``openstack_inventory.py`` 动态清单直接从 OpenStack 中获取有关您的计算实例的信息。

从这里下载最新的 OpenStack Inventory 脚本 `here <https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/openstack_inventory.py>`_.

你可以显式  (传参 `-i openstack_inventory.py` ) 或隐式 ( `/etc/ansible/hosts` ) 的使用 Inventory 脚本。

OpenStack inventory 脚本使用
------------------------------------------

下载最新的 OpenStack 动态 Inventory 脚本并赋予可执行权限::

    wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/openstack_inventory.py
    chmod +x openstack_inventory.py

.. note::
    不修改名字为 `openstack.py`。 这个名字会和 openstacksdk 导入冲突。

加载 OpenStack RC file:

.. code-block:: bash

    source openstack.rc

.. note::
    OpenStack RC 文件包含客户端工具与云厂商建立连接所需要的环境变量, 例如身份验证URL, 用户名，密码和区域名。如何下载，创建或加载 OpenStack RC 文件，更多请参考 `Set environment variables using the OpenStack RC file <https://docs.openstack.org/user-guide/common/cli_set_environment_variables_using_openstack_rc.html>`_.


您可以通过运行一个简单的命令 ( ``nova list`` ) 并确保其不返回错误来确认文件已成功获得源文件。

.. note::
    OpenStack 命令行客户端需要运行 `nova list` 命令。关于如何安装，更多信息请参考 `Install the OpenStack command-line clients <https://docs.openstack.org/user-guide/common/cli_install_openstack_command_line_clients.html>`_

使用如下命令测试动态 Inventory 脚本是否正常运行::

    ./openstack_inventory.py --list

片刻后，你会得到关于实例的 JSON 格式信息。

确认动态清单脚本按预期工作后，可以告诉 Ansible 将 `openstack_inventory.py` 脚本用作 Inventory 清单文件，如下所示：


.. code-block:: bash

    ansible -i openstack_inventory.py all -m ping

隐式使用 OpenStack Inventory 脚本
--------------------------------

下载最新的 OpenStack 动态 Inventory 脚本，赋予可执行权限，复制为 `/etc/ansible/hosts`:

.. code-block:: bash

    wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/openstack_inventory.py
    chmod +x openstack_inventory.py
    sudo cp openstack_inventory.py /etc/ansible/hosts

下载配置模板文件，按需修改并且复制为 `/etc/ansible/openstack.yml`:

.. code-block:: bash

    wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/openstack.yml
    vi openstack.yml
    sudo cp openstack.yml /etc/ansible/

使用如下命令测试 OpenStack 动态 Inventory 脚本是否正常工作:

.. code-block:: bash

    /etc/ansible/hosts --list


片刻后，你会得到关于实例的 JSON 格式信息。

刷新缓存
--------------

需要注意的是， OpenStack 动态 Inventory 脚本会缓存 API 的重复调用。在执行 openstack_inventory.py  使用 ``--refresh`` 参数清除缓存:

.. code-block:: bash

    ./openstack_inventory.py --refresh --list

.. _other_inventory_scripts:

其它 inventory scripts
=======================


所有的脚本这里找 `contrib/inventory directory <https://github.com/ansible/ansible/tree/devel/contrib/inventory>`_. 
所有的 Inventory 脚本的通用用法都差不多，你可以在这里查看 :ref:`write your own inventory script <developing_inventory>`.

.. _using_multiple_sources:

Inventory 目录和多个 Inventory 源的使用
=======================================

如果 Ansible 运行时 ``-i`` 指定给的位置是目录（或在ansible.cfg中配置），则 Ansible 可以同时使用多个清单资源。 这样做时，可以在的运行中同时混合使用动态和静态管理的 Inventory 资源。 即时混合云！

在 Inventory 目录中，可执行文件会被视为动态 Inventory 源， 其它文件绝大部分会被视为静态 Inventory 源。以如下后缀结尾的文件将被忽略:

.. code-block:: text

    ~, .orig, .bak, .ini, .cfg, .retry, .pyc, .pyo


您可以通过在 ansible.cfg 中配置 ``inventory_ignore_extensions`` 列表，或设置环境变量 :envvar:`ANSIBLE_INVENTORY_IGNORE` 制定自己希望忽略的后缀。 不论哪种情况，该值都应是逗号分隔，如上所示。

库存目录中的任何 ``group_vars`` 和 ``host_vars`` 子目录都将按预期方式进行解析，从而使 Inventory 目录成为组织不同配置集的有力方法。 更多请参考 :ref:`using_multiple_inventory_sources` .

.. _static_groups_of_dynamic:

动态组中的静态组
====================

在静态 Inventory 文件中定义嵌套组时，子组必须被事先定义，不然 Ansible 会抛错。如果要定义动态子组的静态组，请预先在静态 Inventory 文件中定义动态组为空。

.. code-block:: text

    [tag_Name_staging_foo]

    [tag_Name_staging_bar]

    [staging:children]
    tag_Name_staging_foo
    tag_Name_staging_bar


.. seealso::

   :ref:`intro_inventory`
       静态 Inventory 文件介绍
   `Mailing List <https://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
