Livewire 提供了一個簡單的 `wire:click` 指示詞，用於在使用者點擊頁面上特定元素時調用元件方法（又稱行為）。

例如，給定以下 `ShowInvoice` 元件：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Invoice;

class ShowInvoice extends Component
{
    public Invoice $invoice;

    public function download()
    {
        return response()->download(
            $this->invoice->file_path, 'invoice.pdf'
        );
    }
}
```

您可以通過添加 `wire:click="download"` 來觸發上述類別中的 `download()` 方法，當使用者點擊 "下載發票" 按鈕時：

```html
<button type="button" wire:click="download"> <!-- [tl! highlight] -->
    下載發票
</button>
```

## 在連結上使用 `wire:click`

在 `<a>` 標籤上使用 `wire:click` 時，您必須附加 `.prevent` 以防止瀏覽器默認處理連結。否則，瀏覽器將訪問提供的連結並更新頁面的 URL。

```html
<a href="#" wire:click.prevent="...">
```

## 進一步了解

`wire:click` 指示詞只是 Livewire 中許多可用事件監聽器之一。有關其（以及其他事件監聽器）功能的完整文檔，請訪問 [Livewire 行為文檔頁面](/docs/actions)。
