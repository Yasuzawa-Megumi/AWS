# AWS DMS を使用し DB を移行する

<img alt="構成図" src="img/AWS DMS/image-1.png" width="60%">

#### DMS の移行プロセス

<img alt="DMSの移行プロセス" src="img/AWS DMS/image-2.png" width="60%">

---

### ⓵ 移行前の準備作業

1. ターゲットデータベース作成(Aurora Serverless v2)
2. 移行前評価の準備

   - バケット作成
     ：移行前評価のレポートが S3 バケットに出力するため、「dms-liddeldev-test-bucket」をレポート配置用のバケットとして作成する
   - ロール作成：DMS サービスから S3 バケットにアクセスするため、作成したバケットに権限を付与するようにして、「dms-s3-test-policy」という名前のポリシーを作成する

     - 以下の公式ドキュメントを参照してロールを作成する

       [タスクの移行前評価の有効化と操作](https://docs.aws.amazon.com/ja_jp/dms/latest/userguide/CHAP_Tasks.AssessmentReport.html)

   - 「dms-s3-test-role」という名前の DMS サービスのロールを作成し、「dms-s3-test-policy」ポリシーをアタッチする

3. 移行ログ出力の準備

   - ロール作成
     ：DMS サービスから CloudWatchLogs にアクセスするため、「dms-cloudwatch-logs-role」という名前の DMS サービスロールを作成する

     ※上記には「AmazonDMSCloudWatchLogsRole」をアタッチする

### ⓶ 移行タスクの作成

1. サブネットグループ作成
   ：RDS と Aurora が属する Private Subnet を選択し、サブネットグループを作成する
2. レプリケーションインスタンス作成
   - レプリケーションインスタンスとデータベースが接続できるよう SG インバウンドルールに RDS を追加し、「rep-instance-sg」という名前で作成する
   - 1 で作成したサブネットグループ と作成したセキュリティグループ以外はデフォルト
3. ソースエンドポイント(RDS)、ターゲットエンドポイント(Aurora)を作成する

   - エンドポイントの接続テストを実施し、Success になることを確認する
   - ターゲット、ソース DB 共にセキュリティグループの追加を行わないと Success にならないため注意！

     例：Aurora の VPC のセキュリティグループである「rds-ec2~」のインバウンドルールにタイプ：MYSQL、ソース：rep-instance-sg を設定する
     ⇒ レプリケーションインスタンスからのアクセスを許可する必要がある

4. 全ロードの移行タスク作成

   - 作成したレプリケーションインスタンス、エンドポイント、移行タイプに全ロードの「既存のデータを移行する」を選択する
   - 選択ルールでは、新しい選択ルールの追加を押下し移行するスキーマを設定する

   ★ 細かい移行ルールの作成は参考を参照する

   ※下記のようにスキーマ、テーブルの全てをコピーするルールを設定し、その後、移行先の DB にデフォルトである mysql など移行したくないスキーマをアクション「含まない」で追加設定していく

     <img alt="スキーマ移行設定" src="img/AWS DMS/image-3.png" width="60%">

   - 移行前評価を有効にする(事前作業で作成したバケットとロールを選択)
   - 「後で手動で行う」を選択して、タスクを作成する

---

### ⓷ データ移行

1. RDS、Aurora の VPC にアタッチされているセキュリティグループで「rep-instance-sg」からの通信を許可するインバウンドルールを追加する
2. アクション>開始/再開からデータ移行を実行する

---

<参考 URL>

[AWS Database Migration Service（AWS DMS）を使ったデータ移行](https://ent.iij.ad.jp/articles/2499/)

[AWS DMS 使用方法](https://cloud5.jp/dms-task-20220513/)

---

### JSON を使用するテーブルテーブルマッピング

- 移行対象のテーブルに対して列追加やカラム削除など細かな指定を行いたい場合は「選択ルールと選択アクション」「変換ルールおよび変換アクション」を参照する
- SQLite 関数であれば型変換などのクエリを書くことも可能

※書き方は sample_code 配下のの json ファイルを参照

[JSON を使用するテーブル選択および変換を指定する](https://docs.aws.amazon.com/ja_jp/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TableMapping.SelectionTransformation.html)
