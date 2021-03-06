
.. _config_guide:

========
配置指南
========

--------
配置文件
--------

Linux下RPM、DEB模式安装，或直接公有云直接创建镜像，配置文件在 /etc/emqx/ 目录下:

+----------------------------+------------------------------------+
| 配置文件                   | 说明                               |
+============================+====================================+
| /etc/emqx/emqx.conf        | EMQ X 服务器配置文件               |
+----------------------------+------------------------------------+
| /etc/emqx/acl.conf         | EMQ X 默认ACL规则文件              |
+----------------------------+------------------------------------+
| /etc/emqx/plugins/\*.conf  | EMQ X 插件、存储、桥接配置文件     |
+----------------------------+------------------------------------+

Linux下通用包安装，配置文件位于程序安装路径etc/目录:

+----------------------------+------------------------------------+
| 配置文件                   | 说明                               |
+============================+====================================+
| etc/emqx.conf              | EMQ X 服务器配置文件               |
+----------------------------+------------------------------------+
| etc/acl.conf               | EMQ X 默认ACL规则文件              |
+----------------------------+------------------------------------+
| etc/plugins/\*.conf        | EMQ X 插件、存储、桥接配置文件     |
+----------------------------+------------------------------------+

--------
环境变量
--------

EMQ X 支持通过环境变量在启动时设置系统参数:

+--------------------+------------------------------------------+
| EMQX_NODE_NAME     | Erlang节点名称，例如: emqx@192.168.0.6   |
+--------------------+------------------------------------------+
| EMQX_NODE_COOKIE   | Erlang分布式节点通信Cookie               |
+--------------------+------------------------------------------+
| EMQX_MAX_PORTS     | Erlang虚拟机最大允许打开文件句柄数       |
+--------------------+------------------------------------------+
| EMQX_TCP_PORT      | MQTT TCP监听端口，默认: 1883             |
+--------------------+------------------------------------------+
| EMQX_SSL_PORT      | MQTT SSL监听端口，默认: 8883             |
+--------------------+------------------------------------------+
| EMQX_WS_PORT       | MQTT/WebSocket监听端口，默认: 8083       |
+--------------------+------------------------------------------+
| EMQX_WSS_PORT      | MQTT//WebSocket/SSL 监听端口，默认: 8084 |
+--------------------+------------------------------------------+

--------------
EMQ X 节点名称
--------------

EMQ X 节点名称、分布节点间通信Cookie:

.. code-block:: properties

    ## Node name
    node.name = emqx@127.0.0.1

    ## Cookie for distributed node
    node.cookie = emqx_dist_cookie

.. NOTE::

    Erlang/OTP平台应用由分布的Erlang节点(进程)组成，每个Erlang节点(进程)需指配一个节点名，用于节点间通信互访。所有互连Erlang节点(进程)间通过共用的Cookie进行安全认证。

--------------
Erlang VM 参数
--------------

Erlang 虚拟机参数，默认设置支持10万线连接:

.. code-block:: properties

    ## SMP support: enable, auto, disable
    node.smp = auto

    ## vm.args: -heart
    ## Heartbeat monitoring of an Erlang runtime system
    ## Value should be 'on' or comment the line
    ## node.heartbeat = on

    ## Enable kernel poll
    node.kernel_poll = on

    ## async thread pool
    node.async_threads = 32

    ## Erlang Process Limit
    node.process_limit = 256000

    ## Sets the maximum number of simultaneously existing ports for this system
    node.max_ports = 256000

    ## Set the distribution buffer busy limit (dist_buf_busy_limit)
    node.dist_buffer_size = 32MB

    ## Max ETS Tables.
    ## Note that mnesia and SSL will create temporary ets tables.
    node.max_ets_tables = 256000

    ## Tweak GC to run more often
    node.fullsweep_after = 1000

    ## Crash dump
    node.crash_dump = {{ platform_log_dir }}/crash.dump

    ## Distributed node ticktime
    node.dist_net_ticktime = 60

    ## Distributed node port range
    node.dist_listen_min = 6369
    node.dist_listen_max = 6369

Erlang虚拟机主要参数说明:

