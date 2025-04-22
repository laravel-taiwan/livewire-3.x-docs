要開始 Livewire 之旅，我們將創建一個簡單的 "counter" 元件並在瀏覽器中呈現它。這個範例是體驗 Livewire 的絕佳方式，因為它以最簡單的方式展示了 Livewire 的「即時性」。

## 先決條件

在我們開始之前，請確保您已安裝以下項目：

- Laravel 版本 10 或更新版本
- PHP 版本 8.1 或更新版本

## 安裝 Livewire

從您的 Laravel 應用程式根目錄中，執行以下 [Composer](https://getcomposer.org/) 指令：

```shell
composer require livewire/livewire
```

> [!warning] 請確保尚未安裝 Alpine
> 如果您使用的應用程式已經安裝了 AlpineJS，您將需要將其移除以使 Livewire 正常運作；否則，Alpine 將被加載兩次，Livewire 將無法運作。例如，如果您安裝了 Laravel Breeze 的「Blade with Alpine」起始套件，您將需要從 `resources/js/app.js` 中移除 Alpine。

## 創建 Livewire 元件

Livewire 提供了一個方便的 Artisan 指令，可以快速生成新的元件。執行以下指令以創建新的 `Counter` 元件：

```shell
php artisan make:livewire counter
```

此指令將在您的專案中生成兩個新檔案：
* `app/Livewire/Counter.php`
* `resources/views/livewire/counter.blade.php`

## 撰寫類別

打開 `app/Livewire/Counter.php`，並將其內容替換為以下內容：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function decrement()
    {
        $this->count--;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

這裡是上面程式碼的簡要說明：
- `public $count = 1;` — 宣告一個名為 `$count` 的公共屬性，初始值為 `1`。
- `public function increment()` — 宣告一個名為 `increment()` 的公共方法，每次調用時都會增加 `$count` 屬性的值。像這樣的公共方法可以以各種方式從瀏覽器觸發，包括當使用者點擊按鈕時。
- `public function render()` — 宣告一個 `render()` 方法，返回一個 Blade 視圖。這個 Blade 視圖將包含我們元件的 HTML 模板。

## 撰寫視圖

打開 `resources/views/livewire/counter.blade.php` 檔案，並將其內容替換為以下內容：

```blade
<div>
    <h1>{{ $count }}</h1>

    <button wire:click="increment">+</button>

    <button wire:click="decrement">-</button>
</div>
```

此代碼將顯示 `$count` 屬性的值以及兩個按鈕，分別用於增加和減少 `$count` 屬性。

> [!warning] Livewire 元件必須有單一根元素
> 為了讓 Livewire 正常運作，元件必須只有 **一個** 單一元素作為其根元素。如果檢測到多個根元素，將拋出異常。建議使用 `<div>` 元素，就像範例中一樣。HTML 註釋被視為獨立元素，應放在根元素內。
> 當渲染[全頁面元件](/docs/components#full-page-components)時，用於佈局檔案的命名插槽可能放在根元素之外。這些插槽在元件渲染之前被移除。

## 註冊元件的路由

在 Laravel 應用程式中的 `routes/web.php` 檔案中加入以下程式碼：

```php
use App\Livewire\Counter;

Route::get('/counter', Counter::class);
```

現在，我們的 _counter_ 元件被指派給 `/counter` 路由，因此當使用者訪問您應用程式中的 `/counter` 端點時，此元件將由瀏覽器渲染。

## 建立模板佈局

在您可以在瀏覽器中訪問 `/counter` 之前，我們需要一個 HTML 佈局，供我們的元件在其中渲染。預設情況下，Livewire 將自動尋找名為：`resources/views/components/layouts/app.blade.php` 的佈局檔案。

如果該檔案不存在，您可以執行以下命令來建立：

```shell
php artisan livewire:layout
```

此命令將生成一個名為 `resources/views/components/layouts/app.blade.php` 的檔案，內容如下：

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? 'Page Title' }}</title>
    </head>
    <body>
        {{ $slot }}
    </body>
</html>
```

在上述模板中，_counter_ 元件將被渲染到 `$slot` 變數的位置。

您可能已經注意到 Livewire 沒有提供任何 JavaScript 或 CSS 資源。這是因為 Livewire 3 及以上版本會自動注入其所需的前端資源。

## 測試

有了我們的元件類別和模板，我們的元件已經準備好進行測試了！

在瀏覽器中訪問 `/counter`，您應該會看到屏幕上顯示一個數字，並有兩個按鈕用於增加和減少數字。

點擊其中一個按鈕後，您會注意到計數會實時更新，而無需重新加載頁面。這就是 Livewire 的魔力：完全用 PHP 編寫的動態前端應用程序。

我們僅僅觸及了 Livewire 的功能表面。繼續閱讀文檔，以查看 Livewire 提供的所有功能。
