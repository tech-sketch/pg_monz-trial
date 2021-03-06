# pg_monzの試行環境を構築する
pg_monzをローカルPC上で試行する環境を構築する手順の説明です。

# 構成図
下図の様な環境をホストPCに構築します。
ホストPC（筆者環境はMacOS10.10.3）上のVirtualBoxにCentOSのVMを6台起動します。

![構成図.png](https://qiita-image-store.s3.amazonaws.com/0/13485/e9a0b838-b3a5-8bee-4b1a-76662968aa8b.png "構成図.png")

# サーバの役割
| サーバ名  | IPアドレス     | 役割 |
|:---------|-------------:|:----|
| zabbix   | 192.168.1.20 | Zabbix監視サーバ |
| pgpool01 | 192.168.1.11 | pgpool-II稼働サーバ。Active用の仮想IP割当て済 |
| pqpool02 | 192.168.1.12 | pgpool-II稼働サーバ(Standby) |
| pgsql01  | 192.168.1.13 | PostgreSQL稼働サーバ。SRのprimary |
| pgsql02  | 192.168.1.14 | PostgreSQL稼働サーバ。SRの同期Standby |
| pgsql03  | 192.168.1.15 | PostgreSQL稼働サーバ。SRの非同期Standby |

# 稼働環境
使用しているプロダクトと筆者環境で動作確認しているバージョンは以下のとおりです。
## ホストPC
| プロダクト名 | バージョン |
|:-----------|---------:|
| VirtualBox |   4.3.26 |
| Vagrant    |    1.7.2 |
| Ansible    |    1.9.1 |
## ゲストOS
| プロダクト名 | バージョン |
|:-----------|---------:|
| CentOS     |      6.7 |
| PostgreSQL |    9.4.4 |
| Zabbix     |    2.4.6 |

# 構築手順
## pgpool-II,PostgreSQL SRクラスタの構築
* ホストPCにVirtualBox,Vagrant,Ansibleをインストール（詳細省略）
* 任意のディレクトリにこのリポジトリをクローン

```bash
$ git clone https://github.com/tech-sketch/pg_monz-trial.git
$ cd pg_monz-trial
```

* VM を起動

```bash
$ vagrant up
$ vagrant status
Current machine states:

pgpool01                  running (virtualbox)
pgpool02                  running (virtualbox)
pgsql01                   running (virtualbox)
pgsql02                   running (virtualbox)
pgsql03                   running (virtualbox)
zabbix                    running (virtualbox)
```

* pgcluster_hosts,zabbix_hostsを編集

```Python
ansible_ssh_host=（それぞれのホストのIPアドレス）
ansible_ssh_private_key_file=（pg_monz-trialディレクトリ)/.vagrant.d/machines/(ホスト名)/virtualbox/private_key）
```

* 使用するAnsible Playbookを外部サイトからダウンロード

```bash
$ git clone https://github.com/pg-monz/ansible-pgool-pgsql-cluster.git
$ ansible-galaxy install -p ./zabbix-setting/roles patrik.uytterhoeven.PostgreSQL-For-RHEL6x
$ ansible-galaxy install -p ./zabbix-setting/roles dj-wasabi.zabbix-server
$ ansible-galaxy install -p ./zabbix-setting/roles dj-wasabi.zabbix-agent
```

* group_vars/all.ymlを編集

```Python
# 編集箇所
synchronous_standby_names: '192.168.1.14,192.168.1.15'
vip: 192.168.1.100
pgpool_active_ip: 192.168.1.11
pgpool_standby_ip: 192.168.1.12
pgsql_primary_ip: 192.168.1.13
pgsql_standby01_ip: 192.168.1.14
pgsql_standby02_ip: 192.168.1.15
```

* あらかじめ必要なパッケージをインストールするPlaybookを実行

```bash
$ ansible-playbook --ask-pass -i pgcluster_hosts prepare-setting/main.yml
```

* pgpool-II,PostgreSQL SRクラスタを構築するPlaybookを実行

```bash
$ ansible-playbook --ask-pass -i pgcluster_hosts ansible-pgool-pgsql-cluster/site.yml
```

## Zabbixサーバの構築

* あらかじめ必要なパッケージをインストールするPlaybookを実行

```bash
$ ansible-playbook --ask-pass -i zabbix_hosts prepare-setting/main.yml
```

* Zabbix ServerをインストールするPlaybookを実行

```bash
$ ansible-playbook --ask-pass -i zabbix_hosts zabbix-setting/server.yml
```

## Zabbix エージェントの導入

* ZabbiｘエージェントをインストールするPlaybookを実行

```bash
ansible-playbook --ask-pass -i pgcluster_hosts zabbix-setting/agent.yml
```

## pg_monzの導入

* pg_monzをダウンロードする。

```bash
$ git clone https://github.com/pg-monz/pg_monz.git
```

* pg_monz 設定ファイル、スクリプトを配置するPlaybookを実行

```bash
$ ansible-playbook --ask-pass -i pgcluster_hosts pgmonz_deploy/main.yml
```

* Zabbix Webインタフェース http://192.168.1.20 にアクセスする。

(ここからZabbix Webインタフェースでの作業)

* pg_monzのtemplateフォルダにあるテンプレート(.xml)をZabbix インポートする。

* テンプレートTemplate App PostgreSQLのマクロを編集する。

> {$PGLOGDIR} : /var/log/pgsql

* ホストを作成し、テンプレートを割り当てる。

| ホスト名 | テンプレート名 |
|:-----------|:---------|
| pgpool01,pgpool02 | Template App pgpool-II |
| pgsql01,pgsql02,pgsql03 | Template App PostgreSQL SR |
| PostgreSQL Cluster | Template App pgpool-II watchdog, Template App PostgreSQL SR Cluster |

* ホストグループを作成する。

| ホストグループ名 | ホスト名 |
|:-----------|:---------|
| pgpool | pgpool01,pgpool02 |
| PostgreSQL | pgsql01,pgsql02,pgsql03 |
| PostgreSQL Cluster | PostgreSQL Cluster |
