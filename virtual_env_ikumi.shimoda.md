# 使用技術一覧
|  OS  |  DB  |  Webサーバ  |  サーバ言語  |  フレームワーク |
|  :--: | :--: | :--: | :--: | :--: |
|  CentOS7  |  MySQL5.7  |  Nginx  |  PHP7.3  |  laravel6.0  |

***  

#　環境構築の流れ

## OSの設定  
- 初めに、vagrantの作業用ディレクトリを作成します。  
    ``$ mkdir vagrant_laravel``  

- 続いて、ゲストOSの設定。今回はLinuxのCentOSのバージョン7のbox名 centos/7 を指定して実行。  
  ``$ vagrant box add centos/7``  

- コマンドを実行すると、下記の選択肢が表示されるので、3を選択肢、enterを押します。  
  ```1) hyperv  2) libvirt  3) virtualbox  4) vmware_desktop```

- 下記のように表示されれば、OSの設定は完了です。  
  ``Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!``  

***
## vagrantの使用
- 先ほど作成した、vagrant_laravelディレクトリに移動します。  
    ``$ cd vagrant_laravel``
  
- このディレクトリでVirtualBoxを使用します。  
    ``$ vagrant init centos/7``  

- これで、vagrant_laravel直下にVagrantfileが作成されたと思います。  
  次に、インターネット接続を行うために、テキストエディタでvagrantfileを編集します。  
    変更点① #を外してコメントイン。  
    ``# config.vm.network "forwarded_port", guest: 80, host: 8080``

    変更点②   
    ``# config.vm.network "private_network", ip: "192.168.33.10"``  
    ↓以下に編集。  
     ``config.vm.network "private_network", ip: "192.168.33.19"``  

    変更点③  
    ``# config.vm.synced_folder "../data", "/vagrant_data"``  
    ↓以下に編集。  
    ``config.vm.synced_folder "./", "/vagrant", type:"virtualbox"``

  > 補足説明
    - 変更点①  
    今回はホストOSのポート8080番を使用します。起動が上手くいかない場合は、既に8080番が使用されている可能性があるので、調べて該当のポートを開放してください。  
    - 変更点②  
    今回はプライベートネットワークを192.168.33.19に変更しましたが、過去にVagrantを使用したことがありプライベートネットワークIP 192.168.33.19 が既に使われている場合、別のipアドレスを設定してください。  
    - 変更点③  
    ./ はカレントディレクトリ(vagrant_laravel)を示しており、ホストOSのvagrant_laravelディレクトリ内とゲストOS (Vagrant) の /vagrant のディレクトリ内をリアルタイムで同期するための設定です。

- プラグインのインストール  
    vagrant-vbguest(VirtualBoxのバージョンに合わせて最新化してくれるプラグイン)  
    ``$ vagrant plugin install vagrant-vbguest --plugin-version 0.21  ``

    Saharaをインストール(構築中の仮想の状態を保存・巻き戻しすることができるプラグイン)  
    ``$ vagrant plugin install sahara``  
    Saharaを有効化。  
    ``$ vagrant sandbox on``　

  > プラグインの確認   
    インストールしたプラグインは``vagrant plugin list``で確認が出来ます。  
    今回は、二つのプラグインをインストールしているので、  
    sahara (0.0.17, global)  
    vagrant-vbguest (0.29.0, global)  
    このように表示されていると思います。

- ゲストOSの起動  
    ``$ vagrant up``コマンドでゲストOSを起動します。  

- ゲストOSへのログイン  
    ``$ vagrant ssh``  

- ログインに成功すると、以下の文言が表示されます。  
    ``[vagrant@localhost ~]$``  
***
## PHPのインストール
- パッケージのインストール  
構築されたゲストOSには、まだ開発に必要となるソフトウェアやコマンドなどがインストールされていない状態です。  
先ほどログインした、ゲストOSの中で以下のコマンドを実行し、パッケージをインストールしていきます。  
```
$ sudo yum -y groupinstall "development tools"  
$ sudo yum -y install epel-release wget  
$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm  
$ sudo rpm -Uvh remi-release-7.rpm  
```

- 次は、PHPのインストールを行います。  
``$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip``  
- バージョンの確認  
 ``$ php -v``  
 以下のように、バージョン7.3が確認出来れば完了です。  
