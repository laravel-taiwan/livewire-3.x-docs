計算屬性是 Livewire 中創建“衍生”屬性的一種方式。就像 Eloquent 模型上的取值器一樣，計算屬性允許您訪問值並在請求期間將其緩存以供將來訪問。

計算屬性特別適用於與元件的公共屬性結合使用。

## 基本用法

要創建計算屬性，您可以在 Livewire 元件中的任何方法上方添加 `#[Computed]` 屬性。一旦將屬性添加到方法中，您就可以像訪問任何其他屬性一樣訪問它。

> [!warning] 確保導入屬性類
> 確保導入任何屬性類。例如，下面的 `#[Computed]` 屬性需要以下導入 `use Livewire\Attributes\Computed;`。

例如，這裡有一個名為 `ShowUser` 的元件，該元件使用一個名為 `user()` 的計算屬性來訪問基於名為 `$userId` 的屬性的 `User` Eloquent 模型：

```php
<?php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

class ShowUser extends Component
{
    public $userId;

    #[Computed]
    public function user()
    {
        return User::find($this->userId);
    }

    public function follow()
    {
        Auth::user()->follow($this->user);
    }

    public function render()
    {
        return view('livewire.show-user');
    }
}
```

```blade
<div>
    <h1>{{ $this->user->name }}</h1>

    <span>{{ $this->user->email }}</span>

    <button wire:click="follow">Follow</button>
</div>
```

因為 `user()` 方法上已添加了 `#[Computed]` 屬性，所以該值可以在元件的其他方法和 Blade 模板中訪問。

> [!info] 必須在您的模板中使用 `$this`
> 與普通屬性不同，計算屬性在您的元件模板內部並不直接可用。相反，您必須在 `$this` 對象上訪問它們。例如，名為 `posts()` 的計算屬性必須在您的模板內部通過 `$this->posts` 訪問。

> [!warning] 計算屬性不支持在 `Livewire\Form` 對象上使用。
> 嘗試在 [Form](https://livewire.laravel.com/docs/forms) 內使用計算屬性將導致當您嘗試使用 $form->property 語法在 Blade 中訪問屬性時出錯。

## 性能優勢

您可能會問自己：為什麼要使用計算屬性？為什麼不直接調用方法？

將方法作為計算屬性訪問比調用方法具有性能優勢。內部地，當第一次執行計算屬性時，Livewire 會緩存返回的值。這樣，在請求中的任何後續訪問將返回緩存的值，而不是多次執行。 

--- 

*這是原始文件的翻譯版本，請核對原始文件以獲得最準確的資訊。*

這允許您自由訪問衍生值，而不必擔心性能影響。

> [!warning] 計算屬性僅在單個請求期間緩存
> 一個常見的誤解是 Livewire 會在頁面上 Livewire 元件的整個生命週期中緩存計算屬性。然而，事實並非如此。實際上，Livewire 僅在單個元件請求的持續時間內緩存結果。這意味著如果您的計算屬性方法包含昂貴的資料庫查詢，則每次 Livewire 元件執行更新時都會執行該查詢。

### 清除快取

考慮以下問題情境：
1) 您訪問一個依賴於特定屬性或資料庫狀態的計算屬性
2) 底層屬性或資料庫狀態發生變化
3) 屬性的緩存值變得過時並需要重新計算

要清除或“破壞”存儲的快取，您可以使用 PHP 的 `unset()` 函數。

以下是一個名為 `createPost()` 的動作的示例，通過在應用程序中創建新文章，使 `posts()` 計算變得過時 — 這意味著需要重新計算計算屬性 `posts()` 以包含新添加的文章：

```php
<?php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

class ShowPosts extends Component
{
    public function createPost()
    {
        if ($this->posts->count() > 10) {
            throw new \Exception('Maximum post count exceeded');
        }

        Auth::user()->posts()->create(...);

        unset($this->posts); // [tl! highlight]
    }

    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    // ...
}
```

在上面的元件中，在創建新文章之前，計算屬性被緩存，因為 `createPost()` 方法在創建新文章之前訪問了 `$this->posts`。為了確保在視圖內部訪問時 `$this->posts` 包含最新的內容，使用 `unset($this->posts)` 來使快取失效。

### 在請求之間進行快取