+-------------------------+---------------------------------------------------------------------------------------------+
| node.process_limit      | Erlang虚拟机允许的最大进程数，一个MQTT连接会消耗2个Erlang进程，所以参数值 > 最大连接数 * 2  |
+-------------------------+---------------------------------------------------------------------------------------------+
| node.max_ports          | Erlang虚拟机允许的最大Port数量，一个MQTT连接消耗1个Port，所以参数值 > 最大连接数            |
+-------------------------+---------------------------------------------------------------------------------------------+
| node.dist_listen_min    | Erlang分布节点间通信使用TCP连接端口范围。注: 节点间如有防火墙，需要配置该端口段             |
+-------------------------+---------------------------------------------------------------------------------------------+
| node.dist_listen_max    | Erlang分布节点间通信使用TCP连接端口范围。注: 节点间如有防火墙，需要配置该端口段             |
+-------------------------+---------------------------------------------------------------------------------------------+

--------------
EMQ X 集群通信
--------------

EMQ X 支持Scalable RPC架构，分离节点间的消息转发通道与集群控制通道，以提高集群的稳定性和消息转发性能:

.. code-block:: properties

    ## TCP server port.
    rpc.tcp_server_port = 5369

    ## Default TCP port for outgoing connections
    rpc.tcp_client_port = 5369

    ## Client connect timeout
    rpc.connect_timeout = 5000

    ## Client and Server send timeout
    rpc.send_timeout = 5000

    ## Authentication timeout
    rpc.authentication_timeout = 5000

    ## Default receive timeout for call() functions
    rpc.call_receive_timeout = 15000

    ## Socket keepalive configuration
    rpc.socket_keepalive_idle = 7200

    ## Seconds between probes
    rpc.socket_keepalive_interval = 75

    ## Probes lost to close the connection
    rpc.socket_keepalive_count = 9

--------------
日志级别与文件
--------------

console日志
-----------

.. code-block:: properties

    ## Console log. Enum: off, file, console, both
    log.console = console

    ## Console log level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.console.level = error

error日志
---------

.. code-block:: properties

    ## Error log file
    log.error.file = {{ platform_log_dir }}/error.log

crash日志
---------

.. code-block:: properties

    ## Enable the crash log. Enum: on, off
    log.crash = on

    log.crash.file = {{ platform_log_dir }}/crash.log

syslog日志
----------

.. code-block:: properties

    ## Syslog. Enum: on, off
    log.syslog = on

    ##  syslog level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.syslog.level = error

-----------------
匿名认证与ACL文件
-----------------

EMQ X 默认开启匿名认证，允许任意客户端登录:

.. code-block:: properties

    ## Allow Anonymous authentication
    mqtt.allow_anonymous = true

访问控制(ACL)文件
-----------------

默认基于acl.conf文件的ACL访问控制, 在etc/emqx.conf中设置:

.. code-block:: properties
    
    ## ACL nomatch
    mqtt.acl_nomatch = allow

    ## ACL nomatch
    mqtt.acl_nomatch = allow

    ## Default ACL File
    mqtt.acl_file = etc/acl.conf

acl.conf访问控制规则定义::

    允许|拒绝  用户|IP地址|ClientID  发布|订阅  主题列表

访问控制规则采用Erlang元组格式，访问控制模块逐条匹配规则:

.. image:: _static/images/authn_2.png

acl.conf默认访问规则设置:

.. code-block:: erlang

    %% 允许'dashboard'用户订阅 '$SYS/#'
    {allow, {user, "dashboard"}, subscribe, ["$SYS/#"]}.

    %% 允许本机用户发布订阅全部主题
    {allow, {ipaddr, "127.0.0.1"}, pubsub, ["$SYS/#", "#"]}.

    %% 拒绝用户订阅'$SYS#'与'#'主题
    {deny, all, subscribe, ["$SYS/#", {eq, "#"}]}.

.. NOTE:: 默认规则只允许本机用户订阅'$SYS/#'与'#'

EMQ X 接收到MQTT客户端发布(PUBLISH)或订阅(SUBSCRIBE)请求时，会逐条匹配ACL访问控制规则，直到匹配成功返回allow或deny。

是否缓存访问控制(ACL)
---------------------

