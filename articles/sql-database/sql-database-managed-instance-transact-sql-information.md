---
title: Azure SQL Database マネージド インスタンスの T-SQL の違い | Microsoft Docs
description: この記事では、Azure SQL Database Managed Instance と SQL Server の T-SQL の相違点について説明します
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, carlrab, bonova
ms.date: 08/12/2019
ms.custom: seoapril2019
ms.openlocfilehash: cad04df9ba76ce483a308411949e6f98bab23bf9
ms.sourcegitcommit: 65131f6188a02efe1704d92f0fd473b21c760d08
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2019
ms.locfileid: "70858546"
---
# <a name="managed-instance-t-sql-differences-limitations-and-known-issues"></a>マネージド インスタンスの T-SQL の相違点、制限、既知の問題

この記事では、Azure SQL Database マネージド インスタンスとオンプレミスの SQL Server データベース エンジンの構文および動作の相違点について簡単に説明します。 マネージド インスタンスのデプロイ オプションは、オンプレミスの SQL Server データベース エンジンとの高い互換性を備えています。 SQL Server データベース エンジンのほとんどの機能がマネージド インスタンスでサポートされています。

![移行](./media/sql-database-managed-instance/migration.png)

マネージド インスタンスには、SQL Server と比べて PaaS の制限と動作の違いがいくつかあります。 それらの相違点は、次のカテゴリに分けることができます。<a name="Differences"></a>

