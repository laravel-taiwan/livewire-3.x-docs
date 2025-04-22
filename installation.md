Livewire 是 Laravel 的一個套件，因此在安裝和使用 Livewire 之前，您需要啟動並運行一個 Laravel 應用程式。如果您需要幫助設置新的 Laravel 應用程式，請參閱[官方 Laravel 文件](https://laravel.com/docs/installation)。

要安裝 Livewire，請打開您的終端機並導航到 Laravel 應用程式目錄，然後執行以下命令：

```shell
composer require livewire/livewire
```

就是這樣 — 真的。如果您想要更多自定義選項，請繼續閱讀。否則，您可以直接開始使用 Livewire。

> [!warning] `/livewire/livewire.js` 返回 404 狀態碼
> 默認情況下，Livewire 在您的應用程式中公開一個路由以提供其 JavaScript 資源：`/livewire/livewire.js`。對於大多數應用程式來說，這是可以接受的，但是，如果您使用具有自定義配置的 Nginx，則可能會從此端點收到 404。要解決此問題，您可以自行[編譯 Livewire 的 JavaScript 資源](#manually-bundling-livewire-and-alpine)，或者[配置 Nginx 允許此操作](https://benjamincrozat.com/livewire-js-404-not-found)。

## 發佈組態檔案

Livewire 是“零配置”的，這意味著您可以按照慣例使用它，而無需進行任何額外的配置。但是，如果需要，您可以通過運行以下 Artisan 命令來發佈和自定義 Livewire 的組態檔案：

```shell
php artisan livewire:publish --config
```

這將在 Laravel 應用程式的 `config` 目錄中創建一個新的 `livewire.php` 檔案。

## 手動包含 Livewire 的前端資源

默認情況下，Livewire 將其所需的 JavaScript 和 CSS 資源注入到包含 Livewire 元件的每個頁面中。

如果您想要更多控制這種行為，您可以使用以下 Blade 指示詞在頁面上手動包含資源：

```blade
<html>
<head>
	...
	@livewireStyles
</head>
<body>
	...
	@livewireScripts
</body>
</html>
```

通過在頁面上手動包含這些資源，Livewire 將不會自動注入資源。

> [!warning] AlpineJS 與 Livewire 捆綁
> 因為 Alpine 與 Livewire 的 JavaScript 資源捆綁在一起，您必須在希望使用 Alpine 的每個頁面上包含 `@livewireScripts`。即使您在該頁面上沒有使用 Livewire。

雖然很少需要，您可以通過更新應用程式的 `config/livewire.php` 檔案中的 `inject_assets` [組態選項](#publishing-the-configuration-file) 來禁用 Livewire 的自動注入資源行為：

```php
'inject_assets' => false,
```

如果您寧願強制 Livewire 在單個頁面或多個頁面上注入其資源，您可以從當前路由或服務提供者調用以下全域方法。

```php
\Livewire\Livewire::forceAssetInjection();
```

## 配置 Livewire 的更新端點

Livewire 組件中的每次更新都會向伺服器發送網路請求到以下端點：`https://example.com/livewire/update`

對於使用本地化或多租戶的某些應用程式來說，這可能會成為問題。

在這些情況下，您可以按照自己的喜好註冊自己的端點，只要您將其放在 `Livewire::setUpdateRoute()` 內，Livewire 將知道要使用此端點進行所有組件更新：

```php
Livewire::setUpdateRoute(function ($handle) {
	return Route::post('/custom/livewire/update', $handle);
});
```

現在，Livewire 將會將組件更新發送到 `/custom/livewire/update` 而不是使用 `/livewire/update`。

由於 Livewire 允許您註冊自己的更新路由，您可以直接在 `setUpdateRoute()` 內聲明您想要 Livewire 使用的任何額外中介層：

```php
Livewire::setUpdateRoute(function ($handle) {
	return Route::post('/custom/livewire/update', $handle)
        ->middleware([...]); // [tl! highlight]
});
```

## 自訂資源 URL

預設情況下，Livewire 將從以下 URL 提供其 JavaScript 資源：`https://example.com/livewire/livewire.js`。此外，Livewire 將像這樣從 script 標籤引用此資源：

```blade
<script src="/livewire/livewire.js" ...
```

如果您的應用程式由於本地化或多租戶而具有全域路由前綴，您可以註冊 Livewire 內部在擷取其 JavaScript 時應使用的自訂端點。

要使用自訂的 JavaScript 資源端點，您可以在 `Livewire::setScriptRoute()` 內註冊自己的路由：

```php
Livewire::setScriptRoute(function ($handle) {
    return Route::get('/custom/livewire/livewire.js', $handle);
});
```

現在，Livewire 將以以下方式加載其 JavaScript：

```blade
<script src="/custom/livewire/livewire.js" ...
```

## 手動捆綁 Livewire 和 Alpine

預設情況下，Alpine 和 Livewire 是使用 `<script src="livewire.js">` 標籤加載的，這意味著您無法控制這些庫的加載順序。因此，導入和註冊 Alpine 插件，如下面的示例所示，將不再起作用：

```js
// Warning: This snippet demonstrates what NOT to do...

import Alpine from 'alpinejs'
import Clipboard from '@ryangjchandler/alpine-clipboard'

Alpine.plugin(Clipboard)
Alpine.start()
```

為了解決這個問題，我們需要告訴 Livewire 我們希望自己使用 ESM（ECMAScript 模塊）版本並阻止注入 `livewire.js` 腳本標籤。為了實現這一點，我們必須將 `@livewireScriptConfig` 指示詞添加到我們的佈局文件（`resources/views/components/layouts/app.blade.php`）中：

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

當 Livewire 檢測到 `@livewireScriptConfig` 指示詞時，它將避免注入 Livewire 和 Alpine 腳本。如果您正在使用 `@livewireScripts` 指示詞手動加載 Livewire，請確保將其刪除。請確保如果尚未存在，添加 `@livewireStyles` 指示詞。

最後一步是在我們的 `app.js` 文件中導入 Alpine 和 Livewire，這樣我們就可以註冊任何自定義資源，最終啟動 Livewire 和 Alpine：

```js
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';
import Clipboard from '@ryangjchandler/alpine-clipboard'

Alpine.plugin(Clipboard)

Livewire.start()
```

> [!tip] 在 composer update 後重新構建您的資源
> 請確保如果您正在手動捆綁 Livewire 和 Alpine，每次運行 `composer update` 後重新構建您的資源。

> [!warning] 與 Laravel Mix 不相容
> 如果您正在手動捆綁 Livewire 和 AlpineJS，Laravel Mix 將無法正常工作。相反，我們建議您[切換到 Vite](https://laravel.com/docs/vite)。

## 發佈 Livewire 的前端資源

> [!warning] 發佈資源並非必要
> 發佈 Livewire 的資源對於 Livewire 的運行並非必要。只有在您有特定需求時才這樣做。

如果您希望 JavaScript 資源由您的 Web 伺服器提供而不是通過 Laravel，請使用 `livewire:publish` 命令：

```bash
php artisan livewire:publish --assets
```

為了保持資源檔最新並避免未來更新時出現問題，我們強烈建議您將以下指令添加到您的 composer.json 檔案中：

```json
{
    "scripts": {
        "post-update-cmd": [
            // Other scripts
            "@php artisan vendor:publish --tag=livewire:assets --ansi --force"
        ]
    }
}
```
