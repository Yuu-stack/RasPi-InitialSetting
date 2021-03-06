# RasPi-InitialSetting 

#このページでできる事  
簡易設定  //2020/12/3 シンプル化  
piユーザーを新規ユーザーに置き換え  
hostnameの変更  
NASのマウント (おまけ)

# 簡易設定  

日本時間に合わせる  
日本語(en_USとja_JP)を有効にする-->これはなに？？  
localeをja_JP.UTF-8に変更する  
Wi-Fi利用国に日本を設定する  
リポジトリを国内のミラーサイトに変更する  
aptアップデート&アップグレード  
nasで使うファイルシステム（NTFS, exFat）をインストール  
インストールされている Vim の確認   
Vim-tiny のアンインストール  
通常のvim をインストール  
Vimのカスタマイズ(.vimrc)設定  
githubアカウントに登録している公開鍵を raspiに登録  
IP固定化 (192.168.0.2/24)  
hostnameの変更 (raspi1)  

    sudo raspi-config nonint do_change_timezone Asia/Tokyo && \
    sudo raspi-config nonint do_change_locale ja_JP.UTF-8 && \
    sudo raspi-config nonint do_wifi_country JP && \
    sudo sed -i "s/deb.debian.org\/debian /ftp.jp.debian.org\/debian /g" /etc/apt/sources.list && \
    sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && \
    sudo apt install -y exfat-fuse exfat-utils ntfs-3g && \
    dpkg -l | grep vim && \
    sudo apt --purge remove -y vim-common vim-tiny && \
    sudo apt install vim-gtk -y && sudo cp /etc/vim/vimrc /etc/vim/vimrc.bak && \
    wget -O .vimrc https://gist.github.com/Yuu-stack/afc3644c76d10dc39bd4c0ad48a0bc86/raw/6ca7b465dae295db9789a4bdd6806a1629610d11/.vimrc && \
    sudo cp ~/.vimrc /etc/vim/vimrc && \
    ssh-import-id gh:Yuu-stack && \
    echo 'interface eth0' | tee -a /etc/dhcpcd.conf && \
    echo 'static ip_address=192.168.0.2/24' | tee -a /etc/dhcpcd.conf && \
    echo 'static routers=192.168.0.1' | tee -a /etc/dhcpcd.conf && \
    echo 'staitc domain_name_servers=192.168.0.1' | tee -a /etc/dhcpcd.conf && \
    sudo raspi-config nonint do_hostname raspi1 && \
    sudo reboot
 
# piユーザーを新規ユーザーに置き換え  

#ユーザ名の変更  
以下手順です。ユーザー名変更のため別のユーザを作成し、管理者権限で実行する必要があります。  

01."pi"を操作するユーザーを作成  
02.このユーザにsudo権限を与える  
03.自動ログインユーザー(pi)を削除する  
04.このユーザーでログインし直す  
05."pi"ユーザー名を変更する  
06.ホームディレクトリ、グループ名を変更する  
07.作成したユーザーを削除する  


    # 操作用ユーザー作成(例: tmp)
    pi@raspberrypi:~ $ sudo adduser tmp
    
    # sudoグループに追加
    pi@raspberrypi:~ $ sudo gpasswd -a tmp sudo
    
    # 自動ログインユーザー設定を削除する
    $ sudo vim /etc/lightdm/lightdm.conf 
      autologin-user=pi
      #autologin-user=pi
    
    $ sudo vim /etc/systemd/system/autologin@.service 
      # ExecStart=-/sbin/agetty --autologin pi --noclear %I $TERM
      ExecStart=-/sbin/agetty --autologin tmp --noclear %I $TERM
    
    # 作成したユーザー(tmp)でログインする(ログインユーザーを変更する)
    pi@raspberrypi:~ $ exit
    mac@:~ $ ssh tmp@raspberrypi.local
    
    # 以下コマンドの後、Boot Options -> Desktop / CLI -> Console Autoの順に選択しユーザーがtmpに変更されたのを確認、その後自動でrebootされる。
    tmp@raspberrypi:~ $ sudo raspi-cofig
    参考:http://teppodone.hatenadiary.jp/entry/HowToChangeNameOfPi
    
    # who　を実行しpiユーザーのプロセスがない事を確認
    tmp@raspberrypi:~ $ who
    tmp     tty1         2020-08-08 01:09
    tmp     pts/0        2020-08-08 01:09 (192.168.10.13)
    
    # ユーザー名"pi"を変更する(例: apple)
    tmp@raspberrypi:~ $ sudo usermod -l apple pi
    
    # ホームディレクトリを更新する
    tmp@raspberrypi:~ $ sudo usermod -d /home/apple -m apple
    
    # グループ名を更新する "エーリアス問題未解決"
    tmp@raspberrypi:~ $ sudo groupmod -n apple pi
    

    #更新したユーザー名でログイン後、ユーザー(apple)パスワードの変更
    apple@raspberrypi:~ $ sudo passwd apple
    
    # 自動ログインユーザーを再設定する
    
    $ sudo vim /etc/lightdm/lightdm.conf 
      #autologin-user=pi
      autologin-user=apple
      
    $ sudo vim /etc/systemd/system/autologin@.service 
      # ExecStart=-/sbin/agetty --autologin tmp --noclear %I $TERM
      ExecStart=-/sbin/agetty --autologin apple --noclear %I $TERM
    
    再起動し ユーザーアカウント(apple)でログインする
      
    # 以下コマンドの後、Boot Options -> Desktop / CLI -> Console Autoの順に選択しユーザーがappleに変更されたのを確認する。
    
    apple@raspberrypi:~ $ sudo raspi-cofig
    参考:http://teppodone.hatenadiary.jp/entry/HowToChangeNameOfPi
    
    # ユーザー"tmp"を削除
    apple@raspberrypi:~ $ sudo userdel tmp
    
    

