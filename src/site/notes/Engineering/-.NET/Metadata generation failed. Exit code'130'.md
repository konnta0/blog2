---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"Metadata generation failed. Exit code: '130'","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Metadata generation failed. Exit code: '130'","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/-.NET/Metadata generation failed. Exit code'130'/","metatags":{"og:title":"Metadata generation failed. Exit code: '130'","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Metadata generation failed. Exit code: '130'","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-19T01:40:00.419+09:00","updated":"2025-01-19T01:51:39.898+09:00"}
---


#AzureFunctions #dotnet #macOS

Azure Functions のプロジェクトを新規に作成してビルドしたときに以下のようなエラーに出くわして解消したので、そのメモです。

### 出くわしたエラー
```shell
0>Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator.targets(37,5): Error : Metadata generation failed. Exit code: '130' Error: 'Failed to load /usr/local/share/dotnet/shared/Microsoft.NETCore.App/2.1.23/libhostpolicy.dylib, error: dlopen(/usr/local/share/dotnet/shared/Microsoft.NETCore.App/2.1.23/libhostpolicy.dylib, 0x0001): tried: '/usr/local/share/dotnet/shared/Microsoft.NETCore.App/2.1.23/libhostpolicy.dylib' (mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64')), '/System/Volumes/Preboot/Cryptexes/OS/usr/local/share/dotnet/shared/Microsoft.NETCore.App/2.1.23/libhostpolicy.dylib' (no such file), '/usr/local/share/dotnet/shared/Microsoft.NETCore.App/2.1.23/libhostpolicy.dylib' (mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64'))An error occurred while loading required library libhostpolicy.dylib from [/usr/local/share/dotnet/shared/Microsoft.NETCore.App/2.1.23]'
```

### 解決方法
.NET SDK 等を一旦削除する
```shell
sudo rm -rf /usr/local/share/dotnet
sudo rm -rf /etc/dotnet
sudo rm -rf ~/.dotnet
```

公式サイトより再度 .NET SDK 等をダウンロードしてインストールする
https://dotnet.microsoft.com/ja-jp/download


参考 issue: https://github.com/dotnet/sdk/issues/27761#issuecomment-1244399021
(Azure Funcitons が何か悪い訳ではなさそうな雰囲気)