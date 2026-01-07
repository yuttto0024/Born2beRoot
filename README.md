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
* **Security:**&nbsp;&nbsp;SSH, Pasward Policy
* **Firewall:**&nbsp;&nbsp;UFW
* **Scheduling:**&nbsp;&nbsp;Systemd timer

# Project Description
本プロジェクトにおいて、osに **Debian** を選択した。  
以下、他OSと比較を説明する。
## Operating System

### Debian
- **目的**  
  安定性を最優先に設計されたOS。「とにかく壊れない」ことを重視。
- **開発**  
  特定の企業に依存しない、完全なコミュニティ主導開発。フリーソフトウェアを重視。
- **特徴**  
  - **安定思想**  
    パッケージのバージョンはあえて古く保たれており、バグが少なく非常に安定している。長期稼働サーバー（Web, DB）に最適。
  - **学習面**  
    初心者に優しく、ネット上に情報が多いためトラブルシューティングしやすい。また、Linuxの下の構造を理解しやすい。

### CentOS
- **目的**  
  企業向け商用Linuxである RHEL (Red Hat Enterprise Linux) の無料クローン版。
  RHEL (Red Hat Enterprise Linux) と互換性があり、企業採用率が高い。
- **開発**  
  企業（Red Hat社）の開発ライン上にある。
- **特徴**  
  企業の開発環境に採用率が高い。また、RHELと互換性があり、商用環境の事前検証や、Red Hat系スキルの習得に向いている。

### Rocky
- CentOSの正統後継OS。
- 以前のCentOSと同じく、RHELと互換性がある。

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
  「誰が実行しているか」に関わらず、「そのアプリが何をしていいか」を強制的に制限するセキュリティシステム。
- **役割**  
  もしWebサーバーなどのアプリがハッカーに乗っ取られても、アプリ自体に**「Web用の画像フォルダ以外は触るな」**という制限（プロファイル）をかけておくことで、パスワードファイルなどを盗まれるのを防ぐ。
- **プロファイルとは**  
  アプリごとの「許可リスト」。「このアプリは、このフォルダは読み込んでいいが、書き込みは不許可」「ネット接続は許可」といった細かいルールが書かれた設定ファイルのこと。

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

## Phase2 
### ユーザー、グループ周り
- sudoインストール:```apt install sudo```、デバッグ:```sudo --version```
- ユーザー作成：```sudo adduser <username>```、デバッグ：```getent passwd <username>```
- グループ作成（sudo addgroup ~~, デバッグ：getent group <groupname>）
- グループに追加（sudo adduser <username> <groupname>）
### sshインストールと設定
- apt install sshで入れた（デバッグ：systemctl status ssh）
- /etc/ssh/sshd_config のPort番号変更、SSH経由のrootログイン禁止（デバッグ：sysctl + restart）
- 外部からポート番号 + ユーザーログイン（ssh yuonishi@localhost -p 4242）

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

