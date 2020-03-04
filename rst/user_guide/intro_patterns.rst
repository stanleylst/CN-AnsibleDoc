.. _intro_patterns:

Pattern: 正则匹配主机和组
====================================

当你执行 ad-hoc 临时命令或 playbook 时，你必须指定要被变更的节点或组。 Pattern 可以让你更方便的指定特定主机或组。 可以单独指定一台主机，一个 IP 地址，一个 Inventory 清单组， 一个子网的组或者你 Inventory 中的所有主机。 Pattern 具有高度的灵活性- 您可以排除或要求主机的子集，使用通配符或正则表达式等等。 Ansible 在模式中包含的所有清单主机上执行。

.. contents::
   :local:

使用 patterns 模式
--------------

执行临时命令或剧本的任何时候都可以使用模式。 该模式是 :ref:`ad-hoc command<intro_adhoc>` 中唯一没有标志的参数。 它通常是第二个参数::

    ansible <pattern> -m <module_name> -a "<module options>"

举例::

    ansible webservers -m service -a "name=httpd state=restarted"

在Playbook 中，pattern 是 ``hosts:`` 的值。

.. code-block:: yaml

   - name: <play_name>
     hosts: <pattern>

举例::

    - name: restart webservers
      hosts: webservers


由于您通常想一次对多个主机运行命令或剧本，因此模式通常指代清单组
由于你通常希望针对多台主机一次性运行一个命令或 playbook， patterns 通常关联是一组主机列表。 ad-hoc 命令和 playbook 将对 ``webservers`` 中的所有主机做变更。

.. _common_patterns:

通过 patterns
---------------

下表展示了 patterns 模式的用法。

.. table::
   :class: documentation-table

   ====================== ================================ ===================================================
   Description            Pattern(s)                       Targets
   ====================== ================================ ===================================================
   All hosts              all (or \*)

   One host               host1

   Multiple hosts         host1:host2 (or host1,host2)

   One group              webservers

   Multiple groups        webservers:dbservers             all hosts in webservers plus all hosts in dbservers

   Excluding groups       webservers:!atlanta              all hosts in webservers except those in atlanta

   Intersection of groups webservers:&staging              any hosts in webservers that are also in staging
   ====================== ================================ ===================================================

.. note:: 你可以使用  (``,``) 或(``:``) 分隔多个主机。 推荐使用逗号，因为在 IPv6 的场景下首先逗号

当你掌握基础 pattern 后，你就可以高级进阶混合使用他们了。示例:

    webservers:dbservers:&staging:!phoenix

该 pattern 表示 'webservers' and 'dbservers' 组的所有主机，和 'staging' 组中的交集的所有主机，并且这些主机不在 'phoenix' 组中。

你也可以使用正则表示的 FQDN 主机名或 IP 地址:

   192.0.\*
   \*.example.com
   \*.com

你也可以同时混合使用正则和 patterns:

    one*.com:dbservers

patterns 局限性
--------------

Patterns 依赖于 Inventory。 如果 host 或 group 不在 Inventory 清单仓库，则不能使用 Pattern 。 如果你使用的 Patterns IP 或主机名不存在。 会引发如下报错:

.. code-block:: text

   [WARNING]: No inventory was parsed, only implicit localhost is available
   [WARNING]: Could not match supplied host pattern, ignoring: *.not_in_inventory.com


Pattern 必须遵循 Inventory 语法规则。如果你定义了一台 host :ref:`alias<inventory_aliases>`:

.. code-block:: yaml

    atlanta:
      host1:
        http_port: 80
        maxRequestsPerChild: 808
        host: 127.0.0.2

你必须在 Pattern 使用别名。 在如上的案例中，你必须在 pattern 中使用 ``host1``。 如果你使用 IP 地址，会报错::

   [WARNING]: Could not match supplied host pattern, ignoring: 127.0.0.2

高级 Pattern 选项
------------------------

如上所述的通用 patterns 已经满足你的绝大部分需求，但 Ansible 也提供了几种其它的方式来定义你的目标主机和组。

在 patterns 使用变量
^^^^^^^^^^^^^^^^^^^

ansible-playbook 通过 ``-e`` 选项可以接受变量，在 pattern 中可以使用 ``{{ 变量 }}``::

    webservers:!{{ excluded }}:&{{ required }}

在 patterns 使用组位置参数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


您可以根据主机在主机组中的位置定义主机或主机子集. 如下示例::

    [webservers]
    cobweb
    webbing
    weber

你可以使用下标切割选择需要的主机或主机组::

    webservers[0]       # == cobweb
    webservers[-1]      # == weber
    webservers[0:2]     # == webservers[0],webservers[1]
                        # == cobweb,webbing
    webservers[1:]      # == webbing,weber
    webservers[:3]      # == cobweb,webbing,weber

在 patterns 中使用正则
^^^^^^^^^^^^^^^^^^^^^^^^^

以 ``~`` 开始的模式将会被认定为正则表达式::

    ~(web|db).*\.example\.com

Patterns and ansible-playbook 标志
-----------------------------------

命令行的优先级比 playbook 高，通过指定命令行选项可以覆盖 playbook 中的定义。 举例：你在 playbook 中定义 ``hosts: all``，但在命令行中指定 ``-i 127.0.0.2`` , 命令行会覆盖 playbook 中的定义。 这种方式甚至在 Inventory 中没有定义目标主机都可行。同样，你可以使用 ``--limit`` 标识指定特定的目标主机::

    ansible-playbook site.yml --limit datacenter2

最后，你可以使用 ``--limit`` 从一个文件中读取主机列表， 使用时在文件名前加 ``@``::

    ansible-playbook site.yml --limit @retry_hosts.txt

如果 :ref:`RETRY_FILES_ENABLED` 设置 ``True``， 当 ``ansible-playbook`` 执行失败的主机记录到以 ``retry`` 结尾的文件中。这个文件在每次 ``ansible-playook`` 执行结束后都会被覆盖。


    ansible-playbook site.yml --limit @site.retry


更多关于 ad-hoc 和 playbook 的 pattern 信息请参考 :ref:`intro_adhoc` and :ref:`playbooks_intro`.

.. seealso::

   :ref:`intro_adhoc`
       基础命令示例
   :ref:`working_with_playbooks`
       学习 Ansible 配置管理语言
   `Mailing List <https://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel
