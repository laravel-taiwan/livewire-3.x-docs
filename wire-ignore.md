Livewire 有能力更新頁面，這使它變得「即時」，但有時您可能希望阻止 Livewire 更新頁面的某部分。

在這些情況下，您可以使用 `wire:ignore` 指示詞，指示 Livewire 忽略特定元素的內容，即使在請求之間發生變化。

這在與第三方 JavaScript 函式庫一起使用自定義表單輸入等情況下非常有用。

以下是將第三方庫使用的元素包裹在 `wire:ignore` 中的示例，以便 Livewire 不會干擾該庫生成的 HTML：

```blade
<form>
    <!-- ... -->

    <div wire:ignore>
        <!-- This element would be reference by a -->
        <!-- third-party library for initialization... -->
        <input id="id-for-date-picker-library">
    </div>

    <!-- ... -->
</form>
```

您還可以通過使用 `wire:ignore.self` 指示 Livewire 只忽略根元素屬性的變化，而不是觀察其內容的變化。

```blade
<div wire:ignore.self>
    <!-- ... -->
</div>
```
