Livewire 允許您延遲加載組件，否則會拖慢初始頁面加載速度。

例如，假設您有一個包含在 `mount()` 中的緩慢數據庫查詢的 `Revenue` 組件：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Transaction;

class Revenue extends Component
{
    public $amount;

    public function mount()
    {
        // Slow database query...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }

    public function render()
    {
        return view('livewire.revenue');
    }
}
```

```blade
<div>
    本月收入：{{ $amount }}
</div>
```

如果不使用延遲加載，這個組件將延遲整個頁面的加載，使整個應用程序感覺緩慢。

要啟用延遲加載，您可以將 `lazy` 參數傳遞給組件：

```blade
<livewire:revenue lazy />
```

現在，Livewire 不會立即加載該組件，而是跳過該組件，將頁面加載完成。然後，當該組件在視口中可見時，Livewire 將發送網絡請求，完全加載該組件到頁面上。

> [!info] 默認情況下，延遲請求是隔離的
> 與 Livewire 中的其他網絡請求不同，當發送到服務器時，延遲加載更新是相互隔離的。這樣做可以保持延遲加載的速度，當頁面加載時並行加載每個組件。[在此處閱讀有關禁用此行為的更多信息 →](#disabling-request-isolation)

## 渲染佔位符 HTML

默認情況下，Livewire 將在完全加載組件之前插入一個空的 `<div></div>`。由於該組件最初對用戶不可見，當該組件突然出現在頁面上時，可能會讓人感到突兀。

為了向用戶表明組件正在加載中，您可以定義一個 `placeholder()` 方法來渲染任何您喜歡的佔位符 HTML，包括加載旋轉器和骨架佔位符：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Transaction;

class Revenue extends Component
{
    public $amount;

    public function mount()
    {
        // Slow database query...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }

    public function placeholder()
    {
        return <<<'HTML'
        <div>
            <!-- Loading spinner... -->
            <svg>...</svg>
        </div>
        HTML;
    }

    public function render()
    {
        return view('livewire.revenue');
    }
}
```

由於上述組件通過從 `placeholder()` 方法返回 HTML 來指定了“佔位符”，用戶將在該組件完全加載之前在頁面上看到 SVG 加載旋轉器。

> [!warning] 佔位符和組件必須共享相同的元素類型
> 例如，如果您的佔位符的根元素類型是 'div'，則您的組件也必須使用 'div' 元素。

### 通過視圖渲染佔位符

對於更複雜的加載器（例如骨架屏），您可以從`placeholder()`返回一個`view`，類似於`render()`。

```php
public function placeholder(array $params = [])
{
    return view('livewire.placeholders.skeleton', $params);
}
```

從懶加載的組件傳遞的任何參數將作為傳遞給`placeholder()`方法的`$params`參數可用。

## 視窗外的懶加載

默認情況下，懶加載的組件在進入瀏覽器的視窗之前不會完全加載，例如當用戶滾動到其中之一時。

如果您希望在頁面加載後立即懶加載所有組件，而不必等待它們進入視窗，您可以通過將"on-load"傳遞給`lazy`參數來實現：

```blade
<livewire:revenue lazy="on-load" />
```

現在，此組件將在頁面準備就緒後加載，而無需等待其進入視窗。

## 傳遞參數

一般情況下，您可以將`lazy`組件視為普通組件，因為您仍然可以從外部將數據傳遞給它們。

例如，這裡是一個情況，您可能從父組件將時間間隔傳遞給`Revenue`組件：

```blade
<input type="date" wire:model="start">
<input type="date" wire:model="end">

<livewire:revenue lazy :$start :$end />
```

您可以像對待任何其他組件一樣在`mount()`中接受這些數據：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Transaction;

class Revenue extends Component
{
    public $amount;

    public function mount($start, $end)
    {
        // Expensive database query...
        $this->amount = Transactions::between($start, $end)->sum('amount');
    }

    public function placeholder()
    {
        return <<<'HTML'
        <div>
            <!-- Loading spinner... -->
            <svg>...</svg>
        </div>
        HTML;
    }

    public function render()
    {
        return view('livewire.revenue');
    }
}
```

但是，與普通組件加載不同，`lazy`組件必須序列化或“脫水”任何傳入的屬性，並將它們暫時存儲在客戶端，直到組件完全加載。

例如，您可能希望像這樣將一個Eloquent模型傳遞給`Revenue`組件：

```blade
<livewire:revenue lazy :$user />
```

在普通組件中，實際的PHP內存中的`$user`模型將被傳遞給`Revenue`的`mount()`方法。但是，由於我們直到下一個網絡請求才運行`mount()`，Livewire將內部將`$user`序列化為JSON，然後在處理下一個請求之前重新查詢它。

通常，這種序列化不應導致應用程序行為上的任何差異。

## 默認懶加載

如果您希望強制所有組件的使用都是懶加載的，您可以在組件類上方添加`#[Lazy]`屬性：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Lazy;

#[Lazy]
class Revenue extends Component
{
    // ...
}
```

如果您想覆蓋延遲加載，可以將 `lazy` 參數設置為 `false`：

```blade
<livewire:revenue :lazy="false" />
```

### 禁用請求隔離

如果頁面上有多個延遲加載的組件，則每個組件將進行獨立的網絡請求，而不是將每個延遲更新捆綁到單個請求中。

如果您想禁用此隔離行為，而是將所有更新捆綁到單個網絡請求中，可以使用 `isolate: false` 參數：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Lazy;

#[Lazy(isolate: false)] // [tl! highlight]
class Revenue extends Component
{
    // ...
}
```

現在，如果同一頁面上有十個 `Revenue` 組件，當頁面加載時，所有十個更新將被捆綁並作為單個網絡請求發送到服務器。

## 全頁面延遲加載

您可能希望延遲加載全頁面 Livewire 組件。您可以通過在路由上調用 `->lazy()` 來實現：

```php
Route::get('/dashboard', \App\Livewire\Dashboard::class)->lazy();
```

或者，如果有一個默認為延遲加載的組件，並且您想要退出延遲加載，則可以使用以下 `enabled: false` 參數：

```php
Route::get('/dashboard', \App\Livewire\Dashboard::class)->lazy(enabled: false);
```

## 默認占位視圖

如果您想為所有組件設置默認占位視圖，可以通過在 `/config/livewire.php` 配置文件中引用視圖來實現：

```php
'lazy_placeholder' => 'livewire.placeholder',
```

現在，當一個組件被延遲加載且未定義 `placeholder()` 時，Livewire 將使用配置的 Blade 視圖（在這種情況下為 `livewire.placeholder`）。

## 在測試中禁用延遲加載

當單元測試一個延遲組件，或者一個帶有嵌套延遲組件的頁面時，您可能希望禁用“延遲”行為，以便您可以斷言最終呈現的行為。否則，在測試期間，這些組件將被呈現為它們的占位符。

您可以使用 `Livewire::withoutLazyLoading()` 測試輔助工具輕鬆禁用延遲加載，如下所示：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\Dashboard;
use Livewire\Livewire;
use Tests\TestCase;

class DashboardTest extends TestCase
{
    public function test_renders_successfully()
    {
        Livewire::withoutLazyLoading() // [tl! highlight]
            ->test(Dashboard::class)
            ->assertSee(...);
    }
}
```

現在，當為這個測試呈現儀表板元件時，它將跳過呈現 `placeholder()`，而是呈現完整的元件，就好像沒有應用延遲載入一樣。
