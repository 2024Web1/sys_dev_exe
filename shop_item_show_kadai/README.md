# 商品詳細画面

- [商品詳細画面](#商品詳細画面)
  - [事前準備](#事前準備)
  - [はじめに](#はじめに)
  - [ルーティングとジャンル別商品一覧画面の修正](#ルーティングとジャンル別商品一覧画面の修正)
    - [ルーティングの修正](#ルーティングの修正)
    - [ジャンル別商品一覧画面の修正](#ジャンル別商品一覧画面の修正)
  - [コントローラの修正](#コントローラの修正)
  - [商品詳細画面の作成](#商品詳細画面の作成)
  - [商品詳細画面のバグ修正](#商品詳細画面のバグ修正)
  - [まとめ](#まとめ)

## 事前準備

前回の[ジャンル別商品一覧画面](../shop_item_index/README.md)でcloneしたコードをそのまま利用してください。

## はじめに

本章では、Laravelを使って、商品詳細画面を作成します。

## ルーティングとジャンル別商品一覧画面の修正

商品詳細画面を作成する前に、前章で作成したものにいくつか修正を加える必要があります。

### ルーティングの修正

現状、ジャンル別商品一覧画面の「詳細」リンクをクリックしても商品詳細画面に遷移しません。
そのため、商品詳細画面に遷移するためのルーティングを追加する必要があります。

ルーティングを追加するためには、`routes/web.php`ファイルを以下のように修正してください。

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ItemController;

// Route::get('/', function () {
//     return view('welcome');
// });

Route::get('/', function () {
    return view('index');
});
Route::post('item', [ItemController::class, 'index'])->name('item.index');
// --- 以下を追加 ---  
Route::get('item/show/{item}', [ItemController::class, 'show'])->name('item.show');
```

**【解説】**

`Route::get('item/{id}', [ItemController::class, 'show'])->name('item.show');`: <br>
`item/show/{item}`は、商品詳細画面に遷移するためのURLです。
`{item}`は、商品IDを表しています。
※なぜ商品IDなのに、`{item}`という名前なのかは、後述します。

`[ItemController::class, 'show']`は、`ItemController`クラスの`show`メソッドを呼び出すことを意味します。
`name('item.show')`は、このルートに名前を付けています。
この名前は、ビューでリンクを生成する際に使用します。

### ジャンル別商品一覧画面の修正

次に、ジャンル別商品一覧画面(`resources/views/index.blade.php`)を修正します。
上記のルーティングを参考に、商品詳細画面に遷移するためのリンクを追加します。

```php
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="stylesheet" href="{{ asset('css/minishop.css')}}">
<title>ショッピングサイト</title>
</head>
<body>
<h3>ジャンル別商品一覧</h3>
    <table>
        <tr>
            <th>&nbsp;</th>
            <th>商品名</th>
            <th>メーカー・著者<br>アーティスト</th>
            <th>価格</th>
            <th>詳細</th>
        </tr>
    @foreach( $items  as  $item )
        <tr>
        <td class="td_mini_img"><img class="mini_img" src="{{ asset('images/'.$item->image )}}"></td>
        <td class="td_item_name"> {{  $item->name }} </td>
        <td class="td_item_maker"> {{  $item->maker }}  </td>
        <td class="td_right">&yen; {{  number_format( $item->price) }} </td>
        <!-- 以下のhref属性を修正 -->
        <td><a href="{{ route('item.show',  ['item' => $item->ident]) }}">詳細</a></td>
        </tr>
    @endforeach
    </table>
    <br>
    <a href="{{ route('index') }}">ジャンル選択に戻る</a>
</body>
</html>
```

**【解説】**

`<a href="{{ route('item.show',  ['item' => $item->ident]) }}">詳細</a>`: <br>
`route('item.show',  ['item' => $item->ident])`は、商品詳細画面に遷移するためのリンクを生成しています。
`item.show`は、`web.php`で定義したルート名です。
`['item' => $item->ident]`は、商品IDを指定しています。
ここで指定した商品IDは、ルーティングの`{item}`に渡され、ItemControllerの`show`メソッドで受け取ることができます。
※なぜ商品IDなのに、`{item}`という名前なのかは、後述します。

## コントローラの修正

次に、商品詳細画面のビューを表示するために、コントローラに`show`メソッドを作成します。
ItemController(`app/Http/Controllers/ItemController.php`)を以下のように修正してください。

```php
<?php

namespace App\Http\Controllers;

use App\Models\Item;
use Illuminate\Http\Request;

class ItemController extends Controller
{
    public function index(Request $request)
    {
        $items = Item::where('genre', $request->genre)->get();
        return view('item.index', ['items' => $items]);
    }

    // --- ここから追加 ---
    public function show(Item $item)
    {
        return view('item.show', ['item' => $item]);
    }
    // --- ここまで追加 ---
}
```

**【解説】**

`public function show(Item $item)`: <br>
`show`メソッドは、商品詳細画面を表示するためのメソッドです。
注目すべきは、いきなりreturn文がありますが、これは、商品IDに対応する商品情報を取得してビューに渡していることを意味します。
これを可能にしているのが、`Item $item`です。
この`$item`には、商品IDに対応する商品情報が自動的に格納されています
このLaravelが自動的に商品IDに対する商品情報を取得する機能を、**ルートモデルバインディング(Route Model Binding)**と言います。

このルートモデルバインディングを使用するためには、ルーティングの定義とコントローラのメソッドの引数名が一致している必要があります。
以下のような記述を思い出してみてください。

**`routes/web.php`**

```php
// {item}の部分がコントローラのメソッドの引数名と一致している
Route::get('item/show/{item}', [ItemController::class, 'show'])->name('item.show');
```

**`resources/views/index.blade.php`**

```php
// ['item' => $item->ident]の`item`の部分がコントローラのメソッドの引数名と一致している
<a href="{{ route('item.show',  ['item' => $item->ident]) }}">詳細</a>
```

Laravelでは、コントローラに記述する`show` メソッドは、一般的に、指定されたリソースを表示するためのメソッドとして使われます。
例えば、今回のように商品IDに対応する商品情報を表示する場合に使われます。

## 商品詳細画面の作成

次に、商品詳細画面を作成します。
`resources/views/item`ディレクトリに`show.blade.php`ファイルを作成し、以下のように記述してください。

```php
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="stylesheet" href="{{ asset('css/minishop.css')}}">
<title>ショッピングサイト</title>
</head>
<body>
<h3>商品詳細</h3>
<!-- action属性は空にしています -->
<form method="POST" action="">
    @csrf
    <input type="hidden" name="ident" value="{{ $item->ident }}">
    <table>
        <tr><th>商品名</th>
        <td>{{ $item->name }}</td></tr>
        <tr><td colspan="2"><div class="td_center">
        <img class="detail_img" src="{{ asset('images/'.$item->image )}}"></div></td></tr>
        <tr><th>メーカー・著者<br>アーティスト</th>
        <td>{{ $item->maker }}</td></tr>
        <tr><th>価 格</th>
        <td>&yen;{{  number_format( $item->price) }}</td></tr>
        <tr><th>注文数</th>
        <td><select name="quantity">
            @for ( $i=1;  $i<=10;  $i++ )
                <option value="{{ $i }}"> {{ $i }} </option>
            @endfor
        </select></td></tr>
        <tr><th colspan="2"><input type="submit" value="カートに入れる"></th></tr>
    </table>
</form>
<br>
<a href="{{ route('item.index',['genre' => $item->genre])}}">ジャンル別商品一覧に戻る</a>
</body>
</html>
```

**【解説】**

`<input type="hidden" name="ident" value="{{ $item->ident }}">`: <br>
`<input type="hidden" name="ident" value="{{ $item->ident }}">`は、商品IDをPOSTするための隠しフィールドです。

`<a href="{{ route('item.index',['genre' => $item->genre])}}">ジャンル別商品一覧に戻る</a>`: <br>
`route('item.index',['genre' => $item->genre])`は、ジャンル別商品一覧画面に遷移するためのリンクを生成しています。

以上で、商品詳細画面の作成は完了です。
実際に、動作確認をしてみましょう。
以下のように、ジャンル選択画面から商品詳細画面まで遷移できればOKです。

![](./images/genre.png)
![](./images/item_list.png)
![](./images/item_show.png)

## 商品詳細画面のバグ修正

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
実際に、動作確認をしてみましょう。

![](./images/genre.png)
![](./images/item_list.png)
![](./images/item_show_back.png)
![](./images/item_list.png)

## まとめ

本章では、商品詳細画面を作成しました。

商品詳細画面を作成する際に、データを取得するコードを記述することなく、自動でデータを取得するルートモデルバインディングを学びました。
また、GETリクエストとPOSTリクエストの両方を受け付けるルーティングを学びました。
