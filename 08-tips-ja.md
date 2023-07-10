# ヒント

## .NET inotifyのクラッシュ

デフォルトでは、.NETはすべての`appsettings.json`ファイルに対してinotifyによるファイル監視がされています。Kubernetesでは、これは一般的に不要であり、問題を引き起こす可能性があります。例えば、起動中のクラッシュがあります。
```
Unhandled exception. System.IO.IOException: The configured user limit (128) on the number of inotify instances has been reached, or the per-process limit on the number of open file descriptors has been reached.
   at System.IO.FileSystemWatcher.StartRaisingEvents()
   at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.TryEnableFileSystemWatcher()
   at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.CreateFileChangeToken(String filter)
   at Microsoft.Extensions.Primitives.ChangeToken.OnChange(Func`1 changeTokenProducer, Action changeTokenConsumer)
   at Microsoft.Extensions.Configuration.FileConfigurationProvider..ctor(FileConfigurationSource source)
   at Microsoft.Extensions.Configuration.Json.JsonConfigurationSource.Build(IConfigurationBuilder builder)
   at Microsoft.Extensions.Configuration.ConfigurationManager.AddSource(IConfigurationSource source)
   at Microsoft.Extensions.Configuration.ConfigurationManager.Microsoft.Extensions.Configuration.IConfigurationBuilder.Add(IConfigurationSource source)
   at Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ApplyDefaultAppConfiguration(HostBuilderContext hostingContext, IConfigurationBuilder appConfigBuilder, String[] args)
   at Microsoft.Extensions.Hosting.HostApplicationBuilder..ctor(HostApplicationBuilderSettings settings)
   at Microsoft.AspNetCore.Builder.WebApplicationBuilder..ctor(WebApplicationOptions options, Action`1 configureDefaults)
   at Microsoft.AspNetCore.Builder.WebApplication.CreateBuilder(String[] args)
   at Program.<Main>$(String[] args) in /source/Program.cs:line 1
```
.NETがこれらのファイル監視をするのを防ぎ、上記のクラッシュを防ぐために、Contrastは以下の環境変数を適用することを推奨しています：

```
DOTNET_HOSTBUILDER__RELOADCONFIGONCHANGE=false
```
