在即時應用程式中，提供視覺指示來顯示使用者的裝置已經斷開網路連線可能會很有幫助。

Livewire 提供了 `wire:offline` 指示詞來處理這種情況。

將 `wire:offline` 加入到 Livewire 元件內的元素中，該元素將預設隱藏，並在使用者失去連線時顯示出來：

```blade
<div wire:offline>
    這個裝置目前已離線。
</div>
```

## 切換類別

使用 `class` 修飾符可以讓您在使用者失去連線時將類別添加到元素中。一旦使用者重新連線，該類別將被移除：

```blade
<div wire:offline.class="bg-red-300">
```

或者，使用 `.remove` 修飾符，您可以在使用者失去連線時移除一個類別。在這個例子中，當使用者失去連線時，`bg-green-300` 類別將從 `<div>` 中移除：

```blade
<div class="bg-green-300" wire:offline.class.remove="bg-green-300">
```

## 切換屬性

`.attr` 修飾符允許您在使用者失去連線時將屬性添加到元素中。在這個例子中，當使用者失去連線時，"儲存" 按鈕將被禁用：

```blade
<button wire:offline.attr="disabled">儲存</button>
```
