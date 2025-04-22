Livewire 的 DOM 差異比對對於更新頁面上現有元素非常有用，但偶爾您可能需要強制某些元素從頭重新渲染以重置內部狀態。

在這些情況下，您可以使用 `wire:replace` 指示詞指示 Livewire 跳過對元素的子元素進行 DOM 差異比對，而是完全用來自伺服器的新元素替換內容。

這在與第三方 JavaScript 函式庫和自定義 Web 元件一起工作時特別有用，或者在保持狀態時元素重複使用可能會導致問題時。

以下是將 Web 元件包裹在陰影 DOM 中的 `wire:replace` 的示例，以便 Livewire 完全替換元素，使自定義元素能夠處理自己的生命週期：

```blade
<form>
    <!-- ... -->

    <div wire:replace>
        <!-- This custom element would have its own internal state -->
        <json-viewer>@json($someProperty)</json-viewer>
    </div>

    <!-- ... -->
</form>
```

您還可以指示 Livewire 用 `wire:replace.self` 替換目標元素以及所有子元素。

```blade
<div x-data="{open: false}" wire:replace.self>
  <!-- 確保在每次渲染時 "open" 狀態被重置為 false -->
</div>
```
