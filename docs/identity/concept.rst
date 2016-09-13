概念
==================

（编者按：这里介绍概念的时候直接采用英文名词，后面文档中涉及相关名词也直接采用英文，方便大家和系统中的名词直接对应上。）

**Authentication**

 确认用户身份的流程。为确认一个请求，OpenStack Identity 会验证用户提供的一组凭证。初始化时，凭证是用户名和密码，或者用户名和 API 密钥， 当 OpenStack Identity 验证用户凭证成功后，则发布一个 token。用户在接下来的请求中使用该 token 认证。

**Credentials**

 用来确认用户身份的数据。例如，用户名和密码，用户名和 API key，或者由身份服务发布的验证 token。

**Domain**

 一个 OpenStack Identity v3 版本的实体。Domain 是一组 User 和 Project 的集合，用来定义被管理身份实体的管理边界。Domain 可以用来表示一个个体，公司或者一个拥有操作权限的空间。它们将管理活动直接暴露给系统用户。用户可以被授权整个 Domain 的管理 Role。一个 Domain 管理员可以创建该 Domain 内的 Project,User 和 Group，并可以给该 Domain 的 User 和 Group 指定 Role。

**Endpoint**

 一个网络可访问的地址，通常为 URL，用来访问对应的服务。如果使用模板扩展，我们可以创建一个 Endpoint 模板用来表示该区域所有可以消费的服务。

**Group**

 一个 OpenStack Identity v3 版本的实体。Group 是属于一个 Domain 中的 User 的集合。一个 Group Role， 对 Domain 或者 Project 授权，相当于给所有该 Group 的 User 授权。向该组添加和删除 User 都会改变该 User 的 Role 和对相关 Domain 或 Group 的认证。

**OpenStackClient**

 包括 Idenity API 在内的多个 OpenStack 服务的命令行接口。例如，用户可以执行 ``openstack service create`` 和 ``openstack endpoint create`` 命令在 OpenStack 安装过程中注册服务。

**Project**

 一个用来组织和隔离资源和身份对象的容器。根据服务操作不同，一个 Project 可能代表一个顾客、账户、组织或者租户。

**Region**

 一个 OpenStack Identity v3版本的实体。表示 OpenStack 部署中的通用分隔。我们可以将 0 到多个子 Region 关联到某个 Region 来构建一个树形结构。虽然 Region 并没有一个明显的地理含义，部署时可以给一个很地理化的名字，如 ``us-east``。

**Role**

 用来表示一组用户权限可以执行一组特定操作。身份服务为用户生成一个包含 Role 列表的 token， 当一个用户调用某服务，该服务解析用户 role 列表，然后确认每个 role 可以执行的操作和管理的资源。

**Service**

 一个 OpenStack Service，如计算（Nova），对象存储（swift），或镜像服务（glance），提供一到多个 endpoint，用户通过这些接口访问资源，执行操作。

**Token**

 一个由字母、数字组成文本字符串用来访问 OpenStack API 和资源。 Token 可能在任何时候被回收，且只在一段时间内有效。OpenStack 目前只支持基于 token 的认证，并计划支持更多的协议。OpenStack Identity 是一个集成服务，并不致力于成为一个完整的身份存储和管理解决方案。

**User**

 使用 OpenStack 云服务的人、系统、或者服务的数字表达。身份服务通过声明发起请求的 User 来验证该请求。 User 有一个 login 方法，并可以通过被赋予的 token 来访问资源。User 可以直接赋予特定的 Project，并拥有属于该 Project 的权限。

用户管理
-----------
身份服务用户管理例子：

* 创建一个 User ``alice``：

  .. code:: shell

	    $ openstack user create --password-prompt --email alice@example.com alice

  
* 创建一个 Project ``acme``：

  .. code:: shell

	    $ openstack project create acme


* 创建一个 Domain ``emea``：

  .. code:: shell

	    $ openstack --os-identity-api-version=3 domain creat emea


* 创建一个 Role ``compute-user``：

  .. code:: shell

	    $ openstack role create compute-user


.. note ::	    

  各独立服务赋予 Role 意义， 通过 Role 和服务支持的操作映射的方式来限制或授权该 Role 对应用户的访问。 Role 访问权限通常配置在服务的 ``policy.json`` 文件。例如，如果要限制 ``compute-user`` 对计算的访问，修改 ``policy.json`` 文件来控制该 Role 对计算的操作。


身份服务将一个 Project 和一个 Role 指定给一个 User。你可以在 Project ``acme`` 将 Role ``compute-user`` 指定给 User ``alice``：

.. code:: shell

  $ openstack role add --project acme --user alice compute-user


