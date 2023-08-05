---
title: "LaravelからSupabaseでPGroongaを使う"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['laravel','php','postgresql','全文検索','pgroonga']
published: false
---

# SupabaseをLaravelから使う方法

「いや〜、Laravelでアプリを開発する方法は色々ありますが、やっぱり一番興奮するのはマネージドなDBで高速日本語全文検索のPGroonga使えた時ですね！」

『間違いないね！』

## サインアップ
まずは[supabase](https://supabase.com)にサインアップしてアカウントを作成し、適当に組織を作り、下記のようなダッシュボードにアクセスします

下図のように **Project Settings > Database > Database Setting** の箇所にDBに必要な情報が書かれているので、これを使います

(これはあくまでサンプルなので、既にこのスクリーンショットのプロジェクトは消してあります)

<img width="1422" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/93650fab-da39-4512-975b-dee71a17b5b0">

## Laravelの`.env`へのDB設定

自分のLaravelの`.env`ファイルにダッシュボードでメモした設定値を下記のように入れて行きます（それぞれの箇所は自分の値に直してね）
```vim
DB_CONNECTION=pgsql
DB_HOST=db.phimnigkzjblpukeobou.supabase.co
DB_PORT=5432
DB_DATABASE=postgres
DB_USERNAME=postgres
DB_PASSWORD=自分のパスワード
```

## PGroongaを使っていくぞ！

## ExtensionをONにする
Supabaseは日本語検索に強い`PGroonga`を使える唯一のマネージドなDBなので、使います！

左側の**Database**の**Extension**を開いて検索フィールドに`PGroonga`と入れると出てきます（左側のPGroongaを選択してね）

<img width="1419" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/9938e224-d403-4b81-98fa-0fda1ce1b589">

## Schemaはpublicを選択

データテーブルは基本`public`スキーマに作られるので、`public`にExtensionをONにしておきます

<img width="1419" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/1e35ac08-b290-4534-8304-7d235f797323">

## Laravel側でテーブル作成やダミーデータ作成

### 雛形(Laravel Breeze)準備
画面生成がしやすいのでLaravel Breezeとその日本語化パッケージを入れておきます

```bash
composer require laravel/breeze --dev
composer require askdkc/breezejp --dev

php artisan breeze:install blade
php artisan breezejp
```
これでLaravel Breezeと日本語化が完了しました👍🇯🇵(Breezejpは便利なのでGithubのレポにスターしてあげましょう💝)

### migrationとfactoryの作成

サンプルとしてブログ記事（Post）モデルを作って行きます
```bash
php artisan make:model -mf Post
```

- `database/migrations/yyyy_mm_dd_ssssss_create_posts_table.php`
 
```php
public function up(): void
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->text('title');
        $table->text('body')->nullable();
        $table->timestamps();
    });
}
```

- `database/factories/PostFactory.php`
 
```php
public function definition(): array
{
    return [
        'title' => $this->faker->realText(20),
        'body' => $this->faker->realText(200),
    ];
}
```

- `database/seeders/DatabaseSeeder.php`

```php
public function run(): void
{
    for ($i = 0; $i < 100; $i++) {
        \App\Models\Post::factory(100)->create();
    }
}
```

## migrateと同時にダミーデータ放り込み
```bash
php artisan migrate --seed
```

10,000件のダミーデータを**Supabase**側に作るのには10分程度かかるのでコーヒーでも飲んで待ちます☕️

なお、途中経過はSupabaseダッシュボードの**Table editor > posts**から確認可能です

<img width="1402" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/93bc3b75-e57c-4ace-bc7e-1b097eb6c2e4">

## PGroonga用のインデックス作成（重要）
`PGroonga`を使う上での最重要項目：インデックスを作成します

**SQL Editor > +New query**して、下記を実行します

```sql
CREATE INDEX pg_title_index ON posts USING pgroonga (title);
CREATE INDEX pg_body_index ON posts USING pgroonga (body);
```
<img width="1419" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/e7a0e86a-a914-481f-8166-a1c45dac9c78">

## Laravel側で検索してみるぞ🔍

## 検索用コントローラやビュー作成

### Postモデルに検索機能付与

- `app/Models/Post.php`

```php
class Post extends Model
{
    use HasFactory;

    public static function search($keyword)
    {
        if(empty($keyword)){
            return static::query();
        }

        $search_columns = ['title', 'body'];

        $search_query = static::query();

        foreach($search_columns as $column){
            $search_query->orWhereRaw("$column &@~ ?", [$keyword]);
        }

        return $search_query;
    }
}
```

### 検索用コントローラ作成

```bash
php artisan make:controller SearchController
```

- `app/Http/Controllers/SearchController.php`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class SearchController extends Controller
{
    public function index() : View
    {
        $posts = \App\Models\Post::query()->paginate(10);

        return view('posts.index', compact('posts'));
    }

    public function search(Request $request) : View
    {
        $keyword = $request->input('keyword');

        $posts = \App\Models\Post::search($keyword)->paginate(10)->withQueryString();

        return view('posts.index', compact('posts', 'keyword'));
    }

}
```

### 検索用ビュー作成

- `resources/views/layouts/guest.blade.php`

検索画面に使いやすいように少し幅を拡大
 
**Before**
```html
<!-- Line 25 -->
	<div class="w-full sm:max-w-md mt-6 px-6 py-4 bg-white shadow-md overflow-hidden sm:rounded-lg">
		{{ $slot }}
	</div>
```

**After**
```html
<!-- Line 25 -->
	<div class="w-full lg:max-w-6xl mt-6 px-6 py-4 bg-white shadow-md overflow-hidden sm:rounded-lg">
	    {{ $slot }}
	</div>
```

### 検索兼一覧表示画面
- `resources/views/posts/index.blade.php`
 
```php
<x-guest-layout>
    <div class="py-2">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-2 bg-white border-b border-gray-200">

                    <div>
                        <div class="flex w-full justify-between items-center sm:mb-2">
                            <h2 class="text-3xl font-extrabold tracking-tight text-gray-900 sm:text-4xl sm:mb-4"><a href="/">Supabase Laravel</a> </h2>
                        </div>
                        <form action="{{ route('posts.search') }}" method="GET">
                            <div class="flex mb-4 justify-between items-center">
                                <div class="block w-3/4">
                                    <div class="flex flex-col sm:flex-row justify-start items-center pl-2">
                                    <input type="search" name="keyword" class="form-control w-full sm:w-5/6 " type="text" value="@if (isset($keyword)) {{ $keyword }} @endif" placeholder="{{ __('Enter search keyword') }}">
                                    <button class="collapse sm:visible inline-block align-left text-base sm:w-20 rounded-md border border-gray-700 sm:p-2 sm:ml-4" type="submit">{{ __('Search') }}</button>
                                    </div>
                                </div>
                            </div>
                        </form>

                        @if($posts ?? false)
                        <div class="container">
                            <div class="bg-white">
                              <div class="max-w-2xl mx-auto px-4 grid items-center grid-cols-1 gap-y-16 gap-x-8 sm:px-6 lg:max-w-7xl lg:px-8 lg:grid-cols-1">
                                <div>

                                  <dl class="sm:mt-8 grid grid-cols-1 gap-x-6 gap-y-2 grid-cols-1 sm:grid-cols-7 sm:gap-y-2 lg:gap-x-8">
                                    <div class="border-t border-gray-200">
                                      <dt class="text-sm sm:text-base sm:font-medium text-gray-900">
                                              ID
                                      </dt>
                                    </div>

                                    <div class="border-t border-gray-200 sm:col-span-2">
                                      <dt class="text-sm sm:text-base sm:font-medium text-gray-900">
                                        {{ __('Title') }}
                                      </dt>
                                    </div>

                                    <div class="border-t border-gray-200 sm:col-span-4">
                                      <dt class="text-sm sm:text-base font-medium text-gray-900">
                                        {{ __('Body') }}
                                      </dt>
                                    </div>

                                    @foreach($posts as $post)
                                        <div class="border-t border-gray-200 pt-1">
                                            <dd class="sm:mt-2 text-sm text-gray-500">
                                                <span class="block m-1">ID: {{ $post->id }}</span>
                                            </dd>
                                        </div>

                                        <div class="border-t border-gray-200 pt-1 sm:col-span-2">
                                            <dd class="sm:mt-2 text-sm text-gray-500">
                                                <span class="block m-1">{{ $post->title }}</span>
                                            </dd>
                                        </div>

                                        <div class="border-t border-gray-200 pt-1 sm:col-span-4">
                                          <dd class="sm:mt-2 text-sm text-gray-500">
                                              <span class="block m-1 line-break">{{ $post->body }}</span>
                                          </dd>
                                        </div>

                                    @endforeach
                                  </dl>
                                </div>

                                  <div>
                                    {{ $posts->links() }}
                                  </div>

                              </div>
                            </div>
                        </div>
                        @endif

                    </div>

                </div>
            </div>
        </div>
    </div>
</x-guest-layout>
```

## Route設定

- `routes/web.php`

```php
/* 元々のTOPページの箇所をコメントアウト
Route::get('/', function () {
    return view('welcome');
});
*/

Route::get('/',  [\App\Http\Controllers\SearchController::class, 'index']) // 代わりに追加
    ->name('posts.index'); // 代わりに追加
Route::get('/search',  [\App\Http\Controllers\SearchController::class, 'search']) // 代わりに追加
    ->name('posts.search'); // 代わりに追加

(残りは省略)
```

## 動作確認

必要なCSSとかビルドして起動
```bash
npm run build
php artisan serve
```

http://127.0.0.1:8000 へアクセス

「カムパネルラ」で検索してみます

<img width="1402" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/3cafbff0-9ff2-4a43-bee8-d32023a8da6d">


## PGroongaの動作確認(インデックス利用)
<img width="1417" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/3ab5b920-32a5-427f-aa38-70556530ce91">

## ちなみに 10,000件程度でも結構容量食い😅
<img width="1417" alt="image" src="https://github.com/askdkc/supa-laravel/assets/7894265/7b58f1a0-4bc1-415d-89b3-a72589ac5275">

