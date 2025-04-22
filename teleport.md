Livewire 允許您將模板的一部分完全傳送到頁面上 DOM 的另一部分。

這對於像巢狀對話框之類的事情很有用。當將一個對話框嵌套在另一個對話框內時，父對話框的 z-index 會應用於嵌套對話框。這可能會導致樣式背景和覆蓋物的問題。為了避免這個問題，您可以使用 Livewire 的 `@teleport` 指令將每個嵌套對話框呈現為 DOM 中的同級元素。

此功能由 [Alpine 的 `x-teleport` 指令](https://alpinejs.dev/directives/teleport) 提供支援。

## 基本用法

要將模板的一部分傳送到 DOM 的另一部分，您可以將其包裹在 Livewire 的 `@teleport` 指令中。

以下是使用 `@teleport` 將模態對話框的內容呈現在頁面上 `<body>` 元素末尾的示例：

```blade
<div>
    <!-- Modal -->
    <div x-data="{ open: false }">
        <button @click="open = ! open">Toggle Modal</button>

        @teleport('body')
            <div x-show="open">
                Modal contents...
            </div>
        @endteleport
    </div>
</div>
```

> [!info]
> `@teleport` 選擇器可以是您通常會傳遞給 `document.querySelector()` 之類的任何字符串。
>
> 您可以通過查閱其 [MDN 文件](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) 來了解有關 `document.querySelector()` 的更多信息。

現在，當上述 Livewire 模板在頁面上呈現時，模態的 _內容_ 部分將呈現在 `<body>` 的末尾：

```html
<body>
    <!-- ... -->

    <div x-show="open">
        Modal contents...
    </div>
</body>
```

> [!warning] 您必須在組件外部進行傳送
> Livewire 僅支持將 HTML 傳送到組件外部。例如，將模態傳送到 `<body>` 標記是可以的，但將其傳送到組件內的另一個元素將無法正常工作。

> [!warning] 傳送只能與單個根元素一起使用
> 請確保在 `@teleport` 語句中僅包含單個根元素。
