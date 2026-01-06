_This project has been created as part of the 42 curriculum by yuonishi._

## 目次

1. [Description](#description)
2. [Project Description](#project-description)
3. [Instruction](#instruction)
4. [Bonus](#bonus)
5. [Resources](#resources)

# Description
このプロジェクトは、システム管理および仮想化機能の基礎を実践的に学ぶ課題です。<br>仮想化ソフトウェアの&nbsp;VirtualBox&nbsp;を使用して、物理マシン上に仮想マシンを構築し、その上でサーバーを構築しました。<br>「最小限のサービスのみをインストールする」というレギュレーションの下、以下のサービスとセキュリティ設定を実装しました。
* **Operating System:**&nbsp;&nbsp;Debian
* **Partitioning:**&nbsp;&nbsp;LVM
* **Security:**&nbsp;&nbsp;SSH, Pasward Policy
* **Firewall:**&nbsp;&nbsp;UFW
* **Scheduling:**&nbsp;&nbsp;Systemd timer

# Project Description
## Chioce of Operating System
本プロジェクトにおいて、osは**Debian**を選択しました。<br>
以下に、Rockyと比較した際の利点と欠点を説明します。
### Debian vs Rocky
* **Debian**
  * **利点**
  * **欠点**
* **Rocky**
  * **利点**
  * **欠点**
## Comparison of Tools
本プロジェクトの設計おいて、**VirtualBox**、**AppArmor**、**UFW**を選択しました。<br>
競合サービスとの比較は以下の通りです。

### VirtualBox vs UTM
* **VirtualBox:**&nbsp;&nbsp;
* **UTM:**&nbsp;&nbsp;

### AppArmor vs SELinux
* **AppArmor:**&nbsp;&nbsp;
* **SELinux:**&nbsp;&nbsp;

### UFW vs firewalld
* **UFW:**&nbsp;&nbsp;
* **firewalld:**&nbsp;&nbsp;

# Instruction
以下、本プロジェクトの実装手順をまとめる。
## Phase1. 
### 仮想マシンのsetup
- debian13-2-0-amd64をインストール
- VBは入ってた。
- VBで仮想ドライブ（ディスクのフリをするファイル）作成
  - BaseMemory:2048MB..2GB割当
  - CPUは2つ
  - EFIは無効、レガシーBIOSで行く。（仮想マシン上で別々のosは切り替えないため、設定の量も多いため、当課題には適していない）
  - ストレージは30.0GB、pre-allocateはしない。逐次マッピングで軽量化。
- ISO連携（仮想ドライブにosを入れる）
  - Controler:IDEってのにDLしたDebianのパスを指定。
- 仮想マシン起動
  - 言語、root、ゲストユーザー初期化
  - パーティション設定（ボーナスゆえ手動）
    - sda1はprimalyで作成（primaryは4つまで）
    - 論理ボリュームとか、ファイルサイズを分担（ココ言語化未完成）
  - package managerのインストール
  - GRUBのインストールが終了
ここまでで、「ホストマシンのCPUとRAM（計算資源）を借りて、USBメモリ上のファイルを「仮想的なハードディスク」と認識させ、そこでDebian OSを動かしている状態」
以下、図式。
1. CPU/メモリ：iMac
2. ハードディスク：USBメモリ
3. OS: USBの中の.vbiファイルの中にDebian

## Phase2. 
### ユーザー、グループ周り
- sudoインストール（apt install sudo）（デバッグ：which sudo, sudo -V）
- ユーザー作成（sudo adduser ~~, デバッグ：/grep/passwd）
- グループ作成（sudo addgroup ~~, デバッグ：getent group <groupname>）
- グループに追加（sudo adduser <username> <groupname>）
### sshインストールと設定
- apt install sshで入れた（デバッグ：systemctl status ssh）
- /etc/ssh/sshd_config のPort番号変更、SSH経由のrootログイン禁止（デバッグ：sysctl + restart）
- 外部からポート番号 + ユーザーログイン（ssh yuonishi@localhost -p 4242）
### ufwインストールと設定
- インストール + 有効化
- ファイアウォールによるポート番号の接続許可（sudo ufw allow 4242）（デバッグ：sudo lsof -i -P）
## sudoポリシー
- sudoers.dというsudoサービスの常駐プロセスの設定用のファイルを作成（touch /etc/sudoers.d/sudo_config）
- mkdir /var/log/sudo（varに記録残して、sudoは、管理者権限を実行かつ、実行下ユーザーのログを残すシステム）
- sudoの設定（sudo visudo -f /etc/sudoers.d/sudo_config）
sudoの設定、特にrootユーザーのパスワードが無効化した状態でsudoが壊れると、管理者権限になれなくなり修正不能。
visudoは、保存前に文法チェックが走る。
sudo visudoを打つと自動でメインの設定ファイル（/etc/sudoers）を開くため、-fフラグ
```
root@yuonishi42:/etc#cat /etc/sudoers.d/sudo_config
Defaults	passwd_tries=3
Defaults	badpass_message="Wrong Password!!"
Defaults	logfile="/var/log/sudo/sudo_config"
Defaults	log_input, log_output
Defaults	iolog_dir="/var/log/sudo"
Defaults	requiretty
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```
適当なsudoでデバッグ。
### パスワードポリシー
- パスワードの有効期限編集
- パスワードモジュールPAMのインストール（デバッグ：passwdでパスワード変更）
## Phase3.
# Resoureces
[Markdown記法 チートシート - Qiita](https://qiita.com/Qiita/items/c686397e4a0f4f11683d)
