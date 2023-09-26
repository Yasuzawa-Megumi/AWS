# S3 からファイルを取得する

### AWS SDK for Python (Boto3)

#### Boto3：Python から操作するためのライブラリ

- クライアント API(boto3.client)：AWS の各リソースが持っている REST API と 1 対 1 に対応した単純な Python ラッパー
- リソース API(boto3.resource)：AWS のリソースをオブジェクト指向のプログラムで操作できるようにしたもの(resource を使うのがベスト)

<参考 URL>
[Boto3 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)

---

### S3 からファイルを読み込む

- s3.Object()：Boto3 の s3.Object() メソッドを使用して、指定されたバケットとキーに対応する S3 オブジェクトを取得
- input_key（バケット名を除くフォルダの出力ファイルパス）

  例：buket_name が「dev_test」の場合、key はその後のフォルダ階層を指定「10_master/10_import/dev/test」(先頭に／は不要)

```
import boto3
import pandas as pd

buket_name = 'your-bucket-name'
input_key = 'directory_name'

s3 = boto3.resource('s3')

src_obj = s3.Object(buket_name,input_key)
df = pd.read_csv(src_obj.get()['Body'],delimiter=',',dtype=object)
print(df)
```

---

### S3 へファイルを出力する

```
outout_key = 'directory_name'

# s3出力処理
s3_obj = s3.Object(buket_name,outout_key)
s3_obj.put(Body=df.to_csv(index=False).encode('utf_8'))
```

---

### S3 から parquet ファイルを取得する

- 下記コードは AWS Glue で使用した例
- bucket_uri は 's3://your-bucket-name/directory_name' の形式になる

```
import s3fs
import pyarrow.parquet as pq

fs = s3fs.S3FileSystem()

bucket = 'your-bucket-name'
path = 'directory_name'
bucket_uri = f's3://{bucket}/{path}'

dataset = pq.ParquetDataset(bucket_uri, filesystem=fs)
table = dataset.read()
df = table.to_pandas()
```
