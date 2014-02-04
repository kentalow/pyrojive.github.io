---
layout: post
title: Ubuntu 12.04 LTSにAlfresco Community 4.2.cをインストール
category : alfresco
tagline: 
tags : [alfresco, ubuntu]
---
{% include JB/setup %}

Alfresco Community 4.2.cをVMWare Fusion 5で構築した時の手順です。

## Ubuntu 12.04 LTSインストール

サーバーソフトウェアにOpenSSHを指定する以外デフォルトでインストール。インストール後は適宜ネットワークの設定をおこないます。

	$ sudo vi /etc/network/interfaces
	-- snip --
	iface eth0 inet static
	  address 192.168.1.100
	  netmask 255.255.255.0
	  network 192.168.1.0
	  broadcast 192.168.1.255
	  gateway 192.168.1.1
	  dns-nameservers 8.8.8.8
	-- snip --
	$ sudo reboot

最後にサーバーのソフトウェアの状態を最新に保ちます。

	$ sudo aptitude update
	$ sudo aptitude upgrade

## 日本語フォントなどの必要ソフトウェアのインストール

### Alfrescoプレビュー関連

	$ sudo aptitude install python-software-properties # ← add-apt-repositoryをインストールするため
	$ sudo add-apt-repository ppa:guilhem-fr/swftools # ←swftoolsをインストールするため
	$ sudo add-apt-repository ppa:libreoffice/ppa # ←LibreOffice3.6をインストールするため
	$ sudo aptitude update
	$ sudo aptitude install imagemagick ffmpeg swftools libreoffice ttf-mscorefonts-installer


ここで入れるswftoolsが0.9.2の場合、Alfrescoのプレビューで表示がずれてしまうので0.9.1を手動でインストールする必要があります。

	$ sudo aptitude install build-essential zlib1g-dev libjpeg62-dev libgif-dev libpng12-dev libfreetype6-dev
	$ wget http://www.swftools.org/swftools-0.9.1.tar.gz
	$ tar zxf swftools-0.9.1.tar.gz
	$ cd swftools-0.9.1
	$ ./configure
	$ make
	$ sudo make install

### 日本語関連

	$ wget -q https://www.ubuntulinux.jp/ubuntu-ja-archive-keyring.gpg -O- | sudo apt-key add -
	$ wget -q https://www.ubuntulinux.jp/ubuntu-jp-ppa-keyring.gpg -O- | sudo apt-key add -
	$ sudo wget https://www.ubuntulinux.jp/sources.list.d/precise.list -O /etc/apt/sources.list.d/ubuntu-ja.list
	$ sudo aptitude update
	$ sudo aptitude install fonts-ipafont-gothic fonts-ipafont-mincho libreoffice-l10n-ja

日本語フォントが入ります。

## JavaとTomcatインストール

### Java7インストール

	$ sudo add-apt-repository ppa:webupd8team/java
	$ sudo aptitude update
	$ sudo aptitude install oracle-java7-installer
	$ sudo update-alternatives --config java # java-7-oracleを選択
	$ sudo rm /usr/lib/jvm/default-java
	$ sudo ln -s /usr/lib/jvm/java-7-oracle /usr/lib/jvm/default-java


### Tomcat7インストール

	$ sudo aptitude install tomcat7 tomcat7-admin

「/etc/tomcat7/tomcat-users.xml」に以下の設定を追加します。

	$ sudo vi /etc/tomcat7/tomcat-users.xml
	-- snip --
	<role rolename="manager-gui"/>
	<role rolename="admin-gui"/>
	<user username="manager" password="manager" roles="manager-gui"/>
	<user username="admin"   password="admin"   roles="admin-gui"/>
	-- snip --

サービスの再起動。

	$ sudo service tomcat7 restart

## MySQLインストール

### MySQLインストール

	$ sudo aptitude install mysql-server

インストール途中に聞かれる「root」のパスワードは「alfresco」としました。

