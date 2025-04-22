## 基本用法

在 Livewire 中顯示或隱藏內容就像使用 Blade 的條件指示詞之一 `@if` 一樣簡單。為了增強用戶的體驗，Livewire 提供了一個 `wire:transition` 指示詞，允許您平滑地在頁面中切換條件元素。

例如，下面是一個 `ShowPost` 元件，具有切換查看評論的功能：

```php
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public $showComments = false;
}
```

```blade
<div>
    <!-- ... -->

    <button wire:click="$set('showComments', true)">Show comments</button>

    @if ($showComments)
        <div wire:transition> <!-- [tl! highlight] -->
            @foreach ($post->comments as $comment)
                <!-- ... -->
            @endforeach
        </div>
    @endif
</div>
```

因為 `wire:transition` 已添加到包含文章評論的 `<div>` 中，當按下 "顯示評論" 按鈕時，`$showComments` 將設置為 `true`，評論將以 "淡入" 到頁面上，而不是突然出現。

## 限制

目前，`wire:transition` 僅支持在 Blade 條件語句中的單個元素上。當在一組同級元素中使用時，它將無法正常工作。例如，以下情況將無法正常工作：

```blade
<!-- Warning: The following is code that will not work properly -->
<ul>
    @foreach ($post->comments as $comment)
        <li wire:transition wire:key="{{ $comment->id }}">{{ $comment->content }}</li>
    @endforeach
</ul>
```

如果上述評論 `<li>` 元素中的一個被移除，您期望 Livewire 將其過渡移出。但是，由於 Livewire 底層 "morph" 機制的障礙，這將不會發生。目前沒有辦法使用 `wire:transition` 在 Livewire 中過渡動態列表。

## 預設過渡樣式

預設情況下，Livewire 對具有 `wire:transition` 的元素應用了不透明度和縮放的 CSS 過渡。這裡是一個視覺預覽：

<div x-data="{ show: false }" x-cloak class="border border-gray-700 rounded-xl p-6 w-full flex justify-between">
    <a href="#" x-on:click.prevent="show = ! show" class="py-2.5 outline-none">
        預覽過渡 <span x-text="show ? '淡出' : '淡入 →'">淡入</span>
    </a>
    <div class="hey">
        <div
            x-show="show"
            x-transition
            class="inline-flex px-16 py-2.5 rounded-[10px] bg-pink-400 text-white uppercase font-medium transition focus-visible:outline-none focus-visible:!ring-1 focus-visible:!ring-white"
            style="
                background: linear-gradient(109.48deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.1) 100%), #EE5D99;
                box-shadow: inset 0px -1px 0px rgba(0, 0, 0, 0.5), inset 0px 1px 0px rgba(255, 255, 255, 0.1);
            "
        >
            &nbsp;
        </div>
    </div>
</div>

上述轉換使用以下預設值進行過渡：

進入過渡 | 離開過渡
--- | ---
`持續時間：150毫秒` | `持續時間：75毫秒`
`不透明度：[0 - 100]` | `不透明度：[100 - 0]`
`變換：縮放([0.95 - 1])` | `變換：縮放([1 - 0.95])`

## 自訂過渡

要自訂 CSS Livewire 在過渡時內部使用的樣式，您可以使用任何可用修飾符的組合：

修飾符 | 說明
--- | ---
`.in` | 僅在元素「進入」時過渡
`.out` | 僅在元素「離開」時過渡
`.duration.[?]ms` | 自訂過渡持續時間（毫秒）
`.duration.[?]s` | 自訂過渡持續時間（秒）
`.delay.[?]ms` | 自訂過渡延遲時間（毫秒）
`.delay.[?]s` | 自訂過渡延遲時間（秒）
`.opacity` | 僅應用不透明度過渡
`.scale` | 僅應用縮放過渡
`.origin.[top\|bottom\|left\|right]` | 自訂使用的縮放「原點」

以下是各種過渡組合的清單，可能有助於更好地視覺化這些自訂：

**僅淡出過渡**