有時您希望將計算屬性的值緩存到 Livewire 元件的生命週期中，而不是在每次請求後清除。在這些情況下，您可以使用 [Laravel 的快取工具](https://laravel.com/docs/cache#retrieve-store)。

以下是一個名為 `user()` 的計算屬性的示例，其中我們將 Eloquent 查詢包裹在 `Cache::remember()` 中，以確保任何未來的請求從 Laravel 的快取中檢索它，而不是重新執行查詢：

因為每個 Livewire 元件的唯一實例都有一個獨特的 ID，我們可以使用 `$this->getId()` 來生成一個獨特的快取鍵，這個鍵只會應用於對該相同元件實例的未來請求。

但是，正如您可能已經注意到的，大部分這段程式碼是可預測的並且可以輕鬆抽象化。因此，Livewire 的 `#[Computed]` 屬性提供了一個有用的 `persist` 參數。通過將 `#[Computed(persist: true)]` 應用於一個方法，您可以在不添加任何額外程式碼的情況下達到相同的結果：

```php
use Livewire\Attributes\Computed;
use App\Models\User;

#[Computed(persist: true)]
public function user()
{
    return User::find($this->userId);
}
```

在上面的示例中，當從您的元件訪問 `$this->user` 時，它將在頁面上 Livewire 元件的持續時間內繼續被快取。這意味著實際的 Eloquent 查詢只會執行一次。

Livewire 將持續值快取 3600 秒（一小時）。您可以通過向 `#[Computed]` 屬性傳遞額外的 `seconds` 參數來覆蓋此默認值：

```php
#[Computed(persist: true, seconds: 7200)]
```

> [!tip] 調用 `unset()` 將清除此快取
> 如前所述，您可以使用 PHP 的 `unset()` 方法清除計算屬性的快取。這也適用於使用 `persist: true` 參數的計算屬性。當在快取的計算屬性上調用 `unset()` 時，Livewire 將不僅清除計算屬性快取，還會清除 Laravel 快取中的底層快取值。

## 跨所有元件進行快取

您可以使用 `#[Computed]` 屬性提供的 `cache: true` 參數，將計算屬性的值在應用程序中所有元件的生命週期中進行快取，而不僅僅是單個元件的生命週期：

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed(cache: true)]
public function posts()
{
    return Post::all();
}
```

在上面的示例中，直到快取過期或被清除，應用程序中每個此元件的實例將共享 `$this->posts` 的相同快取值。

如果您需要手動清除計算屬性的快取，您可以使用 `key` 參數設置自定義快取鍵：

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed(cache: true, key: 'homepage-posts')]
public function posts()
{
    return Post::all();
}
```

## 何時使用計算屬性？

除了提供性能優勢外，還有一些其他情況下計算屬性很有幫助。

具體來說，在將數據傳遞到組件的 Blade 模板時，有幾種情況下計算屬性是更好的選擇。以下是一個簡單組件的 `render()` 方法將一個 `posts` 集合傳遞給 Blade 模板的示例：

```php
public function render()
{
    return view('livewire.show-posts', [
        'posts' => Post::all(),
    ]);
}
```

```blade
<div>
    @foreach ($posts as $post)
        <!-- ... -->
    @endforeach
</div>
```

雖然對於許多用例來說這已經足夠，但以下是三種情況下使用計算屬性會是更好的選擇：

### 有條件地訪問值

如果您在 Blade 模板中有條件地訪問一個計算成本高昂的值，您可以使用計算屬性來減少性能開銷。

考慮以下沒有計算屬性的模板：

```blade
<div>
    @if (Auth::user()->can_see_posts)
        @foreach ($posts as $post)
            <!-- ... -->
        @endforeach
    @endif
</div>
```

如果用戶被限制查看帖子，則檢索帖子的數據庫查詢已經完成，但帖子在模板中從未被使用。

以下是使用計算屬性的上述情況版本：

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed]
public function posts()
{
    return Post::all();
}

public function render()
{
    return view('livewire.show-posts');
}
```

```blade
<div>
    @if (Auth::user()->can_see_posts)
        @foreach ($this->posts as $post)
            <!-- ... -->
        @endforeach
    @endif
</div>
```

現在，因為我們使用計算屬性將帖子提供給模板，只有在需要數據時才執行數據庫查詢。

### 使用內聯模板

另一種計算屬性有幫助的情況是在組件中使用[內聯模板](/docs/components#inline-components)。

以下是一個內聯組件的示例，因為我們直接在 `render()` 內返回模板字符串，所以我們從未有機會將數據傳遞給視圖：

```php
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    #[Computed]
    public function posts()
    {
        return Post::all();
    }

    public function render()
    {
        return <<<HTML
        <div>
            @foreach ($this->posts as $post)
                <!-- ... -->
            @endforeach
        </div>
        HTML;
    }
}
```

在上面的示例中，如果沒有計算屬性，我們將無法明確將數據傳遞給 Blade 模板。

### 省略 render 方法

在 Livewire 中，另一種減少組件中樣板代碼的方法是完全省略 `render()` 方法。當省略時，Livewire 將使用自己的 `render()` 方法按照慣例返回相應的 Blade 視圖。

在這些情況下，顯然您沒有一個`render()`方法，可以通過該方法將數據傳遞給 Blade 視圖。

與其將`render()`方法重新引入到您的組件中，您可以通過計算屬性將該數據提供給視圖：

```php
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    #[Computed]
    public function posts()
    {
        return Post::all();
    }
}
```

```blade
<div>
    @foreach ($this->posts as $post)
        <!-- ... -->
    @endforeach
</div>
```
