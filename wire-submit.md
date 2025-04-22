Livewire 使通過 `wire:submit` 指示詞輕鬆處理表單提交。通過將 `wire:submit` 添加到 `<form>` 元素，Livewire 將攔截表單提交，阻止默認的瀏覽器處理，並調用任何 Livewire 組件方法。

這是使用 `wire:submit` 處理 "建立文章" 表單提交的基本示例：

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

        $this->redirect('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```blade
<form wire:submit="save"> <!-- [tl! highlight] -->
    <input type="text" wire:model="title">

    <textarea wire:model="content"></textarea>

    <button type="submit">Save</button>
</form>
```

在上面的示例中，當用戶通過點擊 "儲存" 提交表單時，`wire:submit` 會攔截 `submit` 事件並在伺服器上調用 `save()` 動作。

> [!info] Livewire 自動調用 `preventDefault()`
> `wire:submit` 與其他 Livewire 事件處理程序不同，它在內部調用 `event.preventDefault()` 而無需使用 `.prevent` 修飾符。這是因為很少有情況下您會監聽 `submit` 事件並且不希望阻止其默認的瀏覽器處理（執行對端點的完整表單提交）。

> [!info] Livewire 在提交時自動禁用表單
> 默認情況下，當 Livewire 將表單提交到伺服器時，它將禁用表單提交按鈕並將所有表單輸入標記為 `readonly`。這樣一來，用戶在初始提交完成之前無法再次提交相同的表單。

## 深入了解

`wire:submit` 只是 Livewire 提供的許多事件監聽器之一。以下兩個頁面提供了更完整的文件，介紹如何在應用程式中使用 `wire:submit`：

* [使用 Livewire 回應瀏覽器事件](/docs/actions)
* [在 Livewire 中創建表單](/docs/forms)
