.. _become:

******************************************
了解特权升级: become
******************************************

`Ansible` 使用现有的权限升级系统以 `root` 或指定用户的权限执行任务。 这个特性允许你使用不同于你的登录用户的用户来在远程主机执行任务，我们称之为 ``become``。 ``become`` 模块使用现有的权限升级工具来完成该项工作，比如： `sudo`, `su`, `pfexec`, `doas`, `pbrun`, `dzdo`, `ksu`, `runas`, `machinectl`等。

.. contents::
   :local:

使用 become
==============

你可以通过剧本、任务指令、连接变量或者命令行控制 ``become`` 的使用。如果你以多种方式设置特权提升，阅读了解 :ref:`general precedence rules<general_precedence_rules>`

Ansible 包含的所有 ``become`` 插件都可以在这里找到: :ref:`become_plugin_list`.

Become 指令介绍
-----------------

你可以在剧本或任务层级设置指令来控制 ``become``。 您可以通过设置连接变量来覆盖如前所述的设置，连接变量通常在主机之间会有所不同。 这些变量和指令是独立的。比如： ``become_user`` 和 ``become`` 是不一样的。

become
    ``yes`` 表示激活特权。

become_user    
    设置远程执行用户，这个账户通常和你当前使用的账户不同，因为需要 `become` 成要运行账户的权限。但这并不意味着设置 ``become: yes`` 等同于在主机系统级别做变更设置。默认值是 ``root``。[原文：set to user with desired privileges — the user you `become`, NOT the user you login as. Does NOT imply ``become: yes``, to allow it to be set at host level. Default value is ``root``.]

become_method
    (在剧本或任务级别)覆盖 ``ansible.cfg`` 中默认的方法。具体参见 :ref:`become_plugins`.

become_flags
    (在剧本或任务级别)允许为任务或角色使用特定的标志。 一种常见的用法是：当用户 ``SHELL`` 为 ``nologin``时，将用户更改为 ``nobody``。 该功能需 ``Ansible 2.2``+版本。

举个例子：当管理一个需要 ``root`` 权限的系统服务时，你的连接用户是非 ``root`` 用户，你可以使用 ``become_user`` 切换成(``root``) 用户权限:

.. code-block:: yaml

    - name: Ensure the httpd service is running
      service:
        name: httpd
        state: started
      become: yes

以 ``apache``  用户身份执行一条命令:

.. code-block:: yaml

    - name: Run a command as the apache user
      command: somecommand
      become: yes
      become_user: apache

当 ``shell`` 是 ``nologin``时，以 ``nobody`` 用户身份执行操作:

.. code-block:: yaml

    - name: Run a command as nobody
      command: somecommand
      become: yes
      become_method: su
      become_user: nobody
      become_flags: '-s /bin/sh'

执行 ``sudo`` 时需输入密码， ``ansible-playbook`` 使用 ``--ask-become-pass`` 参数 (或``-K`` 短命令参数的形式指定要求输入密码).


Become 连接变量
---------------------------

你可以为不同的受控节点或受控组定义不同的 ``become``  配置。你可以在 Inventory 中定义这些变量或者在像普通变量一样使用他们。

ansible_become
    等同 ``become`` 指令，定义是否使用提权功能

ansible_become_method
    使用哪种提权方式

ansible_become_user
    通过提权设置用户；但并不表示 ``ansible_become: yes``

ansible_become_password
    设置提权用户密码。 :ref:`playbooks_vault` 详细描述了如何在普通文本中定义密码。

举例： 如果你希望以 ``root``  用户在在 ``webserver`` 服务器运行变更，但你只能以 ``manager`` 连接远程节点 ，你可以使用如下 Inventory 的方式定义:

.. code-block:: text

    webserver ansible_user=manager ansible_become=yes

