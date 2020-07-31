# lsyncd

## 概要

この role では以下の2項目ができる.

* lsyncd のインストール・設定
* rsyncd のインストール


## 動作確認バージョン

* Ubuntu 18.04 (bionic)
* ansible >= 2.8.7
* Jinja2 2.10.3 


## 注意

複数ホストでの双方向同期も実現可能だが, 
アプリケーションログのように頻繁に更新されるファイルの双方向同期は次のような問題が発生するので
双方向同期の対象からは外すこと.

* 一度しか出力されないはずのログが複数出力される
* 出力されるはずのログが出力されない


## 使い方 (ansible)

### Role variables

```yaml
### インストール設定 ###############################################################################
## 基本設定
lsyncd_install_flag: True  # インストールフラグ


### conf ファイル設定 ##############################################################################
## 各デフォルト値
lsyncd_hosts: []            # 同期元ホスト名リスト (デフォルトは空のリスト)
lsyncd_targets: []          # ターゲット先 IP アドレスリスト (デフォルトは空のリスト)
lsyncd_rsync_opts_default:  # rsync オプションのデフォルト値
  archive: "true"

## settings
# ※ lsync.conf.lua の settings の項目
lsyncd_status_file: "/tmp/lsyncd.stat"
lsyncd_status_logfile: "/var/log/lsyncd.log"

## sync
# lsync.conf.lua の sync 設定するときに使用
lsyncd_sync_conf: []
# ※ 設定例
#lsyncd_sync_conf:
#  # デフォルト値に従った同期設定例
#  - sync_src: "/tmp/sync_hogehoge/"
#    module_name: "sample_sync"
#    excludes:
#      - ".*"
#      - "lost+found"
#    sync_delete: "running"
#    sync_init: false
#    src_hosts: "{{ lsyncd_hosts }}"  # ※ オプション. 必須ではない.
#    targets: "{{ lsyncd_targets }}"  # ※ オプション. 必須ではない.
#    rsync_opts:                      # ※ オプション. 必須ではない.
#      archive: "true"
#    comment: ""                      # 同期設定のコメント
#
#  # ※ infra-c から infra-a, infra-b への片方向同期設定例
#  - sync_src: "/tmp/sync_fugafuga/"
#    module_name: "sample_sync"
#    excludes:
#      - ".*"
#      - "lost+found"
#    sync_delete: "running"
#    sync_init: false
#    src_hosts:
#      - "infra-c"
#    targets:
#      - "infra-a"        # インベントリファイルに記載されているホスト名での指定方法
#      - "198.51.100.1"   # IP アドレスによる指定方法.
#    comment: "infra-c から infra-a, 198.51.100.1 への片方向同期"

lsyncd_sync_conf_by_target: {}
#lsyncd_sync_conf_by_target:
#  infra-a:  # インベントリホスト名で指定
#    - sync_src: "/tmp/sync_hogehoge/"
#      module_name: "sample_sync_1"
#      excludes:
#        - ".*"
#        - "lost+found"
#      sync_delete: "running"
#      sync_init: false
#      src_hosts: "{{ lsyncd_hosts }}"  # ※ オプション. 必須ではない. デフォルトは lsyncd_hosts
#      rsync_opts:                      # ※ オプション. 必須ではない. デフォルトは 記述している通り.
#        archive: "true"
#      comment: ""                      # 同期設定のコメント ※ オプション. 必須ではない.
#
#    - sync_src: "/tmp/sync_fugafuga/"
#      module_name: "sample_sync_2"
#      excludes:
#        - ".*"
#        - "lost+found"
#      sync_delete: "running"
#      sync_init: false
#      src_hosts: "{{ lsyncd_hosts }}"  # ※ オプション. 必須ではない. デフォルトは lsyncd_hosts
#      rsync_opts:                      # ※ オプション. 必須ではない. デフォルトは 記述している通り.
#        archive: "true"
#      comment: ""                      # 同期設定のコメント. ※ オプション. 必須ではない.

lsyncd_sync_conf_by_src_host: {}
#lsyncd_sync_conf_by_src_host:
#  infra-a:  # インベントリホスト名で指定
#    - sync_src: "/tmp/sync_hogehoge/"
#      module_name: "sample_sync_1"
#      excludes:
#        - ".*"
#        - "lost+found"
#      sync_delete: "running"
#      sync_init: false
#      targets: "{{ lsyncd_targets }}"  # ※ オプション. 必須ではない.
#      rsync_opts:                      # ※ オプション. 必須ではない. デフォルトは 記述している通り.
#        archive: "true"
#      comment: ""                      # 同期設定のコメント ※ オプション. 必須ではない.
#
#    - sync_src: "/tmp/sync_fugafuga/"
#      module_name: "sample_sync_2"
#      excludes:
#        - ".*"
#        - "lost+found"
#      sync_delete: "running"
#      sync_init: false
#      targets: "{{ lsyncd_targets }}"  # ※ オプション. 必須ではない.
#      rsync_opts:                      # ※ オプション. 必須ではない. デフォルトは 記述している通り.
#        archive: "true"
#      comment: ""                      # 同期設定のコメント. ※ オプション. 必須ではない.


### logrotate 設定 ################################################################################
lsyncd_use_logrotate: yes
lsyncd_logrotate_scripts:
  - name: lsyncd
    paths:
      - "{{ lsyncd_status_logfile }}"
    options:
      - daily
      - missingok
      - rotate 14
      - compress
      - delaycompress
      - notifempty
      - dateext

  - name: rsyncd
    paths:
      - /var/log/rsyncd.log
    options:
      - daily
      - missingok
      - rotate 14
      - compress
      - delaycompress
      - notifempty
      - dateext
```

### Example playbook

```yaml
- hosts:
    - servers
  roles:
    - { role: lsyncd, tags: ["lsyncd"] }
```


## 後方互換性について

### 削除された変数の一覧

以下の変数は `group_vars` から削除して頂いて大丈夫です.

* `rsyncd_hosts`
  * rsyncd のインストールに関する細かい設定は rsyncd role でしたほうが良いと考えたため削除しました.

* `rsyncd_ufw_allow_from`
  * ufw の設定に関する項目であり, それらは ufw role で実行する方針に切り替えました.

### 変更された変数の一覧

`group_vars` において以下のように新しい書き方に変えて頂いて大丈夫です.

* `lsyncd_sync_conf_defaults` → `lsyncd_sync_conf`
* `lsyncd_sync_conf_by_hostname` → `lsyncd_sync_conf_by_src_host` 