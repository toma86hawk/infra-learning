# 環境構築メモ

第1週の成果物です。実行した結果と学んだことを、各手順の「記録」に自分の言葉で書きます。
公開前に、実IPアドレスやアカウント名が入っていないか見直します。

## 開始時点の状態（2026-11-06）

| 項目 | 状態 | 対応 |
| --- | --- | --- |
| OSビルド | 19045.5011（2024年10月の更新で停止） | 手順1で更新 |
| 配信待ちの更新 | KB5066791（2025年10月の累積更新）ほか | 手順1で適用 |
| ESU | 未登録 | 手順1で登録 |
| WSL | 機能が未有効 | 手順2で導入 |
| VirtualBox | 7.1.4導入済み、VMなし | 今後のVMはHyper-Vに統一 |
| Wireshark | 未導入（Npcap 1.79のみ導入済み） | 手順3で導入 |
| C:の空き | 30.7GB | WSLの置き場所はE:にする |

## 手順1：Windows Updateを適用してESUに登録する

順番が要件です。累積更新の次に準備パッケージが入り、そのあとで登録ボタンが出ます。

1. 設定 > 更新とセキュリティ > Windows Update で更新を確認し、KB5066791をインストールして再起動する。「オプションの更新プログラム」のドライバーは入れない。
2. KB5126256（ESU準備パッケージ）が自動で入るのを待つ。インストール後にPCが自動で再起動する。来ない場合は [Microsoft Update カタログ](https://www.catalog.update.microsoft.com/Search.aspx?q=KB5126256) から入手する。
3. Windows Updateの画面に出る「今すぐ登録」を押し、PC設定の同期を使う無料の選択肢で登録する。
4. もう一度更新を確認し、ESUの更新を入れる。
5. `winver` でビルド番号を確認する。

記録：

- 2026-11-05：KB5066791を適用。初回は失敗（結果コード5）し、再試行で成功。ビルドは19045.5011から19045.6456になった。
- 2026-11-06：KB5126256がWindows Updateに配信された。
- 2026-11-07：KB5126256とKB5071959（2025年11月の累積更新）を適用して再起動。ビルドは19045.6466になった。
- 登録できた日：2026-11-07（ConsumerESUのESUEligibilityが 6→2→3 と変化。2が登録可能、3が登録済みと読める）
- 学んだこと：

## 手順2：WSL2とUbuntuを入れる

`%UserProfile%\.wslconfig` に、メモリ8GB、スワップ4GB、保存先E:\WSLを設定済みです。

1. PowerShellを管理者として開き、`wsl --install -d Ubuntu` を実行する。
2. 再起動する。
3. Ubuntuを起動し、Linux用のユーザー名とパスワードを決める。
4. 下の確認コマンドを実行する。

```powershell
wsl --version
wsl -l -v
Get-ChildItem E:\WSL -Recurse -Depth 2
```

```bash
cat /etc/os-release
ip addr show eth0
ps -p 1 -o comm=
```

確認すること：

- `wsl -l -v` のVERSIONが2になっている。
- E:\WSLの下に `ext4.vhdx` がある。
- `ps -p 1 -o comm=` の結果が `systemd` か `init` か。Windows 10でのsystemdの動作はロードマップの未確認事項。

記録：

- WSLのバージョン：3.0.1.0（カーネル 6.18.40.1、WSL2）
- Ubuntuのバージョン：26.04.1 LTS
- PID 1の名前：systemd（/etc/wsl.conf に systemd=true が自動で入っていた。Windows 10でも動いた）
- 保存先：E:\WSL\distros 配下の ext4.vhdx（約1.4GB）。C:には置かれていない
- WindowsのIPとUbuntuのIPが違う理由：

## 手順3：Wiresharkを入れる

1. インストーラーを進める。wingetが使うMSI版にはNpcapが同梱されない可能性があるので、Npcapの画面が出なくても問題ない。
2. Wiresharkを起動し、使っているネットワークのインターフェースが一覧に出ることを確認する。
3. 一覧に出ない場合や、Npcapを新しくしたい場合は、[Npcap公式](https://npcap.com/#download) の最新版を入れる。

記録：

- 2026-11-06：Wireshark 4.6.9をwingetで導入し、E:\Program Files\Wireshark に置いた。Npcapは導入済みの1.79のまま（ドライバーは動作中）。
- Wiresharkのバージョン：4.6.9
- Npcapのバージョン：1.79
- キャプチャフィルターと表示フィルターの違い：

## 手順4：GitHubに公開する

1. GitHubのアカウントを作り、2要素認証を有効にする。
2. 設定の Emails で「Keep my email addresses private」を有効にし、`ID+ユーザー名@users.noreply.github.com` の形式のアドレスを控える。
3. 中身が空の公開リポジトリ `infra-learning` を作る。READMEは追加しない。
4. このフォルダで下のコマンドを実行する。初回のpushではブラウザーのサインイン画面が出る。

```powershell
git config user.name "GitHubのユーザー名"
git config user.email "控えたnoreplyアドレス"
git add .
git status
git commit -m "Add README and setup log"
git remote add origin https://github.com/ユーザー名/infra-learning.git
git push -u origin main
```

記録：

- commitとpushの違い：
- 草が増えることを確認した日：

## 参考

- [Windows 10 ESU（Microsoft）](https://www.microsoft.com/windows/extended-security-updates)
- [KB5126256（Microsoft）](https://support.microsoft.com/en-us/servicing/os/windows-10/2026/09/kb5126256-windows-10-21h2-22h2-standalone-cbs)
- [WSLのインストール（Microsoft Learn）](https://learn.microsoft.com/ja-jp/windows/wsl/install)
- [WSLの詳細設定（Microsoft Learn）](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)
- [Wireshark ダウンロード](https://www.wireshark.org/download.html)
