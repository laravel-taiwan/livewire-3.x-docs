`wire:text` 是一個指示詞，根據元件屬性或表達式動態更新元素的文字內容。與使用 Blade 的 `{{ }}` 語法不同，`wire:text` 在不需要網路往返重新渲染元件的情況下更新內容。

如果您熟悉 Alpine 的 `x-text` 指示詞，這兩者本質上是相同的。

## 基本用法

這裡是使用 `wire:text` 的範例，樂觀地顯示 Livewire 屬性的更新，而不需要等待網路往返。

```php
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public $likes;

    public function mount()
    {
        $this->likes = $this->post->like_count;
    }

    public function like()
    {
        $this->post->like();

        $this->likes = $this->post->fresh()->like_count;
    }
}
```

```blade
<div>
    <button x-on:click="$wire.likes++" wire:click="like">❤️ Like</button>

    Likes: <span wire:text="likes"></span>
</div>
```

當按下按鈕時，`$wire.likes++` 會立即透過 `wire:text` 更新顯示的計數，同時 `wire:click="like"` 會在背景中將變更持久化到資料庫。

這種模式使得在 Livewire 中建立樂觀 UI 的 `wire:text` 非常適用。