.. note::
    如上变量的定义方式适用于所有 ``become`` 插件，但同样也可以以该方式为其它插件设置变量。 具体请参考各插件对应文件，查看如何定义。 ``become`` 插件更详细信息请参考 :ref:`become_plugins`.


Become 命令行参数
---------------------------

--ask-become-pass, -K
    要求输入提权用户密码; 但并不意味着会使用 ``become`` 模块。需要注意的是，密码是针对本次执行命令的所有主机。

--become, -b
    调用 ``become`` 模块运行命令(不用输入密码)

--become-method=BECOME_METHOD
    提权方式，默认 sudo
    可用选项： [ sudo | su | pbrun | pfexec | doas | dzdo | ksu | runas | machinectl ]

--become-user=BECOME_USER
    run operations as this user (default=root), does not imply --become/-b
    提权运行的用户，默认 ``root``,但并不意味着使用 ``--become/-b``

become 的使用风险和局限性
===============================

尽管特权升级在大多数情况下都是直观的，但对其工作方式仍存在一些限制。 用户应注意这些事项，以免出现意外。

成为非特权用户的风险
--------------------------------------

Ansible 模块运行命令的原理： 通过将参数替换为模块文件，然后将文件复制到远程计算机，最后在该计算机上执行，

当不使用 ``become``, 或者 ``become_user = root``  或使用 ``root`` 用户连接远程主机时，模块文件的执行都很正常。但在前述这些特殊情况下使用时， Ansible 模块文件的创建需要预先判断用户是否有权限，或者提权用户只有可读权限时，就会有错误发生。

然而，如果连接用户和 ``become_user`` 提权后的用户均没有权限，模块文件是以 Ansible 连接用户编辑的，但文件需要 ``become`` 后的用户有可读权限。 这种情况下， 在 Ansible 模块执行期间，会让文件所有用户可读，一旦执行完毕，Ansible 会临时删除该文件。

传递给模块的任何参数本质上是敏感的，而客户端计算机是否安全你并不知道，则可能存在危险。

解决此问题的方法包括:

* 使用 `pipelining`，当管道开启后， Ansible 不在远程主机上保存临时文件，取而代之的是将模块放在远程主机的 Python 环境标准输入。管道对于 python 模块文件传输的场景不合适(比如: :ref:`copy <copy_module>`,
  :ref:`fetch <fetch_module>`, :ref:`template <template_module>`)，没有 Python 模块的环境也不行。
* 在受控机上安装 POSIX.1e 文件系统 acl 支持。如果远程主机临时目录是以

* Install POSIX.1e filesystem acl support on the POSIX acls 挂载的，并且远程 ``PATH`` 使用 :command:`setfacl` 作了设置。 Ansible 将使用 POSIX acls 对提权用户分享模块，而不用把文件权限设置为所有人可读。

* 避免 ``become`` 为没有权限的用户。当你 ``become`` 为 ``root`` 或者不使用 ``become``时，临时文件的权限是被 Unix 文件权限保护。从 Ansible 2.1+开始，如果你连接受控机使用的用户是``root``,然后 ``become`` 后的用户却没有权限，那么 Unix 文件权限系统依然是安全的。

.. warning:: 尽管 ``Solaris ZFS`` 文件系统使用 ``ACLs`` 权限系统，但 ``ACLs`` 却不支持 ``POSIX.1e`` 文件系统权限(其使用 NFSV4 ACLS 代替)。所以，Ansible 无法使用 ``ACLs`` 文件系统权限管理临时文件权限，因此，你需要回到 ``allow_world_readable_tmpfiles`` 的方式管理使用 ``ZFS`` 文件系统的受控主机文件权限。

.. versionchanged:: 2.1

