# Azure App Service + SQL Server ハンズオン (2020年12月 更新)

このハンズオンでは、次の図に示すように App Service と SQL Server を連携させた ToDo アプリを設置する方法を実践します。設置作業は、 Azure ポータルと Azure Cloud Shell を用いて行います。

<div align="center">
    <img src="imgs/00-goal-architecture.png" width="60%">
</div>


## 0. 事前準備

このハンズオンでは、 Azure ポータルと Azure Cloud Shell で作業を行います。予めAzureアカウントの取得とサブスクリプションの作成、及び、会社でアカウントが払い出された場合、適切なアクセス権の設定が必要となります。


## 1. SQL Database の作成

ここでは SQL Database を作成します。 SQL Database の作成とは、 SQL Server を設置して、 SQL Server 上にデータベースを作成していくという手順となります。 Azure では SQL Database 作成画面で SQL Server を作成することが可能です。


### 1-1. Azure ポータルからデータベースを作成

まず、SQL Database 作成画面へ移動します。Azure ポータルの左上にあるアイコンをクリックして、メニューを開き、「リソースの作成」ボタンをクリックします。

<div align="center">
    <img src="imgs/01-choose-create-resource-menu.png" width="60%">
</div>

次にSQL Databaseをメニューで選択します。

<div align="center">
    <img src="imgs/02-choose-create-sql-database-menu.png" width="60%">
</div>

次に SQL Database 作成のフォームが表示されるので、次のように入力します。SQL Database 作成画面で 「サーバ > 新規作成」を押すと、画面の右側に SQL Server の作成画面が現れます。ここでは、SQL Serverも同時に作成します。

<div align="center">
    <img src="imgs/03-fill-create-sql-database-general-settings-form.png" width="60%">
</div>

作成する SQL Server の情報は次のようになります。

| パラメータ名 | 値 |
| ---------- | -- |
| サーバ名 | handson4<名字> |
| サーバ管理者ログイン | dbadmin |
| パスワード | P@ssw0rd |
| 場所 | 東日本 |

作成する SQL Database の情報は次のようになります。

| パラメータ名 | 値 |
| ---------- | -- |
| リソースグループ | handson4<名字> |
| データベース名 | coreDB |
| エラスティックプール | いいえ |
| コンピューティングストレージ | サーバレス、Gen5、1 vCore、1 GB ストレージ |

入力が完了したら、「次: ネットワーク」ボタンを押し次のページに進みます。

<div align="center">
    <img src="imgs/04-fill-create-sql-database-network-settings-form.png" width="60%">
</div>

ネットワーク情報の設定画面では、次のように設定します。

※ クエリエディタでテーブルが作成されたことを確認する場合、「ファイアウォール規則：現在のクライアント」の値を「はい」に設定する必要があります。ただし、運用データベースではこの設定は推奨されません。

| パラメータ名 | 値 |
| ---------- | -- |
| 接続方法 | パブリック エンドポイント |
| ファイアウォール規則：Azure サービス | はい |
| ファイアウォール規則：現在のクライアント | いいえ |

入力が完了したら、「次：追加機能」ボタンを押し、次のページに進みます。

<div align="center">
    <img src="imgs/05-fill-create-sql-database-additional-settings-form.png" width="60%">
</div>

追加機能画面では、「Azure Defender for SQL の有効化」を「後で」に変更し、「確認および作成」ボタンを押します。確認の結果、エラーが発生しなければ、「作成」ボタンを押して、SQL Database の作成を行います。

| パラメータ名 | 値 |
| ---------- | -- |
| 既存データの使用 | なし |
| 照合順序（変更なし） | SQL_Latin1_General_CP1_CI_AS |
| Azure Defender for SQL | 後で |


### 1-2. Azure Cloud Shell で SQL データベースにテーブルを作成

