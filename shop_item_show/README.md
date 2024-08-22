# 商品詳細画面

- [商品詳細画面](#商品詳細画面)
  - [事前準備](#事前準備)
  - [はじめに](#はじめに)
  - [ルーティングとジャンル別商品一覧画面の修正](#ルーティングとジャンル別商品一覧画面の修正)
    - [ルーティングの修正](#ルーティングの修正)
    - [ジャンル別商品一覧画面の修正](#ジャンル別商品一覧画面の修正)
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
Route::get('item/{id}', [ItemController::class, 'show'])->name('item.show');
```

**【解説】**

`Route::get('item/{id}', [ItemController::class, 'show'])->name('item.show');`: <br>
`item/{id}`は、商品詳細画面に遷移するためのURLです。
`{id}`は、商品IDを表しています。

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
        <td><a href="">詳細</a></td>
        </tr>
    @endforeach
    </table>
    <br>
    <a href="{{ route('index') }}">ジャンル選択に戻る</a>
</body>
</html>
```


## まとめ

本章では、Laravelのモデルとコントローラについて学びました。
前章と合わせて、Laravelを通じて、MVCモデルの基本である、モデル、ビュー、コントローラについて経験しましたがいかがだったでしょうか。

大切なことは、Laravelの基本的なルールに従いコードを作成することにより、意識せずともMVCモデルを実装できる、つまり、オブジェクト思考に則った効率性・保守性の高いコードに標準化されるということです。

これがフレームワークが大規模開発に向いていると言われる理由の一つです。
