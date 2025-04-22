## 在 Livewire 元件中使用 JavaScript

Livewire 和 Alpine 提供了許多工具，可以直接在 HTML 中建構動態元件，但有時候從 HTML 中脫離並執行純粹的 JavaScript 來編寫元件會更有幫助。Livewire 的 `@script` 和 `@assets` 指示詞讓您以一種可預測、易於維護的方式執行這些操作。

### 執行腳本

要在 Livewire 元件中執行自訂 JavaScript，只需將 `<script>` 元素用 `@script` 和 `@endscript` 包裹起來。這將告訴 Livewire 處理這段 JavaScript 的執行。

因為 `@script` 內部的腳本由 Livewire 處理，它們會在頁面加載完成後、Livewire 元件渲染之前的完美時機執行。這意味著您不再需要將腳本包裹在 `document.addEventListener('...')` 中以正確加載它們。

這也意味著懶加載或有條件加載的 Livewire 元件仍能在頁面初始化後執行 JavaScript。

```blade
<div>
    ...
</div>

@script
<script>
    // This Javascript will get executed every time this component is loaded onto the page...
</script>
@endscript
```

以下是一個更完整的範例，您可以像註冊一個在 Livewire 元件中使用的 JavaScript 操作一樣進行操作。

```blade
<div>
    <button wire:click="$js.increment">+</button>
</div>

@script
<script>
    $js('increment', () => {
        console.log('increment')
    })
</script>
@endscript
```

