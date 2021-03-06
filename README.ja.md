# Bookmeter Scraper

[![Gem Version](https://badge.fury.io/rb/bookmeter_scraper.svg)](https://badge.fury.io/rb/bookmeter_scraper)
[![Build Status](https://travis-ci.org/kymmt90/bookmeter_scraper.svg?branch=master)](https://travis-ci.org/kymmt90/bookmeter_scraper)
[![Code Climate](https://codeclimate.com/github/kymmt90/bookmeter_scraper/badges/gpa.svg)](https://codeclimate.com/github/kymmt90/bookmeter_scraper)
[![Test Coverage](https://codeclimate.com/github/kymmt90/bookmeter_scraper/badges/coverage.svg)](https://codeclimate.com/github/kymmt90/bookmeter_scraper/coverage)

[読書メーター](http://bookmeter.com)の情報をスクレイピングして Ruby で扱えるようにするための gem です。

- 書籍情報
  - 読んだ本
  - 読んでる本
  - 積読本
  - 読みたい本
- お気に入り / お気に入られユーザ
- ユーザプロフィール

を取得可能です。

## 注意

スクレイピングの頻度は常識の範囲内にとどめてください。読書メーターのサーバーへ故意に著しい負荷をかける行為は、利用規約の第 9 条で禁止されています。

- [利用規約 - 読書メーター](http://bookmeter.com/terms.php)

## 使いかた

この gem を使うときは以下のコードが必要です。

```ruby
require 'bookmeter_scraper'
```

### ログイン

書籍情報、お気に入り / お気に入られユーザ情報を取得するには、`Bookmeter.log_in` または `Bookmeter#log_in` でログインしておく必要があります。

ログイン情報の入力には以下の 3 通りの方法があります。

1. 引数として渡す
2. `config.yml` へ記述しておく
3. ブロック内で設定する

#### 1. 引数として渡す

以下のように `Bookmeter.log_in` へメールアドレスとパスワードを引数として渡すことで、ログインできます。

```ruby
bookmeter = BookmeterScraper::Bookmeter.log_in('example@example.com', 'password')
bookmeter.logged_in?    # true
```

`Bookmeter#log_in` でもログイン可能です。

```ruby
bookmeter = BookmeterScraper::Bookmeter.new
bookmeter.log_in('example@example.com', 'password')
```

#### 2. `config.yml` へ記述しておく

まず、以下のように YAML ファイル `config.yml` を記述し、Ruby スクリプトを実行する場所と同じディレクトリに置きます。

```yml
mail: example@example.com
password: your_password
```

次に、引数なしで `Bookmeter.log_in` または `Bookmeter#log_in` を呼ぶと、`config.yml` からログイン情報を読みとり、ログインできます。

```ruby
bookmeter = BookmeterScraper::Bookmeter.log_in
bookmeter.logged_in?    # true
```

#### 3. ブロック内で設定する

以下のように `Bookmeter.log_in` へブロックを渡すことで、ログインできます。

```ruby
bookmeter = BookmeterScraper::Bookmeter.log_in do |configuration|
  configuration.mail     = 'example@example.com'
  configuration.password = 'password'
end
bookmeter.logged_in?    # true
```

`Bookmeter#log_in` でもログイン可能です。

```ruby
bookmeter = BookmeterScraper::Bookmeter.new
bookmeter.log_in do |configuration|
  configuration.mail     = 'example@example.com'
  configuration.password = 'password'
end
```

### 書籍情報の取得

以下の書籍情報

- 読んだ本
- 読んでる本
- 積読本
- 読みたい本

を取得できます。取得には `Bookmeter.log_in` などによる事前のログインが必要です。

#### 読んだ本

`Bookmeter#read_books` で「読んだ本」情報が取得できます。

```ruby
books = bookmeter.read_books        # ログインユーザの「読んだ本」を取得
bookmeter.read_books('01010101')    # 他のユーザの ID を指定して、そのユーザの「読んだ本」を取得
```

書籍情報は

- 書名 `name`
- 著者 `author`
- 読了日（初読了日と再読日の両方）の配列 `read_dates`
- 読書メーター内の書籍ページの URI `uri`
- 書籍の表紙画像 URI `image_uri`

を属性として持つ `Book` の配列として取得できます。

```ruby
books[0].name
books[0].author
books[0].read_dates
books[0].uri
books[0].image_uri
```

さらに、`Bookmeter#read_books_in` で特定年月の「読んだ本」情報が取得できます。

```ruby
books = bookmeter.read_books_in(2016, 1)                # ログインユーザが 2016 年 1 月に「読んだ本」を取得
books = bookmeter.read_books_in(2016, 1, '01010101')    # ID で指定した他のユーザが 2016 年 1 月に「読んだ本」を取得
```

#### 読んでる本 / 積読本 / 読みたい本

「読んだ本」以外の書籍情報

- 読んでる本
- 積読本
- 読みたい本

も、それぞれ

- `Bookmeter#reading_books`
- `Bookmeter#tsundoku`
- `Bookmeter#wish_list`

で取得できます。

```ruby
books = bookmeter.reading_books    # ログインユーザの「読んでる本」を取得
books[0].name
books[0].author
books[0].read_dates    # 読了日の Array は空
books[0].uri
books[0].image_uri

bookmeter.tsundoku     # ログインユーザの「積読本」を取得
bookmeter.wish_list    # ログインユーザの「読みたい本」を取得
```

### お気に入り / お気に入られユーザ情報の取得

`Bookmeter#followings` と `Bookmeter#followers` でログインユーザが参照できるお気に入り / お気に入られユーザの情報を取得できます。取得には事前のログインが必要です。

```ruby
following_users = bookmeter.followings    # 「お気に入り」ユーザの情報を取得
followers = bookmeter.followers           # 「お気に入られ」ユーザの情報を取得
```

ユーザ情報は

- ユーザ名 `name`
- ユーザ ID `id`
- 読書メーター内のユーザページの URI `uri`

を持つ `User` の配列として取得できます。

```ruby
following_users[0].name
following_users[0].id
followers[0].name
followers[0].id
followers[0].uri
```

#### 注意

**お気に入り / お気に入られのページにページネーションが存在する場合には未対応です。**

### ユーザのプロフィールの取得

`Bookmeter#profile` でユーザのプロフィールを取得できます。プロフィールはログインなしで閲覧できるため、ログインは不要です。

```ruby
bookmeter = BookmeterScraper::Bookmeter.new
user_id = '000000'
profile = bookmeter.profile(user_id)    # 任意ユーザの ID を指定してプロフィールを取得可能
```

プロフィール情報は以下の属性を持つ `Struct` として取得できます。プロフィールで設定されていない属性は `nil` となります。

```ruby
profile.name                # ユーザ名
profile.gender              # 性別
profile.age                 # 年齢
profile.blood_type          # 血液型
profile.job                 # 職業
profile.address             # 現住所
profile.url                 # URL / ブログ
profile.description         # 自己紹介
profile.first_day           # 記録初日
profile.elapsed_days        # 経過日数
profile.read_books_count    # 読んだ本の数
profile.read_pages_count    # 読んだページの数
profile.reviews_count       # 感想/レビューの数
profile.bookshelfs_count    # 本棚の数
```

## ライセンス

[MIT License](http://opensource.org/licenses/MIT)
