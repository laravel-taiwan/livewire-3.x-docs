在 Livewire 中執行危險操作之前，您可能希望為用戶提供某種視覺確認。

Livewire 通過在任何操作（`wire:click`、`wire:submit` 等）中添加 `wire:confirm` 來輕鬆實現這一點。

以下是將確認對話框添加到「刪除帖子」按鈕的示例：

```blade
<button
    type="button"
    wire:click="delete"
    wire:confirm="Are you sure you want to delete this post?"
>
    Delete post <!-- [tl! highlight:-2,1] -->
</button>
```

當用戶點擊「刪除帖子」時，Livewire 將觸發一個確認對話框（默認瀏覽器確認警報）。如果用戶按下 Esc 鍵或按下取消，則不會執行該操作。如果他們按下「確定」，則操作將完成。

## 提示用戶輸入

對於更危險的操作，例如完全刪除用戶帳戶，您可能希望向他們提供一個確認提示，他們需要輸入特定的字符串來確認操作。

Livewire 提供了一個有用的 `.prompt` 修飾符，當應用於 `wire:confirm` 時，它將提示用戶輸入，並僅在輸入匹配（區分大小寫）所提供的字符串時（在 `wire:confirm` 值的末尾由 "|"（管道）字符指定）確認操作：

```blade
<button
    type="button"
    wire:click="delete"
    wire:confirm.prompt="Are you sure?\n\nType DELETE to confirm|DELETE"
>
    Delete account <!-- [tl! highlight:-2,1] -->
</button>
```

當用戶按下「刪除帳戶」時，只有在輸入「DELETE」到提示中時，操作才會執行，否則將取消該操作。