ここでは、SQLデータベースにテーブルを作成します。テーブルの作成には、.NETに用意されているEntity Frameworkを利用して、ソースコードから自動的にテーブルを作成します。ここでの操作は、全てCloud Shell上で実行します。

まず、ToDoアプリのソースコードをCloud Shellにダウンロードします。

```
git clone https://github.com/uenohajime/dotnetcore-sqldb-tutorial.git
```

次にダウンロードしたプロジェクト・フォルダに移動します。

```
cd dotnetcore-sqldb-tutorial
```

ソースコードの書き換えのため、エディタを起動します。ここで変更する内容は、もともと、ローカルのSQLiteをデータベースとして利用するアプリをSQL Databaseをデータベースとして利用するように書き換えます。

```
code .
```

Startup.cs ファイルを開き 31 - 32行目あたりにある次の行を、行の先頭に「//」を追記することでコメントアウトします。

```
services.AddDbContext<MyDatabaseContext>(options =>
    options.UseSqlite("Data Source=localdatabase.db"));
```

次にコメントアウトした行の次の行に、次の２行を追加します。

```
services.AddDbContext<MyDatabaseContext>(options =>
   options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
```

<div align="center">
    <img src="imgs/06-modify-db-settings-on-vscode.png" width="60%">
</div>

次に既存のSQLiteアクセス用のソースコードを削除します。

```
rm Migrations -r
```

そして、dotnetコマンドの Entity Framework を使って、SQL Database用のソースコードを生成します。

```
dotnet ef migrations add InitialCreate
```

SQL Database へアクセスするための接続文字列を取得します。

```
az sql db show-connection-string --client ado.net --server handson4<名字> --name coreDB
```

ここで取得できる接続文字列は、User IDとPasswordの値はマスキングされているので、先ほど、SQLサーバを作成時に入力したあたいに置き換えてください。取得ができたら、その接続文字列を環境変数に保存します。

```
export ConnectionStrings__MyDbConnection="Server=tcp:handson4<名字>.database.windows.net,1433;Database=coreDB;User ID=dbadmin;Password=P@ssw0rd;Encrypt=true;Connection Timeout=30;"
```

最後にEntity Frameworkのコマンドを実行して、ソースコードに記述されたテーブルの定義をSQL Databaseに反映します。

```
dotnet ef database update
```

正しくテーブルが作成できたかについては、Azureポータル上でSQLデータベースへ移動し、「クエリエディター（プレビュー）」を選択し、ログインをします。

この時、下の図のように「テーブル」の中に「dbo.Todo」と言う名前のテーブルが存在することが確認できると、正しくテーブルの作成が完了したことになります。

<div align="center">
    <img src="imgs/07-create-sql-server.png" width="60%">
</div>


## 2. App Service 作成と Web アプリ設置

### 2-1. App Service 作成

ここでは App Service を作成し、 Web アプリを設置します。Azure ポータルの左上にあるアイコンをクリックして、メニューを開き、「リソースの作成」ボタンをクリックします。

<div align="center">
    <img src="imgs/01-choose-create-resource-menu.png" width="60%">
</div>

次に「 Web アプリ」を選択します。

<div align="center">
    <img src="imgs/08-choose-web-app-menu.png" width="60%">
</div>

表示された画面で Web アプリに必要な項目を次のように入力します。その後、「確認および作成」ボタンを押し、問題が生じなければ、「作成」ボタンを押します。

<div align="center">
    <img src="imgs/09-fill-create-web-app-form.png" width="60%">
</div>

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


### 2-2. App Service に Connection Strings を設定

「Web アプリ」の作成を完了すると、次に App Service から SQL データベースへ接続するためのアクセスキーを Connection Strings として設定します。

まず、Azureポータル上で作成したApp Serviceの画面を開き、左のメニューの「設定 > 構成」を選択します。次に表示された画面の下の方にある「接続文字列」の「新しい接続文字列」ボタンを押します。

そして、表示される画面上で次のように入力します。その後、「保存」ボタンを押し、追加した内容を保存します。