Ansible ``become`` 的安全隐患很隐蔽。 从 Ansible 2.1开始， 当 ``become`` 涉及安全问题时，Ansible 默认引发错误。 如果你不使用管理或者 ``POSIX ACLs``, 你必须使用非特权用户连接，你必须使用 ``become`` 为一个非特权用户执行操作，并且你自己决定受控节点是否足够安全以便你可以设置权限为所有人可读，如果是，则可以打开 :file:`ansible.cfg` 中的 ``allow_world_readable_tmpfiles`` 开关。 该设置为将不再报 ``error``， 而是像 2.1 版本以前的方式执行，只是报 ``warning`` 后继续执行命令。


并非所有连接插件都支持
---------------------------------------

使用的连接插件必须支持提权方法。大部分的插件如果不支持 ``become`` 会有告警。一些刚直接忽略直接使用 ``root`` 执行(比如： jail, chroot, etc).

每台主机只能启用一种办法
---------------------------------------

方法不能混用。你不能使用 ``sudo /bin/su -`` 来 ``become`` 一个用户，你执行命令的用户必须是在 sudo 权限白名单中或者可以直接直接 su 命令来执行（ pbrun, pfexec 或其它方法同样的要求）

Privilege escalation must be general
------------------------------------

You cannot limit privilege escalation permissions to certain commands.
Ansible does not always
use a specific command to do something but runs modules (code) from
a temporary file name which changes every time.  If you have '/sbin/service'
or '/bin/chmod' as the allowed commands this will fail with ansible as those
paths won't match with the temporary file that Ansible creates to run the
module. If you have security rules that constrain your sudo/pbrun/doas environment
to running specific command paths only, use Ansible from a special account that
does not have this constraint, or use :ref:`ansible_tower` to manage indirect access to SSH credentials.

May not access environment variables populated by pamd_systemd
--------------------------------------------------------------

For most Linux distributions using ``systemd`` as their init, the default
methods used by ``become`` do not open a new "session", in the sense of
systemd. Because the ``pam_systemd`` module will not fully initialize a new
session, you might have surprises compared to a normal session opened through
ssh: some environment variables set by ``pam_systemd``, most notably
``XDG_RUNTIME_DIR``, are not populated for the new user and instead inherited
or just emptied.

This might cause trouble when trying to invoke systemd commands that depend on
``XDG_RUNTIME_DIR`` to access the bus:

.. code-block:: console

   $ echo $XDG_RUNTIME_DIR

   $ systemctl --user status
   Failed to connect to bus: Permission denied

To force ``become`` to open a new systemd session that goes through
``pam_systemd``, you can use ``become_method: machinectl``.

For more information, see `this systemd issue
<https://github.com/systemd/systemd/issues/825#issuecomment-127917622>`_.

.. _become_network:

Become and network automation
=============================

As of version 2.6, Ansible supports ``become`` for privilege escalation (entering ``enable`` mode or privileged EXEC mode) on all :ref:`Ansible-maintained platforms<network_supported>` that support ``enable`` mode. Using ``become`` replaces the ``authorize`` and ``auth_pass`` options in a ``provider`` dictionary.

You must set the connection type to either ``connection: network_cli`` or ``connection: httpapi`` to use ``become`` for privilege escalation on network devices. Check the :ref:`platform_options` and :ref:`network_modules` documentation for details.

You can use escalated privileges on only the specific tasks that need them, on an entire play, or on all plays. Adding ``become: yes`` and ``become_method: enable`` instructs Ansible to enter ``enable`` mode before executing the task, play, or playbook where those parameters are set.

If you see this error message, the task that generated it requires ``enable`` mode to succeed:

.. code-block:: console

   Invalid input (privileged mode required)

To set ``enable`` mode for a specific task, add ``become`` at the task level:

.. code-block:: yaml

   - name: Gather facts (eos)
     eos_facts:
       gather_subset:
         - "!hardware"
     become: yes
     become_method: enable

To set enable mode for all tasks in a single play, add ``become`` at the play level:

.. code-block:: yaml

   - hosts: eos-switches
     become: yes
     become_method: enable
     tasks:
       - name: Gather facts (eos)
         eos_facts:
           gather_subset:
             - "!hardware"