- [可用性](#availability)には、[Always On](#always-on-availability) と[バックアップ](#backup)の相違点が含まれます。
- [セキュリティ](#security)には、[監査](#auditing)、[証明書](#certificates)、[資格情報](#credential)、[暗号化プロバイダー](#cryptographic-providers)、[ログイン/ユーザー](#logins-and-users)、[サービス キーとサービス マスター キー](#service-key-and-service-master-key)の相違点が含まれます。
- [構成](#configuration)には、[バッファー プール拡張機能](#buffer-pool-extension)、[照合順序](#collation)、[互換性レベル](#compatibility-levels)、[データベース ミラーリング](#database-mirroring)、[データベース オプション](#database-options)、[SQL Server エージェント](#sql-server-agent)、[テーブル オプション](#tables)の相違点が含まれます。
- [機能](#functionalities)には、[BULK INSERT/OPENROWSET](#bulk-insert--openrowset)、[CLR](#clr)、[DBCC](#dbcc)、[分散トランザクション](#distributed-transactions)、[拡張イベント](#extended-events)、[外部ライブラリ](#external-libraries)、[filestream と FileTable](#filestream-and-filetable)、[フルテキスト セマンティック検索](#full-text-semantic-search)、[リンク サーバー](#linked-servers)、[PolyBase](#polybase)、[レプリケーション](#replication)、[RESTORE](#restore-statement)、[Service Broker](#service-broker)、[ストアド プロシージャ、関数、およびトリガー](#stored-procedures-functions-and-triggers)が含まれます。
- VNet やサブネット構成などの[環境設定](#Environment)。

これらの機能の大半はアーキテクチャ上の制約であり、サービスの機能を表します。

このページでは、マネージド インスタンスに見つかった[一時的な既知の問題](#Issues)についても説明しています。これらの問題は、将来解決される予定です。

## <a name="availability"></a>可用性

### <a name="always-on-availability"></a>Always On

[高可用性](sql-database-high-availability.md)はマネージド インスタンスに組み込まれており、ユーザーが制御することはできません。 次のステートメントはサポートされていません。

- [CREATE ENDPOINT … FOR DATABASE_MIRRORING](https://docs.microsoft.com/sql/t-sql/statements/create-endpoint-transact-sql)
- [CREATE AVAILABILITY GROUP](https://docs.microsoft.com/sql/t-sql/statements/create-availability-group-transact-sql)
- [ALTER AVAILABILITY GROUP](https://docs.microsoft.com/sql/t-sql/statements/alter-availability-group-transact-sql)
- [DROP AVAILABILITY GROUP](https://docs.microsoft.com/sql/t-sql/statements/drop-availability-group-transact-sql)
- [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql) ステートメントの [SET HADR](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-hadr) 句

### <a name="backup"></a>バックアップ

Managed Instance には自動バックアップがあるので、ユーザーは完全なデータベースの `COPY_ONLY` バックアップを作成できます。 差分、ログ、ファイル スナップショットの各バックアップはサポートされていません。

- Managed Instance では、Azure BLOB ストレージ アカウントにのみインスタンス データベースをバックアップできます。
  - `BACKUP TO URL` だけがサポートされています。
  - `FILE`、`TAPE`、およびバックアップ デバイスはサポートされていません。
- ほとんどの一般的な `WITH` オプションがサポートされています。
  - `COPY_ONLY` は必須です。
  - `FILE_SNAPSHOT` はサポートされていません。
  - テープ オプション: `REWIND`、`NOREWIND`、`UNLOAD`、`NOUNLOAD` はサポートされていません。
  - ログ固有のオプション: `NORECOVERY`、`STANDBY`、`NO_TRUNCATE` はサポートされていません。

制限事項: 

- Managed Instance では、最大 32 個のストライプを使用するバックアップにインスタンス データベースをバックアップできます。バックアップの圧縮を使用した場合、このバックアップで最大 4 TB のデータベースに十分対応できます。
- サービス管理 Transparent Data Encryption (TDE) を使用して暗号化されたデータベースでは、`BACKUP DATABASE ... WITH COPY_ONLY` は実行できません。 サービス管理 TDE では、バックアップを内部の TDE のキーで暗号化するように強制します。 キーはエクスポートできないので、バックアップを復元することはできません。 自動バックアップとポイントインタイム リストアを使用するか、代わりに[顧客管理 (BYOK) TDE](https://docs.microsoft.com/azure/sql-database/transparent-data-encryption-azure-sql#customer-managed-transparent-data-encryption---bring-your-own-key) を使用します。 また、データベースで暗号化を無効にすることができます。
- `BACKUP` コマンドを使用した場合の最大バックアップ ストライプ サイズは、最大 BLOB サイズである 195 GB です。 バックアップ コマンドのストライプ数を増やして、個々のストライプ サイズを小さくし、この制限を超えないようにします。

    > [!TIP]
    > 次の操作を実行すると、オンプレミス環境の SQL Server または仮想マシンからデータベースをバックアップするときに、この制限を回避できます。
    >
    > - `URL` にバックアップする代わりに `DISK` にバックアップします。
    > - バックアップ ファイルを BLOB ストレージにアップロードします。
    > - Managed Instance に復元します。
    >
    > Managed Instance で `Restore` コマンドを実行すると、アップロードされたバックアップ ファイルに対して異なる BLOB タイプが使用されるため、より大きな BLOB サイズがサポートされます。

T-SQL を使用したバックアップについては、[BACKUP](https://docs.microsoft.com/sql/t-sql/statements/backup-transact-sql) に関する記事をご覧ください。

## <a name="security"></a>セキュリティ

### <a name="auditing"></a>監査

Azure SQL Database のデータベースと SQL Server のデータベースにおける監査の主な相違点は、次のとおりです。

- Azure SQL Database でマネージド インスタンスのデプロイ オプションを使用すると、監査はサーバー レベルで機能します。 `.xel` ログ ファイルは、Azure BLOB ストレージに格納されます。
- Azure SQL Database で単一データベースとエラスティック プールのデプロイ オプションを使用すると、監査はデータベース レベルで機能します。
- オンプロミスまたは仮想マシンの SQL Server では、監査はサーバー レベルで機能します。 イベントは、ファイル システムまたは Windows イベント ログに格納されます。
 
マネージド インスタンスの XEvent 監査では、Azure Blob Storage のターゲットがサポートされます。 ファイル ログと Windows ログはサポートされていません。

Azure Blob Storage を監査するための `CREATE AUDIT` 構文の主な相違点は次のとおりです。

- `.xel` ファイルが配置されている Azure BLOB ストレージ コンテナーの URL の指定に使用できる新しい `TO URL` 構文が用意されています。
- Managed Instance は Windows ファイル共有にアクセスできないため、`TO FILE` 構文はサポートされていません。

詳細については、次を参照してください。 

- [CREATE SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/create-server-audit-transact-sql) 
- [ALTER SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/alter-server-audit-transact-sql)
- [監査](https://docs.microsoft.com/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

### <a name="certificates"></a>証明書

Managed Instance　はファイル共有と Windows フォルダーにはアクセスできないので、次の制約が適用されます。

- `CREATE FROM`/`BACKUP TO` ファイルは、証明書ではサポートされていません。
- `FILE`/`ASSEMBLY` からの `CREATE`/`BACKUP` 証明書はサポートされていません。 秘密キー ファイルは使用できません。 

[CREATE CERTIFICATE](https://docs.microsoft.com/sql/t-sql/statements/create-certificate-transact-sql) に関する記事、および [BACKUP CERTIFICATE](https://docs.microsoft.com/sql/t-sql/statements/backup-certificate-transact-sql) に関する記事をご覧ください。 
 
**対処法**: 証明書または秘密キーのスクリプトを作成し、.sql ファイルとして保存してバイナリから作成します。

```sql
CREATE CERTIFICATE  
   FROM BINARY = asn_encoded_certificate
WITH PRIVATE KEY (<private_key_options>)
```

### <a name="credential"></a>資格情報

Azure Key Vault と `SHARED ACCESS SIGNATURE` の ID だけがサポートされています。 Windows ユーザーはサポートされていません。

[CREATE CREDENTIAL](https://docs.microsoft.com/sql/t-sql/statements/create-credential-transact-sql) に関する記事、および [ALTER CREDENTIAL](https://docs.microsoft.com/sql/t-sql/statements/alter-credential-transact-sql) に関する記事をご覧ください。

### <a name="cryptographic-providers"></a>暗号化プロバイダー

Managed Instance はファイルにアクセスできないため、暗号化プロバイダーは作成できません。

- `CREATE CRYPTOGRAPHIC PROVIDER` はサポートされていません。 [CREATE CRYPTOGRAPHIC PROVIDER](https://docs.microsoft.com/sql/t-sql/statements/create-cryptographic-provider-transact-sql) に関する記事をご覧ください。
- `ALTER CRYPTOGRAPHIC PROVIDER` はサポートされていません。 [ALTER CRYPTOGRAPHIC PROVIDER](https://docs.microsoft.com/sql/t-sql/statements/alter-cryptographic-provider-transact-sql) に関する記事をご覧ください。

### <a name="logins-and-users"></a>ログインとユーザー

- `FROM CERTIFICATE`、`FROM ASYMMETRIC KEY`、`FROM SID` を使用して作成された SQL ログインはサポートされています。 [CREATE LOGIN](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql) に関する記事をご覧ください。
- [CREATE LOGIN](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) 構文または [CREATE USER FROM Login [Azure AD Login]](https://docs.microsoft.com/sql/t-sql/statements/create-user-transact-sql?view=azuresqldb-mi-current) 構文を使用して作成された Azure Active Directory (Azure AD) サーバー プリンシパル (ログイン) がサポートされます (パブリック プレビュー)。 これらのログインは、サーバー レベルで作成されます。

    マネージド インスタンスは、`CREATE USER [AADUser/AAD group] FROM EXTERNAL PROVIDER` 構文で Azure AD データベース プリンシパルをサポートします。 この機能は、Azure AD 包含データベース ユーザーとも呼ばれます。

- `CREATE LOGIN ... FROM WINDOWS` 構文を使用して作成された Windows ログインはサポートされていません。 Azure Active Directory のログインとユーザーを使用します。
- インスタンスを作成した Azure AD ユーザーは、[無制限の管理特権](sql-database-manage-logins.md#unrestricted-administrative-accounts)を持ちます。
- 管理者以外の、データベース レベルの Azure AD ユーザーは、`CREATE USER ... FROM EXTERNAL PROVIDER` 構文を使用して作成できます。 [CREATE USER ...FROM EXTERNAL PROVIDER](sql-database-manage-logins.md#non-administrator-users) を参照してください。
- Azure AD サーバー プリンシパル (ログイン) は、1 つの Managed Instance 内のみで SQL 機能をサポートします。 同じ Azure AD テナント内か、または異なるテナント内かに関係なく、Azure AD ユーザーの場合、クロス インスタンス対話を必要とする機能はサポートされていません。 そのような機能の例として次のものがあります。

  - SQL トランザクション レプリケーション。
  - リンク サーバー。

- Azure AD グループにマップされている Azure AD ログインをデータベース所有者として設定することはサポートされていません。
- [EXECUTE AS](/sql/t-sql/statements/execute-as-transact-sql) 句など、他の Azure AD プリンシパルを使用することによる Azure AD サーバー レベル プリンシパルの権限の借用はサポートされています。 EXECUTE AS の制限事項は次のとおりです。

  - 名前がログイン名と異なる場合、EXECUTE AS USER は Azure AD ユーザーに対してサポートされません。 たとえば、CREATE USER [myAadUser] FROM LOGIN [john@contoso.com] 構文を使用してユーザーが作成されて、EXEC AS USER = _myAadUser_ を使用して権限借用が試行された場合などです。 Azure AD サーバー プリンシパル (ログイン) から **USER** を作成するときは、user_name を **LOGIN** と同じ login_name として指定します。
  - Azure AD プリンシパルを対象とした次の操作を実行できるのは、`sysadmin` ロールに属している SQL Server レベル プリンシパル (ログイン) のみです。

    - EXECUTE AS USER
    - EXECUTE AS LOGIN

- Azure AD サーバー プリンシパル (ログイン) のパブリック プレビュー制限事項:

  - マネージド インスタンスに対する Active Directory 管理者の制限事項:

    - Managed Instance の設定に使用された Azure AD 管理者は、Managed Instance 内での Azure AD サーバー プリンシパル (ログイン) の作成には使用できません。 `sysadmin` ロールである SQL Server アカウントを使用することにより、最初の Azure AD サーバー プリンシパル (ログイン) を作成する必要があります。 この一時的な制限事項は、Azure AD サーバー プリンシパル (ログイン) が一般提供されると削除されます。 Azure AD 管理者アカウントを使用してログインを作成しようとした場合、次のエラーが表示されます。`Msg 15247, Level 16, State 1, Line 1 User does not have permission to perform this action.`
      - 現時点では、マスター データベースに作成される最初の Azure AD ログインは、[CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) FROM EXTERNAL PROVIDER を使用することにより、`sysadmin` である標準の SQL Server アカウント (Azure AD ではない) が作成する必要があります。 一般提供後、この制限事項は削除されます。 続いて、マネージド インスタンスの Active Directory 管理者によって初期 Azure AD ログインを作成できます。
    - SQL Server Management Studio または SqlPackage で使用される DacFx (エクスポート/インポート) は、Azure AD ログインではサポートされていません。 この制限事項は、Azure AD サーバー プリンシパル (ログイン) が一般提供されると削除されます。
    - SQL Server Management Studio での Azure AD サーバー プリンシパル (ログイン) の使用:

      - 任意の認証ログインを使用した Azure AD ログインのスクリプトはサポートされていません。
      - IntelliSense は、CREATE LOGIN FROM EXTERNAL PROVIDER ステートメントを認識せず、赤の下線を表示します。

- マネージド インスタンスのプロビジョニング プロセスによって作成されるサーバーレベル プリンシパル ログイン、`securityadmin` や `sysadmin` などのサーバー ロールのメンバー、あるいはサーバー レベルの ALTER ANY LOGIN アクセス許可を持つその他のログインのみが、マネージド インスタンスのマスター データベースに Azure AD サーバー プリンシパル (ログイン) を作成できます。
- ログインが SQL プリンシパルの場合、作成コマンドを使用して Azure AD アカウントのログインを作成できるのは、`sysadmin` ロールに属しているログインのみです。
- Azure AD ログインは、Azure SQL Database マネージド インスタンスに使用されるものと同じディレクトリ内の Azure AD のメンバーでなければなりません。
- Azure AD サーバー プリンシパル (ログイン) は、SQL Server Management Studio 18.0 プレビュー 5 から、オブジェクト エクスプローラーに表示されます。
- Azure AD サーバー プリンシパル (ログイン) と Azure AD 管理者アカウントとの重複は許可されます。 プリンシパルを解決してアクセス許可を Managed Instance に適用するとき、Azure AD の管理者よりも Azure AD サーバー プリンシパル (ログイン) が優先されます。
- 認証時、認証するプリンシパルを解決するために次の順序が適用されます。

    1. Azure AD アカウントが、Azure AD サーバー プリンシパル (ログイン) に直接マップされているものとして存在する (sys.server_principals に type "E" と指定されている) 場合、アクセスを許可し、Azure AD サーバー プリンシパル (ログイン) のアクセス権を適用する。
    2. Azure AD アカウントが、Azure AD サーバー プリンシパル (ログイン) にマップされている Azure AD グループのメンバーである (sys.server_principals に type "X" と指定されている) 場合、アクセスを許可し、Azure AD グループ ログインのアクセス権を適用する。
    3. Azure AD アカウントが マネージド インスタンス用のポータル構成の特殊な Azure AD 管理者である (マネージド インスタンス システム ビューに存在しない) 場合は、マネージド インスタンスの Azure AD 管理者の特殊な固定アクセス許可を適用する (レガシー モード)。
    4. Azure AD アカウントが、データベース内の Azure AD ユーザーに直接マップされたものとして存在する (sys.database_principals に type "E" と指定されている) 場合、アクセスを許可し、Azure AD データベース ユーザーのアクセス許可を適用する。
    5. Azure AD アカウントが、データベース内の Azure AD ユーザーにマップされた Azure AD グループのメンバーである (sys.database_principals に type "X" と指定されている) 場合、アクセスを許可し、Azure AD グループ ログインのアクセス許可を適用する。
    6. Azure AD ユーザー アカウントまたは Azure AD グループ アカウントのいずれかにマップされている Azure AD ログインがあり、認証しているユーザーを解決する場合、この Azure AD ログインのすべてのアクセス許可が適用される。

### <a name="service-key-and-service-master-key"></a>サービス キーとサービス マスター キー

- [マスター キーのバックアップ](https://docs.microsoft.com/sql/t-sql/statements/backup-master-key-transact-sql)はサポートされていません (SQL Database サービスによって管理されます)。
- [マスター キーの復元](https://docs.microsoft.com/sql/t-sql/statements/restore-master-key-transact-sql)はサポートされていません (SQL Database サービスによって管理されます)。
- [サービス マスター キーのバックアップ](https://docs.microsoft.com/sql/t-sql/statements/backup-service-master-key-transact-sql)はサポートされていません (SQL Database サービスによって管理されます)。
- [サービス マスター キーの復元](https://docs.microsoft.com/sql/t-sql/statements/restore-service-master-key-transact-sql)はサポートされていません (SQL Database サービスによって管理されます)。

## <a name="configuration"></a>構成

### <a name="buffer-pool-extension"></a>バッファー プール拡張機能

- [バッファー プール拡張機能](https://docs.microsoft.com/sql/database-engine/configure-windows/buffer-pool-extension)はサポートされていません。
- `ALTER SERVER CONFIGURATION SET BUFFER POOL EXTENSION` はサポートされていません。 [ALTER SERVER CONFIGURATION](https://docs.microsoft.com/sql/t-sql/statements/alter-server-configuration-transact-sql) に関する記事をご覧ください。

### <a name="collation"></a>Collation

既定のインスタンスの照合順序は `SQL_Latin1_General_CP1_CI_AS` であり、作成パラメーターとして指定できます。 「[照合順序](https://docs.microsoft.com/sql/t-sql/statements/collations)」をご覧ください。

### <a name="compatibility-levels"></a>互換性レベル

- サポートされている互換性レベルは、100、110、120、130、140、150 です。
- 100 より低い互換性レベルはサポートされていません。
- 新しいデータベースの既定の互換性レベルは 140 です。 互換性レベルが 100 以上のデータベースが復元された場合、互換性レベルは変更されません。

[ALTER DATABASE の互換性レベル](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level)に関する記事をご覧ください。

### <a name="database-mirroring"></a>データベース ミラーリング

データベース ミラーリングはサポートされていません。

- `ALTER DATABASE SET PARTNER` オプションと `SET WITNESS` オプションはサポートされていません。
- `CREATE ENDPOINT … FOR DATABASE_MIRRORING` はサポートされていません。

詳細については、[ALTER DATABASE の SET PARTNER と SET WITNESS](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-database-mirroring)、および [CREATE ENDPOINT … FOR DATABASE_MIRRORING](https://docs.microsoft.com/sql/t-sql/statements/create-endpoint-transact-sql) の説明をご覧ください。

### <a name="database-options"></a>データベース オプション

- 複数のログ ファイルはサポートされていません。
- インメモリ オブジェクトは、General Purpose サービス レベルではサポートされていません。 
- General Purpose インスタンスあたり 280 ファイル (データベースあたり最大 280 ファイル) の制限があります。 General Purpose レベルのデータ ファイルとログ ファイルの両方がこの制限にカウントされます。 [Business Critical レベルでは、データベースあたり 32,767 ファイルがサポートされます](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-resource-limits#service-tier-characteristics)。
- filestream データを含むファイル グループをデータベースに含めることはできません。 .bak に `FILESTREAM` データが含まれていると、復元は失敗します。 
- すべてのファイルが Azure Blob Storage に配置されます。 ファイルあたりの IO およびスループットは、個々のファイルのサイズによって異なります。

#### <a name="create-database-statement"></a>CREATE DATABASE ステートメント

次の制限事項が `CREATE DATABASE` に適用されます。

- ファイルおよびファイル グループは定義できません。 
- `CONTAINMENT` オプションはサポートされていません。 
- `WITH` オプションはサポートされていません。 
   > [!TIP]
   > 対処法として、`CREATE DATABASE` の後に `ALTER DATABASE` を使用して、ファイルの追加またはコンテインメントの設定を行うデータベース オプションを設定します。 

- `FOR ATTACH` オプションはサポートされていません。
- `AS SNAPSHOT OF` オプションはサポートされていません。

詳細については、[CREATE DATABASE](https://docs.microsoft.com/sql/t-sql/statements/create-database-sql-server-transact-sql) に関する記事をご覧ください。

#### <a name="alter-database-statement"></a>ALTER DATABASE ステートメント

ファイルの一部のプロパティは設定または変更できません。

- `ALTER DATABASE ADD FILE (FILENAME='path')` T-SQL ステートメントでは、ファイル パスは指定できません。 ファイルはマネージド インスタンスによって自動的に配置されるため、スクリプトから `FILENAME` を削除します。 
- `ALTER DATABASE` ステートメントを使用してファイル名を変更することはできません。

次のオプションは既定で設定されており、変更することはできません。

- `MULTI_USER`
- `ENABLE_BROKER ON`
- `AUTO_CLOSE OFF`

次のオプションは変更できません。

- `AUTO_CLOSE`
- `AUTOMATIC_TUNING(CREATE_INDEX=ON|OFF)`
- `AUTOMATIC_TUNING(DROP_INDEX=ON|OFF)`
- `DISABLE_BROKER`
- `EMERGENCY`
- `ENABLE_BROKER`
- `FILESTREAM`
- `HADR`
- `NEW_BROKER`
- `OFFLINE`
- `PAGE_VERIFY`
- `PARTNER`
- `READ_ONLY`
- `RECOVERY BULK_LOGGED`
- `RECOVERY_SIMPLE`
- `REMOTE_DATA_ARCHIVE` 
- `RESTRICTED_USER`
- `SINGLE_USER`
- `WITNESS`

詳細については、[ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-file-and-filegroup-options) に関する記事をご覧ください。

### <a name="sql-server-agent"></a>SQL Server エージェント

- Managed Instance では現在、SQL Server エージェントの有効化/無効化はサポートされていません。 SQL エージェントは常時稼動状態となります。
- SQL Server エージェントの設定は読み取り専用です。 `sp_set_agent_properties` プロシージャは、マネージド インスタンスではサポートされていません。 
- [ジョブ]
  - T-SQL ジョブ ステップがサポートされています。
  - 次のレプリケーション ジョブがサポートされています。
    - トランザクション ログ リーダー
    - スナップショット
    - ディストリビューター
  - SSIS ジョブ ステップがサポートされています。
  - 他の種類のジョブ ステップは現在サポートされていません。
    - マージ レプリケーション ジョブ ステップはサポートされていません。 
    - キュー リーダーはサポートされていません。 
    - コマンド シェルはまだサポートされていません。
  - Managed Instance は、外部リソース (例: robocopy を介したネットワーク共有) にはアクセスできません。 
  - SQL Server Analysis Services はサポートされていません。
- 通知は部分的にサポートされています。
- 電子メール通知がサポートされていますが、データベース メール プロファイルを構成する必要があります。 SQL Server エージェントで使用できるデータベース メール プロファイルは 1 つだけであり、`AzureManagedInstance_dbmail_profile` という名前である必要があります。 
  - Pager はサポートされていません。
  - NetSend はサポートされていません。
  - アラートはまだサポートされていません。
  - プロキシはサポートされていません。
- EventLog はサポートされていません。

現在、次の SQL エージェント機能はサポートされていません。

- プロキシ
- アイドル状態の CPU でのジョブのスケジューリング
- エージェントの有効化または無効化
- アラート

SQL Server エージェントについては、「[SQL Server エージェント](https://docs.microsoft.com/sql/ssms/agent/sql-server-agent)」をご覧ください。

### <a name="tables"></a>テーブル

次のテーブルの種類はサポートされていません。

- [FILESTREAM](https://docs.microsoft.com/sql/relational-databases/blob/filestream-sql-server)
- [FILETABLE](https://docs.microsoft.com/sql/relational-databases/blob/filetables-sql-server)
- [EXTERNAL TABLE](https://docs.microsoft.com/sql/t-sql/statements/create-external-table-transact-sql) (Polybase)
- [MEMORY_OPTIMIZED](https://docs.microsoft.com/sql/relational-databases/in-memory-oltp/introduction-to-memory-optimized-tables) (General Purpose レベルでのみサポートされません)

テーブルの作成および変更方法については、[CREATE TABLE](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql) に関する記事および [ALTER TABLE](https://docs.microsoft.com/sql/t-sql/statements/alter-table-transact-sql) に関する記事をご覧ください。

## <a name="functionalities"></a>機能

### <a name="bulk-insert--openrowset"></a>一括挿入/OPENROWSET

Managed Instance はファイル共有や Windows フォルダーにアクセスできないため、ファイルは Azure BLOB ストレージからインポートする必要があります。

- が、Azure BLOB ストレージからファイルをインポートしている間、`BULK INSERT` コマンドで `DATASOURCE` が必要です。 [BULK INSERT](https://docs.microsoft.com/sql/t-sql/statements/bulk-insert-transact-sql) に関する記事をご覧ください。
- Azure BLOB ストレージからのファイルの内容を読み取る場合、`OPENROWSET` 関数で `DATASOURCE` が必要です。 [OPENROWSET](https://docs.microsoft.com/sql/t-sql/functions/openrowset-transact-sql) に関する記事をご覧ください。

### <a name="clr"></a>CLR

Managed Instance　はファイル共有と Windows フォルダーにはアクセスできないので、次の制約が適用されます。

- `CREATE ASSEMBLY FROM BINARY` だけがサポートされています。 [CREATE ASSEMBLY FROM BINARY](https://docs.microsoft.com/sql/t-sql/statements/create-assembly-transact-sql) に関する記事をご覧ください。 
- `CREATE ASSEMBLY FROM FILE` はサポートされていません。 [CREATE ASSEMBLY FROM FILE](https://docs.microsoft.com/sql/t-sql/statements/create-assembly-transact-sql) に関する記事をご覧ください。
- `ALTER ASSEMBLY` ではファイルを参照できません。 [ALTER ASSEMBLY](https://docs.microsoft.com/sql/t-sql/statements/alter-assembly-transact-sql) に関する記事をご覧ください。

### <a name="database-mail-db_mail"></a>データベース メール (db_mail)
 - `sp_send_dbmail` では、@file_attachments パラメーターを使用して添付ファイルを送信できません。 この手順では、ローカル ファイル システムと外部共有または Azure Blob ストレージにはアクセスできません。
 - `@query` パラメーターと認証に関連する既知の問題をご覧ください。
 
### <a name="dbcc"></a>DBCC

SQL Server で有効になっている、ドキュメントに記載されていない DBCC ステートメントは、Managed Instance ではサポートされていません。

- サポートされるグローバル `Trace flags` の数が限られています。 セッションレベルの `Trace flags` はサポートされません。 [トレース フラグ](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql)に関する記事をご覧ください。
- [DBCC TRACEOFF](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-traceoff-transact-sql) と [DBCC TRACEON](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-traceon-transact-sql) で使用できるトレースフラグの数が限られています。
- [DBCC CHECKDB](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql) とオプション REPAIR_ALLOW_DATA_LOSS、REPAIR_FAST、REPAIR_REBUILD の組み合わせは使用できません。これは、データベースを `SINGLE_USER` モードで設定することはできないためです。[ALTER DATABASE の相違点](#alter-database-statement)を参照してください。 データベースが破損した場合は Azure サポート チームが対応します。 データベースに修正すべき破損が見つかった場合は、Azure サポートにお問い合わせください。

### <a name="distributed-transactions"></a>分散トランザクション

MSDTC も[エラスティック トランザクション](sql-database-elastic-transactions-overview.md)も現在、Managed Instance ではサポートされていません。

### <a name="extended-events"></a>拡張イベント

拡張イベント (XEvents) の一部の Windows 固有のターゲットはサポートされていません。

- `etw_classic_sync` ターゲットはサポートされていません。 Azure BLOB ストレージに `.xel` ファイルを保存します。 「[etw_classic_sync ターゲット](https://docs.microsoft.com/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#etw_classic_sync_target-target)」をご覧ください。
- `event_file` ターゲットはサポートされていません。 Azure BLOB ストレージに `.xel` ファイルを保存します。 「[event_file ターゲット](https://docs.microsoft.com/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#event_file-target)」をご覧ください。

### <a name="external-libraries"></a>外部ライブラリ

In-Database R および In-Database Python 外部ライブラリはまだサポートされていません。 「[SQL Server Machine Learning Services](https://docs.microsoft.com/sql/advanced-analytics/r/sql-server-r-services)」をご覧ください。

### <a name="filestream-and-filetable"></a>Filestream と FileTable

- filestream データはサポートされていません。
- `FILESTREAM` データを含むファイル グループをデータベースに含めることはできません。
- `FILETABLE` はサポートされていません。
- テーブルに `FILESTREAM` 型を含めることはできません。
- 次の関数はサポートされていません。
  - `GetPathLocator()`
  - `GET_FILESTREAM_TRANSACTION_CONTEXT()`
  - `PathName()`
  - `GetFileNamespacePat)`
  - `FileTableRootPath()`

詳細については、[FILESTREAM](https://docs.microsoft.com/sql/relational-databases/blob/filestream-sql-server) に関する記事、および [FileTables](https://docs.microsoft.com/sql/relational-databases/blob/filetables-sql-server) に関する記事をご覧ください。

### <a name="full-text-semantic-search"></a>フルテキスト セマンティック検索

[セマンティック検索](https://docs.microsoft.com/sql/relational-databases/search/semantic-search-sql-server)はサポートされていません。

### <a name="linked-servers"></a>リンク サーバー

マネージド インスタンスのリンク サーバーがサポートするターゲットの数は限られています。

- サポートされているターゲットは、Managed Instance、Single Database、および SQL Server インスタンスです。 
- リンク サーバーは、分散型の書き込み可能なトランザクション (MS DTC) をサポートしていません。
- サポートされていないターゲットは、ファイル、Analysis Services、他の RDBMS です。 ファイルのインポートの代わりに、`BULK INSERT` または `OPENROWSET` を使用して、Azure Blob Storage からネイティブ CSV インポートを使用してください。

Operations

- クロス インスタンス書き込みトランザクションはサポートされていません。
- リンク サーバーの削除で `sp_dropserver` がサポートされています。 [sp_dropserver](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-dropserver-transact-sql) に関する記事をご覧ください。
- SQL Server インスタンスでのみ、`OPENROWSET` 関数を使用してクエリを実行できます。 これらは、マネージド、オンプレミス、仮想マシンのいずれかで配置できます。 [OPENROWSET](https://docs.microsoft.com/sql/t-sql/functions/openrowset-transact-sql) に関する記事をご覧ください。
- SQL Server インスタンスでのみ、`OPENDATASOURCE` 関数を使用してクエリを実行できます。 これらは、マネージド、オンプレミス、仮想マシンのいずれかで配置できます。 プロバイダーとしてサポートされる値は、`SQLNCLI`、`SQLNCLI11`、`SQLOLEDB` だけです。 例: `SELECT * FROM OPENDATASOURCE('SQLNCLI', '...').AdventureWorks2012.HumanResources.Employee`。 [OPENDATASOURCE](https://docs.microsoft.com/sql/t-sql/functions/opendatasource-transact-sql) に関する記事をご覧ください。
- リンク サーバーを使用してネットワーク共有からファイル (Excel、CSV) を読み取ることはできません。 Azure Blob Storage から CSV ファイルを読み取る [BULK INSERT](https://docs.microsoft.com/sql/t-sql/statements/bulk-insert-transact-sql#e-importing-data-from-a-csv-file) または [OPENROWSET](https://docs.microsoft.com/sql/t-sql/functions/openrowset-transact-sql#g-accessing-data-from-a-csv-file-with-a-format-file) を使用してください。 この要求は、[マネージド インスタンス フィードバック項目](https://feedback.azure.com/forums/915676-sql-managed-instance/suggestions/35657887-linked-server-to-non-sql-sources)|で追跡します

### <a name="polybase"></a>PolyBase

HDFS または Azure BLOB ストレージ内のファイルを参照する外部テーブルはサポートされていません。 PolyBase については、[PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) に関する記事をご覧ください。

### <a name="replication"></a>レプリケーション

- スナップショットおよび双方向のレプリケーションの種類がサポートされています。 マージ レプリケーション、ピア ツー ピア レプリケーション、および更新可能サブスクリプションはサポートされていません。
- [トランザクション レプリケーション](sql-database-managed-instance-transactional-replication.md)はマネージド インスタンスのパブリック プレビューで使用できますが、制約がいくつかあります。
    - すべての種類のレプリケーション参加者 (パブリッシャー、ディストリビューター、プル サブスクライバー、プッシュ サブスクライバー) をマネージド インスタンスに配置できますが、パブリッシャーとディストリビューターを異なるインスタンスに配置することはできません。
    - マネージド インスタンスは、最新バージョンの SQL Server と通信できます。 サポートされているバージョンについては、[こちら](sql-database-managed-instance-transactional-replication.md#supportability-matrix-for-instance-databases-and-on-premises-systems)をご覧ください。
    - トランザクション レプリケーションには、いくつかの[追加のネットワーク要件](sql-database-managed-instance-transactional-replication.md#requirements)があります。

レプリケーションの構成の詳細については、[レプリケーションに関するチュートリアル](replication-with-sql-database-managed-instance.md)を参照してください。


[フェールオーバー グループ](sql-database-auto-failover-group.md)内のデータベースでレプリケーションが有効な場合、マネージド インスタンス管理者は、フェールオーバーが発生した後で、古いプライマリ上のすべてのパブリケーションをクリーンアップしてから、新しいプライマリ上でそれらを再構成する必要があります。 このシナリオでは、次のアクティビティが必要です。

1. データベース上で実行されているレプリケーション ジョブがある場合は、すべて停止します。
2. パブリッシャーからサブスクリプションのメタデータを削除するには、パブリッシャー データベース上で次のスクリプトを実行します。

   ```sql
   EXEC sp_dropsubscription @publication='<name of publication>', @article='all',@subscriber='<name of subscriber>'
   ```             
 
1. サブスクライバーからサブスクリプションのメタデータを削除します。 サブスクライバー インスタンス上のサブスクリプション データベースで、次のスクリプトを実行します。

   ```sql
   EXEC sp_subscription_cleanup
      @publisher = N'<full DNS of publisher, e.g. example.ac2d23028af5.database.windows.net>', 
      @publisher_db = N'<publisher database>', 
      @publication = N'<name of publication>'; 
   ```                

1. パブリッシャーからすべてのレプリケーション オブジェクトを強制的に削除するには、発行されたデータベースで次のスクリプトを実行します。

   ```sql
   EXEC sp_removedbreplication
   ```

1. 元のプライマリ インスタンスから以前のディストリビューターを強制的に削除します (ディストリビューターを持つために使用されていた以前のプライマリにフェールバックする場合)。 以前のディストリビューター マネージド インスタンスのマスター データベース上で次のスクリプトを実行します。

   ```sql
   EXEC sp_dropdistributor 1,1
   ```

### <a name="restore-statement"></a>RESTORE ステートメント 

- サポートされている構文:
  - `RESTORE DATABASE`
  - `RESTORE FILELISTONLY ONLY`
  - `RESTORE HEADER ONLY`
  - `RESTORE LABELONLY ONLY`
  - `RESTORE VERIFYONLY ONLY`
- サポートされていない構文:
  - `RESTORE LOG ONLY`
  - `RESTORE REWINDONLY ONLY`
- ソース: 
  - `FROM URL` (Azure BLOB ストレージ) が、サポートされている唯一のオプションです。
  - `FROM DISK`/`TAPE`/バックアップ デバイスはサポートされていません。
  - バックアップ セットはサポートされていません。
- `WITH` オプションはサポートされていません (`DIFFERENTIAL`、`STATS` などは使用不可)。
- `ASYNC RESTORE`:クライアント接続が切断されても、復元は続行されます。 接続が破棄された場合は、`sys.dm_operation_status` ビューで復元操作とデータベースの CREATE 操作および DROP 操作の状態を確認できます。 [sys.dm_operation_status](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database) に関する記事をご覧ください。 

次のデータベース オプションを設定またはオーバーライドします。これらを後で変更することはできません。 

- `NEW_BROKER` (.bak ファイルでブローカーが有効になっていない場合)。 
- `ENABLE_BROKER` (.bak ファイルでブローカーが有効になっていない場合)。 
- `AUTO_CLOSE=OFF` (.bak ファイル内のデータベースに `AUTO_CLOSE=ON` が設定されている場合)。 
- `RECOVERY FULL` (.bak ファイル内のデータベースが `SIMPLE` または `BULK_LOGGED` 回復モードの場合)。
- メモリ最適化ファイル グループが追加され、(ソースの .bak ファイルに含まれていなかった場合) XTP という名前が付けられます。 
- 既存のメモリ最適化ファイル グループの名前が XTP に変更されます。 
- `SINGLE_USER` および `RESTRICTED_USER` のオプションは、`MULTI_USER` に変換されます。

制限事項: 

- 破損したデータベースのバックアップは、破損の種類によっては復元される可能性がありますが、破損が修正されるまで自動バックアップは実行されません。 この問題を回避するには、必ずソース インスタンス上で `DBCC CHECKDB` を実行し、バックアップ `WITH CHECKSUM` を使用します。
- このドキュメントで説明されている制限 (たとえば `FILESTREAM` または `FILETABLE` オブジェクト) を含むデータベースの `.BAK` ファイルの復元は、Managed Instance では復元できません。
- `.BAK` ファイルに複数のバックアップ セットが含まれている場合、復元できません。 
- `.BAK` ファイルに複数のログ ファイルが含まれている場合、復元できません。
- 8 TB を超えるデータベース、アクティブなインメモリ OLTP オブジェクト、または 280 個を超えるファイルを含むバックアップは、General Purpose インスタンスでは復元できません。 
- 4 TB を超えるデータベース、または[リソースの制限](sql-database-managed-instance-resource-limits.md)のページで説明されているサイズよりも合計サイズが大きいインメモリ OLTP オブジェクトを含むバックアップは、Business Critical インスタンスでは復元できません。
RESTORE ステートメントについては、[RESTORE ステートメント](https://docs.microsoft.com/sql/t-sql/statements/restore-statements-transact-sql)に関する記事をご覧ください。

### <a name="service-broker"></a>Service Broker

クロス インスタンス Service Broker はサポートされていません。

- `sys.routes`:前提条件として sys.routes からアドレスを選択する必要があります。 アドレスは、各ルートで LOCAL である必要があります。 [sys.routes](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-routes-transact-sql) に関する記事をご覧ください。
- `CREATE ROUTE`:`LOCAL` 以外の `ADDRESS` で `CREATE ROUTE` を使用することはできません。 [CREATE ROUTE](https://docs.microsoft.com/sql/t-sql/statements/create-route-transact-sql) に関する記事をご覧ください。
- `ALTER ROUTE`:`LOCAL` 以外の `ADDRESS` で `ALTER ROUTE` を使用することはできません。 [ALTER ROUTE](https://docs.microsoft.com/sql/t-sql/statements/alter-route-transact-sql) に関する記事をご覧ください。 

### <a name="stored-procedures-functions-and-triggers"></a>ストアド プロシージャ、関数、トリガー

- `NATIVE_COMPILATION` は、General Purpose レベルではサポートされていません。
- [sp_configure](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql) の次のオプションはサポートされていません。 
  - `allow polybase export`
  - `allow updates`
  - `filestream_access_level`
  - `remote data archive`
  - `remote proc trans`
- `sp_execute_external_scripts` はサポートされていません。 [sp_execute_external_scripts](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql#examples) に関するセクションをご覧ください。
- `xp_cmdshell` はサポートされていません。 [xp_cmdshell](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql) に関する記事をご覧ください。
- `Extended stored procedures` はサポートされておらず、これには `sp_addextendedproc`  および `sp_dropextendedproc` が含まれます。 [拡張ストアド プロシージャ](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/general-extended-stored-procedures-transact-sql)に関する記事をご覧ください。
- `sp_attach_db`、`sp_attach_single_file_db`、`sp_detach_db` はサポートされていません。 [sp_attach_db](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-attach-db-transact-sql)、[sp_attach_single_file_db](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-attach-single-file-db-transact-sql)、[sp_detach_db](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-detach-db-transact-sql) に関する各記事をご覧ください。

### <a name="system-functions-and-variables"></a>システム関数とシステム変数

次の変数、関数、ビューは異なる結果を返します。

- `SERVERPROPERTY('EngineEdition')` は値 8 を返します。 このプロパティは、マネージド インスタンスを一意に識別します。 [SERVERPROPERTY](https://docs.microsoft.com/sql/t-sql/functions/serverproperty-transact-sql) に関する記事をご覧ください。
- `SERVERPROPERTY('InstanceName')` は NULL を返します。これは SQL Server に関して存在するインスタンスの概念が、Managed Instance には該当しないためです。 [SERVERPROPERTY('InstanceName')](https://docs.microsoft.com/sql/t-sql/functions/serverproperty-transact-sql) に関する説明をご覧ください。
- `@@SERVERNAME` は、"接続可能な" 完全 DNS 名を返します (例: my-managed-instance.wcus17662feb9ce98.database.windows.net)。 [@@SERVERNAME](https://docs.microsoft.com/sql/t-sql/functions/servername-transact-sql) に関する記事をご覧ください。 
- `SYS.SERVERS` は、"name" プロパティと "data_source" プロパティを表す `myinstance.domain.database.windows.net` のような、"接続可能な" 完全 DNS 名を返します。 [SYS.SERVERS](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-servers-transact-sql) に関する記事をご覧ください。
- `@@SERVICENAME` は NULL を返します。これは SQL Server に関して存在するサービスの概念が、Managed Instance には該当しないためです。 [@@SERVICENAME](https://docs.microsoft.com/sql/t-sql/functions/servicename-transact-sql) に関する記事をご覧ください。
- `SUSER_ID` がサポートされています。 Azure AD ログインが sys.syslogins に含まれていない場合は、NULL を返します。 [SUSER_ID](https://docs.microsoft.com/sql/t-sql/functions/suser-id-transact-sql) に関する記事をご覧ください。 
- `SUSER_SID` はサポートされていません。 正しくないデータが返されます。これは既知の一時的な問題です。 [SUSER_SID](https://docs.microsoft.com/sql/t-sql/functions/suser-sid-transact-sql) に関する記事をご覧ください。 

## <a name="Environment"></a>環境の制約

### <a name="subnet"></a>Subnet
-  マネージド インスタンスをデプロイしたサブネットに他のリソース (たとえば仮想マシン) を配置することはできません。 別のサブネットを使用してこれらのリソースをデプロイしてください。
- サブネットには、十分な数の利用可能な [IP アドレス](sql-database-managed-instance-connectivity-architecture.md#network-requirements)が含まれている必要があります。 最小値は 16 ですが、サブネットに少なくとも 32 個の IP アドレスを含めることをお勧めします。
- [マネージド インスタンスのサブネットにサービス エンドポイントを関連付けることはできません](sql-database-managed-instance-connectivity-architecture.md#network-requirements)。 仮想ネットワークの作成時に、サービス エンドポイント オプションが無効になっていることを確認してください。
- リージョンでデプロイできるインスタンスの仮想コア数と種類には、いくつかの[制約と制限](sql-database-managed-instance-resource-limits.md#regional-resource-limitations)があります。
- [サブネットで適用される必要があるセキュリティ規則](sql-database-managed-instance-connectivity-architecture.md#network-requirements)があります。

### <a name="vnet"></a>VNet
- VNet はリソース モデルを使用してデプロイできます。VNet のクラシック モードはサポートされていません。
- マネージド インスタンスを作成した後は、マネージド インスタンスまたは VNet を別のリソース グループまたはサブスクリプションに移動することはできません。
- VNet が[グローバル ピアリング](../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers)を使用して接続されている場合、App Service Environment、ロジック アプリ、マネージド インスタンス (geo レプリケーション、トランザクション レプリケーション、またはリンクされたサーバー経由で使用) など、一部のサービスでは、さまざまなリージョンにあるマネージド インスタンスにアクセスできません。 ExpressRoute、または VNet ゲートウェイ経由の VNet 対 VNet を介してこのようなリソースに接続できます。

### <a name="tempdb"></a>TEMPDB

`tempdb` の最大ファイル サイズは、General Purpose レベルではコアあたり 24 GB より大きくすることはできません。 Business Critical レベルでは、`tempdb` の最大サイズはインスタンス ストレージ サイズによって制限されます。 `Tempdb` ログ ファイルのサイズは、General Purpose レベルでも Business Critical レベルでも 120 GB に制限されています。 `tempdb` のサイズがコアあたり 24 GB を超える場合、または 120 GB を超えるログ データが生成される場合は、一部のクエリでエラーが返されます。

### <a name="error-logs"></a>エラー ログ

マネージド インスタンスでは、エラー ログに詳細情報が書き込まれます。 エラー ログに記録される内部システム イベントが数多く存在します。 カスタムの手順を使用して、関連のない項目をフィルターで除外するエラー ログを読み取ります。 詳細については、[マネージド インスタンス - sp_readmierrorlog](https://blogs.msdn.microsoft.com/sqlcat/2018/05/04/azure-sql-db-managed-instance-sp_readmierrorlog/) に関する記事を参照してください。

## <a name="Issues"></a> 既知の問題

### <a name="resource-governor-on-business-critical-service-tier-might-need-to-be-reconfigured-after-failover"></a>Business Critical サービス レベルの Resource Governor をフェールオーバー後に再構成しなければならない場合がある

**日付:** 2019 年 9 月

ユーザー ワークロードに割り当てられているリソースを制限することを可能にする [Resource Governor](https://docs.microsoft.com/sql/relational-databases/resource-governor/resource-governor) 機能では、フェールオーバー後またはサービス レベルの変更 (たとえば、仮想コアやインスタンスの最大ストレージ サイズを変更します) をユーザーが開始した後、一部のユーザー ワークロードが間違って分類されることがあります。

**対処法**: [Resource Governor](https://docs.microsoft.com/sql/relational-databases/resource-governor/resource-governor) を使用している場合、`ALTER RESOURCE GOVERNOR RECONFIGURE` を定期的に、または、インスタンスの開始時に SQL タスクを実行する SQL エージェント ジョブの一環として実行します。

### <a name="cannot-authenicate-to-external-mail-servers-using-secure-connection-ssl"></a>セキュリティで保護された接続 (SSL) を使用して外部メール サーバーに対する認証ができない

**日付:** 2019 年 8 月

[セキュリティで保護された接続 (SSL) を使用して構成されている](https://docs.microsoft.com/sql/relational-databases/database-mail/configure-database-mail)データベース メールは、Azure の外部にある一部のメール サーバーでは認証できません。 これはセキュリティ構成の問題であり、まもなく解決される予定です。

**対処法:** この問題が解決されるまで、セキュリティで保護された接続 (SSL) をデータベース メールの構成から一時的に削除してください。 

### <a name="cross-database-service-broker-dialogs-must-be-re-initialized-after-service-tier-upgrade"></a>サービス レベルのアップグレード後は、複数データベースにまたがる Service Broker のダイアログを再初期化する必要があります。

**日付:** 2019 年 8 月

サービス レベルの変更操作の後、複数データベースにまたがる Service Broker のダイアログで、他のデータベースのサービスにメッセージが配信されなくなります。 メッセージは**失われたわけではなく**、センダーのキューに存在します。 マネージド インスタンスの仮想コアやインスタンスのストレージ サイズを変更すると、すべてのデータベースについて、[sys.databases](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-databases-transact-sql) ビューの `service_broke_guid` 値が変更されます。 [BEGIN DIALOG](https://docs.microsoft.com/en-us/sql/t-sql/statements/begin-dialog-conversation-transact-sql) ステートメントを使用して作成された、他のデータベースの Service Broker を参照する `DIALOG` から、ターゲット サービスにメッセージが配信されなくなります。

**対処法:** 複数データベースにまたがる Service Broker ダイアログのメッセージ交換を使用するアクティビティはすべて、サービス レベルを更新する前に停止し、後から再初期化してください。 サービス レベルの変更後、未配信のままのメッセージがあった場合は、それらのメッセージをソース キューから読み取って、ターゲット キューに再送信します。

### <a name="impersonification-of-aad-login-types-is-not-supported"></a>AAD ログイン タイプの偽装はサポートされない

**日付:** 2019 年 7 月

`EXECUTE AS USER` または `EXECUTE AS LOGIN` を使用した次の AAD プリンシパルの偽装はサポートされません。
-   別名が付けられた AAD ユーザー。 この場合は `15517` エラーが返されます。
- AAD アプリケーションまたはサービス プリンシパルに基づく AAD のログインおよびユーザー。 この場合は `15517` および `15406` エラーが返されます。

### <a name="query-parameter-not-supported-in-sp_send_db_mail"></a>sp_send_db_mail の @query パラメーターはサポートされない

**日付:** 2019 年 4 月

[sp_send_db_mail](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql) プロシージャの `@query` パラメーターは機能しません。

### <a name="transactional-replication-must-be-reconfigured-after-geo-failover"></a>geo フェールオーバー後、トランザクション レプリケーションを再構成する必要がある

**日付:** 2019 年 3 月

自動フェールオーバー グループ内のデータベースでトランザクション レプリケーションが有効な場合、マネージド インスタンス管理者は、別のリージョンへのフェールオーバーが発生した後で、古いプライマリ上のすべてのパブリケーションをクリーンアップしてから、新しいプライマリ上でそれらを再構成する必要があります。 詳細については、「[レプリケーション](#replication)」を参照してください。

### <a name="aad-logins-and-users-are-not-supported-in-tools"></a>ツール内では AAD のログインとユーザーがサポートされない

**日付:** 2019 年 1 月

SQL Server Management Studio および SQL Server Data Tools では、Azure Active Directory のログインとユーザーが完全にはサポートされません。
- 現在のところ、SQL Server Data Tools で Azure AD サーバー プリンシパル (ログイン) とユーザー (パブリック プレビュー) を使用することはできません。
- Azure AD サーバー プリンシパル (ログイン) とユーザー (パブリック プレビュー) のスクリプトは、SQL Server Management Studio ではサポートされていません。

### <a name="temporary-database-is-used-during-restore-operation"></a>一時的なデータベースが RESTORE 操作中に使用される

データベースがマネージド インスタンス上で復元されるとき、復元サービスではまず、所望の名前で空のデータベースが作成され、インスタンス上でその名前が割り当てられます。 しばらくすると、このデータベースは削除され、実際のデータベースの復元が開始されます。 *復元*状態のデータベースには、名前ではなくランダムな GUID 値が一時的に与えられます。 復元プロセスが完了すると、`RESTORE` ステートメントに指定されている所望の名前に一時的な名前が変更されます。 初期フェーズでは、ユーザーは空のデータベースにアクセスしたり、さらにはテーブルを作成したり、このデータベースにデータを読み込んだりできます。 この一時的なデータベースは、復元サービスで 2 つ目のフェーズが開始されると削除されます。

**対処法**: 復元の完了を確認するまで、復元中のデータベースにはアクセスしないでください。

### <a name="tempdb-structure-and-content-is-re-created"></a>TEMPDB の構造と内容は再作成される

`tempdb` は常に 12 個のデータ ファイルに分割され、ファイルの構造は変更できません。 ファイルあたりの最大サイズは変更できず、`tempdb` に新しいファイルを追加することはできません。 `Tempdb` は、インスタンスが開始またはフェールオーバーしたときに、常に空のデータベースとして再作成され、`tempdb` で行われたどの変更も保持されません。

### <a name="exceeding-storage-space-with-small-database-files"></a>小さなデータベース ファイルによる記憶域の超過

インスタンスが Azure ストレージの制限に達する可能性があるため、`CREATE DATABASE`、`ALTER DATABASE ADD FILE`、および `RESTORE DATABASE` ステートメントが失敗することがあります。

各 General Purpose Managed Instance には、Azure Premium ディスク領域用に予約された 最大 35 TB のストレージがあります。 各データベース ファイルは、個別の物理ディスクに配置されます。 ディスク サイズとして、128 GB、256 GB、512 GB、1 TB、4 TB のいずれかを指定できます。 ディスク上の未使用領域については請求されませんが、Azure Premium ディスクのサイズ合計が 35 TB を超えることはできません。 場合によっては、内部の断片化のために、合計で 8 TB を必要としない Managed Instance が、記憶域のサイズに関する 35 TB の Azure での制限を超える場合があります。

たとえば、General Purpose マネージド インスタンスに、4 TB のディスクにサイズが 1.2 TB の大きなファイルが 1 つ配置される場合があります。 また、248 ファイルの 1 GB のファイルがあり、それぞれが個別の 128 GB ディスクに配置される場合があります。 次の点に注意してください。

- 割り当てられるディスク ストレージの合計サイズは、1 x 4 TB + 248 x 128 GB = 35 TB となります。
- インスタンス上のデータベースの予約済み領域の合計は、1 x 1.2 TB + 248 x 1 GB = 1.4 TB となります。

この例は、特定の状況下では、特殊なファイルの配分のために、Managed Instance が、想定していなかったアタッチ済み Azure Premium ディスクに予約されている 35 TB の制限に到達する可能性があることを示しています。

この例では既存のデータベースは引き続き機能し、新しいファイルを追加しない限りは問題なく拡張できます。 すべてのデータベースの合計サイズがインスタンス サイズの上限に到達しない場合でも、新しいディスク ドライブ用の十分な領域がないため、新しいデータベースの作成や復元はできません。 その場合に返されるエラーは明確ではありません。

システム ビューを使用して、[残りのファイルの数を特定](https://medium.com/azure-sqldb-managed-instance/how-many-files-you-can-create-in-general-purpose-azure-sql-managed-instance-e1c7c32886c1)できます。 この制限に達した場合は、[DBCC SHRINKFILE ステートメントを使用して、より小さなファイルをいくつか空にして削除](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-shrinkfile-transact-sql#d-emptying-a-file)してみるか、[この制限のない Business Critical レベル](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-resource-limits#service-tier-characteristics)に切り替えてください。

### <a name="guid-values-shown-instead-of-database-names"></a>データベース名の代わりに GUID 値が表示される

複数のシステム ビュー、パフォーマンス カウンター、エラー メッセージ、XEvent、およびエラー ログ エントリで、実際のデータベース名ではなく GUID データベース識別子が表示されています。 将来、実際のデータベース名に置き換えられるため、これらの GUID 識別子には依存しないでください。

### <a name="error-logs-arent-persisted"></a>エラー ログが非永続的である

マネージド インスタンスで利用可能なエラー ログは永続的ではなく、このログのサイズは、最大ストレージ上限には含まれません。 フェールオーバーが発生した場合、エラー ログは自動的に消去される可能性があります。 マネージド インスタンスが複数の仮想マシンで複数回移動されたことが原因で、エラー ログの履歴に欠落部分が生じる可能性があります。

### <a name="transaction-scope-on-two-databases-within-the-same-instance-isnt-supported"></a>同じインスタンス内にある 2 つのデータベース上でトランザクション スコープがサポートされない

`TransactionScope` クラス (.NET) は、同じトランザクション スコープ下では、同一インスタンス内にある 2 つのデータベースに対して 2 つのクエリが送信された場合に機能しません。

```csharp
using (var scope = new TransactionScope())
{
    using (var conn1 = new SqlConnection("Server=quickstartbmi.neu15011648751ff.database.windows.net;Database=b;User ID=myuser;Password=mypassword;Encrypt=true"))
    {
        conn1.Open();
        SqlCommand cmd1 = conn1.CreateCommand();
        cmd1.CommandText = string.Format("insert into T1 values(1)");
        cmd1.ExecuteNonQuery();
    }

    using (var conn2 = new SqlConnection("Server=quickstartbmi.neu15011648751ff.database.windows.net;Database=b;User ID=myuser;Password=mypassword;Encrypt=true"))
    {
        conn2.Open();
        var cmd2 = conn2.CreateCommand();
        cmd2.CommandText = string.Format("insert into b.dbo.T2 values(2)");        cmd2.ExecuteNonQuery();
    }

    scope.Complete();
}

```

このコードは同一インスタンス内でデータを操作しますが、MSDTC を必要とします。

**対処法:** 2 つの接続を使用する代わりに [SqlConnection.ChangeDatabase(String)](https://docs.microsoft.com/dotnet/api/system.data.sqlclient.sqlconnection.changedatabase) を使って、接続コンテキスト内で他のデータベースを使用します。

### <a name="clr-modules-and-linked-servers-sometimes-cant-reference-a-local-ip-address"></a>CLR モジュールとリンク サーバーでローカル IP アドレスを参照できないことがある

Managed Instance に配置された CLR モジュールと、現在のインスタンスを参照しているリンク サーバーまたは分散クエリでは、ローカル インスタンスの IP を解決できないことがあります。 このエラーは一時的な問題です。

**対処法:** 可能であれば、CLR モジュールでコンテキスト接続を使用します。

## <a name="next-steps"></a>次の手順

- Managed Instance の詳細については、[Managed Instance の概要](sql-database-managed-instance.md)に関するページを参照してください
- 機能と比較の一覧については、「[Azure SQL Database の機能の比較](sql-database-features.md)」を参照してください。
- 新しい Managed Instance の作成方法を示したクイックスタートについては、[Managed Instance の作成](sql-database-managed-instance-get-started.md)に関するページを参照してください。
