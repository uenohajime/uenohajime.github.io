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

| パラメータ名 | 値 |
| ---------- | -- |
| サーバ名 | handson4<名字> |
| サーバ管理者ログイン | dbadmin |
| パスワード | P@ssw0rd |
| 場所 | 東日本 |

| パラメータ名 | 値 |
| ---------- | -- |
| リソースグループ | handson4<名字> |
| データベース名 | coreDB |
| エラスティックプール | いいえ |
| コンピューティングストレージ | サーバレス、Gen5、1 vCore、1 GB ストレージ |

| パラメータ名 | 値 |
| ---------- | -- |
| 接続方法 | パブリック エンドポイント |
| ファイアウォール規則：Azure サービス | はい |
| ファイアウォール規則：現在のクライアント | いいえ |

| パラメータ名 | 値 |
| ---------- | -- |
| 既存データの使用 | なし |
| 照合順序（変更なし） | SQL_Latin1_General_CP1_CI_AS |
| Azure Defender for SQL | 後で |

```
git clone https://github.com/uenohajime/dotnetcore-sqldb-tutorial.git
```

```
cd dotnetcore-sqldb-tutorial
```

```
code .
```

```
services.AddDbContext<MyDatabaseContext>(options =>
   options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
```

```
rm Migrations -r
```

```
dotnet ef migrations add InitialCreate
```

```
az sql db show-connection-string --client ado.net --server handson4<名字> --name coreDB
```

```
export ConnectionStrings__MyDbConnection="Server=tcp:handson4<名字>.database.windows.net,1433;Database=coreDB;User ID=dbadmin;Password=P@ssw0rd;Encrypt=true;Connection Timeout=30;"
```

```
dotnet ef database update
```


## 2. App Service 作成とWebアプリ設置

ここでは App Service を作成し、 Web アプリを設置します。

| パラメータ名 | 値 |
| ---------- | -- |
| リソースグループ | handson4<名字> |
| 名前 | handson4<名字> |
| 公開 | コード |
| ランタイム スタック | .NET Core 2.1 |
| オペレーティング システム | Windows |
| 地域 | Japan East |
| Windows プラン | handson4<名字> |
| SKUとサイズ | Standard S1 |

| パラメータ名 | 値 |
| ---------- | -- |
| 名前 | MyDbConnection |
| 値 | Server=tcp:handson4<名字>.database.windows.net,1433;Database=coreDB;User ID=dbadmin;Password=P@ssw0rd;Encrypt=true;Connection Timeout=30; |
| 種類 | SQLAzure |

```
cd dotnetcore-sqldb-tutorial
```

```
git config --global user.email “you@example.com” && git config --global user.name “Your Name”
```

```
git add . && git commit –m “connect to SQLDB in Azure”
```

```
az webapp deployment user set --user-name handson4<名字> --password P@ssw0rd
```

```
git remote add azure https://handson4<名字>.scm.azurewebsites.net:443/handson4<名字>.git
```

```
git push azure master
```


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
