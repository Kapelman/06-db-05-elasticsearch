# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.

Решение:

- получим образ образ centos:7:
```
vagrant@vagrant:~/elastic-search$ sudo docker pull centos:7
7: Pulling from library/centos
2d473b07cdd5: Pull complete
Digest: sha256:c73f515d06b0fa07bb18d8202035e739a494ce760aa73129f60f4bf2bd22b407
Status: Downloaded newer image for centos:7
docker.io/library/centos:7
vagrant@vagrant:~/elastic-search$ sudo docker image ls
centos                                          7             eeb6ee3f44bd   12 months ago            204MB

vagrant@vagrant:~/elastic-search$ sudo docker image ls
REPOSITORY                 TAG           IMAGE ID       CREATED                  SIZE
centos                     7             eeb6ee3f44bd   12 months ago            204MB
centos                     latest        5d0da3dc9764   12 months ago            231MB
```
- создадим свой образ контейнера my-es-image
```
vagrant@vagrant:~/elastic-search$ sudo docker build -t my-es-image .
Sending build context to Docker daemon  566.3MB
Step 1/22 : FROM        centos:7
 ---> eeb6ee3f44bd
Step 2/22 : RUN mkdir /var/lib/elasticsearch
 ---> Using cache
 ---> 9a11b7f59dd6
Step 3/22 : RUN mkdir /var/log/elasticsearch
 ---> Using cache
 ---> 94e2f5c9a58b
Step 4/22 : RUN yum -y install java-1.8.0-openjdk.x86_64
 ---> Using cache
 ---> 099896cd2407
Step 5/22 : RUN yum -y install wget
 ---> Using cache
 ---> 3c8b84907b9c
Step 6/22 : COPY elasticsearch-8.4.1-linux-x86_64.tar.gz ./
 ---> Using cache
 ---> 757690f814b2
Step 7/22 : RUN tar -xzf elasticsearch-8.4.1-linux-x86_64.tar.gz
 ---> Using cache
 ---> a7766a5386a8
Step 8/22 : RUN  cd elasticsearch-8.4.1
 ---> Using cache
 ---> 1d94dd5d75e3
Step 9/22 : RUN groupadd -g 1000 elasticsearch && useradd elasticsearch -u 1000 -g 1000
 ---> Using cache
 ---> c648242eb192
Step 10/22 : WORKDIR /elasticsearch-8.4.1
 ---> Using cache
 ---> 89daddf87f0f
Step 11/22 : RUN cd /elasticsearch-8.4.1
 ---> Using cache
 ---> f9d09585193a
Step 12/22 : RUN set -ex && for path in data logs config config/scripts bin jdk; do         mkdir -p "$path";         chown -R elasticsearch:elasticsearch "$path";     done
 ---> Using cache
 ---> 1639e618c490
Step 13/22 : RUN  chown -R elasticsearch:elasticsearch /elasticsearch-8.4.1/jdk/bin/
 ---> Using cache
 ---> ae2a6bef04d5
Step 14/22 : RUN  chown -R elasticsearch:elasticsearch /elasticsearch-8.4.1/jdk/bin/java
 ---> Using cache
 ---> ba75dc3d71b4
Step 15/22 : RUN  chown -R elasticsearch:elasticsearch /var/lib/elasticsearch
 ---> Using cache
 ---> a435a9849153
Step 16/22 : RUN  chown -R elasticsearch:elasticsearch /var/log/elasticsearch
 ---> Using cache
 ---> 68c63d60e907
Step 17/22 : USER elasticsearch
 ---> Using cache
 ---> a7a7e99219d9
Step 18/22 : COPY logging.yml /elasticsearch-8.4.1/config/
 ---> Using cache
 ---> 88384dc066e8
Step 19/22 : COPY elasticsearch.yml /elasticsearch-8.4.1/config/
 ---> Using cache
 ---> ca7e2fa8cc65
Step 20/22 : ENV PATH=$PATH:/elasticsearch-8.4.1/bin
 ---> Using cache
 ---> 36bb87777bd3
Step 21/22 : CMD ["elasticsearch"]
 ---> Using cache
 ---> 846cded9ed27
Step 22/22 : EXPOSE 9200 9300
 ---> Using cache
 ---> 0b1917e11265
Successfully built 0b1917e11265
Successfully tagged my-es-image:latest

vagrant@vagrant:~/elastic-search$ sudo docker image ls
REPOSITORY                 TAG           IMAGE ID       CREATED                  SIZE
postgres                   13            e30fb19f297b   Less than a second ago   373MB
my-es-image                latest        35b091a078ee   51 minutes ago           2.78GB
```
- отправим свой образ  в  docker.io репозиторий
```
vagrant@vagrant:~/elastic-search$ sudo docker tag  my-es-image kapelman/06-db-05-elasticsearch:es-image
vagrant@vagrant:~/elastic-search$ sudo docker image ls
REPOSITORY                        TAG           IMAGE ID       CREATED                  SIZE
postgres                          13            e30fb19f297b   Less than a second ago   373MB
kapelman/06-db-05-elasticsearch   es-image      35b091a078ee   2 hours ago              2.78GB
my-es-image                       latest        35b091a078ee   2 hours ago              2.78GB
mysql                             8.0           ff3b5098b416   13 days ago              447MB
postgres                          12            f2f1f275f1a1   2 weeks ago              373MB
adminer                           latest        75cd6c93316c   4 weeks ago              90.7MB
kapelman/05-virt-02-iaac          first-image   11b04307b160   8 weeks ago              142MB
kaplin-nginx                      first_ver     11b04307b160   8 weeks ago              142MB
nginx                             latest        41b0e86104ba   2 months ago             142MB
debian                            latest        123c2f3835fd   2 months ago             124MB
hello-world                       latest        feb5d9fea6a5   11 months ago            13.3kB
centos                            7             eeb6ee3f44bd   12 months ago            204MB
centos                            latest        5d0da3dc9764   12 months ago            231MB
vagrant@vagrant:~/elastic-search$ sudo docker push kapelman/06-db-05-elasticsearch:es-image
The push refers to repository [docker.io/kapelman/06-db-05-elasticsearch]
ddec43cf9b21: Pushed
7060d12e620e: Pushed
391aa29e8a74: Pushed
a9f80196cdf0: Pushed
a27bbdad1fa2: Pushed
5b642024fa86: Pushed
4891ec7f4e9c: Pushed
3190dc197c8a: Pushed
adfe99a9958e: Pushed
4f99dc9c3e90: Pushed
35fa8d4e6490: Pushed
066b48516e36: Pushed
41b1789b77ab: Pushed
cb1ebb79b7db: Pushed
adbfaeace5bd: Pushed
28cc9d89d851: Pushed
174f56854903: Mounted from library/centos
es-image: digest: sha256:91182de6e56a86ad250ff88c522b31a899a009c000a11490453e5d0dc41e126f size: 3875
```
ссылка на образ в Registry:
https://hub.docker.com/layers/kapelman/06-db-05-elasticsearch/es-image/images/sha256-91182de6e56a86ad250ff88c522b31a899a009c000a11490453e5d0dc41e126f?context=repo