一个用户可以在不同 Project 中拥有不同 Role。例如， Alice 也可以有 Project ``Cyberdyne`` 的 Role ``admin``。一个用户也可以拥有同一个 Project 中的多个 Role。

文件 ``/etc/[SERVICE_CODENAME]/plicy.json`` 控制用户可以对该服务执行的任务。例如， ``/etc/nova/policy.json`` 文件描述了计算服务的访问策略， ``/etc/glance/policy.json`` 文件描述了镜像服务的访问策略， ``/etc/keystone/policy.json`` 文件描述了身份服务的访问策略。

计算、身份、和镜像服务里默认的 ``policy.json`` 文件只能区分 ``admin`` Role。任何 Project 中的任何 User 都可以访问不需要 ``admin`` Role 的所有操作。

如果要限制 User 访问某些操作，例如在计算服务中，我们需要在身份服务中创建一个 Role，然后修改 ``/etc/nova/policy.json`` 来限制只有此 Role 才能访问部分计算操作。

例如， ``/etc/cinder/policy.json`` 中下面的配置并没有限制谁能创建卷：

.. code:: json

	  "volume:create": "",


如果 User 在一个 Project 中拥有任何 Role，他就可以在该 Project 中创建卷。

现在要限制创建卷的操作给 User 拥有 Role ``compute-user``，我们添加 ``"role:compute-user"`` ：

.. code:: json

	  "volume:create": "role:compute-user",


如果要限制所有的计算服务请求需要为此 Role，结果文件类似于：
	  
.. code:: json

  {
   "admin_or_owner": "role:admin or project_id:%(project_id)s",
   "default": "rule:admin_or_owner",
   "compute:create": "role:compute-user",
   "compute:create:attach_network": "role:compute-user",
   "compute:create:attach_volume": "role:compute-user",
   "compute:get_all": "role:compute-user",
   "compute:unlock_override": "rule:admin_api",
   "admin_api": "role:admin",
   "compute_extension:accounts": "rule:admin_api",
   "compute_extension:admin_actions": "rule:admin_api",
   "compute_extension:admin_actions:pause": "rule:admin_or_owner",
   "compute_extension:admin_actions:unpause": "rule:admin_or_owner",
   "compute_extension:admin_actions:suspend": "rule:admin_or_owner",
   "compute_extension:admin_actions:resume": "rule:admin_or_owner",
   "compute_extension:admin_actions:lock": "rule:admin_or_owner",
   "compute_extension:admin_actions:unlock": "rule:admin_or_owner",
   "compute_extension:admin_actions:resetNetwork": "rule:admin_api",
   "compute_extension:admin_actions:injectNetworkInfo": "rule:admin_api",
   "compute_extension:admin_actions:createBackup": "rule:admin_or_owner",
   "compute_extension:admin_actions:migrateLive": "rule:admin_api",
   "compute_extension:admin_actions:migrate": "rule:admin_api",
   "compute_extension:aggregates": "rule:admin_api",
   "compute_extension:certificates": "role:compute-user",
   "compute_extension:cloudpipe": "rule:admin_api",
   "compute_extension:console_output": "role:compute-user",
   "compute_extension:consoles": "role:compute-user",
   "compute_extension:createserverext": "role:compute-user",
   "compute_extension:deferred_delete": "role:compute-user",
   "compute_extension:disk_config": "role:compute-user",
   "compute_extension:evacuate": "rule:admin_api",
   "compute_extension:extended_server_attributes": "rule:admin_api",
   "compute_extension:extended_status": "role:compute-user",
   "compute_extension:flavorextradata": "role:compute-user",
   "compute_extension:flavorextraspecs": "role:compute-user",
   "compute_extension:flavormanage": "rule:admin_api",
   "compute_extension:floating_ip_dns": "role:compute-user",
   "compute_extension:floating_ip_pools": "role:compute-user",
   "compute_extension:floating_ips": "role:compute-user",
   "compute_extension:hosts": "rule:admin_api",
   "compute_extension:keypairs": "role:compute-user",
   "compute_extension:multinic": "role:compute-user",
   "compute_extension:networks": "rule:admin_api",
   "compute_extension:quotas": "role:compute-user",
   "compute_extension:rescue": "role:compute-user",
   "compute_extension:security_groups": "role:compute-user",
   "compute_extension:server_action_list": "rule:admin_api",
   "compute_extension:server_diagnostics": "rule:admin_api",
   "compute_extension:simple_tenant_usage:show": "rule:admin_or_owner",
   "compute_extension:simple_tenant_usage:list": "rule:admin_api",
   "compute_extension:users": "rule:admin_api",
   "compute_extension:virtual_interfaces": "role:compute-user",
   "compute_extension:virtual_storage_arrays": "role:compute-user",
   "compute_extension:volumes": "role:compute-user",
   "compute_extension:volume_attachments:index": "role:compute-user",
   "compute_extension:volume_attachments:show": "role:compute-user",
   "compute_extension:volume_attachments:create": "role:compute-user",
   "compute_extension:volume_attachments:delete": "role:compute-user",
   "compute_extension:volumetypes": "role:compute-user",
   "volume:create": "role:compute-user",
   "volume:get_all": "role:compute-user",
   "volume:get_volume_metadata": "role:compute-user",
   "volume:get_snapshot": "role:compute-user",
   "volume:get_all_snapshots": "role:compute-user",
   "network:get_all_networks": "role:compute-user",
   "network:get_network": "role:compute-user",
   "network:delete_network": "role:compute-user",
   "network:disassociate_network": "role:compute-user",
   "network:get_vifs_by_instance": "role:compute-user",
   "network:allocate_for_instance": "role:compute-user",
   "network:deallocate_for_instance": "role:compute-user",
   "network:validate_networks": "role:compute-user",
   "network:get_instance_uuids_by_ip_filter": "role:compute-user",
   "network:get_floating_ip": "role:compute-user",
   "network:get_floating_ip_pools": "role:compute-user",
   "network:get_floating_ip_by_address": "role:compute-user",
   "network:get_floating_ips_by_project": "role:compute-user",
   "network:get_floating_ips_by_fixed_address": "role:compute-user",
   "network:allocate_floating_ip": "role:compute-user",
   "network:deallocate_floating_ip": "role:compute-user",
   "network:associate_floating_ip": "role:compute-user",
   "network:disassociate_floating_ip": "role:compute-user",
   "network:get_fixed_ip": "role:compute-user",
   "network:add_fixed_ip_to_instance": "role:compute-user",
   "network:remove_fixed_ip_from_instance": "role:compute-user",
   "network:add_network_to_project": "role:compute-user",
   "network:get_instance_nw_info": "role:compute-user",
   "network:get_dns_domains": "role:compute-user",
   "network:add_dns_entry": "role:compute-user",
   "network:modify_dns_entry": "role:compute-user",
   "network:delete_dns_entry": "role:compute-user",
   "network:get_dns_entries_by_address": "role:compute-user",
   "network:get_dns_entries_by_name": "role:compute-user",
   "network:create_private_dns_domain": "role:compute-user",
   "network:create_public_dns_domain": "role:compute-user",
   "network:delete_dns_domain": "role:compute-user"
  }



