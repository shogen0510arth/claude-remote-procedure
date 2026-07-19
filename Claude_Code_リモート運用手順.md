# Claude Code Remote Control 運用手順

更新日：2026年7月18日  
対象端末：Mac mini M4／MacBook Air M5／iPhone 15 Pro  
対象者：プログラミング初心者を含む、ターミナルを日常操作に使いたくない利用者

## この手順書の目的

オフィスのMac miniをClaude Codeの常時稼働環境として使用し、MacBook AirまたはiPhoneから、画面共有を使わずにアプリ開発を継続する。

- コードの実行・ファイルの保存：Mac mini
- 指示・進捗確認・承認：MacBook AirまたはiPhone
- 会話・作業ログ：ClaudeのWeb／アプリで確認
- ソースコードの履歴・バックアップ：GitとGitHub

---

# 0. 最初に覚える3つの原則

## 原則1：アプリ本体は「フォルダ」、セッションは「作業の会話」

例えば、次のフォルダがアプリ本体である。

```text
~/Developer/hotel-app/
```

Claude Codeのセッションは、そのアプリについてClaudeと行う個別の会話・作業記録である。セッションが複数に分かれても、同じフォルダを対象にしていれば、コードやファイルは同じアプリに保存される。

ただし、複数セッションが同じファイルを同時に編集すると上書きや矛盾が発生する可能性がある。原則として、1つのアプリにつき同時に編集するメインセッションは1つにする。

## 原則2：継続開発は必ずプロジェクトフォルダから始める

継続中のアプリを扱うときは、Mac miniで必ずそのアプリのフォルダへ移動してからClaude CodeまたはRemote Controlを起動する。

```bash
cd ~/Developer/hotel-app
claude
```

Claude Codeのセッション履歴はプロジェクトディレクトリに紐づく。そのため、プロジェクトごとに`cd`してから起動すると、`claude --resume`で表示される履歴が原則としてそのプロジェクト関連に絞られ、別アプリの履歴によるノイズが減る。

## 原則3：会話履歴だけに頼らず、重要情報をアプリの中へ残す

各アプリには、最低限次のファイルを用意する。

```text
~/Developer/hotel-app/
├── .git/
├── CLAUDE.md
├── README.md
├── STATUS.md
├── docs/
│   └── DECISIONS.md
└── src/
```

| ファイル | 役割 |
| --- | --- |
| `CLAUDE.md` | Claudeが毎回守るルール、対象範囲、禁止事項 |
| `README.md` | アプリの目的、構成、起動方法 |
| `STATUS.md` | 現在の実装状況、未解決事項、次に行う作業 |
| `docs/DECISIONS.md` | 重要な設計判断と、その理由 |
| `.git/` | コードの変更履歴と復旧手段 |

セッションは「会議」、上記ファイルとGitは「正式な議事録・成果物」と考える。

---

# 1. 履歴に関する正しい理解

## 1-1. `claude history`について

2026年7月時点のClaude Code標準CLIには、`claude history`という公式コマンドはない。環境によって実行できる場合は、独自のalias、スクリプト、プラグイン等である可能性がある。

標準機能で履歴を確認・再開するときは、次を使用する。

| 操作 | コマンド |
| --- | --- |
| 現在のプロジェクトの直近セッションを再開 | `claude -c`または`claude --continue` |
| 現在のプロジェクトの履歴一覧を開く | `claude --resume` |
| 名前を指定して再開 | `claude --resume hotel-app-main` |
| Claude Code起動中に履歴一覧を開く | `/resume` |

## 1-2. なぜプロジェクトごとに履歴が絞られるのか

Claude Codeのセッションは、開始したプロジェクトディレクトリごとに保存される。例えば次のように分けて起動する。

```bash
cd ~/Developer/hotel-app
claude
```

```bash
cd ~/Developer/corporate-site
claude
```

この場合、`hotel-app`で`claude --resume`を実行すると、通常は`hotel-app`のセッションが中心に表示される。

セッション選択画面では、必要に応じて表示範囲を広げられる。

- `Ctrl + A`：Mac mini内のすべてのプロジェクトを表示
- `Ctrl + W`：同じGitリポジトリのすべてのworktreeを表示
- `Ctrl + B`：現在のGitブランチに絞る

## 1-3. 現在の「Developer開発ハブ」方式の注意点

次のように`~/Developer`でサーバーを起動し、`--spawn same-dir`を使用すると、そこで作られたセッションの開始地点はすべて`~/Developer`になる。