系统是否会缓存PUBLISH消息的ACL规则:

.. code-block:: properties

    ## Cache ACL for PUBLISH
    mqtt.cache_acl = true

.. NOTE:: 如单客户端有多ACL条目，缓存会导致内存占用过多。

------------
MQTT协议参数
------------

ClientId最大允许长度
--------------------

.. code-block:: properties

    ## Max ClientId Length Allowed.
    mqtt.max_clientid_len = 1024

MQTT最大报文尺寸
----------------

.. code-block:: properties

    ## Max Packet Size Allowed, 64K by default.
    mqtt.max_packet_size = 64KB

客户端连接闲置时间
------------------

Socket连接建立至收到CONNECT报文的最大允许间隔时间:

.. code-block:: properties

    ## Client Idle Timeout (Second)
    mqtt.client.idle_timeout = 30

客户端连接强制GC
----------------

该参数优化MQTT连接的CPU/内存占用，收发一定数量消息后强制GC:

.. code-block:: properties

    ## Force GC: integer. Value 0 disabled the Force GC.
    mqtt.conn.force_gc_count = 100

单连接统计支持
--------------

启用单客户端连接统计:

.. code-block:: properties

    ## Enable client Stats: on | off
    mqtt.client.enable_stats = off

------------
MQTT会话参数
------------

EMQ X为每个MQTT连接创建会话:

.. code-block:: properties

    ## Max Number of Subscriptions, 0 means no limit.
    mqtt.session.max_subscriptions = 0

    ## Upgrade QoS?
    mqtt.session.upgrade_qos = off

    ## Max Size of the Inflight Window for QoS1 and QoS2 messages
    ## 0 means no limit
    mqtt.session.max_inflight = 32

    ## Retry Interval for redelivering QoS1/2 messages.
    mqtt.session.retry_interval = 20s

    ## Client -> Broker: Max Packets Awaiting PUBREL, 0 means no limit
    mqtt.session.max_awaiting_rel = 100

    ## Awaiting PUBREL Timeout
    mqtt.session.await_rel_timeout = 20s

    ## Enable Statistics: on | off
    mqtt.session.enable_stats = off

    ## Expired after 1 day:
    ## w - week
    ## d - day
    ## h - hour
    ## m - minute
    ## s - second
    mqtt.session.expiry_interval = 2h

+---------------------------+----------------------------------------------------------+
| session.max_subscriptions | 最大允许创建订阅数量                                     |
+---------------------------+----------------------------------------------------------+
| session.upgrade_qos       | 根据订阅升级消息QoS                                      |
+---------------------------+----------------------------------------------------------+
| session.max_inflight      | 飞行窗口。最大允许同时下发的Qos1/2报文数，0表示没有限制。|
|                           | 窗口值越大，吞吐越高；窗口值越小，消息顺序越严格         |
+---------------------------+----------------------------------------------------------+
| session.retry_interval    | 下发QoS1/2消息未收到PUBACK响应的重试间隔                 |
+---------------------------+----------------------------------------------------------+
| session.max_awaiting_rel  | 最大等待PUBREL的QoS2报文数                               |
+---------------------------+----------------------------------------------------------+
| session.await_rel_timeout | 收到QoS2消息，等待PUBREL报文超时时间                     |
+---------------------------+----------------------------------------------------------+
| session.enable_stats      | 启用会话指标统计                                         |
+---------------------------+----------------------------------------------------------+
| session.expiry_interval   | 持久会话超期时间，从客户端断开算起，单位：分钟           |
+---------------------------+----------------------------------------------------------+

----------------
MQTT消息队列参数
----------------

EMQ X为每个会话创建消息队列缓存Qos1/Qos2消息:

1. 持久会话(Session)的离线消息

2. 飞行窗口满而延迟下发的消息

队列参数设置:

.. code-block:: properties

    ## Type: simple | priority
    mqtt.mqueue.type = simple

    ## Topic Priority: 0~255, Default is 0
    ## mqtt.mqueue.priority = topic/1=10,topic/2=8

    ## Max queue length. Enqueued messages when persistent client disconnected,
    ## or inflight window is full. 0 means no limit.
    mqtt.mqueue.max_length = 0

    ## Low-water mark of queued messages
    mqtt.mqueue.low_watermark = 20%

    ## High-water mark of queued messages
    mqtt.mqueue.high_watermark = 60%

    ## Queue Qos0 messages?
    mqtt.mqueue.store_qos0 = true