# EX.番外編　NASマウント  

下記コマンドを使用し ラズパイの /mnt/ に NAStest フォルダーを作成し 今回は`HOME-NAS/` 以下の `NAS01` をマウントしていきます。  
`//192.168.10.xxx/HOME-NAS/` には ディレクトリ以下に NAS01,NAS02,NAS03 があると仮定しています。  
各自のNAS直下に存在するディレクトリ(フォルダ📁)を使用してください。  

    $ sudo mkdir /mnt/NAStest && sudo chmod 755 /mnt/NAStest

    $ sudo mount -t cifs //192.168.10.xxx/HOME-NAS/NAS01 /mnt/NAStest --verbose -o username=hogehogeuser,password=hogehogepass,uid=1000,gid=1000,file_mode=0666,dir_mode=0755,iocharset=utf8,defaults,vers=3.0
    
hogehogeuserとhogehogepasswordは各自NASへログインする際に必要な物に変更してください。 
設定してない方はこの箇所のみ削除し実行してください。  
確認してませんが下記コマンドでマウントできると思います。  
参考にしてください。 
    
    $ sudo mount -t cifs //192.168.10.xxx/HOME-NAS/NAS01 /mnt/NAStest --verbose -o uid=1000,gid=1000,file_mode=0666,dir_mode=0755,iocharset=utf8,defaults,vers=3.0
    
    
ここで`uid=xxxx,gid=xxxx`の値は `$ id` コマンドで確認した使用しているユーザーの値になります。  

    pi@raspberrypi:~ $ id
    uid=1000(pi) gid=1000(pi) groups=1000(pi)


マウント時、下記の様な mount error が出る場合があります。  

    mount error: cifs filesystem not supported by the system
    mount error(19): No such device
    Refer to the mount.cifs(8) manual page (e.g. man mount.cifs)
    
この場合は　https://x.momo86.net/?p=41  
を参考にし下記コマンドを実行、再起動してください。  

    $ modprobe cifs 
    $ sudo reboot

再度実行するとマウントできる様になっているはずです.  
概要を確認できてませんが、マウントする際に必要な`cifs`が 実機で使用してるカーネル側で有効になっていない時に発生するエラーの様です.  
`apt upgrade`などでカーネルがアップデートされた際(最新のカーネル,対応していないカーネルで)稀に出ます。  

参考:CIFSの導入について https://armadillo.atmark-techno.com/forum/armadillo/1767  


#マウントできる事を確認できたら起動時に自動でマウントできる様に設定していきます。  

# NASマウント自動起動の設定 

 `$ sudo vim /etc/fstab`で下記追加  

    //192.168.10.xxx/HOME-NAS/NAS01 /mnt/NAStest cifs username=hogehogeuser,password=hogehogepass,uid=1000,gid=1000,file_mode=0666,dir_mode=0755,iocharset=utf8,defaults 0 0 
 
vimを使用している方であれば esc key +`:%s/hogehogeuser/yomogi` の様に `:%s/変更したい文字列/変更後文字列`で書き換えができます.  

> <s>`$ sudo raspi-config`から`2 Network Options` >> で接続されるまで待機をオンにしておきます。  </s>
    
#ネットワークサービス起動後にマウントするように設定  
通信可能な状態になってからNASのマウントを行わないと失敗するの下記設定を行います。  
`$ sudo raspi-config`

3 Boot Options  >>  B2 Wait for Network at Boot Choose.. >> yes >> OK.  
上記の様に選択し、`$ df`コマンドを使用しマウントされているか確認.  

    $ sudo reboot
    $ df 

/mnt/NAStest に //192.168.10.xxx/HOME-NAS/NAS01 がマウントされていることを確認し終了です！  
これでエラーが表示されずマウントできていることが確認できたら起動時に自動マウントされるようになります。  

# 最後に

お疲れ様でした！早く設定を自動化し楽したいですね😭  




