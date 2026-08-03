Создаем простанство имён homework:
kubectl create ns otus-debug

Создаем под из манифеста:
kubectl apply -f pod.yaml

Создаем эфемерный контейнер с доступом к пространству имён pid:
kubectl debug -n otus-debug -it nginx-distroless --image=alpine:latest --target=nginx --share-processes=true --profile=general


Выводим содержимое директории /etc/nginx контейнера nginx-distroless:
/ # ls -la /proc/1/root/etc/nginx/
total 48
drwxr-xr-x    3 root     root          4096 Oct  5  2020 .
drwxr-xr-x    1 root     root          4096 Aug  3 03:47 ..
drwxr-xr-x    2 root     root          4096 Oct  5  2020 conf.d
-rw-r--r--    1 root     root          1007 Apr 21  2020 fastcgi_params
-rw-r--r--    1 root     root          2837 Apr 21  2020 koi-utf
-rw-r--r--    1 root     root          2223 Apr 21  2020 koi-win
-rw-r--r--    1 root     root          5231 Apr 21  2020 mime.types
lrwxrwxrwx    1 root     root            22 Apr 21  2020 modules -> /usr/lib/nginx/modules
-rw-r--r--    1 root     root           643 Apr 21  2020 nginx.conf
-rw-r--r--    1 root     root           636 Apr 21  2020 scgi_params
-rw-r--r--    1 root     root           664 Apr 21  2020 uwsgi_params
-rw-r--r--    1 root     root          3610 Apr 21  2020 win-utf


Отладка сети
установка нужных программ
apk add --no-cache tcpdump curl

Запускаем tcpdump
tcpdump -nn -i any -e port 80

В соседнем терминале запускаем несколько раз curl
kubectl exec nginx-distroless -n otus-debug -c $(kubectl get pod/nginx-distroless -n otus-debug -o jsonpath='{.spec.ephemeralContainers[0].name}') -- sh -c "curl http://localhost"


