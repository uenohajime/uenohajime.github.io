# Azure App Service + SQL Server ハンズオン (2020/11)

このハンズオンでは、次の図に示すように App Service と SQL Server を連携させた ToDo アプリを設置する方法を実践します。設置作業は、 Azure ポータルと Azure Cloud Shell を用いて行います。

<div align="center">
    <img src="imgs/00-goal-architecture.png" width="60%">
</div>

## 0. 事前準備

このハンズオンでは、 Azure ポータルと Azure Cloud Shell で作業を行います。予めAzureアカウントの取得とサブスクリプションの作成、及び、会社でアカウントが払い出された場合、適切なアクセス権の設定が必要となります。


## 1. SQL Database の作成

ここでは SQL Database を作成します。 SQL Database の作成とは、 SQL Server を設置して、 SQL Server 上にデータベースを作成していくという手順となります。 Azure では SQL Database 作成画面で SQL Server を作成することが可能です。

このハンズオンでは、次の画面に示すようにパラメータを設定していきます。


## 2. App Service 作成とWebアプリ設置

ここでは App Service を作成し、 Web アプリを設置します。

## 3. App Service への Web アプリの複数バージョン設置

ここでは App Service に複数のアプリを設置します。

## 4. App Service の Blue / Green デプロイメント

ここでは 1 つの App Service に設置された複数のアプリへアクセスするユーザの流れを調整します。

## 参考サイト

* [チュートリアル:Azure App Service での ASP.NET Core および Azure SQL Database アプリの作成](https://docs.microsoft.com/ja-jp/azure/app-service/tutorial-dotnetcore-sqldb-app?pivots=platform-windows)
* [Azure SQL Database 料金](https://azure.microsoft.com/ja-jp/pricing/details/sql-database/single/)
* [Azure App Service 料金](https://azure.microsoft.com/ja-jp/pricing/details/app-service/windows/)

## スライド

* [スライド (パスワード付き)](../../../pdf/AppService-NET-SQL-DB.zip)
