---
layout: post
title: Alfrescoでマルチテナントを設定する
category : alfresco
tagline: 
tags : [alfresco]
---
{% include JB/setup %}

Alfrescoでは1つのインスタンスで複数の独立したサイトを運用できます。以下、Alfresco Community 4.2.cでの操作です。

## 有効化

古いバージョンだと、Multi-Tenancyを有効にするには設定ファイルをリネームしてTomcatを再起動する必要がありますが、4.2.cではインストール時に有効になっていました。

4.0.d/4.0.1以前の場合は以下3ファイルをリネームし、Tomcatを再起動すればオッケー。

- alfresco/extension/mt/mt-context.xml.sample → mt-context.xml
- alfresco/extension/mt/mt-admin-context.xml.sample → mt-admin-context.xml
- alfresco/extension/mt/mt-contentstore-context.xml.sample → mt-contentstore-context.xml

## 管理コンソールへのアクセス

まずadminでAlfresco(/alfresco/)にログインした後、以下のURLにアクセスします。

	http://HOST:8080/alfresco/faces/jsp/admin/tenantadmin-console.jsp

ここのCommandエリアにコマンドを打ち込んで、テナントを設定していきます。

## テナントの一覧

現在設定されているテナントを状態とともに一覧表示します。

	show tenants


## テナントの追加

テナントを追加します。`root contentstore dir`は指定しないと`dir.root`になるので、特に指定しなくても大丈夫です。

	create <tenant domain> <tenant admin password> [<root contentstore dir>]


こんな感じで追加します。

	create example admin

## テナントの無効化

一時的にアクセスさせない場合に無効化します。

	disable <tenant domain>

## テナントの有効化

無効化したテナントを有効化します。追加された時点では有効になっています。

	enable <tenant domain>

## その他の操作

「help」と入力すると、コマンドのヘルプが表示されます。

- delete
- import
- export

などができるようですけど、export以外は「BETA」となっていて未検証です。

## テナントにログイン

ログイン時のユーザー名に`<username>@<tenant domain>`という形でテナントを指定します。`example`ドメインの`admin`ユーザーだったら`admin@example`です。

## 制限事項

サポートされていない機能もあるようですけど、未調査です。