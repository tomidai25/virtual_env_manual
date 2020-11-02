# バージョン一覧 
|PHP |Nginx |MYSQL |Laravel |0S |
|---|---|---|---|---|
|7.3 |1.19.3 |5.7 |6.0 |CentOS7 | 

# 環境構築の流れ 
## vagrantboxをダウンロードしてvagrant用のディレクトリを用意 
### 1.vagrantboxのダウンロード 
今いるディレクトリでvagrantboxをコマンド`vagrant box add centos/7`を実行する。 
コマンドを実行後、virtualboxの番号を選択してenterを押す。  
>Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!  

と表示されたら、ダウンロード完了。 

<br>

### 2.vagrant用ディレクトリの作成 
以下のどちらかのディレクトリに移動して`mkdir ディレクトリ名`を実行する。ディレクトリを移動する際、コマンド`pwd`で現在いるディレクトリを確認してコマンド`cd ディレクトリ名`で移動できる。  
例：`mkdir vagrant_markdown`,`cd vagrant_markdown`  
- 自分用の作業用ディレクトリ
- デスクトップ  

作成したディレクトリの中で`vagrant init centos/7`を実行すると、問題がなければ  
>A 'Vagrantfile' has been placed in this directory. You are now
>ready to `vagrant up` your first virtual environment! Please read
>the comments in the Vagrantfile as well as documentation on 
>'vagrantup.com` for more information on using Vagrant.  

が表示される。 

<br>

### 3.Vagrantfileの編集 
`vi vagrantfile`を実行して 
```nginx  
config.vm.network "forwarded_port", guest: 80, host: 8080 
config.vm.network "private_network", ip: "192.168.33.19"  
```  
上記の2箇所の`#`を外し、コメントアウトして下記の箇所は変更を加える。  
```shell  
config.vm.synced_folder "../data", "/vagrant_data" -> config.vm.synced_folder "./", "/vagrant", type:"virtualbox"   # 変更  
```  

注：ipの部分は指定されたipアドレスにする。  

<br>

### 4.vagrant プラグインのインストール 
```nginx  
vagrant plugin install vagrant-vbguest 
vagrant plugin list  
```  

上記の順にコマンドを実行して`vagrant-vbguest`がインストールされているか確認する。 

<br>

### 5.ゲストOSの起動 
vagrantfileのあるディレクトリで`vagrant up`を実行してvagrantを起動をする。  
作成したvarant用のディレクトリで`vagrant ssh`を実行してゲストOSにログインする。  
>Welcome to your Vagrant-built virtual machine.  
>[vagrant@localhost ~]$  

と表示されればゲストOSにログインできている。 

<br>

## ゲストOS内にPHPやパッケージをインストールして環境を整える。 
### 1.パッケージをインストール 
ゲストOSにログインした状態で`sudo yum -y groupinstall "development tools"`を実行してgitなどの開発に必要なパッケージを一括でインストールする。 

<br>

### 2.PHPをインストール 
Laravelを動作させるにはPHPのバージョンが7以上である必要がある。今回はPHPのバージョン7.3を以下のコマンドを実行してインストールする。  
```nginx
sudo yum -y install epel-release wget  
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm  
sudo rpm -Uvh remi-release-7.rpm  
sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip  
```  

`php -v`を実行してphpのバージョンが確認できたら、インストール完了。 

<br>

### 3.composerをインストール 
phpパッケージ管理ツールであるcomposerをインストールする。  
```nginx
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"  
php composer-setup.php  
php -r "unlink('composer-setup.php');"  
```  

`sudo mv composer.phar /usr/local/bin/composer`を実行して、どのディレクトリにいてもcomposerコマンドを使用を使用できるようfileの移動を行う。 
`composer -v`を実行してcomposerのバージョンが確認できたら、インストール成功。 

<br>

