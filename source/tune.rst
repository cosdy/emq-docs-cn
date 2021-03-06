
========
测试调优
========

*EMQ* 消息服务器1.x版本MQTT连接压力测试到130万，在一台12核心、32G内存的CentOS服务器上。

100万连接测试所需的Linux内核参数，网络协议栈参数，Erlang虚拟机参数，emqttd消息服务器参数设置如下:

-----------------
Linux操作系统参数
-----------------

系统全局允许分配的最大文件句柄数::

    # 2 millions system-wide
    sysctl -w fs.file-max=2097152
    sysctl -w fs.nr_open=2097152
    echo 2097152 > /proc/sys/fs/nr_open

允许当前会话/进程打开文件句柄数::

    ulimit -n 1048576

/etc/sysctl.conf
----------------

持久化'fs.file-max'设置到/etc/sysctl.conf文件::

    fs.file-max = 1048576

/etc/security/limits.conf
-------------------------

/etc/security/limits.conf持久化设置允许用户/进程打开文件句柄数::

    *      soft   nofile      1048576
    *      hard   nofile      1048576

-----------------
TCP协议栈网络参数
-----------------

并发连接backlog设置::

    sysctl -w net.core.somaxconn=32768
    sysctl -w net.ipv4.tcp_max_syn_backlog=16384
    sysctl -w net.core.netdev_max_backlog=16384

可用知名端口范围::

    sysctl -w net.ipv4.ip_local_port_range='1000 65535'

TCP Socket读写Buffer设置::

    sysctl -w net.core.rmem_default=262144
    sysctl -w net.core.wmem_default=262144
    sysctl -w net.core.rmem_max=16777216
    sysctl -w net.core.wmem_max=16777216
    sysctl -w net.core.optmem_max=16777216

    #sysctl -w net.ipv4.tcp_mem='16777216 16777216 16777216'
    sysctl -w net.ipv4.tcp_rmem='1024 4096 16777216'
    sysctl -w net.ipv4.tcp_wmem='1024 4096 16777216'

TCP连接追踪设置::

    sysctl -w net.nf_conntrack_max=1000000
    sysctl -w net.netfilter.nf_conntrack_max=1000000
    sysctl -w net.netfilter.nf_conntrack_tcp_timeout_time_wait=30

TIME-WAIT Socket最大数量、回收与重用设置::

    net.ipv4.tcp_max_tw_buckets=1048576

    # 注意: 不建议开启該设置，NAT模式下可能引起连接RST
    # net.ipv4.tcp_tw_recycle = 1
    # net.ipv4.tcp_tw_reuse = 1

FIN-WAIT-2 Socket超时设置::

    net.ipv4.tcp_fin_timeout = 15

----------------
Erlang虚拟机参数
----------------

优化设置Erlang虚拟机启动参数，配置文件emqttd/etc/emq.conf:

.. code-block:: properties

    ## Erlang Process Limit
    node.process_limit = 2097152

    ## Sets the maximum number of simultaneously existing ports for this system
    node.max_ports = 1048576

-----------------
EMQ消息服务器参数
-----------------

设置TCP监听器的Acceptor池大小，最大允许连接数。配置文件emqttd/etc/emq.conf:

.. code-block:: properties

    ## TCP Listener
    mqtt.listener.tcp = 1883
    mqtt.listener.tcp.acceptors = 64
    mqtt.listener.tcp.max_clients = 1000000

--------------
测试客户端设置
--------------

测试客户端服务器在一个接口上，最多只能创建65000连接::

    sysctl -w net.ipv4.ip_local_port_range="500 65535"
    echo 1000000 > /proc/sys/fs/nr_open
    ulimit -n 100000

emqtt_benchmark
---------------

并发连接测试工具: http://github.com/emqtt/emqtt_benchmark