Setting enable mode for all tasks
---------------------------------

Often you wish for all tasks in all plays to run using privilege mode, that is best achieved by using ``group_vars``:

**group_vars/eos.yml**

.. code-block:: yaml

   ansible_connection: network_cli
   ansible_network_os: eos
   ansible_user: myuser
   ansible_become: yes
   ansible_become_method: enable

Passwords for enable mode
^^^^^^^^^^^^^^^^^^^^^^^^^

If you need a password to enter ``enable`` mode, you can specify it in one of two ways:

* providing the :option:`--ask-become-pass <ansible-playbook --ask-become-pass>` command line option
* setting the ``ansible_become_password`` connection variable

.. warning::

   As a reminder passwords should never be stored in plain text. For information on encrypting your passwords and other secrets with Ansible Vault, see :ref:`vault`.

authorize and auth_pass
-----------------------

Ansible still supports ``enable`` mode with ``connection: local`` for legacy network playbooks. To enter ``enable`` mode with ``connection: local``, use the module options ``authorize`` and ``auth_pass``:

.. code-block:: yaml

   - hosts: eos-switches
     ansible_connection: local
     tasks:
       - name: Gather facts (eos)
         eos_facts:
           gather_subset:
             - "!hardware"
         provider:
           authorize: yes
           auth_pass: " {{ secret_auth_pass }}"

We recommend updating your playbooks to use ``become`` for network-device ``enable`` mode consistently. The use of ``authorize`` and of ``provider`` dictionaries will be deprecated in future. Check the :ref:`platform_options` and :ref:`network_modules` documentation for details.

.. _become_windows:

Become and Windows
==================

Since Ansible 2.3, ``become`` can be used on Windows hosts through the
``runas`` method. Become on Windows uses the same inventory setup and
invocation arguments as ``become`` on a non-Windows host, so the setup and
variable names are the same as what is defined in this document.

While ``become`` can be used to assume the identity of another user, there are other uses for
it with Windows hosts. One important use is to bypass some of the
limitations that are imposed when running on WinRM, such as constrained network
delegation or accessing forbidden system calls like the WUA API. You can use
``become`` with the same user as ``ansible_user`` to bypass these limitations
and run commands that are not normally accessible in a WinRM session.

Administrative rights
---------------------

Many tasks in Windows require administrative privileges to complete. When using
the ``runas`` become method, Ansible will attempt to run the module with the
full privileges that are available to the remote user. If it fails to elevate
the user token, it will continue to use the limited token during execution.

A user must have the ``SeDebugPrivilege`` to run a become process with elevated
privileges. This privilege is assigned to Administrators by default. If the
debug privilege is not available, the become process will run with a limited
set of privileges and groups.

To determine the type of token that Ansible was able to get, run the following
task:

.. code-block:: yaml

    - win_whoami:
      become: yes

The output will look something similar to the below:

.. code-block:: ansible-output

    ok: [windows] => {
        "account": {
            "account_name": "vagrant-domain",
            "domain_name": "DOMAIN",
            "sid": "S-1-5-21-3088887838-4058132883-1884671576-1105",
            "type": "User"
        },
        "authentication_package": "Kerberos",
        "changed": false,
        "dns_domain_name": "DOMAIN.LOCAL",
        "groups": [
            {
                "account_name": "Administrators",
                "attributes": [
                    "Mandatory",
                    "Enabled by default",
                    "Enabled",
                    "Owner"
                ],
                "domain_name": "BUILTIN",
                "sid": "S-1-5-32-544",
                "type": "Alias"
            },
            {
                "account_name": "INTERACTIVE",
                "attributes": [
                    "Mandatory",
                    "Enabled by default",
                    "Enabled"
                ],
                "domain_name": "NT AUTHORITY",
                "sid": "S-1-5-4",
                "type": "WellKnownGroup"
            },
        ],
        "impersonation_level": "SecurityAnonymous",
        "label": {
            "account_name": "High Mandatory Level",
            "domain_name": "Mandatory Label",
            "sid": "S-1-16-12288",
            "type": "Label"
        },
        "login_domain": "DOMAIN",
        "login_time": "2018-11-18T20:35:01.9696884+00:00",
        "logon_id": 114196830,
        "logon_server": "DC01",
        "logon_type": "Interactive",
        "privileges": {
            "SeBackupPrivilege": "disabled",
            "SeChangeNotifyPrivilege": "enabled-by-default",
            "SeCreateGlobalPrivilege": "enabled-by-default",
            "SeCreatePagefilePrivilege": "disabled",
            "SeCreateSymbolicLinkPrivilege": "disabled",
            "SeDebugPrivilege": "enabled",
            "SeDelegateSessionUserImpersonatePrivilege": "disabled",
            "SeImpersonatePrivilege": "enabled-by-default",
            "SeIncreaseBasePriorityPrivilege": "disabled",
            "SeIncreaseQuotaPrivilege": "disabled",
            "SeIncreaseWorkingSetPrivilege": "disabled",
            "SeLoadDriverPrivilege": "disabled",
            "SeManageVolumePrivilege": "disabled",
            "SeProfileSingleProcessPrivilege": "disabled",
            "SeRemoteShutdownPrivilege": "disabled",
            "SeRestorePrivilege": "disabled",
            "SeSecurityPrivilege": "disabled",
            "SeShutdownPrivilege": "disabled",
            "SeSystemEnvironmentPrivilege": "disabled",
            "SeSystemProfilePrivilege": "disabled",
            "SeSystemtimePrivilege": "disabled",
            "SeTakeOwnershipPrivilege": "disabled",
            "SeTimeZonePrivilege": "disabled",
            "SeUndockPrivilege": "disabled"
        },
        "rights": [
            "SeNetworkLogonRight",
            "SeBatchLogonRight",
            "SeInteractiveLogonRight",
            "SeRemoteInteractiveLogonRight"
        ],
        "token_type": "TokenPrimary",
        "upn": "vagrant-domain@DOMAIN.LOCAL",
        "user_flags": []
    }

Under the ``label`` key, the ``account_name`` entry determines whether the user
has Administrative rights. Here are the labels that can be returned and what
they represent:

* ``Medium``: Ansible failed to get an elevated token and ran under a limited
  token. Only a subset of the privileges assigned to user are available during
  the module execution and the user does not have administrative rights.

* ``High``: An elevated token was used and all the privileges assigned to the
  user are available during the module execution.

* ``System``: The ``NT AUTHORITY\System`` account is used and has the highest
  level of privileges available.

The output will also show the list of privileges that have been granted to the
user. When the privilege value is ``disabled``, the privilege is assigned to
the logon token but has not been enabled. In most scenarios these privileges
are automatically enabled when required.

If running on a version of Ansible that is older than 2.5 or the normal
``runas`` escalation process fails, an elevated token can be retrieved by:

* Set the ``become_user`` to ``System`` which has full control over the
  operating system.

* Grant ``SeTcbPrivilege`` to the user Ansible connects with on
  WinRM. ``SeTcbPrivilege`` is a high-level privilege that grants
  full control over the operating system. No user is given this privilege by
  default, and care should be taken if you grant this privilege to a user or group.
  For more information on this privilege, please see
  `Act as part of the operating system <https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn221957(v=ws.11)>`_.
  You can use the below task to set this privilege on a Windows host:

  .. code-block:: yaml

    - name: grant the ansible user the SeTcbPrivilege right
      win_user_right:
        name: SeTcbPrivilege
        users: '{{ansible_user}}'
        action: add