```
PHP 7.3.27 (cli) (built: Feb  2 2021 10:32:50) ( NTS )  
Copyright (c) 1997-2018 The PHP Group  
Zend Engine v3.3.27, Copyright (c) 1998-2018 Zend Technologies
```
  > 補足説明  
  > - sudoとはrootユーザー(システム管理者)の権限を借りるコマンドです。  
  > vagrant sshを実行してゲストOSにログインした場合、ログインユーザーはvagrantとなります。  
  > 今回のように、vagrantユーザーがrootユーザーにしか許されていない操作を実行する際には、sudoを使ってrootユーザーの権限を一時的に借りる必要があります。  
  > - yumとは、CentOSを代表とするLinuxOSのRedHat系ディストリビューションと呼ばれる種類のOSで使用されているパッケージ管理ツール・コマンドです。  
  > -yオプションをつけることで、インストール中に聞かれるyes/noをすべてyesと答えてくれるため、処理を止めずにインストールすることが出来ます。

- composerのインストール  
  次に、PHPのパッケージ管理ツールであるcomposerをインストールしていきます。  
```
  php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"  
  php composer-setup.php  
  php -r "unlink('composer-setup.php');
```  
  インストールが終わったら、すべてのディレクトリからcomposerコマンドを実行できるようにファイルを移動します。 
  ``sudo mv composer.phar /usr/local/bin/composer`` 

- バージョンの確認  
  ``$ composer -v``  
  以下のように確認出来れば完了です。  
  ``Composer version 2.0.11 2021-02-24 14:57:23``  
***
## laravelの作成  
- ゲストOSのvagrant直下にtest_appという名前のプロジェクトを作成します。  
  ``composer create-project --prefer-dist laravel/laravel test_app "6.*"``  
- インストールの確認  
  ``$ cd test_app``  
  ``$ php artisan -v``  
  このように、バージョンが表示されれば完了です。  
- Authの実装  
  laravel6.0以降はlaravel/uiを使っています。  
  ``$ composer require laravel/ui:^1.0 --dev``  
  ``$ php artisan ui bootstrap --auth``  
  これでlaravelプロジェクトは作成完了です。  
***
## Nginxの設定  

- 最新バージョンのインストールの前に、下記のコマンドでNginxのファイル編集します 。  
	``$ sudo vi /etc/yum.repos.d/nginx.repo``  
- 追記内容。  
```
	[nginx]  
	name=nginx repo  
  baseurl=https://nginx.org/packages/mainline/centos/\$releasever\$basearch/  
	gpgcheck=0  
	enabled=1  
```
- 編集を終えたら、下記のコマンドでNginxをインストール。  
	``$ sudo yum install -y nginx``  
- バージョンの確認  
	``$ nginx -v``  
  インストールが完了されていれば、下記が表示されます。  
  ``nginx version: nginx/1.19.8``  