队列参数说明:

+-----------------------------+---------------------------------------------------+
| mqueue.type                 | 队列类型。simple: 简单队列，priority: 优先级队列  |
+-----------------------------+---------------------------------------------------+
| mqueue.priority             | 主题(Topic)队列优先级设置                         |
+-----------------------------+---------------------------------------------------+
| mqueue.max_length           | 队列长度, infinity表示不限制                      |
+-----------------------------+---------------------------------------------------+
| mqueue.low_watermark        | 解除告警水位线                                    |
+-----------------------------+---------------------------------------------------+
| mqueue.high_watermark       | 队列满告警水位线                                  |
+-----------------------------+---------------------------------------------------+
| mqueue.store_qos0           | 是否缓存QoS0消息                                  |
+-----------------------------+---------------------------------------------------+

--------------
Broker心跳参数
--------------

设置系统发布$SYS/#消息周期:

.. code-block:: properties

    ## System Interval of publishing broker $SYS Messages
    mqtt.broker.sys_interval = 60

--------------------
发布订阅(PubSub)参数
--------------------

.. code-block:: properties

    ## PubSub Pool Size. Default should be scheduler numbers.
    mqtt.pubsub.pool_size = 8

    mqtt.pubsub.by_clientid = true

    ## Subscribe Asynchronously
    mqtt.pubsub.async = true

----------------
桥接(Bridge)参数
----------------

EMQ X 节点间可以桥接方式组网:

.. code-block:: properties

    ## Bridge Queue Size
    mqtt.bridge.max_queue_len = 10000

    ## Ping Interval of bridge node. Unit: Second
    mqtt.bridge.ping_down_interval = 1

----------------
插件配置文件目录
----------------

EMQ X 插件配置文件的存放路径:

.. code-block:: properties

    ## Dir of plugins' config
    mqtt.plugins.etc_dir ={{ platform_etc_dir }}/plugins/

    ## File to store loaded plugin names.
    mqtt.plugins.loaded_file = {{ platform_data_dir }}/loaded_plugins

---------------
MQTT 监听器配置
---------------

EMQ X 默认启用MQTT、MQTT/SSL、MQTT/WS、MQTT/WS/SSL监听器:

+-----------+-----------------------------------+
| 1883      | MQTT/TCP端口                      |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL端口                      |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket端口                |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket/SSL端口            |
+-----------+-----------------------------------+

EMQ X 允许为同一服务启用多个监听器，监听器主要参数:

+------------------------------------+----------------------------------------------------+
| listener.tcp.${name}.acceptors     | TCP Acceptor 池                                    |
+------------------------------------+----------------------------------------------------+
| listener.tcp.${name}.max_clients   | 最大允许 TCP 连接数                                |
+------------------------------------+----------------------------------------------------+
| listener.tcp.${name}.max_conn_rate | 最大连接创建速率，缺省每秒 1000                    |
+------------------------------------+----------------------------------------------------+
| listener.tcp.${name}.rate_limit    | 连接限速配置，例如限速1KB/秒，峰值10KB/秒:  "1,10" |
+------------------------------------+----------------------------------------------------+
| listener.tcp.${name}.access.${id}  | 限制客户端IP地址段                                 |
+------------------------------------+----------------------------------------------------+

---------------------
MQTT/TCP监听器 - 1883
---------------------

