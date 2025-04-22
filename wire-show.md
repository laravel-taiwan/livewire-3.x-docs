Livewire 的 `wire:show` 指示詞讓根據表達式結果顯示和隱藏元素變得容易。

`wire:show` 指示詞與在 Blade 中使用 `@if` 不同，它使用 CSS（`display: none`）來切換元素的可見性，而不是從 DOM 完全移除元素。這意味著元素仍然存在於頁面中，但被隱藏，可以實現更流暢的過渡，而無需進行伺服器往返。

## 基本用法

這裡是使用 `wire:show` 切換 "建立文章" 對話框的實際範例：

```php
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $showModal = false;

    public $content = '';

    public function save()
    {
        Post::create(['content' => $this->content]);

        $this->reset('content');

        $this->showModal = false;
    }
}
```

```blade
<div>
    <button x-on:click="$wire.showModal = true">New Post</button>

    <div wire:show="showModal">
        <form wire:submit="save">
            <textarea wire:model="content"></textarea>

            <button type="submit">Save Post</button>
        </form>
    </div>
</div>
```

當點擊 "建立新文章" 按鈕時，對話框會出現，而無需進行伺服器往返。成功保存文章後，對話框將被隱藏，並重置表單。

## 使用過渡效果

您可以將 `wire:show` 與 Alpine.js 過渡結合，創建平滑的顯示/隱藏動畫。由於 `wire:show` 只切換 CSS 的 `display` 屬性，Alpine 的 `x-transition` 指示詞與之完美配合：

```blade
<div>
    <button x-on:click="$wire.showModal = true">New Post</button>

    <div wire:show="showModal" x-transition.duration.500ms>
        <form wire:submit="save">
            <textarea wire:model="content"></textarea>
            <button type="submit">Save Post</button>
        </form>
    </div>
</div>
```

上述 Alpine.js 過渡類別將在對話框顯示和隱藏時創建淡入淡出和縮放效果。

[查看完整 x-transition 文件 →](https://alpinejs.dev/directives/transition)
