# aws_moto_docker_example1

実験中

## 概要
* AWS の Mock の Moto を Docker で試してみる

Moto: Mock AWS Services  
https://docs.getmoto.org/en/latest/  
AWS インフラストラクチャに基づいたテストを簡単にモック化できるライブラリ  

GitHub - Moto - Mock AWS Services  
https://github.com/getmoto/moto  

DockerHub - motoserver/moto  
https://hub.docker.com/r/motoserver/moto  

## 環境
* Ubuntu 24.04
* Docker version 27.3.1, build ce12230

## 詳細

### AWS CLIの最新バージョンのインストールまたは更新  
https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html  

Command line installer - Linux x86 (64-bit)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

```
$ aws --version
aws-cli/2.18.6 Python/3.12.6 Linux/6.8.0-45-generic exe/x86_64.ubuntu.24
```

### Moto サーバー起動

Run using Docker  
https://docs.getmoto.org/en/latest/docs/server_mode.html#run-using-docker  

```
$ docker compose up
[+] Running 9/9
 ✔ motoserver Pulled                                                                                                                                                                                             10.8s 
   ✔ 302e3ee49805 Already exists                                                                                                                                                                                  0.0s 
   ✔ 4aeebc636537 Pull complete                                                                                                                                                                                   1.7s 
   ✔ 2aee89346c42 Pull complete                                                                                                                                                                                   2.3s 
   ✔ 0538735e543f Pull complete                                                                                                                                                                                   2.4s 
   ✔ 0fe6e0364519 Pull complete                                                                                                                                                                                   3.0s 
   ✔ 4f4fb700ef54 Pull complete                                                                                                                                                                                   3.0s 
   ✔ 4cd65ff7d65c Pull complete                                                                                                                                                                                   6.5s 
   ✔ f4d1330797d8 Pull complete                                                                                                                                                                                   6.5s 
[+] Running 2/2
 ✔ Network aws_moto_docker_example1_default         Created                                                                                                                                                       0.1s 
 ✔ Container aws_moto_docker_example1-motoserver-1  Created                                                                                                                                                       0.3s 
Attaching to motoserver-1
motoserver-1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
motoserver-1  |  * Running on all addresses (0.0.0.0)
motoserver-1  |  * Running on http://127.0.0.1:3000
motoserver-1  |  * Running on http://172.20.0.2:3000
motoserver-1  | Press CTRL+C to quit
```
> WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
警告: これは開発サーバーです。運用環境では使用しないでください。代わりに実稼働 WSGI サーバーを使用してください。

### ダッシュボード
http://localhost:3000/moto-api/

### aws コマンド

#### プロファイル作成
```
$ aws configure --profile moto
AWS Access Key ID [None]: dummy
AWS Secret Access Key [None]: dummy
Default region name [None]: ap-northeast-1                
Default output format [None]: json
```

#### S3 バケット作成
```
$ aws s3 mb s3://test-bucket \
    --endpoint-url=http://localhost:3000 \
    --profile moto
make_bucket: test-bucket
```
```
$ aws s3api create-bucket \
    --bucket sample-bucket \
    --create-bucket-configuration LocationConstraint=ap-northeast-1 \
    --endpoint-url=http://localhost:3000 \
    --profile moto
```

#### S3 バケット一覧取得
```
$ aws s3 ls \
    --endpoint-url=http://localhost:3000 \
    --profile moto
2024-10-15 22:05:19 test-bucket
```
```
$ aws s3api list-buckets \
    --endpoint-url=http://localhost:3000 \
    --profile moto
{
    "Buckets": [
        {
            "Name": "test-bucket",
            "CreationDate": "2024-10-15T13:05:19+00:00"
        },
        {
            "Name": "sample-bucket",
            "CreationDate": "2024-10-15T13:26:39+00:00"
        }
    ],
    "Owner": {
        "DisplayName": "webfile",
        "ID": "bcaf1ffd86f41161ca5fb16fd081034f"
    }
}
```

#### S3 ファイルアップロード
```
$ aws s3 cp index.html s3://test-bucket/ \
    --endpoint-url=http://localhost:3000 \
    --profile moto
upload: ./index.html to s3://test-bucket/index.html    
```
```
$ aws s3api put-object \
    --bucket sample-bucket \
    --key index.html \
    --body index.html \
    --endpoint-url=http://localhost:3000 \
    --profile moto
{
    "ETag": "\"a5c24d869fe449303807c2ebec7a4d2a\""
}
```

#### S3 オブジェクトの署名済み URL を生成
```
$ aws s3 presign s3://test-bucket/index.html \
    --endpoint-url=http://localhost:3000 \
    --profile moto
http://localhost:3000/test-bucket/index.html?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=dummy%2F20241015%2Fap-northeast-1%2Fs3%2Faws4_request&X-Amz-Date=20241015T133206Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=07e200497bf41ca53c5ee49ef757e21b928d85f96d4953a2c8a4f67510fed676
```

↑で生成された URL にアクセス
![alt text](images/README/image.png)

#### 実験中
```
aws s3api put-bucket-cors \
    --bucket sample-bucket \
    --cors-configuration file://cors-config.json \
    --endpoint-url=http://localhost:3000 \
    --profile moto
```

```
aws s3 website s3://sample-bucket/ \
    --index-document index.html \
    --endpoint-url=http://localhost:3000 \
    --profile moto
```

```
aws s3api put-bucket-policy \
    --bucket sample-bucket \
    --policy file://policy.json \
    --endpoint-url=http://localhost:3000 \
    --profile moto
```

```
aws s3api put-public-access-block \
    --bucket sample-bucket \
    --public-access-block-configuration \
    "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false" \
    --endpoint-url=http://localhost:3000 \
    --profile moto

```

```
aws s3api put-bucket-policy \
    --bucket sample-bucket \
    --policy file://./bucket-policy.json \
    --endpoint-url=http://localhost:3000 \
    --profile moto

```

http://localhost:3000/sample-bucket/index.html
http://localhost:3000/test-bucket/index.html

#### S3 バケット内のオブジェクト一覧取得
```
$ aws s3 ls s3://test-bucket/ \
    --endpoint-url=http://localhost:3000 \
    --profile moto
2024-10-15 22:14:44        242 index.html
```

#### S3 バケット内のオブジェクト削除
```
$ aws s3 rm s3://test-bucket/index.html \
    --endpoint-url=http://localhost:3000 \
    --profile moto
delete: s3://test-bucket/index.html
```

#### S3 バケットの削除
```
$ aws s3 rb s3://sample-bucket/ \
    --force \
    --endpoint-url=http://localhost:3000 \
    --profile moto
delete: s3://sample-bucket/index.html
remove_bucket: sample-bucket
```