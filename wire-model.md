Livewire 讓使用 `wire:model` 輕鬆將元件屬性的值與表單輸入綁定。

以下是在 "Create Post" 元件中使用 `wire:model` 綁定 `$title` 和 `$content` 屬性與表單輸入的簡單範例：

```php
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
		$post = Post::create([
			'title' => $this->title
			'content' => $this->content
		]);

        // ...
    }
}
```

```blade
<form wire:submit="save">
    <label>
        <span>Title</span>

        <input type="text" wire:model="title"> <!-- [tl! highlight] -->
    </label>

    <label>
        <span>Content</span>

        <textarea wire:model="content"></textarea> <!-- [tl! highlight] -->
    </label>

	<button type="submit">Save</button>
</form>
```

因為兩個輸入都使用了 `wire:model`，當按下 "Save" 按鈕時，它們的值將與伺服器的屬性同步。

> [!warning] "為什麼我的元件在我輸入時沒有即時更新？"
> 如果您在瀏覽器中嘗試並且對於標題沒有自動更新感到困惑，這是因為 Livewire 只有在提交 "動作" 時才會更新元件，例如按下提交按鈕，而不是在使用者輸入時。這樣可以減少網路請求並提升效能。若要啟用使用者輸入時的 "即時" 更新，您可以改用 `wire:model.live`。[了解更多關於資料綁定](/docs/properties#data-binding)。

## 自訂更新時機

預設情況下，Livewire 只會在執行動作時（如 `wire:click` 或 `wire:submit`）才會發送網路請求，而不是在更新 `wire:model` 輸入時。

這大幅提升了 Livewire 的效能，減少了網路請求，並為使用者提供更流暢的體驗。

然而，有時您可能希望更頻繁地更新伺服器，例如即時驗證。

### 即時更新

要在使用者輸入到輸入欄時將屬性更新發送至伺服器，您可以在 `wire:model` 後附加 `.live` 修飾符：

```html
<input type="text" wire:model.live="title">
```

#### 自訂防彈跳時間

預設情況下，使用 `wire:model.live` 時，Livewire 會對伺服器更新增加 150 毫秒的防彈跳。這表示如果使用者持續輸入，Livewire 將等待使用者停止輸入 150 毫秒後才發送請求。

您可以透過在輸入後附加 `.debounce.Xms` 來自訂這個時間。以下是將防彈跳更改為 250 毫秒的範例：

```html
<input type="text" wire:model.live.debounce.250ms="title">
```

### 在 "blur" 事件上更新

通過附加 `.blur` 修飾符，Livewire 只會在用戶點擊輸入框外部或按下 Tab 鍵移動到下一個輸入框時，發送具有屬性更新的網絡請求。

添加 `.blur` 對於您希望更頻繁地更新服務器，但不是在用戶輸入時很有幫助。例如，即時驗證是一個常見的情況，這時 `.blur` 很有幫助。

```html
<input type="text" wire:model.blur="title">
```

### 在 "change" 事件上更新

有時 `.blur` 的行為並不完全符合您的期望，而是使用 `.change`。

例如，如果您希望每次選擇輸入框時運行驗證，通過添加 `.change`，Livewire 將發送一個網絡請求並在用戶選擇新選項時立即驗證屬性。與 `.blur` 相反，後者只會在用戶從選擇輸入框切換時更新服務器。

```html
<select wire:model.change="title">
    <!-- ... -->
</select>
```

對文本輸入框所做的任何更改將自動與 Livewire 組件中的 `$title` 屬性同步。

## 所有可用的修飾符

 修飾符            | 說明
-------------------|-------------------------------------------------------------------------
 `.live`           | 當用戶輸入時發送更新
 `.blur`           | 僅在 `blur` 事件上發送更新
 `.change`         | 僅在 `change` 事件上發送更新
 `.lazy`           | `.change` 的別名
 `.debounce.[?]ms` | 以指定的毫秒延遲來延遲發送更新
 `.throttle.[?]ms` | 以指定的毫秒間隔節流網絡請求更新
 `.number`         | 在服務器上將輸入的文本值轉換為 `int`
 `.boolean`        | 在服務器上將輸入的文本值轉換為 `bool`
 `.fill`           | 在頁面加載時使用 "value" HTML 屬性提供的初始值

## 輸入字段

Livewire 支持大多數原生輸入元素。這意味著您應該能夠將 `wire:model` 附加到瀏覽器中的任何輸入元素並輕鬆地將屬性綁定到它們上。 

--- 

*本翻譯遵循嚴格的翻譯規則，如有任何問題，請隨時告知。*

這裡是不同可用輸入類型的全面清單，以及在 Livewire 上下文中如何使用它們。

### 文本輸入

首先，文本輸入是大多數表單的基石。以下是如何將名為 "title" 的屬性綁定到一個文本輸入框：

```blade
<input type="text" wire:model="title">
```

### 文本區輸入

文本區元素同樣直截了當。只需將 `wire:model` 添加到文本區，值將被綁定：

```blade
<textarea type="text" wire:model="content"></textarea>
```

如果 "content" 值初始化為一個字符串，Livewire 將使用該值填充文本區 - 不需要像下面這樣做：

```blade
<!-- 警告：此片段演示了不應該做的事情... -->

<textarea type="text" wire:model="content">{{ $content }}</textarea>
```

### 复选框

复选框可用於單個值，例如切換布爾屬性時。或者，复选框可以用於在一組相關值中切換單個值。我們將討論這兩種情況：

#### 單個复选框

在註冊表單的末尾，您可能會有一個允許用戶選擇是否接收電子郵件更新的复选框。您可以將此屬性命名為 `$receiveUpdates`。您可以使用 `wire:model` 輕鬆將此值綁定到复选框：

```blade
<input type="checkbox" wire:model="receiveUpdates">
```

現在當 `$receiveUpdates` 值為 `false` 時，复选框將未選中。當值為 `true` 時，复选框將被選中。

#### 多個复选框

現在，假設除了允許用戶決定是否接收更新之外，您的類中還有一個名為 `$updateTypes` 的數組屬性，允許用戶從各種更新類型中選擇：

```php
public $updateTypes = [];
```

通過將多個复选框綁定到 `$updateTypes` 屬性，用戶可以選擇多個更新類型，並將它們添加到 `$updateTypes` 數組屬性中：

```blade
<input type="checkbox" value="email" wire:model="updateTypes">
<input type="checkbox" value="sms" wire:model="updateTypes">
<input type="checkbox" value="notification" wire:model="updateTypes">
```

例如，如果使用者勾選了前兩個方塊但沒有勾選第三個，`$updateTypes` 的值將是：`["email", "sms"]`

### 單選按鈕

要在單一屬性的兩個不同值之間切換，您可以使用單選按鈕：

```blade
<input type="radio" value="yes" wire:model="receiveUpdates">
<input type="radio" value="no" wire:model="receiveUpdates">
```

### 選擇下拉選單

Livewire 讓使用 `<select>` 下拉選單變得簡單。將 `wire:model` 添加到下拉選單時，目前選定的值將綁定到提供的屬性名稱，反之亦然。

此外，您無需手動將 `selected` 添加到將被選中的選項 - Livewire 會自動為您處理。

以下是一個填充靜態州列表的選擇下拉選單示例：

```blade
<select wire:model="state">
    <option value="AL">Alabama</option>
    <option value="AK">Alaska</option>
    <option value="AZ">Arizona</option>
    ...
</select>
```

當選擇特定的州時，例如 "阿拉斯加"，元件上的 `$state` 屬性將設置為 `AK`。如果您希望值設置為 "阿拉斯加" 而不是 "AK"，您可以完全省略 `<option>` 元素的 `value=""` 屬性。

通常，您可以使用 Blade 動態生成下拉選項：

```blade
<select wire:model="state">
    @foreach (\App\Models\State::all() as $state)
        <option value="{{ $state->id }}">{{ $state->label }}</option>
    @endforeach
</select>
```

如果您沒有特定的預設選項，您可能希望默認顯示一個灰色的佔位符選項，例如 "選擇一個州"：

```blade
<select wire:model="state">
    <option disabled value="">Select a state...</option>

    @foreach (\App\Models\State::all() as $state)
        <option value="{{ $state->id }}">{{ $state->label }}</option>
    @endforeach
</select>
```

如您所見，對於下拉選單，沒有像文本輸入框那樣的 "placeholder" 屬性。相反，您必須將 `disabled` 選項元素添加為列表中的第一個選項。

### 依賴選擇下拉選單

有時您可能希望一個下拉選單依賴於另一個。例如，一個基於所選州而變化的城市列表。

在大多數情況下，這將按您的預期工作，但有一個重要的要點：您必須向變化的下拉選單添加 `wire:key`，以便 Livewire 在選項更改時正確刷新其值。

這是兩個選擇的示例，一個用於州，一個用於城市。當州選擇更改時，城市選擇中的選項將正確更改：

再次，這裡唯一不標準的是在第二個選擇框中添加的 `wire:key`。這確保當狀態改變時，“selectedCity”值將被正確重置。

### 多選下拉菜單

如果您使用“multiple”選單，Livewire將按預期工作。在這個例子中，當選擇狀態時，它們將被添加到 `$states` 陣列屬性中，如果取消選擇，則將被移除：

```blade
<select wire:model="states" multiple>
    <option value="AL">Alabama</option>
    <option value="AK">Alaska</option>
    <option value="AZ">Arizona</option>
    ...
</select>
```

## 深入了解

有關在 HTML 表單中使用 `wire:model` 的更完整文檔，請訪問[Livewire 表單文檔頁面](/docs/forms)。