.. code-block:: properties

    ##--------------------------------------------------------------------
    ## MQTT/TCP - External TCP Listener for MQTT Protocol

    ## listener.tcp.<name> is the IP address and port that the MQTT/TCP
    ## listener will bind.
    ##
    ## Value: IP:Port | Port
    ##
    ## Examples: 1883, 127.0.0.1:1883, ::1:1883
    listener.tcp.external = 0.0.0.0:1883

    ## The acceptor pool for external MQTT/TCP listener.
    ##
    ## Value: Number
    listener.tcp.external.acceptors = 16

    ## Maximum number of concurrent MQTT/TCP connections.
    ##
    ## Value: Number
    listener.tcp.external.max_clients = 102400

    ## Maximum external connections per second.
    ##
    ## Value: Number
    listener.tcp.external.max_conn_rate = 1000

    ## Maximum publish rate of MQTT messages.
    ##
    ## Value: Number,Seconds
    ## Default: 10 messages per minute
    ## listener.tcp.external.max_publish_rate = 10,60

    ## TODO: Zone of the external MQTT/TCP listener belonged to.
    ##
    ## Value: String
    ## listener.tcp.external.zone = external

    ## Mountpoint of the MQTT/TCP Listener. All the topics of this
    ## listener will be prefixed with the mount point if this option
    ## is enabled.
    ## Notice that EMQ X supports wildcard mount:%c clientid, %u username
    ##
    ## Value: String
    ## listener.tcp.external.mountpoint = external/

    ## Rate limit for the external MQTT/TCP connections. Format is 'rate,burst'.
    ##
    ## Value: rate,burst
    ## Unit: Bps
    ## listener.tcp.external.rate_limit = 1024,4096

    ## The access control rules for the MQTT/TCP listener.
    ##
    ## See: https://github.com/emqtt/esockd#allowdeny
    ##
    ## Value: ACL Rule
    ##
    ## Example: allow 192.168.0.0/24
    listener.tcp.external.access.1 = allow all

    ## Enable the Proxy Protocol V1/2 if the EMQ cluster is deployed
    ## behind HAProxy or Nginx.
    ##
    ## See: https://www.haproxy.com/blog/haproxy/proxy-protocol/
    ##
    ## Value: on | off
    ## listener.tcp.external.proxy_protocol = on

    ## Sets the timeout for proxy protocol. EMQ will close the TCP connection
    ## if no proxy protocol packet recevied within the timeout.
    ##
    ## Value: Duration
    ## listener.tcp.external.proxy_protocol_timeout = 3s

    ## Enable the option for X.509 certificate based authentication.
    ## EMQ will Use the PP2_SUBTYPE_SSL_CN field in Proxy Protocol V2
    ## as MQTT username.
    ##
    ## Value: cn
    ## listener.tcp.external.peer_cert_as_username = cn

    ## The TCP backlog defines the maximum length that the queue of pending
    ## connections can grow to.
    ##
    ## Value: Number >= 0
    listener.tcp.external.backlog = 1024

    ## The TCP send timeout for external MQTT connections.
    ##
    ## Value: Duration
    listener.tcp.external.send_timeout = 15s

    ## Close the TCP connection if send timeout.
    ##
    ## Value: on | off
    listener.tcp.external.send_timeout_close = on

    ## The TCP receive buffer(os kernel) for MQTT connections.
    ##
    ## See: http://erlang.org/doc/man/inet.html
    ##
    ## Value: Bytes
    ## listener.tcp.external.recbuf = 4KB

    ## The TCP send buffer(os kernel) for MQTT connections.
    ##
    ## See: http://erlang.org/doc/man/inet.html
    ##
    ## Value: Bytes
    ## listener.tcp.external.sndbuf = 4KB

    ## The size of the user-level software buffer used by the driver.
    ## Not to be confused with options sndbuf and recbuf, which correspond
    ## to the Kernel socket buffers. It is recommended to have val(buffer)
    ## >= max(val(sndbuf),val(recbuf)) to avoid performance issues because
    ## of unnecessary copying. val(buffer) is automatically set to the above
    ## maximum when values sndbuf or recbuf are set.
    ##
    ## See: http://erlang.org/doc/man/inet.html
    ##
    ## Value: Bytes
    ## listener.tcp.external.buffer = 4KB

    ## Sets the 'buffer = max(sndbuf, recbuf)' if this option is enabled.
    ##
    ## Value: on | off
    ## listener.tcp.external.tune_buffer = off

    ## The TCP_NODELAY flag for MQTT connections. Small amounts of data are
    ## sent immediately if the option is enabled.
    ##
    ## Value: true | false
    listener.tcp.external.nodelay = true

    ## The SO_REUSEADDR flag for TCP listener.
    ##
    ## Value: true | false
    listener.tcp.external.reuseaddr = true

