Livewire 允許您將元件屬性存儲在 URL 的查詢字串中。例如，您可能希望在您的元件中有一個 `$search` 屬性包含在 URL 中：`https://example.com/users?search=bob`。這對於過濾、排序和分頁等功能特別有用，因為它允許用戶分享和書籤頁面的特定狀態。

## 基本用法

以下是一個 `ShowUsers` 元件，允許您通過一個簡單的文本輸入來搜索用戶的名稱：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Url;
use Livewire\Component;
use App\Models\User;

class ShowUsers extends Component
{
    public $search = '';

    public function render()
    {
        return view('livewire.show-users', [
            'users' => User::search($this->search)->get(),
        ]);
    }
}
```

```blade
<div>
    <input type="text" wire:model.live="search">

    <ul>
        @foreach ($users as $user)
            <li wire:key="{{ $user->id }}">{{ $user->name }}</li>
        @endforeach
    </ul>
</div>
```

正如您所看到的，因為文本輸入使用了 `wire:model.live="search"`，當用戶在字段中輸入時，將發送網絡請求以更新 `$search` 屬性並在頁面上顯示一組過濾後的用戶。

然而，如果訪客刷新頁面，搜索值和結果將會丟失。

為了在頁面加載期間保留搜索值，以便訪客可以刷新頁面或分享 URL，我們可以通過在 `$search` 屬性上方添加 `#[Url]` 屬性來將搜索值存儲在 URL 的查詢字串中，如下所示：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Url;
use Livewire\Component;
use App\Models\User;

class ShowUsers extends Component
{
    #[Url] // [tl! highlight]
    public $search = '';

    public function render()
    {
        return view('livewire.show-users', [
            'posts' => User::search($this->search)->get(),
        ]);
    }
}
```

現在，如果用戶在搜索字段中輸入 "bob"，瀏覽器的 URL 欄將顯示：

```
https://example.com/users?search=bob
```

如果他們現在從新的瀏覽器窗口加載此 URL，"bob" 將填寫在搜索字段中，並相應地過濾用戶結果。

## 從 URL 初始化屬性

正如您在前面的示例中看到的，當屬性使用 `#[Url]` 時，它不僅將其更新後的值存儲在 URL 的查詢字串中，還會在頁面加載時參考任何現有的查詢字串值。

例如，如果用戶訪問 URL `https://example.com/users?search=bob`，Livewire 將把 `$search` 的初始值設置為 "bob"。

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url]
    public $search = ''; // Will be set to "bob"...

    // ...
}
```

### 可為空的屬性

默認情況下，如果頁面加載時帶有空的查詢字串項目，如 `?search=`，Livewire 將將該值視為空字符串。在許多情況下，這是預期的，但有時您希望將 `?search=` 視為 `null`。

在這些情況下，您可以像這樣使用可為空的型別提示：

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url]
    public ?string $search; // [tl! highlight]

    // ...
}
```

因為上面的型別提示中存在 `?`，Livewire 將看到 `?search=` 並將 `$search` 設置為 `null` 而不是空字符串。

這也適用於相反的情況，如果在應用程序中設置了 `$this->search = null`，則在查詢字符串中表示為 `?search=`。

## 使用別名

Livewire 允許您完全控制 URL 查詢字符串中顯示的名稱。例如，您可能有一個 `$search` 屬性，但希望將實際屬性名稱混淆或縮短為 `q`。

您可以通過為 `#[Url]` 屬性提供 `as` 參數來指定查詢字符串別名：

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(as: 'q')]
    public $search = '';

    // ...
}
```

現在，當用戶在搜索欄中輸入 "bob" 時，URL 將顯示為：`https://example.com/users?q=bob` 而不是 `?search=bob`。

## 排除特定值

默認情況下，Livewire 只會在值從初始化時發生變化時將條目放入查詢字符串中。大多數情況下，這是期望的行為，但是，在某些情況下，您可能希望更多地控制 Livewire 排除哪些值不包含在查詢字符串中。在這些情況下，您可以使用 `except` 參數。

例如，在下面的組件中，`$search` 的初始值在 `mount()` 中被修改。為了確保瀏覽器只會在 `search` 值為空字符串時才從查詢字符串中排除 `search`，已將 `except` 參數添加到 `#[Url]`：

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(except: '')]
    public $search = '';

    public function mount() {
        $this->search = auth()->user()->username;
    }

    // ...
}
```

在上面的示例中，如果沒有 `except`，Livewire 將在 `search` 的值等於 `auth()->user()->username` 的初始值時從查詢字符串中刪除 `search` 條目。相反，因為使用了 `except: ''`，Livewire 將保留所有查詢字符串值，除非 `search` 為空字符串。

## 頁面加載時顯示

默認情況下，Livewire 只會在頁面上的值發生更改後才在查詢字符串中顯示值。例如，如果 `$search` 的默認值為空字符串：`""`，當實際搜索輸入為空時，在 URL 中將不會顯示任何值。

如果您希望在查詢字符串中始終包含 `?search` 項目，即使值為空，您可以在 `#[Url]` 屬性中提供 `keep` 參數：

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(keep: true)]
    public $search = '';

    // ...
}
```

現在，當頁面加載時，URL 將更改為以下內容：`https://example.com/users?search=`

## 存儲在歷史記錄中

默認情況下，Livewire 使用 [`history.replaceState()`](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState) 來修改 URL，而不是使用 [`history.pushState()`](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState)。這意味著當 Livewire 更新查詢字符串時，它會修改瀏覽器歷史狀態中的當前項目，而不是添加新的項目。

由於 Livewire "替換"了當前歷史記錄，按下瀏覽器的 "返回" 按鈕將返回到上一個頁面，而不是返回到先前的 `?search=` 值。

要強制 Livewire 在更新 URL 時使用 `history.pushState`，您可以在 `#[Url]` 屬性中提供 `history` 參數：

```php
use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(history: true)]
    public $search = '';

    // ...
}
```

在上面的示例中，當用戶將搜索值從 "bob" 更改為 "frank"，然後點擊瀏覽器的返回按鈕時，搜索值（和 URL）將被設置回 "bob"，而不是導航到先前訪問的頁面。

## 使用 queryString 方法

查詢字符串也可以定義為組件上的一個方法。如果某些屬性具有動態選項，這可能很有用。

```php
use Livewire\Component;

class ShowUsers extends Component
{
    // ...

    protected function queryString()
    {
        return [
            'search' => [
                'as' => 'q',
            ],
        ];
    }
}
```

## Trait 鉤子

Livewire 還為查詢字符串提供了[鉤子](/docs/lifecycle-hooks)。

```php
trait WithSorting
{
    // ...

    protected function queryStringWithSorting()
    {
        return [
            'sortBy' => ['as' => 'sort'],
            'sortDirection' => ['as' => 'direction'],
        ];
    }
}
```
