Livewire 使得使用 `#[Session]` 屬性跨頁面刷新/更改持久化屬性值變得容易。

通過在組件的屬性中添加 `#[Session]`，Livewire 將在每次屬性更改時將該屬性的值存儲在會話中。這樣，當頁面刷新時，Livewire 將從會話中獲取最新值並在您的組件中使用它。

`#[Session]` 屬性類似於 [`#[Url]`](/docs/url) 屬性。它們在類似的情況下都很有用。主要區別在於 `#[Session]` 在不修改 URL 查詢字符串的情況下持久化值，這有時是需要的，有時不是。

## 基本用法

這裡有一個 `ShowPosts` 組件，允許用戶通過存儲在 `$search` 屬性中的字符串來篩選可見帖子：

```php
<?php

use Livewire\Attributes\Session;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    #[Session] // [tl! highlight]
    public $search;

    protected function posts()
    {
        return $this->search === ''
            ? Post::all()
            : Post::where('title', 'like', '%'.$this->search.'%');
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => $this->posts(),
        ]);
    }
}
```

因為 `$search` 屬性已添加 `#[Session]` 屬性，用戶輸入搜索值後，他們可以刷新頁面，搜索值將被持久化。每次 `$search` 更新時，其新值將存儲在用戶的會話中並在頁面加載時使用。

> [!warning] 性能影響
> 由於 Laravel 會話在每次請求時都加載到內存中，通過在用戶會話中存儲過多內容，可能會降低整個應用程序對於特定用戶的性能。

## 設置自定義鍵

使用 `[#Session]` 時，Livewire 將使用由組件名稱結合屬性名稱組成的動態生成鍵將屬性值存儲在會話中。

這確保了跨組件實例的屬性將使用相同的會話值。它還確保了來自不同組件的同名屬性不會發生衝突。

如果您想完全控制 Livewire 為給定屬性使用的會話鍵，您可以傳遞 `key:` 參數：

```php
<?php

use Livewire\Attributes\Session;
use Livewire\Component;

class ShowPosts extends Component
{
    #[Session(key: 'search')] // [tl! highlight]
    public $search;

    // ...
}
```

當 Livewire 存儲和檢索 `$search` 屬性的值時，將使用給定的鍵："search"。

此外，如果您想從組件中的其他屬性動態生成鍵，可以使用以下大括號表示法：

在上面的例子中，如果 `$author` 模型的 id 是 "4"，則會將 session 金鑰變為：`search-4`。
