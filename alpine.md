[AlpineJS](https://alpinejs.dev/) 是一個輕量級的 JavaScript 函式庫，可輕鬆為您的網頁添加客戶端互動功能。它最初是為了配合像 Livewire 這樣的工具而建立的，其中更加偏向 JavaScript 的效用有助於在應用程式中添加互動功能。

Livewire 預設即附帶 Alpine，因此無需將其單獨安裝到您的專案中。

學習如何使用 AlpineJS 的最佳地方是[Alpine 文件](https://alpinejs.dev)。

## 基本的 Alpine 元件

為了為本文件的其餘部分打下基礎，這裡是一個最簡單且資訊豐富的 Alpine 元件範例之一。一個小型的 "計數器"，在頁面上顯示一個數字，並允許使用者通過點擊按鈕來增加該數字：

```html
<!-- Declare a JavaScript object of data... -->
<div x-data="{ count: 0 }">
    <!-- Render the current "count" value inside an element... -->
    <h2 x-text="count"></h2>

    <!-- Increment the "count" value by "1" when a click event is dispatched... -->
    <button x-on:click="count++">+</button>
</div>
```

上面的 Alpine 元件可以毫不費力地在應用程式中的任何 Livewire 元件中使用。Livewire 負責在 Livewire 元件更新時維護 Alpine 的狀態。實質上，您應該可以自由地在 Livewire 中使用 Alpine 元件，就像在任何其他非 Livewire 上下文中使用 Alpine 一樣。

## 在 Livewire 中使用 Alpine

讓我們探索在 Livewire 元件中使用 Alpine 元件的更實際範例。

以下是一個簡單的 Livewire 元件，顯示來自資料庫的文章模型的詳細資訊。默認情況下，僅顯示文章的標題：

```html
<div>
    <h1>{{ $post->title }}</h1>

    <div x-data="{ expanded: false }">
        <button type="button" x-on:click="expanded = ! expanded">
            <span x-show="! expanded">Show post content...</span>
            <span x-show="expanded">Hide post content...</span>
        </button>

        <div x-show="expanded">
            {{ $post->content }}
        </div>
    </div>
</div>
```

通過使用 Alpine，我們可以在使用者按下 "顯示文章內容..." 按鈕之前隱藏文章的內容。在那時，Alpine 的 `expanded` 屬性將設置為 `true`，並且因為使用了 `x-show="expanded"`，Alpine 將控制文章內容的可見性並顯示在頁面上。

這是 Alpine 發揮作用的一個例子：在應用程式中添加互動功能，而不觸發 Livewire 伺服器往返。

## 使用 `$wire` 從 Alpine 控制 Livewire

作為 Livewire 開發者，您可以使用的最強大功能之一是 `$wire`。`$wire` 物件是一個神奇的物件，可用於所有在 Livewire 中使用的 Alpine 元件。

您可以將`$wire`視為從JavaScript進入PHP的門戶。它允許您訪問和修改Livewire元件的屬性，調用Livewire元件的方法，以及更多功能；全部都可以在AlpineJS內部完成。

### 存取Livewire屬性

這裡有一個簡單的「字元計數」實用程式範例，用於創建文章的表單。當用戶輸入時，這將立即顯示用戶文章內容中包含的字元數：

```html
<form wire:submit="save">
    <!-- ... -->

    <input wire:model="content" type="text">

    <small>
        Character count: <span x-text="$wire.content.length"></span> <!-- [tl! highlight] -->
    </small>

    <button type="submit">Save</button>
</form>
```

正如您在上面的範例中所看到的，`x-text`被用來允許Alpine控制`<span>`元素的文字內容。`x-text`接受其中的任何JavaScript表達式，並在任何相依性更新時自動做出反應。因為我們使用`$wire.content`來存取`$content`的值，Alpine將在每次從Livewire更新`$wire.content`時自動更新文字內容；在這種情況下是透過`wire:model="content"`。

### 變更Livewire屬性

這裡有一個在Alpine內部使用`$wire`來清除創建文章表單中「標題」欄位的範例。

```html
<form wire:submit="save">
    <input wire:model="title" type="text">

    <button type="button" x-on:click="$wire.title = ''">Clear</button> <!-- [tl! highlight] -->

    <!-- ... -->

    <button type="submit">Save</button>
</form>
```

當用戶填寫上述Livewire表單時，他們可以按下「清除」按鈕，而不需要從Livewire發送網路請求即可清除標題欄位。這個互動將是「即時的」。

以下是使這個操作發生的簡要說明：

* `x-on:click`告訴Alpine監聽按鈕元素上的點擊事件
* 點擊時，Alpine執行提供的JS表達式：`$wire.title = ''`
* 因為`$wire`是代表Livewire元件的神奇物件，您可以直接從JavaScript訪問或變更您元件的所有屬性
* `$wire.title = ''`將您Livewire元件中的`$title`值設置為空字串
* 任何Livewire工具，如`wire:model`，將立即對此更改做出反應，而無需發送伺服器往返
* 在下一個Livewire網路請求中，後端將更新`$title`屬性為空字串

### 調用Livewire方法

Alpine也可以輕鬆地通過直接在`$wire`上調用任何Livewire方法/操作來執行。

這是使用 Alpine 監聽輸入框上的 "blur" 事件並觸發表單保存的範例。當使用者按下 "Tab" 鍵將焦點從當前元素移開並將焦點移到頁面上的下一個元素時，瀏覽器會觸發 "blur" 事件：

```html
<form wire:submit="save">
    <input wire:model="title" type="text" x-on:blur="$wire.save()">  <!-- [tl! highlight] -->

    <!-- ... -->

    <button type="submit">Save</button>
</form>
```

通常，在這種情況下，您只需使用 `wire:model.blur="title"`，但是，展示如何使用 Alpine 實現這一點對於示範目的很有幫助。

#### 傳遞參數

您也可以通過將參數傳遞給 `$wire` 方法調用來將參數傳遞給 Livewire 方法。

考慮一個具有 `deletePost()` 方法的組件，如下所示：

```php
public function deletePost($postId)
{
    $post = Post::find($postId);

    // Authorize user can delete...
    auth()->user()->can('update', $post);

    $post->delete();
}
```

現在，您可以從 Alpine 將 `$postId` 參數傳遞給 `deletePost()` 方法，如下所示：

```html
<button type="button" x-on:click="$wire.deletePost(1)">
```

通常，像 `$postId` 這樣的東西會在 Blade 中生成。以下是使用 Blade 確定 Alpine 傳遞給 `deletePost()` 的 `$postId` 的示例：

```html
@foreach ($posts as $post)
    <button type="button" x-on:click="$wire.deletePost({{ $post->id }})">
        Delete "{{ $post->title }}"
    </button>
@endforeach
```

如果頁面上有三篇文章，則上面的 Blade 模板將在瀏覽器中呈現為以下內容：

```html
<button type="button" x-on:click="$wire.deletePost(1)">
    Delete "The power of walking"
</button>

<button type="button" x-on:click="$wire.deletePost(2)">
    Delete "How to record a song"
</button>

<button type="button" x-on:click="$wire.deletePost(3)">
    Delete "Teach what you learn"
</button>
```

正如您所見，我們已經使用 Blade 將不同的文章 ID 渲染到 Alpine 的 `x-on:click` 表達式中。

#### Blade 參數注意事項

這是一個非常強大的技術，但在閱讀 Blade 模板時可能會讓人困惑。一開始很難知道哪些部分是 Blade，哪些部分是 Alpine。因此，檢查頁面上呈現的 HTML 以確保您期望呈現的內容是正確的是很有幫助的。

這裡有一個常常讓人困惑的例子：

假設，您的 Post 模型使用 UUID 作為索引（ID 是整數，UUID 是一長串字符）。

如果我們像處理 ID 一樣處理以下內容，將會出現問題：

```html
<!-- Warning: this is an example of problematic code... -->
<button
    type="button"
    x-on:click="$wire.deletePost({{ $post->uuid }})"
>
```

上面的 Blade 模板將在您的 HTML 中呈現如下內容： 

```html
<!-- Warning: this is an example of problematic code... -->
<button
    type="button"
    x-on:click="$wire.deletePost(93c7b04c-c9a4-4524-aa7d-39196011b81a)"
>
```

注意到UUID字符串周圍缺少引號嗎？當Alpine試圖評估這個表達式時，JavaScript會拋出錯誤："Uncaught SyntaxError: Invalid or unexpected token"。

為了解決這個問題，我們需要在Blade表達式周圍添加引號，如下所示：

```html
<button
    type="button"
    x-on:click="$wire.deletePost('{{ $post->uuid }}')"
>
```

現在上面的模板將正確呈現，一切都將按預期工作：

```html
<button
    type="button"
    x-on:click="$wire.deletePost('93c7b04c-c9a4-4524-aa7d-39196011b81a')"
>
```

### 刷新組件

您可以輕鬆地刷新Livewire組件（觸發網絡往返以重新渲染組件的Blade視圖），使用 `$wire.$refresh()`：

```html
<button type="button" x-on:click="$wire.$refresh()">
```

## 使用 `$wire.entangle` 共享狀態

在大多數情況下，使用 `$wire` 就足以從Alpine與Livewire之間交互Livewire狀態。但是，Livewire提供了一個額外的 `$wire.entangle()` 實用程序，可用於保持Livewire中的值與Alpine中的值同步。

為了演示，考慮這個下拉菜單示例，其中 `showDropdown` 屬性使用 `$wire.entangle()` 在Livewire和Alpine之間進行了糾纏。通過使用糾纏，我們現在能夠從Alpine和Livewire控制下拉菜單的狀態：

```php
use Livewire\Component;

class PostDropdown extends Component
{
    public $showDropdown = false;

    public function archive()
    {
        // ...

        $this->showDropdown = false;
    }

    public function delete()
    {
        // ...

        $this->showDropdown = false;
    }
}
```

```blade
<div x-data="{ open: $wire.entangle('showDropdown') }">
    <button x-on:click="open = true">Show More...</button>

    <ul x-show="open" x-on:click.outside="open = false">
        <li><button wire:click="archive">Archive</button></li>

        <li><button wire:click="delete">Delete</button></li>
    </ul>
</div>
```

現在用戶可以立即使用Alpine切換下拉菜單，但當他們點擊Livewire動作如"Archive"時，下拉菜單將從Livewire那裡被告知關閉。Alpine和Livewire都可以操作各自的屬性，另一方將自動更新。

默認情況下，更新狀態是延遲的（在客戶端上更改，但不會立即在服務器上）。如果您需要在用戶點擊時立即在服務器端更新狀態，請像這樣鏈接`.live`修飾符：

```blade
<div x-data="{ open: $wire.entangle('showDropdown').live }">
    ...
</div>
```

> [!tip] 您可能不需要 `$wire.entangle`
> 在大多數情況下，您可以通過使用 `$wire` 直接從Alpine訪問Livewire屬性來實現您想要的效果，而不是將它們糾纏在一起。將兩個屬性糾纏在一起而不是依賴一個可能會在使用頻繁更改的深度嵌套對象時引起可預測性和性能問題。因此，從Livewire 3版本開始，Livewire的文檔中已經淡化了 `$wire.entangle`。

> [!warning] 請勿使用 @@entangle 指示詞
> 在 Livewire 版本 2 中，建議使用 Blade 的 `@@entangle` 指示詞。但在 v3 中不再建議這樣做。建議使用 `$wire.entangle()`，因為這是一個更強大的工具，可以避免某些[刪除 DOM 元素時的問題](https://github.com/livewire/livewire/pull/6833#issuecomment-1902260844)。

## 在您的 JavaScript 構建中手動捆綁 Alpine

預設情況下，Livewire 和 Alpine 的 JavaScript 會自動注入到每個 Livewire 頁面中。

這對於簡單的設置是理想的，但您可能希望將自己的 Alpine 元件、存儲和插件包含到您的專案中。

要在頁面上通過您自己的 JavaScript 捆綁包含 Livewire 和 Alpine 是很簡單的。

首先，您必須在您的佈局檔案中包含 `@livewireScriptConfig` 指示詞，如下所示：

```blade
<html>
<head>
    <!-- ... -->
    @livewireStyles
    @vite(['resources/js/app.js'])
</head>
<body>
    {{ $slot }}

    @livewireScriptConfig <!-- [tl! highlight] -->
</body>
</html>
```

這允許 Livewire 為您的捆綁提供一些必要的配置，以使應用程序正常運行。

現在，您可以在您的 `resources/js/app.js` 檔案中導入 Livewire 和 Alpine，如下所示：

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';

// Register any Alpine directives, components, or plugins here...

Livewire.start()
```

這是在應用程序中註冊自定義 Alpine 指示詞 "x-clipboard" 的示例：

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';

Alpine.directive('clipboard', (el) => {
    let text = el.textContent

    el.addEventListener('click', () => {
        navigator.clipboard.writeText(text)
    })
})

Livewire.start()
```

現在，`x-clipboard` 指示詞將對您 Livewire 應用程序中的所有 Alpine 元件可用。
