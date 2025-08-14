# 3 標準入出力とフィルタコマンド

Linux のコマンドは「処理を行うデータの入力」と「処理結果の出力」を持っており、これを**標準入出力**と呼ぶ。  
複数のコマンドの標準入出力を組み合わせて処理することで、必要な結果を得ることができる。このようなコマンドは**フィルタコマンド**と呼ばれる。

## 本章の内容

- 標準入出力
- リダイレクト
- 標準エラー出力
- パイプ
- データの先頭や末尾の表示（head・tail コマンド）
- テキストファイルのソート（sort コマンド）
- 行の重複の消去（uniq コマンド）
- 文字をカウントする（wc コマンド）
- 文字列を検索する（grep コマンド）

## 3-1 標準入出力

Linux の多くのコマンドは、以下の 3 種類の入出力を持つ。

### 標準入力（stdin）

- コマンドに渡されるデータ
- デフォルトはキーボード

### 標準出力（stdout）

- コマンドの実行結果の出力先
- デフォルトはコンソール

### 標準エラー出力（stderr）

- エラーメッセージの出力先
- デフォルトはコンソール

例：`ls`コマンドを実行すると、カレントディレクトリのファイル・ディレクトリ一覧がコンソールに表示される。

---

## 3-2-1 リダイレクト（標準出力の書き込み）

- 標準出力は通常コンソールに表示される
- リダイレクトを使うと標準出力をファイルに書き込める
- リダイレクトの書式：

コマンド > 出力先ファイル

- 作業用ディレクトリを作成

```bash

$ cd
$ mkdir work
$ cd work
$ touch test

```

- リダイレクトなしで ls を実行

```bash

$ ls
test

```

- 標準出力をファイルにリダイレクト

```bash

$ ls > ls-output
$ ls
ls-output test
$ cat ls-output
test

```

リダイレクトにより、コマンドの実行結果はファイルに書き込まれ、コンソールには表示されない

リダイレクト先のファイル（ls-output）はコマンド実行前に作成されるため、ls の結果に自身も含まれる

---

## 3-2-2 標準出力の追加リダイレクト（アペンド）

- 通常のリダイレクト（`>`）は、既存ファイルがある場合に上書きされる
- 以前の結果を残して追記する場合は、アペンド（`>>`）を使用する

### 書式

コマンド >> 出力先ファイル

- 通常のリダイレクト（上書き）

```bash

$ ls > ls-output
$ cat ls-output
ls-output
test

```

- 既存ファイルの内容は上書きされる

- 追加リダイレクト（追記）

```bash

$ ls >> ls-output
$ cat ls-output
ls-output
test
ls-output
test

```

2 回目以降のコマンド実行結果は、既存内容の末尾に追加される

---

## 3-2-3 cat コマンドによるファイル作成

- `cat` コマンドは引数なしで実行すると、標準入力から受け取ったデータをそのまま標準出力に表示する
- 標準出力をリダイレクトすることで、キーボード入力をファイルに保存可能

### 書式

cat > 出力先ファイル

```bash

$ cat > cat-output
Hello!
This is cat redirect.
# 入力終了は Ctrl + d
$ cat cat-output
Hello!
This is cat redirect.

```

Ctrl + d は EOF（End Of File）を意味し、入力終了を示す

入力した内容が指定ファイルに書き込まれる

---

## 3-3-1 標準エラー出力

- コマンド実行時のエラーは標準エラー出力（番号 2）に出力される
- 標準出力（番号 1）とは別扱いでき、リダイレクトでファイルに保存可能

### 標準エラー出力のリダイレクト

コマンド 2> 出力先ファイル

実例

```bash

$ ls nodir
ls: 'nodir' にアクセスできません: そのようなファイルやディレクトリはありません

$ ls nodir 2> ls-error
$ cat ls-error
ls: 'nodir' にアクセスできません: そのようなファイルやディレクトリはありません

```

メッセージはファイルに書き込まれ、コンソールには表示されない

標準出力と標準エラー出力を別々にリダイレクト

コマンド > 標準出力先ファイル 2> 標準エラー出力先ファイル

実例

```bash

$ ls -R /etc > ls-etc 2> ls-etc-error
$ cat ls-etc
/etc:
DIR_COLORS
（略）
almalinux-saphana.repo

$ cat ls-etc-error
ls: ディレクトリ '/etc/audit' を開くことが出来ません: 許可がありません
ls: ディレクトリ '/etc/sudoers.d' を開くことが出来ません: 許可がありません

```

