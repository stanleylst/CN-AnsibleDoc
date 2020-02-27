.. _installation_guide:
.. _intro_installation_guide:

安装 Ansible
===================

该页面展示如何在不同平台安装 Ansible。
Ansible 不用安装客户端，通过 SSH 协议管理远程机器。 一旦安装， Ansible 不需要数据库，也不需要后台保持运行。 一旦安装( 一台简单的笔记本也可以安装 )完成，它可以管理整个集群。 当 Ansible 管理远程机器时，它不需要软件保持安装状态或者运行状态。 因此，当 Ansible 的升级很简单，只需要迁移到新版本即可。


.. contents::
  :local:

准备工作
--------------

准备一台管理机，该管理机可通过 SSH 连接到你所有的被管理机。


.. _control_node_requirements:

管理机环境要求
^^^^^^^^^^^^^^^^^^^^^^^^^

运行 Ansible 的服务器必须且只需要安装有 Python 2.7+ 或者 Python 3.5+。 Red Hat, Debian, CentOS, macOS, 任一 BSD 系列的系统均可。 但`Windows` 不能用于管理机。


选择管理机时，需要注意的时，网络条件越好越便于管理。 比如：当你选择在云上使用 Ansible 时，那么管理机和管理节点都在云上是最佳选择，连接外网的节点速度则会慢很多，也存在很大的安全风险。

.. note::

    macOS 系统默认的文件句柄数比较小，如果你希望并发15个或者更多线程，你需要通过如下命令增加系统文件句柄打开上限。 ``sudo launchctl limit maxfiles unlimited``。 不然，过多的线程数会提示报错： Too many open files 。


.. warning::

    需要注意的是，部分模块或者组件在使用时需要额外安装插件。 管理节点需要安装必要的插件后方可正常被管理。具体请参考对应的模块文档


.. _managed_node_requirements:


受管节点环境要求
^^^^^^^^^^^^^^^^^^^^^^^^^

受管节点需要和外界正常通信，默认使用 SSH 协议。 默认使用 SFTP 。 如果 SFTP 无法使用，你可以在 `ansible.cfg` 中将其修改为 SCP . 同样，受管机需要有 Python 2.6+ 或 Python 3.5以上的环境

.. note::

   * 如果受管机开启了 SELinux，你需要安装 libselinux-python ，不然 copy/file/template 等任何相关联的功能都无法使用。 你可以使用 :ref:`yum module<yum_module>` 或 :ref:`dnf module<dnf_module>` 在受管机安装软件。

   * 默认情况下 Ansible 使用 :file:`/usr/bin/python` 下的Python 解释器运行命令。 但部分 Linux 发行版可能只安装了 Python 3 解释器，可以从 :file:`/usr/bin/python3` 找到。如果遇到类似如下的错误，表示你需要检查 Python 环境 ::

        "module_stdout": "/bin/sh: /usr/bin/python: No such file or directory\r\n"

     你可以设置仓库文件对应亦是 :ref:`ansible_python_interpreter<ansible_python_interpreter>` (参考 :ref:`inventory`) 指明解释器位置，或者干脆安装一个 Python 2的解释器。 如果 Python 2 的解释器没有安装在指定目录 :command:`/usr/bin/python` 。 那你依然需要修改仓库文件指定解释器的位置。 :ref:`ansible_python_interpreter<ansible_python_interpreter>`


   * Ansible 的 :ref:`raw module<raw_module>` 模块和 :ref:`script module<script_module>` 不依赖受管机的 Python 环境。 因此，从技术角度上讲，我可以使用 Ansible 这两个模块 ( :ref:`raw module<raw_module>` 和 :ref:`script module<script_module>` ) 编译安装 Python 环境。 举例如下： 你想在 RHEL 系列系统上安装 Python 2环境，请参考如下命令：

     .. code-block:: shell

        $ ansible myhost --become -m raw -a "yum install -y python2"

.. _what_version:

指定安装具体的版本
---------------------------------------

具体安装哪个版本取决于你的需求。 你可以选择如下的任何一种方式来安装Ansible:

* 使用系统默认的包管理器安装 (for Red Hat Enterprise Linux (TM), CentOS, Fedora, Debian, or Ubuntu).
* Install with ``pip`` ( Python 包管理器 ).
* 源码安装 ``devel`` 版本的Ansible w体验最新版本的功能

.. note::

    只有当你希望修改 Ansible 引擎或者尝试修改码源时，你才会需要安装 ``devel``  版本，因为 ``devel`` 版本是非稳定版本，变化 非常快。

Ansible 每年发布 2-3 个新版本。 得益于发布周期短，小 BUGS 通常在下一个版本修复而不会在稳定分支上保留。 主要 BUGS 如有需要会使用专门的维护分支，当然，这种情况并不多见。


