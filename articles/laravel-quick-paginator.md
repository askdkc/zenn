---
title: "Laravelのページネーションを瞬時に高速化"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [laravel, php]
published: true
---
# Laravelのページネーションを一瞬で高速化

Laravelで一覧画面を作るとき、ほぼ必ず使うのがページネーションです。

```php
User::query()->paginate(50);
```

とても便利ですが、データ量が増えてくるとページネーションが急に重くなることがあります。

この記事では、Laravelのページネーションを簡単に高速化できるパッケージである自作パッケージ：Laravel-quick-paginator

https://github.com/askdkc/laravel-quick-paginator

を紹介します。

## Laravelのpaginateが重くなる理由

Laravelの通常の `paginate()` は、ページング用のデータ取得に加えて、総件数を取得するための `count` クエリを実行します。

イメージとしては、以下のようなSQLが実行されます。

```sql
select count(*) as aggregate from users;
```

そして実際のデータ取得として、

```sql
select * from users limit 50 offset 0;
```

のようなクエリも実行されます。

つまりLaravelの `paginate()` は基本的に、

1. 総件数を取得する
2. 指定ページのデータを取得する

というSQLが2回走る動きで出来ています。

小規模なテーブルであれば問題ありませんが、以下のような条件が重なると `count` がボトルネックになりやすいです。

- レコード数が多い
- `join` が多い
- `where` 条件が複雑
- `group by` を使っている
- 検索条件が多い
- インデックスが効きづらい

管理画面や検索画面で、ページネーションがやたら遅い場合は、`count` クエリが原因になっていることがあります。

## laravel-quick-paginatorとは

`laravel-quick-paginator` は、Laravelのページネーションを高速化するためのパッケージです。

通常の `paginate()` の代わりに、このパッケージが提供するページネーション `quickPaginate()` を使うことで、一覧画面の表示速度改善が期待できます。

特に、総件数の取得が重いケースで効果を発揮します。

### 通常のページネーション (795.90ms)
![](https://static.zenn.studio/user-upload/0dc47ee16b5d-20260619.png)

### パッケージ利用時のQuickページネーション (264.69ms)
![](https://static.zenn.studio/user-upload/5059dcf2a6be-20260619.png)

## インストール

Composerでインストールします。

```bash
composer require askdkc/laravel-quick-paginator
```

Laravelではパッケージの自動検出が有効になっていれば、基本的に追加設定なしで使えます。

## 使い方

通常のLaravelページネーションでは、次のように書きます。

```php
$users = User::query()
    ->where('status', 'active')
    ->orderByDesc('id')
    ->paginate(50);
```

`laravel-quick-paginator` を使う場合は、パッケージが提供するメソッドに置き換えます。

```php
$users = User::query()
    ->where('status', 'active')
    ->orderByDesc('id')
    ->quickPaginate(50);
```

基本的には `paginate()` を `quickPaginate()` に変えるだけです。

```diff
- ->paginate(50);
+ ->quickPaginate(50);
```

これだけで導入できるのが便利です。

## Controllerでの使用例

例えば、ユーザー一覧画面があるとします。

```php
use App\Models\User;

class UserController extends Controller
{
    public function index()
    {
        $users = User::query()
            ->where('status', 'active')
            ->orderByDesc('id')
            ->quickPaginate(50);

        return view('users.index', compact('users'));
    }
}
```

Blade側は通常のページネーションと同じように扱えます。

```blade
@foreach ($users as $user)
    <div>
        {{ $user->name }}
    </div>
@endforeach

{{ $users->links() }}
```

既存の `links()` がそのまま使えるのは嬉しいポイントです。

## 検索条件付きのページネーション

検索フォームと組み合わせる場合も同じです。

```php
use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index(Request $request)
    {
        $users = User::query()
            ->when($request->filled('keyword'), function ($query) use ($request) {
                $query->where('name', 'like', '%' . $request->keyword . '%');
            })
            ->when($request->filled('status'), function ($query) use ($request) {
                $query->where('status', $request->status);
            })
            ->orderByDesc('id')
            ->quickPaginate(50)
            ->withQueryString();

        return view('users.index', compact('users'));
    }
}
```

`withQueryString()` も通常のページネーションと同じように使えます。

```blade
{{ $users->links() }}
```

## まとめ

データ数が大量になってきた際に「ページネーション遅いなぁ、、、」となって来たら、是非ともご利用ください。
