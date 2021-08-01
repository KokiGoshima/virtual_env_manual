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
4. `sudo systemctl start nginx`でNginxを起動
5. `sudo systemctl status nginx`でNginxが起動していることを確認


