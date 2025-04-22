加載指示器是打造良好使用者介面的重要部分。當向伺服器發送請求時，它們為使用者提供視覺反饋，讓使用者知道他們正在等待一個過程完成。

## 基本用法

Livewire 提供了一個簡單但非常強大的語法來控制加載指示器：`wire:loading`。將 `wire:loading` 添加到任何元素將使其默認隱藏（使用 CSS 中的 `display: none`），並在向伺服器發送請求時顯示它。

以下是一個 `CreatePost` 組件表單的基本示例，其中使用 `wire:loading` 來切換加載消息：

```blade
<form wire:submit="save">
    <!-- ... -->

    <button type="submit">Save</button>

    <div wire:loading> <!-- [tl! highlight:2] -->
        Saving post...
    </div>
</form>
```

當使用者按下「儲存」時，在執行「儲存」動作時，「正在儲存文章...」消息將出現在按鈕下方。當從伺服器接收到回應並由 Livewire 處理時，該消息將消失。

### 移除元素

或者，您可以附加 `.remove` 以獲得相反效果，默認情況下顯示一個元素，並在向伺服器發送請求時隱藏它：

```blade
<div wire:loading.remove>...</div>
```

## 切換類別

除了切換整個元素的可見性之外，通常還可以在向伺服器發送請求期間通過切換 CSS 類別來更改現有元素的樣式。這種技術可用於更改背景顏色、降低不透明度、觸發旋轉動畫等操作。