Логи работы tcpdump
tcpdump: WARNING: any: That device doesn't support promiscuous mode
(Promiscuous mode not supported on the "any" device)
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
03:53:00.161238 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv6 (0x86dd), length 100: ::1.57918 > ::1.80: Flags [S], seq 3915711997, win 65476, options [mss 65476,sackOK,TS val 3864623336 ecr 0,nop,wscale 7], length 0
03:53:00.161243 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv6 (0x86dd), length 80: ::1.80 > ::1.57918: Flags [R.], seq 0, ack 3915711998, win 0, length 0
03:53:00.161273 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.55436 > 127.0.0.1.80: Flags [S], seq 154486770, win 65495, options [mss 65495,sackOK,TS val 659130528 ecr 0,nop,wscale 7], length 0
03:53:00.161278 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.80 > 127.0.0.1.55436: Flags [S.], seq 3967440690, ack 154486771, win 65483, options [mss 65495,sackOK,TS val 659130528 ecr 659130528,nop,wscale 7], length 0
03:53:00.161282 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.55436 > 127.0.0.1.80: Flags [.], ack 1, win 512, options [nop,nop,TS val 659130528 ecr 659130528], length 0
03:53:00.161355 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 145: 127.0.0.1.55436 > 127.0.0.1.80: Flags [P.], seq 1:74, ack 1, win 512, options [nop,nop,TS val 659130528 ecr 659130528], length 73: HTTP: GET / HTTP/1.1
03:53:00.161360 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.55436: Flags [.], ack 74, win 512, options [nop,nop,TS val 659130528 ecr 659130528], length 0
03:53:00.161552 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 310: 127.0.0.1.80 > 127.0.0.1.55436: Flags [P.], seq 1:239, ack 74, win 512, options [nop,nop,TS val 659130528 ecr 659130528], length 238: HTTP: HTTP/1.1 200 OK
03:53:00.161559 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.55436 > 127.0.0.1.80: Flags [.], ack 239, win 511, options [nop,nop,TS val 659130529 ecr 659130528], length 0
03:53:00.161580 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 684: 127.0.0.1.80 > 127.0.0.1.55436: Flags [P.], seq 239:851, ack 74, win 512, options [nop,nop,TS val 659130529 ecr 659130529], length 612: HTTP
03:53:00.161582 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.55436 > 127.0.0.1.80: Flags [.], ack 851, win 507, options [nop,nop,TS val 659130529 ecr 659130529], length 0
03:53:00.161846 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.55436 > 127.0.0.1.80: Flags [F.], seq 74, ack 851, win 507, options [nop,nop,TS val 659130529 ecr 659130529], length 0
03:53:00.161920 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.55436: Flags [F.], seq 851, ack 75, win 512, options [nop,nop,TS val 659130529 ecr 659130529], length 0
03:53:00.161927 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.55436 > 127.0.0.1.80: Flags [.], ack 852, win 507, options [nop,nop,TS val 659130529 ecr 659130529], length 0
03:53:53.344013 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv6 (0x86dd), length 100: ::1.41370 > ::1.80: Flags [S], seq 3050619681, win 65476, options [mss 65476,sackOK,TS val 3864676519 ecr 0,nop,wscale 7], length 0
03:53:53.344019 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv6 (0x86dd), length 80: ::1.80 > ::1.41370: Flags [R.], seq 0, ack 3050619682, win 0, length 0
03:53:53.344049 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.42656 > 127.0.0.1.80: Flags [S], seq 3208819541, win 65495, options [mss 65495,sackOK,TS val 659183711 ecr 0,nop,wscale 7], length 0
03:53:53.344054 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.80 > 127.0.0.1.42656: Flags [S.], seq 1723704660, ack 3208819542, win 65483, options [mss 65495,sackOK,TS val 659183711 ecr 659183711,nop,wscale 7], length 0
03:53:53.344058 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.42656 > 127.0.0.1.80: Flags [.], ack 1, win 512, options [nop,nop,TS val 659183711 ecr 659183711], length 0
03:53:53.344157 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 145: 127.0.0.1.42656 > 127.0.0.1.80: Flags [P.], seq 1:74, ack 1, win 512, options [nop,nop,TS val 659183711 ecr 659183711], length 73: HTTP: GET / HTTP/1.1
03:53:53.344162 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.42656: Flags [.], ack 74, win 512, options [nop,nop,TS val 659183711 ecr 659183711], length 0
03:53:53.344261 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 310: 127.0.0.1.80 > 127.0.0.1.42656: Flags [P.], seq 1:239, ack 74, win 512, options [nop,nop,TS val 659183711 ecr 659183711], length 238: HTTP: HTTP/1.1 200 OK
03:53:53.344268 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.42656 > 127.0.0.1.80: Flags [.], ack 239, win 511, options [nop,nop,TS val 659183711 ecr 659183711], length 0
03:53:53.344280 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 684: 127.0.0.1.80 > 127.0.0.1.42656: Flags [P.], seq 239:851, ack 74, win 512, options [nop,nop,TS val 659183711 ecr 659183711], length 612: HTTP
03:53:53.344284 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.42656 > 127.0.0.1.80: Flags [.], ack 851, win 507, options [nop,nop,TS val 659183711 ecr 659183711], length 0
03:53:53.344371 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.42656 > 127.0.0.1.80: Flags [F.], seq 74, ack 851, win 507, options [nop,nop,TS val 659183711 ecr 659183711], length 0
03:53:53.344568 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.42656: Flags [F.], seq 851, ack 75, win 512, options [nop,nop,TS val 659183712 ecr 659183711], length 0
03:53:53.344575 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.42656 > 127.0.0.1.80: Flags [.], ack 852, win 507, options [nop,nop,TS val 659183712 ecr 659183712], length 0


Ищем на какой ноде запущен pod
root@work01:~# kubectl get pod nginx-distroless -n otus-debug -o wide
NAME               READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
nginx-distroless   1/1     Running   0          12m   10.244.3.81   work03.lan   <none>           <none>

Смотрим логи на ноде work03.lan
root@work03:~# ll /var/log/pods/otus-debug_nginx-distroless_1dd1edfd-4a39-4144-8e85-04c3b78e0020/nginx/0.log 
-rw-r----- 1 root root 390 Aug  3 03:57 /var/log/pods/otus-debug_nginx-distroless_1dd1edfd-4a39-4144-8e85-04c3b78e0020/nginx/0.log
root@work03:~# cat /var/log/pods/otus-debug_nginx-distroless_1dd1edfd-4a39-4144-8e85-04c3b78e0020/nginx/0.log 
2026-08-03T03:53:00.161627557Z stdout F 127.0.0.1 - - [03/Aug/2026:11:53:00 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/8.21.0" "-"
2026-08-03T03:53:53.344442863Z stdout F 127.0.0.1 - - [03/Aug/2026:11:53:53 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/8.21.0" "-"
2026-08-03T03:57:01.820833234Z stdout F 127.0.0.1 - - [03/Aug/2026:11:57:01 +0800] "GET / HTTP/1.1" 200 612 "-" "curl/8.21.0" "-"