---
title: Oracle-官方程序包
abbrlink: '23939191'
date: 2019-02-14 08:46:33
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---

> Oracle 官方数据库程序包的使用

<!--more-->



# 官方提供

| 包名 | 简介 |
|----|----|
| DBMS_ADDM | 数据库诊断相关 |
| DBMS_ADVANCED_REWRITE | 重写用户查询 |
| DBMS_ADVISOR | 识别数据库插件问题 |
| DBMS_ALERT | 通过与触发器配合,实现对数据库警告 |
| DBMS_APP_CONT | 数据库事务提交.限制Session执行完成后提交与否 |
| DBMS_APPLICATION_INFO | 注册程序以便性能调优. |
| DBMS_APPLY_ADM | 提供启动，停止和配置Oracle Streams应用进程，XStream出站服务器和XStream入站服务器的接口 |
| DBMS_AQ | 提供了Oracle Streams Advanced Queuing（AQ）的接口。仅限管理员才可以管理队列 |
| DBMS_AQADM | 程序包提供了管理Oracle数据库高级队列（AQ）配置和管理信息的过程。|
| DBMS_AQELM | 提供了子程序，用于通过电子邮件和HTTP管理Oracle Streams高级队列（AQ）异步通知的配置。|
| DBMS_AQIN | 在提供对Oracle JMS接口的安全访问方面发挥了作用。 |
| DBMS_ASSERT | 提供一个验证输入值属性的接口 |
| DBMS_AUDIT_MGMT | 提供子程序来管理审计跟踪记录。这些子程序使审计管理员能够管理审计跟踪。|
| DBMS_AUDIT_UTIL | 软件包提供的功能，使您能够查询的输出格式中DBA_FGA_AUDIT_TRAIL，DBA_AUDIT_TRAIL，UNIFIED_AUDIT_TRAIL，和V$XML_AUDIT_TRAIL意见。|
| DBMS_AUTO_REPORT | 该DBMS_AUTO_REPORT软件包提供了一个接口，用于查看已捕获到自动工作负载存储库（AWR）中的SQL监控和实时自动数据库诊断监控（ADDM）数据。它还提供子程序来控制如何将这些数据捕获到AWR的行为。|
| DBMS_AUTO_SQLTUNE | 管理自动SQL调整任务的接口。与此不同DBMS_SQLTUNE，DBMS_AUTO_SQLTUNE包需要DBA角色。|
| DBMS_AUTO_TASK_ADMIN | 提供了AUTOTASK功能接口。DBA和企业管理器使用它来访问AUTOTASK控件。企业管理器也使用AUTOTASKAdvisor。|
| DBMS_AW_STATS | 管理多维数据集和维度的优化程序统计信息的子程序。生成统计信息不会产生显着的性能成本。|
| DBMS_CAPTURE_ADM | 该DBMS_CAPTURE_ADM软件包是一组Oracle Streams软件包之一，它提供了用于启动，停止和配置捕获过程的子程序。捕获的更改的来源是重做日志，捕获的更改的存储库是队列。|
| DBMS_COMPARISON | 该DBMS_COMPARISON包提供了在不同数据库中比较和聚合数据库对象的接口,便于数据共享 |
| DBMS_COMPRESSION | 该DBMS_COMPRESSION软件包提供了一个界面，便于为应用程序选择正确的压缩级别。|
| DBMS_CONNECTION_POOL | 该DBMS_CONNECTION_POOL软件包提供了一个用于管理数据库驻留连接池的接口。|
| DBMS_CQ_NOTIFICATION | 数据库更改通知功能的一部分，它提供了在客户端应用程序指定的查询上创建注册的功能，以便接收通知以响应与查询关联的对象的DML或DDL更改。当DML或DDL事务提交时，通知由数据库发布。|
| DBMS_CREDENTIAL | 该DBMS_CREDENTIAL软件包提供了一个接口，用于验证和模拟EXTPROC标注功能，以及外部作业，远程作业和文件监视器SCHEDULER。|
| DBMS_CRYPTO | DBMS_CRYPTO提供加密和解密存储数据的接口，可以与运行网络通信的PL / SQL程序一起使用。它支持多种行业标准加密和散列算法，包括高级加密标准（AES）加密算法。AES已被美国国家标准与技术研究院（NIST）批准用于取代数据加密标准（DES）。|
| DBMS_CSX_ADMIN | 该DBMS_CSX_ADMIN包提供了一个接口，用于在传输包含二进制XML数据的表空间时自定义设置。|
| DBMS_CUBE | DBMS_CUBE 包含创建OLAP多维数据集和维度的子程序，以及加载和处理数据以进行查询的子程序。|
| DBMS_CUBE_ADVISE | DBMS_CUBE_ADVISE 包含用于评估多维数据集物化视图的子程序，以支持基于日志的快速刷新和查询重写。|
| DBMS_CUBE_LOG | DBMS_CUBE_LOG 包含用于创建和管理多维数据集和多维数据集维度的日志的子程序。|
| DBMS_DATA_MINING | 该DBMS_DATA_MINING包是用于创建，评估和查询数据挖掘模型的应用程序编程接口。|
| DBMS_DATA_MINING_TRANSFORM | DBMS_DATA_MINING_TRANSFORM 实现一组通常用于数据挖掘的转换。|
| DBMS_DATAPUMP | 该DBMS_DATAPUMP包用于在数据库之间移动数据库的全部或部分，包括数据和元数据。|
| DBMS_DB_VERSION | 该DBMS_DB_VERSION软件包指定Oracle版本号和其他对基于Oracle版本的简单条件编译选择有用的信息。|
| DBMS_DBCOMP | 该DBMS_DBCOMP程序包执行块比较以检测主数据库与一个或多个物理备用数据库之间的丢失写入或数据库不一致。它包含一个过程，DBCOMP可以随时执行。（它不需要DB_LOST_WRITE_PROTECT启用初始化参数。）|
| DBMS_DBFS_CONTENT | 该DBMS_DBFS_CONTENT包提供包括由一个或多个商店提供商支持的类似文件系统的抽象的接口。|
| DBMS_DBFS_CONTENT_SPI | 该DBMS_DBFS_CONTENT_SPI包是DBMS_DBFS_CONTENT存储提供程序的规范，必须实现。应用程序设计人员可以创建符合此规范的PL / SQL包，以扩展DBMS_DBFS_CONTENT为使用自定义存储提供程序。|
| DBMS_DBFS_HS | Oracle数据库文件系统分层存储在DBMS_DBFS_HS包中实现。该软件包使用户能够在为其数据库表执行信息生命周期管理时将磁带或Amazon S3 Web服务用作存储层。|
| DBMS_DBFS_SFS | 该DBMS_DBFS_SFS包提供了一个接口，用于为DBMS_DBFS_CONTENT包中描述的内容接口操作基于SecureFile的存储（SFS）。|
| DBMS_DEBUG | DBMS_DEBUG已弃用。请DBMS_DEBUG_JDWP改用。 |
| DBMS_DEBUG_JDWP | 它DBMS_DEBUG_JDWP提供了通过Java调试线协议（JDWP）启动和控制PL / SQL存储过程和Java存储过程的调试的接口。|
| DBMS_DEBUG_JDWP_CUSTOM | 该DBMS_DEBUG_JDWP_CUSTOM程序包为数据库用户提供了一种使用Java调试线协议（JDWP）对调试器执行数据库会话的调试连接请求的自定义处理的方法。|
| DBMS_DESCRIBE | 您可以使用该DBMS_DESCRIBE包获取有关PL / SQL对象的信息。指定对象名称时，DBMS_DESCRIBE返回一组带有结果的索引表。执行全名转换，并检查最终对象的安全检查。|
| DBMS_DG | 该DBMS_DG程序包允许应用程序在Oracle Data Guard代理环境中通知主数据库或快速启动故障转移目标数据库，以在应用程序遇到保证故障转移的条件时启动快速启动故障转移。|
| DBMS_DIMENSION | DBMS_DIMENSION 使您可以验证维度关系，并提供企业管理器维度向导的替代方法，以显示维度定义。|
| DBMS_DISTRIBUTED_TRUST_ADMIN | DBMS_DISTRIBUTED_TRUST_ADMIN过程维护可信服务器列表。使用这些过程来定义服务器是否可信。如果数据库不受信任，Oracle将拒绝来自数据库的当前用户数据库链接。|
| DBMS_DNFS | 该DBMS_DNFS软件包提供了一个接口，可帮助使用备份集中的文件创建数据库。 |
| DBMS_DST | DBMS_DST包提供了一个接口，用于将夏令时（DST）修补程序应用于具有时区数据类型的时间戳。|
| DBMS_EDITIONS_UTILITIES | 该DBMS_EDITIONS_UTILITIES软件包为与版本相关的操作提供帮助函数。|
| DBMS_EPG | 该DBMS_EPG包实现了嵌入式PL / SQL网关，使Web浏览器能够通过HTTP侦听器调用PL / SQL存储过程。|
| DBMS_ERRLOG | 该DBMS_ERRLOG软件包提供了一个过程，使您可以创建错误日志记录表，以便DML操作在遇到错误后可以继续，而不是中止和回滚。这使您可以节省时间和系统资源。|
| DBMS_FGA | 该DBMS_FGA 软件包提供细粒度的安全功能。|
| DBMS_FILE_GROUP | 该DBMS_FILE_GROUP软件包是一组Oracle Streams软件包之一，提供管理文件组，文件组版本和文件的管理界面。文件组存储库是数据库中所有文件组的集合，可以包含特定文件组的多个版本。您可以使用此包来创建和管理文件组存储库。|
| DBMS_FILE_TRANSFER | 该DBMS_FILE_TRANSFER程序包提供了在数据库中复制二进制文件或在数据库之间传输二进制文件的过程。|
| DBMS_FLASHBACK | 使用时DBMS_FLASHBACK，可以在指定时间或指定的系统更改号（SCN）闪回数据库版本。|
| DBMS_FLASHBACK_ARCHIVE | 该DBMS_FLASHBACK_ARCHIVE程序包包含执行各种闪回任务的过程。|
| DBMS_FREQUENT_ITEMSET | DBMS_FREQUENT_ITEMSET包启用频繁项集计数。除输入光标格式差异外，这两个功能相同。|
| DBMS_FS | 该DBMS_FS软件包包含Oracle文件系统（OFS）过程，可用于创建，装载，卸载和销毁Oracle文件系统。|
| DBMS_GOLDENGATE_ADM | 该DBMS_GOLDENGATE_ADM软件包提供了用于在Oracle GoldenGate配置中配置自动冲突检测和解决的接口，该配置可在Oracle数据库之间复制表。|
| DBMS_GOLDENGATE_AUTH | 该DBMS_GOLDENGATE_AUTH软件包提供了用于授予GoldenGate管理员权限和撤销权限的子程序。|
| DBMS_HADOOP | DBMS_HADOOP包提供了两个用于创建Oracle外部表和同步Oracle外部表分区的过程。|
| DBMS_HANG_MANAGER | DBMS_HANG_MANAGER包提供了一种更改某些Hang Manager配置参数的方法。|
| DBMS_HEAT_MAP | 该DBMS_HEAT_MAP软件包提供了一个接口，用于在各种存储级别（包括块，范围，段，对象和表空间）外部化热图。第二组子程序外部化了由前N个表空间背景实现的热图。|
| DBMS_HIERARCHY | DBMS_HIERARCHY 包含用于验证层次结构和分析视图使用的表中的数据的子程序。|
| DBMS_HM | 此包包含用于运行状况检查管理的常量和过程声明。Health Monitor提供运行检查存储并通过DBMS_HM包 检索报告的工具|
| DBMS_HPROF | 该DBMS_HPROF软件包提供了一个用于分析PL / SQL应用程序执行情况的界面。它提供用于收集分层分析器数据，分析原始分析器输出和分析信息生成的服务。|
| DBMS_HS_PARALLEL | 该DBMS_HS_PARALLELPL / SQL包使异构目标访问并行处理。此程序包旨在提高从大型外表检索数据时的性能。|
| DBMS_HS_PASSTHROUGH | 在DBMS_HS_PASSTHROUGHPL / SQL包可以让你直接发送声明，非Oracle系统，而不由Oracle服务器进行解释。如果非Oracle系统允许在Oracle中没有等效语句的语句中进行操作，这将非常有用。|
| DBMS_ILM | 该DBMS_ILM软件包提供了一个使用自动数据优化（ADO）策略实现信息生命周期管理（ILM）策略的接口。|
| DBMS_ILM_ADMIN | 该DBMS_ILM_ADMIN软件包提供了一个自定义自动数据优化（ADO）策略执行的接口。结合分区和压缩，ADO策略可用于帮助实施信息生命周期管理（ILM）策略。|
| DBMS_INMEMORY | 该DBMS_INMEMORY软件包为内存中列存储（IM列存储）功能提供了一个接口。|
| DBMS_INMEMORY_ADMIN | DBMS_INMEMORY_ADMIN 提供用于管理内存中快速启动（IM FastStart）区域和内存中表达式（IM表达式）的接口。|
| DBMS_IOT | 该DBMS_IOT包创建一个表，使用该ANALYZE命令可以在其中放置对索引组织表的链接行的引用。DBMS_IOT还可以创建一个异常表，在该异常表中，可以在enable_constraint操作期间放置对违反约束的索引组织表的行的引用。|
| DBMS_JAVA | DBMS_JAVA包提供了一个PL / SQL接口，用于从Java访问数据库功能。|
| DBMS_JOB |　该DBMS_JOB程序包计划和管理作业队列中的作业，已被软件包DBMS_SCHEDULER取代 |
| DBMS_JSON | Package DBMS_JSON提供了用于处理存储在Oracle数据库中的JavaScript Object Notation（JSON）数据的子程序。|
| DBMS_LDAP | DBMS_LDAP包允许您从LDAP服务器访问数据。|
| DBMS_LDAP_UTL | DBMS_LDAP_UTL包中包含Oracle Extension实用程序功能。|
| DBMS_LIBCACHE | The DBMS_LIBCACHE程序包由一个子程序组成，该子程序通过从远程实例中提取SQL和PL / SQL并在本地编译此SQL而不执行来在Oracle实例上准备库高速缓存。编译实例的缓存的价值在于准备应用程序在故障转移或切换之前执行所需的信息。|
| DBMS_LOB | 所述DBMS_LOB包提供子程序进行操作BLOBs，CLOBs，NCLOBs，BFILEs，和临时LOBs。您可以使用它DBMS_LOB来访问和操作LOB的特定部分或完成LOB。|
| DBMS_LOCK | 该DBMS_LOCK软件包提供了Oracle Lock Management服务的接口。您可以请求锁定特定模式，在同一个或另一个实例中为其指定另一个过程中可识别的唯一名称，更改锁定模式并释放它。|
| DBMS_LOGMNR | 该DBMS_LOGMNR包是一组LogMiner包之一，包含用于初始化LogMiner工具以及开始和结束LogMiner会话的子程序。|
| DBMS_LOGMNR_D | 当LogMiner向您返回重做数据时，它需要一个字典将对象ID转换为对象名称。|
| DBMS_LOGSTDBY | 该DBMS_LOGSTDBY程序包提供了用于配置和管理逻辑备用数据库环境的子程序。|
| DBMS_LOGSTDBY_CONTEXT | 从Oracle Database 12 c第 1版（12.1）开始，SQL Apply进程可以访问名为的上下文命名空间LSBY_APPLY_CONTEXT。您可以使用DBMS_LOGSTDBY_CONTEXT包中提供的过程来设置和检索与之关联的各种参数LSBY_APPLY_CONTEXT。在使用DBMS_LOGSTBDY.SKIP和DBMS_LOGSTDBY.SKIP_ERROR过程编写使用SQL Apply注册的跳过程序时，这非常有用。|
| DBMS_METADATA | 该DBMS_METADATA包提供了一种方法，您可以从数据库字典中检索元数据作为XML或创建DDL，并提交XML以重新创建对象。|
| DBMS_METADATA_DIFF | 该DBMS_METADATA_DIFF软件包包含用于比较SXML格式的两个元数据文档的接口。|
| DBMS_MGD_ID_UTL | 该DBMS_MGD_ID_UTL软件包包含各种实用程序功能和过程。|
| DBMS_MGWADM | DBMS_MGWADM定义Messaging Gateway管理界面。包和对象类型归SYS。|
| DBMS_MGWMSG | DBMS_MGWMSG 提供规范消息类型用于转换消息体的对象类型，以及用于处理Messaging Gateway消息类型的方法，常量和子程序。|
| DBMS_MONITOR | 该DBMS_MONITOR程序包使您可以使用PL / SQL来控制其他跟踪和统计信息收集。|
| DBMS_MVIEW | DBMS_MVIEW使您能够了解物化视图和潜在物化视图的功能，包括重写可用性。它还使您能够刷新不属于同一刷新组和清除日志的实体化视图。|
| DBMS_MVIEW_STATS | DBMS_MVIEW_STATS package提供了一个界面来管理物化视图刷新操作的统计信息的收集和保留。|
| DBMS_NETWORK_ACL_ADMIN | 该DBMS_NETWORK_ACL_ADMIN程序包提供管理网络访问控制列表（ACL）的界面。|
| DBMS_NETWORK_ACL_UTILITY | 该DBMS_NETWORK_ACL_UTILITY软件包提供实用程序功能，以便于评估管理到网络主机的TCP连接的访问控制列表（ACL）分配。|
| DBMS_ODCI | DBMS_ODCI package包含与使用Data Cartridges相关的单个用户功能。|
| DBMS_OUTLN | 与此DBMS_OUTLN同义的包OUTLN_PKG包含与存储的轮廓管理相关的子程序的功能接口。|
| DBMS_OUTPUT | 该DBMS_OUTPUT程序包使您可以从存储过程，包和触发器发送消息。该包对于显示PL / SQL调试信息特别有用。|
| DBMS_PARALLEL_EXECUTE | 该DBMS_PARALLEL_EXECUTE软件包可以并行地增量更新表数据。|
| DBMS_PART | 该DBMS_PART软件包为分区对象提供了维护和管理操作的接口。|
| DBMS_PCLXUTIL | 该DBMS_PCLXUTIL软件包提供了分区内并行性，用于创建分区方式的本地索引。DBMS_PCLXUTIL避免了这样的限制：对于本地索引创建，并行度被限制为分区数，因为每个分区仅使用一个并行执行服务器进程。|
| DBMS_PDB | 该DBMS_PDB软件包提供了一个接口，用于检查和操作多租户容器数据库（CDB）中可插拔数据库（PDB）的数据。它还包含一个接口，指定哪些数据库对象是应用程序公共对象。|
| DBMS_PDB_ALTER_SHARING | 在具有预安装应用程序的应用程序容器中，该DBMS_PDB_ALTER_SHARING程序包提供了一个接口，用于将数据库对象设置为应用程序公共对象，或指定数据库对象不是应用程序公共对象。|
| DBMS_PERF | DBMS_PERF程序包提供和接口以生成用于监视数据库性能的活动报告|
| DBMS_PIPE | 该DBMS_PIPE程序包允许同一实例中的两个或多个会话进行通信。Oracle管道在概念上与UNIX中使用的管道类似，但Oracle管道未使用操作系统管道机制实现。|
| DBMS_PLSQL_CODE_COVERAGE | 该DBMS_PLSQL_CODE_COVERAGE包提供了一个接口，用于在基本块级别收集PL / SQL应用程序的代码覆盖率数据。|
| DBMS_PREDICTIVE_ANALYTICS | 数据挖掘可以发现埋藏在大量数据中的有用信息。但是，通常情况下，获取这些结果所需的编程接口和数据挖掘专业知识过于复杂，无法被广泛的受众使用，这些受众可以从使用Oracle数据挖掘中获益。|
| DBMS_PREPROCESSOR | DBMS_PREPROCESSOR包提供了一个接口，用于在其后处理的表单中打印或检索PL / SQL单元的源文本。|
| DBMS_PRIVILEGE_CAPTURE | 该DBMS_PRIVILEGE_CAPTURE包提供了数据库权限分析的接口。|
| DBMS_PROCESS | 该DBMS_PROCESS软件包提供了一个用于管理预先生成的服务器的接口。|
| DBMS_PROFILER | 该软件包提供了一个界面来分析现有的PL / SQL应用程序并识别性能瓶颈。然后，您可以收集并持久存储PL / SQL分析器数据。|
| DBMS_PROPAGATION_ADM | 该DBMS_PROPAGATION_ADM程序包是一组Oracle Streams程序包之一，它提供了用于配置从源队列到目标队列的传播的管理接口。|
| DBMS_QOPATCH | 该DBMS_QOPATCH软件包提供了一个界面来查看已安装的数据库修补程|
| DBMS_RANDOM | 该DBMS_RANDOM软件包提供了一个内置的随机数生成器。DBMS_RANDOM不适用于加密。|
| DBMS_REDACT | 该DBMS_REDACT软件包提供了Oracle Data Redaction的接口，使您可以屏蔽（编辑）由低特权用户或应用程序发出的查询返回的数据。|
| DBMS_REDEFINITION | 该DBMS_REDEFINITION包提供了一个用于执行表的在线重新定义的接口。|
| DBMS_REFRESH | 该DBMS_REFRESH程序包使您可以创建实体化视图组，这些视图可以一起刷新到事务一致的时间点。这些组称为刷新组。|
| DBMS_REPAIR | 该DBMS_REPAIR软件包包含数据损坏修复过程，使您能够检测和修复表和索引中的损坏块。您可以尽可能地解决损坏，并在尝试重建或修复对象时继续使用对象。|
| DBMS_RESCONFIG | 该DBMS_RESCONFIG包提供了一个接口，用于在资源配置列表上操作，以及检索资源的侦听器信息。|
| DBMS_RESOURCE_MANAGER | 该DBMS_RESOURCE_MANAGER软件包维护计划，消费者组和计划指令。它还提供语义，以便您可以将对计划架构的更改组合在一起。|
| DBMS_RESOURCE_MANAGER_PRIVS | 该DBMS_RESOURCE_MANAGER_PRIVS程序包维护与资源管理器关联的权限。|
| DBMS_RESULT_CACHE | 该DBMS_RESULT_CACHE软件包提供了一个接口，允许DBA管理SQL结果缓存和PL / SQL函数结果缓存使用的共享池的那部分。|
| DBMS_RESUMABLE | 使用该DBMS_RESUMABLE程序包，您可以在执行很长时间后暂停空间不足或达到空间限制的大型操作，修复问题并使语句恢复执行。通过这种方式，您可以编写应用程序而无需担心遇到与空间相关的错误。|
| DBMS_RLS | 该DBMS_RLS软件包包含细粒度的访问控制管理界面，用于实现虚拟专用数据库（VPD）。|
| DBMS_ROLLING | 在 DBMS_ROLLINGPL / SQL包来实现滚动升级使用Active Data Guard特性，从而简化以滚动方式在Data Guard配置升级Oracle数据库软件的过程。使用Active Data Guard功能进行滚动升级需要Oracle Active Data Guard选件的许可证，并且可以用于从Oracle Database 12 c的第一个补丁集开始的数据库版本升级。|
| DBMS_ROWID | 该DBMS_ROWID软件包允许您从PL / SQL程序和SQL语句中创建ROWIDs和获取有关ROWIDs的信息。您可以找到数据块编号，对象编号和其他ROWID组件，而无需编写代码来解释外部的base-64字符ROWID。|
| DBMS_RULE | 该DBMS_RULE程序包包含子程序，可用于评估指定事件的规则集。|
| DBMS_RULE_ADM | 该DBMS_RULE_ADM包提供了用于创建和管理规则，规则集和规则评估上下文的子程序。|
| DBMS_SCHEDULER | 该DBMS_SCHEDULER包提供了一组可以从任何PL / SQL程序调用的调度函数和过程。|
| DBMS_SERVER_ALERT | 通过该DBMS_SERVER_ALERT程序包，您可以将Oracle数据库服务器配置为在违反指定服务器度量标准的阈值时发出警报。您可以为大量预定义指标配置警告和严重阈值。|
| DBMS_SERVICE | 该DBMS_SERVICE软件包允许您为单个实例创建，删除，激活和停用服务。|
| DBMS_SESSION | 该包提供对PL / SQL的SQL ALTER SESSION和SET ROLE语句以及其他会话信息的访问。您可以使用它DBMS_SESSION来设置首选项和安全级别。|
| DBMS_SFW_ACL_ADMIN | DBMS_SFW_ACL_ADMIN包提供用于管理和管理“数据库服务防火墙”功能的访问控制策略的接口。每个策略都由一个访问控制列表（ACL）表示，该列表包含允许访问特定数据库服务的主机。本地侦听器和服务器进程根据ACL验证所有入站客户端连接。|
| DBMS_SHARED_POOL | 该DBMS_SHARED_POOL包提供对共享池的访问，共享池是存储游标和PL / SQL对象的共享内存区域。DBMS_SHARED_POOL使您能够显示共享池中对象的大小，并标记它们是为了保持还是不保留，以减少内存碎片。|
| DBMS_SPACE | 该DBMS_SPACE软件包使您可以分析细分市场增长和空间需求。|
| DBMS_SPACE_ADMIN | 该DBMS_SPACE_ADMIN包提供本地管理的表空间的功能。|
| DBMS_SPD | 该DBMS_SPD软件包提供了用于管理SQL计划指令（SPD）的子程序。|
| DBMS_SPM | 该DBMS_SPM软件包支持SQL计划管理功能，为DBA或其他用户提供接口，以执行为各种SQL语句维护的计划历史记录和SQL计划基准的受控操作。|
| DBMS_SQL | 该DBMS_SQL包提供了一个接口，使用动态SQL来解析使用PL / SQL的任何数据操作语言（DML）或数据定义语言（DDL）语句。|
| DBMS_SQL_MONITOR | 该DBMS_SQL_MONITOR程序包提供有关实时SQL监视和实时数据库操作监视的信息。|
| DBMS_SQL_TRANSLATOR | 该DBMS_SQL_TRANSLATOR软件包提供了用于创建，配置和使用SQL翻译配置文件的接口。|
| DBMS_SQLDIAG | DBMS_SQLDIAG包提供SQL Diagnosability功能的接口。|
| DBMS_SQLPA | DBMS_SQLPA包提供了实现SQL Performance Analyzer的接口。|
| DBMS_SQLTUNE | 该DBMS_SQLTUNE包是用于按需调优SQL的接口。相关的软件DBMS_AUTO_SQLTUNE包包为SQL Tuning Advisor提供了作为自动化任务运行的接口。|
| DBMS_STAT_FUNCS | 该DBMS_STAT_FUNCS包提供统计功能。|
| DBMS_STATS | 使用该DBMS_STATS包，您可以查看和修改为数据库对象收集的优化程序统计信息。|
| DBMS_STORAGE_MAP | 使用该DBMS_STORAGE_MAP包，您可以与Oracle后台进程FMON进行通信，以调用填充映射视图的映射操作。FMON与操作和存储系统供应商提供的映射库进行通信。|
| DBMS_STREAMS | 该DBMS_STREAMS程序包是一组Oracle Streams程序包之一，它提供子程序以将ANYDATA对象转换为逻辑更改记录（LCR）对象，返回有关Oracle Streams属性和Oracle Streams客户端的信息，以及使用二进制文件注释会话生成的重做条目标签。|
| DBMS_STREAMS_ADM | 该DBMS_STREAMS_ADM软件包是一组Oracle Streams软件包之一，它提供了用于配置Oracle Streams环境的子程序。|
| DBMS_STREAMS_ADVISOR_ADM | 该DBMS_STREAMS_ADVISOR_ADM软件包是一组Oracle Streams软件包之一，它提供了一个接口，用于收集有关Oracle Streams环境的信息，并根据收集的信息为数据库管理员提供建议。|
| DBMS_STREAMS_AUTH | 该DBMS_STREAMS_AUTH程序包是一组Oracle Streams程序包之一，它提供了子程序，用于向Oracle Streams管理员授予权限以及从Oracle Streams管理员撤消权限。|
| DBMS_STREAMS_HANDLER_ADM | 该DBMS_STREAMS_HANDLER_ADM软件包是一组Oracle Streams软件包之一，它提供了管理语句DML处理程序的接口。|
| DBMS_STREAMS_MESSAGING | 该DBMS_STREAMS_MESSAGING程序包是一组Oracle Streams程序包之一，它提供了将消息排入ANYDATA队列并使队列中的消息出列的接口。|
| DBMS_STREAMS_TABLESPACE_ADM | 该DBMS_STREAMS_TABLESPACE_ADM软件包是一组Oracle Streams软件包之一，它提供了管理接口，用于在数据库之间复制表空间以及将表空间从一个数据库移动到另一个数据库。|
| DBMS_SYNC_REFRESH | 该DBMS_SYNC_REFRESH包提供了一个接口，用于执行物化视图的同步刷新。|
| DBMS_TDB | 该DBMS_TDB程序包报告是否可以使用RMAN CONVERT DATABASE命令在平台之间传输数据库。该程序包验证当前主机平台上的数据库是否与目标平台具有相同的endian格式，并且当前数据库的状态不会阻止数据库的传输。|
| DBMS_TNS | 该DBMS_TNS软件包提供了RESOLVE_TNSNAME解析TNS名称并返回相应Oracle Net8连接字符串的功能。|
| DBMS_TRACE | 该DBMS_TRACE包包含跟踪PL / SQL函数，过程和异常的接口。|
| DBMS_TRANSACTION | DBMS_TRANSACTION包提供对存储过程的SQL事务语句的访问。|
| DBMS_TRANSFORM | 该DBMS_TRANSFORM软件包提供了Oracle Advanced Queuing的消息格式转换功能的接口。|
| DBMS_TSDP_MANAGE | 该DBMS_TSDP_MANAGE软件包提供了一个用于导入和管理数据库中敏感列和敏感列类型的接口，并与透明敏感数据保护（TSDP）策略的DBMS_TSDP_PROTECT程序包结合使用。|
| DBMS_TSDP_PROTECT | 该DBMS_TSDP_PROTECT软件包提供了一个接口，用于配置透明敏感数据保护（TSDP）策略以及DBMS_TSDP_MANAGE包。|
| DBMS_TTS | DBMS_TTS包检查可传输集是否是自包含的。所有违规都将插入到可从视图中选择的临时表中TRANSPORT_SET_VIOLATIONS。|
| DBMS_TYPES | 该DBMS_TYPES包由常量组成，表示内置和用户定义的类型。|
| DBMS_UMF | 该DBMS_UMF软件包提供了一个用于为Oracle数据库部署远程管理框架（RMF）的界面。RMF用于收集Oracle数据库的性能统计信息。|
| DBMS_UTILITY | 该DBMS_UTILITY包提供各种实用程序子程序。|
| DBMS_WARNING | 该DBMS_WARNING包提供了一种操作PL / SQL警告消息行为的方法，特别是通过读取和更改PLSQL_WARNINGS初始化参数的设置来控制哪些类型的警告被抑制，显示或视为错误。此包提供查询，修改和删除当前系统或会话设置的界面。|
| DBMS_WM | 该DBMS_WM软件包提供了Oracle Database Workspace Manager（通常称为Workspace Manager）的接口。|
| DBMS_WORKLOAD_CAPTURE | 该DBMS_WORKLOAD_CAPTURE软件包配置Workload Capture系统并生成工作负载捕获数据。|
| DBMS_WORKLOAD_REPLAY | 该DBMS_WORKLOAD_REPLAY软件包提供了重播工作负载捕获的接口。|
| DBMS_WORKLOAD_REPOSITORY | 该DBMS_WORKLOAD_REPOSITORY软件包允许您通过执行操作来管理自动工作负载存储库（AWR），例如管理快照和基线。|
| DBMS_XA | 该DBMS_XA软件包包含XA / Open接口，供应用程序在PL / SQL中调用XA接口。使用此包，应用程序开发人员可以使用PL / SQL跨SQL * Plus会话或进程切换或共享事务。|
| DBMS_XDB | DBMS_XDB包子程序和常数弃用与Oracle数据库12 Ç。虽然继续支持所有功能以实现向后兼容，但Oracle建议您使用每种情况下提供的备用过程。|
| DBMS_XDB_ADMIN | 该DBMS_XDB_ADMIN软件包提供了一个用于管理Oracle XML DB存储库的界面。|
| DBMS_XDB_CONFIG | 该DBMS_XDB_CONFIG软件包提供了用于配置Oracle XML DB及其存储库的接口。|
| DBMS_XDB_CONSTANTS | 该DBMS_XDB_CONSTANTS包提供了常用常量的接口。用户应使用常量而不是动态字符串来避免印刷错误。|
| DBMS_XDB_REPOS | 该DBMS_XDB_REPOS软件包提供了一个在Oracle XML数据库存储库上运行的接口。|
| DBMS_XDB_VERSION | DBMS_XBD_VERSION程序包中包含Oracle XML DB版本控制界面。DBMS_XDB_VERSION帮助创建VCR和管理版本历史中的版本的功能和过程。|
| DBMS_XDBRESOURCE | 该DBMS_XDBRESOURCE包提供了对资源的元数据和内容进行操作的接口。|
| DBMS_XDBT | 该DBMS_XDBT 软件包为管理员提供了一种CONTEXT在Oracle XML DB层次结构上设置索引的便捷机制。|
| DBMS_XDBZ | DBMS_XDBZ包控制Oracle XML DB存储库安全性，该安全性基于访问控制列表（ACL）。|
| DBMS_XEVENT | DBMS_XEVENTpackage提供与事件相关的类型和支持子程序。|
| DBMS_XMLDOM | 该DBMS_XMLDOM包用于访问XMLType对象，并实现文档对象模型（DOM），这是HTML和XML文档的应用程序编程接口。|
| DBMS_XMLGEN | 该DBMS_XMLGEN包将SQL查询的结果转换为规范的XML格式。该包采用任意SQL查询作为输入，将其转换为XML格式，并将结果作为a返回CLOB。这个包类似于DBMS_XMLQUERY包，除了它是用C编写并编译到内核中。此包只能在数据库上运行。|
| DBMS_XMLINDEX | 该DBMS_XMLINDEX包提供了实现异步索引的接口。|
| DBMS_XMLPARSER | 使用DBMS_XMLPARSER，您可以访问XML文档的内容和结构。XML描述了一类数据XML文档对象。它部分描述了处理它们的计算机程序的行为。通过构造，XML文档符合SGML文档。|
| DBMS_XMLQUERY | DBMS_XMLQUERY提供数据库到XMLType功能。只要有可能，请使用DBMS_XMLGEN内置包而不是DBMS_XMLQUERY。|
| DBMS_XMLSAVE | DBMS_XMLSAVE 为数据库类型的功能提供XML。|
| DBMS_XMLSCHEMA | DBMS_XMLSCHEMA package提供了管理XML模式的过程。|
| DBMS_XMLSCHEMA_ANNOTATE | 该DBMS_XMLSCHEMA_ANNOTATE软件包提供了一个界面来管理和配置结构化存储模型，主要是通过使用预注册模式注释。|
| DBMS_XMLSTORAGE_MANAGE | 该DBMS_XMLSTORAGE_MANAGE程序包提供了一个接口，用于在架构注册完成后管理和修改XML存储。|
| DBMS_XMLSTORE | DBMS_XMLSTORE 提供在关系表中存储XML数据的功能。|
| DBMS_XMLTRANSLATIONS | 不推荐使用的DBMS_XMLTRANSLATIONS软件包,提供了一个执行翻译的接口，以便可以使用各种语言搜索或显示字符串。|
| DBMS_XPLAN | 该DBMS_XPLAN软件包提供了一种以EXPLAIN PLAN多种预定义格式显示命令输出的简便方法。|
| DBMS_XSLPROCESSOR | 该DBMS_XSLPROCESSOR包提供了一个界面来管理XML文档的内容和结构。|
| DBMS_XSTREAM_ADM | 此DBMS_XSTREAM_ADM包提供用于在Oracle数据库和其他系统之间进行流数据库更改的接口。XStream使应用程序能够流出或流式传输数据库更改。|
| DBMS_XSTREAM_AUTH | 该DBMS_XSTREAM_AUTH程序包提供了用于向XStream管理员授予权限和撤消权限的子程序。|





# DBMS_CRYPTO  加解密


# DBMS_ERRLOG 错误日志记录

# DBMS_FLASHBACK 闪回


# DBMS_JSON 处理JSON

# DBMS_LOB 处理

# DBMS_LOCK 锁

# DBMS_NETWORK_ACL_ADMIN 管理ACL

# DBMS_NETWORK_ACL_UTILITY 评估ACL

# DBMS_OUTPUT 输出消息

# DBMS_PDB 管理PDB数据库

# DBMS_RANDOM 随机数

# DBMS_ROWID 主列

# DBMS_SCHEDULER 调度

# DBMS_SESSION 修改Session

# DBMS_SQL 动态SQL

# DBMS_TNS TNS状态

# DBMS_TRACE 过程,函数跟踪

# DBMS_TRANSACTION 事务控制

# DBMS_TYPES 类型定义

# DBMS_XMLDOM 修改XML

# DBMS_XMLPARSER 解析

# UTL_HTTP 网络请求