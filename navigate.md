許多現代網頁應用程式都是以"單頁應用程式"（SPA）的形式建立。在這些應用程式中，應用程式渲染的每個頁面不再需要完整重新載入瀏覽器頁面，避免了在每次請求時重新下載JavaScript和CSS資源的開銷。

*單頁應用程式*的替代方案是*多頁應用程式*。在這些應用程式中，每次使用者點擊連結時，都會請求並在瀏覽器中渲染一個全新的HTML頁面。

雖然大多數PHP應用程式傳統上都是多頁應用程式，但Livewire通過在應用程式中的連結上添加一個簡單的屬性`wire:navigate`，提供了單頁應用程式的體驗。

## 基本用法

讓我們來看一個使用`wire:navigate`的範例。以下是一個典型的Laravel路由文件（`routes/web.php`），其中定義了三個Livewire元件作為路由：

```php
use App\Livewire\Dashboard;
use App\Livewire\ShowPosts;
use App\Livewire\ShowUsers;

Route::get('/', Dashboard::class);

Route::get('/posts', ShowPosts::class);

Route::get('/users', ShowUsers::class);
```

通過在每個頁面的導覽菜單中的每個連結上添加`wire:navigate`，Livewire將阻止標準的連結點擊處理，並用自己更快速的版本替換：

```blade
<nav>
    <a href="/" wire:navigate>Dashboard</a>
    <a href="/posts" wire:navigate>Posts</a>
    <a href="/users" wire:navigate>Users</a>
</nav>
```

以下是當點擊`wire:navigate`連結時發生的情況：

* 使用者點擊連結
* Livewire阻止瀏覽器訪問新頁面
* 取而代之，Livewire在後台請求頁面並在頁面頂部顯示載入進度條
* 當接收到新頁面的HTML時，Livewire將當前頁面的URL、`<title>`標籤和`<body>`內容替換為新頁面的元素

這種技術導致頁面加載時間大大加快 — 往往快兩倍 — 並使應用程式“感覺”像是由JavaScript驅動的單頁應用程式。

## 重新導向

當您的Livewire元件之一將使用者重新導向到應用程式中的另一個URL時，您也可以指示Livewire使用其`wire:navigate`功能來載入新頁面。為此，請將`navigate`參數提供給`redirect()`方法：

```php
return $this->redirect('/posts', navigate: true);
```

現在，Livewire 將取代完整頁面請求以將使用者重新導向至新 URL，而是將目前頁面的內容和 URL 替換為新頁面。

## 預取連結

預設情況下，Livewire 包含一種溫和的策略來在使用者點擊連結之前預取頁面：

* 使用者按下滑鼠按鈕
* Livewire 開始請求頁面
* 他們放開滑鼠按鈕以完成點擊
* Livewire 完成請求並導航至新頁面

令人驚訝的是，使用者按下滑鼠按鈕和放開之間的時間通常足夠載入一半甚至整個頁面的內容。

如果您想要更積極的預取方法，您可以在連結上使用 `.hover` 修飾符：

```blade
<a href="/posts" wire:navigate.hover>文章</a>
```

`.hover` 修飾符將指示 Livewire 在使用者在連結上懸停超過 `60` 毫秒後預取該頁面。

> [!warning] 懸停預取會增加伺服器使用量
> 因為並非所有使用者都會點擊他們懸停的連結，添加 `.hover` 將請求可能不需要的頁面，儘管 Livewire 會在預取頁面之前等待 `60` 毫秒來減輕部分開銷。

## 跨頁面保留元素

有時，有些使用者介面的部分需要在頁面之間保留，例如音訊或影片播放器。例如，在播客應用程式中，使用者可能希望在瀏覽其他頁面時繼續收聽一集節目。

您可以在 Livewire 中使用 `@persist` 指示詞來實現這一點。

通過使用 `@persist` 將元素包裹起來並為其提供名稱，當使用 `wire:navigate` 請求新頁面時，Livewire 將尋找新頁面上具有匹配 `@persist` 的元素。Livewire 不會像平常那樣替換元素，而是在新頁面中使用前一頁中的現有 DOM 元素，保留元素內的任何狀態。

以下是使用 `@persist` 跨頁面保留 `<audio>` 播放器元素的示例：

```blade
@persist('player')
    <audio src="{{ $episode->file }}" controls></audio>
@endpersist
```

如果上述 HTML 出現在當前頁面和下一個頁面上，原始元素將在新頁面上被重複使用。對於音頻播放器，當從一個頁面導航到另一個頁面時，音頻播放不會中斷。

請注意，持久化元素必須放在 Livewire 組件之外。一個常見的做法是將持久化元素放在您的主佈局中，例如 `resources/views/components/layouts/app.blade.php`。

