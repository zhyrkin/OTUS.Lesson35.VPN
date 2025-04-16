# OTUS.Lesson35.VPN
Тестовый стенд:
2 VM (2CPU, 2GB RAM, 10Gb HDD, OS Ubuntu 22.04)
Описание домашнего задания:
1) Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях
2) Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ


1) Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях
    Запустим плейбук VPN.part1.yml. Укажем переменной dev_type режим tap. После выполнения плейбука на сервере запустим ipef в режиме серевера (iperf3 -s)
    и на клиенте  'iperf3 -c 10.10.10.1 -t 40 -i 5' и зафиксируем результат
    ```
zhurkin@vpnclient:~$ iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 58616 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  66.4 MBytes   111 Mbits/sec   50   1.17 MBytes       
[  5]   5.00-10.00  sec  58.8 MBytes  98.6 Mbits/sec  903    328 KBytes       
[  5]  10.00-15.00  sec  58.8 MBytes  98.6 Mbits/sec   34    299 KBytes       
[  5]  15.00-20.00  sec  57.5 MBytes  96.5 Mbits/sec   42    264 KBytes       
[  5]  20.00-25.00  sec  61.2 MBytes   103 Mbits/sec    0    396 KBytes       
[  5]  25.00-30.00  sec  61.2 MBytes   103 Mbits/sec    0    492 KBytes       
[  5]  30.00-35.00  sec  61.2 MBytes   103 Mbits/sec    0   1.12 MBytes       
[  5]  35.00-40.00  sec  60.0 MBytes   101 Mbits/sec   66   1.26 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec   485 MBytes   102 Mbits/sec  1095             sender
[  5]   0.00-40.14  sec   482 MBytes   101 Mbits/sec                  receiver

iperf Done.
```
Поменяем в плейбуке переменную dev_type на tun, запустим плейбук и проверим результат работы в режиме tun 
```
zhurkin@vpnclient:~$ iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 39850 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  63.8 MBytes   107 Mbits/sec   31    477 KBytes       
[  5]   5.00-10.00  sec  66.0 MBytes   111 Mbits/sec    0    552 KBytes       
[  5]  10.00-15.00  sec  62.3 MBytes   105 Mbits/sec  223    238 KBytes       
[  5]  15.00-20.00  sec  58.9 MBytes  98.8 Mbits/sec   35    240 KBytes       
[  5]  20.00-25.00  sec  59.0 MBytes  99.0 Mbits/sec   64    205 KBytes       
[  5]  25.00-30.00  sec  60.4 MBytes   101 Mbits/sec    0    359 KBytes       
[  5]  30.00-35.00  sec  56.4 MBytes  94.6 Mbits/sec    0    458 KBytes       
[  5]  35.00-40.00  sec  60.3 MBytes   101 Mbits/sec  180    460 KBytes       

[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec   487 MBytes   102 Mbits/sec  533             sender
[  5]   0.00-40.08  sec   485 MBytes   102 Mbits/sec                  receiver
```
В тестовой среде результаты +- одинаковые, но в продакшене лучше использовать режим tun, так как этот режим работает только с ip адресами и не нагружает систему обработкой