- запустим контейнер:
```
sudo docker run -p 9200:9200 -p 9300:9300 -d my-es-image

[2022-09-13T00:11:34,997][INFO ][o.e.n.Node               ] [c3df0a5f344a] version[8.4.1], pid[74], build[tar/2bd229c8e56650b42e40992322a76e7914258f0c/2022-08-26T12:11:43.232597118Z], OS[Linux/5.4.0-121-generic/amd64], JVM[Oracle Corporation/OpenJDK 64-Bit Server VM/18.0.2/18.0.2+9-61]
[2022-09-13T00:11:35,003][INFO ][o.e.n.Node               ] [c3df0a5f344a] JVM home [/elasticsearch-8.4.1/jdk], using bundled JDK [true]
[2022-09-13T00:11:35,004][INFO ][o.e.n.Node               ] [c3df0a5f344a] JVM arguments [-Des.networkaddress.cache.ttl=60, -Des.networkaddress.cache.negative.ttl=10, -Djava.security.manager=allow, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -XX:-OmitStackTraceInFastThrow, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -Dlog4j2.formatMsgNoLookups=true, -Djava.locale.providers=SPI,COMPAT, --add-opens=java.base/java.io=ALL-UNNAMED, -XX:+UseG1GC, -Djava.io.tmpdir=/tmp/elasticsearch-472549858283971630, -XX:+HeapDumpOnOutOfMemoryError, -XX:+ExitOnOutOfMemoryError, -XX:HeapDumpPath=data, -XX:ErrorFile=logs/hs_err_pid%p.log, -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m, -Xms2974m, -Xmx2974m, -XX:MaxDirectMemorySize=1559232512, -XX:G1HeapRegionSize=4m, -XX:InitiatingHeapOccupancyPercent=30, -XX:G1ReservePercent=15, -Des.distribution.type=tar, --module-path=/elasticsearch-8.4.1/lib, --add-modules=jdk.net, -Djdk.module.main=org.elasticsearch.server]
[2022-09-13T00:11:36,786][INFO ][c.a.c.i.j.JacksonVersion ] [c3df0a5f344a] Package versions: jackson-annotations=2.13.2, jackson-core=2.13.2, jackson-databind=2.13.2.2, jackson-dataformat-xml=2.13.2, jackson-datatype-jsr310=2.13.2, azure-core=1.27.0, Troubleshooting version conflicts: https://aka.ms/azsdk/java/dependency/troubleshoot
[2022-09-13T00:11:37,961][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [aggs-matrix-stats]
[2022-09-13T00:11:37,961][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [analysis-common]
[2022-09-13T00:11:37,962][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [constant-keyword]
[2022-09-13T00:11:37,962][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [data-streams]
[2022-09-13T00:11:37,966][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [frozen-indices]
[2022-09-13T00:11:37,967][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [ingest-attachment]
[2022-09-13T00:11:37,967][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [ingest-common]
[2022-09-13T00:11:37,971][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [ingest-geoip]
[2022-09-13T00:11:37,972][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [ingest-user-agent]
[2022-09-13T00:11:37,972][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [kibana]
[2022-09-13T00:11:37,972][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [lang-expression]
[2022-09-13T00:11:37,972][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [lang-mustache]
[2022-09-13T00:11:37,973][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [lang-painless]
[2022-09-13T00:11:37,973][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [legacy-geo]
[2022-09-13T00:11:37,973][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [mapper-extras]
[2022-09-13T00:11:37,974][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [mapper-version]
[2022-09-13T00:11:37,974][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [old-lucene-versions]
[2022-09-13T00:11:37,974][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [parent-join]
[2022-09-13T00:11:37,974][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [percolator]
[2022-09-13T00:11:37,974][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [rank-eval]
[2022-09-13T00:11:37,974][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [reindex]
[2022-09-13T00:11:37,975][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [repositories-metering-api]
[2022-09-13T00:11:37,975][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [repository-azure]
[2022-09-13T00:11:37,975][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [repository-encrypted]
[2022-09-13T00:11:37,976][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [repository-gcs]
[2022-09-13T00:11:37,977][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [repository-s3]
[2022-09-13T00:11:37,980][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [repository-url]
[2022-09-13T00:11:37,982][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [runtime-fields-common]
[2022-09-13T00:11:37,984][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [search-business-rules]
[2022-09-13T00:11:37,984][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [searchable-snapshots]
[2022-09-13T00:11:37,989][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [snapshot-based-recoveries]
[2022-09-13T00:11:37,990][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [snapshot-repo-test-kit]
[2022-09-13T00:11:37,992][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [spatial]
[2022-09-13T00:11:37,992][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [transform]
[2022-09-13T00:11:37,992][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [transport-netty4]
[2022-09-13T00:11:37,992][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [unsigned-long]
[2022-09-13T00:11:37,994][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [vector-tile]
[2022-09-13T00:11:37,997][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [wildcard]
[2022-09-13T00:11:37,998][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-aggregate-metric]
[2022-09-13T00:11:37,998][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-analytics]
[2022-09-13T00:11:37,998][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-async]
[2022-09-13T00:11:37,998][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-async-search]
[2022-09-13T00:11:37,998][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-autoscaling]
[2022-09-13T00:11:37,998][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-ccr]
[2022-09-13T00:11:37,998][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-core]
[2022-09-13T00:11:37,999][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-deprecation]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-enrich]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-eql]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-fleet]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-graph]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-identity-provider]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-ilm]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-logstash]
[2022-09-13T00:11:38,000][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-ml]
[2022-09-13T00:11:38,001][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-monitoring]
[2022-09-13T00:11:38,001][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-ql]
[2022-09-13T00:11:38,001][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-rollup]
[2022-09-13T00:11:38,001][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-security]
[2022-09-13T00:11:38,001][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-shutdown]
[2022-09-13T00:11:38,002][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-sql]
[2022-09-13T00:11:38,002][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-stack]
[2022-09-13T00:11:38,002][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-text-structure]
[2022-09-13T00:11:38,002][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-voting-only-node]
[2022-09-13T00:11:38,002][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] loaded module [x-pack-watcher]
[2022-09-13T00:11:38,002][INFO ][o.e.p.PluginsService     ] [c3df0a5f344a] no plugins loaded
[2022-09-13T00:11:40,254][INFO ][o.e.e.NodeEnvironment    ] [c3df0a5f344a] using [1] data paths, mounts [[/ (overlay)]], net usable_space [5.4gb], net total_space [30.8gb], types [overlay]
[2022-09-13T00:11:40,257][INFO ][o.e.e.NodeEnvironment    ] [c3df0a5f344a] heap size [2.9gb], compressed ordinary object pointers [true]
[2022-09-13T00:11:40,267][INFO ][o.e.n.Node               ] [c3df0a5f344a] node name [c3df0a5f344a], node ID [dwhiZizcQIiLBykeyjcdVg], cluster name [netology_test], roles [ingest, data_cold, data, remote_cluster_client, master, data_warm, data_content, transform, data_hot, ml, data_frozen]
[2022-09-13T00:11:43,163][INFO ][o.e.x.s.Security         ] [c3df0a5f344a] Security is enabled
[2022-09-13T00:11:43,465][INFO ][o.e.x.s.a.s.FileRolesStore] [c3df0a5f344a] parsed [0] roles from file [/elasticsearch-8.4.1/config/roles.yml]

[2022-09-13T00:11:43,855][INFO ][o.e.x.m.p.l.CppLogMessageHandler] [c3df0a5f344a] [controller/99] [Main.cc@123] controller (64 bit): Version 8.4.1 (Build c0373714f3bc4b) Copyright (c) 2022 Elasticsearch BV
[2022-09-13T00:11:44,387][INFO ][o.e.t.n.NettyAllocator   ] [c3df0a5f344a] creating NettyAllocator with the following configs: [name=elasticsearch_configured, chunk_size=1mb, suggested_max_allocation_size=1mb, factors={es.unsafe.use_netty_default_chunk_and_page_size=false, g1gc_enabled=true, g1gc_region_size=4mb}]
[2022-09-13T00:11:44,424][INFO ][o.e.i.r.RecoverySettings ] [c3df0a5f344a] using rate limit [40mb] with [default=40mb, read=0b, write=0b, max=0b]
[2022-09-13T00:11:44,469][INFO ][o.e.d.DiscoveryModule    ] [c3df0a5f344a] using discovery type [multi-node] and seed hosts providers [settings]
[2022-09-13T00:11:45,536][INFO ][o.e.n.Node               ] [c3df0a5f344a] initialized
[2022-09-13T00:11:45,538][INFO ][o.e.n.Node               ] [c3df0a5f344a] starting ...
[2022-09-13T00:11:45,554][INFO ][o.e.x.s.c.f.PersistentCache] [c3df0a5f344a] persistent cache index loaded
[2022-09-13T00:11:45,555][INFO ][o.e.x.d.l.DeprecationIndexingComponent] [c3df0a5f344a] deprecation component started
[2022-09-13T00:11:45,645][INFO ][o.e.t.TransportService   ] [c3df0a5f344a] publish_address {172.17.0.2:9300}, bound_addresses {0.0.0.0:9300}
[2022-09-13T00:11:45,745][INFO ][o.e.b.BootstrapChecks    ] [c3df0a5f344a] bound or publishing to a non-loopback address, enforcing bootstrap checks
[2022-09-13T00:11:45,748][INFO ][o.e.c.c.ClusterBootstrapService] [c3df0a5f344a] this node has not joined a bootstrapped cluster yet; [cluster.initial_master_nodes] is set to [c3df0a5f344a]
[2022-09-13T00:11:45,756][INFO ][o.e.c.c.Coordinator      ] [c3df0a5f344a] setting initial configuration to VotingConfiguration{dwhiZizcQIiLBykeyjcdVg}
[2022-09-13T00:11:45,973][INFO ][o.e.c.s.MasterService    ] [c3df0a5f344a] elected-as-master ([1] nodes joined)[_FINISH_ELECTION_, {c3df0a5f344a}{dwhiZizcQIiLBykeyjcdVg}{5hx7J7qwS7GD6KYp30zxRA}{c3df0a5f344a}{172.17.0.2}{172.17.0.2:9300}{cdfhilmrstw} completing election], term: 1, version: 1, delta: master node changed {previous [], current [{c3df0a5f344a}{dwhiZizcQIiLBykeyjcdVg}{5hx7J7qwS7GD6KYp30zxRA}{c3df0a5f344a}{172.17.0.2}{172.17.0.2:9300}{cdfhilmrstw}]}
[2022-09-13T00:11:46,002][INFO ][o.e.c.c.CoordinationState] [c3df0a5f344a] cluster UUID set to [WqGs8h1FSD-uWkUYF9EMnQ]
[2022-09-13T00:11:46,026][INFO ][o.e.c.s.ClusterApplierService] [c3df0a5f344a] master node changed {previous [], current [{c3df0a5f344a}{dwhiZizcQIiLBykeyjcdVg}{5hx7J7qwS7GD6KYp30zxRA}{c3df0a5f344a}{172.17.0.2}{172.17.0.2:9300}{cdfhilmrstw}]}, term: 1, version: 1, reason: Publication{term=1, version=1}
[2022-09-13T00:11:46,046][INFO ][o.e.r.ReadinessService   ] [c3df0a5f344a] readiness service up and running on 127.0.0.1:9399
[2022-09-13T00:11:46,051][INFO ][o.e.r.s.FileSettingsService] [c3df0a5f344a] starting file settings watcher ...
[2022-09-13T00:11:46,062][INFO ][o.e.r.s.FileSettingsService] [c3df0a5f344a] file settings service up and running [tid=50]
[2022-09-13T00:11:46,065][INFO ][o.e.h.AbstractHttpServerTransport] [c3df0a5f344a] publish_address {172.17.0.2:9200}, bound_addresses {0.0.0.0:9200}
[2022-09-13T00:11:46,067][INFO ][o.e.n.Node               ] [c3df0a5f344a] started {c3df0a5f344a}{dwhiZizcQIiLBykeyjcdVg}{5hx7J7qwS7GD6KYp30zxRA}{c3df0a5f344a}{172.17.0.2}{172.17.0.2:9300}{cdfhilmrstw}{xpack.installed=true, ml.allocated_processors=6, ml.max_jvm_size=3120562176, ml.machine_memory=6237782016}
[2022-09-13T00:11:46,121][INFO ][o.e.g.GatewayService     ] [c3df0a5f344a] recovered [0] indices into cluster_state
[2022-09-13T00:11:46,292][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [ilm-history] for index patterns [ilm-history-5*]
[2022-09-13T00:11:46,301][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.slm-history] for index patterns [.slm-history-5*]
[2022-09-13T00:11:46,321][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.watch-history-16] for index patterns [.watcher-history-16*]
[2022-09-13T00:11:46,332][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [.deprecation-indexing-mappings]
[2022-09-13T00:11:46,336][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [.deprecation-indexing-settings]
[2022-09-13T00:11:46,339][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [metrics-mappings]
[2022-09-13T00:11:46,343][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [logs-settings]
[2022-09-13T00:11:46,348][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [synthetics-mappings]
[2022-09-13T00:11:46,350][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [synthetics-settings]
[2022-09-13T00:11:46,357][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [data-streams-mappings]
[2022-09-13T00:11:46,361][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [logs-mappings]
[2022-09-13T00:11:46,364][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding component template [metrics-settings]
[2022-09-13T00:11:46,376][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.monitoring-ent-search-mb] for index patterns [.monitoring-ent-search-8-*]
[2022-09-13T00:11:46,407][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.monitoring-es-mb] for index patterns [.monitoring-es-8-*]
[2022-09-13T00:11:46,416][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding template [.monitoring-alerts-7] for index patterns [.monitoring-alerts-7]
[2022-09-13T00:11:46,424][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding template [.monitoring-logstash] for index patterns [.monitoring-logstash-7-*]
[2022-09-13T00:11:46,429][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding template [.monitoring-kibana] for index patterns [.monitoring-kibana-7-*]
[2022-09-13T00:11:46,439][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding template [.monitoring-es] for index patterns [.monitoring-es-7-*]
[2022-09-13T00:11:46,445][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding template [.monitoring-beats] for index patterns [.monitoring-beats-7-*]
[2022-09-13T00:11:46,457][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.monitoring-logstash-mb] for index patterns [.monitoring-logstash-8-*]
[2022-09-13T00:11:46,474][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.monitoring-beats-mb] for index patterns [.monitoring-beats-8-*]
[2022-09-13T00:11:46,491][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.monitoring-kibana-mb] for index patterns [.monitoring-kibana-8-*]
[2022-09-13T00:11:46,498][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.ml-stats] for index patterns [.ml-stats-*]
[2022-09-13T00:11:46,512][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.ml-anomalies-] for index patterns [.ml-anomalies-*]
[2022-09-13T00:11:46,517][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.ml-notifications-000002] for index patterns [.ml-notifications-000002]
[2022-09-13T00:11:46,520][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.ml-state] for index patterns [.ml-state*]
[2022-09-13T00:11:46,578][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [.deprecation-indexing-template] for index patterns [.logs-deprecation.*]
[2022-09-13T00:11:46,582][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [synthetics] for index patterns [synthetics-*-*]
[2022-09-13T00:11:46,585][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [logs] for index patterns [logs-*-*]
[2022-09-13T00:11:46,588][INFO ][o.e.c.m.MetadataIndexTemplateService] [c3df0a5f344a] adding index template [metrics] for index patterns [metrics-*-*]
[2022-09-13T00:11:46,621][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [watch-history-ilm-policy-16]
[2022-09-13T00:11:46,667][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [ilm-history-ilm-policy]
[2022-09-13T00:11:46,735][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [.deprecation-indexing-ilm-policy]
[2022-09-13T00:11:46,779][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [slm-history-ilm-policy]
[2022-09-13T00:11:46,813][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [synthetics]
[2022-09-13T00:11:46,840][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [logs]
[2022-09-13T00:11:46,865][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [metrics]
[2022-09-13T00:11:46,889][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [30-days-default]
[2022-09-13T00:11:46,919][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [180-days-default]
[2022-09-13T00:11:46,948][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [365-days-default]
[2022-09-13T00:11:46,972][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [90-days-default]
[2022-09-13T00:11:47,000][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [7-days-default]
[2022-09-13T00:11:47,024][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [.monitoring-8-ilm-policy]
[2022-09-13T00:11:47,043][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [ml-size-based-ilm-policy]
[2022-09-13T00:11:47,064][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [c3df0a5f344a] adding index lifecycle policy [.fleet-actions-results-ilm-policy]
[2022-09-13T00:11:47,151][INFO ][o.e.l.LicenseService     ] [c3df0a5f344a] license [309bff94-39ce-405d-be8c-7ba5a5bc192e] mode [basic] - valid
[2022-09-13T00:11:47,153][INFO ][o.e.x.s.a.Realms         ] [c3df0a5f344a] license mode is [basic], currently licensed security realms are [reserved/reserved,file/default_file,native/default_native]
[2022-09-13T00:11:48,052][INFO ][o.e.c.m.MetadataCreateIndexService] [c3df0a5f344a] [.geoip_databases] creating index, cause [auto(bulk api)], templates [], shards [1]/[0]
[2022-09-13T00:11:48,198][INFO ][o.e.c.r.a.AllocationService] [c3df0a5f344a] current.health="GREEN" message="Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[.geoip_databases][0]]])." previous.health="YELLOW" reason="shards started [[.geoip_databases][0]]"
[2022-09-13T00:11:48,593][INFO ][o.e.i.g.GeoIpDownloader  ] [c3df0a5f344a] successfully downloaded geoip database [GeoLite2-ASN.mmdb]
[2022-09-13T00:11:48,724][INFO ][o.e.i.g.DatabaseNodeService] [c3df0a5f344a] successfully loaded geoip database file [GeoLite2-ASN.mmdb]
[2022-09-13T00:11:51,731][INFO ][o.e.i.g.GeoIpDownloader  ] [c3df0a5f344a] successfully downloaded geoip database [GeoLite2-City.mmdb]
[2022-09-13T00:11:52,352][INFO ][o.e.i.g.GeoIpDownloader  ] [c3df0a5f344a] successfully downloaded geoip database [GeoLite2-Country.mmdb]
[2022-09-13T00:11:52,407][INFO ][o.e.i.g.DatabaseNodeService] [c3df0a5f344a] successfully loaded geoip database file [GeoLite2-Country.mmdb]
[2022-09-13T00:11:52,462][INFO ][o.e.i.g.DatabaseNodeService] [c3df0a5f344a] successfully loaded geoip database file [GeoLite2-City.mmdb]
[2022-09-13T00:11:55,170][INFO ][o.e.x.s.InitialNodeSecurityAutoConfiguration] [c3df0a5f344a] HTTPS has been configured with automatically generated certificates, and the CA's hex-encoded SHA-256 fingerprint is [019133568d2746ea9bbd6701121038e2e93dfeac0a1c7cc3b38c1f30b003b535]
[2022-09-13T00:11:55,172][INFO ][o.e.x.s.s.SecurityIndexManager] [c3df0a5f344a] security index does not exist, creating [.security-7] with alias [.security]
[2022-09-13T00:11:55,245][INFO ][o.e.c.m.MetadataCreateIndexService] [c3df0a5f344a] [.security-7] creating index, cause [api], templates [], shards [1]/[0]
[2022-09-13T00:11:55,281][INFO ][o.e.x.s.s.SecurityIndexManager] [c3df0a5f344a] security index does not exist, creating [.security-7] with alias [.security]
[2022-09-13T00:11:55,327][INFO ][o.e.c.r.a.AllocationService] [c3df0a5f344a] current.health="GREEN" message="Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[.security-7][0]]])." previous.health="YELLOW" reason="shards started [[.security-7][0]]"
```

