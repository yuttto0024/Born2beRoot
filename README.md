_This project has been created as part of the 42 curriculum by yuonishi._

## 目次

1. [Description](#description)
2. [Project Description](#project-description)
3. [Instruction](#instruction)
4. [Bonus](#bonus)
5. [Resources](#resources)

# Description
本プロジェクトは、システム管理および仮想化機能の基礎を実践的に学ぶ課題である。  
仮想化ソフトウェアの VirtualBox を使用して、物理マシン上に仮想マシンを構築し、その上でサーバーを構築した。また、「最小限のサービスのみをインストールする」というレギュレーションの下、以下のサービスとセキュリティ設定を実装した。
* **Operating System:**&nbsp;&nbsp;Debian
* **Partitioning:**&nbsp;&nbsp;LVM
* **Security:**&nbsp;&nbsp;SSH, Passward Policy
* **Firewall:**&nbsp;&nbsp;UFW
* **Scheduling:**&nbsp;&nbsp;Systemd timer

# Project Description
本プロジェクトにおいて、osに **Debian** を選択した。
以下、他OSと比較を説明する。
## Operating System
利用目的やスキルレベルに応じて、適切なディストリビューションを選ぶ。例えば、初心者にはUbuntu、サーバー用途にはCentOSやDebianなどがある。
### Debian
- **目的**  
  本課題においては、複雑な商用仕様（RHEL系）よりも、ドキュメントが豊富でセキュリティ設定（AppArmor）やパッケージ管理（APT）の挙動が理解しやすいDebianを採用することで、Linuxシステム管理の基礎を学習しやすい。
- **特徴**  
  - 特定の親企業（Red Hatなど）を持たず、世界中のボランティア開発者によって運営されている。商業的な理由による仕様変更やサポート終了がない。
  - パッケージ管理において、Rocky Linuxの dnf と比較して、構文がシンプルで、システム構築の学習に最適。
  - Rocky Linux における SELinux は設定が複雑だが、Debianの AppArmor はファイルパスベースで設定できるため、手軽に設定できる。

### CentOS
- **目的**  
  基本的に商用で無料で使いたければ最適なos。
- **特徴**  
  企業向け商用Linuxで、高価なRHEL (Red Hat Enterprise Linux) の無料クローン版。
  クローンなため特徴はほぼ変わらないが、RHELを追従する形なので、セキュリティフィックス（ソフトウェアもセキュリティ上の脆弱性を修正する更新プログラム（セキュリティパッチ））が若干遅れるラグ存在する。

## Comparison of Tools
本プロジェクトの設計おいて、**VirtualBox**、**AppArmor**、**UFW**を選択した。
### Apt vs Aptitude
どちらもDebian系のパッケージ管理ツールだが、役割と機能に明確な違いがある。
Debianのパッケージ管理システムにおいて、aptは標準的なインターフェースであり、aptitudeはより高度な機能を持つフロントエンドである。
* **Apt（Advanced Packaging Tool）**  
  - **特徴**  
    - 機能がシンプルで動作が軽量。
    - コマンドライン操作に特化しており、スクリプト処理などに適している。
  - **基本的な使い方**  
    ```sh
    # パッケージリストの更新
    sudo apt update
    # パッケージのインストール
    sudo apt install <package_name>
    # パッケージの削除
    sudo apt remove <package_name>
    ```
* **Aptitude:**  
  - **特徴**  
    - より高機能なパッケージ管理ツール。
    - インストール時に競合が発生した際、解決策（ダウングレードや削除など）を提案してくれる。
    - TUI (Text User Interface) を持っており、引数なしで実行するとビジュアルモード（マウス操作のように視覚的にパッケージを選べるモード）になる。
  - **基本的な使い方**  
    ```sh
    # 通常のインストール（aptと同様）
    sudo aptitude install <package_name>
    # ビジュアルモードの起動
    sudo aptitude
    ```

### AppArmor とは
- **概要**  
  Linuxカーネルのセキュリティモジュール（LSM）の一種で、MACを実装するシステム。
- **役割**  
  プログラム（WebサーバーやDBなど）に脆弱性があり、万が一乗っ取られたとしても、そのプログラムが本来アクセスする必要のないファイルへのアクセスを遮断する。具体的には、特定のシステムコールを監視し、アクセス許可がないとブロックしてエラーを返す。
- **プロファイルとは**  
  アプリごとの「許可リスト」。「このアプリは、このフォルダは読み込んでいいが、書き込みは不許可」「ネット接続は許可」といった細かいルールが書かれた設定ファイルのこと。

### DAC（任意アクセス制御）とMAC（強制アクセス制御）
これらの違いは、「誰がルールを決めるか」と「何を防ぎたいか」
1. DAC（Discretionary Access Control）
- **目的**  
  ユーザーごとのプライバシーを守る。
- **仕組み**  
  ファイルの所有者が、「誰に見せていいか」を自分で決める。
- **弱点**
  なりすましに弱い。もしウイルスが自分の権限で実行されたら、そのウイルスはあなたがアクセスできる全てのファイルを盗めてしまう。システムは「自分のやっている」と判断して止めてくれない。
2. MAC（Mandatory Access Control）
- **目的**
  被害を最小限に抑える
- **仕組み**  
  管理者一人が、アクセス制御を行う。サブジェクトがオブジェクトに対してアクセスできる範囲を制限できる。
- **強み**  
  プログラム毎に制限できるため、Webサーバーが乗っ取られてもMACによって、/homeやパスワードファイルを守れる。

### UFW とは
- **概要**  
  Linuxの「関所（セキュリティゲート）」の設定サービス。本来、Linuxの通信制御は設定が難しいが、それを人間が直感的に操作できるようにしたもの。
- **役割**  
  サーバーに届く大量のパケットの宛先であるポート番号を見て、特定のポート宛てのパケットだけ通す。それ以外はすべて拒否して捨てる、というルールを強制する。
- **リソース保護の観点**  
  SSHサーバー（アプリ）まで通信を通してから拒否すると、アプリ側で「暗号化鍵の交換」や「パスワード照合」のための複雑な計算とメモリ確保が発生し、CPUリソースを食いつぶされる（DoS攻撃に弱い）。 UFWを使うと、アプリに届く手前の、OSの入り口（カーネルレベル）で、「ポート番号が違うから弾く」という単純な条件分岐だけで門前払いできるため、サーバーの負荷を最小限に抑えることができる。
- **パケットとは**  
  インターネット上のデータ（Webページやメール、チャット等）は、「パケット」という小さな箱に分割されて届く。この箱には、送信元やポート番号の情報も付いている。

### LVMとは
パーティションを可動式に変える技術。
- メモリ管理が「仮想アドレス」と「物理アドレス」をページテーブルで管理するように、 LVMは「論理エクステント (LE)」と「物理エクステント (PE)」をマッピングテーブルで管理している。  
- OS（ファイルシステム）は「連続したディスク領域」と思って書き込むが、LVMが裏で物理ディスク上の最適な場所にデータを分散させている（Device Mapper機能）。


# Instruction
以下、本プロジェクトの実装手順をまとめる。
## Phase1 

### 1. 仮想ドライブのセットアップ
物理マシンの中に、ソフトウェアで再現した仮想のハードウェアを作成する工程。
- **OS: Debian**  
  debian13-2-0-amd6を使用。CentOSと比較してパッケージ管理が容易で、初心者向けのドキュメントが豊富なため。
- **Firmware: Legacy BIOS**  
本課題のパーティション構成（MBR方式）に合わせるため。EFIを有効にするとパーティション設計が複雑化するため、学習目的ではレガシーBIOSの方が構造を理解しやすい。
- **Storage (Virtual Disk)**
  - File Type: .vdi
  - Size: 15.0 GB
  - Allocation: Dynamically allocated (可変サイズ / 逐次マッピング)  
  「15GBの箱」を先に確保するのではなく、データが増えた分だけホストの容量を使う設定。ホストのストレージ圧迫を防ぐため。

### 2. osインストール
仮想マシンを起動し、仮想ディスクにDebianをインストールする工程。
- **ISO Mount**  
  ダウンロードしたDebianの .iso ファイルを、仮想マシンの仮想ドライブにセットして起動。
- **Partitioning (Manual)**  
  LVMと暗号化を手動で構築した。
- **Primary Partition (sda1)**  
  - **Mount: /boot**  
    起動用プログラム（Kernel）を置く場所。ここは暗号化してはいけない（鍵を開けるプログラム自体が読めなくなるため）。
  - **Logical Partition (sda5)**  
    - Type: Encrypted Volume (LUKS)
    - 残りの全領域。ここを暗号化し、その中にLVM（論理ボリューム）を作成する。OS本体（root, home, swap）は全てこの中に格納される。
- **Boot Loader**  
  GRUBをメインドライブ（/dev/sda）にインストール。
- **詳細**
  - **計算資源（CPU/RAM）**  
    物理マシンの一部を、VirtualBoxが借りて、仮想マシンに分け与えている。
  - **ハードディスク**  
    .vdi という巨大ファイルを、仮想マシンが「本物のハードディスク」と認識。Debianがこのファイルの中身に書き込みを行う。
  - **os**
    仮想ハードの上で、Debianが動いている。

### パーティション構造
- **sda1 (/boot)**
  - **Unencrypted**
  - **理由**  
    PC起動時、BIOS/UEFIが最初に読み込む場所。ここには「暗号化解除画面を出すためのプログラム（Bootloader）」が入っているため、暗号化してはいけない（暗号化すると起動不能になる）。
- **sda2 (Extended)**  
  - MBRの制限（プライマリは4つまで）のため作成された、論理パーティションを入れるためのコンテナ。
- sda5  
  - **Encrypted**  
  - **理由** 
    - ここにOSの本体（/root, /home, /var, swap）が全て格納されている。
    - 物理ディスク上では乱数に見えるため、PC盗難時等の情報漏洩を防ぐ。

## Phase2 
### ユーザー、グループ周り
- sudoインストール:```apt install sudo```、デバッグ:```sudo --version```
- ユーザー作成：```sudo adduser <username>```、デバッグ：```getent passwd <username>```
- グループ作成：```sudo addgroup <groupname>```、デバッグ：```getent group <groupname>```
- グループに追加：```sudo adduser <username> <groupname>```  
### sshインストールと設定
1. Concept：セットアップとサーバーデーモンの区別
- /etc/ssh/ssh_config (Client Side)
  自分が他のサーバーへ接続しに行く時の設定。
- /etc/ssh/sshd_config (Server Daemon Side)
  外部からの接続を受け入れる時の設定。Debianをサーバーとして機能させるための設定。
- インストール + 有効化：```sudo apt install openssh-server``` デバッグ：```systemctl status ssh```
2. Configuration  
  デーモンにセキュリティを設定する：```sudo vim /etc/ssh/sshd_config```
- Port 4242：デフォルトの22番は狙われやすいため変更。
- PermitRootLogin no  
  Rootがログインするのは危険。「一般ユーザー」としてログインさせ、必要な時だけ sudo させることで、「誰が操作したか」のログを確実に残す。
3. 設定ファイル適用：```sudo systemctl restart ssh```  
  デーモンは起動時にしか設定ファイルを読まないため、書き換えたら再起動して読み直させる。
4. Port Forwarding
  仮想マシンは「ホストPCの中の隔離された箱」にあるため、そのままでは外から繋がらない。VirtualBoxの設定で、ホストとゲストの間にパイプを通す。
- Host: 4242 → Guest: 4242  
  「ホストの4242番ポートに来た通信を、そのままゲストの4242番に転送しろ」という命令。
5. Connection：```ssh yuonishi@localhost -p 4242```、デバッグ：```ssh root@localhost -p 4242```  
  ホストマシンのターミナルから、実際に接続して確認する。

### ufwインストールと設定
- インストール + 有効化
- ファイアウォールによるポート番号の接続許可：```ufw allow 4242```
- ufwデバッグ
  - ufwが存在するか：```ufw --version``` ```which ufw```
  - ufw サービスが有効か：```ufw status verbose```
  - ルール確認：```ufw numbered```
  - 通信しているか：```lsof -i -P```
- systemctl と ufw は思想が違う
  - systemctl
    - 「プログラムは動いているか」状態確認
    - デーモン管理
  - ufw
    - 「通信を通すか」ルール管理

### 典型的なデバッグフロー
1. sshdは動いてるか：``` systemctl status ssh ```
2. ポートは開いてるか：``` lsof/ ss ```
3. firewall 通してるか：``` ufw status ```

### sudoポリシー
ココでは、メインの設定ファイル（/etc/sudoers）を直接編集せず、/etc/sudoers.d/ ディレクトリ内に専用のファイルを作成して管理している。
- ファイル構造
  - Config file：```/etc/sudoers.d/sudo_config```
  - Config file：```/var/log/sudo/```
  - sudoers.dというsudoサービスの常駐プロセスの設定用のファイルを作成（touch /etc/sudoers.d/sudo_config）
  - mkdir /var/log/sudo（varに記録残して、sudoは、管理者権限を実行かつ、実行下ユーザーのログを残すシステム）
  - sudoの設定（sudo visudo -f /etc/sudoers.d/sudo_config）
    - sudoの設定、特にrootユーザーのパスワードが無効化した状態でsudoが壊れると、管理者権限になれなくなり修正不能。  
    - visudoは、保存前に文法チェックが走る。sudo visudoは自動で（/etc/sudoers）を開くため、-fフラグをつける。
```sh
Defaults	passwd_tries=3
Defaults	badpass_message="Wrong Password!!"
Defaults	logfile="/var/log/sudo/sudo.log"  #sudoの入力履歴のみを記録する
Defaults	log_input, log_output
Defaults	iolog_dir="/var/log/sudo"
Defaults	requiretty
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```
### visudo
設定ファイルの編集には必ず visudo コマンドを使用する。
- **文法チェック機能**  
  visudo は保存時に自動で構文チェックを行う。もし設定に記述ミスがある場合、保存をブロックして警告を出す。
- **ロックアウト防止**  
  sudo の設定が壊れると、管理者権限（root）になれなくなり、システムの修正が不可能になる。visudo はこの事故を防ぐための安全装置である。
- **使用コマンド**  
  標準では /etc/sudoers を開くため、-f フラグでファイルを指定する。
  ```sh
  sudo EDITOR=vim visudo -f /etc/sudoers.d/sudo_config
  ```
- **デバッグ：**  
  - **設定ルールの確認：**```sudo -l -U <username>```
    現在ログインしているユーザーに適用されているSudoのルールと、実行可能な権限一覧を表示する。設定ファイルを開かずに、有効な設定を表示できる。
  - **ログの確認**  
    コマンドの実行履歴（Input/Output）が正しく保存されているか確認する。
    ```sh
    sudo ls /var/log/sudo/
    sudo cat /var/log/sudo/sudo.log
    ```

### パスワードポリシー
1. Password Age（/etc/login.defs）
パスワードの「有効期限」に関する設定。一度設定したパスワードを使わせないためのルール。
- 30日ごとに変更を強制：```PASS_MAX_DAYS 30```
- 最低2日間は変更不可：```PASS_MIN_DAYS 2```
- 期限切れの7日前から警告を出す：```PASS_WARN_AGE 7```
2. Password Complexity (PAM)```/etc/pam.d/common-password```
パスワードの「強度（複雑さ）」に関する設定。 標準機能だけでは「大文字必須」などの細かい制限ができないため、専用のモジュールを追加する。```apt install libpam-pwquality```
- retry=3 の後ろに以下を追記
```sh
minlen=10 #最低10文字
ucredit=-1 #大文字を最低1文字含む
dcredit=-1 #数字を最低1文字含む
lcredit=-1 #小文字を最低1文字含む
maxrepeat=3 #同じ文字を3回以上連続させない
difok=7 #変更前のパスワードと7文字以上異なっていること
reject_username #ユーザー名を含んでいたら拒否
enforce_for_root #このルールをRootユーザーにも強制する
```
- デバッグ：```passwd```
## Phase3
### monitoring.sh：/usr/local/bin/monitoring.sh
ここでは、以下のシステム情報を収集し、全端末に通知するBashスクリプトを作成。   
アドレス：```/usr/local/bin/monitoring.sh```
1. Architecture  
  ```uname -a```（Unix Name）コマンドを使用し、os名、カーネルバージョン、ハードウェアアーキテクチャ等のシステムの基本情報を網羅的に表示。
2. CPU Physical  
  /proc/cpuinfo ファイルから physical id エントリを検索し、その数をカウントすることで物理プロセッサの数を算出。
3. vCPU  
  /proc/cpuinfo ファイル内の processor エントリの行数をカウントし、OSが認識している論理コア（スレッド）の総数を算出。
4. Memory Usage (RAM)  
  ```free -m``` コマンドでメモリ情報をMB単位で取得し、awkで使用量と合計値から使用率（％）を計算・整形。
5. Disk Usage  
  ```df -m``` コマンドでファイルシステムのディスク容量をMB単位で取得。```grep "^/dev/"``` を用いて物理デバイスおよびLVMパーティションのみを抽出し、```awk``` でシステム全体の「使用量」と「合計容量」を合算。課題の出力例（Used/TotalGb）に従い、合計容量のみGB単位に変換して整形・表示。
6. CPU Load  
  ```top -bn1``` コマンドを使用して、その瞬間のプロセスのスナップショットを取得。```grep``` でCPU情報の行を抽出し、```awk``` を用いて ユーザープロセス（$2） と システムプロセス（$4） の使用率を合算して表示。
7. Last boot  
  システムの最終起動日時を取得するため ```uptime -s``` コマンドを使用。```who -b``` コマンドは出力形式が変動するため、安定したフォーマット（YYYY-MM-DD HH:MM:SS）を提供する ```uptime -s``` を採用、```awk```で秒数を除去しています。
8. LVM use  
  ```lsblk``` コマンドを使用してブロックデバイス一覧を取得し、```grep``` で "lvm" タイプが含まれる行数をカウント。その結果を基に if 文で条件分岐を行い、LVMが有効な場合は "yes"、無効な場合は "no" を表示。
9. Connection  
  ```ss``` コマンドを使用してソケット統計を取得し、```grep``` で "ESTAB"（確立済み）の状態にあるTCP接続を抽出。その行数をカウントして表示。
10. User log  
  ```users``` コマンドで現在ログインしているユーザーの一覧を取得し、```wc -w```を用いてユーザー数を算出。
11. Network  
  ```hostname -I``` コマンドでIPアドレスを取得し、```ip link``` コマンドから ```grep``` と ```awk``` を用いてMACアドレス（Ethernetアドレス）を抽出。これらを組み合わせて表示しています。
12. Sudo  
  Sudoのログファイル（/var/log/sudo/sudo.log）内に記録された "COMMAND" という文字列を含む行を ```grep``` で検索し、その行数を ```wc -l``` でカウントすることで、Sudoコマンドの実行総数を算出。
13. wall  
  ```echo ""```メッセージを```wall```に渡す。```wall```は、ログインしている全員の画面にブロードキャストする。
## Phase4
### Systemd Timer
- cron vs systemd timer
  - cron：時計を見て動き、10:08に起動した場合、最初の実行は10:10。
  - systemed timer：10:08に起動した場合、最初の実行は10:18。　　
    SystemdはOSの起動プロセスそのものを管理しているため、「OSが立ち上がった瞬間」 を基準に物事を進めるのが得意
### systemd Timer の仕組み
Systemdは 「実行する役割（Service）」 と 「時間を計る役割（Timer）」 を明確に分ける設計思想。
Systemdで定期実行するには、2つのファイルをペアで作る必要がある。
- monitoring.service （実行係 / Worker）
  - 何を実行するかだけを知っている。
  - スクリプトのパス、実行権限、プロセスの種類など。
  - タイマーと無関係に sudo systemctl start monitoring.service でテスト実行ができる。
- Timerファイル: 「いつやるか」を書いたタイマー設定。
  - 「いつ」実行係を呼び出すかだけを知っている。
  - 起動してからの時間、繰り返し間隔など。
  - 時間が来たら monitoring.service のスイッチを押すだけ。
Timerファイルが時間を計り、時間になったらServiceファイルを呼び出す。
Systemdの設定ファイルは「INIファイル形式」という古いルールを採用しており、セクション で区切らないと読み取ってくれない。
- [Unit]： ファイルの説明や依存関係を書く場所。
- [Service]： 実行に関する設定を書く場所。
- [Timer]： 時間に関する設定を書く場所。
- [Install]: enable した時の登録先を書く場所。

### monitoring.service  
アドレス：/etc/systemd/system/monitoring.service
```sh
Description=Monitoring Script Service #実行するスクリプトの絶対パス。
ExecStart=/usr/local/bin/monitoring.sh  #常駐せず、スクリプトが終わったらプロセスも終了させる。これがないと「実行中」の状態が続き、タイマーが次を呼べなくなる。
Type=oneshot
```
### monitoring.timer
アドレス：/etc/systemd/system/monitoring.timer
```sh
Description=Run monitoring script every 10 minutes  #サーバー起動(Boot)してから、最初の1回目を実行するまでの待機時間。
OnBootSec=1min  #前回の実行(Active)から、次の実行までの間隔。
OnUnitActiveSec=10min #時間が来たら、monitoring.serviceを呼び出すという指定。
Unit=monitoring.service #PC起動時にタイマーを自動有効化するための決まり文句。
WantedBy=timers.target
```
設定後は、デーモンに設定ファイルを読ませて、再起動
```sh
sudo systemctl daemon-reload
sudo systemctl restart monitoring.timer
```

### Systemd Timer の停止
1. ログのリアルタイム監視：```sudo journalctl -u monitoring.service -f```
2. タイマーの停止：```sudo systemctl stop monitoring.timer```
3. 実行間隔の変更：```sudo vim /etc/systemd/system/monitoring.timer```
4. 設定反映
```sh
sudo systemctl daemon-reload
sudo systemctl restart monitoring.timer
```
5. ログで~分毎に動くか確認。

### hostnameの変更
- /etc/hostname：表札。ターミナルの@の後ろが変わる。
- /etc/hosts：システムによる、アドレスと名前の対応表
1. デバッグ：```hostnamectl```
2. 表札を書き換える：```sudo hostnamectl set-hostname new_host```
3. 住所録を書き換える：```sudo vim /etc/hosts```
4. 再起動して反映```sudo reboot```
5. デバッグ

## Bonus Part

## 1. Web Server Implementation  
### Step1.内部（Debian）からLighttpdへのアクセス  
- apt install lighttpd
  - デーモン確認：systemctl status lighttpd
- ufw allow 80
  - ポートの待ち受け確認：ss -tulpn | grep lighttpd
- ufw status
- curl http://localhost

### Step 2. 外部（Host OS）からWebサーバーへのアクセス  
Webサーバーが正常に稼働していても、ネットワーク設定が不十分だと外部から閲覧できない。  　
VirtualBoxのポートフォワーディング機能を使用して、ホストマシンと仮想マシン間の通信経路を確立する。

1. **VirtualBox Port Forwarding**  
   ホストマシンのポート 8080 へのアクセスを、ゲストマシンのWebサーバーポート 80 へ転送する設定を追加。
   - **Protocol:** TCP  
   - **Host Port:** 8080  
   - **Guest Port:** 80  

2. **Access from Host Browser**  
   ホストOS（Windows/Mac）のブラウザから以下のURLへアクセスし、Lighttpdのデフォルトページが表示されることを確認。
   - **URL:** `http://localhost:8080`  
   - **Validation:** "Placeholder page" が表示されれば、Webサーバーの公開設定は完了。  

---

## 2. Database Implementation (MariaDB)
WordPressのデータを管理・保存するために、RDBMS（リレーショナルデータベース管理システム）である **MariaDB** を採用した。

### MariaDB
- **目的** WordPressの投稿データ、設定情報、ユーザー認証情報などを永続的に保存する。Webサーバー（Lighttpd）からのリクエストに応じ、必要なデータを検索・提供するバックエンドの役割を担う。
- **セキュリティ設定** インストール直後のデフォルト設定はセキュリティが低いため、専用スクリプトを用いて堅牢化を行った。最新のDebian環境に合わせ、コマンドは `mariadb-secure-installation` を使用。
  - **匿名ユーザーの削除**: 誰でもログインできる不要なアカウントを削除。
  - **リモートRootログインの禁止**: 管理者権限（Root）でのログインをローカル（localhost）のみに制限し、外部からの攻撃を防ぐ。

### Implementation Steps
1. **Installation**
   ```bash
   sudo apt install mariadb-server -y


  ---

  ### Database Configuration (SQL)
WordPress用のデータベースと専有ユーザーを作成し、権限を付与した。

1. **Create Database & User**
   MariaDBコンソールにログインし、以下のSQLコマンドを実行。
   ```sql
   /* 1. Database作成 */
   CREATE DATABASE wordpress_db;
   
   /* 2. User作成 (パスワードは適切に設定) */
   CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'password123';
   
   /* 3. 権限付与 (WordPressDBへの全権限をwp_userに与える) */
   GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
   
   /* 4. 設定反映 */
   FLUSH PRIVILEGES;

## 3. PHP Implementation (Processor)
WordPressはPHP言語で記述された動的なCMSであるため、WebサーバーがPHPコードを解釈・実行できる環境を構築した。

### PHP Components
- **php-cgi**
  - **目的:** Lighttpd上でPHPを実行するためのFastCGI対応インタプリタ。Webサーバーからのリクエストを受け取り、PHPスクリプトを処理してHTMLを生成する。
- **php-mysql**
  - **目的:** PHPアプリケーション（WordPress）がバックエンドのデータベース（MariaDB）と通信するためのドライバモジュール。

### Implementation Steps
1. **Installation**
   必要なパッケージを一括インストール。
   ```bash
   sudo apt install php-cgi php-mysql -y

---

# Resources
[Markdown記法 チートシート - Qiita](https://qiita.com/Qiita/items/c686397e4a0f4f11683d)
[Linuxのディストリビューション比較](https://eng-entrance.com/linux-distribution-compare)
[DACとMAC、RBACの違い](https://qiita.com/miyuki_samitani/items/acde77784237e482aef8)