---------------------
MQTT/SSL监听器 - 8883
---------------------

SSL 安全连接监听器，默认支持 SSL 单向认证:

.. code-block:: properties

    ##--------------------------------------------------------------------
    ## MQTT/SSL - External SSL Listener for MQTT Protocol

    ## listener.ssl.<name> is the IP address and port that the MQTT/SSL
    ## listener will bind.
    ##
    ## Value: IP:Port | Port
    ##
    ## Examples: 8883, 127.0.0.1:8883, ::1:8883
    listener.ssl.external = 8883

    ## The acceptor pool for external MQTT/SSL listener.
    ##
    ## Value: Number
    listener.ssl.external.acceptors = 16

    ## Maximum number of concurrent MQTT/SSL connections.
    ##
    ## Value: Number
    listener.ssl.external.max_clients = 102400

    ## Maximum MQTT/SSL connections per second.
    ##
    ## Value: Number
    listener.ssl.external.max_conn_rate = 500

    ## Maximum publish rate of MQTT messages.
    ##
    ## See: listener.tcp.<name>.max_publish_rate
    ##
    ## Value: Number,Seconds
    ## Default: 10 messages per minute
    ## listener.ssl.external.max_publish_rate = 10,60

    ## TODO: Zone of the external MQTT/SSL listener belonged to.
    ##
    ## Value: String
    ## listener.ssl.external.zone = external

    ## Mountpoint of the MQTT/SSL Listener.
    ##
    ## Value: String
    ## listener.ssl.external.mountpoint = inbound/

    ## The access control rules for the MQTT/SSL listener.
    ##
    ## See: listener.tcp.<name>.access
    ##
    ## Value: ACL Rule
    listener.ssl.external.access.1 = allow all

    ## Rate limit for the external MQTT/SSL connections.
    ##
    ## Value: rate,burst
    ## Unit: Bps
    ## listener.ssl.external.rate_limit = 1024,4096

    ## Enable the Proxy Protocol V1/2 if the EMQ cluster is deployed behind
    ## HAProxy or Nginx.
    ##
    ## See: listener.tcp.<name>.proxy_protocol
    ##
    ## Value: on | off
    ## listener.ssl.external.proxy_protocol = on

    ## Sets the timeout for proxy protocol.
    ##
    ## See: listener.tcp.<name>.proxy_protocol_timeout
    ##
    ## Value: Duration
    ## listener.ssl.external.proxy_protocol_timeout = 3s

    ## TLS versions only to protect from POODLE attack.
    ##
    ## See: http://erlang.org/doc/man/ssl.html
    ##
    ## Value: String, seperated by ','
    ## listener.ssl.external.tls_versions = tlsv1.2,tlsv1.1,tlsv1

    ## TLS Handshake timeout.
    ##
    ## Value: Duration
    listener.ssl.external.handshake_timeout = 15s

    ## Path to the file containing the user's private PEM-encoded key.
    ##
    ## See: http://erlang.org/doc/man/ssl.html
    ##
    ## Value: File
    listener.ssl.external.keyfile = {{ platform_etc_dir }}/certs/key.pem

    ## Path to a file containing the user certificate.
    ##
    ## See: http://erlang.org/doc/man/ssl.html
    ##
    ## Value: File
    listener.ssl.external.certfile = {{ platform_etc_dir }}/certs/cert.pem

    ## Path to the file containing PEM-encoded CA certificates. The CA certificates
    ## are used during server authentication and when building the client certificate chain.
    ##
    ## Value: File
    ## listener.ssl.external.cacertfile = {{ platform_etc_dir }}/certs/cacert.pem

    ## The Ephemeral Diffie-Helman key exchange is a very effective way of
    ## ensuring Forward Secrecy by exchanging a set of keys that never hit
    ## the wire. Since the DH key is effectively signed by the private key,
    ## it needs to be at least as strong as the private key. In addition,
    ## the default DH groups that most of the OpenSSL installations have
    ## are only a handful (since they are distributed with the OpenSSL
    ## package that has been built for the operating system it’s running on)
    ## and hence predictable (not to mention, 1024 bits only).
    ## In order to escape this situation, first we need to generate a fresh,
    ## strong DH group, store it in a file and then use the option above,
    ## to force our SSL application to use the new DH group. Fortunately,
    ## OpenSSL provides us with a tool to do that. Simply run:
    ## openssl dhparam -out dh-params.pem 2048
    ##
    ## Value: File
    ## listener.ssl.external.dhfile = {{ platform_etc_dir }}/certs/dh-params.pem

    ## A server only does x509-path validation in mode verify_peer,
    ## as it then sends a certificate request to the client (this
    ## message is not sent if the verify option is verify_none).
    ## You can then also want to specify option fail_if_no_peer_cert.
    ## More information at: http://erlang.org/doc/man/ssl.html
    ##
    ## Value: verify_peer | verify_none
    ## listener.ssl.external.verify = verify_peer

    ## Used together with {verify, verify_peer} by an SSL server. If set to true,
    ## the server fails if the client does not have a certificate to send, that is,
    ## sends an empty certificate.
    ##
    ## Value: true | false
    ## listener.ssl.external.fail_if_no_peer_cert = true

    ## This is the single most important configuration option of an Erlang SSL
    ## application. Ciphers (and their ordering) define the way the client and
    ## server encrypt information over the wire, from the initial Diffie-Helman
    ## key exchange, the session key encryption ## algorithm and the message
    ## digest algorithm. Selecting a good cipher suite is critical for the
    ## application’s data security, confidentiality and performance.
    ##
    ## The cipher list above offers:
    ##
    ## A good balance between compatibility with older browsers.
    ## It can get stricter for Machine-To-Machine scenarios.
    ## Perfect Forward Secrecy.
    ## No old/insecure encryption and HMAC algorithms
    ##
    ## Most of it was copied from Mozilla’s Server Side TLS article
    ##
    ## Value: Ciphers
    ## listener.ssl.external.ciphers = ECDHE-ECDSA-AES256-GCM-SHA384,ECDHE-RSA-AES256-GCM-SHA384,ECDHE-ECDSA-AES256-SHA384,ECDHE-RSA-AES256-SHA384,ECDHE-ECDSA-DES-CBC3-SHA,ECDH-ECDSA-AES256-GCM-SHA384,ECDH-RSA-AES256-GCM-SHA384,ECDH-ECDSA-AES256-SHA384,ECDH-RSA-AES256-SHA384,DHE-DSS-AES256-GCM-SHA384,DHE-DSS-AES256-SHA256,AES256-GCM-SHA384,AES256-SHA256,ECDHE-ECDSA-AES128-GCM-SHA256,ECDHE-RSA-AES128-GCM-SHA256,ECDHE-ECDSA-AES128-SHA256,ECDHE-RSA-AES128-SHA256,ECDH-ECDSA-AES128-GCM-SHA256,ECDH-RSA-AES128-GCM-SHA256,ECDH-ECDSA-AES128-SHA256,ECDH-RSA-AES128-SHA256,DHE-DSS-AES128-GCM-SHA256,DHE-DSS-AES128-SHA256,AES128-GCM-SHA256,AES128-SHA256,ECDHE-ECDSA-AES256-SHA,ECDHE-RSA-AES256-SHA,DHE-DSS-AES256-SHA,ECDH-ECDSA-AES256-SHA,ECDH-RSA-AES256-SHA,AES256-SHA,ECDHE-ECDSA-AES128-SHA,ECDHE-RSA-AES128-SHA,DHE-DSS-AES128-SHA,ECDH-ECDSA-AES128-SHA,ECDH-RSA-AES128-SHA,AES128-SHA

    ## SSL parameter renegotiation is a feature that allows a client and a server
    ## to renegotiate the parameters of the SSL connection on the fly.
    ## RFC 5746 defines a more secure way of doing this. By enabling secure renegotiation,
    ## you drop support for the insecure renegotiation, prone to MitM attacks.
    ##
    ## Value: on | off
    ## listener.ssl.external.secure_renegotiate = off

    ## A performance optimization setting, it allows clients to reuse
    ## pre-existing sessions, instead of initializing new ones.
    ## Read more about it here.
    ##
    ## See: http://erlang.org/doc/man/ssl.html
    ##
    ## Value: on | off
    ## listener.ssl.external.reuse_sessions = on

    ## An important security setting, it forces the cipher to be set based
    ## on the server-specified order instead of the client-specified order,
    ## hence enforcing the (usually more properly configured) security
    ## ordering of the server administrator.
    ##
    ## Value: on | off
    ## listener.ssl.external.honor_cipher_order = on

    ## Use the CN or DN value from the client certificate as a username.
    ## Notice that 'verify' should be set as 'verify_peer'.
    ##
    ## Value: cn | dn
    ## listener.ssl.external.peer_cert_as_username = cn

    ## TCP backlog for the SSL connection.
    ##
    ## See listener.tcp.<name>.backlog
    ##
    ## Value: Number >= 0
    ## listener.ssl.external.backlog = 1024

    ## The TCP send timeout for the SSL connection.
    ##
    ## See listener.tcp.<name>.send_timeout
    ##
    ## Value: Duration
    ## listener.ssl.external.send_timeout = 15s

    ## Close the SSL connection if send timeout.
    ##
    ## See: listener.tcp.<name>.send_timeout_close
    ##
    ## Value: on | off
    ## listener.ssl.external.send_timeout_close = on

    ## The TCP receive buffer(os kernel) for the SSL connections.
    ##
    ## See: listener.tcp.<name>.recbuf
    ##
    ## Value: Bytes
    ## listener.ssl.external.recbuf = 4KB

    ## The TCP send buffer(os kernel) for internal MQTT connections.
    ##
    ## See: listener.tcp.<name>.sndbuf
    ##
    ## Value: Bytes
    ## listener.ssl.external.sndbuf = 4KB

    ## The size of the user-level software buffer used by the driver.
    ##
    ## See: listener.tcp.<name>.buffer
    ##
    ## Value: Bytes
    ## listener.ssl.external.buffer = 4KB

    ## Sets the 'buffer = max(sndbuf, recbuf)' if this option is enabled.
    ##
    ## See: listener.tcp.<name>.tune_buffer
    ##
    ## Value: on | off
    ## listener.ssl.external.tune_buffer = off

    ## The TCP_NODELAY flag for SSL connections.
    ##
    ## See: listener.tcp.<name>.nodelay
    ##
    ## Value: true | false
    ## listener.ssl.external.nodelay = true

    ## The SO_REUSEADDR flag for MQTT/SSL Listener.
    ##
    ## Value: true | false
    listener.ssl.external.reuseaddr = true

