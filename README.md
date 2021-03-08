# Linux系システムのバックアップ方法（Raspberry piでルーターを構成した際の方法)

## Raspberrypi OSに対してパッケージのアップグレードやシステムの変更をする際は、**必ずバックアップイメージを作成してから行うこと**
アップグレードをした後にシステムが動作しなかった時に、すぐに動作できる状態に戻すことが出来るようにするため。   
新しいバージョンにするときは、数日動作できることを確認した後にバックアップする。   

**注意：ddコマンドを使用したバックアップは、入力デバイスと出力ファイル名を間違えるとバックアップファイルまたはデバイスのシステムを破損する可能性があるため、入出力名は注意して入力すること**

## バックアップ方法 
1. ddコマンドを使用してUSBにimageファイルの作成
2. 設定済Raspberrypi OSをSD Card CopierでmicroSDカードにバックアップ

## 1.imageファイルによるバックアップ(ターミナルやssh経由)
外部のPCを介さず起動中のシステムから直接USBメモリへのバックアップ対応
1. imgファイルを保存するUSBメモリをスロットに挿入
2. USBメモリとバックアップ元のデバイス名を確認、以下のコマンドで表示する   
    >  sudo fdisk -l   
    > ext)バックアップ元 /dev/mmcblk0   
    > ext)USBメモリ     /dev/sda   
    ~~~
    ディスク /dev/mmcblk0: 15 GiB, 16088301568 バイト, 31422464 セクタ
    単位: セクタ (1 * 512 = 512 バイト)
    セクタサイズ (論理 / 物理): 512 バイト / 512 バイト
    I/O サイズ (最小 / 推奨): 512 バイト / 512 バイト
    ディスクラベルのタイプ: dos
    ディスク識別子: 0x8193936a
    ~~~
3. USBメモリをマウントするフォルダの作成    
    > sudo mkdir /mnt/USBdisk
4. USBメモリをext4でフォーマット
    > sudo mkfs -t ext4 /dev/sda
5. USBメモリを/mnt/USBdiskフォルダにマウントする。これでファイルシステムとして認識される
    > sudo mount -t ext4 /dev/sda /mnt/USBdisk/
6. imgファイルとしてバックアップファイルを作成する
    > sudo dd bs=100M if=/dev/sda of=/mnt/USBdisk/\<FileName\>yyyymmdd.img status=progress conv=fsync    
    
    if=バックアップするSDカードのデバイス名   
    of=バックアップの保存パスと名前    
    bs=バッファーサイズ   
    status=progress 進捗状況の表示   
7. USBメモリをアンマウント
    > sudo umount /dev/sda

- ## 復元の方法 (Linux環境で実行)
    **※バックアップしたSDカードと同じサイズのSDカードに復元すること**

    - imgファイルからSDカードに復元する
        > sudo dd bs=100M if=\<FileName\>.img of=/dev/mmcblk0 status=progress conv=fsync   
    
    if=バックアップファイル   
    of=復元先のSDカードのデバイス名    
    bs=バッファーサイズ   
    status=progress 進捗状況の表示   

## 2. SD Card CopierでmicroSDカードにバックアップ(Raspberrypi OS内で実行)
以下参照   
- [設定済RaspbianをSD Card CopierでmicroSDカードにバックアップ](https://www.fabshop.jp/%E3%80%90step-24%E3%80%91%E8%A8%AD%E5%AE%9A%E6%B8%88raspbian%E3%82%92sd-card-copier%E3%81%A7microsd%E3%82%AB%E3%83%BC%E3%83%89%E3%81%AB%E3%83%90%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97/)