- выполним запрос
```
vagrant@vagrant:/etc/ssl/certs$ curl -X GET 'http://localhost:9200'
{
  "name" : "45156ea03fb0",
  "cluster_name" : "netology_test",
  "cluster_uuid" : "d3FDke3xQlORwiQj7B3U0Q",
  "version" : {
    "number" : "8.4.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "2bd229c8e56650b42e40992322a76e7914258f0c",
    "build_date" : "2022-08-26T12:11:43.232597118Z",
    "build_snapshot" : false,
    "lucene_version" : "9.3.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
- Итоговый Dockerfile 
```
FROM    centos:7

RUN mkdir /var/lib/elasticsearch
RUN mkdir /var/log/elasticsearch

RUN yum -y install java-1.8.0-openjdk.x86_64
RUN yum -y install wget

COPY elasticsearch-8.4.1-linux-x86_64.tar.gz ./

RUN tar -xzf elasticsearch-8.4.1-linux-x86_64.tar.gz
RUN  cd elasticsearch-8.4.1

RUN groupadd -g 1000 elasticsearch && useradd elasticsearch -u 1000 -g 1000

WORKDIR /elasticsearch-8.4.1

RUN cd /elasticsearch-8.4.1

RUN set -ex && for path in data logs config config/scripts bin jdk; do \
        mkdir -p "$path"; \
        chown -R elasticsearch:elasticsearch "$path"; \
    done