預設情況下，Livewire 在過渡時同時淡出和縮放元素。您可以通過添加`.opacity`修飾符來禁用縮放並僅進行淡出。這對於像是過渡全頁疊層之類的情況很有用，其中添加縮放並不合適。

```html
<div wire:transition.opacity>
```

<div x-data="{ show: false }" x-cloak class="border border-gray-700 rounded-xl p-6 w-full flex justify-between">
    <a href="#" x-on:click.prevent="show = ! show" class="py-2.5 outline-none">
        預覽過渡 <span x-text="show ? '離開' : '進入 →'">進入</span>
    </a>
    <div class="hey">
        <div
            x-show="show"
            x-transition.opacity
            class="inline-flex px-16 py-2.5 rounded-[10px] bg-pink-400 text-white uppercase font-medium transition focus-visible:outline-none focus-visible:!ring-1 focus-visible:!ring-white"
            style="
                background: linear-gradient(109.48deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.1) 100%), #EE5D99;
                box-shadow: inset 0px -1px 0px rgba(0, 0, 0, 0.5), inset 0px 1px 0px rgba(255, 255, 255, 0.1);
            "
        >
            ...
        </div>
    </div>
</div>

**淡出過渡**

一個常見的過渡技術是在過渡進入時立即顯示元素，並在過渡離開時淡化其不透明度。您會注意到這種效果在大多數原生 MacOS 下拉菜單和選單上。因此，它通常應用於網頁上的下拉菜單、彈出窗口和選單。

```html
<div wire:transition.out.opacity.duration.200ms>
```

<div x-data="{ show: false }" x-cloak class="border border-gray-700 rounded-xl p-6 w-full flex justify-between">
    <a href="#" x-on:click.prevent="show = ! show" class="py-2.5 outline-none">
        預覽過渡 <span x-text="show ? '離開' : '進入 →'">進入</span>
    </a>
    <div class="hey">
        <div
            x-show="show"
            x-transition.out.opacity.duration.200ms
            class="inline-flex px-16 py-2.5 rounded-[10px] bg-pink-400 text-white uppercase font-medium transition focus-visible:outline-none focus-visible:!ring-1 focus-visible:!ring-white"
            style="
                background: linear-gradient(109.48deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.1) 100%), #EE5D99;
                box-shadow: inset 0px -1px 0px rgba(0, 0, 0, 0.5), inset 0px 1px 0px rgba(255, 255, 255, 0.1);
            "
        >
            ...
        </div>
    </div>
</div>

**頂部起源過渡**

使用 Livewire 來過渡一個元素，例如下拉菜單時，從菜單頂部作為起源進行縮放，而不是中心（Livewire 的默認）。這樣，菜單在視覺上感覺與觸發它的元素相連。

```html
<div wire:transition.scale.origin.top>
```

<div x-data="{ show: false }" x-cloak class="border border-gray-700 rounded-xl p-6 w-full flex justify-between">
    <a href="#" x-on:click.prevent="show = ! show" class="py-2.5 outline-none">
        預覽過渡 <span x-text="show ? '離開' : '進入 →'">進入</span>
    </a>
    <div class="hey">
        <div
            x-show="show"
            x-transition.origin.top
            class="inline-flex px-16 py-2.5 rounded-[10px] bg-pink-400 text-white uppercase font-medium transition focus-visible:outline-none focus-visible:!ring-1 focus-visible:!ring-white"
            style="
                background: linear-gradient(109.48deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.1) 100%), #EE5D99;
                box-shadow: inset 0px -1px 0px rgba(0, 0, 0, 0.5), inset 0px 1px 0px rgba(255, 255, 255, 0.1);
            "
        >
            ...
        </div>
    </div>
</div>

> [!tip] Livewire 在幕後使用 Alpine 轉場效果
> 當在元素上使用 `wire:transition` 時，Livewire 內部會應用 Alpine 的 `x-transition` 指示詞。因此，您可以使用大多數甚至所有您通常會與 `x-transition` 一起使用的語法。查看 [Alpine 的轉場文件](https://alpinejs.dev/directives/transition) 以了解其所有功能。 