.. _installing_the_control_node:
.. _from_yum:

Installing Ansible on RHEL, CentOS, or Fedora
----------------------------------------------

On Fedora:

.. code-block:: bash

    $ sudo dnf install ansible

On RHEL and CentOS:

.. code-block:: bash

    $ sudo yum install ansible

RPMs for RHEL 7  and RHEL 8 参考 `Ansible Engine repository <https://access.redhat.com/articles/3174981>`_.

RHEL 8 开启repository :

.. code-block:: bash

    $ sudo subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms

RHEL 7 开启repository : 

.. code-block:: bash

    $ sudo subscription-manager repos --enable rhel-7-server-ansible-2.9-rpms

RHEL, CentOS, and Fedora 的最新 RPM 版本获取方式： `EPEL <https://fedoraproject.org/wiki/EPEL>`_ as well as `releases.ansible.com <https://releases.ansible.com/ansible/rpm>`_.

Ansible 2.4+ 可以管理包含 Python 2.6 或更高版本的早期操作系统。

你也可以编译自己的 RPM 包：


.. code-block:: bash

    $ git clone https://github.com/ansible/ansible.git
    $ cd ./ansible
    $ make rpm
    $ sudo rpm -Uvh ./rpm-build/ansible-*.noarch.rpm

.. _from_apt:

Installing Ansible on Ubuntu
----------------------------

Ubuntu builds are available `in a PPA here <https://launchpad.net/~ansible/+archive/ubuntu/ansible>`_.


配置 PPA 或者安装 Ansible:

.. code-block:: bash

    $ sudo apt update
    $ sudo apt install software-properties-common
    $ sudo apt-add-repository --yes --update ppa:ansible/ansible
    $ sudo apt install ansible

.. note:: 旧的 Ubuntu 发行版 "software-properties-common" 名字是 "python-software-properties". 使用 ``apt-get`` 而不是 ``apt`` 。 同时，只有比较新的发行版才有 (i.e. 18.04, 18.10, etc.)  ``-u`` or ``--update`` 参数。 根据情况调整你的脚本。

Debian/Ubuntu 也可以从源码编译:

.. code-block:: bash

    $ make deb


如果您希望从源头开始获得开发分支，请参考接下来的介绍

Installing Ansible on Debian
----------------------------

Debian 用户可以使用和 Ubuntu PPA 一样的源。



增加如下行到 /etc/apt/sources.list:

.. code-block:: bash

    deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main

执行如下命令:

.. code-block:: bash

    $ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
    $ sudo apt update
    $ sudo apt install ansible

.. note:: 该方法已在Debian Jessie和Stretch中的Trusty来源中得到验证，但在早期版本中可能不受支持。 旧版本中使用 ``apt-get`` 而不是 ``apt`` .

Installing Ansible on Gentoo with portage
-----------------------------------------

.. code-block:: bash

    $ emerge -av app-admin/ansible

要安装最新版本，可能需要先屏蔽Ansible软件包，然后再进行开发: 

.. code-block:: bash

    $ echo 'app-admin/ansible' >> /etc/portage/package.accept_keywords

Installing Ansible on FreeBSD
-----------------------------


尽管 Ansible 可以工作在 Python 2 或 3 版本, 但 FreeBSD 每个版本有不同的名字， 安装方式如下:

.. code-block:: bash

    $ sudo pkg install py27-ansible

or:

.. code-block:: bash

    $ sudo pkg install py36-ansible


你也可以使用 ports 安装:

.. code-block:: bash

    $ sudo make -C /usr/ports/sysutils/ansible install

同样可以指定版本安装，如 ``ansible25``

旧版本的 FreeBSD 使用 pkg 管理方式( 具体看你使用什么包管理工具 ):


.. code-block:: bash

    $ sudo pkg install ansible

.. _on_macos:

Installing Ansible on macOS
---------------------------

Mac 安装 Ansible 推荐使用 ``pip``



具体文档请参考 :ref:`from_pip`。  如果你的 macOS 系统是 10.12+ , 你最好升级到最新的 ``pip`` 版本， pip 必须作为模块在 macOS 运行， 具体参考如上文档。

.. _from_pkgutil:

Installing Ansible on Solaris
-----------------------------

参考 `SysV package from OpenCSW <https://www.opencsw.org/packages/ansible/>`_.

.. code-block:: bash

    # pkgadd -d http://get.opencsw.org/now
    # /opt/csw/bin/pkgutil -i ansible

.. _from_pacman:

Installing Ansible on Arch Linux
---------------------------------

通用仓库包含有 Ansible ，直接安装即可 ::

    $ pacman -S ansible

AUR 的 `ansible-git <https://aur.archlinux.org/packages/ansible-git>`_.  拥有一个 PKGBUILD 功能，可以直接从 GitHub 拉取数据。