RUN  chown -R elasticsearch:elasticsearch /elasticsearch-8.4.1/jdk/bin/
RUN  chown -R elasticsearch:elasticsearch /elasticsearch-8.4.1/jdk/bin/java
RUN  chown -R elasticsearch:elasticsearch /var/lib/elasticsearch
RUN  chown -R elasticsearch:elasticsearch /var/log/elasticsearch

USER elasticsearch

COPY logging.yml /elasticsearch-8.4.1/config/
COPY elasticsearch.yml /elasticsearch-8.4.1/config/

ENV PATH=$PATH:/elasticsearch-8.4.1/bin

CMD ["elasticsearch"]

EXPOSE 9200 9300
```

- Итоговый файл elasticsearch.yml
```
cluster.name: netology_test
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
http.host: 0.0.0.0
```

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

Решение:

- создадим индексы
```
vagrant@vagrant:/etc/ssl/certs$ curl -X PUT "localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 1,
>       "number_of_replicas": 0
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-1"
}

vagrant@vagrant:/etc/ssl/certs$ curl -X PUT "localhost:9200/ind-2?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 2,
>       "number_of_replicas": 1
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-2"
}

vagrant@vagrant:/etc/ssl/certs$ curl -X PUT "localhost:9200/ind-3?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 4,
>       "number_of_replicas": 2
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "ind-3"
}

