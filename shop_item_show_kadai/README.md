# 商品詳細画面

- [商品詳細画面](#商品詳細画面)
  - [事前準備](#事前準備)
  - [本章の狙い](#本章の狙い)
  - [①Laravel環境の構築](#laravel環境の構築)
  - [⑦動作確認(2回目)](#動作確認2回目)
  - [課題の提出について](#課題の提出について)
    - [課題の合格基準](#課題の合格基準)
    - [合格確認方法](#合格確認方法)

## 事前準備

[こちらのページ]()から、ソースコードを`C:¥sys_dev_exe`へcloneしてください。

## 本章の狙い

- [Laravelの便利な実装(ルートモデルバインディング)](../shop_item_show/README.md)の章で学んだ知識を定着させる
- 商品詳細画面を再構築する

## ①Laravel環境の構築

1. VSCode上で、`Ctrl+Shift+P`(Macの場合は`Cmd+Shift+P`)を押し、コンテナを起動する
2. VSCode上で、`Ctrl+J`(Macの場合は`Cmd+J`)を押し、ターミナルを表示する
3. ターミナルに`composer create-project laravel/laravel .` と入力し、`Enter`で実行する<br>
   ![](./images/composer_command_1.png)
4. [【課題】ジャンル別商品一覧画面の作成](../shop_item_index_kadai/README.md)までで作成した以下のコードを、上記「1.」でcloneしたソースコードと同じ場所に上書きする
   
   ```text
    app
    ├── Http
    │   └── Controllers
    │       └── ItemController.php
    ├── Models
    │   └── Item.php
    │
    途中省略
    │
    database
    ├── migrations
    │   └── 20XX_XX_XX_XXXXXX_create_items_table.php
    ├── seeds
    │   ├── DatabaseSeeder.php
    │   └── ItemsTableSeeder.php
    public
    ├── css
    │   └── minishop.css
    ├── images
    │   └── xxx.png(15個の画像ファイル)
    │
    resources
    ├── views
    │   ├── items
    │   │   └── index.blade.php    
    │   └── index.blade.php
    routes
    ├── web.php
    │
    途中省略
    │
    .env
    ```

5. コマンドでマイグレーションとシーダーを実行する(※コマンドを忘れた人は、[モデル、コントローラ](../shop_item_index/README.md)の章を参照すること)
6. phpmyadminで`items`テーブルが確認できればOK<br>
   ![](./images/phpmyadmin_1.png)<br>
   ![](./images/phpmyadmin_2.png)<br>
   ![](./images/phpmyadmin_3.png)

## ②ルーティングとジャンル別商品一覧画面の修正

商品詳細画面を作成する前に、前章で作成したものにいくつか修正を加える必要があります。

### ②−1 ルーティングの修正

---

現状、ジャンル別商品一覧画面の「詳細」リンクをクリックしても商品詳細画面に遷移しません。
そのため、商品詳細画面に遷移するためのルーティングを追加する必要があります。

ルーティングを追加するためには、`routes/web.php`ファイルを以下の観点から修正してください。

- GETリクエスト時にItemControllerの`show`メソッドを呼び出すルートを追加する
- マッピングするURLを`item/show/{xxxx}`に設定する(`xxxx`にはルートモデルバインディングのための名前を指定する※わからなければ[Laravelの便利な実装(ルートモデルバインディング)](../shop_item_show/README.md)の章を参考にすること)
- ルーティングの名前を`item.show`に設定する

### ②−2 ジャンル別商品一覧画面の修正

---

次に、ジャンル別商品一覧画面(`resources/views/index.blade.php`)を修正します。
上記の[ルーティングの修正](#ルーティングの修正)を参考に、商品詳細画面に遷移するための詳細リンクを以下の観点から修正してください。

- 詳細リンクの宛先に対象のルーティングを指定する(ルーティングの名前で指定すること)
- 商品IDを渡す※商品IDを渡すときの名前に注意！わからなければ[Laravelの便利な実装(ルートモデルバインディング)](../shop_item_show/README.md)の章を参考にすること

## ③コントローラの修正

次に、商品詳細画面のビューを表示するために、`ItemController`(`app/Http/Controllers/ItemController.php`)を修正します。
以下の観点から修正してください。

- `show`メソッドの追加(`show`メソッドの仕様は以下のとおり)
  - ルートモデルバインディングにより、商品IDと一致する商品情報を取得
  - 商品情報をビューに渡す

## ④ビューの作成

次に、ビューとして商品詳細画面を作成します。
`resources/views/item`ディレクトリに`show.blade.php`ファイルを作成し、以下の観点からコードを記述してください。

- コードは前期の[ミニショップ②(商品詳細画面)](https://classroom.google.com/c/NjYwMjEyMzgyMzQ2/a/Njk3NzM1MzQwMjM2/details)を参考にし、[Laravelの便利な実装(ルートモデルバインディング)](../shop_item_show/README.md)までの章で学んだ内容を反映させること
{% raw %}
  - cssを適用する
  - 商品データの埋め込みには、`{{ }}`を使用する
  - ディレクティブを使用する
  - 注文数のプルダウンは、1から10までの数値を表示するために、以下の例にならって`@for`ディレクティブを使用すること

    ```php
    @for ( $i=1;  $i<=10;  $i++ )
        <option value="{{ $i }}"> {{ $i }} </option>
    @endfor
    ```

  - ジャンル別商品一覧に戻るリンクを作成する
{% endraw %}

## ⑤動作確認(1回目)

これで、商品詳細画面の作成は完了です。
では、実際に動作確認を行いましょう。

以下のように、商品詳細画面に遷移するリンクをクリックすると、商品詳細画面が表示されればOKです。

![](./images/genre.png)
![](./images/item_list.png)
![](./images/item_show.png)

## ⑥商品詳細画面のバグ修正

現状のままだと、商品詳細画面からジャンル別商品一覧に戻るリンクをクリックすると、以下のようなエラーが発生します。

![](./images/item_show_back.png)
![](./images/error.png)

このエラーは、商品詳細画面からジャンル別商品一覧に戻るリンクをクリックした際のルーティングが設定されていないために発生します。
そのため、商品詳細画面からジャンル別商品一覧に戻るリンクをクリックした際に、ジャンル別商品一覧画面に遷移するためのルーティングを追加する必要があります。

`routes/web.php`ファイルを以下のように修正してください。

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ItemController;

// Route::get('/', function () {
//     return view('welcome');
// });

Route::get('/', function () {
    return view('index');
});
// 以前までのはコメントアウト
//Route::post('item', [ItemController::class, 'index'])->name('item.index');
// --- 以下を追加 ---
Route::match(['get', 'post'], 'item/{genre?}', [ItemController::class, 'index'])->name('item.index');

Route::get('item/show/{item}', [ItemController::class, 'show'])->name('item.show');
```

**【解説】**

`Route::match(['get', 'post'], 'item/{genre?}', [ItemController::class, 'index'])->name('item.index');`: <br>
`Route::match`は、GETリクエストとPOSTリクエストの両方を受け付けるルートを定義しています。
第1引数の`['get', 'post']`は、GETリクエストとPOSTリクエストを受け付けることを意味します。
第2引数の`'item/{genre?}'`は、商品詳細画面からジャンル別商品一覧に戻るリンクをクリックした際に、ジャンル別商品一覧画面に遷移するためのURLです。
`{genre?}`の`?`は、POSTリクエスト時にURL末尾にパラメータでジャンルが指定されていない場合でも、問題なく遷移できるようにするためのものです。

以上で、商品詳細画面のバグ修正は完了です。

## ⑦動作確認(2回目)

以下のように、商品詳細画面からジャンル別商品一覧に戻るリンクをクリックしても、エラーが発生しないことが確認できればOKです。

![](./images/genre.png)
![](./images/item_list.png)
![](./images/item_show_back.png)
![](./images/item_list.png)

## 課題の提出について

提出した課題はGitHub上で自動採点されます。
従来通りGitHub上にpushすれば完了で、自動採点がはじまります。
ただし、**pushする前に以下の作業を事前に行わないと自動採点ができない**ので、以下の対応を忘れずに行ってください。

### 課題の合格基準

---

以下を合格基準とします。

1. ジャンル別商品一覧画面で詳細リンクをクリックすると、商品詳細画面が表示されること

### 合格確認方法

---

1. 本課題の[課題ページ]()に再度アクセスする
2. 画面上部にある`Actions`をクリックする<br>
![](./images/acions.png)
1. **一番上**の行に、緑色のチェックが入っていればOK<br>
![](./images/pass.png)