参考 ArchWiki： `Ansible <https://wiki.archlinux.org/index.php/Ansible>`_ 。

.. _from_sbopkg:

Installing Ansible on Slackware Linux
-------------------------------------

Ansible 编译脚本的repository ： `SlackBuilds.org <https://slackbuilds.org/apps/ansible/>`_ .
编译安装参考： `sbopkg <https://sbopkg.org/>`_.

使用 Ansible 和所有依赖创建队列 ::

    # sqg -p ansible

从创建的队列文件构建和安装软件包（ 如 sbopkg 提示是否应使用队列或软件包，输入 Q ）::

    # sbopkg -k -i ansible

.. _from swupd:

Installing Ansible on Clear Linux
---------------------------------

Linux 发行包版本的软件包中默认带有 Ansible 及其依赖包 ::

    $ sudo swupd bundle-add sysadmin-hostmgmt

使用 swupd 工具包升级软件 ::

   $ sudo swupd update

.. _from_pip:

Installing Ansible with ``pip``
--------------------------------

Ansible 可以使用 Python 包管理器 ``pip`` 安装。 但 macOS 因为 ``openssl`` 协议要求的原因， ``pip`` 和 ``*nix`` 的使用和其它系统会有一些区别， pip 以模块的方式运行。 ( 英文原文： It should be noted that macOS requires a slightly different use of ``pip`` than ``*nix`` due to ``openssl`` requirements, therefore pip must be run as a module. ) 如果 ``pip`` 事先没有安装，使用如下命令安装 ::

    $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    $ python get-pip.py --user

安装 Ansible [1]_::

    $ pip install --user ansible

macOS 系统不需要需要 ``sudo`` 或者额外安装其它补丁包，只需要 ``pip`` 安装即可::

    $ python -m pip install --user ansible

如果希望安装开发版本::

    $ pip install --user git+https://github.com/ansible/ansible.git@devel

For macOS::

    $ python -m pip install --user git+https://github.com/ansible/ansible.git@devel

如果你使用的是 macOS Mavericks (10.9)， 编译的时候可能会有一些 warnging 。 建议额外声明如下编译变量::

    $ CFLAGS=-Qunused-arguments CPPFLAGS=-Qunused-arguments pip install --user ansible

如果希望使用 ``paramiko`` 插件或者模块依赖 ``paramiko``, 安装方式如下 [2]_::::

    $ pip install --user paramiko

For macOS::

    $ python -m pip install --user paramiko

Ansible 也可以安装在 Python 虚拟环境管理器 ``virtualenv`` 指定的环境中 ::

    $ python -m virtualenv ansible  # Create a virtualenv if one does not already exist
    $ source ansible/bin/activate   # Activate the virtual environment
    $ pip install ansible

如果想全局安装 Ansible ::

    $ sudo python get-pip.py
    $ sudo pip install ansible

.. note::

    需要注意的是 使用 ``sudo pip`` 是全局安装的模式。 由于 ``pip`` 无法与系统管理包器协调，所以如果使用 ``sudo`` 模式可能会改变系统环境，导致系统无法运行， macOS 系统尤其容易发生该问题。 如果你非专门人士，对系统全局文件功能完全了解前，建议使用 ``--user`` 参数。

.. note::

    旧版本的 ``pip``  默认网址是 http://pypi.python.org/simple, 现在已经不再维护。
    安装 Ansile 前请升级 ``pip`` 到最新版本。

    如果你已经安装的旧版本的 ``pip``， 升级文档可参考  <https://pip.pypa.io/en/stable/installing/#upgrading-pip>`_ .



.. _from_source:

从源码运行 Ansible (devel)
-----------------------------------

.. note::

    只有当你正在修改 Ansible 引擎或者尝试修改 Ansible 源码时，你才需要使用 ``devel`` 版本。 ``devel`` 分支会随时改变，功能也不稳定。

从源码编译安装 Ansible 也很容易， 不需要 ``root`` 权限，也不需要事先安装软件，更不需要保持后台运行或者设置数据库。

.. note::
   
   如果你希望使用 Ansible Tower 作为管理节点，不要使用源码编译安装。 请使用 OS 管理工具 ( 比如： ``apt`` or ``yum``or ``pip`` ) 安装稳定版.


源码安装: 

.. code-block:: bash

    $ git clone https://github.com/ansible/ansible.git
    $ cd ./ansible

``git``下载后 Ansible 后，设置环境变量:

Using Bash:

.. code-block:: bash

    $ source ./hacking/env-setup

Using Fish::

    $ source ./hacking/env-setup.fish

If you want to suppress spurious warnings/errors, use::

    $ source ./hacking/env-setup -q

请保证已经事先安装了 Python 包管理工具 ``pip``::

    $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    $ python get-pip.py --user

