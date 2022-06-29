# EntityFramework 迁移笔记

本文记录使用EF执行数据库迁移的过程，过程参考[MSDN官方文档](https://docs.microsoft.com/zh-cn/ef/ef6/modeling/code-first/migrations)。

实现迁移的项目有两个，一个使用 `.NET Framework 4.8` + `Entity Framework 6`，另一个项目使用`.NET 6` + `Entity Framework Core`。

首先实现EFCore的迁移功能。

## 安装迁移工具
在“包管理器控制台”中运行以下命令，安装包管理器控制台工具：
```powershell
PM> Install-Package Microsoft.EntityFrameworkCore.Tools
PM> Update-Package Microsoft.EntityFrameworkCore.Tools
PM> Get-Help about_EntityFrameworkCore

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\
```
具体帮助参考[MSDN](https://docs.microsoft.com/zh-cn/ef/core/cli/powershell)

## EF Core
### 开发手动迁移
```powershell
PM> Add-Migration InitialCreate
Build started...
Build succeeded.
Microsoft.EntityFrameworkCore.Model.Validation[10400]
      Sensitive data logging is enabled. Log entries and exception messages may include sensitive application data; this mode should only be enabled during development.
Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core 6.0.6 initialized 'BFCenterDbContext' using provider 'Microsoft.EntityFrameworkCore.Sqlite:6.0.6' with options: SensitiveDataLoggingEnabled DetailedErrorsEnabled 
To undo this action, use Remove-Migration.
```
_由于我在调试中启用了 `Sensitive data` 选项，因此出现以上提示。_

执行完成后，EFCore在项目中创建了一个名为“Migrations”的目录，并生成一些文件。其中包含了一个DbContext的快照，以及`日期时间_InitialCreate.cs`的文件。

然后执行`Update-Database`就会自动创建数据库了，后续有更改则同样使用`Add-Migration`添加迁移来生成迁移类，然后用`Update-Database`来更新数据库。

### 生产环境迁移

显然，以上做法更适合开发环境，你可以手工执行命令迁移数据库，但这不适合生产使用，根据[MSDN的建议](https://docs.microsoft.com/zh-cn/ef/core/managing-schemas/migrations/applying?tabs=vs#:~:text=%E5%BB%BA%E8%AE%AE%E9%80%9A%E8%BF%87%E7%94%9F%E6%88%90%20SQL%20%E8%84%9A%E6%9C%AC%EF%BC%8C%E5%B0%86%E8%BF%81%E7%A7%BB%E9%83%A8%E7%BD%B2%E5%88%B0%E7%94%9F%E4%BA%A7%E6%95%B0%E6%8D%AE%E5%BA%93%E3%80%82%20%E6%AD%A4%E7%AD%96%E7%95%A5%E7%9A%84%E4%BC%98%E7%82%B9%E5%8C%85%E6%8B%AC%EF%BC%9A)，应该通过生成SQL脚本，将迁移部署到生产数据库。此策略的优点包括：
 - 可以检查 SQL 脚本的准确性；这一点很重要，因为将架构更改应用于生产数据库是一项可能导致数据丢失的潜在危险操作。
 - 在某些情况下，可以根据生产数据库的特定需求调整这些脚本。
 - SQL 脚本可以与部署技术结合使用，甚至可以在 CI 过程中生成。
 - SQL 脚本可以提供给 DBA，并且可以单独管理和存档。

### SQL 脚本 命令基本用法
以下命令将生成一个从空白数据库到最新迁移的 SQL 脚本：
```powershell
Script-Migration
```

使用 From（to 隐含）
以下命令将生成一个从给定迁移到最新迁移的 SQL 脚本。
```powershell
Script-Migration AddNewTables
```
使用 From 和 To
以下命令将生成一个从指定 `from` 迁移到指定 `to` 迁移的 SQL 脚本。
```powershell
Script-Migration AddNewTables AddAuditTable
```
可以使用比 `to` 新的 `from` 来生成回退脚本。

脚本生成接受以下两个参数，以指示应生成的迁移范围：
 - from 迁移应是运行该脚本前应用到数据库的最后一个迁移。 如果未应用任何迁移，请指定 `0`（默认值）。
 - to 迁移是运行该脚本后应用到数据库的最后一个迁移。 它默认为项目中的最后一个迁移。

### 自动迁移
经过研究，无论是生成执行代码然后手工更新数据库，还是生成SQL然后用其它方法执行脚本，都不太符合我的需求，我想实现的是像以前EF6一样自动迁移到新的版本，然后还可以手动对中间的迁移步骤进行调整。

根据文档[在运行时应用迁移](https://docs.microsoft.com/zh-cn/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#apply-migrations-at-runtime)章节所述，这样做存在许多问题，不适合管理生产数据库，列举了如下原因：
 - 如果应用程序的多个实例正在运行，这两个应用程序可能会尝试同时应用迁移并失败（更糟糕的情况是导致数据损坏）。
 - 同样，如果一个应用程序正在访问数据库，而另一个应用程序正在迁移它，这可能会导致严重的问题。
 - 应用程序必须具有提升的访问权限才能修改数据库架构。 在生产环境中限制应用程序的数据库权限通常是一种很好的做法。
 - 出现问题时，能够回滚已应用的迁移很重要。 其他策略可以轻松提供此功能，并且开箱即用。
 - 程序会直接应用 SQL 命令，不给开发人员检查或修改的机会。 这在生产环境中可能会很危险。

有一说一，确实有道理，但问题在于我不是应用于自家服务，没有自己的运维来升级，我需要应用程序自动将数据库更新到新版本，所以我必须使用应用自动迁移。

EFCore 自动迁移需要调用 `context.Database.Migrate()` 方法，典型的 ASP.NET 应用可以执行以下操作：
```cs
public static void Main(string[] args)
{
    var host = CreateHostBuilder(args).Build();

    using (var scope = host.Services.CreateScope())
    {
        var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
        db.Database.Migrate();
    }

    host.Run();
}
```
> 请注意，`Migrate()` 构建于 `IMigrator` 服务之上，可用于更高级的方案。 请使用 `myDbContext.GetInfrastructure().GetService<IMigrator>()` 进行访问。

感谢[这篇博客](https://www.cnblogs.com/kasnti/p/10864668.html)，让我知道可以使用 `DbContext.Database.GetPendingMigrations()` 方法来检测程序集中是否包含待迁移的内容，如果有的话就自动应迁移。

> [GetPendingMigrations方法官方文档说明](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.getpendingmigrations)
获取在程序集中定义但尚未应用于目标数据库的所有迁移。
Gets all migrations that are defined in the assembly but haven't been applied to the target database.

```cs
if (DbContext.Database.GetPendingMigrations().Any())
    DbContext.Database.Migrate(); //执行迁移
```

### 测试验证
调试，执行 `DbContext.Database.GetPendingMigrations()` 发现返回了前面用 `Add-Migration` 生成的类名 `日期时间_InitialCreate` ，再下一步执行 `Migrate()` 时，自动运行了 `InitialCreate` 的 `Up` 方法，成功迁移。

迁移完成后，此时再次执行 `GetPendingMigrations()` 方法发现返回列表为空，证明该方法有效。

再更改实体，添加迁移，显然也没有问题。

### 总结
根据项目实际情况，结合建议，我需要的迁移方法为：开发时生成迁移代码，运行时自动检查并迁移。

具体流程为，在程序包管理器控制台中执行 `Add-Migration` 命令来生成迁移类，在代码中调用 `dbContext.Database.Migrate()` 进行迁移，使用 `dbContext.Database.GetPendingMigrations()` 方法可以获取到未应用的迁移。

不得不说，EF Core 迁移好像更简单了些，接下来则是对另一个使用 EF6 的 .NET Framework 4.8 的项目实现迁移了。

## EF6 Code First 迁移
MSDN地址：https://docs.microsoft.com/zh-cn/ef/ef6/modeling/code-first/migrations/

### 项目背景
该项目在本次迁移之前，一直使用的是自动迁移功能，大部分时候都可以自动完成从旧的数据库升级到新的实体过程，为什么这次要改成手动迁移，是因为这一次迁移如果使用自动完成会造成数据丢失。

我需要将一张表的多个列合并为一个列，自动生成的代码只会删除这些列，然后新建列，这会造成数据的丢失，因此我需要手工编写迁移的SQL，将旧的列的数据合并到新的列，然后再删除这些列。这就是本次改造的目标。

### 自动迁移
在之前的代码中，我使用的是自动迁移，代码入下：
```cs
public BaseContext(DbConnection connection)
    : base(connection, true)
{
#if DEBUG
    Database.Log += Console.Write;
#endif

    // 解决链接字符串中使用相对路径导致无法创建数据库问题（SQLite）
    // https://blog.csdn.net/kindmb/article/details/102328189
    Environment.CurrentDirectory = AppDomain.CurrentDomain.BaseDirectory;
    // 设定初始化器为自动迁移数据库到最新版本
    Database.SetInitializer(new MigrateDatabaseToLatestVersion<BaseContext, Configuration>(true));
}
```
在这个 `Configuration` 中设置了自动迁移的属性，代码如下：
```cs
class Configuration : DbMigrationsConfiguration<BaseContext>
{
    public Configuration()
    {
        AutomaticMigrationsEnabled = true;
        AutomaticMigrationDataLossAllowed = true;
        SetSqlGenerator("MySql.Data.MySqlClient", new MySqlMigrationSqlGenerator());
    }
}
```
像以上这样做了以后，只要修改了实体，程序启动时会自动检查数据库的版本，然后自动迁移到最新实体，我这么干已经两年了，没出什么问题，但是现在自动的行为不能满足我的需求了，我需要合并一些列，因此开始研究如何进行手动迁移。

### 自动迁移+手动迁移其中一部分
为了保险起见，新建一个项目来进行测试。

这篇博客非常有用，可以参考。
https://www.cnblogs.com/pergf/p/12718403.html

经过实践，EF的迁移非常强大，可以实现我的需求，下面总结一下我做了什么。

我完整的按照[官方文档](https://docs.microsoft.com/zh-cn/ef/ef6/modeling/code-first/migrations/automatic)一步一步的学习了整个迁移流程，非常有效，一定要先阅读这些文档！

首先，我新建了一个`.NET Framework`的控制台项目，在NuGet管理器中添加了`EntityFrameword`的引用，现在的版本是`6.4.4`。

为了贴合我的项目实际情况，我还添加了MySQL相关的库，包括`MySql.Data`和`MySql.Data.EntityFramework`，这两个包现在是`8.0.29`版本，这和我的项目不太一样，以前用的是`MySql.Data.Entity 6.10.9`，现在这个包被弃用了，改成了`MySql.Data.EntityFramework`，因此可以使用最新版本的MySQL EF，非常好！

在将MySql添加到项目后，我首先做的是在`App.config`中删除了关于`SqlServer.Client`相关的内容，毕竟我不需要SqlServer的库。

然后在配置文件中我添加了连接字符串相关的配置。
```xml
<connectionStrings>
	<add name="MySql" providerName="MySql.Data.MySqlClient" connectionString="Server=localhost;Database=test;Uid=root;Pwd=123456;" />
</connectionStrings>
```

接下来，我按照教程添加了`Model.cs`，并且添加了`BlogContext`类和`Blog`类，由于我使用了自定义的连接字符串，因此我在构造函数中指定了连接字符串的名称 `public BlogContext(): base("name=MySql")`

然后直接运行程序，果不其然出现了错误，错误告诉我找不到MySql.Client提供程序，翻了下旧代码，发现配置文件中还需要增加一段数据提供工厂的配置
```xml
  <system.data>
	<DbProviderFactories>
	  <remove invariant="MySql.Data.MySqlClient" />
	  <add name="MySQL Data Provider" invariant="MySql.Data.MySqlClient" description=".Net Framework Data Provider for MySQL" type="MySql.Data.MySqlClient.MySqlClientFactory, MySql.Data, Version=8.0.29.0, Culture=neutral, PublicKeyToken=c5687fc88969c44d" />
	</DbProviderFactories>
  </system.data>
```
这些程序集名称要和前面的那段一样，完整的配置文件如下
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<configSections>
		<!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
		<section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
	</configSections>
	<startup>
		<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.8" />
	</startup>
	<entityFramework>
		<providers>
			<provider invariantName="MySql.Data.MySqlClient" type="MySql.Data.MySqlClient.MySqlProviderServices, MySql.Data.EntityFramework, Version=8.0.29.0, Culture=neutral, PublicKeyToken=c5687fc88969c44d" />
		</providers>
	</entityFramework>
	<connectionStrings>
		<add name="MySql" providerName="MySql.Data.MySqlClient" connectionString="Server=localhost;Database=test;Uid=root;Pwd=123456;" />
	</connectionStrings>
	<system.data>
		<DbProviderFactories>
			<remove invariant="MySql.Data.MySqlClient" />
			<add name="MySQL Data Provider" invariant="MySql.Data.MySqlClient" description=".Net Framework Data Provider for MySQL" type="MySql.Data.MySqlClient.MySqlClientFactory, MySql.Data, Version=8.0.29.0, Culture=neutral, PublicKeyToken=c5687fc88969c44d" />
		</DbProviderFactories>
	</system.data>
	<runtime>
		<assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
			<dependentAssembly>
				<assemblyIdentity name="System.Runtime.CompilerServices.Unsafe" publicKeyToken="b03f5f7f11d50a3a" culture="neutral" />
				<bindingRedirect oldVersion="0.0.0.0-6.0.0.0" newVersion="6.0.0.0" />
			</dependentAssembly>
			<dependentAssembly>
				<assemblyIdentity name="Google.Protobuf" publicKeyToken="a7d26565bac4d604" culture="neutral" />
				<bindingRedirect oldVersion="0.0.0.0-3.21.2.0" newVersion="3.21.2.0" />
			</dependentAssembly>
			<dependentAssembly>
				<assemblyIdentity name="K4os.Compression.LZ4.Streams" publicKeyToken="2186fa9121ef231d" culture="neutral" />
				<bindingRedirect oldVersion="0.0.0.0-1.2.16.0" newVersion="1.2.16.0" />
			</dependentAssembly>
			<dependentAssembly>
				<assemblyIdentity name="BouncyCastle.Crypto" publicKeyToken="0e99375e54769942" culture="neutral" />
				<bindingRedirect oldVersion="0.0.0.0-1.8.9.0" newVersion="1.8.9.0" />
			</dependentAssembly>
			<dependentAssembly>
				<assemblyIdentity name="System.Memory" publicKeyToken="cc7b13ffcd2ddd51" culture="neutral" />
				<bindingRedirect oldVersion="0.0.0.0-4.0.1.2" newVersion="4.0.1.2" />
			</dependentAssembly>
			<dependentAssembly>
				<assemblyIdentity name="K4os.Hash.xxHash" publicKeyToken="32cd54395057cec3" culture="neutral" />
				<bindingRedirect oldVersion="0.0.0.0-1.0.7.0" newVersion="1.0.7.0" />
			</dependentAssembly>
		</assemblyBinding>
	</runtime>
</configuration>
```
后面那堆乱七八糟的引用都是在添加了MySQL以后关联的，如果只是SqlServer应该没那么多东西。

目前为止我主动添加的NuGet包仅有EF和MySql的两个，关联的包我也把它们全都升级了，现在的packages.config如下
```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="BouncyCastle" version="1.8.9" targetFramework="net48" />
  <package id="EntityFramework" version="6.4.4" targetFramework="net48" />
  <package id="Google.Protobuf" version="3.21.2" targetFramework="net48" />
  <package id="K4os.Compression.LZ4" version="1.2.16" targetFramework="net48" />
  <package id="K4os.Compression.LZ4.Streams" version="1.2.16" targetFramework="net48" />
  <package id="K4os.Hash.xxHash" version="1.0.7" targetFramework="net48" />
  <package id="MySql.Data" version="8.0.29" targetFramework="net48" />
  <package id="MySql.Data.EntityFramework" version="8.0.29" targetFramework="net48" />
  <package id="System.Buffers" version="4.5.1" targetFramework="net48" />
  <package id="System.Memory" version="4.5.5" targetFramework="net48" />
  <package id="System.Numerics.Vectors" version="4.5.0" targetFramework="net48" />
  <package id="System.Runtime.CompilerServices.Unsafe" version="6.0.0" targetFramework="net48" />
</packages>
```

配置完成后，再次启动程序，还是错误，这次是报告Sql生成器的错误，查找资料后，在DbContext增加注解`DbConfigurationType`解决，完整的类如下：
```cs
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Data.Entity;

using MySql.Data.EntityFramework;

namespace ConsoleApp
{
    [DbConfigurationType(typeof(MySqlEFConfiguration))]
    public class BlogContext : DbContext
    {
        public BlogContext(): base("name=MySql")
        {
        }
        public DbSet<Blog> Blogs { get; set; }
    }

    public class Blog
    {
        public int BlogId { get; set; }
        public string Name { get; set; }
        public string Url { get; set; }
        public int Rating { get; set; }
        public virtual List<Post> Posts { get; set; }
    }

    public class Post
    {
        public int PostId { get; set; }
        [MaxLength(200)]
        public string Title { get; set; }
        public string Content { get; set; }
        public string Abstract { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
}
```

以上代码是最终代码，具体过程看官方文档操作。修改完成后再次启动，所有问题解决。

使用`Enable-Migrations -EnableAutomaticMigrations`启动迁移功能，并且使能自动迁移。这样会在目录下生成一个`Migrations`文件夹，并且生成一个`Configuration`类。生成代码中使能了自动迁移，根据我的经验，还要把自动迁移允许删除数据给启用，否则如果自动迁移要删除列存在数据会导致迁移失败：`AutomaticMigrationDataLossAllowed = true;`

按照以前的经验，还需要加上`SetSqlGenerator("MySql.Data.MySqlClient", new MySqlMigrationSqlGenerator());`来使用MySQL生成SQL代码，但是我发现只需要在Context上注解`[DbConfigurationType(typeof(MySqlEFConfiguration))]`就可以自动使用MySql的提供程序来生成SQL。完整的代码如下：
```cs
namespace ConsoleApp.Migrations
{
    using System;
    using System.Data.Entity;
    using System.Data.Entity.Migrations;
    using System.Linq;

    internal sealed class Configuration : DbMigrationsConfiguration<BlogContext>
    {
        public Configuration()
        {
            AutomaticMigrationsEnabled = true;
            AutomaticMigrationDataLossAllowed = true;
            ContextKey = "ConsoleApp.BlogContext";
        }

        protected override void Seed(BlogContext context)
        {
            //  This method will be called after migrating to the latest version.

            //  You can use the DbSet<T>.AddOrUpdate() helper extension method
            //  to avoid creating duplicate seed data.
        }
    }
}
```

继续按照教程，使用`Add-Migration AddBlogRating`命令来给Blogs添加Rating字段时手动修改升级内容，在生成的迁移文件中，修改`Up()`中的代码，给Rating字段加了个默认值3，测试后默认值没有效果，查看文档还有一个`defaultValueSql`的字段，使用该字段设置默认值后生效，可能是之前的`defaultValue`字段在MySQL里不兼容？完整代码如下：
```cs
namespace ConsoleApp.Migrations
{
    using System;
    using System.Data.Entity.Migrations;
    
    public partial class AddBlogRating : DbMigration
    {
        public override void Up()
        {
            AddColumn("dbo.Blogs", "Rating", c => c.Int(nullable: false, defaultValueSql: "3"));
        }
        
        public override void Down()
        {
            DropColumn("dbo.Blogs", "Rating");
        }
    }
}
```

再按照官方文档操作后，我已经了解了这个迁移的过程。生成的手动迁移类中包含了迁移前后的实体快照，因此迁移旧的数据库时会先迁移到上个版本的快照，然后再执行手动迁移的代码，然后再通过迁移后的快照，迁移到程序集的最新版本。

按照这个思路，我原先的项目还是用之前的自动迁移，然后在合并字段的步骤生成一个手工迁移类，手动把这些字段合并，就可以完美解决问题。

## 总结
EF的迁移功能非常强大，官方文档非常详细，过程中遇到的一些错误网上也有许多博客分享了经验，.NET真香！

EFCore的迁移就用 `Add-Migration` 生成迁移代码，然后服务启动时检查是否存在未应用的迁移，自动迁移即可。

EF6的迁移则延续以前的自动迁移设置，只是在需要手动便携迁移代码时，使用`Add-Migration`来生成迁移类，修改`Up()`方法内的代码达到手动调整迁移的目的。

官方文档非常重要，一定要仔细阅读官方文档：[EF Core 迁移文档](https://docs.microsoft.com/zh-cn/ef/core/managing-schemas/migrations/) [EF6 迁移文档](https://docs.microsoft.com/zh-cn/ef/ef6/modeling/code-first/migrations/)