まずは「/etc/mysql/my.cnf 」の編集。デフォルト文字コードをUTF-8にします。これをおこなわないと初期処理で作成されるテーブルに文字コードが指定されないものがあり、日本語を含むファイル名のものをアップロードするとエラーになってしまいます。

	$ sudo vi /etc/mysql/my.cnf
	-- snip --
	[client]
	default-character-set = utf8
	-- snip --
	[mysqld]
	character-set-server = utf8
	-- snip --

編集後はサービスを再起動して設定を反映します。

	$ sudo service mysql restart

続いてデータベースと接続ユーザーの作成します。

	$ mysql -uroot -p
	mysql> create database alfresco;
	mysql> grant all privileges on alfresco.* to 'alfresco'@'localhost' identified by 'alfresco';
	mysql> flush privileges;
	mysql> exit

### MySQLコネクターインストール

	$ sudo aptitude install libmysql-java
	$ sudo cp /usr/share/java/mysql-connector-java-5.1.16.jar /usr/share/tomcat7/lib/
	$ sudo service tomcat7 restart

## Alfresco 4.2.cインストール

### アプリケーションの設定

以下の内容のファイルを「/etc/tomcat7/Catalina/localhost/alfresco.xml」に作成します。「参考にしたURL」にこの記述があったのですが、詳しい内容はわかりません。

	$ sudo vi /etc/tomcat7/Catalina/localhost/alfresco.xml

{% gist 4975316 %}

### ファイルの配置

	$ mkdir ~/alfresco
	$ cd ~/alfresco
	$ wget http://dl.alfresco.com/release/community/build-04576/alfresco-community-4.2.c.zip
	$ sudo aptitude install unzip
	$ unzip alfresco-community-4.2.c.zip
	$ sudo cp -r ~/alfresco/web-server/shared /var/lib/tomcat7
	$ sudo cp -r ~/alfresco/web-server/webapps /var/lib/tomcat7
	$ sudo cp -r ~/alfresco/web-server/lib /var/lib/tomcat7/shared/lib
	$ sudo cp -r ~/alfresco/bin /var/lib/tomcat7/bin
	$ sudo cp -r ~/alfresco/licenses /var/lib/tomcat7/licenses
	$ sudo cp -r ~/alfresco/README.txt /var/lib/tomcat7/README.txt
	$ sudo mv /var/lib/tomcat7/shared/classes/alfresco-global.properties.sample /var/lib/tomcat7/shared/classes/alfresco-global.properties
	$ sudo mv /var/lib/tomcat7/shared/classes/alfresco/web-extension/share-config-custom.xml.sample /var/lib/tomcat7/shared/classes/alfresco/web-extension/share-config-custom.xml

続いてAlfrescoデータ格納用ディレクトリを作成、パーミッションも変更します。

	$ sudo mkdir -p /opt/alfresco/data
	$ sudo chown -R tomcat7:tomcat7 /var/lib/tomcat7 /opt/alfresco

### Javaの設定変更

メモリ等の設定を変えるため、「/usr/share/tomcat7/bin/setenv.sh」を作成して以下の内容を記述します。

	$ sudo vi /usr/share/tomcat7/bin/setenv.sh
	CATALINA_OPTS="-Djava.awt.headless=true -Xmx2048m -XX:+UseConcMarkSweepGC -XX:MaxPermSize=512m -Xms128m -Dalfresco.home=/opt/alfresco -Dcom.sun.management.jmxremote -XX:+CMSIncrementalMode"

### alfresco-global.propertiesの変更

「/var/lib/tomcat7/shared/classes/alfresco-global.properties」を下記の通りに編集します。

{% gist 4975217 %}

編集後、Tomcatを再起動すればオッケーです。

	$ sudo service tomcat7 restart

## Shareにログイン

`http://HOST:8080/share/`にアクセスして、デフォルトで設定されているユーザー名`admin`、パスワード`admin`でログインします。

## 参考にしたURL

- http://www.whiteboardcoder.com/2013/01/install-alfresco-community-42-on-ubuntu_31.html
- http://paultiseo.wordpress.com/2012/06/27/installing-alfresco-community-4-on-ubuntu-server-12/
- http://blog.jeshurun.ca/technology/alfresco-on-ubuntu-complete-installation-guide
