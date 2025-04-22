`wire:current` 指示詞允許您輕鬆地檢測並為頁面上當前活動的連結添加樣式。

以下是將 `wire:current` 添加到導覽列中的連結的簡單示例，以便當前活動的連結具有更強的字體加粗效果：

```blade
<nav>
    <a href="/dashboard" ... wire:current="font-bold text-zinc-800">Dashboard</a>
    <a href="/posts" ... wire:current="font-bold text-zinc-800">Posts</a>
    <a href="/users" ... wire:current="font-bold text-zinc-800">Users</a>
</nav>
```

現在，當用戶訪問 `/posts` 時，“Posts” 連結將比其他連結具有更強的字體樣式。

您應該注意，`wire:current` 可與 `wire:navigate` 連結和頁面更改直接配合使用。

## 精確匹配

默認情況下，`wire:current` 使用部分匹配策略，這意味著如果連結和當前頁面共享 Url 路徑的開頭部分，則將應用它。

例如，如果連結是 `/posts`，當前頁面是 `/posts/1`，則將應用 `wire:current` 指示詞。

如果您希望使用精確匹配，可以將 `.exact` 修飾符添加到指示詞中。

以下是一個示例，您可能希望使用精確匹配來防止當用戶訪問 `/posts` 時突出顯示“Dashboard” 連結：

```blade
<nav>
    <a href="/" wire:current.exact="font-bold">Dashboard</a>
</nav>
```

## 嚴格匹配

默認情況下，`wire:current` 將從其比較中刪除尾隨斜杠（`/`）。

如果您希望禁用此行為並強制進行嚴格的路徑字符串比較，可以附加 `.strict` 修飾符：

```blade
<nav>
    <a href="/posts/" wire:current.strict="font-bold">Dashboard</a>
</nav>
```

## 故障排除

如果 `wire:current` 未正確檢測當前連結，請確保以下事項：

* 頁面上至少有一個 Livewire 元件，或在您的佈局中硬編碼了 `@livewireScripts`
* 連結上有一個 `href` 屬性。