## ゲストOS内にNginx,MYSQLをインストール後、Laravelを動かす。 
### 1.composerコマンドを使用してLaravel6.0のプロジェクトを作成する。 
最初に、ゲストOSにログインしている場合は、`exit`を実行してログアウトする。 
次に、vagrant用ディレクトリ内で`composer create-project laravel/laravel --prefer-dist laravel_server 6.0`を実行する。 
laravel_serverの部分はディレクトリの名前を入力する。 
作成したLaravelプロジェクトのディレクトリで`php artisan ui vue --auth`を実行して認証機能を実装する。 

<br>

### 2.MYSQLをインストール 
`vagrant ssh`を実行してゲストOSにログインする。 
`cd vgrant/Laravel_server`を実行してLaravel_serverに移動した後、rpmに新たなリポジトリを追加しMYSQLをインストールする。 
```nginx
sudo wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm 
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm 
sudo yum install -y mysql-community-server  
```  

順にコマンドを実行した後に`mysql --version`を実行してMYSQLのバージョンが確認できたら、MYSQLの起動・接続を行う。  

<br>

### 3.MYSQLの起動・接続 
```nginx
sudo systemctl start mysqld
mysql -u root -p  
```  

上記の順でコマンドを実行すると、Enter password:が表示される。 
デフォルトでパスワードが設定されてしまっているので、パスワードを調べて接続しパスワードの再設定をする必要がある。  
`sudo cat /var/log/mysqld.log | grep 'temporary password'`を実行すると下記のようにパスワードが表示される。  
> 2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: `******`  

root@localhost:の後のランダムな文字列がパスワードとなる。  
パスワードを再設定する前に、以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更する。  
```shell
sudo vi /etc/my.cnf  
validate-password=OFF  # 追記  
```  

ファイルを編集後、`sudo systemctl restart mysqld`を実行してMYSQLを再起動する。  
再起動が完了したら再度`mysql -u root -p`を実行して、Enter password:にコピーしたパスワードをペーストしてMYSQLに接続する。  
接続した状態で`mysql > set password = "新たなpassword";`のコマンドでパスワードを設定する。  
注：この方法はセキュリティ的によろしくはないため本番環境と呼ばれる環境でこの方法で再設定するのは避ける。  

<br>

### 4.データベースの作成 
`mysql > create database laravel_markdown;`を実行してデータベースの作成を行う。Query OKと表示されたら作成は完了。  

<br>

## NginxをインストールしてLaravelを動かす。 
### 1.Nginxをインストール  
`sudo vi /etc/yum.repos.d/nginx.repo`を実行してファイルを作成する。  
作成したファイルに以下の内容を書き込む。  
```nginx
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1  
```  

書き終えたら、保存して以下のコマンドを実行しNginxのインストールを実行する。  
```nginx
sudo yum install -y nginx  
nginx -v  
```  

Nginxのバージョンが確認できたら、以下のコマンドを実行してNginxを起動する。  
`sudo systemctl start nginx`  
ブラウザ上で`http://192.168.33.19`でアクセスする。NginxのWelcomeページが表示されれば、問題なく動いているのでLaravelを動かす作業に入る。  
注：vagrantfileでipを変更している場合は、変更したipアドレスを入力する。  

<br>

### 2.Laravelを動かす 
まず、作成したLaravelプロジェクトのディレクトリ下の.envファイルの内容を以下に変更する。  
```nginx
DB_PASSWORD= -> DB_PASSWORD=登録したパスワード  #変更
DB_DATABASE=作成したデータベースの名前(Laravel_markdown)  #変更  
```

変更が完了したら`php artisan migrate`を実行する。  
次に、Nginxには設定ファイルが存在しているので編集を行う。Nginxは、php-fpmとセットで使用する。  
使用しているOSがCentOSの場合、`/etc/nginx/conf.d`ディレクトリ下の`default.conf`ファイルが設定ファイルとなる。  
まずはNginxの設定ファイルを編集していく。  
`sudo vi /etc/nginx/conf.d/default.conf`でファイルを開き、  
serverの中で
```nginx
server_name=192.168.33.19;  # vagrantfileでコメントを外した箇所のipアドレスを記述する。
root index  index.html index.htm index.php;  # 追記  
index  index.html index.htm index.php;  # 追記  
```  

