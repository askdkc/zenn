---
title: "LaravelのBladeにSvelteコンポーネントを埋め込む"
emoji: "😉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Laravel','Svelte','Vite']
published: false
---
# Laravel 10 with ViteでSvelte.svelteを使う下準備
SvelteコンポーネントをLaravelのbladeから呼び出して使うメモ

## 最新版Laravelを入れよう
```
composer global update
```

## 新規Laravelプロジェクト作成
```
laravel new laravel-vite
cd laravel-vite
```

## まずはBreezeを入れる（何故ならTailwindcssが使いやすいから）
```
composer require laravel/breeze --dev
php artisan breeze:install blade
```

## Svelteプラグインのインストール
[SvelteのViteプラグイン](https://github.com/sveltejs/vite-plugin-svelte)入れます
```
npm i --save-dev @sveltejs/vite-plugin-svelte
```

## vite.config.jsの書き換え
```
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
            ],
            refresh: true,
        }),
    ],
});
```
↓
```
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { svelte } from '@sveltejs/vite-plugin-svelte'; // 追加

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
                'resources/js/main.js', // 追加
            ],
            refresh: true,
        }),
        // この下追加
        svelte({
            /* plugin options */
        }),
    ],
});

```

## resources/js/app.jsの修正
```
import './bootstrap';

import Alpine from 'alpinejs';

window.Alpine = Alpine;

Alpine.start();
```
↓
```
import './bootstrap';

// cssの読み込み追加
import '../css/app.css';

import Alpine from 'alpinejs';

window.Alpine = Alpine;

Alpine.start();
```

## tailwind.config.jsの修正（これしないとSvelteにTailwindcssのスタイルが反映されない）
```
const defaultTheme = require('tailwindcss/defaultTheme');

/** @type {import('tailwindcss').Config} */
module.exports = {
    content: [
        './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
        './storage/framework/views/*.php',
        './resources/views/**/*.blade.php',
    ],

    theme: {
        extend: {
            fontFamily: {
                sans: ['Nunito', ...defaultTheme.fontFamily.sans],
            },
        },
    },

    plugins: [require('@tailwindcss/forms')],
};
```
↓
```
const defaultTheme = require('tailwindcss/defaultTheme');

/** @type {import('tailwindcss').Config} */
module.exports = {
    content: [
        './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
        './storage/framework/views/*.php',
        './resources/views/**/*.blade.php',
        
        // Svelte用の設定追加
        './resources/js/**/*.svelte',
    ],

    theme: {
        extend: {
            fontFamily: {
                sans: ['Figtree', ...defaultTheme.fontFamily.sans],
            },
        },
    },

    plugins: [require('@tailwindcss/forms')],
};
```

## Svelteコンポーネント`Hello.svelte`と呼び出し用のjs（`main.js`とします）を作成
Svelte用のコンポーネントとして `/resources/js/Hello.svelte` を作る例(自分のプロジェクトに合わせて適当に変えてね)
`resources/js`配下

- `main.js`
```main.js
import App from "./Hello.svelte";

const app = new App({
    target: document.getElementById("app"),
});

export default app;
```

- `Hello.svelte`
```svelte
<script></script>

<h1>Hello Svelte!</h1>
```

## guest.blade.php or app.blade.phpを使うと便利
*.blade.phpはテンプレートとして
```
<x-guest-layout>
</x-guest-layout>

もしくは

<x-app-layout>
</x-app-layout>
```
を使うと便利。
何故なら↓のように<head>タグ内に@viteでスクリプトの読み込み設定が既に記載済みなので
```
<head>
    (省略)
    <!-- Scripts -->
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

## Svelteコンポーネントの埋め込み
```
(welcome.blade.phpを丸ごと↓置き換えちゃうとテストしやすい)

<x-guest-layout>
    <div id="app"></div>
    @vite(['resources/js/main.js'])
</x-guest-layout>
```

## 試してみよう (Laravel Valet利用例)
```
npm run dev

を実行後表示される

LARAVEL v10.17.1  plugin v0.7.8 

  > APP_URL: http://laravel-vite.test  ←こちらにアクセス(Valet無しの場合、Localhost or 127.0.0.1)
```

## スクリーンショット