curl -X GET "localhost:9200/my-index-000001/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
'
```
- проверим состояние индексов
```
vagrant@vagrant:/etc/ssl/certs$ curl -X GET "localhost:9200/_cat/indices"
green  open ind-1 lASeIWn_R5WulDAvZF-Xqg 1 0 0 0 225b 225b
yellow open ind-3 MyCi5CK7QnSQ_wXInluEYA 4 2 0 0 900b 900b
yellow open ind-2 WvN7s8oCSEeACGuvJqH25w 2 1 0 0 450b 450b
```
- проверим состояние кластера
```
vagrant@vagrant:/etc/ssl/certs$ curl -X GET "localhost:9200/_cluster/health"
{"cluster_name":"netology_test","status":"yellow","timed_out":false,"number_of_nodes":1,"number_of_data_nodes":1,"active_primary_shards":8,"active_shards":8,"relocating_shards":0,"initializing_shards":0,"unassigned_shards":10,"delayed_unassigned_shards":0,"number_of_pending_tasks":0,"number_of_in_flight_fetch":0,"task_max_waiting_in_queue_millis":0,"active_shards_percent_as_number":44.44444444444444}
```

Часть индексов и кластер находятся в в состоянии yellow, т.к. все primary шарды в состоянии assigned. Часть secondary шард в состоянии unassigned;
Что и показывает запрос ниже:
```
:/etc/ssl/certs$ curl -X GET "localhost:9200/_cat/shards"
ind-3            0 p STARTED     0 225b 172.17.0.2 45156ea03fb0
ind-3            0 r UNASSIGNED
ind-3            0 r UNASSIGNED
ind-3            1 p STARTED     0 225b 172.17.0.2 45156ea03fb0
ind-3            1 r UNASSIGNED
ind-3            1 r UNASSIGNED
ind-3            2 p STARTED     0 225b 172.17.0.2 45156ea03fb0
ind-3            2 r UNASSIGNED
ind-3            2 r UNASSIGNED
ind-3            3 p STARTED     0 225b 172.17.0.2 45156ea03fb0
ind-3            3 r UNASSIGNED
ind-3            3 r UNASSIGNED
.geoip_databases 0 p STARTED    41 39mb 172.17.0.2 45156ea03fb0
ind-1            0 p STARTED     0 225b 172.17.0.2 45156ea03fb0
ind-2            0 p STARTED     0 225b 172.17.0.2 45156ea03fb0
ind-2            0 r UNASSIGNED
ind-2            1 p STARTED     0 225b 172.17.0.2 45156ea03fb0
ind-2            1 r UNASSIGNED
```
## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.


Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

---

Решение:

- изменим Dockerfile и elasticsearch.yml 
Dockerfile:
RUN chown -R elasticsearch:elasticsearch  /elasticsearch-8.4.1/snapshots
RUN mkdir /elasticsearch-8.4.1/snapshots

elasticsearch.yml:
path.repo: /elasticsearch-8.4.1/snapshots

- пересоберем образ и перезапустим контейнер

- зарегистрируем новый репозитарий с помощью REST API
```
vagrant@vagrant:~/elastic-search$ curl -X PUT "localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
{
   "type": "fs",
   "settings": {
     "location": "netology_backup"
   }
 }
 '
{
  "acknowledged" : true
}
```

- создадим индекс
```
vagrant@vagrant:~/elastic-search$ curl -X PUT "localhost:9200/test?pretty" -H 'Content-Type: application/json' -d'
>  {
>    "settings": {
>      "index": {
>        "number_of_shards": 1,
>        "number_of_replicas": 0
>      }
>    }
>  }
>  '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test"
}
```

- сделаем snapshot:
```
vagrant@vagrant:~/elastic-search$ curl -X PUT "localhost:9200/_snapshot/netology_backup/first_snapshot?wait_for_completion=true&pretty" -H 'Content-Type: application/json' -d'
> {
>   "metadata": {
>     "taken_by": "kaplin",
>     "taken_because": "backup before deleting index test"
>   }
> }
> '
{
  "snapshot" : {
    "snapshot" : "first_snapshot",
    "uuid" : "IM9r5BtqSqyFQLd5EQKiGA",
    "repository" : "netology_backup",
    "version_id" : 8040199,
    "version" : "8.4.1",
    "indices" : [
      "test",
      ".geoip_databases"
    ],
    "data_streams" : [ ],
    "include_global_state" : true,
    "metadata" : {
      "taken_by" : "kaplin",
      "taken_because" : "backup before deleting index test"
    },
    "state" : "SUCCESS",
    "start_time" : "2022-09-13T06:32:42.363Z",
    "start_time_in_millis" : 1663050762363,
    "end_time" : "2022-09-13T06:32:43.592Z",
    "end_time_in_millis" : 1663050763592,
    "duration_in_millis" : 1229,
    "failures" : [ ],
    "shards" : {
      "total" : 2,
      "failed" : 0,
      "successful" : 2
    },
    "feature_states" : [
      {
        "feature_name" : "geoip",
        "indices" : [
          ".geoip_databases"
        ]
      }
    ]
  }
}
``` 
- зайдем в контейнер и посмотрим на файловое хранилище:
```
[elasticsearch@66cbead6f651 netology_backup]$ vagrant@vagrant:~/elastic-search$ sudo docker ps
CONTAINER ID   IMAGE         COMMAND           CREATED         STATUS         PORTS                                                                                            NAMES
4798e5a2f4be   my-es-image   "elasticsearch"   4 minutes ago   Up 4 minutes   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp, 9399/tcp   strange_wiles
vagrant@vagrant:~/elastic-search$ sudo docker exec -it 4798e5a2f4be /bin/bash
[elasticsearch@4798e5a2f4be elasticsearch-8.4.1]$ cd snapshots/
[elasticsearch@4798e5a2f4be snapshots]$ ls -al
total 16
drwxr-xr-x 1 elasticsearch elasticsearch 4096 Sep 13 06:30 .
drwxr-xr-x 1 root          root          4096 Sep 13 06:05 ..
drwxr-xr-x 3 elasticsearch elasticsearch 4096 Sep 13 06:32 netology_backup
[elasticsearch@4798e5a2f4be snapshots]$ cd netology_backup/
[elasticsearch@4798e5a2f4be netology_backup]$ ls -al
total 44
drwxr-xr-x 3 elasticsearch elasticsearch  4096 Sep 13 06:32 .
drwxr-xr-x 1 elasticsearch elasticsearch  4096 Sep 13 06:30 ..
-rw-r--r-- 1 elasticsearch elasticsearch   847 Sep 13 06:32 index-0
-rw-r--r-- 1 elasticsearch elasticsearch     8 Sep 13 06:32 index.latest
drwxr-xr-x 4 elasticsearch elasticsearch  4096 Sep 13 06:32 indices
-rw-r--r-- 1 elasticsearch elasticsearch 18588 Sep 13 06:32 meta-IM9r5BtqSqyFQLd5EQKiGA.dat
-rw-r--r-- 1 elasticsearch elasticsearch   400 Sep 13 06:32 snap-IM9r5BtqSqyFQLd5EQKiGA.dat
```

- удалил индекс test и создадим индекс test- 2
```
vagrant@vagrant:~/elastic-search$ curl -X DELETE "localhost:9200/test?pretty"
{
  "acknowledged" : true
}

