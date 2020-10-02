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

ビルド中に設定をみていきます。  
`builders` に記載されている項目と内容は下記になります。  

| 項目名 | 設定値 | 内容 |
| ---- | ---- | ---- |
| type | amazon-ebs | ビルダーをEC2上に立ち上げる|
| region | ap-northeast-1 | ビルダーを起動するregion。設定値は東京リージョン |
| source_ami | ami-0f9ae750e8274075b | ビルダーを起動するEC2のAMI。設定値はAmazon Linux2 |
| instance_type | t2.micro | ビルダーを起動するEC2のインスタンスサイズ |
| ssh_username | ec2-user | SSH接続するユーザ名。ここではAmazon Linuxのユーザ名。 |
| ami_name | packer-lesson-ami | 作成するAMIの名前 |

ビルドが完了しましたら、AWSコンソール上から、AMIを使ってEC2を立ち上げてみましょう。  
立ち上がって、80でアクセスして、Apacheのページが出たら完了です。

### Ansible

Packerは構成管理ツールのAnsibleを使うこともできます。
先程記載したものをAnsible化してみます。

まずはApacheをインストールするplaybookを用意します。  
playbooksディレクトリを作成し、webserver.yamlを作ってください。  
webserver.yamlには下記の内容を記載します。  

```yaml
- name: Configure webservers
  hosts: all
  become: true
  tasks:
      - name: Install apache
        yum:
          name: httpd
          state: latest

      - name: enabled apache
        systemd:
          name: httpd
          state: started
          enabled: yes
```

これでAnsibleの準備は完了です。  
では次にPackerの設定を用意しましょう。ansible.jsonを作成し、下記の内容を記載してください。

```json
{
    "builders": [
      {
        "type": "amazon-ebs",
        "region": "ap-northeast-1",
        "source_ami": "ami-0f9ae750e8274075b",
        "instance_type": "t2.micro",
        "ssh_username": "ec2-user",
        "ami_name": "packer-lesson-ami_{{isotime \"2006_01_02_03_04\"}}"
      }
    ],
    "provisioners": [
      {
        "type": "shell",
        "inline": [
            "sudo yum clean all",
            "sudo yum -y update",
            "sudo amazon-linux-extras install epel",
            "sudo yum -y install ansible"
        ]
      },
      {
        "type": "ansible-local",
        "playbook_file": "./webserver.yaml"
      }
    ]
}
```

ポイントは、インラインでシェルを実行するの際に、Ansibleをインストールしておくことです。これがないとAnsibleが実行できません。

準備ができましたら、先程と同様に構文の確認をし、エラーが無いことを確認して実行します。

```
# 構文の確認
packer validate ansible.json

packer build ansible.json
```

ビルドが完了しましたら、AWSコンソール上から、AMIを使ってEC2を立ち上げてみましょう。  
立ち上がって、80でアクセスして、Apacheのページが出たら完了です。