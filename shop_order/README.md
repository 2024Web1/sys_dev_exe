# 注文機能

## 事前準備

1. [こちらのページ]()から、ソースコードを`C:¥sys_dev_exe`へcloneする
2. VSCode上で、`Ctrl+Shift+P`(Macの場合は`Cmd+Shift+P`)を押し、コンテナを起動する
3. VSCode上で、`Ctrl+J`(Macの場合は`Cmd+J`)を押し、ターミナルを表示する
4. `composer create-project laravel/laravel .` を実行し、Laravel環境を構築する
5. 過去に作成した以下のコードを、上記「1.」でcloneしたソースコードと同じ場所に上書きする
   
    ```text
    app
    ├── Http
    │   └── Controllers
    │       ├── CartController.php
    │       └── ItemController.php
    ├── Models
    │   ├── Cart.php
    │   └── Item.php
    database
    ├── migrations
    │   ├── 20xx_xx_xx_xxxxxx_create_cart_table.php
    │   └── 20xx_xx_xx_xxxxxx_create_items_table.php
    ├── seeders
    │   └── ItemTableSeeder.php
    │
    途中省略
    │
    resources
    ├── views
    │   ├── cart
    │   │   └── index.blade.php
    │   ├── item
    │   │   ├── index.blade.php
    │   │   └── show.blade.php
    │   └── index.blade.php
    routes
    ├── web.php
    .env
    ```

## はじめに

本章では、Laravelを使い、注文機能を実装します。

## データベース環境構築

新しくソースコードをcloneしたので、再度データベース環境構築をする必要があります。
今回は、itemsテーブルに加え、カート内の商品を管理するためのcartテーブルを作成します。
なお、.envファイルは既に編集済みのものを上書きしているので、再度編集する必要はありません。

## マイグレーション

今回は、ordersテーブルを作成するためのマイグレーションファイルを追加し、コマンドを実行してテーブルを作成します。
なお、ordersテーブルの構造は前期同様以下の通りです。

新たにテーブルorders, orderdetailsを作成します。

**テーブル名：orders**

| カラム名 | データ型 | 制約 | 備考 |
| - | - | - | - |
|orderId|int型|主キー、auto_increment|注文番号|
|orderdate|datetime型||注文日時|

**テーブル名：orderdetails**

| カラム名 | データ型 | 制約 | 備考 |
| - | - | - | - |
|orderId|int型|主キー|注文番号(テーブルordersのorderId)|
|itemId|int型|主キー|商品番号|
|quantity|int型||注文数|

※primary keyは「orderId」と「itemId」の複数列を使った複合主キーとします。

1. VSCode上で、`Ctrl+Shift+P`(Macの場合は`Cmd+Shift+P`)を押し、コンテナを起動する(既に起動しているなら不要)
2. VSCode上で、`Ctrl+J`(Macの場合は`Cmd+J`)を押し、ターミナルを表示する
3. 以下のコマンドを実行して、orders,orderdetailsテーブル用のマイグレーションファイルを作成する

```bash
php artisan make:migration create_orders_orderdetails_table
```

4. 以下のファイルが作成されていることを確認する
   - `database/migrations/20xx_xx_xx_xxxxxx_create_orders_orderdetails_table.php`

5. 作成したマイグレーションファイルのupメソッドを編集する

    **database/migrations/20xx_xx_xx_xxxxxx_create_orders_table.php**

    ```php
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->integer('orderId')->autoIncrement()->primary();
            $table->dateTime('orderdate');
        });

        Schema::create('orderdetails', function (Blueprint $table) {
            $table->integer('orderId');
            $table->integer('itemId');
            $table->integer('quantity');
            $table->primary(['orderId', 'itemId']); // 複合主キーの設定
            $table->foreign('orderId')->references('orderId')->on('orders')->onDelete('cascade');
            $table->foreign('itemId')->references('ident')->on('items')->onDelete('cascade');
        });
    }
    ```

6. 以下のコマンドを実行して、orders,orderdetailsテーブルを作成する

    ```bash
    php artisan migrate
    ```

7. 以下のコマンドを実行して、seedを実行する

    ```bash
    php artisan db:seed --class=ItemTableSeeder
    ```

## コントローラに注文データを保存する処理を追加

1. コントローラを作成する

    ```bash
    php artisan make:controller OrderController
    ```