2. RAS на базе OpenVPN
Запускаем VPN.part2.yml. Лучше это сделать на чистых VM.
После завершения работы плейбука зайдем на client и подключимся к серверу 'openvpn --config client.conf'
```
root@vpnclient:~# openvpn --config client.conf
2025-04-16 12:58:39 WARNING: Compression for receiving enabled. Compression has been used in the past to break encryption. Sent packets are not compressed unless "allow-compression yes" is also set.
2025-04-16 12:58:39 --cipher is not set. Previous OpenVPN version defaulted to BF-CBC as fallback when cipher negotiation failed in this case. If you need this fallback please add '--data-ciphers-fallback BF-CBC' to your configuration and/or add BF-CBC to --data-ciphers.
2025-04-16 12:58:39 OpenVPN 2.5.11 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Sep 17 2024
2025-04-16 12:58:39 library versions: OpenSSL 3.0.2 15 Mar 2022, LZO 2.10
2025-04-16 12:58:39 TCP/UDP: Preserving recently used remote address: [AF_INET]10.200.3.95:1207
2025-04-16 12:58:39 Socket Buffers: R=[212992->212992] S=[212992->212992]
2025-04-16 12:58:39 UDP link local (bound): [AF_INET][undef]:1194
2025-04-16 12:58:39 UDP link remote: [AF_INET]10.200.3.95:1207
2025-04-16 12:58:39 TLS: Initial packet from [AF_INET]10.200.3.95:1207, sid=21cd0934 f03e6884
2025-04-16 12:58:39 VERIFY OK: depth=1, CN=server
2025-04-16 12:58:39 VERIFY KU OK
2025-04-16 12:58:39 Validating certificate extended key usage
2025-04-16 12:58:39 ++ Certificate has EKU (str) TLS Web Server Authentication, expects TLS Web Server Authentication
2025-04-16 12:58:39 VERIFY EKU OK
2025-04-16 12:58:39 VERIFY OK: depth=0, CN=server
2025-04-16 12:58:39 Control Channel: TLSv1.3, cipher TLSv1.3 TLS_AES_256_GCM_SHA384, peer certificate: 2048 bit RSA, signature: RSA-SHA256
2025-04-16 12:58:39 [server] Peer Connection Initiated with [AF_INET]10.200.3.95:1207
2025-04-16 12:58:39 PUSH: Received control message: 'PUSH_REPLY,route 10.10.10.0 255.255.255.0,topology net30,ping 10,ping-restart 120,ifconfig 10.10.10.6 10.10.10.5,peer-id 0,cipher AES-256-GCM'
2025-04-16 12:58:39 OPTIONS IMPORT: timers and/or timeouts modified
2025-04-16 12:58:39 OPTIONS IMPORT: --ifconfig/up options modified
2025-04-16 12:58:39 OPTIONS IMPORT: route options modified
2025-04-16 12:58:39 OPTIONS IMPORT: peer-id set
2025-04-16 12:58:39 OPTIONS IMPORT: adjusting link_mtu to 1625
2025-04-16 12:58:39 OPTIONS IMPORT: data channel crypto options modified
2025-04-16 12:58:39 Data Channel: using negotiated cipher 'AES-256-GCM'
2025-04-16 12:58:39 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
2025-04-16 12:58:39 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
2025-04-16 12:58:39 net_route_v4_best_gw query: dst 0.0.0.0
2025-04-16 12:58:39 net_route_v4_best_gw result: via 10.200.3.1 dev ens18
2025-04-16 12:58:39 ROUTE_GATEWAY 10.200.3.1/255.255.255.0 IFACE=ens18 HWADDR=bc:24:11:b8:07:77
2025-04-16 12:58:39 TUN/TAP device tun0 opened
2025-04-16 12:58:39 net_iface_mtu_set: mtu 1500 for tun0
2025-04-16 12:58:39 net_iface_up: set tun0 up
2025-04-16 12:58:39 net_addr_ptp_v4_add: 10.10.10.6 peer 10.10.10.5 dev tun0
2025-04-16 12:58:39 net_route_v4_add: 10.10.10.0/24 via 10.10.10.5 dev [NULL] table 0 metric -1
2025-04-16 12:58:39 net_route_v4_add: 10.10.10.0/24 via 10.10.10.5 dev [NULL] table 0 metric -1
2025-04-16 12:58:39 Initialization Sequence Completed
```

При успешном подключении проверяем пинг по внутреннему IP адресу  сервера в туннеле: ping -c 4 10.10.10.1 
```
zhurkin@vpnclient:~$ ping -c 4 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=1.45 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=1.19 ms
^C
--- 10.10.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.185/1.316/1.447/0.131 ms
```
