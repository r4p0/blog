---
title: Unity Hub 跳过登录
date: 2024-07-25 15:10:45
tags: 
- Unity
- Patch
- Hacker
---
> 本方法参考 Uni-HubHacker 仓库（已和谐）

## 1. 原理说明
由于Unity Hub使用的是`electron`框架，所以其部分运行时`js`代码会存放在`app.asar`文件中。由于Unity Hub未对`app.asar`文件进行加密切未对其进行hash校验（但疑似会对文件大小校验），所以可以修改`app.asar`文件中登陆相关的代码，从而跳过登陆验证。由于`.asar`文件是归档文件，所以需要使用二进制工具打开，直接对代码进行明文修改。

<!--more-->

## 2. 具体操作步骤
- 找到Unity Hub安装目录下的`Unity Hub/resources/app.asar`文件使用二进制工具打开（这里以`HxD`为例，修改前请自行做好备份工作）。
- 定为到以下文本并逐一替换

```diff
...
-const isFirstTimeOpen = yield promisify(jsonStorage.get)(IS_FIRST_TIME_OPEN_KEY);
+const isFirstTimeOpen = false;/*omisify(jsonStorage.get)(IS_FIRST_TIME_OPEN_KEY*/
...
-return this.getConfig(localSettings_1.default.keys.DISABLE_AUTO_UPDATE);
+return true;/*tConfig(localSettings_1.default.keys.DISABLE_AUTO_UPDATE*/
...
-return this.getConfig(localSettings_1.default.keys.DISABLE_SIGNIN);
+return true;/*tConfig(localSettings_1.default.keys.DISABLE_SIGNIN*/
...
-return this.getConfig(localSettings_1.default.keys.DISABLE_WELCOME_SCREEN);
+return true;/*tConfig(localSettings_1.default.keys.DISABLE_WELCOME_SCREEN*/
```

- 修改完成后保存即可