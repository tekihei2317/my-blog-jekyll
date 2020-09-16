---
layout: posts
title: Ubuntuでrails newした後にrails db:createやrails s等をすると、Access denied for user 'root'@'localhost'が出る
category: Rails
---

## 環境

* Ubuntu 18.04
* MySQL 5.7.31

## 状況

rails newした直後に、rails db:createを実行したり、rails sを実行後localhost:3000にアクセスすると、Access denied for user 'root'@'localhost'というエラーが出る。

## やったこと

とりあえずMySQLが起動していることを確認します。

{% highlight shell %}

tekihei:~ $ systemctl status mysql
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-09-16 08:06:27 JST; 5h 51min ago
  Process: 25754 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid (code=exited, status=0/SUCCESS)
  Process: 25745 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 25756 (mysqld)
    Tasks: 28 (limit: 4306)
   CGroup: /system.slice/mysql.service
           └─25756 /usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid

 9月 16 08:06:26 tekihei-ThinkPad-X250 systemd[1]: Starting MySQL Community Server...
 9月 16 08:06:27 tekihei-ThinkPad-X250 systemd[1]: Started MySQL Community Server.

{% endhighlight %}

Active: activeとなっているので、起動しているっぽいです。


MySQLに入ってみます。

{% highlight shell %}

tekihei:~ $ mysql -u root -p
Enter password: 
ERROR 1698 (28000): Access denied for user 'root'@'localhost'

{% endhighlight %}
Access deniedされました。

{% highlight shell %}
tekihei:~ $ sudo mysql -u root
[sudo] password for tekihei: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 5.7.31-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
{% endhighlight %}
こっちだと入れました。

何かここがおかしい気がしたので調べてみると、この二つのコマンドでユーザー認証の仕方が違うようです。

現在のrootユーザーの認証方法を確認してみると、auth_socketというものが使われているようです。

{% highlight shell %}

mysql> select User, plugin, authentication_string from mysql.user;
+------------------+-----------------------+-------------------------------------------+
| User             | plugin                | authentication_string                     |
+------------------+-----------------------+-------------------------------------------+
| root             | auth_socket           |                                           |
| mysql.session    | mysql_native_password | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys        | mysql_native_password | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| debian-sys-maint | mysql_native_password | *7C5D7A711BA14DD755BA0480832042D601D8E025 |
+------------------+-----------------------+-------------------------------------------+
4 rows in set (0.00 sec)

{% endhighlight %}

## 解決方法

rootの認証方法をauth_socketからmysql_native_passwordに変えます。

'mysql -u root -p'ではmysql_native_password、'sudo mysql -u root' ではauth_socketが認証方法として使われるようです。railsはMySQLに入るときに、mysql -u root -pコマンド(か、それに似たもの)を使うっぽいです。なのでrootの認証方法をmy_sql_native_passwordに変えれば、'mysql -u root -p'でMySQLに入ることができるようになり、Railsのエラーも解決できます。

認証方法を変えるには、ALTER USER文を使います。

{% highlight shell %}
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
Query OK, 0 rows affected (0.00 sec)
{% endhighlight %}

あとはrailsプロジェクトのconfig/database.ymlに設定したパスワード(今回は'password')を指定すればよいです。

{% highlight yaml %}
default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: password
  host: localhost
  # socket: /tmp/mysql.sock
{% endhighlight %}

これでrails db:createとrails sが実行できるようになりました。

## 参考

* [Ubuntu 18.04 (Bionic Beaver) に MySQL をインストールする](http://mosyoesyoe.hatenablog.jp/entry/2018/05/08/160820)
* [MySQLのrootでログインできない：ubuntu 18.04 Server とMysql5.7](http://mosyoesyoe.hatenablog.jp/entry/2018/05/08/160820)

## まとめ

環境構築でこんなにハマるとは思いませんでした。でも解決できたときは気持ちよかったです。MySQLに限らず分からないことがたくさんあるなぁと思いました。デプロイのときもハマりそうです。