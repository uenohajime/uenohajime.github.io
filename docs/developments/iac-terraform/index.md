# Terraform を利用した IaC について

Terraform は、HashiCorp 製の Infrastructure as Code (IaC) です。安全に繰り返しインフラストラクチャを構築、変更、管理するためのツールです。運用者とインフラ・チームは、 HashiCorp 構成言語 (HCL) と呼ばれる構成言語で環境を管理できます。
([Terraform HP より](https://learn.hashicorp.com/tutorials/terraform/infrastructure-as-code?in=terraform/azure-get-started))

## IaC のメリットとデメリット

まず、 IaC のメリットは、簡単に環境を再現できることです。開発環境で定義した環境を簡単にステージング環境に設置することが可能になります。さらに、他の人が環境を再現したい時に簡単に環境を定義することが可能になります。

一方でIaC を利用するデメリットは、次のようなことが考えられます。
システムの巨大化するにつれて、メンテナンスコストが高くなってくることです。また、 IaC 実行でエラーが発生した場合、実行前の環境に戻すことが容易ではありません。
([redhat記事](https://www.redhat.com/sysadmin/pros-and-cons-infrastructure-code))

## Terraform の基本的な流れ

## 実装のポイント

## 参考サイト

* [Terraform](https://www.terraform.io/)