```html
<!-- resources/views/components/layouts/app.blade.php -->

<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? 'Page Title' }}</title>
    </head>
    <body>
        <main>
            {{ $slot }}
        </main>

        @persist('player') <!-- [tl! highlight:2] -->
            <audio src="{{ $episode->file }}" controls></audio>
        @endpersist
    </body>
</html>
```

### 高亮顯示活動鏈接

您可能習慣使用服務器端 Blade 來突出顯示導航欄中當前活動頁面鏈接，如下所示：

```blade
<nav>
    <a href="/" class="@if (request->is('/')) font-bold text-zinc-800 @endif">Dashboard</a>
    <a href="/posts" class="@if (request->is('/posts')) font-bold text-zinc-800 @endif">Posts</a>
    <a href="/users" class="@if (request->is('/users')) font-bold text-zinc-800 @endif">Users</a>
</nav>
```

但是，在持久化元素中，這將無法正常工作，因為它們在頁面加載之間被重複使用。相反，您應該使用 Livewire 的 `wire:current` 指令來突出顯示當前活動鏈接。

只需將您想應用於當前活動鏈接的任何 CSS 類傳遞給 `wire:current`：

```blade
<nav>
    <a href="/dashboard" ... wire:current="font-bold text-zinc-800">Dashboard</a>
    <a href="/posts" ... wire:current="font-bold text-zinc-800">Posts</a>
    <a href="/users" ... wire:current="font-bold text-zinc-800">Users</a>
</nav>
```

現在，當訪問 `/posts` 頁面時，“Posts” 鏈接將比其他鏈接具有更強的字體樣式。

在 [`wire:current` 文件](/docs/wire-current) 中閱讀更多。

### 保留滾動位置

默認情況下，Livewire 在頁面之間來回導航時會保留頁面的滾動位置。但是，有時您可能希望保留在頁面加載之間持久化的個別元素的滾動位置。

為此，您必須將 `wire:scroll` 添加到包含滾動條的元素中，如下所示：

```html
@persist('scrollbar')
<div class="overflow-y-scroll" wire:scroll> <!-- [tl! highlight] -->
    <!-- ... -->
</div>
@endpersist
```

## JavaScript 鉤子

每次頁面導航都會觸發三個生命週期鉤子：

* `livewire:navigate`
* `livewire:navigating`
* `livewire:navigated`

重要的是要注意，這三個鉤子事件在所有類型的導航中都會被分發。這包括使用 `Livewire.navigate()` 進行手動導航，啟用導航進行重定向，以及在瀏覽器中按下返回和前進按鈕。

這裡是一個為每個事件註冊監聽器的示例：

```js
document.addEventListener('livewire:navigate', (event) => {
    // Triggers when a navigation is triggered.

    // Can be "cancelled" (prevent the navigate from actually being performed):
    event.preventDefault()

    // Contains helpful context about the navigation trigger:
    let context = event.detail

    // A URL object of the intended destination of the navigation...
    context.url

    // A boolean [true/false] indicating whether or not this navigation
    // was triggered by a back/forward (history state) navigation...
    context.history

    // A boolean [true/false] indicating whether or not there is
    // cached version of this page to be used instead of
    // fetching a new one via a network round-trip...
    context.cached
})

document.addEventListener('livewire:navigating', () => {
    // Triggered when new HTML is about to swapped onto the page...

    // This is a good place to mutate any HTML before the page
    // is navigated away from...
})

document.addEventListener('livewire:navigated', () => {
    // Triggered as the final step of any page navigation...

    // Also triggered on page-load instead of "DOMContentLoaded"...
})
```

> [!warning] 事件監聽器將在各個頁面之間持續存在
>
> 當您將事件監聽器附加到文件時，當您導航到不同頁面時，它將不會被移除。如果您需要在導航到特定頁面後運行代碼，或者如果您在每個頁面上添加相同的事件監聽器，這可能會導致意外行為。如果您不移除事件監聽器，當它尋找不存在的元素時，可能會在其他頁面上引發異常，或者在每次導航時事件監聽器可能會執行多次。
>
> 一個簡單的方法是在運行後移除事件監聽器，只需將選項 `{once: true}` 作為第三個參數傳遞給 `addEventListener` 函數。
> ```js
> document.addEventListener('livewire:navigated', () => {
>     // ...
> }, { once: true })
> ```

## Manually visiting a new page

In addition to `wire:navigate`, you can manually call the `Livewire.navigate()` method to trigger a visit to a new page using JavaScript:

```html
<script>
    // ...

    Livewire.navigate('/new/url')
</script>

## Using with analytics software

When navigating pages using `wire:navigate` in your app, any `<script>` tags in the `<head>` only evaluate when the page is initially loaded.

This creates a problem for analytics software such as [Fathom Analytics](https://usefathom.com/). These tools rely on a `<script>` snippet being evaluated on every single page change, not just the first.

Tools like [Google Analytics](https://marketingplatform.google.com/about/analytics/) are smart enough to handle this automatically, however, when using Fathom Analytics, you must add `data-spa="auto"` to your script tag to ensure each page visit is tracked properly:

```blade
<head>
    <!-- ... -->

    <!-- Fathom Analytics -->
    @if (! config('app.debug'))
        <script src="https://cdn.usefathom.com/script.js" data-site="ABCDEFG" data-spa="auto" defer></script> <!-- [tl! highlight] -->
    @endif
