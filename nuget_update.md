layout: page
title: "VS项目包管理器更新所有依赖到正确版本的方法"
permalink: /nuget_update.md

# VS项目包管理器更新所有依赖到正确版本的方法

首先删除所有项目app.config中AssemblyBinding小节

然后在包管理器命令行中运行以下命令更新所有依赖包到最新版本:

```
Get-Project –All | % { Get-Package -ProjectName $_.ProjectName | % { update-package $_.Id -reinstall -ProjectName $_.ProjectName -ignoreDependencies } }
```

最后运行以下命令更新所有项目绑定依赖:

```
Get-Project –All | Add-BindingRedirect
```
