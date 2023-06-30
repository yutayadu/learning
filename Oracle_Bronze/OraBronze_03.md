# 3章 Oracle Enterprise Manager Database Express および SQL管理ツールの使用
# 3-1 Oracle Enterprise Manager Database Express
## 3-1-1 Oracle Enterprise Manager Database Express とは
- WebブラウザからGUI操作でOracleデータベースの管理作業を行えるツール
    - パフォーマンスの監視
    - 構成管理
    - 診断とチューニング
- EM Expressのメリットは、軽量でかつGUIで簡単に操作できる点
- OUIでOracleソフトウェアをインストールすると自動的にインストールされる
- DBCAでデータベースを作成する時に構成できる
## 3-1-2 EM Expressのアーキテクチャ
- リスナー、ディスパッチャ、共有サーバー、XML DB機能が連携して動作する
    - リスナー：ネットワーク経由の接続を中継するOracleのコンポーネント
    - ディスパッチャ、共有サーバー：共有サーバー接続と呼ばれる接続方法を可能にするコンポーネント
    - XML形式のデータをOracleデータベースで管理するためのコンポーネント
- EM Expressのコア部分はOracleデータベースの中で動作する
    - **EM ExpressからOracle1データベースを起動することはできない**

## 3-1-3 EM Express の構成
- リスナーが起動されていることを確認する
- **DISPATCHERS**初期化パラメータに、PROTOCOL=TCP属性が含まれていることを確認する
- **SHARED_SERVERS**初期化パラメータがゼロより大きいことを確認する
- **DBMS_XDB_CONFIG.SETHTTPSPORT**プロシージャを実行して、ExpressのHTTPSポートを設定する
- **EM Express** のHTTPSポートを確認する
    ```sql
    SQL> show parameter dispatchers
 
    NAME                                 TYPE        VALUE
    ------------------------------------ ----------- ------------------------------
    dispatchers                          string      (PROTOCOL=TCP) (SERVICE=orclXD
                                                 B)
    max_dispatchers                      integer
    ```
    ```sql
    SQL> show parameter shared_server
 
    NAME                                 TYPE        VALUE
    ------------------------------------ ----------- ------------------------------
    max_shared_servers                   integer
    shared_server_sessions               integer
    shared_servers                       integer     1
    ```
    ```sql
    SQL> exec DBMS_XDB_CONFIG.SETHTTPSPORT(5500);
 
    PL/SQLプロシージャが正常に完了しました。
    ```
    ```sql
    SQL> SELECT DBMS_XDB_CONFIG.GETHTTPSPORT FROM DUAL;
 
    GETHTTPSPORT
    ------------
            5500
    ```
- EM ExpressのURLは以下の通り。
    ```
    htttps://<データベースサーバーのホスト名またはIPアドレス>:<EM ExpressのHTTPSポート番号>/em
    ```

### リスナーを起動※後でてくるけど
```bash
$ netmgr

$ ls -l /u01/app/oracle/product/19.0.0/dbhome_1/network/admin
    listener.ora

$ lsnrctl start
 
    LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 29-6月 -2023 03:47:57
 
    Copyright (c) 1991, 2019, Oracle.  All rights reserved.
 
    /u01/app/oracle/product/19.0.0/dbhome_1/bin/tnslsnrを起動しています。お待ちください...
 
    TNSLSNR for Linux: Version 19.0.0.0.0 - Production
    システム・パラメータ・ファイルは/u01/app/oracle/product/19.0.0/dbhome_1/network/admin/listener.ora
    です。
    ログ・メッセージを/u01/app/oracle/diag/tnslsnr/localhost/listener/alert/log.xmlに書き込みました。
    リスニングしています: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521)))
 
    (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost.localdomain)(PORT=1521)))に接続中
    リスナーのステータス
    ------------------------
    別名                      LISTENER
    バージョン                TNSLSNR for Linux: Version 19.0.0.0.0 - Production
    開始日                    29-6月 -2023 03:47:58
    稼働時間                  0 日 0 時間 0 分 0 秒
    トレース・レベル          off
    セキュリティ              ON: Local OS Authentication
    SNMP                      OFF
    パラメータ・ファイル      /u01/app/oracle/product/19.0.0/dbhome_1/network/admin/listener.ora
    ログ・ファイル            /u01/app/oracle/diag/tnslsnr/localhost/listener/alert/log.xml
    リスニング・エンドポイントのサマリー...
      (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521)))
    リスナーはサービスをサポートしていません。
    コマンドは正常に終了しました。
```
```bash
    # ポートフォアリングで画面を確認
    # ローカルホストで以下を実行
    ssh 192.168.21.4 -L 5500:localhost:5500
```

## 3-1-4 EM Expressのページ
- 管理機能
    - 構成
    - 記憶域
    - セキュリティ
    - パフォーマンス
- 以下の機能はない
    - 起動や停止
    - バックアップ/リカバリ
