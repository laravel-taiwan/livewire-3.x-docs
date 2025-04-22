在某些情況下，讓您的使用者知道他們目前是否連接到互聯網可能會有所幫助。

例如，如果您在 Livewire 上建立了一個部落格平台，您可能希望以某種方式通知使用者，如果他們離線，以免他們在沒有 Livewire 將其保存到資料庫的能力下起草整篇部落格文章。

Livewire 通過提供 `wire:offline` 指示詞來輕鬆實現這一點。通過將 `wire:offline` 附加到 Livewire 元件中的元素，它將默認隱藏，並且只有在 Livewire 檢測到網絡連接已中斷且不可用時才會顯示。當網絡重新連接時，它將再次消失。

例如：

```blade
<p class="alert alert-warning" wire:offline>
    糟糕，您的設備已斷開連接。您正在查看的網頁處於離線狀態。
</p>
```
