Livewire 屬性可以在前端和後端自由修改，使用 `wire:model` 等工具。如果要防止屬性（例如模型 ID）在前端被修改，可以使用 Livewire 的 `#[Locked]` 屬性。

## 基本用法

以下是一個 `ShowPost` 元件，將 `Post` 模型的 ID 存儲為名為 `$id` 的公共屬性。為了防止好奇或惡意用戶修改此屬性，可以將 `#[Locked]` 屬性添加到該屬性：

> [!warning] 確保導入屬性類
> 確保導入任何屬性類。例如，下面的 `#[Locked]` 屬性需要以下導入 `use Livewire\Attributes\Locked;`。

```php
use Livewire\Attributes\Locked;
use Livewire\Component;

class ShowPost extends Component
{
	#[Locked] // [tl! highlight]
    public $id;

    public function mount($postId)
    {
        $this->id = $postId;
    }

	// ...
}
```

通過添加 `#[Locked]` 屬性，確保 `$id` 屬性不會被篡改。

> [!tip] 模型屬性默認安全
> 如果將 Eloquent 模型存儲在公共屬性中，而不僅僅是模型的 ID，Livewire 將確保 ID 不會被篡改，而無需明確將 `#[Locked]` 屬性添加到該屬性。對於大多數情況，這比使用 `#[Locked]` 更好：
> ```php
> class ShowPost extends Component
> {
>    public Post $post; // [tl! highlight]
>
>    public function mount($postId)
>    {
>        $this->post = Post::find($postId);
>    }
>
>	// ...
>}
> ```

### 為什麼不使用受保護屬性？

您可能會問：為什麼不只使用受保護屬性來存儲敏感數據？

請記住，Livewire 只在網絡請求之間保留公共屬性。對於靜態、硬編碼數據，受保護屬性是合適的。但是，對於運行時存儲的數據，您必須使用公共屬性來確保數據正確保留。

### Livewire 不能自動執行此操作嗎？

在理想的情況下，Livewire 應該默認鎖定屬性，只有在 `wire:model` 用於該屬性時才允許修改。

不幸的是，這將要求 Livewire 解析所有 Blade 模板，以了解屬性是否被 `wire:model` 或類似 API 修改。 

--- 

*本翻譯遵循指定的翻譯規則，如有任何問題，請隨時告知。*

不僅會增加技術和性能開銷，而且將無法檢測屬性是否被像 Alpine 或任何其他自定義 JavaScript 所變異。

因此，Livewire 將繼續使公共屬性默認可自由變異，並為開發人員提供必要的工具來鎖定它們。
