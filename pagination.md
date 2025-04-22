Laravel的分頁功能允許您查詢部分資料，並讓用戶能夠在這些結果的*頁面*之間導航。

由於Laravel的分頁器是為靜態應用程序設計的，在非Livewire應用程序中，每次頁面導航都會觸發對包含所需頁面的新URL的完整瀏覽器訪問（`?page=2`）。

然而，當您在Livewire組件內部使用分頁時，用戶可以在保持在同一頁面的情況下在不同頁面之間導航。Livewire將在幕後處理所有事情，包括使用當前頁面更新URL查詢字符串。

## 基本用法

以下是在`ShowPosts`組件內部使用分頁的最基本示例，每次只顯示十篇文章：

> [!warning] 您必須使用`WithPagination`特性
> 為了利用Livewire的分頁功能，每個包含分頁的組件都必須使用`Livewire\WithPagination`特性。

```php
<?php

namespace App\Livewire;

use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    use WithPagination;

    public function render()
    {
        return view('show-posts', [
            'posts' => Post::paginate(10),
        ]);
    }
}
```

```blade
<div>
    <div>
        @foreach ($posts as $post)
            <!-- ... -->
        @endforeach
    </div>

    {{ $posts->links() }}
</div>
```

如您所見，除了通過`Post::paginate()`方法限制顯示的文章數量外，我們還將使用`$posts->links()`來呈現頁面導航鏈接。