</head>

## Script evaluation

When navigating to a new page using `wire:navigate`, it _feels_ like the browser has changed pages; however, from the browser's perspective, you are technically still on the original page.

Because of this, styles and scripts are executed normally on the first page, but on subsequent pages, you may have to tweak the way you normally write JavaScript.

Here are a few caveats and scenarios you should be aware of when using `wire:navigate`.

### Don't rely on `DOMContentLoaded`

It's common practice to place JavaScript inside a `DOMContentLoaded` event listener so that the code you want to run only executes after the page has fully loaded.

When using `wire:navigate`, `DOMContentLoaded` is only fired on the first page visit, not subsequent visits.

To run code on every page visit, swap every instance of `DOMContentLoaded` with `livewire:navigated`:

```js
document.addEventListener('DOMContentLoaded', () => { // [tl! remove]
document.addEventListener('livewire:navigated', () => { // [tl! add]
    // ...
})

Now, any code placed inside this listener will be run on the initial page visit, and also after Livewire has finished navigating to subsequent pages.

Listening to this event is useful for things like initializing third-party libraries.

### Scripts in `<head>` are loaded once

If two pages include the same `<script>` tag in the `<head>`, that script will only be run on the initial page visit and not on subsequent page visits.

```blade
<!-- 第一頁 -->
<head>
    <script src="/app.js"></script>
</head>

<!-- 第二頁 -->
<head>
    <script src="/app.js"></script>
</head>

### New `<head>` scripts are evaluated

If a subsequent page includes a new `<script>` tag in the `<head>` that was not present in the `<head>` of the initial page visit, Livewire will run the new `<script>` tag.

In the below example, _page two_ includes a new JavaScript library for a third-party tool. When the user navigates to _page two_, that library will be evaluated.

```blade
<!-- 第一頁 -->
<head>
    <script src="/app.js"></script>
</head>

<!-- 第二頁 -->
<head>
    <script src="/app.js"></script>
    <script src="/third-party.js"></script>
</head>

> [!info] Head assets are blocking
> If you are navigating to a new page that contains an asset like `<script src="...">` in the head tag. That asset will be fetched and processed before the navigation is complete and the new page is swapped in. This might be surprising behavior, but it ensures any scripts that depend on those assets will have immediate access to them.

### Reloading when assets change

It's common practice to include a version hash in an application's main JavaScript file name. This ensures that after deploying a new version of your application, users will receive the fresh JavaScript asset, and not an old version served from the browser's cache.

But, now that you are using `wire:navigate` and each page visit is no longer a fresh browser page load, your users may still be receiving stale JavaScript after deployments.

To prevent this, you may add `data-navigate-track` to a `<script>` tag in `<head>`:

```blade
<!-- 第一頁 -->
<head>
    <script src="/app.js?id=123" data-navigate-track></script>
</head>

<!-- 第二頁 -->
<head>
    <script src="/app.js?id=456" data-navigate-track></script>
</head>

When a user visits _page two_, Livewire will detect a fresh JavaScript asset and trigger a full browser page reload.

If you are using [Laravel's Vite plug-in](https://laravel.com/docs/vite#loading-your-scripts-and-styles) to bundle and serve your assets, Livewire adds `data-navigate-track` to the rendered HTML asset tags automatically. You can continue referencing your assets and scripts like normal:

```blade
<head>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>

Livewire will automatically inject `data-navigate-track` onto the rendered HTML tags.

> [!warning] Only query string changes are tracked
> Livewire will only reload a page if a `[data-navigate-track]` element's query string (`?id="456"`) changes, not the URI itself (`/app.js`).

### Scripts in the `<body>` are re-evaluated

Because Livewire replaces the entire contents of the `<body>` on every new page, all `<script>` tags on the new page will be run:

```blade
<!-- 第一頁 -->
<body>
    <script>
        console.log('Runs on page one')
    </script>
</body>

<!-- 第二頁 -->
<body>
    <script>
        console.log('在第二頁運行')
    </script>
</body>

If you have a `<script>` tag in the body that you only want to be run once, you can add the `data-navigate-once` attribute to the `<script>` tag and Livewire will only run it on the initial page visit:

```blade
<script data-navigate-once>
    console.log('僅在第一頁運行')
</script>

## Customizing the progress bar

When a page takes longer than 150ms to load, Livewire will show a progress bar at the top of the page.

You can customize the color of this bar or disable it all together inside Livewire's config file (`config/livewire.php`):

```php
'navigate' => [
    'show_progress_bar' => false,
    'progress_bar_color' => '#2299dd',
],
```