```bash
cd ~/Developer

claude remote-control \
  --name "Mac mini 開発ハブ" \
  --spawn same-dir \
  --capacity 10 \
  --sandbox
```

プロンプトで`~/Developer/hotel-app`へ移動させても、セッション履歴の分類は最初に起動したディレクトリの影響を受ける。そのため、すべてを`Developer`ハブから継続開発すると、複数アプリの履歴が同じ場所に集まりやすい。

今後は次のように役割を分ける。

- `Developer`開発ハブ：新しいアプリのフォルダ作成、短い調査、臨時作業
- プロジェクト別Remote Control：既存アプリの継続的な開発

---

# 2. 推奨する全体構成

```text
Mac mini
│
├── Developer開発ハブ
│   └── 新規アプリの作成・臨時作業
│
├── ~/Developer/hotel-app/
│   └── hotel-app専用Remote Control
│
├── ~/Developer/corporate-site/
│   └── corporate-site専用Remote Control
│
└── ~/Developer/reservation-system/
    └── reservation-system専用Remote Control
```

すべてのプロジェクトサーバーを常時起動する必要はない。頻繁に使用するアプリだけ常時起動し、それ以外は必要なときにMac miniで起動する。

---

# 3. 初期設定

## 3-1. Mac mini

### macOSの設定

- Mac本体をスリープさせない。
- ディスプレイは消灯してよい。
- ログインしたまま画面をロックする。
- インターネット接続を維持する。
- 可能であればUPSを使用する。

FileVaultが有効な場合、再起動後は利用者が一度Mac miniへログインするまでRemote Controlを起動できない。セキュリティ上、FileVaultを無効にするより、再起動時だけ手動ログインする運用を推奨する。

### 開発用親フォルダ

```text
~/Developer/
```

`~/Developer`はアプリを格納する親フォルダであり、各アプリはその直下に個別のフォルダを持つ。

### Claude Codeのログイン

初回のみ次を実行し、MacBook Air・iPhoneと同じClaudeアカウントでログインする。

```bash
cd ~/Developer
claude
```

表示されたログインとWorkspace Trustを完了する。

### 新規作成用Developerハブ

新しいアプリの作成場所を用意するため、次を起動しておく。

```bash
cd ~/Developer

claude remote-control \
  --name "Mac mini 開発ハブ" \
  --spawn same-dir \
  --capacity 3 \
  --sandbox
```

DeveloperハブのCapacityは、新規作成と臨時作業が中心なので、通常は3程度で十分である。既存アプリの継続開発は、後述するプロジェクト別環境を使用する。

### プロジェクト別Remote Control

既存アプリを継続開発するときは、別のターミナルウインドウを開き、アプリのルートフォルダでサーバーを起動する。

```bash
cd ~/Developer/hotel-app

claude remote-control \
  --name "hotel-app-main" \
  --spawn same-dir \
  --capacity 3 \
  --sandbox
```

これにより、Web／アプリから作成するセッションは`~/Developer/hotel-app`から始まり、履歴もこのプロジェクトに紐づきやすくなる。

ターミナルは閉じない。`Control + C`、ターミナル終了、Macのログアウト・再起動、Claude Codeプロセスの終了が発生するとRemote Controlも終了する。

## 3-2. MacBook Air