服务管理
-------------

身份服务提供身份、Token、目录和策略等服务，它包括：

* keystone WSGI 服务

 可以运行在任意 WSGI 兼容的 web 容器中，如 Apache，来提供身份服务。该服务和管理 API 各自以独立的 WSGI 服务运行。

* 身份服务功能

 每个功能都有可插拔的后端来允许用不同的方式来使用具体的服务。大多数支持标注后端如 LDAP 或者 SQL。

* keystone-all

 同时在一个进程中启动服务和管理 API。此服务不支持 federation token 模型。优先选择 WSGI 方式启动服务，且在 Newton 版本中，此方式将被删除。

身份服务同时也为每个服务维护一个 User，如计算服务有一个 User ``nova`` 相对应，和一个特殊的服务 Project ``service``。

如果需要更多关于创建 Service 和 Endpoint 的信息，请查看 :ref:`cli/manageservice` 。


组
--------------

组是同一 Domain 中一些用户的集合。管理员可以创建 Group 并向组中增加用户。一个 Role 可以赋予一个 Group 而不是单独的一个 User， Group 实在 OpenStack Identity V3 中引入的。

Identity V3 提供如下这些 Group 相关的操作：

* 创建一个 Group
* 删除一个 Group
* 修改一个 Group
* 增加一个 User 到一个 Group
* 从一个 Group 中删除一个 User
* 列出 Group 中的成员
* 列出 User 所属的 Group
* 把一个 Project 的 Role 赋予一个 Group
* 把一个 Domain 的 Role 赋予一个 Group
* 查询一个 Role 赋予的所有 Group
  
.. note ::

 身份服务可能不能支持所有的操作。例如，如果身份服务使用 LDAP 作为后端存储，且禁用了组修改，则组创建、删除、修改的请求就会失败。

如下是一些示例：

* Group A 使用 Role A 来授权访问 Project A。 如果 User A 是 Group A 的成员，当 User A 获得 Project A 范围的 Token 时，该 Token 包含 Role A。
* Group B 使用 Role B 来授权访问 Domain B。如果 User B 是 Group B 的成员，当 User B 获得 Domain B 范围的 Token 时，该 Token 包含 Role B。
