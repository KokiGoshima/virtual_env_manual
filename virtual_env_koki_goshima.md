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


