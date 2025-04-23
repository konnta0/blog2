---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"XmlDoc を記載していない場合にワーニングとする方法","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"XmlDoc を記載していない場合にワーニングとする方法","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dg-permalink":"2025/04/24","permalink":"/2025/04/24/","metatags":{"og:title":"XmlDoc を記載していない場合にワーニングとする方法","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"XmlDoc を記載していない場合にワーニングとする方法","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-04-09T01:07:38.465+09:00"}
---


#dotnet 

`csproj` にて `GenerateDocumentationFile` を有効化することでメンバーに XmlDoc が記載されていない場合に警告を発生させることができます。

例)
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">  
  
    <PropertyGroup>  
        <TargetFramework>net9.0</TargetFramework>  
        <Nullable>enable</Nullable>  
        <ImplicitUsings>enable</ImplicitUsings>  
        <!-- 以下を追加する -->  
        <GenerateDocumentationFile>true</GenerateDocumentationFile>  
    </PropertyGroup>  

	<!-- 必要に応じて無効にする -->
    <PropertyGroup Condition="'$(Configuration)' == 'Release'">  
        <GenerateDocumentationFile>false</GenerateDocumentationFile>  
    </PropertyGroup>  
  
</Project>
```

以下のような実装をビルド
```cs
#pragma warning disable CS0414 // Field is assigned but its value is never used

public class Example
{
    public void PublicExampleMethod()
    {
    }

    public int PublicField = 0;
    internal int InternalField = 0;
    private int _privateField = 0;
    
    public int ExampleProperty { get; set; } = 0;
    internal int InternalExampleProperty { get; set; } = 0;
    private int PrivateExampleProperty { get; set; } = 0;
}

internal class InternalExample
{
    public void PublicExampleMethod()
    {
    }

    public int PublicField = 0;
    internal int InternalField = 0;
    private int _privateField = 0;
    
    public int ExampleProperty { get; set; } = 0;
    internal int InternalExampleProperty { get; set; } = 0;
    private int PrivateExampleProperty { get; set; } = 0;
}

file class PrivateExample
{
    public void PublicExampleMethod()
    {
    }

    public int PublicField = 0;
    internal int InternalField = 0;
    private int _privateField = 0;
    
    public int ExampleProperty { get; set; } = 0;
    internal int InternalExampleProperty { get; set; } = 0;
    private int PrivateExampleProperty { get; set; } = 0;
}

#pragma warning restore CS0414 // Field is assigned but its value is never used
```

警告発生時は以下のような表示になります。
![XmlDoc を記載していない場合にワーニングとする方法-1.png](/img/user/Engineering/-.NET/XmlDoc%20%E3%82%92%E8%A8%98%E8%BC%89%E3%81%97%E3%81%A6%E3%81%84%E3%81%AA%E3%81%84%E5%A0%B4%E5%90%88%E3%81%AB%E3%83%AF%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0%E3%81%A8%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95-1.png)

> [!note]
> GenerateDocumentationFile はその名の通り有効にするとドキュメントファイルを生成します。
> 場合によってはビルド時間が長くなる可能性があります。
> また、プロジェクトの規模によってはドキュメントのメンテナンスに時間や労力の負担となる場合があります。用法容量を守って利用しましょう。