1. [claude.ai/code](https://claude.ai/code)を開く。
2. Mac miniと同じClaudeアカウントでログインする。
3. 新規作成時は`Remote Control → Developer`を選ぶ。
4. 既存アプリの開発時は、対象プロジェクト名が表示されたRemote Control環境を選ぶ。
5. コンピューターアイコンと緑色の接続表示を確認する。

`Default`はAnthropicのクラウド環境であり、Mac miniのローカルファイルを直接操作する環境ではない。

## 3-3. iPhone

1. Claude公式アプリを最新版にする。
2. Mac miniと同じClaudeアカウントでログインする。
3. iOSの「設定 → 通知 → Claude」で通知を許可する。
4. Claudeアプリの「Code」を開く。
5. 対象アプリのRemote Control環境を選ぶ。

iPhoneでは、長いコードを直接編集するより、要件追加、進捗確認、承認、方向修正を中心に行う。

---

# 4. 新しいアプリを作る

## 4-1. MacBook AirまたはiPhoneから新規作成する

1. ClaudeのWeb／アプリで新規セッションを作る。
2. `Remote Control → Developer`を選ぶ。
3. 次のプロンプトを送る。

```text
Mac miniのローカル環境で作業してください。

~/Developer直下に「hotel-app」という新しいフォルダを作成してください。
このアプリに関するファイルは、すべて
~/Developer/hotel-app
の中だけに作成してください。

最初に以下を行ってください。

1. フォルダを作成
2. Gitを初期化
3. CLAUDE.mdを作成
4. README.mdを作成
5. STATUS.mdを作成
6. docs/DECISIONS.mdを作成

実装前に、要件と実装方針を提示してください。
私の承認を受けるまで本格的な実装は開始しないでください。
```

## 4-2. 作成後にプロジェクト専用環境へ切り替える

アプリのフォルダができたら、Mac miniでプロジェクト専用Remote Controlを起動する。

```bash
cd ~/Developer/hotel-app

claude remote-control \
  --name "hotel-app-main" \
  --spawn same-dir \
  --capacity 3 \
  --sandbox
```

以後、`hotel-app`の開発ではDeveloperハブではなく、`hotel-app`のRemote Control環境を選ぶ。

---

# 5. セッションの作り方

## 5-1. 基本方針

1つのプロジェクトにつき、まずメインセッションを1つ作る。

```text
/rename hotel-app-main
```

新しいセッションを作るのは、次の場合に限定する。

- メインセッションが終了して復旧できない。
- 完全に別の機能を並行して検討する。
- 調査だけを分離したい。
- Git worktreeや別ブランチで安全に並行作業する。

会話が長くなっただけなら、新しいセッションを作らず`/compact`を使う。

```text
/compact 今後の開発に必要な仕様、設計判断、現在の実装状況、未完了作業を優先して整理してください
```

## 5-2. 新規セッション開始時のプロンプト

```text
このセッションでは、現在のプロジェクトフォルダだけを対象にしてください。
ほかのプロジェクトやディレクトリは変更しないでください。

最初に以下を確認してください。

- 現在の作業ディレクトリ
- CLAUDE.md
- README.md
- STATUS.md
- docs/DECISIONS.md
- git status
- 直近のgit log

現在の実装状況、未解決事項、次に行うべき作業を報告してください。
私が確認するまでファイルを変更しないでください。
```

## 5-3. セッション名の付け方

```text
プロジェクト名__役割
```

例：

```text
hotel-app__main
hotel-app__login-fix
hotel-app__payment-research
```

セッション内で次を実行する。

```text
/rename hotel-app__main
```

---

# 6. 既存セッションを継続する

## 6-1. Web／iPhoneでセッションがオンラインの場合

新規セッションを作らず、セッション一覧から既存のメインセッションを開く。

- コンピューターアイコンがある。
- 緑色の接続表示がある。
- セッション名が`hotel-app__main`等になっている。

スマホアプリを閉じた、ブラウザを閉じた、一時的に通信が切れた、というだけなら新規セッションは不要である。Remote Controlは短時間の通信中断から自動再接続する。

## 6-2. Mac miniで直近セッションを再開する

```bash
cd ~/Developer/hotel-app
claude -c
```

`claude -c`は、現在のプロジェクトディレクトリで直近に使用したセッションを再開する。

## 6-3. 履歴一覧から選んで再開する

```bash
cd ~/Developer/hotel-app
claude --resume
```

一覧から対象セッションを選ぶ。名前が分かる場合は直接指定できる。

```bash
cd ~/Developer/hotel-app
claude --resume hotel-app__main
```

再開時、Claude Codeは以前のRemote Controlへの再接続を試みる。再接続できない場合は、セッション内で次を実行する。

```text
/remote-control
```

`/resume`は対話式の選択画面を開くため、Web版やiPhoneアプリではなくMac miniのローカルCLIで使用する。

## 6-4. Remote Controlサーバーとして直近セッションを継続する

Claude Code v2.1.200以降では、プロジェクトフォルダから次を実行できる。

```bash
cd ~/Developer/hotel-app
claude remote-control --continue --sandbox
```

これは、そのディレクトリから開始された直近のRemote Controlセッションを再開する。

ただし、`--continue`は次のオプションと同時に使用できない。

- `--spawn`
- `--capacity`
- `--create-session-in-dir`
- `--session-id`

したがって、`--continue`は「特定の1セッションを継続する運用」、`--spawn same-dir --capacity 3`は「そのプロジェクトで複数セッションを作れるサーバー運用」と考える。

---

# 7. 通常の運用

## 7-1. Mac mini

- 電源を入れたままにする。
- Mac本体をスリープさせない。
- 使用中プロジェクトのRemote Controlターミナルを閉じない。
- 画面はロックまたは消灯してよい。
- インターネット接続を維持する。

## 7-2. MacBook Air

1. [claude.ai/code](https://claude.ai/code)を開く。
2. 対象プロジェクトのRemote Control環境を選ぶ。
3. 既存のメインセッションを開く。
4. Claudeの計画、差分、コマンドを確認する。
5. 実装後にテスト結果とGitの状態を確認する。

## 7-3. iPhone

1. Claudeアプリの「Code」を開く。
2. 対象プロジェクトのメインセッションを開く。
3. 進捗確認、追加指示、承認を行う。
4. 作業終了時に、後述のまとめプロンプトを送る。

## 7-4. 1回の作業の標準的な流れ

1. 対象プロジェクトの既存メインセッションを開く。
2. `STATUS.md`とGitの状態を確認させる。
3. 実装計画を提示させる。
4. 計画を確認して実装を承認する。
5. テストを実行させる。
6. 変更ファイルと差分を確認する。
7. `STATUS.md`等を更新させる。
8. 必要に応じてGitへコミットする。
9. GitHubの非公開リポジトリへpushする。

---

# 8. 作業終了時のまとめ方

## 8-1. 一時的に離れるだけの場合

`/exit`せず、次のプロンプトを送って作業状況を整理する。

```text
ここで一旦作業を止めます。

現在の状況を確認し、STATUS.mdを更新してください。

以下を記載してください。

1. 今回完了した内容
2. 変更したファイル
3. 実行したテストと結果
4. 未解決の問題
5. 次に行う作業
6. 再開時に注意すること

最後にgit statusを確認し、未コミットの変更を報告してください。
私の承認なしにコミット、push、本番公開は行わないでください。
```

アイドル状態でClaudeがタスクを実行していなければ、トークンを継続的に消費しない。ただし、起動中セッションとしてCapacityとMac miniのメモリを使用する。

## 8-2. 会話が長くなった場合

新しいセッションを作る前に、次を実行する。

```text
/compact 今後必要な仕様、設計判断、変更済みファイル、未解決事項、次の作業を優先して要約してください
```

## 8-3. その日の作業を完全に終了する場合

次の順で確認する。

1. `STATUS.md`が更新されている。
2. `docs/DECISIONS.md`に重要な判断が残っている。
3. テストが完了している。
4. `git status`と`git diff`を確認した。
5. 必要な変更をコミットした。
6. GitHubへpushした。
7. セッションを終了してよいと判断した。

終了する場合：

```text
/exit
```

後日、Mac miniでプロジェクトフォルダへ`cd`してから`claude -c`または`claude --resume`で再開する。

---

# 9. 複数セッションに分かれた場合

2つのセッションの会話履歴を、標準機能で1本の履歴へ直接結合することはできない。ただし、作業内容はメインセッションへ引き継げる。

## 9-1. メインセッションを決める

```text
/rename hotel-app__main
```

もう一方は次のようにする。

```text
/rename hotel-app__old
```

## 9-2. 古いセッションで引き継ぎ書を作る

```text
このセッションの内容をメインセッションへ引き継ぎます。

docs/SESSION_HANDOFF.mdを作成し、以下を記載してください。

1. 決定した要件
2. 設計判断と理由
3. 実装した内容
4. 変更したファイル
5. 未完了の作業
6. 問題点
7. テスト結果
8. 次に行う作業

最後にgit statusとgit diffも確認してください。
新しい実装は行わず、引き継ぎ情報の整理だけをしてください。
```

## 9-3. メインセッションで統合する

```text
docs/SESSION_HANDOFF.md、STATUS.md、CLAUDE.md、README.md、
git status、git diff、直近のgit logを確認してください。

別セッションの変更による矛盾、上書き、未完成の実装がないか確認し、
今後の作業方針を報告してください。
確認が終わるまで新しい実装は行わないでください。
```

統合確認が終わるまで古いセッションを`/exit`しない。

---

# 10. Capacityの考え方

`Capacity: 1/3`は、そのRemote Controlサーバーが最大3セッションを扱え、現在1セッションが起動していることを意味する。

Capacityは次の上限ではない。

- トークンを消費中のタスク数
- 接続しているiPhoneやMacBook Airの台数
- 会話履歴の保存件数

Capacityは、そのサーバーが同時に保持するClaude Codeセッション数の上限である。

プロジェクト別環境では、通常`3`程度から始める。

- メインセッション：1
- 調査・修正用：1
- 予備：1

`Capacity: 3/3`になった場合は、完全に終了してよいセッションだけを整理する。後でスマホから続けたい重要なセッションは、むやみに`/exit`しない。

---

# 11. トラブルシューティング

## 11-1. プロジェクトの履歴に別アプリが大量に表示される

原因として、`~/Developer`のような共通親フォルダからすべてのセッションを起動している可能性がある。

Mac miniで対象アプリへ移動してから履歴を開く。

```bash
cd ~/Developer/hotel-app
claude --resume
```

今後は、そのアプリ専用のRemote Controlサーバーをアプリのルートフォルダから起動する。

## 11-2. `Developer`しか表示されない

プロジェクト専用Remote Controlがまだ起動していない。Mac miniで別ターミナルを開き、対象フォルダから起動する。

```bash
cd ~/Developer/hotel-app

claude remote-control \
  --name "hotel-app-main" \
  --spawn same-dir \
  --capacity 3 \
  --sandbox
```

## 11-3. Mac miniのファイルが見つからない

実行環境が`クラウド → Default`になっていないか確認する。Mac miniのファイルを扱う場合は、コンピューターアイコンのあるRemote Control環境を選ぶ。

## 11-4. 意図しないフォルダを変更した

次を確認する。

```text
現在の作業ディレクトリを絶対パスで報告してください。
git statusとgit diffを表示してください。
これ以上ファイルを変更しないでください。
```

ユーザーが差分を確認するまで、広範囲な削除や自動修正を行わせない。

## 11-5. Remote Controlが切れた

- iPhoneアプリやブラウザだけを閉じた：既存セッションを開く。
- 短時間の通信切断：自動再接続を待つ。
- Mac miniのClaudeプロセスが終了：プロジェクトフォルダから`claude -c`または`claude --resume`を実行する。
- Mac miniが起動している状態で約10分以上ネットワークへ接続できない：プロセスが終了することがあるため再起動する。

## 11-6. `/exit`したセッションをスマホだけで復旧したい

同じローカルセッションを確実に指定して復旧する`/resume`はMac miniのローカルCLI専用である。

Mac miniを操作できない場合は、対象プロジェクトのRemote Control環境で新規セッションを作り、次を送る。

```text
CLAUDE.md、README.md、STATUS.md、docs/DECISIONS.md、
git status、git diff、直近のgit logを確認してください。

現在の実装状況と次に行う作業を報告してください。
確認が終わるまでファイルを変更しないでください。
```

## 11-7. 複数セッションが同じファイルを変更した

すべてのセッションで作業を停止し、メインセッションに次を依頼する。

```text
git statusとgit diffを確認してください。
変更を自動修正せず、競合・矛盾・未完成の箇所を報告してください。
```

並行作業が必要な場合は、Gitブランチとworktreeを使用する。初心者のうちは、同じアプリを複数セッションで同時編集しない。

## 11-8. 緊急停止

1. Web／アプリの停止ボタンを押す。
2. Mac miniの該当Remote Control画面で`Control + C`を押す。
3. `git status`と`git diff`を確認する。
4. 意図しない外部送信があれば、関連サービスの認証情報を無効化する。

---

# 12. コマンド早見表

| 目的 | 操作 |
| --- | --- |
| プロジェクトへ移動 | `cd ~/Developer/hotel-app` |
| 新規の通常セッション | `claude` |
| 直近セッションを継続 | `claude -c` |
| 履歴一覧から再開 | `claude --resume` |
| 名前で再開 | `claude --resume hotel-app__main` |
| 起動中にセッションを切り替える | `/resume` |
| セッション名を付ける | `/rename hotel-app__main` |
| 会話を要約して継続 | `/compact 指示` |
| Remote Controlを再接続 | `/remote-control` |
| セッションを終了 | `/exit` |
| 現在の作業場所を確認 | `pwd` |
| ファイルの変更状況を確認 | `git status` |
| 変更内容を確認 | `git diff` |

---

# 参考リンク

- [Claude Code Remote Control](https://code.claude.com/docs/en/remote-control)
- [Claude Code Sessions](https://code.claude.com/docs/en/sessions)
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [How Claude Code Works](https://code.claude.com/docs/en/how-claude-code-works)
- [Claude Code Desktop](https://code.claude.com/docs/en/desktop)
- [Claude Code on the Web](https://code.claude.com/docs/en/claude-code-on-the-web)
- [Apple：Macデスクトップコンピュータのエネルギー設定](https://support.apple.com/guide/mac-help/change-energy-settings-mchlp1168/mac)
