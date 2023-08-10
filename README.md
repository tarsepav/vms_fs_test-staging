### OS Ubuntu 18.04


### Приложения:

* git
* postgresql-13
* sqitch
* temporal_tables
* is_jsonb_valid
* kerl
* erlang 21.3.3
* rebar 3.9.1

### Доступ:

* ((https://gitlab.sifox.ru https://gitlab.sifox.ru))
* vpn ((http://192.168.11.202 http://192.168.11.202))

### Зависимости:
```bash
sudo apt-get update
sudo apt-get install -y git unzip subversion build-essential autoconf automake libtool libncurses5 libncurses5-dev make libjpeg-dev libtool libtool-bin libsqlite3-dev libpcre3-dev libspeexdsp-dev libldns-dev libedit-dev yasm liblua5.2-dev libopus-dev cmake xclip libcurl4-openssl-dev libexpat1-dev libgnutls28-dev libtiff5-dev libx11-dev unixodbc-dev libssl-dev python-dev zlib1g-dev libasound2-dev libogg-dev libvorbis-dev libperl-dev libgdbm-dev libdb-dev uuid-dev libsndfile1-dev # не хватает sqitch
```

### Git:
```bash

sudo apt-get install git
```


* Формируем ssh ключ (((https://gitlab.sifox.ru/help/ssh/README#generate-an-ssh-key-pair) https://gitlab.sifox.ru/help/ssh/README#generate-an-ssh-key-pair)))

```
ssh-keygen -t ed25519
```

(на каждом этаме жмем enter)


* копируем содержимое публичного ключа с помощью 

```
xclip -sel clip < ~/.ssh/id_ed25519.pub
```


* вставляем данные в https://gitlab.sifox.ru/-/profile/keys -&gt; SSH Keys
* создаем ~/.ssh/config и заполняем:

```
Host gitlab.sifox.ru
Port 10622
User <username@sifox.ru>
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_ed25519
```

* создаем ~/.gitconfig и заполняем:

>[user]  email = ((mailto:pavel.abramov@sifox.com pavel.abramov@sifox.com))  name = ((mailto:pavel.abramov@sifox.com pavel.abramov@sifox.com))
>
>[url "ssh://git@gitlab.sifox.ru:10622/"]  insteadOf = "((https://gitlab.sifox.ru/%22 https://gitlab.sifox.ru/"))
### БД:
#### PostgreSQL:

* устанавливаем (((https://www.postgresql.org/download/linux/ubuntu/): https://www.postgresql.org/download/linux/ubuntu/):))

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql-13
```


* создаем роль пользователя:

```
sudo -u postgres psql << EOF
CREATE ROLE develop WITH LOGIN PASSWORD '123' CREATEDB CREATEROLE SUPERUSER; #вместо develop твой username
EOF
createdb
```


* создаем правило /home/develop/.pgpass и заполняем: #вместо develop твой username
```
*:*:*:develop:123 #вместо develop твой username
```
(sqitch deploy проходит из под юзера)

```
sudo chmod 600 /home/develop/.pgpass
```


* подключаемся к базе:

```
psql
  CREATE DATABASE vms_test_db WITH OWNER vms;
  \c vms_test_db
  show wal_level; (посмотреть какой левел) изменить если не logical: #изменения вступят в силу после рестарта сервиса
     ALTER SYSTEM SET wal_level = logical;
  \q
sudo pg_ctlcluster 13 main restart #рестарт psql
psql #не хватало здесь комманды
sqitch deploy db:pg://localhost:5432/vms_test_db (Если появится сообщение с фразой "no password supplied", нужно привести строку запуска к виду
sqitch deploy db:pg://username:123@localhost:5432/vms_test_db, где username - имя пользователя)
sqitch verify db:pg://localhost:5432/vms_test_db

#psql #а здесь лишняя
  \c vms_test_db
  \dt vms.* (посмотреть создались ли таблицы)

psql -h 127.0.0.1 -p 5432 template1 < ./test/config_logical_replication.sql
psql -h 127.0.0.1 -p 5432 template1 < ./test/test_data.sql
```

#### db-vms:
```bash
git clone https://gitlab.sifox.ru/sf-vms/db-vms
```


* в readme есть некоторая информация после установки postgresql
* Создать пользователя vms и vms databse:

```bash
sudo -u postgres psql << EOF
CREATE ROLE vms WITH LOGIN PASSWORD 'vms_2018' CREATEDB CREATEROLE SUPERUSER;
CREATE ROLE provision WITH LOGIN PASSWORD 'provision';
CREATE DATABASE vms WITH OWNER vms;
EOF
```

#### sqitch:

* установка (((http://sqitch.org/download/debian/): http://sqitch.org/download/debian/):))

```bash
sudo apt-get install libdbd-pg-perl postgresql-client-13 postgresql-server-dev-13 sqitch
```

#### tmp_extensions:
```
mkdir tmp_extensions
cd tmp_extensions
```

#### temporal_tables:
```bash
git clone https://github.com/arkhipov/temporal_tables.git
cd temporal_tables
make PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config
sudo make install PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config
cd ..
```

#### is_jsonb_valid:
```bash
git clone https://github.com/furstenheim/is_jsonb_valid
cd is_jsonb_valid
sudo make install
make installcheck
cd ..
```

```bash
rm -fr tmp_extensions
cd db-vms
```

### Erlang (21.3.3):
#### kerl:
```bash
mkdir -p ~/bin
cd ~/bin
curl -O https://raw.githubusercontent.com/kerl/kerl/master/kerl
chmod a+x ./kerl
./kerl update releases
./kerl build 21.3.3 21.3.3
./kerl install 21.3.3 ~/kerl/21.3.3
. ~/kerl/21.3.3/activate
```

<#<table class="wikisnippet_alert" style="border-left:4px solid #aaa; margin:20px; width:90%; background-color:#f1f1f1;">
<tr valign="middle" style="vertical-align: middle;"><td style="vertical-align: middle; padding:20px; 20px 0 0px; text-align:left;">#>
Запускаем всегда когда хотим работать с нодой. Инстанс привязан к терминалу!


<#</td></tr></table>#>
kerl_deactivate - что бы выключить

#### rebar (3.9.1):
```
git clone --branch 3.9.1 https://github.com/erlang/rebar3.git
cd rebar3
./bootstrap 
./rebar3 local install
sudo cp rebar3 /usr/bin/rebar3
cd ..
```

### vms_fs:
```bash
git clone ssh://git@gitlab.sifox.ru:10622/sf-vms/vms_fs.git
cd vms_fs
```


* меняем настройки подключения к бд в vms_fs/config/sys.config и в vms_fs/sys.config:

```
{epgl_port,5432},
{epgl_host,"localhost"},
{epgl_password,"vms_2018"},
{epgl_username,"vms"},
{epgl_database,"vms_test_db"},
```

* и во втором месте:
```
{hostname,"localhost"},
{port,5432},
{database,"vms_test_db"},
{username,"vms"},
{password,"vms_2018"},
{app_name,"vms_fs"}
```

можно посмотреть на следующие настройки:
`kz_profile_id` - поменять на 1, т.к. такого профайла в базе может не быть
`goodok_check_silence` - поменять на false
`{record_storage, <<"swfs_and_s3">>}` - закомментировать строку

```bash
DEBUG=1 rebar3 release
./_build/default/rel/vms_fs/bin/vms_fs console
```

### Freeswitch:
```bash
git clone ssh://git@gitlab.sifox.ru:10622/common/voip/freeswitch_src.git
cd freeswitch_src
./bootstrap.sh -j
```


* модифицируем файл modules.conf, оставляем только:

```
applications/mod_commands
applications/mod_dptools
applications/mod_http_cache
codecs/mod_amr
codecs/mod_amrwb
dialplans/mod_dialplan_xml
endpoints/mod_sofia
endpoints/mod_loopback
event_handlers/mod_event_socket
event_handlers/mod_kazoo
formats/mod_native_file
formats/mod_sndfile
formats/mod_tone_stream
loggers/mod_console
loggers/mod_logfile
say/mod_say_ru
```
**под kerl не ставить fs_epmd и не устанавливается mod_kazoo**


* kerl_deactivate
* sudo apt-get install erlang

```bash
./configure 
sudo make (возможно еще попросит установить spandsp и sofia-mod, перед выполнением sudo make еще может потребоваться sudo apt install yasm)
sudo make install
cd /usr/local
sudo groupadd freeswitch
sudo adduser --quiet --system --home /usr/local/freeswitch --gecos "FreeSWITCH open source softswitch" --ingroup freeswitch freeswitch --disabled-password
sudo chown -R freeswitch:freeswitch /usr/local/freeswitch/
sudo chmod -R ug=rwX,o= /usr/local/freeswitch/
sudo chmod -R u=rwx,g=rx /usr/local/freeswitch/bin/
sudo adduser develop freeswitch
sudo ln -s /usr/local/freeswitch/bin/freeswitch /usr/bin/
sudo ln -s /usr/local/freeswitch/bin/fs_cli /usr/bin
```

* перезапускаем пользовательскую сессию, чтобы применились настройки группы (например, с помощью команды  gnome-session-quit, а далее повторно залогиниться)
* модифицируем файл /usr/local/freeswitch/conf/autoload_configs/modules.conf.xml, оставляем только:

```
<load module="mod_logfile"/>
  <load module="mod_event_socket"/>
  <load module="mod_kazoo"/>
  <load module="mod_loopback"/>
  <load module="mod_commands"/>
  <load module="mod_dptools"/>
  <load module="mod_http_cache"/>
  <load module="mod_dialplan_xml"/>
  <load module="mod_amr"/>
  <load module="mod_amrwb"/>
  <load module="mod_sndfile"/>
  <load module="mod_native_file"/>
  <load module="mod_tone_stream"/>
  <load module="mod_say_ru"/> 
  <load module="sofia"/>
```
**После первого запуска Freeswitch, когда соберется модуль sofia, необходимо в файле**

`/usr/local/freeswitch/conf/autoload_configs/modules.conf.xml` закомментировать данный модуль, т.е. `<!--load module="sofia"--/>`

* модифицируем файл /usr/local/freeswitch/conf/vars.xml:
```
<X-PRE-PROCESS cmd="set" data="internal_sip_port=5060"/>
<X-PRE-PROCESS cmd="set" data="local_sip_ip=-ip-"/>
<X-PRE-PROCESS cmd="set" data="local_rtp_ip=-ip-"/>
<X-PRE-PROCESS cmd="set" data="send_silence_when_idle=400"/>
<X-PRE-PROCESS cmd="set" data="sound_prefix=/usr/local/freeswitch/sounds/ru/RU/elena"/>
<X-PRE-PROCESS cmd="set" data="global_codec_prefs=PCMU,PCMA,AMR,AMR-WB"/>
```
где -ip- - твой ip

```bash
cd ~/src/
git clone ssh://git@gitlab.sifox.ru:10622/sf-vms/mf-vms-ansible.git
cd /usr/local/freeswitch/conf/autoload_configs
sudo cp -a /home/develop/src/mf-vms-ansible/roles/freeswitch/templates/freeswitch/autoload_configs/. .
```


* модифицируем файл acl.conf.xml:
```
<list name="spx" default="allow">
     <node type="allow" cidr="-ip-/24"/>
          <node type="allow" cidr="-ip-/32"/>
</list>
```
где -ip- - твой ip c которого звонить будешь

или на внешний адрес vpn

* модифицируем файл kazoo.conf.xml:
```
<param name="listen-ip" value="127.0.0.1" />
<param name="listen-port" value="8031" />
<param name="cookie" value="fswitch" />
<param name="shortname" value="false" />
<param name="nodename" value="freeswitch@127.0.0.1" />
```

```bash
cd ../sip_profiles/
```


* удаляем все папки и файлы в текущей директории и копируем туда актуальный файл (##vms_spx.xml##) из ##/home/develop/src/mf-vms-ansible/roles/freeswitch/templates/freeswitch/sip_profiles/##

```bash
rm -rf *
sudo cp /home/develop/src/mf-vms-ansible/roles/freeswitch/templates/freeswitch/sip_profiles/vms_spx.xml vms_spx.xml
```

```bash
cd ../lang/ru
sudo cp -a /home/develop/src/mf-vms-ansible/roles/freeswitch/templates/freeswitch/lang/ru/. .
```


* модифицируем файл ru.xml:

>sound-prefix="/usr/local/freeswitch/sounds/ru/RU/elena"

* забираем звуковые файлы из http://192.168.11.202/distr/project/vms/: http://192.168.11.202/distr/project/vms/freeswitch-sounds-ru-RU-elena-8000-1.0.51.tar.gz
http://192.168.11.202/distr/project/vms/mf_sounds_20191205.tar.gz

```bash
cd /usr/local/freeswitch/sounds
sudo tar -xvzf freeswitch-sounds-ru-RU-elena-8000-1.0.51.tar.gz
cd /usr/local/freeswitch/sounds/ru/RU/elena
sudo tar -xvzf mf_sounds_20191205.tar.gz 
```

### Запуск:

* vms_fs:

```bash
cd
. ~/kerl/21.3.3/activate
./_build/default/rel/vms_fs/bin/vms_fs console
```


* freeswitch:

```bash
. ~/kerl/21.3.3/activate
freeswitch
```

Успешный вывод на ноде:

```bash
FSNode freeswitch@127.0.0.1: try connect_to_fs
FSNode freeswitch@127.0.0.1: connected
FSNode freeswitch@127.0.0.1: binded to dialplan
FSNode freeswitch@127.0.0.1: binded to events  ['CHANNEL_PARK','CHANNEL_CREATE','CHANNEL_DESTROY','CHANNEL_EXECUTE_COMPLETE','HEARTBEAT']
trying to load mod_sofia
mod_sofia loaded, ready for traffic, res:<<"+OK Reloading XML\n+OK\n">>
```

Ставим софтфон:


* например zoiper (((https://www.zoiper.com/en/documentation/linux-installation-and-configuration)) https://www.zoiper.com/en/documentation/linux-installation-and-configuration))))

>login: testpwd: 1234domain: -ip-:5060settings -&gt; accounts -&gt; test -&gt; advanced -&gt; register on startup = falseзвонок на 2222 : запись сообщениязвонок на 222 : прослушать записанное сообщение
где -ip- - твой ip