* Turn UAC off on the host and reboot before trying to become the user. UAC is
  a security protocol that is designed to run accounts with the
  ``least privilege`` principle. You can turn UAC off by running the following
  tasks:

  .. code-block:: yaml

    - name: turn UAC off
      win_regedit:
        path: HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system
        name: EnableLUA
        data: 0
        type: dword
        state: present
      register: uac_result

    - name: reboot after disabling UAC
      win_reboot:
      when: uac_result is changed

.. Note:: Granting the ``SeTcbPrivilege`` or turning UAC off can cause Windows
    security vulnerabilities and care should be given if these steps are taken.

Local service accounts
----------------------

Prior to Ansible version 2.5, ``become`` only worked on Windows with a local or domain
user account. Local service accounts like ``System`` or ``NetworkService``
could not be used as ``become_user`` in these older versions. This restriction
has been lifted since the 2.5 release of Ansible. The three service accounts
that can be set under ``become_user`` are:

* System
* NetworkService
* LocalService

Because local service accounts do not have passwords, the
``ansible_become_password`` parameter is not required and is ignored if
specified.

Become without setting a password
---------------------------------

As of Ansible 2.8, ``become`` can be used to become a Windows local or domain account
without requiring a password for that account. For this method to work, the
following requirements must be met:

* The connection user has the ``SeDebugPrivilege`` privilege assigned
* The connection user is part of the ``BUILTIN\Administrators`` group
* The ``become_user`` has either the ``SeBatchLogonRight`` or ``SeNetworkLogonRight`` user right

Using become without a password is achieved in one of two different methods:

* Duplicating an existing logon session's token if the account is already logged on
* Using S4U to generate a logon token that is valid on the remote host only

In the first scenario, the become process is spawned from another logon of that
user account. This could be an existing RDP logon, console logon, but this is
not guaranteed to occur all the time. This is similar to the
``Run only when user is logged on`` option for a Scheduled Task.

In the case where another logon of the become account does not exist, S4U is
used to create a new logon and run the module through that. This is similar to
the ``Run whether user is logged on or not`` with the ``Do not store password``
option for a Scheduled Task. In this scenario, the become process will not be
able to access any network resources like a normal WinRM process.

To make a distinction between using become with no password and becoming an
account that has no password make sure to keep ``ansible_become_password`` as
undefined or set ``ansible_become_password:``.

.. Note:: Because there are no guarantees an existing token will exist for a
  user when Ansible runs, there's a high change the become process will only
  have access to local resources. Use become with a password if the task needs
  to access network resources

Accounts without a password
---------------------------

.. Warning:: As a general security best practice, you should avoid allowing accounts without passwords.

Ansible can be used to become a Windows account that does not have a password (like the
``Guest`` account). To become an account without a password, set up the
variables like normal but set ``ansible_become_password: ''``.

Before become can work on an account like this, the local policy
`Accounts: Limit local account use of blank passwords to console logon only <https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj852174(v=ws.11)>`_
must be disabled. This can either be done through a Group Policy Object (GPO)
or with this Ansible task:

.. code-block:: yaml

   - name: allow blank password on become
     win_regedit:
       path: HKLM:\SYSTEM\CurrentControlSet\Control\Lsa
       name: LimitBlankPasswordUse
       data: 0
       type: dword
       state: present

.. Note:: This is only for accounts that do not have a password. You still need
    to set the account's password under ``ansible_become_password`` if the
    become_user has a password.

Become flags for Windows
------------------------

Ansible 2.5 added the ``become_flags`` parameter to the ``runas`` become method.
This parameter can be set using the ``become_flags`` task directive or set in
Ansible's configuration using ``ansible_become_flags``. The two valid values
that are initially supported for this parameter are ``logon_type`` and
``logon_flags``.

.. Note:: These flags should only be set when becoming a normal user account, not a local service account like LocalSystem.

The key ``logon_type`` sets the type of logon operation to perform. The value
can be set to one of the following:

* ``interactive``: The default logon type. The process will be run under a
  context that is the same as when running a process locally. This bypasses all
  WinRM restrictions and is the recommended method to use.