location / の中で
```nginx  
root   /usr/share/nginx/html; # コメントアウト
index  index.html index.htm; # コメントアウト
try_files $uri $uri/ /index.php$is_args$args; # 追記 
```  

location ~ \.php$の中で
```nginx
root  html;   # rootのみコメントアウト
fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # 変更
```  
次にphp-fpm の設定ファイルを編集する。  

`sudo vi /etc/php-fpm.d/www.conf`でファイルを開き、以下２つを変更する。  
```shell
user = apache -> user = nginx # 変更
group = apache -> group = nginx # 変更
``` 

ファイルの編集が完了したら、以下の順にコマンドを実行してNginxを再起動する。  
```nginx
sudo systemctl restart nginx
sudo systemctl start php-fpm  
```  

ブラウザ上で`http://192.168.33.19`を入力すると` Permission denied`のエラーが表示される。  
これはphp-fpmの設定ファイルのuserとgroupをnginx に変更したが、ファイルとディレクトリの実行useとgroupにnginxが許可されていないため起きているエラーなので試しに以下のコマンドを実行する。  
`ls -la ./ | grep storage && ls -la storage/ | grep logs && ls -la storage/logs/ | grep laravel.log`  
出力結果から、storageディレクトリもlogsディレクトリもlaravel.logファイルも全てuserとgroupがvagrantとなっているので、  
これではnginxというユーザーの権限をもってlaravel.logファイルへの書き込みができないので以下のコマンドを実行してnginxというユーザーでもログファイルでの書き込みができる権限を付与する。  
```nginx  
cd /vagrant/laravel_server  
sudo chmod -R 777 storage  
sudo chown vagrant:vagrant /var  
```  
chmodコマンドで読み書きの権限を付与をして、chownコマンドでユーザーとグループを変更するコマンドです。  
再度ブラウザ上で`http://192.168.33.19`でアクセスして403エラーが出た時は以下の作業を行う。  
`sudo vi /etc/selinux/config`でファイルを開き、  
`SELINUX=disabled`と書き換える。  
注：今回はローカル環境なので無効にしてしまっても問題ないが、セキュリティ的によろしくはないため本番環境と呼ばれる環境でこの方法で再設定するのは避けて別のアプローチが必要。  
設定を反映させるためにゲストOSを再起動する必要があるので、ゲストOSをから一度ログアウトして以下のコマンドを実行する。  
```nginx  
exit  
vagrant reload  
```  

リロードが完了したら以下のコマンドを実行して、再度ゲストOSにログインしてNginxを起動する。  
```nginx  
vagrant ssh  
sudo systemctl restart nginx  
sudo systemctl start php-fpm  
```  

ブラウザ上でユーザーの登録・ログインができればローカルで動かしていたLaravelを仮想環境上で全く同じように動かすことができたということになる。  

<br>

# 環境構築の所感  
`vagrant up`をする時にportが被ってしまっていると実行が失敗してしまうので、被っているportを探して`kill`コマンドを使って消去する必要がある。  
今回、ホストOSでMYSQLのバージョンを確認してしまったことが原因でゲストOSにMYSQLがインストールし忘れていることに気づかず、ブラウザ上でLaravelのログイン・ユーザー登録をした際にデータベースが存在しないというエラーが出てしまうことがあったのでホストOSでの作業とゲストOSでの作業は混ざらないように気をつけなければならい。  
`chmod -R 777 storage` コマンドはゲストOSからログアウトしてホストOSでも実行可能。  
今回、イレギュラーなエラーの対応の為、一度vagrantとNginxを停止してから再起動をかけたことによりyes/noを求められたのでNginxを起動させる為、全てyesを入力した。  

<br>

# 参考サイト  
[Laravel6.x認証](https://readouble.com/laravel/6.x/ja/authentication.html)  
[Giztech](http://giztech.gizumo-inc.work/categories/18/222)  
[ももいろテクノロジー](http://inaz2.hatenablog.com/entry/2013/04/16/222440)  
