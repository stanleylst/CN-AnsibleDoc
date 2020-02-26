.. _ansible_documentation:

Ansible「2.9」 中文官方文档
=====================

Ansible 简介
`````````````

Ansible 是一款 IT 自动化工具。主要应用场景有配置系统、软件部署、持续发布及不停服平滑滚动更新的高级任务编排。

Ansible 本身非常简单易用，同时注重安全和可靠性，以最小化变动为特色，使用 OpenSSH 实现数据传输 ( 如果有需要的话也可以使用其它传输模式或者 pull 模式 )，其语言设计非常利于人类阅读，即使是针对不刚接触 Ansible 的新手来讲亦是如此。

我们坚信无论什么范围的环境，简单都是必须的，所以我们的设计尽可能满足各类型的繁忙人群：开发人员、系统管理员、发布工程师、IT 管理员等所有类型的人。同时， Ansible 适用于各种环境，小到几台多到成千上万台的企业实际环境都完全满足。

Ansible 不使用C/S架构管理节点，即没有 Agent 。 这样的架构使得 Ansible 不会存在如何升级远程 Agent 管理进程或者因为没有安装 Agent 而无法管理系统。 因为 OpenSSH 是非常流行的开源组件，安全问题也非常少 。 Ansible 的 `去中心化` 管理方式深受业内认可， 即它只依赖 OS 的 KEY 认证访问远程主机。 如需， Ansible 可以便捷接入 Kerberos, LDAP 或者其它认证系统。

该文档覆盖了 Ansible 所有版本的文档，可以通过左边栏索引跳转对应页面 「别跳了，就翻译了一个版本，官方更新太快了」。 官方并行管理多个版本 Ansible 及对应版本的文档，所以使用前请确认参考的是对应版本的文档。 最新的功能，我们会在版本中标识新增的功能。

Ansible 主版本大概每年 3-4 个版本。 核心功能应用的发布会很谨慎，每个版本都非常重视代码质量和语言简洁性。 但是， 社区贡献的新模块和组件发展的非常快，每个版本都会合入很多的新模块。


.. toctree::
   :maxdepth: 2
   :caption: 安装、升级 * 配置

   installation_guide/index
   porting_guides/porting_guides

.. toctree::
   :maxdepth: 2
   :caption: Ansible 的使用

   user_guide/index

.. toctree::
   :maxdepth: 2
   :caption: 向 Ansible 贡献代码

   community/index

.. toctree::
   :maxdepth: 2
   :caption: 扩展 Ansible

   dev_guide/index

.. toctree::
   :glob:
   :maxdepth: 1
   :caption: Common Ansible Scenarios

   scenario_guides/cloud_guides
   scenario_guides/network_guides
   scenario_guides/virt_guides

.. toctree::
   :maxdepth: 2
   :caption: Ansible Network Automation

   network/getting_started/index
   network/user_guide/index
   network/dev_guide/index

.. toctree::
   :maxdepth: 2
   :caption: Ansible Galaxy

   galaxy/user_guide.rst
   galaxy/dev_guide.rst


.. toctree::
   :maxdepth: 1
   :caption: Reference & Appendices

   ../modules/modules_by_category
   reference_appendices/playbooks_keywords
   reference_appendices/common_return_values
   reference_appendices/config
   reference_appendices/general_precedence
   reference_appendices/YAMLSyntax
   reference_appendices/python_3_support
   reference_appendices/interpreter_discovery
   reference_appendices/release_and_maintenance
   reference_appendices/test_strategies
   dev_guide/testing/sanity/index
   reference_appendices/faq
   reference_appendices/glossary
   reference_appendices/module_utils
   reference_appendices/special_variables
   reference_appendices/tower
   reference_appendices/automationhub
   reference_appendices/logging


.. toctree::
   :maxdepth: 2
   :caption: Release Notes

.. toctree::
   :maxdepth: 2
   :caption: Roadmaps

   roadmap/index.rst
