---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"Provider の追加方法","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Provider の追加方法","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/CDKTF/Provider の追加方法/","metatags":{"og:title":"Provider の追加方法","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"Provider の追加方法","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2025-01-14T01:17:05.330+09:00"}
---


#cdktf #IaC

例として [kubernetes provider](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs) を追加してみます。

```shell
$ cdktf --version                        
0.20.8
```

```shell
$ cdktf provider add hashicorp/kubernetes                            
[INFO] default - Checking whether pre-built provider exists for the following constraints:
  provider: kubernetes
  version : latest
  language: csharp
  cdktf   : 0.21.0-pre.143

[INFO] default - Pre-built provider does not exist for the given constraints.
[INFO] default - Adding local provider registry.terraform.io/hashicorp/kubernetes with version constraint undefined to cdktf.json
Local providers have been updated. Running cdktf get to update...
Generated csharp constructs in the output directory: .gen
```

```shel
$ git diff cdktf.json 
diff --git a/src/Infrastructure.CDKTF/cdktf.json b/src/Infrastructure.CDKTF/cdktf.json
index 538c1732..f6f40f8d 100644
--- a/src/Infrastructure.CDKTF/cdktf.json
+++ b/src/Infrastructure.CDKTF/cdktf.json
@@ -3,9 +3,9 @@
   "app": "dotnet run -p Infrastructure.CDKTF.csproj",
   "projectId": "c73d9c4b-4e8c-43a3-b37a-48cf6185576c",
   "sendCrashReports": "false",
-  "terraformProviders": [],
+  "terraformProviders": [
+    "hashicorp/kubernetes@~> 2.35"
+  ],
```