正常な出力は標準出力ファイルに、エラーメッセージは標準エラー出力ファイルに分けて保存できる

ls-etc には正常に実行された結果、ls-etc-error にはエラーメッセージがリダイレクトされています。

---

## 3-3-2 標準出力と標準エラー出力のまとめリダイレクト

- 標準出力と標準エラー出力をまとめて 1 つのファイルに書き込みたい場合は `2>&1` を使用
- 「標準エラー出力（2）を標準出力（1）に混ぜる」という意味

### 書式

コマンド > 標準出力先ファイル 2>&1

実例

```bash

$ ls -R /etc > ls-etc-mix 2>&1
$ less ls-etc-mix

```

- ls-etc-mix には正常出力とエラーメッセージがまとめて書き込まれる

- エラーが発生した場所に応じてファイル内に混在する

- less で閲覧後は q で終了

---

## 3-3-3 標準入力へのリダイレクト

ファイルの内容を標準入力としてコマンドに渡すことも可能

### 書式

コマンド < ファイル

実例

```bash

$ less < ls-etc-mix

```

多くのコマンドは引数でファイルを指定できるため、標準入力リダイレクトはあまり使われない

---

## 3-4 パイプ

- パイプを使うと、あるコマンドの標準出力を別のコマンドの標準入力として渡せる
- 複数のコマンドを組み合わせて処理可能
- パイプで受け取る側のコマンドを「フィルタ」と呼ぶ

### 書式

コマンド 1 | コマンド 2

- 「|」でパイプを指定（日本語キーボードでは Shift + 円マークキー）

- コマンド 1 の出力がコマンド 2 の入力として渡される

- このデータの流れは「テキストストリーム」と呼ばれる

---

## 3-4-1 less コマンドによるページング

- `less` コマンドは標準入力のデータをページごとに表示する
- 長い出力結果を遡って確認するのに便利
- 標準出力と標準エラー出力をまとめて表示する場合は `2>&1` を使用

### 例

```bash

# 通常のページング
$ cat /etc/services | less

# ディレクトリ内を再帰表示してページング
$ ls -R /etc | less

# 標準出力と標準エラー出力をまとめてページング
$ ls -R /etc 2>&1 | less

```

- 端末によってはバックスクロール可能だが、表示行数に制限がある場合がある

- 2>&1 を使わないとエラーメッセージがスクロールで消えてしまうことがある

---

## 3-5 データの先頭や末尾の表示 (head / tail)

- `head` コマンド：データの先頭を表示（デフォルト 10 行）
- `tail` コマンド：データの末尾を表示（デフォルト 10 行）
- 行数を指定する場合は `-n 行数` オプションを使用

### 例

```bash
# /etc/services の先頭5行を表示
$ cat -n /etc/services | head -n 5
1  # /etc/services:
2  # $Id: services ,v 1.49 2017/08/18 12:43:23 ovasik Exp $
3  #
4  # Network services , Internet style
5  # IANA services version: last updated 2016 -07 -08

# /etc/services の末尾10行を表示（デフォルト）
$ tail /etc/services

# 末尾から5行を表示
$ tail -n 5 /etc/services

```

### 例（末尾の一部を表示）

```bash

# /etc/services の末尾5行を表示（-行数オプション指定）
$ cat -n /etc/services | tail -5
11469  axio-disc       35100/udp    # Axiomatic discovery protocol
11470  pmwebapi        44323/tcp    # Performance Co-Pilot client HTTP API
11471  cloudcheck-ping 45514/udp    # ASSIA CloudCheck WiFi Management keepalive
11472  cloudcheck      45514/tcp    # ASSIA CloudCheck WiFi Management System
11473  spremotetablet  46998/tcp    # Capture handwritten signatures

```

---

## 3-6 テキストファイルのソート（sort コマンド）

sort コマンドはデータを指定した条件で並べ替え（ソート）します。

### 書式

sort [オプション] [ファイル]

主なオプション

```bash

-k n : n 列目のデータでソート

-t 文字列 : 列の区切り文字を指定（デフォルトは空白）

-n : 数値としてソート

-r : 逆順でソート

```

---

## 6.6.1 ソート用データの確認

- Linux のユーザー情報が記載されている /etc/passwd を使用します。

- 末尾 5 行を表示してデータを確認します。

```bash

$ tail -5 /etc/passwd
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
chrony:x:982:981:chrony system user:/var/lib/chrony:/sbin/nologin
dnsmasq:x:981:980:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/usr/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
linuc:x:1000:1000:LinuC:/home/linuc:/bin/bash

```