- 以下の管理作業を実行可能（なお、現在はJET版のみのため、※だけ）
    - 初期化パラメータの編集
    - 表領域の管理（作成、変更、削除）
    - UNDO管理（表領域の切り替え、分析パラメータの編集）
    - REDOログファイルの管理（追加、削除、多重化）
    - 制御ファイルの管理（トレースにバックアップ）
    - ユーザーの管理（作成、変更、削除）
    - ロールの管理（作成、削除）
    - ADDMによって検出されたパフォーマンスの結果と推奨事項の表示 **※**
    - AWRに取得した統計の表示　**※**
    - SQLチューニングアドバイザの実行 **※**

## 3-1-5 EM Express と Enterprise Manager Cloud Control
- 複数のサーバーに配置された複数のOracleデータベース、その他の製品を統合的に管理できるツール
    - 管理対象サーバーには、**Oracle Management Agent**を配置する必要がある
    - 原則的にほぼすべてのデータベースの管理作業を実行できる

        | EM Expressにはできず、Cloud Control にはできること |
        | --- |
        | インスタンス（データベース）の起動/停止 |
        | スキーマ（表/索引）の管理 |
        | ジョブの管理および実行 |
        | バックアップ/リカバリ |
        | データベースリソースマネージャ |
        | バッチ推奨（My Oracle Supportパッチ取得） |
        | 複数データベースの統合管理 |
        | セグメントアドバイザ(領域の断片化を分析するアドバイザ) |

# 3-2 SQL*Plus
## 3-2-1 SQL*Plusとは
- SQL、PL/SQLコマンド、SQL*Plusコマンド
    - PL/SQLコマンド　：　Oracle独自のプログラミング言語
        - if文の拡張
        - プログラムをOracleデータベースみ保管できるのが特徴
        - 使用形態
            - ストアドプログラム　  ：　プログラムをデータベースに保管し、繰り返し利用できる
            - トリガー ：　DML実行などの特定のイベントが発生したときに実行
            - 無名ブロック  ：  一時的な用途でPL/SQLプログラムを実行するときに使用
    - SQL\*Plusコマンド　：　SQL*Plusの実行環境を設定するコマンド

## 3-2-2 SQL*Plusの起動と接続
```bash
[oracle@localhost admin]$ sqlplus
 
SQL*Plus: Release 19.0.0.0.0 - Production on 木 6月 29 05:26:51 2023
Version 19.3.0.0.0
 
Copyright (c) 1982, 2019, Oracle.  All rights reserved.
 
ユーザー名を入力してください: system
パスワードを入力してください:
最終正常ログイン時間: 木 6月  29 2023 04:19:31 +09:00
 
 
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0
に接続されました。
SQL>
```
- ユーザ名は英大文字/小文字が**区別されない**
- パスワードは英大文字/小文字が**区別される**

### ユーザ名とパスワードを指定したアクセス
```bash
sqlplus system/system
```
### SQL*Plus起動時にOracleデータベースに接続しない
- CONNECTコマンドでデータベース接続をする
```bash
sqlplus /nolog

CONNECT system/system
```
### SYSユーザでの接続とAS SYSDBAオプション
- SYSユーザー
    - データベースの起動および停止を含むすべての操作を実行できる(**SYSDBA**)
```bash
sqlplus sys/system as sysdba
```
```bash
sqlplus /nolog
connect sys/oracle as sysdba
```
- Oracleのインストール作業をしたOSユーザーでOSにログインしている場合、ユーザ名とパスワードを省略可能
    - **OS認証**
```bash
sqlplus / as sysdba
```

## 3-2-4 SQL*Plusの使い方
- 対話モード
    - 「SQL>」プロンプトに直接コマンドを入力して実行する
        - 単語の途中でなければ、改行が可能：行頭に「２」「３」などの数字が表示される
        - 「;」で文の終わりを判断する
- バッチモード
    - コマンドをファイルに記載しておき、まとめて実行する
        ```bash
        sqlplus /nolog @sample.sql
        ```
### OSコマンドの実行（HOSTコマンド）
- SQL*PlusからOSコマンドを実行できる
    ```bash
    [oracle@localhost admin]$ sqlplus /nolog
 
    SQL*Plus: Release 19.0.0.0.0 - Production on 木 6月 29 05:42:02 2023
    Version 19.3.0.0.0
 
    Copyright (c) 1982, 2019, Oracle.  All rights reserved.
 
    SQL> HOST ls
    listener.ora  samples  shrept.lst
    ```


>SQL*Plusでスクリプトを実行する方法として、正しいものはどれですか
```bash
@pingt.sql

or

START pingt.sql
```

>SQL*Plusで行えること
- インスタンスの起動、停止
- PL/SQLの実行
- SQL文の実行
- バックアップ、リカバリの実行

>データベース作成後、次のSQLを実行しpingtユーザーを作成しました。しかし、pingtユーザーでEnterprise Manager Database Expressにログインしようとするとエラーが発生しました
- pingtユーザーにEM_EXPRESS_BASICもしくはEM_EXPRESS_ALLロールを付与していないため

>Enterprise Manager Database Expressが使用するポート番号を確認する方法
- コマンドラインで「lsnrctl status リスナー名」を実行する
    - HOST lsnrctl status LISTENER
- SQL文「SELECT dbms_xdb_config.gethttpsport() FROM dual;」を実行する