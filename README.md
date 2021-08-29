# Linux系システムのバックアップ方法（Raspberry piでルーターを構成した際の方法)

## バックアップ規約
- Raspberrypi OSに対してパッケージのアップグレードやシステムの変更をする際は、**必ずバックアップイメージを作成してから行うこと**(アップグレードをした後にシステムが動作しなかった時に、すぐに動作できる状態に戻すことが出来るようにするため)   
- 新しいバージョンにするときは、数日動作できることを確認した後にバックアップする。   
- セキュリティなどの緊急性の高いアップグレード以外は半年に一回行う。
- **面倒だがバックアップしてなくて最初から作業したことを思い出してみよう**
### アップデートフロー
1. システムフルバックアップ
2. システムアップデート
3. 1週間動作させ異常がないか確認
4. 問題なければ、フルバックアップ
5. 1.のバックアップは消去

## バックアップ方法 
1. ddコマンドを使用してUSBにimageファイルの作成   
    **注意：ddコマンドを使用したバックアップは、入力デバイスと出力ファイル名を間違えるとバックアップファイルまたはデバイスのシステムを破損する可能性があるため、入出力名は注意して入力すること**
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
    > sudo dd bs=100M if=/dev/mmcblk0 of=/mnt/USBdisk/\<FileName\>yyyymmdd.img status=progress conv=fsync    
    
    if=バックアップするSDカードのデバイス名   
    of=バックアップの保存パスと名前    
    bs=バッファーサイズ   
    status=progress 進捗状況の表示   
7. USBメモリをアンマウント
    > sudo umount /dev/sda

8. imgファイルをtar.gzファイルに圧縮
    > tar c \<image file\>.img | pigz -p 4 > \<out put filename\>.tar.gz   
    https://qiita.com/nkojima/items/1d8491bcec7f71384e8d

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

## バックアップしたイメージファイルのサイズを変更する
USBメモリやSDカードは同じ製品でもセクターの数が微妙に違うこともあり、サイズの小さいイメージファイルを大きいメディアに書き込むのは問題ないが、書き込むメディアよりサイズの大きいイメージファイルは書き込むことはできない。   
イメージファイルのパーテイションサイズを変更することで書き込めるようにする

### 大まかな流れ
1. 縮小したいイメージファイル（\*.img）をマウント
2. gpartedでドライブを縮小
3. 縮小して未使用の領域を除く

### 手順
1. \*.imgファイルをマウントするために空きのループデバイスを確認。下記のコマンドで空いているループデバイス一覧が表示される
   > sudo losetup -f

2. イメージファイル（\*.img）を空きのループデバイスにマウントする。下記の例では空きのloop8にXXX.imgをマウントした場合を示す。
   > sudo losetup /dev/loop8 XXX.img

3. ループデバイスを更新。
   > sudo partprobe /dev/loop8

4. パーティションを編集するためにパーティションツールを起動。
   > sudo gparted /dev/loop8

   - **【インストールコマンド】sudo apt-get install gparted**

5. GUI上でドライブの縮小したいext4パーティションを右クリックし、[リサイズ/移動]をクリックする。右端をスライドさせてパーティションを縮小。100MiB~500MiB程度縮小。

6. gparted の適用ボタンを押して変更を反映。    
**※この時点ではシステム上のパーティションサイズが小さくなっただけでイメージファイル（\*.img）の容量は変わらない。イメージファイルの未使用パーティションの空き領域をイメージファイルから削除する必要あり。**

7. イメージファイルのパーティションセクタサイズを確認する。
   > sudo fdisk -l /dev/loop8
   ~~~　
   パーティションの最終セクタの位置(ZZZZ)をメモする。
   デバイス　ブート  Start   End      Block  Id   System
   UUUUUUU         BBBB   CCCC    AAAA   *  Fat
   WWWWW    *    YYYY   ZZZZ     AAAA   *  Linux
   ~~~
   
8. ループデバイスを解除する。(追記)
   > sudo losetup -d /dev/loop8

9. イメージファイルの空き領域を削除。
   > sudo truncate --size=$[(ZZZZ+1)*512] XXX.img

   - ※重要な点は最終セクタから次の空き領域を削除するため、パーティションの最終セクタ+1を設定する点です。

### 参考
[SBC OSイメージリサイズ方法](http://meerstern.seesaa.net/article/474139330.html)
