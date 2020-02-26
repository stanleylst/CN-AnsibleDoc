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

To install the newest version, you may need to unmask the Ansible package prior to emerging:

.. code-block:: bash

    $ echo 'app-admin/ansible' >> /etc/portage/package.accept_keywords

Installing Ansible on FreeBSD
-----------------------------

Though Ansible works with both Python 2 and 3 versions, FreeBSD has different packages for each Python version.
So to install you can use:

.. code-block:: bash

    $ sudo pkg install py27-ansible

or:

.. code-block:: bash

    $ sudo pkg install py36-ansible


You may also wish to install from ports, run:

.. code-block:: bash

    $ sudo make -C /usr/ports/sysutils/ansible install

You can also choose a specific version, i.e  ``ansible25``.

Older versions of FreeBSD worked with something like this (substitute for your choice of package manager):

.. code-block:: bash

    $ sudo pkg install ansible

.. _on_macos:

Installing Ansible on macOS
---------------------------

The preferred way to install Ansible on a Mac is with ``pip``.

The instructions can be found in :ref:`from_pip`. If you are running macOS version 10.12 or older, then you should upgrade to the latest ``pip`` to connect to the Python Package Index securely. It should be noted that pip must be run as a module on macOS, and the linked ``pip`` instructions will show you how to do that.

.. _from_pkgutil:

Installing Ansible on Solaris
-----------------------------

Ansible is available for Solaris as `SysV package from OpenCSW <https://www.opencsw.org/packages/ansible/>`_.

.. code-block:: bash

    # pkgadd -d http://get.opencsw.org/now
    # /opt/csw/bin/pkgutil -i ansible

.. _from_pacman:

Installing Ansible on Arch Linux
---------------------------------

Ansible is available in the Community repository::

    $ pacman -S ansible

The AUR has a PKGBUILD for pulling directly from GitHub called `ansible-git <https://aur.archlinux.org/packages/ansible-git>`_.

Also see the `Ansible <https://wiki.archlinux.org/index.php/Ansible>`_ page on the ArchWiki.

.. _from_sbopkg:

Installing Ansible on Slackware Linux
-------------------------------------

Ansible build script is available in the `SlackBuilds.org <https://slackbuilds.org/apps/ansible/>`_ repository.
Can be built and installed using `sbopkg <https://sbopkg.org/>`_.

Create queue with Ansible and all dependencies::

    # sqg -p ansible

Build and install packages from a created queuefile (answer Q for question if sbopkg should use queue or package)::

    # sbopkg -k -i ansible

.. _from swupd:

Installing Ansible on Clear Linux
---------------------------------

Ansible and its dependencies are available as part of the sysadmin host management bundle::

    $ sudo swupd bundle-add sysadmin-hostmgmt

Update of the software will be managed by the swupd tool::

   $ sudo swupd update

.. _from_pip:

Installing Ansible with ``pip``
--------------------------------

Ansible can be installed with ``pip``, the Python package manager. It should be noted that macOS requires a slightly different use of ``pip`` than ``*nix`` due to ``openssl`` requirements, therefore pip must be run as a module.  If ``pip`` isn't already available on your system of Python, run the following commands to install it::

    $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    $ python get-pip.py --user

Then install Ansible [1]_::

    $ pip install --user ansible

For macOS, there is no need to use ``sudo`` or install additional fixes, simply access the Python module namespace for ``pip``::

    $ python -m pip install --user ansible

Or if you are looking for the development version::

    $ pip install --user git+https://github.com/ansible/ansible.git@devel

For macOS::

    $ python -m pip install --user git+https://github.com/ansible/ansible.git@devel

If you are installing on macOS Mavericks (10.9), you may encounter some noise from your compiler. A workaround is to do the following::

    $ CFLAGS=-Qunused-arguments CPPFLAGS=-Qunused-arguments pip install --user ansible

In order to use the ``paramiko`` connection plugin or modules that require ``paramiko``, install the required module [2]_::

    $ pip install --user paramiko

For macOS::

    $ python -m pip install --user paramiko

Ansible can also be installed inside a new or existing ``virtualenv``::

    $ python -m virtualenv ansible  # Create a virtualenv if one does not already exist
    $ source ansible/bin/activate   # Activate the virtual environment
    $ pip install ansible

If you wish to install Ansible globally, run the following commands::

    $ sudo python get-pip.py
    $ sudo pip install ansible

.. note::

    Running ``pip`` with ``sudo`` will make global changes to the system. Since ``pip`` does not coordinate with system package managers, it could make changes to your system that leaves it in an inconsistent or non-functioning state. This is particularly true for macOS. Installing with ``--user`` is recommended unless you understand fully the implications of modifying global files on the system.

