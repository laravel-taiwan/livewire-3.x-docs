當使用者執行某些操作，例如提交表單後，您可能希望將他們重新導向到應用程式中的另一個頁面。

由於 Livewire 請求不是標準的完整頁面瀏覽器請求，標準的 HTTP 重新導向將無法正常運作。相反，您需要通過 JavaScript 觸發重新導向。幸運的是，Livewire 提供了一個簡單的 `$this->redirect()` 輔助方法供您在組件中使用。在內部，Livewire 將處理在前端重新導向的過程。

如果您喜歡，您也可以在組件中使用 [Laravel 內建的重新導向工具](https://laravel.com/docs/responses#redirects)。

## 基本用法

以下是一個名為 `CreatePost` 的 Livewire 組件的範例，該範例在使用者提交表單以創建文章後將使用者重新導向到另一個頁面：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
	public $title = '';

    public $content = '';

    public function save()
    {
		Post::create([
			'title' => $this->title,
			'content' => $this->content,
		]);

		$this->redirect('/posts'); // [tl! highlight]
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

正如您所見，當觸發 `save` 操作時，將同時觸發重新導向到 `/posts`。當 Livewire 收到此回應時，它將在前端將使用者重新導向到新的 URL。

## 重新導向至路由

如果您想要使用路由名稱重新導向到一個頁面，您可以使用 `redirectRoute`。

例如，如果您有一個名為 `'profile'` 的路由的頁面如下：

```php
    Route::get('/user/profile', function () {
        // ...
    })->name('profile');
```

您可以使用 `redirectRoute` 通過路由名稱重新導向到該頁面，如下所示：

```php
    $this->redirectRoute('profile');
```

如果您需要將參數傳遞給路由，您可以使用 `redirectRoute` 方法的第二個參數，如下所示：

```php
    $this->redirectRoute('profile', ['id' => 1]);
```

## 重新導向至預期頁面

如果您想要將使用者重新導向回他們之前所在的頁面，您可以使用 `redirectIntended`。它接受一個可選的預設 URL 作為第一個參數，如果無法確定之前的頁面，則將用作後備：

```php
    $this->redirectIntended('/default/url');
```

## 重新導向至完整頁面組件

由於 Livewire 使用 Laravel 內建的重新導向功能，您可以在典型的 Laravel 應用程式中使用所有可用的重新導向方法。

例如，如果您將 Livewire 元件用作路由的全頁元件，如下所示：

```php
use App\Livewire\ShowPosts;

Route::get('/posts', ShowPosts::class);
```

您可以通過將元件名稱提供給 `redirect()` 方法來重定向到該元件：

```php
public function save()
{
    // ...

    $this->redirect(ShowPosts::class);
}
```

## 快閃訊息

除了允許您使用 Laravel 內建的重定向方法外，Livewire 還支持 Laravel 的 [會話快閃資料工具](https://laravel.com/docs/session#flash-data)。

要在重定向時傳遞快閃資料，您可以像這樣使用 Laravel 的 `session()->flash()` 方法：

```php
use Livewire\Component;

class UpdatePost extends Component
{
    // ...

    public function update()
    {
        // ...

        session()->flash('status', 'Post successfully updated.');

        $this->redirect('/posts');
    }
}
```

假設被重定向的頁面包含以下 Blade 片段，用戶在更新文章後將看到 "文章已成功更新。" 的訊息：

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
