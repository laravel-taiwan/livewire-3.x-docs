與許多其他基於組件的框架一樣，Livewire 組件是可以嵌套的 — 這意味著一個組件可以在其內部呈現多個組件。

然而，由於 Livewire 的嵌套系統與其他框架建構方式不同，因此有一些重要的影響和限制需要注意。

> [!tip] 確保您首先了解水合
> 在深入了解 Livewire 的嵌套系統之前，完全了解 Livewire 如何使組件水合是有幫助的。您可以通過閱讀 [水合文檔](/docs/hydration) 來瞭解更多。

## 每個組件都是一個獨立的單元

在 Livewire 中，頁面上的每個組件都跟蹤其狀態並獨立於其他組件進行更新。

例如，考慮以下 `Posts` 和嵌套的 `ShowPost` 組件：

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class Posts extends Component
{
    public $postLimit = 2;

    public function render()
    {
        return view('livewire.posts', [
            'posts' => Auth::user()->posts()
                ->limit($this->postLimit)->get(),
        ]);
    }
}
```

```blade
<div>
    Post Limit: <input type="number" wire:model.live="postLimit">

    @foreach ($posts as $post)
        <livewire:show-post :$post :key="$post->id">
    @endforeach
</div>
```

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

```blade
<div>
    <h1>{{ $post->title }}</h1>

    <p>{{ $post->content }}</p>

    <button wire:click="$refresh">Refresh post</button>
</div>
```

這是整個組件樹在初始頁面加載時可能看起來的 HTML：

```html
<div wire:id="123" wire:snapshot="...">
    Post Limit: <input type="number" wire:model.live="postLimit">

    <div wire:id="456" wire:snapshot="...">
        <h1>The first post</h1>

        <p>Post content</p>

        <button wire:click="$refresh">Refresh post</button>
    </div>

    <div wire:id="789" wire:snapshot="...">
        <h1>The second post</h1>

        <p>Post content</p>

        <button wire:click="$refresh">Refresh post</button>
    </div>
</div>
```

請注意，父組件包含其呈現模板以及其內部所有嵌套組件的呈現模板。

由於每個組件都是獨立的，它們各自在其 HTML 中嵌入了自己的 ID 和快照（`wire:id` 和 `wire:snapshot`），供 Livewire 的 JavaScript 核心提取和跟踪。

讓我們考慮一些不同的更新情境，以查看 Livewire 如何處理不同層次的嵌套的差異。

### 更新子組件

如果您在其中一個子 `show-post` 組件中點擊 "刷新文章" 按鈕，將發送以下內容到伺服器：

```js
{
    memo: { name: 'show-post', id: '456' },

    state: { ... },
}
```

這是將被發送回的 HTML：

```html
<div wire:id="456">
    <h1>The first post</h1>

    <p>Post content</p>

    <button wire:click="$refresh">Refresh post</button>
</div>
```

這裡需要注意的重要一點是，當在子組件上觸發更新時，只會將該組件的數據發送到伺服器，並且只會重新呈現該組件。

現在讓我們看一下不太直觀的情況：更新父組件。

### 更新父元件

作為提醒，這裡是父元件 `Posts` 的 Blade 模板：

```blade
<div>
    Post Limit: <input type="number" wire:model.live="postLimit">

    @foreach ($posts as $post)
        <livewire:show-post :$post :key="$post->id">
    @endforeach
</div>
```

如果使用者將 "文章限制" 的值從 `2` 更改為 `1`，則只會觸發父元件的更新。

以下是請求有效載荷可能的範例：

```js
{
    updates: { postLimit: 1 },

    snapshot: {
        memo: { name: 'posts', id: '123' },

        state: { postLimit: 2, ... },
    },
}
```

正如您所看到的，只有父元件 `Posts` 的快照被發送到伺服器。

您可能會問自己一個重要的問題：當父元件重新渲染並遇到子元件 `show-post` 時會發生什麼？如果它們的快照沒有包含在請求有效載荷中，它們將如何重新渲染子元件？

答案是：它們不會重新渲染。

當 Livewire 渲染 `Posts` 元件時，它將為遇到的任何子元件渲染佔位符。

以下是在上述更新後 `Posts` 元件的渲染 HTML 的範例：

```html
<div wire:id="123">
    Post Limit: <input type="number" wire:model.live="postLimit">

    <div wire:id="456"></div>
</div>
```

正如您所看到的，只有一個子元件被渲染，因為 `postLimit` 已更新為 `1`。但是，您還將注意到，與完整子元件不同，只有一個空的 `<div></div>` 與匹配的 `wire:id` 屬性。

當此 HTML 在前端接收時，Livewire 將會將此元件的舊 HTML _變形_ 為此新 HTML，但智能地跳過任何子元件的佔位符。

效果是，在 _變形_ 後，父元件 `Posts` 的最終 DOM 內容將如下：

```html
<div wire:id="123">
    Post Limit: <input type="number" wire:model.live="postLimit">

    <div wire:id="456">
        <h1>The first post</h1>

        <p>Post content</p>

        <button wire:click="$refresh">Refresh post</button>
    </div>
</div>
```

## 效能影響

Livewire 的 "島嶼" 架構對您的應用程式可能會有正面和負面的影響。

這種架構的優勢是它允許您隔離應用程式中昂貴的部分。例如，您可以將一個較慢的資料庫查詢隔離到其自己獨立的元件中，其性能開銷不會影響頁面的其餘部分。

然而，這種方法的最大缺點是，因為元件完全獨立，元件之間的通訊/依賴關係變得更加困難。

例如，如果您從上面的父級 `Posts` 組件傳遞了一個屬性到嵌套的 `ShowPost` 組件，它將不是 "反應式" 的。因為每個組件都是一個獨立的單元，如果對父組件的請求更改了傳遞給 `ShowPost` 的屬性的值，它將不會在 `ShowPost` 內部更新。

Livewire 已經克服了許多這些障礙，並為這些情況提供了專用的 API，如：[反應式屬性](/docs/nesting#reactive-props)、[可建模組件](/docs/nesting#binding-to-child-data-using-wiremodel) 和 [$parent 對象](/docs/nesting#directly-accessing-the-parent-from-the-child)。

擁有這些關於嵌套 Livewire 組件如何運作的知識，您將能夠更明智地決定何時以及如何在應用程序中嵌套組件。