.. note::

    Older versions of ``pip`` default to http://pypi.python.org/simple, which no longer works.
    Please make sure you have the latest version of ``pip`` before installing Ansible.
    If you have an older version of ``pip`` installed, you can upgrade by following `pip's upgrade instructions <https://pip.pypa.io/en/stable/installing/#upgrading-pip>`_ .



.. _from_source:

Running Ansible from source (devel)
-----------------------------------

.. note::

	You should only run Ansible from ``devel`` if you are modifying the Ansible engine, or trying out features under development. This is a rapidly changing source of code and can become unstable at any point.

Ansible is easy to run from source. You do not need ``root`` permissions
to use it and there is no software to actually install. No daemons
or database setup are required.

.. note::

   If you want to use Ansible Tower as the control node, do not use a source installation of Ansible. Please use an OS package manager (like ``apt`` or ``yum``) or ``pip`` to install a stable version.


To install from source, clone the Ansible git repository:

.. code-block:: bash

    $ git clone https://github.com/ansible/ansible.git
    $ cd ./ansible

Once ``git`` has cloned the Ansible repository, setup the Ansible environment:

Using Bash:

.. code-block:: bash

    $ source ./hacking/env-setup

Using Fish::

    $ source ./hacking/env-setup.fish

If you want to suppress spurious warnings/errors, use::

    $ source ./hacking/env-setup -q

If you don't have ``pip`` installed in your version of Python, install it::

    $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    $ python get-pip.py --user

Ansible also uses the following Python modules that need to be installed [1]_:

.. code-block:: bash

    $ pip install --user -r ./requirements.txt

To update Ansible checkouts, use pull-with-rebase so any local changes are replayed.

.. code-block:: bash

    $ git pull --rebase

.. code-block:: bash

    $ git pull --rebase #same as above
    $ git submodule update --init --recursive

Once running the env-setup script you'll be running from checkout and the default inventory file
will be ``/etc/ansible/hosts``. You can optionally specify an inventory file (see :ref:`inventory`)
other than ``/etc/ansible/hosts``:

.. code-block:: bash

    $ echo "127.0.0.1" > ~/ansible_hosts
    $ export ANSIBLE_INVENTORY=~/ansible_hosts

You can read more about the inventory file at :ref:`inventory`.

Now let's test things with a ping command:

.. code-block:: bash

    $ ansible all -m ping --ask-pass

You can also use "sudo make install".

.. _tagged_releases:

Finding tarballs of tagged releases
-----------------------------------

Packaging Ansible or wanting to build a local package yourself, but don't want to do a git checkout?  Tarballs of releases are available on the `Ansible downloads <https://releases.ansible.com/ansible>`_ page.

These releases are also tagged in the `git repository <https://github.com/ansible/ansible/releases>`_ with the release version.


.. _shell_completion:

Ansible command shell completion
--------------------------------

As of Ansible 2.9, shell completion of the Ansible command line utilities is available and provided through an optional dependency
called ``argcomplete``. ``argcomplete`` supports bash, and has limited support for zsh and tcsh.

You can install ``python-argcomplete`` from EPEL on Red Hat Enterprise based distributions, and or from the standard OS repositories for many other distributions.

For more information about installing and configuration see the `argcomplete documentation <https://argcomplete.readthedocs.io/en/latest/>`_.

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

There are 2 ways to configure ``argcomplete`` to allow shell completion of the Ansible command line utilities: globally or per command.

Globally
"""""""""

Global completion requires bash 4.2.

.. code-block:: bash

    $ sudo activate-global-python-argcomplete

This will write a bash completion file to a global location. Use ``--dest`` to change the location.

Per command
"""""""""""

If you do not have bash 4.2, you must register each script independently.

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

You should place the above commands into your shells profile file such as ``~/.profile`` or ``~/.bash_profile``.

``argcomplete`` with zsh or tcsh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

See the `argcomplete documentation <https://argcomplete.readthedocs.io/en/latest/>`_.

.. _getting_ansible:

Ansible on GitHub
-----------------

You may also wish to follow the `GitHub project <https://github.com/ansible/ansible>`_ if
you have a GitHub account. This is also where we keep the issue tracker for sharing
bugs and feature ideas.


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

.. [1] If you have issues with the "pycrypto" package install on macOS, then you may need to try ``CC=clang sudo -E pip install pycrypto``.
.. [2] ``paramiko`` was included in Ansible's ``requirements.txt`` prior to 2.8.