- Nginxの起動  
  ``$ sudo systemctl start nginx``  
  [http://192.168.33.19](http://192.168.33.19)を入力して、NginxのWelcomeページが表示されればOKです。  
  
  **Nginxがインストールされない場合**  
  ``baseurl=https://nginx.org/packages/mainline/centos/\$releasever\$basearch/``  
  ↓下記に編集します。  
  ``baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/``  

- test_appを表示させるために、Nginxの設定ファイルを編集。  
```
  $ sudo vi /etc/nginx/conf.d/default.conf
　server ｛  
  listen       80;  
  server_name  192.168.33.19; # Vagranfileで設定したipアドレスを記述してください。  
  root /home/vagrant/test_app/public; # 追記  
  index  index.html index.htm index.php; # 追記  

  #charset koi8-r;  
  #access_log  /var/log/nginx/host.access.log  main;  

  location ｛
      #root   /usr/share/nginx/html; #コメントアウト  
      #index  index.html index.htm;  #コメントアウト  
      try_files $uri $uri/ /index.php$is_args$args;  # 追記  
  ｝  

  location ~ \.php$ ｛  
      #root           html;*  
      fastcgi_pass   127.0.0.1:9000;  #コメントイン  
      fastcgi_index  index.php;  #コメントイン  
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;   #コメントインして$fastcgi_script_name以前を /$document_root/に変更  
      include        fastcgi_params;   #コメントイン  
  ｝
```
- php-fpmの設定ファイルを編集  
	``$ sudo vi /etc/php-fpm.d/www.conf``  
	24行目付近  
	``user = apache``  
	↓以下に編集  
	``user = nginx``  

	``group = apache``  
	↓以下に編集  
	``group = nginx``  

## DBの設定  
- MySQLのインストール。
```  
	$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm  
	$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm  
	$ sudo yum install -y mysql-community-server  
```
- バージョンの確認。  
	``$ mysql --version``  
  以下の文章が表示されます。    
  ``mysql  Ver 14.14 Distrib 5.7.33, for Linux (x86_64) using  EditLine wrapper``  

- パスワードの確認。    
  mysqlを起動します。  
	``$ sudo systemctl start mysqld``  
  接続前にパスワードを確認します。  
	``$ sudo cat /var/log/mysqld.log | grep 'temporary password'``  
  2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge  

  hogehogeと表示されている箇所がパスワードとなっているので、こちらをコピーしておきます。  

- パスワードの変更。  
  しかし、このままではパスワードポリシーが厳格で面倒なので、シンプルなパスワードに設定できるように、ファイルを編集します。  

  $ sudo vi /etc/my.cnf  
  [mysqld]  
  datadir=/var/lib/mysql  
  socket=/var/lib/mysql/mysql.sock  

  上の二行に続いて下記の一行を追加  
  ``validate-password=OFF``   

  編集後はMySQLサーバの再起動を行います。  
  ``$ sudo systemctl restart mysqld``  

- mysqlにログインを実行。  
   ``$ mysql -u root -p``  
	Enter password:と表示されたらコピーしてあるパスワードを入力します。  
  
- 次に接続した状態でpasswordの変更を行います。  
  ``mysql > set password = "新たなpassword";``   

- データベースの作成。  
  laravelのtodoアプリに必要なデータベースの作成を行います。  
  ``mysql > create database test_app;`` 

  ここまで完了したら、一度exitをして、test_appディレクトリ下の.envファイルの内容を以下に変更してください。  
  ``DB_DATABASE=laravel``  
    ↓以下に編集  
  ``DB_DATABASE=test_app``  
  
  ``DB_PASSWORD=``  
  ↓ 以下に編集  
  ``DB_PASSWORD=登録したパスワード``  
  これで、次回から設定した新しいパスワードでログインが可能です。  

- マイグレーションの実行  
  ``$ cd test_app``  
  ゲストOS上でtest_appに移動し  
  ``php artisan migrate``を実行します。  
  マイグレーションが実行されれば、データベースの設定は完了です。  

## ブラウザで確認  
- Nginxの起動  
  ``$ sudo systemctl start nginx``  

- php-fpmの起動  
   ``$ sudo systemctl start php-fpm``  

   **403Forbiddenが表示された場合**  
   - SELinuxを無効  
   ``$ sudo vi /etc/selinux/config``  
   ``SELINUX=enforcing``  
   ↓以下に編集  
   ``SELINUX=disabled``  
   ``exit``をしてから、ホストOSで``vagrant reload``その後、Nginxの起動コマンドへ。

-  [http://192.168.33.19](http://192.168.33.19)をブラウザで入力して確認。  
  laravelのwelcomページが表示されれば完了です。  
***
# 環境構築の所感
- コマンドを実行した際に、その都度バージョンの確認や、起動確認を行わないと、後々不具合を起こす可能性があるという重要性を学んだ。    
- コマンドを実行するティレクトリが違うと、挙動が大きく変わってしまう。
- エラーの対処をするコマンドを調べても、ゲストOSとホストOSどちらで実行するか迷うことがあったので、双方がどんな動きをしているかを把握しないといけない。  
- 権限によっても実行できるもの、出来ないものが分かれている。sudoを使うことで、管理者権限を借りることが出来る。  
***
# 参考サイト
[MarkdownでTableを書く](https://notepm.jp/help/markdown-table/)  
[linuxコマンドリファレンス](http://www.redout.net/data/command.html/)  
[CentOS7のPHPを5.6／7.0／7.1／7.2／7.3系にバージョンアップする](https://qiita.com/heimaru1231/items/84d0beca81ca5fdcffd0/)  
[VagrantにSaharaを導入](https://qiita.com/sudachi808/items/09cbd3dd1f5c25c23eaf)  
[【まとめ】Vagrant コマンド一覧](https://qiita.com/oreo3@github/items/4054a4120ccc249676d9)
[Vagrant ファイル共有　決定版](https://qiita.com/oreo3@github/items/4054a4120ccc249676d9))  
[CentOS7でyumでnginxの最新版が入らない時](https://qiita.com/awm-kaeruko/items/860651148974ec07b7bb)  
[Nginxで403 Forbiddenが表示された時のチェックポイント5選](https://engineers.weddingpark.co.jp/nginx-403-forbidden/)
[【Laravel「laravel-ui」について](http://www.code-magagine.com/?p=10606)