<div align="center">
    <img src="imgs/10-add-connection-string-on-web-app.png" width="60%">
</div>

| パラメータ名 | 値 |
| ---------- | -- |
| 名前 | MyDbConnection |
| 値 | Server=tcp:handson4<名字>.database.windows.net,1433;Database=coreDB;User ID=dbadmin;Password=P@ssw0rd;Encrypt=true;Connection Timeout=30; |
| 種類 | SQLAzure |


### 2-3. App Service にアプリを設置

次にソースコードを App Service にアップロードします。いくつかの方法が提供されていますが、今回は「ローカル Git 」と呼ばれる Git コマンドで App Service にファイルをアップロードする方法でアプリを設置します。

手順は、まず、１.ポータルの App Service を設定する画面で、ローカル Git 環境を設定し、２． Cloud Shell でソースコードをアップロードするという手順になります。

#### 2-3-1. Local Git の設定

まず、ポータルでの App Service にローカル Git を設定します。ポータルで作成した App Service の画面に移動し、左側のメニューn「デプロイメント > デプロイ センター」を選択します。表示されたメニューの中で、今回は「 Local Git 」を選択します。

<div align="center">
    <img src="imgs/11-create-local-git-repo-on-web-app.png" width="60%">
</div>

次にビルドする場所を選択します。今回は、ローカル Git にアップロードされた時にビルドするように、「 App Service のビルド サービス」を選択します。

<div align="center">
    <img src="imgs/12-create-local-git-repo-on-web-app-2.png" width="60%">
</div>

その後、表示される Git Clone URI がソースコードをアップロードする URL となるので、メモをしておきます。

<div align="center">
    <img src="imgs/13-create-local-git-repo-on-web-app-3.png" width="60%">
</div>


#### 2-3-2. アプリの設置

次に Cloud Shell でソースコードをアップロードします。まず、 Cloud Shell を立ち上げ、次のコマンドで、アプリのルートディレクトリに移動します。

```
cd ~/dotnetcore-sqldb-tutorial
```

次に、 Git コマンドを使って、ソースコードをアップロードするため、ユーザ情報を Cloud Shell に登録します。登録に必要な情報は、Eメールと名前です。
次のコマンドの中で "you@example.com" の部分をEメールに置き換え、 "Your Name" の部分を名前に置き換えます。

```
git config --global user.email “you@example.com” && git config --global user.name “Your Name”
```

次に、先程変更したソースコードを記録するため、次のコマンドを実行します。このコマンドは、 "git add ." の部分は変更を登録するファイルを選択するコマンドとなり、 "." は変更された全てのファイルを対象にします。
さらに、その変更をコミットするコマンドが、 "git commit -m ..." の部分になります。ここで、 "-m" 以降はコミットと一緒に登録するコメントです。

```
git add . && git commit –m “connect to SQLDB in Azure”
```

次に、 App Service へソースコードをアップロードするためのユーザを作成します。次のコマンドは、Git コマンド用のユーザ名とパスワードを作成するコマンドです。

```
az webapp deployment user set --user-name handson4<名字> --password P@ssw0rd
```

次に、先程メモした URL をアップロード先として次のコマンドで登録します。

```
git remote add azure https://handson4<名字>.scm.azurewebsites.net:443/handson4<名字>.git
```

最後に次のコマンドで、ソースコードをアップロードします。

```
git push azure master
```

アップロード後、ブラウザで App Service の URL にアクセスし、次のような画面が表示されることを確認します。

<div align="center">
    <img src="imgs/14-check-deployed-web-app.png" width="60%">
</div>

また、実際に ToDo を登録できることを確認します。

<div align="center">
    <img src="imgs/15-todo-app-ui.png" width="60%">
</div>

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

## 変更履歴

* 2020/12/18 CloudShellにインストールされているdotnetコマンドのバージョンが3.1となったため、それに伴い内容を変更