## sudoポリシー
ココでは、メインの設定ファイル（/etc/sudoers）を直接編集せず、/etc/sudoers.d/ ディレクトリ内に専用のファイルを作成して管理している。
### ファイル構造
- Config file：```/etc/sudoers.d/sudo_config```
- Config file：```/var/log/sudo/```
- sudoers.dというsudoサービスの常駐プロセスの設定用のファイルを作成（touch /etc/sudoers.d/sudo_config）
- mkdir /var/log/sudo（varに記録残して、sudoは、管理者権限を実行かつ、実行下ユーザーのログを残すシステム）
- sudoの設定（sudo visudo -f /etc/sudoers.d/sudo_config）
sudoの設定、特にrootユーザーのパスワードが無効化した状態でsudoが壊れると、管理者権限になれなくなり修正不能。
visudoは、保存前に文法チェックが走る。
sudo visudoを打つと自動でメインの設定ファイル（/etc/sudoers）を開くため、-fフラグ
```sh
Defaults	passwd_tries=3
Defaults	badpass_message="Wrong Password!!"
Defaults	logfile="/var/log/sudo/sudo_config"
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
- パスワードの有効期限編集
- パスワードモジュールPAMのインストール（デバッグ：passwdでパスワード変更）
## Phase3
### System Monitoring Script (monitoring.sh)
ここでは、以下のシステム情報を収集し、全端末に通知するBashスクリプトを作成。
アドレス：/var/log/sudo/sudo_config
- 1. Architecture
uname -a（Unix Name）コマンドを使用し、os名、カーネルバージョン、ハードウェアアーキテクチャ等のシステムの基本情報を網羅的に表示。
- 2. CPU Physical
/proc/cpuinfo ファイルから physical id エントリを検索し、その数をカウントすることで物理プロセッサの数を算出。
- 3. vCPU
/proc/cpuinfo ファイル内の processor エントリの行数をカウントし、OSが認識している論理コア（スレッド）の総数を算出。
- 4. Memory Usage (RAM)
free -m コマンドでメモリ情報をMB単位で取得し、awkで使用量と合計値から使用率（％）を計算・整形。
- 5. Disk Usage
df -m コマンドでファイルシステムのディスク容量をMB単位で取得。grep "^/dev/" を用いて物理デバイスおよびLVMパーティションのみを抽出し、awk でシステム全体の「使用量」と「合計容量」を合算。課題の出力例（Used/TotalGb）に従い、合計容量のみGB単位に変換して整形・表示。
- 6. CPU Load
top -bn1 コマンドを使用して、その瞬間のプロセスのスナップショットを取得。grep でCPU情報の行を抽出し、awk を用いて ユーザープロセス（$2） と システムプロセス（$4） の使用率を合算して表示。
- 7. Last boot
システムの最終起動日時を取得するため uptime -s コマンドを使用。who -b コマンドは出力形式が変動するため、安定したフォーマット（YYYY-MM-DD HH:MM:SS）を提供する uptime -s を採用、awkで秒数を除去しています。
- 8. LVM use
lsblk コマンドを使用してブロックデバイス一覧を取得し、grep で "lvm" タイプが含まれる行数をカウント。その結果を基に if 文で条件分岐を行い、LVMが有効な場合は "yes"、無効な場合は "no" を表示。
- 9. Connection
ss コマンド（従来の netstat の代替）を使用してソケット統計を取得し、grep で "ESTAB"（確立済み）の状態にあるTCP接続を抽出。その行数をカウントして表示。
- 10. User log
users コマンドで現在ログインしているユーザーの一覧を取得し、wc -w（ワードカウント）を用いてユーザー数を算出。
- 11. Network
hostname -I コマンドでIPアドレスを取得し、ip link コマンドから grep と awk を用いてMACアドレス（Ethernetアドレス）を抽出。これらを組み合わせて表示しています。
- 12. Sudo
Sudoのログファイル（/var/log/sudo/sudo_config）内に記録された "COMMAND" という文字列を含む行を grep で検索し、その行数を wc -l でカウントすることで、Sudoコマンドの実行総数を算出。
---
- wall
echo ""メッセージをwallに渡す。wallは、ログインしている全員の画面にブロードキャストする。
## Systemd Timer
- cron vs systemd timer
cron: 時計を見て動き、10:08に起動した場合、最初の実行は10:10
systemed timer: 10:08に起動した場合、最初の実行は10:18
### systemd Timer の仕組み
Systemdは 「実行する役割（Service）」 と 「時間を計る役割（Timer）」 を明確に分ける設計思想
Systemdで定期実行するには、2つのファイルをペアで作る必要があります。
- monitoring.service （実行係 / Worker）
  - 何を実行するかだけを知っている。
  - スクリプトのパス、実行権限、プロセスの種類など。
  - タイマーと無関係に sudo systemctl start monitoring.service でテスト実行ができる。
- Timerファイル: 「いつやるか」を書いたタイマー設定。
  - 「いつ」実行係を呼び出すかだけを知っている。
  - 起動してからの時間、繰り返し間隔など。
  - 時間が来たら monitoring.service のスイッチを押すだけ。
「Timerファイルが時間を計り、時間になったらServiceファイルを呼び出す
SystemdはOSの起動プロセスそのものを管理しているため、「OSが立ち上がった瞬間」 を基準に物事を進めるのが得意
Systemdの設定ファイルは「INIファイル形式」という古いルールを採用しており、「セクション（章）」 で区切らないと読み取ってくれません。
[Unit]: ファイルの説明や依存関係を書く場所
- [Service]: 実行に関する設定を書く場所。
- [Timer]: 時間に関する設定を書く場所。
- [Install]: enable した時の登録先を書く場所。
各変数は予約語
sudo vim /etc/systemd/system/monitoring.service
```sh
#このサービスの説明。ログなどで表示されます。
Description=Monitoring Script Service
#実行するスクリプトの絶対パス。
ExecStart=/usr/local/bin/monitoring.sh
# 常駐」せず、スクリプトが終わったらプロセスも終了させる設定。
# これがないと「実行中」の状態が続き、タイマーが次を呼べなくなる。
Type=oneshot
```
sudo vim /etc/systemd/system/monitoring.timer
```sh
Description=Run monitoring script every 10 minutes
# 意図: サーバー起動(Boot)してから、最初の1回目を実行するまでの待機時間。
# 起動直後の高負荷を防ぐために設定。
OnBootSec=10min
# 意図: 前回の実行(Active)から、次の実行までの間隔。
# これで「10分おき」のループを作ります。
OnUnitActiveSec=10min
# 時間が来たら、さっき作った「monitoring.service」を呼び出すという指定。
Unit=monitoring.service
# PC起動時にタイマーを自動有効化するための決まり文句
WantedBy=timers.target
```


## Phase4
# Resoureces
[Markdown記法 チートシート - Qiita](https://qiita.com/Qiita/items/c686397e4a0f4f11683d)