## ルーティングとカート内の商品画面の修正

### ルーティング

1. `routes/web.php`を以下のように修正する

    ```php
    <?php

    use Illuminate\Support\Facades\Route;
    use App\Http\Controllers\ItemController;
    use App\Http\Controllers\CartController;
    use App\Http\Controllers\OrderController; // 以下を追加

    // Route::get('/', function () {
    //     return view('welcome');
    // });

    Route::get('/', function () {
        return view('index');
    })->name('index');

    //Route::post('item', [ItemController::class, 'index'])->name('item.index');
    Route::match(['get', 'post'], 'item/{genre?}', [ItemController::class, 'index'])->name('item.index');
    Route::get('item/show/{item}', [ItemController::class, 'show'])->name('item.show');

    Route::get('cart', [CartController::class, 'index'])->name('cart.index');
    Route::post('cart', [CartController::class, 'store'])->name('cart.store');
    Route::delete('cart/{cart}',[CartController::class, 'destroy'])->name('cart.destroy');
    Route::patch('cart/{cart}',[CartController::class, 'update'])->name('cart.update');

    // 以下を追加
    Route::get('order', [OrderController::class, 'store'])->name('order.store');
    ```

    **【解説】**

    `use App\Http\Controllers\OrderController;`: <br>
    OrderControllerを使用するためのuse文を追加しています。

    `Route::get('order', [OrderController::class, 'store'])->name('order.store');`: <br>
    注文機能を実装するためのルーティングを追加しています。

### カート内の商品画面の修正

1. `resources/views/cart/index.blade.php`を以下のように修正する

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
        @if( count($carts) == 0 )
            <h3>カート内に商品はありません</h3>
            <a href="{{ route('index') }}">ジャンル選択に戻る</a>
        @else
            <h3>カート内の商品</h3>
            <table>
            <tr>
                <th>&nbsp;</th>
                <th>商品名</th>
                <th>メーカー・著者<br>アーティスト</th>
                <th>価格</th>
                <th>注文数</th>
                <th>金額</th>
                <th>削除</th>
            </tr>
            @php
                $total = 0;
            @endphp
            @foreach( $carts  as  $cart )
                <tr>
                    <td class="td_mini_img"><img class="mini_img" src="{{ asset('images/'.$cart->item->image )}}"></td>
                    <td class="td_item_name"> {{ $cart->item->name }} </td>
                    <td class="td_item_maker"> {{ $cart->item->maker }} </td>
                    <td class="td_right">&yen; {{  number_format( $cart->item->price) }} </td>
                    <!-- 既存の注文数はコメントアウト -->
                    <!-- <td class="td_right"> {{ $cart->quantity }} </td> -->
                    <!-- プルダウンと更新ボタンを追加したものに変更 -->
                    <!-- 以下を追加 -->
                    <td>
                        <form method="POST" action="{{ route('cart.update', ['cart' => $cart->ident]) }}">
                            @csrf
                            @method('PATCH')
                            <select name="quantity">
                                @for ( $i=1;  $i<=10;  $i++ )
                                    <option value="{{ $i }}"
                                    @if($i == $cart->quantity)
                                        selected
                                    @endif
                                    > {{ $i }} </option>
                                @endfor
                                &nbsp;
                                <input type="submit" value="変更">
                        </form>
                    </td>
                    <!-- ここまで -->
                    <td class="td_right">&yen; {{ number_format( $cart->item->price * $cart->quantity) }}</td>
                    <td>
                        <form method="POST" action="{{ route('cart.destroy', ['cart' => $cart->ident]) }}">
                            @csrf
                            @method('DELETE')
                            <input type="submit" value="削除">
                        </form>
                    </td>
                </tr>
                @php
                    $total += $cart->item->price * $cart->quantity;
                @endphp
            @endforeach
            <tr>
                <th colspan="5">合計金額</th><td class="td_right">&yen; {{ number_format($total) }}</td>
                <td>&nbsp;</td>
            </tr>
            </table>
            <br>
            <!-- 以下を修正 -->
            <a href="{{ route('index') }}">ジャンル選択に戻る</a>&nbsp;&nbsp;<a href="{{ route('order.store') }}">注文する</a>
        @endif
    </body>
    </html>
    ```

    **【解説】**

    `<a href="{{ route('order.store') }}">注文する</a>`: <br>
    注文するリンクを追加しています。


