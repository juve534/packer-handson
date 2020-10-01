# packer-handson
Packerのハンズオン資料

## 概要
Packerについて、ハンズオンを通して理解を深めていきます。

## 事前準備

当ハンズオンには事前準備が必要です。

事前にAWS環境とPackerを実行できる環境を用意しておいてください。  
Packerの環境は自身のマシンでも、EC2を立てても問題ありません。  

参考までにMacならhomebrewでインストールが可能です。

```
brew install packer
```

## Pakcerとは
最初にPackerについて説明します。

>HashiCorp Packer automates the creation of any type of machine image. It embraces modern configuration management by encouraging you to use automated scripts to install and configure the software within your Packer-made images. Packer brings machine images into the modern age, unlocking untapped potential and opening new opportunities.

Packerとは、HashiCorp社が作ったマシンイメージを作成するツールです。  
AWS, GCP, Azureといった主要なクラウドサービスのマシンイメージの生成や、vmwareのイメージも作成できます。  
※vmwareのイメージを生成できるので、オンプレミスでも使えるらしい

サーバを1台用意するだけなら、SSHしてコマンド実行でも良いかもですが、複数台となると時間もかかるし、ミスの可能性も増えます。Packerでマシンイメージを作ることで、そういったミスを減らすことができます。

## ハンズオン


### Hello World

まずは簡単にApacheをインストールしたイメージを作成してみます。  
好きなディレクトリで、ファイルを作成してください。  
PackerはjsonとHCLの2つから記載を選べますが、当ハンズオンではjsonで進めます。

下記の内容を書いた `packer.json` というファイルを作成してください。

```json

{
  "builders": [
    {
      "type": "amazon-ebs",
      "region": "ap-northeast-1",
      "source_ami": "ami-0f9ae750e8274075b",
      "instance_type": "t2.micro",
      "ssh_username": "ec2-user",
      "ami_name": "packer-lesson-ami"
    }
  ],
  "provisioners": [
    {
      "type":  "shell",
      "inline": [
        "sudo yum -y update",
        "sudo yum -y install httpd" ,
        "sudo systemctl enable httpd"
      ]
    }
  ]
}
```

作成したら、Packerを使ってマシンイメージを生成します。
構文の確認をし、エラーが無いことを確認して実行します。

```
# 構文の確認
packer validate packer.json

packer build packer.json
```

ビルドが完了しましたら、AWSコンソール上から、AMIを使ってEC2を立ち上げてみましょう。  
立ち上がって、80でアクセスして、Apacheのページが出たら完了です。