要了解更多有關 JavaScript 操作的資訊，請[查看操作文件](/docs/actions#javascript-actions)。

### 從腳本中使用 `$wire`

使用 `@script` 為您的 JavaScript 的另一個有用功能是您自動可以訪問到 Livewire 元件的 `$wire` 物件。

以下是一個使用簡單的 `setInterval` 來每 2 秒刷新元件的範例（您可以輕鬆使用 [`wire:poll`](/docs/wire-poll) 實現這一點，但這是一種展示重點的簡單方式）：

您可以在[`$wire` 文件](#the-wire-object)中了解更多關於 `$wire` 的資訊。

```blade
@script
<script>
    setInterval(() => {
        $wire.$refresh()
    }, 2000)
</script>
@endscript
```

### 評估一次性 JavaScript 表達式

除了指定整個方法在 JavaScript 中評估之外，您還可以使用 `js()` 方法在後端評估較小的個別表達式。

這通常用於在伺服器端執行某種操作後，執行某種客戶端後續操作。

例如，這裡是一個 `CreatePost` 元件的範例，該元件在將文章保存到資料庫後觸發客戶端警示對話框：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    public $title = '';

    public function save()
    {
        // ...

        $this->js("alert('Post saved!')"); // [tl! highlight:6]
    }
}
```

JavaScript 表達式 `alert('文章已保存！')` 現在將在伺服器端將文章保存到資料庫後，在客戶端執行。

您可以在表達式中訪問當前元件的 `$wire` 物件。

### 載入資源檔

`@script` 指示詞可用於在每次 Livewire 元件載入時執行一小段 JavaScript，但是，有時您可能希望在頁面上與元件一起載入整個腳本和樣式資源。

這是使用 `@assets` 載入名為 [Pikaday](https://github.com/Pikaday/Pikaday) 的日期選擇器庫並在元件內部使用 `@script` 初始化它的範例：

```blade
<div>
    <input type="text" data-picker>
</div>

@assets
<script src="https://cdn.jsdelivr.net/npm/pikaday/pikaday.js" defer></script>
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/pikaday/css/pikaday.css">
@endassets

@script
<script>
    new Pikaday({ field: $wire.$el.querySelector('[data-picker]') });
</script>
@endscript
```

當此元件載入時，Livewire 將確保在該頁面上載入任何 `@assets`，然後再評估 `@script`。此外，它將確保提供的 `@assets` 在頁面上只載入一次，無論有多少此元件的實例，而 `@script` 則會對頁面上每個元件實例進行評估。

## 全域 Livewire 事件

Livewire 為您發送了兩個有用的瀏覽器事件，供您從外部腳本註冊任何自定義擴展點：

```html
<script>
    document.addEventListener('livewire:init', () => {
        // Runs after Livewire is loaded but before it's initialized
        // on the page...
    })

    document.addEventListener('livewire:initialized', () => {
        // Runs immediately after Livewire has finished initializing
        // on the page...
    })
</script>
```

> [!info]
> 通常有益的是在 `livewire:init` 內註冊任何 [自定義指示詞](#registering-custom-directives) 或 [生命週期鉤子](#javascript-hooks)，以便它們在 Livewire 開始在頁面上初始化之前可用。

## `Livewire` 全域物件

Livewire 的全域物件是與外部腳本互動的最佳起點。

您可以在客戶端代碼的任何位置從 `window` 中訪問全域 `Livewire` JavaScript 物件。

通常在 `livewire:init` 事件監聽器中使用 `window.Livewire` 是有幫助的。

### 存取元件

您可以使用以下方法來存取當前頁面上加載的特定 Livewire 元件：

```js
// Retrieve the $wire object for the first component on the page...
let component = Livewire.first()

// Retrieve a given component's `$wire` object by its ID...
let component = Livewire.find(id)

// Retrieve an array of component `$wire` objects by name...
let components = Livewire.getByName(name)

// Retrieve $wire objects for every component on the page...
let components = Livewire.all()
```

> [!info]
> 這些方法中的每一個都會返回一個 `$wire` 物件，代表 Livewire 中元件的狀態。
> <br><br>
> 您可以在[此處](#the-wire-object)了解更多關於這些物件的資訊。

### 與事件互動

除了在 PHP 中從個別元件分派和監聽事件之外，全局的 `Livewire` 物件還允許您從應用程序的任何位置與[Livewire 的事件系統](/docs/events)互動：

```js
// Dispatch an event to any Livewire components listening...
Livewire.dispatch('post-created', { postId: 2 })

// Dispatch an event to a given Livewire component by name...
Livewire.dispatchTo('dashboard', 'post-created', { postId: 2 })

// Listen for events dispatched from Livewire components...
Livewire.on('post-created', ({ postId }) => {
    // ...
})
```

在某些情況下，您可能需要取消註冊全局 Livewire 事件。例如，在使用 Alpine 元件和 `wire:navigate` 時，當在頁面之間導航時會調用 `init`，可能會註冊多個監聽器。為了解決這個問題，請使用 `destroy` 函數，Alpine 會自動調用此函數。在此函數中循環遍歷所有監聽器，以取消註冊它們並防止任何不必要的累積。

```js
Alpine.data('MyComponent', () => ({
    listeners: [],
    init() {
        this.listeners.push(
            Livewire.on('post-created', (options) => {
                // Do something...
            })
        );
    },
    destroy() {
        this.listeners.forEach((listener) => {
            listener();
        });
    }
}));
```

### 使用生命週期鉤子

Livewire 允許您使用 `Livewire.hook()` 來鉤取其全局生命週期的各個部分：

```js
// Register a callback to execute on a given internal Livewire hook...
Livewire.hook('component.init', ({ component, cleanup }) => {
    // ...
})
```

有關 Livewire 的 JavaScript 鉤子的更多信息可以在[下面找到](#javascript-hooks)。

### 註冊自定義指示詞

Livewire 允許您使用 `Livewire.directive()` 註冊自定義指示詞。

以下是一個使用 JavaScript 的 `confirm()` 對話框來確認或取消在將動作發送到伺服器之前的操作的自定義 `wire:confirm` 指示詞的示例：

```html
<button wire:confirm="確定嗎？" wire:click="delete">刪除文章</button>
```

這是使用 `Livewire.directive()` 實現 `wire:confirm` 的範例：

```js
Livewire.directive('confirm', ({ el, directive, component, cleanup }) => {
    let content =  directive.expression

    // The "directive" object gives you access to the parsed directive.
    // For example, here are its values for: wire:click.prevent="deletePost(1)"
    //
    // directive.raw = wire:click.prevent
    // directive.value = "click"
    // directive.modifiers = ['prevent']
    // directive.expression = "deletePost(1)"

    let onClick = e => {
        if (! confirm(content)) {
            e.preventDefault()
            e.stopImmediatePropagation()
        }
    }

    el.addEventListener('click', onClick, { capture: true })

    // Register any cleanup code inside `cleanup()` in the case
    // where a Livewire component is removed from the DOM while
    // the page is still active.
    cleanup(() => {
        el.removeEventListener('click', onClick)
    })
})
```

## 物件結構

在擴展 Livewire 的 JavaScript 系統時，了解您可能遇到的不同物件是很重要的。

以下是 Livewire 相關內部屬性的詳盡參考。

作為提醒，一般 Livewire 使用者可能永遠不會與這些對象互動。這些對象大多數供 Livewire 的內部系統或高級使用者使用。

### `$wire` 對象

給定以下通用的 `Counter` 組件：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

Livewire 以一個對象的形式暴露了伺服器端組件的 JavaScript 表示，通常被稱為 `$wire`：

```js
let $wire = {
    // All component public properties are directly accessible on $wire...
    count: 0,

    // All public methods are exposed and callable on $wire...
    increment() { ... },

    // Access the `$wire` object of the parent component if one exists...
    $parent,

    // Access the root DOM element of the Livewire component...
    $el,

    // Access the ID of the current Livewire component...
    $id,

    // Get the value of a property by name...
    // Usage: $wire.$get('count')
    $get(name) { ... },

    // Set a property on the component by name...
    // Usage: $wire.$set('count', 5)
    $set(name, value, live = true) { ... },

    // Toggle the value of a boolean property...
    $toggle(name, live = true) { ... },

    // Call the method...
    // Usage: $wire.$call('increment')
    $call(method, ...params) { ... },

    // Define a JavaScript action...
    // Usage: $wire.$js('increment', () => { ... })
    $js(name, callback) { ... },

    // Entangle the value of a Livewire property with a different,
    // arbitrary, Alpine property...
    // Usage: <div x-data="{ count: $wire.$entangle('count') }">
    $entangle(name, live = false) { ... },

    // Watch the value of a property for changes...
    // Usage: Alpine.$watch('count', (value, old) => { ... })
    $watch(name, callback) { ... },

    // Refresh a component by sending a commit to the server
    // to re-render the HTML and swap it into the page...
    $refresh() { ... },

    // Identical to the above `$refresh`. Just a more technical name...
    $commit() { ... },

    // Listen for a an event dispatched from this component or its children...
    // Usage: $wire.$on('post-created', () => { ... })
    $on(event, callback) { ... },

    // Listen for a lifecycle hook triggered from this component or the request...
    // Usage: $wire.$hook('commit', () => { ... })
    $hook(name, callback) { ... },

    // Dispatch an event from this component...
    // Usage: $wire.$dispatch('post-created', { postId: 2 })
    $dispatch(event, params = {}) { ... },

    // Dispatch an event onto another component...
    // Usage: $wire.$dispatchTo('dashboard', 'post-created', { postId: 2 })
    $dispatchTo(otherComponentName, event, params = {}) { ... },

    // Dispatch an event onto this component and no others...
    $dispatchSelf(event, params = {}) { ... },

    // A JS API to upload a file directly to component
    // rather than through `wire:model`...
    $upload(
        name, // The property name
        file, // The File JavaScript object
        finish = () => { ... }, // Runs when the upload is finished...
        error = () => { ... }, // Runs if an error is triggered mid-upload...
        progress = (event) => { // Runs as the upload progresses...
            event.detail.progress // An integer from 1-100...
        },
    ) { ... },

    // API to upload multiple files at the same time...
    $uploadMultiple(name, files, finish, error, progress) { },

    // Remove an upload after it's been temporarily uploaded but not saved...
    $removeUpload(name, tmpFilename, finish, error) { ... },

    // Retrieve the underlying "component" object...
    __instance() { ... },
}
```

您可以在 [Livewire 的文檔中關於在 JavaScript 中訪問屬性](/docs/properties#accessing-properties-from-javascript) 了解更多關於 `$wire` 的資訊。

### `snapshot` 對象

在每個網絡請求之間，Livewire 將 PHP 組件序列化為一個可以在 JavaScript 中消耗的對象。此快照用於將組件反序列化回 PHP 對象，因此具有內建機制來防止篡改：

```js
let snapshot = {
    // The serialized state of the component (public properties)...
    data: { count: 0 },

    // Long-standing information about the component...
    memo: {
        // The component's unique ID...
        id: '0qCY3ri9pzSSMIXPGg8F',

        // The component's name. Ex. <livewire:[name] />
        name: 'counter',

        // The URI, method, and locale of the web page that the
        // component was originally loaded on. This is used
        // to re-apply any middleware from the original request
        // to subsequent component update requests (commits)...
        path: '/',
        method: 'GET',
        locale: 'en',

        // A list of any nested "child" components. Keyed by
        // internal template ID with the component ID as the values...
        children: [],

        // Weather or not this component was "lazy loaded"...
        lazyLoaded: false,

        // A list of any validation errors thrown during the
        // last request...
        errors: [],
    },

    // A securely encrypted hash of this snapshot. This way,
    // if a malicious user tampers with the snapshot with
    // the goal of accessing un-owned resources on the server,
    // the checksum validation will fail and an error will
    // be thrown...
    checksum: '1bc274eea17a434e33d26bcaba4a247a4a7768bd286456a83ea6e9be2d18c1e7',
}
```

### `component` 對象

頁面上的每個組件都有一個對應的組件對象，在幕後跟蹤其狀態並公開其底層功能。這比 `$wire` 深一層。僅供高級使用。

這是上述 `Counter` 組件的實際組件對象，其中包含相關屬性的描述在 JS 註釋中：

```js
let component = {
    // The root HTML element of the component...
    el: HTMLElement,

    // The unique ID of the component...
    id: '0qCY3ri9pzSSMIXPGg8F',

    // The component's "name" (<livewire:[name] />)...
    name: 'counter',

    // The latest "effects" object. Effects are "side-effects" from server
    // round-trips. These include redirects, file downloads, etc...
    effects: {},

    // The component's last-known server-side state...
    canonical: { count: 0 },

    // The component's mutable data object representing its
    // live client-side state...
    ephemeral: { count: 0 },

    // A reactive version of `this.ephemeral`. Changes to
    // this object will be picked up by AlpineJS expressions...
    reactive: Proxy,

    // A Proxy object that is typically used inside Alpine
    // expressions as `$wire`. This is meant to provide a
    // friendly JS object interface for Livewire components...
    $wire: Proxy,

    // A list of any nested "child" components. Keyed by
    // internal template ID with the component ID as the values...
    children: [],

    // The last-known "snapshot" representation of this component.
    // Snapshots are taken from the server-side component and used
    // to re-create the PHP object on the backend...
    snapshot: {...},

    // The un-parsed version of the above snapshot. This is used to send back to the
    // server on the next roundtrip because JS parsing messes with PHP encoding
    // which often results in checksum mis-matches.
    snapshotEncoded: '{"data":{"count":0},"memo":{"id":"0qCY3ri9pzSSMIXPGg8F","name":"counter","path":"\/","method":"GET","children":[],"lazyLoaded":true,"errors":[],"locale":"en"},"checksum":"1bc274eea17a434e33d26bcaba4a247a4a7768bd286456a83ea6e9be2d18c1e7"}',
}
```

### `commit` 載荷

當在瀏覽器中對 Livewire 組件執行操作時，將觸發一個網絡請求。該網絡請求包含一個或多個組件以及伺服器的各種指令。在內部，這些組件網絡載荷被稱為 "commits"。

選擇 "commit" 這個術語是為了幫助理解 Livewire 在前端和後端之間的關係。一個組件在前端被呈現和操作，直到執行需要它將其狀態和更新提交到後端的操作。

您將從瀏覽器的 DevTools 的網絡選項卡中的載荷或 [Livewire 的 JavaScript 鉤子](#javascript-hooks) 中認識到這個架構。

```js
let commit = {
    // Snapshot object...
    snapshot: { ... },

    // A key-value pair list of properties
    // to update on the server...
    updates: {},

    // An array of methods (with parameters) to call server-side...
    calls: [
        { method: 'increment', params: [] },
    ],
}
```

## JavaScript 掛勾

對於高級用戶，Livewire 公開了其內部客戶端“掛勾”系統。您可以使用以下掛勾來擴展 Livewire 的功能或獲取有關 Livewire 應用程序的更多信息。

### 元件初始化

每當 Livewire 發現新元件時 - 無論是在初始頁面加載時還是以後 - 都會觸發 `component.init` 事件。您可以掛鉤到 `component.init` 以攔截或初始化與新元件相關的任何內容：

```js
Livewire.hook('component.init', ({ component, cleanup }) => {
    //
})
```

有關更多信息，請參考[元件物件的文件](#the-component-object)。

### DOM 元素初始化

除了在初始化新元件時觸發事件外，Livewire 還會為給定 Livewire 元件中的每個 DOM 元素觸發一個事件。

這可用於在應用程序中提供自定義的 Livewire HTML 屬性：

```js
Livewire.hook('element.init', ({ component, el }) => {
    //
})
```

### DOM 變形掛勾

在 DOM 變形階段 - 在 Livewire 完成網絡往返後發生 - Livewire 會為每個變異的元素觸發一系列事件。

```js
Livewire.hook('morph.updating',  ({ el, component, toEl, skip, childrenOnly }) => {
	//
})

Livewire.hook('morph.updated', ({ el, component }) => {
	//
})

Livewire.hook('morph.removing', ({ el, component, skip }) => {
	//
})

Livewire.hook('morph.removed', ({ el, component }) => {
	//
})

Livewire.hook('morph.adding',  ({ el, component }) => {
	//
})

Livewire.hook('morph.added',  ({ el }) => {
	//
})
```

除了每個元素觸發的事件外，還會為每個 Livewire 元件觸發 `morph` 和 `morphed` 事件：

```js
Livewire.hook('morph',  ({ el, component }) => {
	// Runs just before the child elements in `component` are morphed
})

Livewire.hook('morphed',  ({ el, component }) => {
    // Runs after all child elements in `component` are morphed
})
```

### 提交掛勾

由於 Livewire 請求包含多個元件，將 _請求_ 用於指稱單個元件的請求和響應有效負荷太過廣泛。相反，在內部，Livewire 將元件更新稱為 _提交_ - 指向將元件狀態提交到服務器。

這些掛勾公開了 `commit` 物件。您可以通過閱讀[提交物件文件](#the-commit-payload)來了解更多關於它們的架構。

#### 準備提交

在發送請求到服務器之前，將觸發 `commit.prepare` 掛勾。這讓您有機會向即將發送的請求添加任何最後一刻的更新或操作：

```js
Livewire.hook('commit.prepare', ({ component }) => {
    // 在提交有效載荷被收集並發送到伺服器之前運行...
})
```

#### 截取提交

每次 Livewire 元件被送到伺服器時，都會進行一次 _commit_。為了鉤取個別提交的生命週期和內容，Livewire 提供了一個 `commit` 鉤子。

這個鉤子非常強大，因為它提供了鉤取 Livewire 提交的請求和回應的方法：

```js
Livewire.hook('commit', ({ component, commit, respond, succeed, fail }) => {
    // Runs immediately before a commit's payload is sent to the server...

    respond(() => {
        // Runs after a response is received but before it's processed...
    })

    succeed(({ snapshot, effects }) => {
        // Runs after a successful response is received and processed
        // with a new snapshot and list of effects...
    })

    fail(() => {
        // Runs if some part of the request failed...
    })
})
```

## 請求鉤子

如果您希望鉤取整個從伺服器發送和返回的 HTTP 請求，可以使用 `request` 鉤子：

```js
Livewire.hook('request', ({ url, options, payload, respond, succeed, fail }) => {
    // Runs after commit payloads are compiled, but before a network request is sent...

    respond(({ status, response }) => {
        // Runs when the response is received...
        // "response" is the raw HTTP response object
        // before await response.text() is run...
    })

    succeed(({ status, json }) => {
        // Runs when the response is received...
        // "json" is the JSON response object...
    })

    fail(({ status, content, preventDefault }) => {
        // Runs when the response has an error status code...
        // "preventDefault" allows you to disable Livewire's
        // default error handling...
        // "content" is the raw response content...
    })
})
```

### 自訂頁面過期行為

如果預設的頁面過期對話框不適用於您的應用程式，您可以使用 `request` 鉤子來實現自訂解決方案：

```html
<script>
    document.addEventListener('livewire:init', () => {
        Livewire.hook('request', ({ fail }) => {
            fail(({ status, preventDefault }) => {
                if (status === 419) {
                    confirm('Your custom page expiration behavior...')

                    preventDefault()
                }
            })
        })
    })
</script>
```

使用上述程式碼在您的應用程式中，當使用者的會話過期時，他們將收到一個自訂對話框。
