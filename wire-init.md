Livewire 提供了 `wire:init` 指示詞，以便在元件渲染時立即執行動作。這在您不想阻礙整個頁面加載，但又想在頁面加載後立即加載一些數據時非常有用。

```blade
<div wire:init="loadPosts">
    <!-- ... -->
</div>
```

`loadPosts` 動作將在 Livewire 元件在頁面上渲染後立即運行。

然而，在大多數情況下，[Livewire 的延遲加載功能](/docs/lazy)比使用 `wire:init` 更可取。