使用如下命令安装依赖组件 [1]_:

.. code-block:: bash

    $ pip install --user -r ./requirements.txt

如果希望更改 Ansible checkout的版本， 建议使用 pull-with-rebase 保留原始版本。

.. code-block:: bash

    $ git pull --rebase

.. code-block:: bash

    $ git pull --rebase #same as above
    $ git submodule update --init --recursive

一旦开始运行 env-setup 脚本，默认表示你使用的 inventory 仓库文件是 ``/etc/ansible/hosts`` . 当然你也可以参考这篇文件指定 inventory 仓库 ( :ref:`inventory` ) :

.. code-block:: bash

    $ echo "127.0.0.1" > ~/ansible_hosts
    $ export ANSIBLE_INVENTORY=~/ansible_hosts

关于 inventory 从这里可以了解到更多内容 :ref:`inventory` 。

执行第一条命令:

.. code-block:: bash

    $ ansible all -m ping --ask-pass

也可以尝试 "sudo make install".

.. _tagged_releases:

安装指定版本的 Ansible
-----------------------------------

如果想打包 Ansible 或者自己构建一个本地的包但又不想 git checkout, 可以从这个页面下载指定的版本包安装 `Ansible downloads <https://releases.ansible.com/ansible>`_ page.

这些包同样也放在 Ansible 的发行版仓库中 `git repository <https://github.com/ansible/ansible/releases>`_ 。

.. _shell_completion:

Ansible 命令行补全功能
--------------------------------

Ansible 从 2.9 版本开始支持命令行补全功能，但需要安装 ``argcomplete`` 插件。 ``argcomplete`` 完全支持 bash， 部分功能支持 zsh tcsh。 

你可以从 RedHat 的 EPEL 源直接安装 ``python-argcomplete``，或者从其它发行版的标准 OS repo 库安装。

更多安装配置信息请参考 `argcomplete documentation <https://argcomplete.readthedocs.io/en/latest/>`_.

Installing ``argcomplete`` on RHEL, CentOS, or Fedora
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

On Fedora:

.. code-block:: bash

    $ sudo dnf install python-argcomplete

On RHEL and CentOS:

.. code-block:: bash

    $ sudo yum install epel-release
    $ sudo yum install python-argcomplete


Installing ``argcomplete`` with ``apt``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    $ sudo apt install python-argcomplete


Installing ``argcomplete`` with ``pip``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    $ pip install argcomplete

Configuring ``argcomplete``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

有 2 种方式配置 ``argcomplete`` 使 Ansible 支持 shell 命令补全: 全局模式 ( globally ) 或者单个命令 ( per command )。

Globally
"""""""""

全局模式需要 Bash的版本是 4.2

.. code-block:: bash

    $ sudo activate-global-python-argcomplete

这条命令将生成 bash 补全文件到全局配置默认目录。 可以使用 ``--dest`` 指定位置。

Per command
"""""""""""

如果 bash 的版本不是 4.2，那必须独立声明注册每个脚本。

.. code-block:: bash

    $ eval $(register-python-argcomplete ansible)
    $ eval $(register-python-argcomplete ansible-config)
    $ eval $(register-python-argcomplete ansible-console)
    $ eval $(register-python-argcomplete ansible-doc)
    $ eval $(register-python-argcomplete ansible-galaxy)
    $ eval $(register-python-argcomplete ansible-inventory)
    $ eval $(register-python-argcomplete ansible-playbook)
    $ eval $(register-python-argcomplete ansible-pull)
    $ eval $(register-python-argcomplete ansible-vault)

如果希望永久有效，如上命令需要写入到环境变量文件，``~/.profile`` or ``~/.bash_profile`` .

``argcomplete`` with zsh or tcsh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

zsh 或者 tcsh 命令补全功能请参考 `argcomplete documentation <https://argcomplete.readthedocs.io/en/latest/>`_.

.. _getting_ansible:

Ansible on GitHub
-----------------

Asnible 的 GitHub 地址 `GitHub project <https://github.com/ansible/ansible>`_ 。 我们也可以在该地址提交 issue, bugs, 或者产品功能想法。 


.. seealso::

   :ref:`intro_adhoc`
       Examples of basic commands
   :ref:`working_with_playbooks`
       Learning ansible's configuration management language
   :ref:`installation_faqs`
       Ansible Installation related to FAQs
   `Mailing List <https://groups.google.com/group/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `irc.freenode.net <http://irc.freenode.net>`_
       #ansible IRC chat channel

.. [1] 如果你在 macOS 上安装 "pycrypto" 有问题 ，尝试指定 ``CC=clang sudo -E pip install pycrypto``
.. [2] ``paramiko`` was included in Ansible's ``requirements.txt`` prior to 2.8.
