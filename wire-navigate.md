Livewire 的 `wire:navigate` 功能使頁面導航更快速，為您的用戶提供類似 SPA 的體驗。

這個頁面是 `wire:navigate` 指示詞的簡單參考。請務必閱讀 [Livewire 的 Navigate 功能頁面](/docs/navigate) 以獲得更完整的文件。

以下是在導覽列中添加 `wire:navigate` 到連結的簡單示例：

```blade
<nav>
    <a href="/" wire:navigate>Dashboard</a>
    <a href="/posts" wire:navigate>Posts</a>
    <a href="/users" wire:navigate>Users</a>
</nav>
```

當點擊這些連結中的任何一個時，Livewire 將攔截點擊事件，而不是允許瀏覽器執行完整的頁面訪問，Livewire 將在後台提取頁面並將其與當前頁面交換（從而實現更快速和更流暢的頁面導航）。

## 在懸停時預取頁面

通過添加 `.hover` 修飾符，Livewire 將在用戶懸停在連結上時預取頁面。這樣，當用戶點擊連結時，該頁面已經從伺服器下載完畢。

```blade
<a href="/" wire:navigate.hover>儀表板</a>
```

## 深入了解

欲了解更完整的文件，請訪問 [Livewire 的 navigate 文件頁面](/docs/navigate)。
