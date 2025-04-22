`wire:cloak` 是一個指示詞，在頁面加載時隱藏元素，直到 Livewire 完全初始化。這對於防止頁面在 Livewire 有機會初始化之前加載而出現的「未樣式化內容閃現」非常有用。

## 基本用法

要使用 `wire:cloak`，將指示詞添加到您希望在頁面加載期間隱藏的任何元素中：

```blade
<div wire:cloak>
    這個內容將在 Livewire 完全加載之前隱藏
</div>
```

### 動態內容

`wire:cloak` 在您希望防止用戶看到未初始化的動態內容時特別有用，例如使用 `wire:show` 顯示或隱藏的元素。

```blade
<div>
    <div wire:show="starred" wire:cloak>
        <!-- Yellow star icon... -->
    </div>

    <div wire:show="!starred" wire:cloak>
        <!-- Gray star icon... -->
    </div>
</div>
```

在上面的示例中，如果沒有 `wire:cloak`，在 Livewire 初始化之前兩個圖示都會顯示。但是，有了 `wire:cloak`，這兩個元素將在初始化之前被隱藏。
