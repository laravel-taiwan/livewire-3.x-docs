## 自動升級工具

為了節省您升級的時間，我們已經包含了一個 Artisan 指令，以自動化盡可能多的升級過程。

在[安裝 Livewire 版本 3](/docs/upgrading#update-livewire-to-version-3)之後，執行以下指令，您將收到提示來自動升級每個破壞性更改：

```shell
php artisan livewire:upgrade
```

儘管上述指令可以升級您應用程式的大部分部分，但確保完整升級的唯一方法是按照本頁面上的逐步指南進行操作。

> [!tip] 聘請我們來升級您的應用程式
> 如果您有一個龐大的 Livewire 應用程式，或者只是不想從版本 2 升級到版本 3，您可以聘請我們來為您處理。[在這裡了解更多關於我們的升級服務。](/jumpstart)

## 升級 PHP

Livewire 現在要求您的應用程式運行在 PHP 版本 8.1 或更高版本。

## 更新 Livewire 到版本 3

執行以下 composer 指令來將您應用程式的 Livewire 依賴從版本 2 升級到 3：

```shell
composer require livewire/livewire "^3.0"
```

> [!warning] Livewire 3 套件相容性
> 大多數主要的第三方 Livewire 套件目前支援 Livewire 3，或者正在努力支援它。然而，必然會有一些套件需要更長的時間來釋出對 Livewire 3 的支援。

## 清除視圖快取

從您應用程式的根目錄執行以下 Artisan 指令，以清除任何已快取/編譯的 Blade 視圖，並強制 Livewire 重新編譯以符合 Livewire 3 的相容性：

```shell
php artisan view:clear
```

## 合併新的組態

Livewire 3 已更改多個組態選項。如果您的應用程式有一個已發佈的組態檔案（`config/livewire.php`），您將需要更新它以考慮以下更改。

### 新的組態

以下組態鍵已在版本 3 中引入：

```php
'legacy_model_binding' => false,

'inject_assets' => true,

'inject_morph_markers' => true,

'navigate' => false,

'pagination_theme' => 'tailwind',
```

您可以參考 [Livewire 在 GitHub 上的新組態檔案](https://github.com/livewire/livewire/blob/master/config/livewire.php) 以獲取額外選項描述和可複製貼上的程式碼。

### 已更改的組態設定

以下組態項目已使用新的預設值進行更新：

#### 新的類別命名空間

Livewire 的預設 `class_namespace` 已從 `App\Http\Livewire` 更改為 `App\Livewire`。您可以保留舊的命名空間配置值；但如果您選擇將配置更新為新的命名空間，則必須將 Livewire 元件移至 `app/Livewire`：

```php
'class_namespace' => 'App\\Http\\Livewire', // [tl! remove]
'class_namespace' => 'App\\Livewire', // [tl! add]
```

#### 新的版面視圖路徑

在版本 2 中呈現全頁面元件時，Livewire 將使用 `resources/views/layouts/app.blade.php` 作為預設版面 Blade 元件。

由於社群對匿名 Blade 元件的偏好日益增加，Livewire 3 已將預設位置更改為：`resources/views/components/layouts/app.blade.php`。

```php
'layout' => 'layouts.app', // [tl! remove]
'layout' => 'components.layouts.app', // [tl! add]
```

### 已移除的組態設定

Livewire 不再識別以下組態項目。

#### `app_url`

如果您的應用程式在非根 URI 下運行，在 Livewire 2 中，您可以使用 `app_url` 配置選項來配置 Livewire 用於發送 AJAX 請求的 URL。

在這種情況下，我們發現字符串配置過於嚴格。因此，Livewire 3 選擇改為使用運行時配置。您可以參考我們的[配置 Livewire 更新端點的文件](/docs/installation#configuring-livewires-update-endpoint)以獲取更多信息。

#### `asset_url`

在 Livewire 2 中，如果您的應用程式在非根 URI 下運行，您將使用 `asset_url` 配置選項來配置 Livewire 用於提供其 JavaScript 資源的基本 URL。

Livewire 3 取而代之選擇了一種運行時配置策略。您可以參考我們的[配置 Livewire 腳本資源端點的文件](/docs/installation#customizing-the-asset-url)以獲取更多信息。

#### `middleware_group`

因為 Livewire 現在提供了更靈活的方式來自定義其更新端點，`middleware_group` 組態選項已被移除。

您可以參考我們的[自定義 Livewire 更新端點](/docs/installation#configuring-livewires-update-endpoint)文件，以獲取有關將自訂中介層應用於 Livewire 請求的更多資訊。

#### `manifest_path`

Livewire 3 不再使用元件自動加載的清單檔案。因此，`manifest_path` 組態已不再需要。

#### `back_button_cache`

由於 Livewire 3 現在提供了一個[使用 `wire:navigate` 為應用程式提供 SPA 体驗](/docs/navigate)的功能，`back_button_cache` 組態已不再需要。

## Livewire 應用程式命名空間

在版本 2 中，Livewire 元件是在 `App\Http\Livewire` 命名空間下自動生成和識別的。

Livewire 3 將此預設值更改為：`App\Livewire`。

您可以將所有元件移至新位置，或將以下組態添加到應用程式的 `config/livewire.php` 組態檔案中：

```php
'class_namespace' => 'App\\Http\\Livewire',
```

### 發現

在 Livewire 3 中，不存在清單，因此與 Livewire 元件相關的“發現”內容已不再存在，您可以安全地從構建腳本中刪除任何 livewire:discover 的引用。

## 頁面元件版面視圖

當使用以下語法將 Livewire 元件呈現為完整頁面時：

```php
Route::get('/posts', ShowPosts::class);
```

Livewire 用於呈現元件的 Blade 版面檔案已從 `resources/views/layouts/app.blade.php` 更改為 `resources/views/components/layouts/app.blade.php`：

```shell
resources/views/layouts/app.blade.php #[tl! remove]
resources/views/components/layouts/app.blade.php #[tl! add]
```

您可以將您的版面檔案移至新位置，或在應用程式的 `config/livewire.php` 組態檔案中應用以下組態：

```php
'layout' => 'layouts.app',
```

## Eloquent 模型綁定

Livewire 2 支援將 `wire:model` 直接綁定到 Eloquent 模型屬性。例如，以下是一個常見的模式：

```php
public Post $post;

protected $rules = [
    'post.title' => 'required',
    'post.description' => 'required',
];
```

```html
<input wire:model="post.title">
<input wire:model="post.description">
```

在 Livewire 3 中，直接綁定到 Eloquent 模型已被禁用，建議改為使用個別屬性，或提取[表單物件](/docs/forms#extracting-a-form-object)。

然而，由於這種行為在 Livewire 應用程式中被廣泛依賴，版本 3 通過 `config/livewire.php` 中的配置項目維護對這種行為的支持：

```php
'legacy_model_binding' => true,
```

將 `legacy_model_binding` 設置為 `true`，Livewire 將會像在版本 2 中一樣處理 Eloquent 模型屬性。

## AlpineJS

Livewire 3 預設使用 [AlpineJS](https://alpinejs.dev)。

如果您手動在 Livewire 應用程式中包含 Alpine，您將需要將其移除，以避免 Livewire 的內建版本與之衝突。

### 透過 script 標籤包含 Alpine

如果您透過以下的 script 標籤將 Alpine 包含到您的應用程式中，您可以完全移除它，Livewire 將會載入其內部版本：

```html
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script> <!-- [tl! remove] -->
```

### 透過 script 標籤包含插件

Livewire 3 現在內建以下 Alpine 插件：

* [Anchor](https://alpinejs.dev/plugins/anchor)
* [Collapse](https://alpinejs.dev/plugins/collapse)
* [Focus](https://alpinejs.dev/plugins/focus)
* [Intersect](https://alpinejs.dev/plugins/intersect)
* [Mask](https://alpinejs.dev/plugins/mask)
* [Morph](https://alpinejs.dev/plugins/morph)
* [Persist](https://alpinejs.dev/plugins/persist)

值得留意 [package.json](https://github.com/livewire/livewire/blob/main/package.json) 檔案的變化，因為可能會新增新的 Alpine 插件！

如果您之前通過以下方式在應用程序中包含任何這些內容，應該將它們與 Alpine 的核心一起刪除：

```html
<script defer src="https://cdn.jsdelivr.net/npm/@alpinejs/intersect@3.x.x/dist/cdn.min.js"></script> <!-- [tl! remove:1] -->
<!-- ... -->
```

### 通過腳本標籤訪問 Alpine 全局對象

如果您目前通過腳本標籤訪問 `Alpine` 全局對象，像這樣：

```html
<script>
    document.addEventListener('alpine:init', () => {
        Alpine.data(...)
    })
</script>
```

您可以繼續這樣做，因為 Livewire 內部包含並註冊 Alpine 的全局對象，就像以前一樣。

### 通過 JS 捆綁包含

如果您已經通過 NPM 將 Alpine 或上述任何流行的核心 Alpine 插件之一包含到應用程序的 JavaScript 捆綁包中，像這樣：

```js
// Warning: this is a snippet of the Livewire 2 approach to including Alpine

import Alpine from 'alpinejs'
import intersect from '@alpinejs/intersect'

Alpine.plugin(intersect)

Alpine.start()
```

您可以完全刪除它們，因為 Livewire 默認包含 Alpine 和許多流行的 Alpine 插件。

#### 通過 JS 捆綁包含 Alpine

如果您正在應用程序的 JavaScript 捆綁包中註冊自定義 Alpine 插件或組件，像這樣：

```js
// Warning: this is a snippet of the Livewire 2 approach to including Alpine

import Alpine from 'alpinejs'
import customPlugin from './plugins/custom-plugin'

Alpine.plugin(customPlugin)

Alpine.start()
```

您仍然可以通過將 Livewire 核心 ESM 模塊引入捆綁包並從那裡訪問 `Alpine` 來完成此操作。

要將 Livewire 引入捆綁包，您必須首先禁用 Livewire 的正常 JavaScript 注入，並通過在應用程序的主要佈局中將 `@livewireScripts` 替換為 `@livewireScriptConfig` 來為 Livewire 提供必要的配置：

```blade
    <!-- ... -->

    @livewireScripts <!-- [tl! remove] -->
    @livewireScriptConfig <!-- [tl! add] -->
</body>
```

現在，您可以像這樣將 `Alpine` 和 `Livewire` 引入應用程序的捆綁包：

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';
import customPlugin from './plugins/custom-plugin'

Alpine.plugin(customPlugin)

Livewire.start()
```

請注意，您不再需要調用 `Alpine.start()`。Livewire 將自動啟動 Alpine。

有關更多信息，請參閱我們的[手動捆綁 Livewire 的 JavaScript](/docs/installation#manually-bundling-livewire-and-alpine)文檔。

## `wire:model`

在 Livewire 3 中，`wire:model` 默認是“延遲”的（而不是 `wire:model.defer`）。要實現與 Livewire 2 中 `wire:model` 相同的行為，您必須使用 `wire:model.live`。

以下是您在模板中需要進行的必要替換清單，以保持應用程式的行為一致：

```html
<input wire:model="..."> <!-- [tl! remove] -->
<input wire:model.live="..."> <!-- [tl! add] -->

<input wire:model.defer="..."> <!-- [tl! remove] -->
<input wire:model="..."> <!-- [tl! add] -->

<input wire:model.lazy="..."> <!-- [tl! remove] -->
<input wire:model.blur="..."> <!-- [tl! add] -->
```

## `@entangle`

與 `wire:model` 的更改類似，Livewire 3 默認延遲所有數據綁定。為了匹配這種行為，`@entangle` 也已經更新。

為了使您的應用程式正常運行，請進行以下 `@entangle` 替換：

```blade
@entangle(...) <!-- [tl! remove] -->
@entangle(...).live <!-- [tl! add] -->

@entangle(...).defer <!-- [tl! remove] -->
@entangle(...) <!-- [tl! add] -->
```

## 事件

在 Livewire 2 中，Livewire 有兩種不同的 PHP 方法來觸發事件：

* `emit()`
* `dispatchBrowserEvent()`

Livewire 3 將這兩種方法統一為一個方法：

* `dispatch()`

以下是在 Livewire 3 中調度和監聽事件的基本示例：

```php
// Dispatching...
class CreatePost extends Component
{
    public Post $post;

    public function save()
    {
        $this->dispatch('post-created', postId: $this->post->id);
    }
}

// Listening...
class Dashboard extends Component
{
    #[On('post-created')]
    public function postAdded($postId)
    {
        //
    }
}
```

Livewire 2 到 Livewire 3 的三個主要更改是：

1. `emit()` 已更名為 `dispatch()`（同樣，`emitTo()` 和 `emitSelf()` 現在是 `dispatchTo()` 和 `dispatchSelf()`）
2. `dispatchBrowserEvent()` 已更名為 `dispatch()`
3. 所有事件參數必須命名

欲了解更多信息，請查看新的 [事件文檔頁面](/docs/events)。

這裡是應用於您的應用程式的 "尋找和替換" 差異：

```php
$this->emit('post-created'); // [tl! remove]
$this->dispatch('post-created'); // [tl! add]

$this->emitTo('foo', 'post-created'); // [tl! remove]
$this->dispatch('post-created')->to('foo'); // [tl! add]

$this->emitSelf('post-created'); // [tl! remove]
$this->dispatch('post-created')->self(); // [tl! add]

$this->emit('post-created', $post->id); // [tl! remove]
$this->dispatch('post-created', postId: $post->id); // [tl! add]

$this->dispatchBrowserEvent('post-created'); // [tl! remove]
$this->dispatch('post-created'); // [tl! add]

$this->dispatchBrowserEvent('post-created', ['postId' => $post->id]); // [tl! remove]
$this->dispatch('post-created', postId: $post->id); // [tl! add]
```

```html
<button wire:click="$emit('post-created')">...</button> <!-- [tl! remove] -->
<button wire:click="$dispatch('post-created')">...</button> <!-- [tl! add] -->

<button wire:click="$emit('post-created', 1)">...</button> <!-- [tl! remove] -->
<button wire:click="$dispatch('post-created', { postId: 1 })">...</button> <!-- [tl! add] -->

<button wire:click="$emitTo('foo', post-created', 1)">...</button> <!-- [tl! remove] -->
<button wire:click="$dispatchTo('foo', 'post-created', { postId: 1 })">...</button> <!-- [tl! add] -->

<button x-on:click="$wire.emit('post-created', 1)">...</button> <!-- [tl! remove] -->
<button x-on:click="$dispatch('post-created', { postId: 1 })">...</button> <!-- [tl! add] -->
```

### `emitUp()`

`emitUp` 的概念已完全移除。現在使用瀏覽器事件來調度事件，因此將默認 "冒泡"。

您可以從您的組件中刪除任何 `$this->emitUp(...)` 或 `$emitUp(...)` 的實例。

### 測試事件

Livewire 也已更改事件斷言，以匹配有關調度事件的新統一術語：

```php
Livewire::test(Component::class)->assertEmitted('post-created'); // [tl! remove]
Livewire::test(Component::class)->assertDispatched('post-created'); // [tl! add]

Livewire::test(Component::class)->assertEmittedTo(Foo::class, 'post-created'); // [tl! remove]
Livewire::test(Component::class)->assertDispatchedTo(Foo:class, 'post-created'); // [tl! add]

Livewire::test(Component::class)->assertNotEmitted('post-created'); // [tl! remove]
Livewire::test(Component::class)->assertNotDispatched('post-created'); // [tl! add]

Livewire::test(Component::class)->assertEmittedUp() // [tl! remove]
```

### URL 查詢字串

在先前的 Livewire 版本中，如果將屬性綁定到 URL 的查詢字串，則該屬性值將始終存在於查詢字串中，除非使用 `except` 選項。

在 Livewire 3 中，綁定到查詢字串的所有屬性將只在頁面加載後更改其值後顯示。此默認值消除了對 `except` 選項的需求：

```php
public $search = '';

protected $queryString = [
    'search' => ['except' => ''], // [tl! remove]
    'search', // [tl! add]
];
```

如果您想要恢復到 Livewire 2 的行為，無論其值如何，始終在查詢字串中顯示屬性，您可以使用 `keep` 選項：

```php
public $search = '';

protected $queryString = [
    'search' => ['keep' => true], // [tl! highlight]
];
```

## 分頁

在 Livewire 3 中，分頁系統已經更新，以更好地支持同一組件中的多個分頁器。

### 更新已發佈的分頁視圖

如果您已經發佈了 Livewire 的分頁視圖，您可以在 [GitHub 上的分頁目錄](https://github.com/livewire/livewire/tree/master/src/Features/SupportPagination/views) 中參考新的視圖，並相應地更新您的應用程式。

### 直接訪問 `$this->page`

由於 Livewire 現在支持每個組件多個分頁器，它已從組件類中刪除了 `$page` 屬性，並將其替換為一個 `$paginators` 屬性，該屬性存儲一個分頁器數組：

```php
$this->page = 2; // [tl! remove]
$this->paginators['page'] = 2; // [tl! add]
```

但建議您使用提供的 `getPage` 和 `setPage` 方法來修改和訪問當前頁面：

```php
// Getter...
$this->getPage();

// Setter...
$this->setPage(2);
```

### `wire:click.prefetch`

Livewire 的預取功能 (`wire:click.prefetch`) 已完全刪除。如果您依賴此功能，您的應用程式仍將正常運作，只是在您以前從 `.prefetch` 中受益的情況下，性能略有下降。

```html
<button wire:click.prefetch=""> <!-- [tl! remove] -->
<button wire:click="..."> <!-- [tl! add] -->
```

## 組件類變更

對 Livewire 的基本 `Livewire\Component` 類進行了以下更改，您的應用程式組件可能依賴於這些更改。

### 組件 `$id` 屬性

如果您通過 `$this->id` 直接訪問組件的 ID，您應該改為使用 `$this->getId()`：

```php
$this->id; // [tl! remove]

$this->getId(); // [tl! add]
```

### 方法和屬性名稱重複

PHP 允許您將類屬性和方法使用相同的名稱。在 Livewire 3 中，當通過 `wire:click` 從前端調用方法時，這將導致問題。

建議在元件中使用獨特的名稱來命名所有公共方法和屬性：

```php
public $search = ''; // [tl! remove]

public function search() {
    // ...
}
```

```php
public $query = ''; // [tl! add]

public function search() {
    // ...
}
```

## JavaScript API 變更

### `livewire:load`

在 Livewire 的先前版本中，您可以監聽 `livewire:load` 事件，在 Livewire 初始化頁面之前立即執行 JavaScript 代碼。

在 Livewire 3 中，該事件名稱已更改為 `livewire:init`，以配合 Alpine 的 `alpine:init`：

```js
document.addEventListener('livewire:load', () => {...}) // [tl! remove]
document.addEventListener('livewire:init', () => {...}) // [tl! add]
```

### 頁面過期鉤子

在版本 2 中，Livewire 公開了一個專用的 JavaScript 方法，用於自定義頁面過期行為：`Livewire.onPageExpired()`。為了更強大地直接使用 `request` 鉤子，此方法已被移除：

```js
Livewire.onPageExpired(() => {...}) // [tl! remove]

Livewire.hook('request', ({ fail }) => { // [tl! add:8]
    fail(({ status, preventDefault }) => {
        if (status === 419) {
            preventDefault()

            confirm('Your custom page expiration behavior...')
        }
    })
})
```

### 新的生命週期鉤子

Livewire 3 中許多內部 JavaScript 生命週期鉤子已經改變。

以下是舊鉤子及其新語法的比較，供您在應用程序中查找/替換：

```js
Livewire.hook('component.initialized', (component) => {}) // [tl! remove]
Livewire.hook('component.init', ({ component, cleanup }) => {}) // [tl! add]

Livewire.hook('element.initialized', (el, component) => {}) // [tl! remove]
Livewire.hook('element.init', ({ el, component }) => {}) // [tl! add]

Livewire.hook('element.updating', (fromEl, toEl, component) => {}) // [tl! remove]
Livewire.hook('morph.updating', ({ el, toEl, component }) => {}) // [tl! add]

Livewire.hook('element.updated', (el, component) => {}) // [tl! remove]
Livewire.hook('morph.updated', ({ el, component }) => {}) // [tl! add]

Livewire.hook('element.removed', (el, component) => {}) // [tl! remove]
Livewire.hook('morph.removed', ({ el, component }) => {}) // [tl! add]

Livewire.hook('message.sent', (message, component) => {}) // [tl! remove]
Livewire.hook('message.failed', (message, component) => {}) // [tl! remove]
Livewire.hook('message.received', (message, component) => {}) // [tl! remove]
Livewire.hook('message.processed', (message, component) => {}) // [tl! remove]

Livewire.hook('commit', ({ component, commit, respond, succeed, fail }) => { // [tl! add:14]
    // Equivalent of 'message.sent'

    succeed(({ snapshot, effects }) => {
        // Equivalent of 'message.received'

        queueMicrotask(() => {
            // Equivalent of 'message.processed'
        })
    })

    fail(() => {
        // Equivalent of 'message.failed'
    })
})
```

您可以參考新的[JavaScript 鉤子文件](/docs/javascript)以更全面地了解新的鉤子系統。

## 本地化

如果您的應用程序在 URI 中使用區域設定前綴，例如 `https://example.com/en/...`，Livewire 2 在通過 `https://example.com/en/livewire/update` 進行元件更新時會自動保留此 URL 前綴。

Livewire 3 已停止自動支持此行為。相反，您可以使用 `setUpdateRoute()` 覆蓋 Livewire 的更新端點，以滿足您需要的任何 URI 前綴：

```php
Route::group(['prefix' => LaravelLocalization::setLocale()], function ()
{
    // Your other localized routes...

    Livewire::setUpdateRoute(function ($handle) {
        return Route::post('/livewire/update', $handle);
    });
});
```

有關更多信息，請參考我們關於[配置 Livewire 更新端點](/docs/installation#configuring-livewires-update-endpoint)的文件。
