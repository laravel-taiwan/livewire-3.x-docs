當 Livewire 元件更新瀏覽器的 DOM 時，它以一種智能的方式進行，我們稱之為 "morphing"。_morph_ 這個術語與 _replace_ 這樣的詞相對。

Livewire 不是每次更新元件時都將元件的 HTML 替換為新渲染的 HTML，而是動態比較當前的 HTML 與新的 HTML，識別差異，並僅在需要更改的地方對 HTML 進行手術性更改。

這樣做的好處是保留元件上現有的未更改元素。例如，事件監聽器、焦點狀態和表單輸入值在 Livewire 更新之間都得以保留。當然，與在每次更新時清除並重新渲染新的 DOM 相比，morphing 也提供了更高的性能。

## morphing 的運作方式

要了解 Livewire 如何在 Livewire 請求之間決定要更新哪些元素，請考慮這個簡單的 `Todos` 元件：

```php
class Todos extends Component
{
    public $todo = '';

    public $todos = [
        'first',
        'second',
    ];

    public function add()
    {
        $this->todos[] = $this->todo;
    }
}
```

```blade
<form wire:submit="add">
    <ul>
        @foreach ($todos as $item)
            <li>{{ $item }}</li>
        @endforeach
    </ul>

    <input wire:model="todo">
</form>
```

該元件的初始渲染將輸出以下 HTML：

```html
<form wire:submit="add">
    <ul>
        <li>first</li>

        <li>second</li>
    </ul>

    <input wire:model="todo">
</form>
```

現在，想像你在輸入欄位中輸入了 "third"，並按下 `[Enter]` 鍵。新渲染的 HTML 將是：

```html
<form wire:submit="add">
    <ul>
        <li>first</li>

        <li>second</li>

        <li>third</li> <!-- [tl! add] -->
    </ul>

    <input wire:model="todo">
</form>
```

當 Livewire 處理元件更新時，它將原始 DOM 轉換為新渲染的 HTML。以下視覺化應該直觀地讓您了解它的運作方式：

<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/844600772?badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="morph_basic"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

正如您所見，Livewire 同時遍歷兩個 HTML 樹。當它在兩個樹中遇到每個元素時，它會比較它們的變化、添加和刪除。如果檢測到變化，它將進行適當的手術性更改。

## 變形的缺陷

以下是變形演算法無法正確識別 HTML 樹中變化的情況，因此導致應用程式出現問題的情況。

### 插入中間元素

考慮以下 Livewire Blade 模板，用於虛構的 `CreatePost` 元件：

```blade
<form wire:submit="save">
    <div>
        <input wire:model="title">
    </div>

    @if ($errors->has('title'))
        <div>{{ $errors->first('title') }}</div>
    @endif

    <div>
        <button>Save</button>
    </div>
</form>
```

如果使用者嘗試提交表單，但遇到驗證錯誤，將會出現以下問題：

<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/844600840?badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="morph_problem"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

如您所見，當 Livewire 遇到新的 `<div>` 以顯示錯誤訊息時，它無法確定是要直接更改現有的 `<div>`，還是在中間插入新的 `<div>`。

再次強調正在發生的事情：

* Livewire 在兩個樹中都遇到第一個 `<div>`。它們是相同的，因此繼續進行。
* Livewire 在兩個樹中遇到第二個 `<div>`，認為它們是相同的 `<div>`，只是其中一個內容已更改。因此，Livewire 不是將錯誤訊息插入為新元素，而是將 `<button>` 更改為錯誤訊息。
* Livewire 錯誤地修改了先前的元素後，注意到比較結尾處有額外的元素。然後在先前元素之後創建並附加該元素。
* 因此，破壞了應該簡單移動的元素，然後重新創建。

這種情況幾乎是所有與變形相關錯誤的根源。

以下是這些錯誤的一些具體問題影響：
* 更新之間丟失事件監聽器和元素狀態
* 事件監聽器和狀態錯置在錯誤的元素之間
* 整個 Livewire 元件可能會被重置或複製，因為 Livewire 元件本身也只是 DOM 樹中的元素
* Alpine 元件和狀態可能會丟失或錯置

### 內部向前查看

Livewire 在其變形算法中有一個額外步驟，會在更改元素之前檢查後續元素及其內容。

這可以在許多情況下防止上述情況發生。

以下是「向前查看」算法的視覺化示例：

<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/844600800?badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="morph_lookahead"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

### 注入變形標記

在後端，Livewire 自動檢測 Blade 模板中的條件語句，並將它們包裹在 HTML 註釋標記中，Livewire 的 JavaScript 可以在變形時使用這些標記作為指引。

以下是先前的 Blade 模板，但帶有 Livewire 注入標記的示例：

```blade
<form wire:submit="save">
    <div>
        <input wire:model="title">
    </div>

    <!--[if BLOCK]><![endif]--> <!-- [tl! highlight] -->
    @if ($errors->has('title'))
        <div>Error: {{ $errors->first('title') }}</div>
    @endif
    <!--[if ENDBLOCK]><![endif]--> <!-- [tl! highlight] -->

    <div>
        <button>Save</button>
    </div>
</form>
```

有了這些標記注入到模板中，Livewire 現在可以更容易地檢測更改和添加之間的差異。

這個功能對於 Livewire 應用程序非常有益，但因為它需要通過正則表達式解析模板，有時可能無法正確檢測條件語句。如果這個功能對您的應用程序更多是一個阻礙而不是幫助，您可以在應用程序的 `config/livewire.php` 文件中使用以下配置來禁用它：

```php
'inject_morph_markers' => false,
```

#### 包裹條件語句

如果上述兩種解決方案無法解決您的情況，避免變形問題最可靠的方法是將條件語句和循環包裹在始終存在的自己的元素中。

例如，以下是使用包裹 `<div>` 元素重新編寫的上述 Blade 模板：

```blade
<form wire:submit="save">
    <div>
        <input wire:model="title">
    </div>

    <div> <!-- [tl! highlight] -->
        @if ($errors->has('title'))
            <div>{{ $errors->first('title') }}</div>
        @endif
    </div> <!-- [tl! highlight] -->

    <div>
        <button>Save</button>
    </div>
</form>
```

現在條件語句已經包裹在一個持久元素中，Livewire 將正確地變形這兩個不同的 HTML 樹。

#### 繞過變形

如果您需要完全繞過對元素的變形，您可以使用 [wire:replace](/docs/wire-replace) 指示 Livewire 替換元素的所有子元素，而不是嘗試變形現有元素。
