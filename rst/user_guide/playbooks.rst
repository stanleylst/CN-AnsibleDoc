.. _working_with_playbooks:

Playbooks 介绍
======================

`Playbooks` 是 `Ansible` 的配置、部署和编排语言。 Playbooks 是描述您希望执行的一组策略或IT流程中的一组步骤。

如果 `Ansible` 模块是车间工具，则剧本是说明手册，主机清单是原材料。

最基本的功能， playbooks 可用于管理远程计算机的配置和部署。更高级一点讲，他们可以序列平滑更新多层级结构的部署，并且可以将操作委派给其他主机，并与监视系统和负载均衡系统进行交互。

尽管内容很多，但无需一次学习所有内容。 您可以从小处着手，并在需要时随时间使用更多功能。

playbook 的语法设计非常人性化，使用最基本的文本语言开发。 有多种方式来编排剧本引用文件，要深入使用 `Ansible` 需要了解本本次内容。


playbook 文档请参考 `Example Playbooks <https://github.com/ansible/ansible-examples>`。 这些最佳实践说明将展示如何将许多不同的概念结合在一起解决实际问题。

.. toctree::
   :maxdepth: 2

   playbooks_intro
   playbooks_best_practices
   playbooks_reuse
   playbooks_reuse_roles
   playbooks_variables
   playbooks_templating
   playbooks_conditionals
   playbooks_loops
   playbooks_blocks
   playbooks_special_topics
   playbooks_strategies
   guide_rolling_upgrade
