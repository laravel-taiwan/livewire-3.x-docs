使用 Livewire 感覺就像將一個伺服器端的 PHP 類別直接附加到網頁瀏覽器上。像是直接從按鈕按下呼叫伺服器端函式這樣的事情支持了這種幻覺。但實際上，這僅僅是一種幻覺。

在背景中，Livewire 實際上更像是一個標準的網頁應用程式。它將靜態 HTML 渲染到瀏覽器，監聽瀏覽器事件，然後進行 AJAX 請求以調用伺服器端程式碼。

因為 Livewire 對伺服器發送的每個 AJAX 請求都是“無狀態”的（意味著沒有長時間運行的後端處理程序來保持元件狀態的活躍），Livewire 必須在進行任何更新之前重新創建元件的最後已知狀態。

它通過在每次伺服器端更新後對 PHP 元件進行“快照”來實現這一點，以便在下一個請求中可以重新創建或 _恢復_ 元件。

在本文檔中，我們將將拍攝快照的過程稱為“脫水” ，將從快照重新創建元件的過程稱為“水合”。

## 脫水

當 Livewire _脫水_ 一個伺服器端元件時，它會執行兩個操作：

* 將元件的模板渲染為 HTML
* 創建元件的 JSON 快照

### 渲染 HTML

在掛載元件或進行更新後，Livewire 調用元件的 `render()` 方法將 Blade 模板轉換為原始 HTML。

以以下 `Counter` 元件為例：

```php
class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            Count: {{ $count }}

            <button wire:click="increment">+</button>
        </div>
        HTML;
    }
}
```

在每次掛載或更新後，Livewire 會將上述 `Counter` 元件渲染為以下 HTML：

```html
<div>
    Count: 1

    <button wire:click="increment">+</button>
</div>
```

### 快照

為了在下一個請求期間在伺服器上重新創建 `Counter` 元件，會創建一個 JSON 快照，嘗試捕獲元件狀態的盡可能多的部分：

```js
{
    state: {
        count: 1,
    },

    memo: {
        name: 'counter',

        id: '1526456',
    },
}
```

請注意快照的兩個不同部分：`memo` 和 `state`。

`memo` 部分用於存儲識別和重新創建元件所需的信息，而 `state` 部分則存儲元件所有公共屬性的值。

### 將快照嵌入 HTML 中

當元件首次呈現時，Livewire 將快照以 JSON 格式存儲在名為 `wire:snapshot` 的 HTML 屬性中。這樣，Livewire 的 JavaScript 核心可以提取 JSON 並將其轉換為運行時物件：

```html
<div wire:id="..." wire:snapshot="{ state: {...}, memo: {...} }">
    Count: 1

    <button wire:click="increment">+</button>
</div>
```

## 水合

當觸發元件更新時，例如在 `Counter` 元件中按下 "+" 按鈕，將向伺服器發送如下載荷：

```js
{
    calls: [
        { method: 'increment', params: [] },
    ],

    snapshot: {
        state: {
            count: 1,
        },

        memo: {
            name: 'counter',

            id: '1526456',
        },
    }
}
```

在 Livewire 可以調用 `increment` 方法之前，必須首先創建一個新的 `Counter` 實例並使用快照的狀態來初始化它。

以下是一些實現此結果的 PHP 偽代碼：

```php
$state = request('snapshot.state');
$memo = request('snapshot.memo');

$instance = Livewire::new($memo['name'], $memo['id']);

foreach ($state as $property => $value) {
    $instance[$property] = $value;
}
```

如果按照上述腳本操作，您將看到在創建 `Counter` 物件後，它的公共屬性根據快照提供的狀態進行設置。

## 進階水合

上述 `Counter` 範例很好地演示了水合的概念；但是，它僅演示了 Livewire 如何處理像整數 (`1`) 這樣的簡單值的水合。

正如您可能知道的那樣，Livewire 支持比整數更複雜的屬性類型。

讓我們來看一個稍微複雜一點的例子 - 一個 `Todos` 元件：

```php
class Todos extends Component
{
    public $todos;

    public function mount() {
        $this->todos = collect([
            'first',
            'second',
            'third',
        ]);
    }
}
```

如您所見，我們將 `$todos` 屬性設置為一個包含三個字符串的 [Laravel 集合](https://laravel.com/docs/collections#main-content)。

僅僅使用 JSON 無法表示 Laravel 集合，因此 Livewire 創建了自己的模式，將元數據與快照中的純數據關聯起來。

這是 `Todos` 元件的快照狀態物件：

這可能會讓您感到困惑，如果您預期的內容更直接，像是：

```js
state: {
    todos: [ 'first', 'second', 'third' ],
},
```

然而，如果 Livewire 根據這個資料來初始化元件，它將無法知道這是一個集合而不是一個普通陣列。

因此，Livewire 支援另一種狀態語法，以元組（包含兩個項目的陣列）的形式：

```js
todos: [
    [ 'first', 'second', 'third' ],
    { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
],
```

當 Livewire 在初始化元件狀態時遇到元組時，它會使用存儲在元組第二個元素中的資訊，更智能地初始化存儲在第一個元素中的狀態。

為了更清楚地演示，這裡是簡化的程式碼，顯示了 Livewire 如何根據上面的快照重新創建集合屬性：

```php
[ $state, $metadata ] = request('snapshot.state.todos');

$collection = new $metadata['class']($state);
```

如您所見，Livewire 使用與狀態相關的元數據來推導完整的集合類別。

### 深度嵌套的元組

這種方法的一個明顯優勢是能夠對深度嵌套的屬性進行脫水和初始化。

例如，考慮上面的 `Todos` 範例，但現在第三個項目是 [Laravel Stringable](https://laravel.com/docs/helpers#method-str) 而不是普通字串：

```php
class Todos extends Component
{
    public $todos;

    public function mount() {
        $this->todos = collect([
            'first',
            'second',
            str('third'),
        ]);
    }
}
```

這個元件狀態的脫水快照現在看起來像這樣：

```js
todos: [
    [
        'first',
        'second',
        [ 'third', { s: 'str' } ],
    ],
    { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
],
```

如您所見，集合中的第三個項目已經被脫水為元數據元組。元組中的第一個元素是普通字串值，第二個元素是一個標誌，表示這個字串是一個 _stringable_。

### 支援自定義屬性類型

在內部，Livewire 支援最常見的 PHP 和 Laravel 類型的初始化。但是，如果您希望支援不支持的類型，您可以使用 [合成器](/docs/synthesizers) — Livewire 用於初始化/脫水非原始屬性類型的內部機制。

I'm ready to translate. Please provide the Markdown content for translation.