有關使用Laravel進行分頁的更多信息，請查看[Laravel的全面分頁文檔](https://laravel.com/docs/pagination)。

## 禁用URL查詢字符串跟踪

默認情況下，Livewire的分頁器會在瀏覽器URL的查詢字符串中跟踪當前頁面，如`?page=2`所示。

如果您仍希望使用Livewire的分頁實用程序，但禁用查詢字符串跟踪，可以使用`WithoutUrlPagination`特性：

```php
use Livewire\WithoutUrlPagination;
use Livewire\WithPagination;
use Livewire\Component;

class ShowPosts extends Component
{
    use WithPagination, WithoutUrlPagination; // [tl! highlight]

    // ...
}
```

現在，分頁將按預期工作，但當前頁面不會顯示在查詢字符串中。這也意味著當前頁面在頁面更改時不會保留。

## 自定義滾動行為

默認情況下，Livewire的分頁器在每次頁面更改後滾動到頁面頂部。

您可以通過將`false`傳遞給`links()`方法的`scrollTo`參數來禁用此行為，如下所示：

```blade
{{ $posts->links(data: ['scrollTo' => false]) }}
```

或者，您可以將任何 CSS 選擇器提供給 `scrollTo` 參數，Livewire 將查找與該選擇器匹配的最近元素，並在每次導航後滾動到該元素：

```blade
{{ $posts->links(data: ['scrollTo' => '#paginated-posts']) }}
```

## 重置頁面

在排序或篩選結果時，通常希望將頁碼重置為 `1`。

因此，Livewire 提供了 `$this->resetPage()` 方法，允許您從組件的任何位置重置頁碼。

以下組件演示了在提交搜索表單後使用此方法重置頁面：

```php
<?php

namespace App\Livewire;

use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

class SearchPosts extends Component
{
    use WithPagination;

    public $query = '';

    public function search()
    {
        $this->resetPage();
    }

    public function render()
    {
        return view('show-posts', [
            'posts' => Post::where('title', 'like', '%'.$this->query.'%')->paginate(10),
        ]);
    }
}
```

```blade
<div>
    <form wire:submit="search">
        <input type="text" wire:model="query">

        <button type="submit">Search posts</button>
    </form>

    <div>
        @foreach ($posts as $post)
            <!-- ... -->
        @endforeach
    </div>

    {{ $posts->links() }}
</div>
```

現在，如果用戶在結果的第 `5` 頁，然後進一步篩選結果，按下 "搜索文章"，頁面將被重置為 `1`。

### 可用的頁面導航方法

除了 `$this->resetPage()` 外，Livewire 還提供其他有用的方法，可以在組件中以編程方式在頁面之間導航：

| 方法        | 說明                               |
|-----------------|-------------------------------------------|
| `$this->setPage($page)`    | 將分頁器設置為特定頁碼 |
| `$this->resetPage()`    | 將頁面重置為 1 |
| `$this->nextPage()`    | 轉到下一頁 |
| `$this->previousPage()`    | 轉到上一頁 |

## 多個分頁器

因為 Laravel 和 Livewire 都使用 URL 查詢字符串參數來存儲和跟踪當前頁碼，如果單個頁面包含多個分頁器，重要的是為它們分配不同的名稱。

為了更清楚地演示問題，考慮以下 `ShowClients` 組件：

```php
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Client;

class ShowClients extends Component
{
    use WithPagination;

    public function render()
    {
        return view('show-clients', [
            'clients' => Client::paginate(10),
        ]);
    }
}
```

如您所見，上述組件包含一組分頁的*客戶*。如果用戶要導航到此結果集的第 `2` 頁，URL 可能如下所示：

```
http://application.test/?page=2
```

假設該頁面還包含一個 `ShowInvoices` 組件，該組件也使用分頁。為了獨立跟踪每個分頁器的當前頁碼，您需要為第二個分頁器指定一個名稱，如下所示：

現在，由於已將 `pageName` 參數添加到 `paginate` 方法中，當用戶訪問 *發票* 的第 `2` 頁時，URL 將包含以下內容：

```
https://application.test/customers?page=2&invoices-page=2
```

在使用 Livewire 的頁面導航方法時，對於具名分頁器，您必須提供頁面名稱作為額外參數：

```php
$this->setPage(2, pageName: 'invoices-page');

$this->resetPage(pageName: 'invoices-page');

$this->nextPage(pageName: 'invoices-page');

$this->previousPage(pageName: 'invoices-page');
```

## 鉤取頁面更新

Livewire 允許您在頁面更新之前和之後執行代碼，方法是在組件內定義以下任一方法：

```php
use Livewire\WithPagination;

class ShowPosts extends Component
{
    use WithPagination;

    public function updatingPage($page)
    {
        // Runs before the page is updated for this component...
    }

    public function updatedPage($page)
    {
        // Runs after the page is updated for this component...
    }

    public function render()
    {
        return view('show-posts', [
            'posts' => Post::paginate(10),
        ]);
    }
}
```

### 具名分頁器鉤取

前面的鉤子僅適用於默認分頁器。如果您使用具名分頁器，則必須使用分頁器的名稱定義方法。

例如，以下是針對名為 `invoices-page` 的分頁器的鉤子示例：

```php
public function updatingInvoicesPage($page)
{
    //
}
```

### 通用分頁器鉤取

如果您不想在鉤子方法名稱中引用分頁器名稱，則可以使用更通用的替代方法，並簡單地將 `$pageName` 作為鉤子方法的第二個參數接收：

```php
public function updatingPaginators($page, $pageName)
{
    // Runs before the page is updated for this component...
}

public function updatedPaginators($page, $pageName)
{
    // Runs after the page is updated for this component...
}
```

## 使用簡單主題

您可以使用 Laravel 的 `simplePaginate()` 方法，而不是 `paginate()`，以獲得更快速和簡單的分頁。

使用此方法進行分頁結果時，將僅向用戶顯示 *下一個* 和 *上一個* 導航鏈接，而不是每個頁碼的單獨鏈接：

```php
public function render()
{
    return view('show-posts', [
        'posts' => Post::simplePaginate(10),
    ]);
}
```

有關簡單分頁的更多信息，請查看 [Laravel 的 "simplePaginator" 文件](https://laravel.com/docs/pagination#simple-pagination)。

## 使用游標分頁

Livewire 還支持使用 Laravel 的游標分頁 — 一種在大型數據集中非常有用的更快速的分頁方法：

```php
public function render()
{
    return view('show-posts', [
        'posts' => Post::cursorPaginate(10),
    ]);
}
```

通過使用 `cursorPaginate()` 而不是 `paginate()` 或 `simplePaginate()`，您的應用程序 URL 中的查詢字符串將存儲編碼的 *游標*，而不是標準的頁碼。例如：

```
https://example.com/posts?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

有關游標分頁的更多資訊，請查看[Laravel的游標分頁文件](https://laravel.com/docs/pagination#cursor-pagination)。

## 使用Bootstrap而不是Tailwind

如果您正在使用[Bootstrap](https://getbootstrap.com/)而不是[Tailwind](https://tailwindcss.com/)作為應用程式的CSS框架，您可以配置Livewire以使用Bootstrap風格的分頁視圖，而不是默認的Tailwind視圖。

為此，在應用程式的`config/livewire.php`文件中設置`pagination_theme`配置值：

```php
'pagination_theme' => 'bootstrap',
```

> [!info] 發佈Livewire的配置文件
> 在自定義分頁主題之前，您必須首先將Livewire的配置文件發佈到應用程式的`/config`目錄中，方法是運行以下命令：
> ```shell
> php artisan livewire:publish --config
> ```

## Modifying the default pagination views

If you want to modify Livewire's pagination views to fit your application's style, you can do so by *publishing* them using the following command:

```shell
php artisan livewire:publish --pagination

After running this command, the following four files will be inserted into the `resources/views/vendor/livewire` directory:

| View file name        | Description                               |
|-----------------|-------------------------------------------|
| `tailwind.blade.php`    | The standard Tailwind pagination theme |
| `tailwind-simple.blade.php`    | The *simple* Tailwind pagination theme |
| `bootstrap.blade.php`    | The standard Bootstrap pagination theme |
| `bootstrap-simple.blade.php`    | The *simple* Bootstrap pagination theme |

Once the files have been published, you have complete control over them. When rendering pagination links using the paginated result's `->links()` method inside your template, Livewire will use these files instead of its own.

## Using custom pagination views

If you wish to bypass Livewire's pagination views entirely, you can render your own in one of two ways:

1. The `->links()` method in your Blade view
2. The `paginationView()` or `paginationSimpleView()` method in your component

### Via `->links()`

The first approach is to simply pass your custom pagination Blade view name to the `->links()` method directly:

```blade
{{ $posts->links('custom-pagination-links') }}

When rendering the pagination links, Livewire will now look for a view at `resources/views/custom-pagination-links.blade.php`.

### Via `paginationView()` or `paginationSimpleView()`

The second approach is to declare a `paginationView` or `paginationSimpleView` method inside your component which returns the name of the view you would like to use:

```php
public function paginationView()
{
    return 'custom-pagination-links-view';
}

public function paginationSimpleView()
{
    return 'custom-simple-pagination-links-view';
}

### Sample pagination view

Below is an unstyled sample of a simple Livewire pagination view for your reference.

As you can see, you can use Livewire's page navigation helpers like `$this->nextPage()` directly inside your template by adding `wire:click="nextPage"` to buttons:

```blade
<div>
    @if ($paginator->hasPages())
        <nav role="navigation" aria-label="Pagination Navigation">
            <span>
                @if ($paginator->onFirstPage())
                    <span>Previous</span>
                @else
                    <button wire:click="previousPage" wire:loading.attr="disabled" rel="prev">Previous</button>
                @endif
            </span>

            <span>
                @if ($paginator->onLastPage())
                    <span>Next</span>
                @else
                    <button wire:click="nextPage" wire:loading.attr="disabled" rel="next">Next</button>
                @endif
            </span>
        </nav>
    @endif
</div>
```