以下是一個簡單示例，使用 [Tailwind](https://tailwindcss.com/) 類別 `opacity-50` 使表單提交時的「儲存」按鈕變得較淡：

```blade
<button wire:loading.class="opacity-50">儲存</button>
```

與切換元素一樣，您可以通過在 `wire:loading` 指示詞後附加 `.remove` 來執行相反的類別操作。在下面的示例中，當按下「儲存」按鈕時，將刪除按鈕的 `bg-blue-500` 類別：

```blade
<button class="bg-blue-500" wire:loading.class.remove="bg-blue-500">
    儲存
</button>
```

## 切換屬性

默認情況下，當提交表單時，Livewire 將自動禁用提交按鈕並在處理表單時向每個輸入元素添加 `readonly` 屬性。

然而，除了這種預設行為之外，Livewire 還提供了 `.attr` 修飾符，允許您在元素上切換其他屬性，或切換表單之外元素的屬性：

```blade
<button
    type="button"
    wire:click="remove"
    wire:loading.attr="disabled"
>
    Remove
</button>
```

因為上面的按鈕不是提交按鈕，當按下時不會被 Livewire 的預設表單處理行為禁用。相反，我們手動添加了 `wire:loading.attr="disabled"` 來實現這種行為。

## 針對特定操作

預設情況下，當組件向伺服器發送請求時，`wire:loading` 將被觸發。

然而，在具有多個元素可以觸發伺服器請求的組件中，您應該將加載指示器範圍限制在個別操作中。

例如，考慮以下的「儲存文章」表單。除了一個提交表單的「儲存」按鈕外，還可能有一個執行組件上的「刪除」操作的「刪除」按鈕。

通過將以下 `wire:loading` 元素添加 `wire:target`，您可以指示 Livewire 只在點擊「刪除」按鈕時顯示加載消息：

```blade
<form wire:submit="save">
    <!-- ... -->

    <button type="submit">Save</button>

    <button type="button" wire:click="remove">Remove</button>

    <div wire:loading wire:target="remove">  <!-- [tl! highlight:2] -->
        Removing post...
    </div>
</form>
```

當上面的「刪除」按鈕被按下時，將向用戶顯示「正在刪除文章...」消息。但是，當按下「儲存」按鈕時，該消息將不會顯示。

### 針對多個操作

您可能會遇到一種情況，希望 `wire:loading` 對頁面上的某些操作做出反應，但不是全部。在這些情況下，您可以通過逗號分隔的方式將多個操作傳遞給 `wire:target`。例如：

```blade
<form wire:submit="save">
    <input type="text" wire:model.blur="title">

    <!-- ... -->

    <button type="submit">Save</button>

    <button type="button" wire:click="remove">Remove</button>

    <div wire:loading wire:target="save, remove">  <!-- [tl! highlight:2] -->
        Updating post...
    </div>
</form>
```

現在，只有在按下「刪除」或「儲存」按鈕時才會顯示加載指示器（「正在更新文章...」），而不是當 `$title` 欄位被發送到伺服器時。

在未將 `{{ $post->id }}` 傳遞給 `wire:target="remove"` 的情況下，當頁面上的任何按鈕被點擊時，將顯示 "正在移除文章..." 訊息。

然而，由於我們對每個 `wire:target` 實例傳遞了唯一的參數，Livewire 只會在將匹配的參數傳遞給 "remove" 操作時顯示載入訊息。

> [!warning] 不支援多個動作參數
> 目前，Livewire 僅支援透過單一方法/參數對來定位載入指示器。例如 `wire:target="remove(1)"` 是支援的，但 `wire:target="remove(1), add(1)"` 則不支援。

### 定位屬性更新

Livewire 還允許您通過將屬性名稱傳遞給 `wire:target` 指令來定位特定組件屬性的更新。

考慮以下示例，其中一個名為 `username` 的表單輸入使用 `wire:model.live` 進行實時驗證，當用戶在輸入字段中輸入時：

```blade
<form wire:submit="save">
    <input type="text" wire:model.live="username">
    @error('username') <span>{{ $message }}</span> @enderror

    <div wire:loading wire:target="username"> <!-- [tl! highlight:2] -->
        Checking availability of username...
    </div>

    <!-- ... -->
</form>
```

當伺服器更新為新的使用者名稱時，"正在檢查可用性..." 訊息將顯示。

### 排除特定的載入目標

有時您可能希望為每個 Livewire 請求顯示一個載入指示器，_除了_ 特定的屬性或操作。在這些情況下，您可以使用 `wire:target.except` 修飾符，如下所示：

```blade
<div wire:loading wire:target.except="download">...</div>
```

現在，上述載入指示器將顯示在組件的每個 Livewire 更新請求上，_除了_ "download" 操作。

## 自訂 CSS 顯示屬性

當將 `wire:loading` 添加到元素時，Livewire 會更新元素的 CSS `display` 屬性以顯示和隱藏元素。默認情況下，Livewire 使用 `none` 來隱藏和 `inline-block` 來顯示。

如果您要切換使用除 `inline-block` 之外的顯示值的元素，例如以下示例中的 `flex`，您可以在 `wire:loading` 後附加 `.flex`：

```blade
<div class="flex" wire:loading.flex>...</div>
```

以下是可用的完整顯示值列表：

```blade
<div wire:loading.inline-flex>...</div>
<div wire:loading.inline>...</div>
<div wire:loading.block>...</div>
<div wire:loading.table>...</div>
<div wire:loading.flex>...</div>
<div wire:loading.grid>...</div>
```

## 延遲載入指示器

在快速連線下，更新通常發生得非常快速，以致於載入指示器在畫面上只會短暫閃爍後就被移除。在這些情況下，指示器更像是一種干擾，而非有用的提示。

因此，Livewire 提供了 `.delay` 修飾符來延遲顯示指示器。例如，如果您將 `wire:loading.delay` 添加到元素中，如下所示：

```blade
<div wire:loading.delay>...</div>
```

上述元素只會在請求超過 200 毫秒時才會出現。如果請求在此之前完成，使用者將永遠看不到指示器。

若要自訂延遲載入指示器的時間，您可以使用 Livewire 提供的其中一個有用的間隔別名：

```blade
<div wire:loading.delay.shortest>...</div> <!-- 50ms -->
<div wire:loading.delay.shorter>...</div>  <!-- 100ms -->
<div wire:loading.delay.short>...</div>    <!-- 150ms -->
<div wire:loading.delay>...</div>          <!-- 200ms -->
<div wire:loading.delay.long>...</div>     <!-- 300ms -->
<div wire:loading.delay.longer>...</div>   <!-- 500ms -->
<div wire:loading.delay.longest>...</div>  <!-- 1000ms -->
```
