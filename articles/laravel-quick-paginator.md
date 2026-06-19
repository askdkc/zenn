---
title: "Laravelのページネーションを瞬時に高速化"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [laravel, php]
published: false
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


## paginateとの使い分け

すべての画面で置き換えれば良い、というわけではありません。

通常の `paginate()` は、総件数や最終ページ番号が必要な場合に便利です。

例えば、以下のような表示が必要な画面です。

```text
全 12,345 件中 1 - 50 件を表示
```

または、

```text
1 2 3 4 5 ... 247
```

のように最終ページまで含めたページ番号を正確に出したい場合です。

一方で、一覧画面の表示速度を優先したい場合や、総件数が不要な場合は `quickPaginate()` が向いています。

