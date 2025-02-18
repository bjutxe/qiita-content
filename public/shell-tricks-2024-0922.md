---
title: オレオレシェル芸ワンライナーチートシート
tags:
  - Bash
  - ワンライナー
  - シェル芸
  - チートシート
  - Linuxコマンド
private: false
updated_at: '2024-10-13T11:31:19+09:00'
id: 8b84d6b1ede90cba1966
organization_url_name: null
slide: false
ignorePublish: false
---
# 基本操作や環境周り
## コンソール操作ショートカットキー
`Ctrl` + `L`: clearコマンドと同じ

`alt` + `B`: 1単語分、左に移動
`alt` + `F`: 1単語分、右に移動
`Ctrl` + `W`: 直前の空白までを削除

`Ctrl` + `K`: カーソル右の全てを削除
`Ctrl` + `U`: カーソル左の全てを削除

`Ctrl` + `A`: 行の先頭にカーソル移動
`Ctrl` + `E`: 行の最後にカーソル移動

参考: [linuxショートカットキー一覧 #Linux - Qiita](https://qiita.com/haborym777/items/429ae0cf62ace57d6f1d) @haborym777 さん

## ディストリビューションの確認
`WSL2`では以下で確認している。インストールには`apt`を使用している
```bash
cat /etc/os-release
```
## vi時に文字検索
// TODO: 執筆
`/server` を入力してエンターキーで確定。その後は`n`で次を表示。何回も`n`を押せば良い。
## 変数から文字列を生成
```bash
# 同じパターンの手順を繰り返し実施するときに便利
# 使用する変数を冒頭で確定させてしまうコマンドを流用すると扱いやすい
$ patternname="templateMethod" && patternnum=6 \
> && repositoryname=jdp-${patternname}-`date "+%m%d-%H%M"` \
> && directoryname=jdp-${patternnum}-${patternname} \
> && echo ${repositoryname} && echo ${directoryname}
jdp-templateMethod-0727-1356
jdp-6-templateMethod

# 他のコマンドに埋め込んでそのまま実行できる
$ git clone git@github.com:bjutxe/${repositoryname}.git ${directoryname}

# コマンドをそのまま実行では無くコマンドを作成するのにも良い
$ echo "git remote set-url origin https://bjutxe:ghp_abcde12345@github.com/bjutxe/"${repositoryname}".git"
git remote set-url origin https://bjutxe:ghp_abcde12345@github.com/bjutxe/jdp-templateMethod-0727-1356.git

# 使っているアプリケーションがCLI対応しているのであれば
# 色々と柔軟に組み合わせられる。以下でVSCodeを開ける
$ code ${directoryname}
```
# 標準出力系
## ある行だけを出力する

:::note information
[Man page of SED](https://linuxjm.sourceforge.io/html/GNU_sed/man1/sed.1.html)
sed -n: パターンスペースを出力しない
p: 現在のパターンスペースを出力
:::

```bash
$ sudo cat -n /etc/sudoers | sed -n 46,56p
    46
    47  # Members of the admin group may gain root privileges
    48  %admin ALL=(ALL) ALL
    49
    50  # Allow members of group sudo to execute any command
    51  %sudo   ALL=(ALL:ALL) ALL
    52
    53  # See sudoers(5) for more information on "@include" directives:
    54
    55  @includedir /etc/sudoers.d
    56
```

## 逆にある行だけ出力しない

:::note information
d: パターンスペースを削除します。 次のサイクルを開始します。
:::

```bash
| sed 146,148d
```

# 分析系
## diff ディレクトリ 再帰的 ファイル名のみ
// TODO: 執筆
```bash
# これはフルパスが表示される
$ diff -qrb –exclude=.git –exclude=target –exclude=.idea dir1 dir2 \
> | cut -d" " -f2 | sort | sed -e "s/dir1//g"

# これがファイル名のみ
$ diff -qrb –exclude=.git –exclude=target –exclude=.idea dir1 dir2 \
> | cut -d" " -f2 | sort | xargs -n1 basename
```
## 除外tree
```bash
$ tree -a -I .git -I target -I .vscode -I node_modules
.
├── .gitignore
├── .tool-versions
├── LICENSE.md
├── README.md
├── Tiltfile
├── backend
│   ├── Cargo.lock
│   ├── Cargo.toml
│   ├── Dockerfile
│   └── src
│       └── main.rs
├── docker-compose.yml
└── frontend
    ├── Dockerfile
    ├── README.md
    ├── package-lock.json
    ├── package.json
    ├── public
    │   ├── favicon.ico
    │   ├── index.html
    │   ├── logo192.png
    │   ├── logo512.png
    │   ├── manifest.json
    │   └── robots.txt
    ├── src
    │   ├── App.css
    │   ├── App.test.tsx
    │   ├── App.tsx
    │   ├── index.css
    │   ├── index.tsx
    │   ├── logo.svg
    │   ├── react-app-env.d.ts
    │   ├── reportWebVitals.ts
    │   └── setupTests.ts
    └── tsconfig.json

5 directories, 30 files
```
## ディレクトリ配下の全ファイル内容を吐き出す
// TODO: 執筆
```bash
$ find . -type f -exec cat {} +
```
## 2つのファイルをソートして比較
これは業務で使用したので詳細は割愛しますが、
大量の項目を定義しないといけない設計書などにおいて、
APIレスポンス項目と、画面側項目定義とで表記ゆれがある場合などに使いました。
それぞれのテキストファイルはあらかじめ作成した後にソートして比較すれば、
それなりのタスクに落とし込むことができます。
```bash
$ diff \
> <(cat api-response-property.txt | sort) \
> <(cat ui-js-property.txt | sort)
```
## 重複行を出力
```bash
$ cat watch-repository.txt | sort | uniq --repeated
vuejs/awesome-vue
```
# 検索系
## OR検索

:::note information
[Man page of GREP](https://linuxjm.sourceforge.io/html/GNU_grep/man1/egrep.1.html)
grep -e PATTERN: PATTERN をパターンとして指定します。デフォルトは基本正規表現 (BRE)
grep -E PATTERN: PATTERN を拡張正規表現 (ERE) として扱います
:::

```bash
# 基本正規表現でも連続で-eを指定すればOR検索します
$ cat watch | grep -e awesome -e rust
rust-lang/rust
vuejs/awesome-vue
rustdesk/rustdesk
rust-lang/rustlings
rust-unofficial/awesome-rust

# -eでは拡張正規表現で検索できません
$ cat watch | grep -e "awesome|rust"

# 大文字の-Eだとある程度は柔軟に正規表現を扱えます
$ cat watch | grep -E "awesome|rust"
rust-lang/rust
vuejs/awesome-vue
rustdesk/rustdesk
rust-lang/rustlings
rust-unofficial/awesome-rust
```

## AND検索
パイプで繋げて地道に絞り込み続けるか、拡張正規表現を駆使して絞り込むか
// TODO: 良い具体例が見つかったら書きます

## 再帰的に文字列で検索

:::note information
grep -r: 再帰的に検索
grep -C 1: 前後1行も一緒に出力
grep -n: 行番号を表示
:::

```bash
$ grep -r ChatGPT -C 1 -n
jdp-3-builder/src/main/java/com/example/Main.java-2-
jdp-3-builder/src/main/java/com/example/Main.java:3:// Generated by ChatGPT and
jdp-3-builder/src/main/java/com/example/Main.java-4-// https://java-design-patterns.com/patterns/builder/
--
jdp-5-decorator/src/main/java/com/example/Main.java-3-// 余りにも汎用的すぎて何が良く表せる例なのかが分からん。
jdp-5-decorator/src/main/java/com/example/Main.java:4:// ChatGPTに聞いて生成した。
jdp-5-decorator/src/main/java/com/example/Main.java-5-
--
jdp-4-adapter/src/main/java/com/example/Main.java-5-// https://qiita.com/AsahinaKei/items/8082ef6cfefd74f9b0e1
jdp-4-adapter/src/main/java/com/example/Main.java:6:// 後はChatGPTに投げるだけ。
jdp-4-adapter/src/main/java/com/example/Main.java-7-
--
jdp-2-facade/src/main/java/com/example/Main.java-2-
jdp-2-facade/src/main/java/com/example/Main.java:3:// Generated by ChatGPT and
jdp-2-facade/src/main/java/com/example/Main.java-4-// https://java-design-patterns.com/patterns/facade/
--
jdp-1-strategy/src/main/java/com/example/Main.java-2-
jdp-1-strategy/src/main/java/com/example/Main.java:3:// Generated by ChatGPT and
jdp-1-strategy/src/main/java/com/example/Main.java-4-// https://qiita.com/urakata_engineer/items/cd5eec1b449231daf321

# 1サーバーに設定ファイルをたくさん記述した時などに有用
$ grep -rn "location" /etc/nginx/conf.d/*.conf -C 10
```

## 再帰的grep(ファイル名のみ表示)

:::note information
grep --exclude-dir: ディレクトリ名称が一致するものを検索対象から除外
grep -l: ファイル名のみ表示
:::

```bash
$ grep -rl "main" ./ --exclude-dir={.git,.temp,node_modules,target}
./rust-devcon-template/src/main.rs
./qiita-content/package-lock.json
./qiita-content/.github/workflows/publish.yml
./qiita-content/public/shell-tricks-2024-0922.md
./qiita-content/public/.remote/8b84d6b1ede90cba1966.md
```

参考: [grepで特定のディレクトリを除外する #Linux - Qiita](https://qiita.com/ionis_h/items/9f9642e435e30a33a88c) @ionis_h さん

# 編集系
## 除外リスト
あるファイルに書き出している内容から、ある除外ファイルに書き出している内容を削除する。
```bash
# 適用するリスト
$ cat watch-repository.txt | head
3b1b/manim
rust-lang/rust
cli/cli
microsoft/vscode
golang/go
kubernetes/kubernetes
django/django
mrdoob/three.js
microsoft/terminal
increments/qiita-cli

# 除外するリスト
$ cat remove-repository.txt | head
django/django
increments/qiita-cli
torvalds/linux
JetBrains/kotlin
apache/kafka
apache/spark
django/django
git/git
github/gitignore
increments/qiita-cli

# 除外実行
$ cat remove-repository.txt \
> | xargs -I@ sh -c 'echo sed -iE \"s%@%%g\"' \
> | xargs -I@ sh -c '@ watch-repository.txt'

# 上記は改行まで削除できなかったので改良の余地あり
$ cat watch-repository.txt | head
3b1b/manim
rust-lang/rust
cli/cli
microsoft/vscode
golang/go
kubernetes/kubernetes

mrdoob/three.js
microsoft/terminal

```

## 空行の削除

// TODO: 分かりづらいので別トピックで執筆

```bash
# sedのインプレース処理で空行を削除
$ sed -iE '/^$/d' watch-repository-test.txt
$ diff watch-repository{,-test}.txt
2d1
< increments/qiita-cli
6d4
< torvalds/linux
13d10
< django/django
```
# ディレクトリやファイルの操作
## タイムスタンプを変更せずにコピーする

:::note information
[Man page of CP](https://linuxjm.sourceforge.io/html/GNU_coreutils/man1/cp.1.html)
cp -p: 元のファイルのオーナー、グループ、パーミション、タイムスタンプを保持
:::

```bash
$ ls -lrt | grep ntp
-rw-r--r-- 1 root     root       2136  2月 17  2022 ntp.conf

$ sudo cp -p ntp.conf ntp-origin.conf

# cpはブレース展開を覚えると楽
$ sudo cp -p ntp{,-origin}.conf

$ ls -lrt | grep ntp
-rw-r--r-- 1 root     root       2136  2月 17  2022 ntp.conf
-rw-r--r-- 1 root     root       2136  2月 17  2022 ntp-origin.conf
```

# 便利なコマンド
## バカデカファイルを分割する
`51GB`ものファイルを作成してしまったときに`split`コマンドを知りました。
`1GB`単位で、ファイルの末尾を上手くランダムに、分割してくれます。
```bash
root@8c724cd91a92:/workspaces/rust-devcon-template# ls -lrth
total 52G
drwxr-xr-x 2 1000 1000 4.0K Aug 22 22:15 src
-rw-r--r-- 1 1000 1000  116 Aug 22 22:18 Cargo.toml
-rw-r--r-- 1 1000 1000 1.1K Aug 22 22:28 Cargo.lock
drwxr-xr-x 3 root root 4.0K Aug 22 22:28 target
-rw-r--r-- 1 root root 1.4G Aug 22 23:46 points_output-1.txt
-rw-r--r-- 1 root root  51G Aug 23 04:15 points_output.txt
root@8c724cd91a92:/workspaces/rust-devcon-template# 
root@8c724cd91a92:/workspaces/rust-devcon-template# which split
/usr/bin/split
root@8c724cd91a92:/workspaces/rust-devcon-template# 
root@8c724cd91a92:/workspaces/rust-devcon-template# split -b 1G points_output.txt points_output_part_
root@8c724cd91a92:/workspaces/rust-devcon-template# 
root@8c724cd91a92:/workspaces/rust-devcon-template# ls -l | wc -l
58
root@8c724cd91a92:/workspaces/rust-devcon-template# 
root@8c724cd91a92:/workspaces/rust-devcon-template# ls -lrth | tail
-rw-r--r-- 1 root root 1.0G Aug 23 10:37 points_output_part_bp
-rw-r--r-- 1 root root 1.0G Aug 23 10:37 points_output_part_bq
-rw-r--r-- 1 root root 1.0G Aug 23 10:37 points_output_part_br
-rw-r--r-- 1 root root 1.0G Aug 23 10:38 points_output_part_bs
-rw-r--r-- 1 root root 1.0G Aug 23 10:38 points_output_part_bt
-rw-r--r-- 1 root root 1.0G Aug 23 10:38 points_output_part_bu
-rw-r--r-- 1 root root 1.0G Aug 23 10:38 points_output_part_bv
-rw-r--r-- 1 root root 1.0G Aug 23 10:38 points_output_part_bw
-rw-r--r-- 1 root root 1.0G Aug 23 10:38 points_output_part_bx
-rw-r--r-- 1 root root 212M Aug 23 10:38 points_output_part_by
root@8c724cd91a92:/workspaces/rust-devcon-template# 
```
# トラブルシューティング的
## xargsで上手く差し込む
// TODO: ghがややこしいので簡易化する、別で書き出す
```bash
# 引数が1個の場合は-I@で差し込めば上手く機能するようです
$ gh search repos --stars=">=50000" --sort stars --limit 2 --json fullName --jq '.[].fullName' \
> | xargs -I@ gh api /search/issues?q="repo:@+is:issue+is:open" --jq .total_count
239
28
# 上記をわざわざ-n1指定するとワーニングされる
$ gh search repos --stars=">=50000" --sort stars --limit 2 --json fullName --jq '.[].fullName' \
> | xargs -n1 -I@ gh api /search/issues?q="repo:@+is:issue+is:open" --jq .total_count
xargs: warning: options --max-args and --replace/-I/-i are mutually exclusive, ignoring previous --max-args value
239
28
# 引数が2個以上ある場合はTheBookでやっていたようにsh -cに渡すのが手っ取り早い
$ cat data.txt | xargs -n2 sh -c 'echo "The inputs are [$1, $2]."' sh
The inputs are [aaaa, bbbb].
The inputs are [cccc, dddd].
```
# シェル芸作品
## GitHubリポジトリの選定
`GitHub CLI`の`gh`コマンドを使って、
コントリビュートしたいOSSを探している時にシェル芸をしました。
黒魔術のように見えますが、やっていることは
確認したいリポジトリに`first timers only`のラベルや
`good first issue`のラベルが付いているissueの数を計上しています。
```bash
$ cat watch-repository.txt | sort | uniq \
> | xargs -n1 sh -c 'echo $0 && gh issue list --repo $0 --label "first timers only" --limit 1000 --json id --jq ".[].id" | wc -l && gh issue list --repo $0 --label "good first issue" --limit 1000 --json id --jq ".[].id" | wc -l' \
> | grep -v "repository has disabled issues" \
> | xargs -n3 sh -c 'echo $2" "$3" "$1' sh | sort -k1,1nr -k2,2nr
```
# いまさらシェル芸とは
## 教祖
上田様: [上田ブログ](https://b.ueda.tech/?page=01434)
> なお、このサイトはbashでできている。

上記からも伺える狂気的なカリスマ
## 教典
160本ノック、勝手にTheBookと呼んでいます
[1日1問、半年以内に習得　シェル・ワンライナー160本ノック](https://www.amazon.co.jp/1%E6%97%A51%E5%95%8F%E3%80%81%E5%8D%8A%E5%B9%B4%E4%BB%A5%E5%86%85%E3%81%AB%E7%BF%92%E5%BE%97-%E3%82%B7%E3%82%A7%E3%83%AB%E3%83%BB%E3%83%AF%E3%83%B3%E3%83%A9%E3%82%A4%E3%83%8A%E3%83%BC160%E6%9C%AC%E3%83%8E%E3%83%83%E3%82%AF-Software-Design-plus%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA/dp/4297122677)
プログラマの、というか日本国民の義務
私はこの書籍によってエンジニア生命を保つことが出来ました

## 英語圏
英語圏では`shell tricks`と呼ばれているらしい？: [マルチステージ ビルドを使う](https://docs.docker.jp/develop/develop-images/multistage-build.html)