curl -X PUT "localhost:9200/test-2?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index": {
      "number_of_shards": 1,  
      "number_of_replicas": 0 
    }
  }
}
'

vagrant@vagrant:~/elastic-search$ curl -X PUT "localhost:9200/test-2?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "index": {
>       "number_of_shards": 1,
>       "number_of_replicas": 0
>     }
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
vagrant@vagrant:~/elastic-search$ curl -X GET "localhost:9200/_cat/indices"
green open test-2 GOCx-t9AQeyxrkqSnswcmw 1 0 0 0 225b 225b
```
- восстановим snapshot
Доступные snapshot:
```
vagrant@vagrant:~/elastic-search$ curl -X GET "localhost:9200/_snapshot?pretty"
{
  "netology_backup" : {
    "type" : "fs",
    "uuid" : "mGGAKNpHTe-xvb3zym5yzA",
    "settings" : {
      "location" : "netology_backup"
    }
  }
}
```
- восстановим и проверим индексы
```
vagrant@vagrant:~/elastic-search$ curl -X POST "localhost:9200/_snapshot/netology_backup/first_snapshot/_restore?pretty"
{
  "accepted" : true
}
vagrant@vagrant:~/elastic-search$ curl -X GET "localhost:9200/_cat/indices"
green open test-2 GOCx-t9AQeyxrkqSnswcmw 1 0 0 0 225b 225b
green open test   M603afDvRxK5J2KojFmtNw 1 0 0 0 225b 225b
```
### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
