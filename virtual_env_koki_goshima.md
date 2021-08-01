# Vagrantを用いた環境構築

## バージョン一覧
| 技術 | バージョン |
| :--- | :---: |
| PHP | 7.3 |
| Nginx | - |
| MySQL | 5.7 |
| Laravel | 6.0 |
| OS | CentOS7 |

## 環境構築手順
### CentOS7のダウンロード
1. vagrant box add centos/7を実行
2. 以下の選択肢で3を選択
```
1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```
3. `vagrant init centos/7`を実行してVagrantfileという設定ファイルを作成する
4. Vagrantfileの編集
    - `config.vm.network "forwarded_port", guest: 80, host: 8080`をコメントイン
        - ゲストOSに80番のポートでデータが来たらそれをホストOSの8080へ転送する設定
    - `config.vm.network "private_network", ip: "192.168.33.10"`をコメントイン
        - プライベートネットワークIPを192.168.33.10に設定
    - 以下の編集。ホストOSの`virtual_env_manual`ディレクトリとゲストOSの/vagrantディレクトリ内を同期するための設定
    ```
    config.vm.synced_folder "../data", "/vagrant_data"
    # ↓ 以下に編集
    config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
    ```
5. `vagrant up`でゲストOSの起動
6. `vagrant ssh`でゲストOSへログイン

### PHPのインストール
1. `sudo yum -y install epel-release wget`
    - `remi-release-7.rpm`と依存関係にある`epel-release`というパッケージをインストール
2. `sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm`
    - `remi-release-7.rpm`ファイルをダウンロード
3. `sudo rpm -Uvh remi-release-7.rpm`
    - `remi-release-7.rpm`をインストール
4. `sudo yum -y install --enablerepo=remi-php72 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip`
    - PHPのインストールと、PHPアプリケーションを動かす上で必要となるモジュールをインストール
5. composerのインストール
    - 以下の実行
    ```
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"
    ```
    - `sudo mv composer.phar /usr/local/bin/composer`を実行
        - composerコマンドがどのディレクトリでも使えるようにパスを通す

### Laravelのインストール
1. `vagrant init`を実行したディレクトリの直下に任意の名前でディレクトリを作成/移動
2. laravelをインストール
    - `composer create-project --prefer-dist laravel/laravel 1で作成したディレクトリ名 "6.0"`

### Nginxのインストール
1. `/etc/yum.repos.d/`に`nginx.repo`というファイルを作成
2. `nginx.repo`に以下を追記
```
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
3. `sudo yum install -y nginx`を実行してNginxをインストール
4. Nginxの設定ファイルを編集
    - /etc/nginx/conf.d/default.confを開く
    - 以下のように編集
    ```
    server {
    listen       80;
    server_name  192.168.33.10; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
    # ApacheのDocumentRootにあたります
    root /vagrant/laravel_app/public; # 追記
    index  index.html index.htm index.php; # 追記

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        #root   /usr/share/nginx/html; # コメントアウト
        #index  index.html index.htm;  # コメントアウト
        try_files $uri $uri/ /index.php$is_args$args;  # 追記
    }

    # 省略

    # 該当箇所のコメントを解除し、必要な箇所には変更を加える
    # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

    location ~ \.php$ {
    #    root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
        include        fastcgi_params;
    }

    # 省略
    ```
5. `/etc/php-fpm.d/www.conf`を以下のように編集
```
;24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```
6. nginx のユーザーでもlaravel_app下のログファイルへの書き込みができる権限を付与
```
cd /vagrant/laravel_app
$ sudo chmod -R 777 storage
```
7. `sudo systemctl start nginx`でNginxを起動
8. `sudo systemctl status nginx`でNginxが起動していることを確認