---------------------------
MQTT/WebSocket监听器 - 8083
---------------------------

.. code-block:: properties

    ## HTTP and WebSocket Listener
    listener.http.external = 8083

    listener.http.external.acceptors = 4

    listener.http.external.max_clients = 64

    ## listener.http.external.zone = external

    listener.http.external.access.1 = allow all

-------------------------------
MQTT/WebSocket/SSL监听器 - 8084
-------------------------------

WebSocket安全连接监听器，默认开启单向SSL认证:

.. code-block:: properties

    ## External HTTPS and WSS Listener

    listener.https.external = 8084

    listener.https.external.acceptors = 4

    listener.https.external.max_clients = 64

    ## listener.https.external.zone = external

    listener.https.external.access.1 = allow all

    ## SSL Options
    listener.https.external.handshake_timeout = 15s

    listener.https.external.keyfile = {{ platform_etc_dir }}/certs/key.pem

    listener.https.external.certfile = {{ platform_etc_dir }}/certs/cert.pem

    ## listener.https.external.cacertfile = {{ platform_etc_dir }}/certs/cacert.pem

    ## listener.https.external.verify = verify_peer

    ## listener.https.external.fail_if_no_peer_cert = true

-----------------
Erlang VM监控设置
-----------------

.. code-block:: properties

    ## Long GC, don't monitor in production mode for:
    sysmon.long_gc = false

    ## Long Schedule(ms)
    sysmon.long_schedule = 240

    ## 8M words. 32MB on 32-bit VM, 64MB on 64-bit VM.
    sysmon.large_heap = 8MB

    ## Busy Port
    sysmon.busy_port = false

    ## Busy Dist Port
    sysmon.busy_dist_port = true