* ``batch``: Runs the process under a batch context that is similar to a
  scheduled task with a password set. This should bypass most WinRM
  restrictions and is useful if the ``become_user`` is not allowed to log on
  interactively.

* ``new_credentials``: Runs under the same credentials as the calling user, but
  outbound connections are run under the context of the ``become_user`` and
  ``become_password``, similar to ``runas.exe /netonly``. The ``logon_flags``
  flag should also be set to ``netcredentials_only``. Use this flag if
  the process needs to access a network resource (like an SMB share) using a
  different set of credentials.

* ``network``: Runs the process under a network context without any cached
  credentials. This results in the same type of logon session as running a
  normal WinRM process without credential delegation, and operates under the same
  restrictions.

* ``network_cleartext``: Like the ``network`` logon type, but instead caches
  the credentials so it can access network resources. This is the same type of
  logon session as running a normal WinRM process with credential delegation.

For more information, see
`dwLogonType <https://docs.microsoft.com/en-gb/windows/desktop/api/winbase/nf-winbase-logonusera>`_.

The ``logon_flags`` key specifies how Windows will log the user on when creating
the new process. The value can be set to none or multiple of the following:

* ``with_profile``: The default logon flag set. The process will load the
  user's profile in the ``HKEY_USERS`` registry key to ``HKEY_CURRENT_USER``.

* ``netcredentials_only``: The process will use the same token as the caller
  but will use the ``become_user`` and ``become_password`` when accessing a remote
  resource. This is useful in inter-domain scenarios where there is no trust
  relationship, and should be used with the ``new_credentials`` ``logon_type``.

By default ``logon_flags=with_profile`` is set, if the profile should not be
loaded set ``logon_flags=`` or if the profile should be loaded with
``netcredentials_only``, set ``logon_flags=with_profile,netcredentials_only``.

For more information, see `dwLogonFlags <https://docs.microsoft.com/en-gb/windows/desktop/api/winbase/nf-winbase-createprocesswithtokenw>`_.

Here are some examples of how to use ``become_flags`` with Windows tasks:

.. code-block:: yaml

  - name: copy a file from a fileshare with custom credentials
    win_copy:
      src: \\server\share\data\file.txt
      dest: C:\temp\file.txt
      remote_src: yes
    vars:
      ansible_become: yes
      ansible_become_method: runas
      ansible_become_user: DOMAIN\user
      ansible_become_password: Password01
      ansible_become_flags: logon_type=new_credentials logon_flags=netcredentials_only

  - name: run a command under a batch logon
    win_whoami:
    become: yes
    become_flags: logon_type=batch

  - name: run a command and not load the user profile
    win_whomai:
    become: yes
    become_flags: logon_flags=


Limitations of become on Windows
--------------------------------

* Running a task with ``async`` and ``become`` on Windows Server 2008, 2008 R2
  and Windows 7 only works when using Ansible 2.7 or newer.

* By default, the become user logs on with an interactive session, so it must
  have the right to do so on the Windows host. If it does not inherit the
  ``SeAllowLogOnLocally`` privilege or inherits the ``SeDenyLogOnLocally``
  privilege, the become process will fail. Either add the privilege or set the
  ``logon_type`` flag to change the logon type used.

* Prior to Ansible version 2.3, become only worked when
  ``ansible_winrm_transport`` was either ``basic`` or ``credssp``. This
  restriction has been lifted since the 2.4 release of Ansible for all hosts
  except Windows Server 2008 (non R2 version).

* The Secondary Logon service ``seclogon`` must be running to use ``ansible_become_method: runas``

.. seealso::

   `Mailing List <https://groups.google.com/forum/#!forum/ansible-project>`_
       Questions? Help? Ideas?  Stop by the list on Google Groups
   `webchat.freenode.net <https://webchat.freenode.net>`_
       #ansible IRC chat channel
