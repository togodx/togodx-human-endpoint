# togodx-human-endpoint
docker-compose.yml for TogoDX human endpoint

## URL一覧
### 開発環境
- SPARQList: https://togodx-dev.dbcls.jp/human/sparqlist/
  - SPARQLet GitHub: https://github.com/togodx/togodx-sparqlet/tree/master/dev
  - 旧SPARQLet (github.com/biosciencedbc/togosite-sparqlist) のアーカイブ： https://github.com/togodx/togosite-sparqlist-obsolete
- EP(proxy経由): https://togodx-dev.dbcls.jp/human/sparql
  - SPARQLet内で{{SPARQLIST_TOGODX_SPARQL}}として参照できる。
- EP(virtuoso直接): https://togodx-dev.dbcls.jp/human/virtuoso

### ステージング環境
- SPARQList: https://togodx-stg.dbcls.jp/human/sparqlist/
  - SPARQLet GitHub: https://github.com/togodx/togodx-sparqlet/tree/master/prod
- EP(proxy経由): https://togodx-stg.dbcls.jp/human/sparql
  - SPARQLet内で{{SPARQLIST_TOGODX_SPARQL}}として参照できる。
- EP(virtuoso直接): https://togodx-stg.dbcls.jp/human/virtuoso

### 本番環境
- SPARQList: https://togodx.dbcls.jp/human/sparqlist/ (編集不可）
  - SPARQLet GitHub: https://github.com/togodx/togodx-sparqlet/tree/master/prod (リリース時のcommit)
- EP(proxy経由):  https://togodx.dbcls.jp/human/sparql
  - SPARQLet内で{{SPARQLIST_TOGODX_SPARQL}}として参照できる。
- EP(virtuoso直接): https://togodx.dbcls.jp/human/virtuoso

## インストール手順
1. docker-compose.ymlやDockerfileなどを含むこのレポジトリをGitHubからcloneする
```
dbcls3284:prod-togodx-human-endpoint $ git clone https://github.com/togodx/togodx-human-endpoint
Cloning into 'togodx-human-endpoint'...
remote: Enumerating objects: 34, done.
remote: Counting objects: 100% (34/34), done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 34 (delta 12), reused 27 (delta 8), pack-reused 0
Receiving objects: 100% (34/34), 5.29 KiB | 2.65 MiB/s, done.
Resolving deltas: 100% (12/12), done.
```
2. sparqletをGitHubからcloneする
```
dbcls3284:prod-togodx-human-endpoint $ git clone https://github.com/togodx/togodx-sparqlet
Cloning into 'togodx-sparqlet'...
remote: Enumerating objects: 2901, done.
remote: Counting objects: 100% (68/68), done.
remote: Compressing objects: 100% (48/48), done.
remote: Total 2901 (delta 27), reused 36 (delta 20), pack-reused 2833
Receiving objects: 100% (2901/2901), 471.24 KiB | 10.71 MiB/s, done.
Resolving deltas: 100% (1599/1599), done.
dbcls3284:prod-togodx-human-endpoint $ ls
togodx-human-endpoint	togodx-sparqlet
```
3. .envファイルを作成する
```
dbcls3284:prod-togodx-human-endpoint $head togodx-human-endpoint/.env
# Project name
COMPOSE_PROJECT_NAME=prod-togodx-human-endpoint-0

# Nginx
NGINX_PORT=1208
```
4. .env/VIRTUOSO_VOLUMES_DATABASEに指定したディレクトリにvirtuoso.ini, virtuoso.dbファイルなどを配置する
```
dbcls3284:prod-togodx-human-endpoint $ grep VIRTUOSO_VOLUMES_DATABASE .env
VIRTUOSO_VOLUMES_DATABASE=./virtuoso/togodx-2023-03-30
mitsuhashi@togov01:~/prod-togodx-human-endpoint/togodx-human-endpoint$ ls -R virtuoso/
virtuoso/:
togodx-2023-03-30

virtuoso/togodx-2023-03-30:
virtuoso-temp.db  virtuoso.db  virtuoso.ini  virtuoso.lck  virtuoso.log  virtuoso.pxa  virtuoso.trx
dbcls3284:prod-togodx-human-endpoint $
```
5. コンテナを起動する
```
dbcls3284:togodx-human-endpoint $ docker-compose up -d
dbcls3284:togodx-human-endpoint $ docker-compose ps
                 Name                               Command               State                   Ports
------------------------------------------------------------------------------------------------------------------------
togodx-human-endpoint-1_nginx_1          /docker-entrypoint.sh ngin ...   Up      0.0.0.0:12080->80/tcp,:::12080->80/tcp
togodx-human-endpoint-1_redis_1          docker-entrypoint.sh redis ...   Up      6379/tcp
togodx-human-endpoint-1_sparql-proxy_1   docker-entrypoint.sh /bin/ ...   Up
togodx-human-endpoint-1_sparqlist_1      docker-entrypoint.sh /bin/ ...   Up
togodx-human-endpoint-1_virtuoso_1       /virtuoso-entrypoint.sh start    Up      1111/tcp, 8890/tcp
```
なお、コンテナを停止する場合は以下を実行する
```
dbcls3284:togodx-human-endpoint $ docker-compose down
```
また、SPARQLetをGitHubと同期するためのCronジョブを設定する。
```
$ crontab -l
# m h  dom mon dow   command
15,45 * * * * timeout 15m /opt/services/stg-togodx-human-endpoint/togodx-sparqlet/togodx-sparqlet-sync-github
$ cat /opt/services/stg-togodx-human-endpoint/togodx-sparqlet/togodx-sparqlet-sync-github
#!/bin/bash

SCRIPT_DIR=$(cd $(dirname $0); pwd)

cd $SCRIPT_DIR

git add -A .
git commit -m 'auto commit'
git pull --rebase
git push
mitsuhashi@togov01:~$
```

## docker-compose.ymlの.envファイル
```
# Project name
COMPOSE_PROJECT_NAME=<<同一サーバに複数インスタンスを立てる場合は重複しない文字列を指定する>>

# Nginx
NGINX_PORT=<<NGINXのポート番号>>

# SPARQList
SPARQLIST_ADMIN_PASSWORD=<<管理者パスワード>>

## Default SPARQL endpoint for production
SPARQLIST_TOGODX_SPARQL=https://togodx.dbcls.jp/human/sparql
## Default SPARQL endpoint for staging
#SPARQLIST_TOGODX_SPARQL=https://togodx-stg.dbcls.jp/human/sparql
## Default SPARQL endpoint for development
#SPARQLIST_TOGODX_SPARQL=https://togodx-dev.dbcls.jp/human/sparql

## SPARQLet for production (togodx.dbcls.jp/human) and staging (togodx-stg.dbcls.jp/human)
TOGODX_SPARQLET=../togodx-sparqlet/prod/
## SPARQLet for development
#TOGODX_SPARQLET=../togodx-sparqlet/dev/

## SPARQL-Proxy
SPARQL_PROXY_ADMIN_PASSWORD=<<管理者パスワード>>

# Virtuoso
VIRT_DBA_PASSWORD=<<管理者パスワード>>
VIRTUOSO_VOLUMES_DATABASE=./virtuoso/<<virtuoso.iniを含むディレクトリ名>>
```
