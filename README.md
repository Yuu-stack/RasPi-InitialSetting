# RasPi-InitialSetting

下記作成のimgを使用  
 https://github.com/Yuu-stack/RasPIOS-Custom/blob/master/README.md  
 
# このページでできる事  
01.簡易設定  
02.アップデート  
03.vimのインストールと設定  
04.piユーザーを新規ユーザーに置き換え  
05.hostnameの変更  

# 01.簡易設定  

#日本時間に合わせる  
`$ sudo timedatectl set-timezone Asia/Tokyo`

#日本語(en_USとja_JP)を有効にする  

    $ sudo localedef -f UTF-8 -i en_US en_US
    $ sudo localedef -f UTF-8 -i ja_JP ja_JP

#localeを変更する  
`$ sudo localectl set-locale LANG=en_US.utf8`

# 02.aptアップデート  

`$ sudo apt update && sudo apt upgrade`

# 03.vimのインストールと設定  

#インストールされている Vim の確認  
`$ dpkg -l | grep vim`

#Vim-tiny のアンインストール  
`$ sudo apt --purge remove -y vim-common vim-tiny`

#通常のvim をインストール  
`$ sudo apt install -y vim-gtk`

#Vimのカスタマイズ(.vimrc)設定  
`$ wget https://gist.github.com/Yuu-stack/afc3644c76d10dc39bd4c0ad48a0bc86/raw/89ce616ce37b991b9ccb95addda7d84da084d974/.vimrc && sudo cp ~/.vimrc /etc/vim/vimrc`

/etc/vim/vimrc に存在する元のファイルは消して問題ないと思います。  
削除、上書きは適宜行ってください。  

# 04.piユーザーを新規ユーザーに置き換え  

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
    
    

# 05.hostnameの変更  

`$ sudo raspi-config`から変更するのが一番楽かと  

下記でも変更できる.  

    sudo vim /etc/hostname
    sudo vim /etc/hosts

# EX.番外編　NASマウント  

下記コマンドを使用し /mnt/ に NAS01 フォルダーを作成し `HOME-NAS/` 以下の `NAS01` をマウントしていきます。  
`//192.168.10.xxx/HOME-NAS/` には ディレクトリ以下に NAS01,NAS02,NAS03 があると仮定しています。  

    $ sudo mkdir /mnt/NAS01 && sudo chmod 755 /mnt/NAS01

`$ sudo mount -t cifs //192.168.10.xxx/HOME-NAS/NAS01 /mnt/NAS01 --verbose -o username=********,password=**********,uid=1000,gid=1000,file_mode=0666,dir_mode=0755,iocharset=utf8,defaults,vers=3.0`


この時、下記の様な mount error が出る場合があります。  

    mount error: cifs filesystem not supported by the system
    mount error(19): No such device
    Refer to the mount.cifs(8) manual page (e.g. man mount.cifs)
    
この場合は　https://x.momo86.net/?p=41　を参考にし下記コマンドを実行、再起動してください。  
`$ modprobe cifs && sudo reboot`

再度実行するとマウントできる様になっているはずです。


# 最後に

お疲れ様でした！早く自動化したい😭  




