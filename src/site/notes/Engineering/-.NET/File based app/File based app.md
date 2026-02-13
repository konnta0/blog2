---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"File based app","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"File based app","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/-.NET/File based app/File based app/","metatags":{"og:title":"File based app","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"File based app","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2026-01-05T01:45:12.000+09:00","updated":"2026-02-14T01:46:01.247+09:00"}
---

#dotnet #csharp 

.NET 10 で追加された file based app についての備忘録です。

# 概要
cs ファイル単体でプログラムが実行できるようになりました。
プロジェクトファイル(.csproj)やソリューションファイル(.sln)を作成しなくても C# コードが実行できます。
これが登場する前は、[GitHub - dotnet-script/dotnet-script: Run C# scripts from the .NET CLI.](https://github.com/dotnet-script/dotnet-script) などを利用していましたが、.NET 本体でサポートされた形になります。

# いくつかサンプル
## その 1
スーパーミニマルサンプルとしては以下のよう記述のあるファイルを作成します(`app.cs` とする)
```cs
Console.WriteLine("hello");
```

実行するときは下記です。
```shell
$ dotnet run ./app.cs                            
hello
```

利用する際にプロジェクトファイルを作成しなくてもいいのは以前に比べてかなり手軽になりまたね。

## その 2
所謂 shebang にも対応していて(app2.cs)
```cs
#!/usr/bin/env dotnet
Console.WriteLine("hello");
```

実行する際には下記にようになります。(実行権限の付与は必要)
```shell
$ ./app2.cs
hello
```

## その 3
はたまたディレクティブを設定することで様々なアプリケーションの作成ができます。
例えば、Web API だったら以下のようなイメージ(あくまで超最小構成です)(app3.cs)
```cs
#!/usr/bin/env dotnet
#:sdk Microsoft.NET.Sdk.Web

var app = WebApplication.CreateBuilder(args).Build();
app.MapGet("/", () => "Hello World!");
app.Run();
```

```shell
 ./app3.cs
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5000
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
```

別窓で実行
```shell
$ curl http://localhost:5000
Hello World!
```

6 行で Web API のアプリケーションが作れるのはちょっとしたサンプル実装だったり確認が捗る印象です。

しれっと出現した `sdk` ディレクティブ以外にも `package` や `property` 、`project` ディレクティブなどなどあったりします。
詳しくは[こちら](https://github.com/dotnet/docs/blob/main/docs/core/sdk/file-based-apps.md#supported-directives) を参照ください。
## その 4
標準入力から受け取って C# のプログラム実行することも可能です。
```shell
$ echo 'Console.WriteLine("hello from stdin!");' | dotnet run -
hello from stdin!
```

# file based app からプロジェクトベースのアプリに戻すこともできる
file based でサクッと作っておいたが機能が増えてきたりしてプロジェクト(.csproj) で管理したくなったら変換用のコマンドを実行することで従来の形式に戻すことが可能です。
```shell
$ dotnet project convert app.cs

出力ディレクトリ (app) を指定します: # ディレクトリを変更することも可能ではある

$ ls app   
app.cs          app.csproj
```

# 実行ファイルはどこへ？？
.NET だと dll だったり、exe だったり実行用のバイナリが存在するはずですが、上記のような file based app は cs ファイルなどを直接実行しているため、実行ファイルがどこにあるのか気になるところです。(人による)

デフォルトだと `<temp>/dotnet/runfile/<appname>-<appfilesha>/bin/<configuration>/` 以下に仮想のプロジェクトファイルが作成されます。(以下 mac で確認)
```shell
/Users/XXXXX/Library/Application\ Support/dotnet/runfile/app3-12dad382bc16f2b79de0dfb3fb980498ed66eaeda0ad54726c296f5b27e4b83e/bin/debug/app3
```

```shell
$ ls /Users/XXXXX/Library/Application\ Support/dotnet/runfile/app3-12dad382bc16f2b79de0dfb3fb980498ed66eaeda0ad54726c296f5b27e4b83e/bin/debug
app3                                    app3.dll                                app3.runtimeconfig.json
app3.deps.json                          app3.pdb                                app3.staticwebassets.endpoints.json
```

参考:
https://devblogs.microsoft.com/dotnet/announcing-dotnet-run-app/
https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/tutorials/file-based-programs
https://github.com/dotnet/docs/blob/main/docs/core/sdk/file-based-apps.md