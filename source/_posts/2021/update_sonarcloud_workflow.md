---
title: " [實作筆記] SonarCloud Move analysis to Java 11 更新 Github Action"
date: 2021/02/03
---

## 前情提要

長期使用的 SonarCloud 的 Github Action 突然執行失敗了.
原因是 SonarCloud 在 2021 年的 2 月 1 號開始，[不再支援使用舊的 Java 版本(1.8.0_282)](https://sonarcloud.io/documentation/appendices/end-of-support/),  
至少要更新至 Java 11.
錯誤的訊息如下:

```text
INFO: ------------------------------------------------------------------------

The version of Java (1.8.0_282) you have used to run this analysis is deprecated and we stopped accepting it. Please update to at least Java 11.
Temporarily you can set the property 'sonar.scanner.force-deprecated-java-version-grace-period' to 'true' to continue using Java 1.8.0_282
This will only work until Mon Feb 15 09:00:00 UTC 2021, afterwards all scans will fail.
You can find more information here: https://sonarcloud.io/documentation/upcoming/

ERROR:
The SonarScanner did not complete successfully
08:24:36.417  Post-processing failed. Exit code: 1
Error: Process completed with exit code 1.
```

## 調整方法

移除掉原本的 SonarScanner

```text
- name: SonarScanner Begin
  run: dotnet sonarscanner begin /k:"Marsen.NetCore.Dojo" /o:"marsen-github" /d:"sonar.host.url=https://sonarcloud.io" /d:"sonar.login="$SONAR_LOGIN
## 中略
- name: SonarScanner End
  run: dotnet sonarscanner end /d:"sonar.login="$SONAR_LOGIN
```

在整個方案的根目錄加上一個檔案`sonar-project.properties`,

```text
sonar.organization=<replace with your SonarCloud organization key>
sonar.projectKey=<replace with the key generated when setting up the project on SonarCloud>

# relative paths to source directories. More details and properties are described
# in https://sonarcloud.io/documentation/project-administration/narrowing-the-focus/
sonar.sources=.
```

`sonar.organization` 要如何取得呢 ?  
登入 [SonarCloud](https://sonarcloud.io/), 右上角頭像 > My Organizations 即可查詢到 Organization Key 值。
![Organization Key](https://i.imgur.com/mIL7Wup.jpg)

上方 `My Projects` > `Administration` > `Update Key`  
即可查詢到 Project Key 值
![Project Key](https://i.imgur.com/Jzb69kE.jpg)

最後在 Github Action Workflow 加上這段，Github Action 就復活啦

```text
- uses: sonarsource/sonarcloud-github-action@master
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

## 參考

- [Move analysis to Java 11](https://sonarcloud.io/documentation/appendices/move-analysis-java-11/)
- [sonarcloud-github-action](https://github.com/SonarSource/sonarcloud-github-action)

(fin)