先頭がユーザー名、3 番目がユーザー ID 番号です。これらのデータを使ってどのようにソートが変わるのか確認します。

---

## 3-6-2 単純なソート

ユーザー名（1 列目）をアルファベット順にソートします。

```bash

$ tail -5 /etc/passwd | sort
chrony:x:982:981:chrony system user:/var/lib/chrony:/sbin/nologin
dnsmasq:x:981:980:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/usr/sbin/nologin
linuc:x:1000:1000:LinuC:/home/linuc:/bin/bash
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin

```

結果はユーザー名のアルファベット順に並んでいる。

---

## 3-6-3 逆順のソート

ユーザー名をアルファベットの逆順でソートします。

```bash

$ tail -5 /etc/passwd | sort -r
tcpdump:x:72:72::/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
linuc:x:1000:1000:LinuC:/home/linuc:/bin/bash
dnsmasq:x:981:980:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/usr/sbin/nologin
chrony:x:982:981:chrony system user:/var/lib/chrony:/sbin/nologin

```

結果はユーザー名のアルファベット逆順で並んでいる。

---

## 3-6-4 列を指定したソート

3 列目（ユーザー ID）でソートを行います。区切り文字は「:」に指定します。

```bash

$ tail -5 /etc/passwd | sort -k 3 -t :
linuc:x:1000:1000:LinuC:/home/linuc:/bin/bash
tcpdump:x:72:72::/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
dnsmasq:x:981:980:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/usr/sbin/nologin
chrony:x:982:981:chrony system user:/var/lib/chrony:/sbin/nologin

```

文字列としてソートされるため、数値の大小順にはならず、1000 が先頭に来てしまう。

---

## 3-6-5 数値としてのソート

3 列目（ユーザー ID）を数値としてソートするには `-n` オプションを追加します。

```bash

$ tail -5 /etc/passwd | sort -k 3 -t : -n
tcpdump:x:72:72::/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
dnsmasq:x:981:980:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/usr/sbin/nologin
chrony:x:982:981:chrony system user:/var/lib/chrony:/sbin/nologin
linuc:x:1000:1000:LinuC:/home/linuc:/bin/bash

```

数値順にソートされ、ユーザー ID の小さい順に並ぶ。

---

## 3-7 行の重複の消去（uniq コマンド）

`uniq` コマンドは連続する重複行を 1 行にまとめて出力します。

### 3-7-1 データ作成例

```bash

$ cat > uniq-test
ABACCD
（Ctrl+d で入力終了）
$ cat uniq-test
ABACCD

```

### 3-7-2 重複行の消去

```bash

$ cat uniq-test | uniq
ABACD

```

- 前後で連続する重複文字だけが削除される。

- 先に sort すると全体の重複も削除可能。

```bash

$ cat uniq-test | sort | uniq
ABCD

```

今度は A も重複が消去されました。

---

## 3-8 文字をカウントする（wc コマンド）

`wc` コマンドはファイルや標準入力の文字数・行数・単語数をカウントできます。

### 書式

wc [オプション] [ファイル]

主なオプション

- -c 文字数をカウント

- -l 行数をカウント

- -w 単語数をカウント

---

## 3-8-1 文字のカウント

`wc` コマンドを使うと、文字数・行数・単語数をカウントできます。

- オプションなしでは、行数・単語数・文字数がまとめて表示される

```bash

$ cat /etc/services | wc
11473 63129 692252

```

行数のみをカウント

```bash

$ cat /etc/services | wc -l
11473

```

単語数のみをカウント

```bash

$ cat /etc/services | wc -w
63129

```

文字数のみをカウント

```bash

$ cat /etc/services | wc -c
692252

```

---

## 3-9 grep コマンドによる文字列検索

`grep` コマンドは、指定した条件に一致する文字列をファイルや標準入力から検索します。

### 書式

grep [オプション] 検索条件 [ファイル]

正規表現の利用

```bash

^ : 行頭

$ : 行末

. : 任意の1文字

* : 直前の文字の0回以上の繰り返し

[ ... ] : 中の任意の1文字

[^ ... ] : 中の文字以外の任意1文字

\ : 正規表現の特殊文字をエスケープ

```

正規表現例

```bash

^a : 行頭が a

b$ : 行末が b

a.b : a と b の間に1文字

[ab]ab : a または b に続く ab (aab、bab)

[^ab]ab : a または b で始まらない ab (xab、zab)

```

---
