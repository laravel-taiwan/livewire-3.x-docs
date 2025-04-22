在 Livewire HQ，我們致力於在您遇到問題之前消除您的路徑上的問題。然而，有時候，有些問題我們無法解決而不引入新問題，而其他時候，有些問題我們無法預料。

以下是您在 Livewire 應用程式中可能遇到的一些常見錯誤和情況。

## 元件不匹配

在與您頁面上的 Livewire 元件互動時，您可能會遇到奇怪的行為或以下類似的錯誤訊息：

```
錯誤：元件已初始化
```

```
錯誤：Livewire 元件缺少快照，ID 為：...
```

您可能會遇到這些訊息的原因有很多，但最常見的原因是忘記在 `@foreach` 迴圈內的元素和元件中添加 `wire:key`。

### 添加 `wire:key`

每當您在 Blade 模板中使用 `@foreach` 之類的迴圈時，您需要在迴圈內的第一個元素的開始標籤中添加 `wire:key`：

```blade
@foreach($posts as $post)
    <div wire:key="{{ $post->id }}"> <!-- [tl! highlight] -->
        ...
    </div>
@endforeach
```

這確保了 Livewire 在迴圈變更時能夠追蹤不同的元素。

對 Livewire 元件在迴圈內也適用：

```blade
@foreach($posts as $post)
    <livewire:show-post :$post :key="$post->id" /> <!-- [tl! highlight] -->
@endforeach
```

然而，這裡有一個您可能沒有考慮到的棘手情況：

當您在 `@foreach` 迴圈內深度嵌套一個 Livewire 元件時，仍然需要為其添加一個鍵。例如：

```blade
@foreach($posts as $post)
    <div wire:key="{{ $post->id }}">
        ...
        <livewire:show-post :$post :key="$post->id" /> <!-- [tl! highlight] -->
        ...
    </div>
@endforeach
```

如果在嵌套的 Livewire 元件上沒有鍵，Livewire 將無法在網路請求之間匹配迴圈中的元件。

#### 添加前綴鍵

您可能遇到的另一個棘手情況是在同一元件內具有重複的鍵。這通常是由於使用模型 ID 作為鍵而導致的。

這裡有一個示例，我們需要為每組鍵添加 `post-` 和 `author-` 前綴以將每組鍵標記為唯一。否則，如果您有具有相同 ID 的 `$post` 和 `$author` 模型，將會發生 ID 衝突：

```blade
<div>
    @foreach($posts as $post)
        <div wire:key="post-{{ $post->id }}">...</div> <!-- [tl! highlight] -->
    @endforeach

    @foreach($authors as $author)
        <div wire:key="author-{{ $author->id }}">...</div> <!-- [tl! highlight] -->
    @endforeach
</div>
```

## 多個 Alpine 實例

在安裝 Livewire 時，您可能會遇到以下類似的錯誤訊息：

```
錯誤：偵測到多個正在運行的 Alpine 實例
```

```
Alpine 表達式錯誤：$wire 未定義
```

如果是這種情況，您可能在同一頁面上運行兩個版本的 Alpine。Livewire 在內部包含了自己的 Alpine 捆綁，因此您必須在應用程式的 Livewire 頁面上移除任何其他版本的 Alpine。

一個常見的情況是將 Livewire 添加到已包含 Alpine 的現有應用程式中。例如，如果您安裝了 Laravel Breeze 起始套件，然後稍後添加了 Livewire，您將遇到這個問題。

解決方法很簡單：移除額外的 Alpine 實例。

### 移除 Laravel Breeze 的 Alpine

如果您正在現有的 Laravel Breeze（Blade + Alpine 版本）中安裝 Livewire，您需要從 `resources/js/app.js` 中移除以下行：

```js
import './bootstrap';

import Alpine from 'alpinejs'; // [tl! remove:4]

window.Alpine = Alpine;

Alpine.start();
```

### 移除 CDN 版本的 Alpine

因為 Livewire 版本 2 及以下不會預設包含 Alpine，您可能已經在版面的 head 中將 Alpine CDN 作為 script 標籤包含進來。在 Livewire v3 中，您可以完全移除這個 CDN，Livewire 將自動為您提供 Alpine：

```html
    ...
    <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script> <!-- [tl! remove] -->
</head>
```

注意：您也可以移除任何額外的 Alpine 插件，因為 Livewire 包含了除了 `@alpinejs/ui` 之外的所有 Alpine 插件。

## 缺少 `@alpinejs/ui`

Livewire 捆綁的 Alpine 版本包含了所有 Alpine 插件，但不包括 `@alpinejs/ui`。如果您正在使用來自 [Alpine Components](https://alpinejs.dev/components) 的無頭元件，而這些元件依賴於此插件，您可能會遇到以下錯誤：

```
未捕獲的 Alpine 錯誤：未提供 x-anchor 元素
```

要解決這個問題，您可以在版面檔案中簡單地將 `@alpinejs/ui` 插件作為 CDN 包含，如下所示：

```html
    ...
    <script defer src="https://unpkg.com/@alpinejs/ui@3.13.7-beta.0/dist/cdn.min.js"></script> <!-- [tl! add] -->
</head>
```

請注意：請確保包含此插件的最新版本，您可以在[任何元件的文件頁面](https://alpinejs.dev/component/headless-dialog/docs)找到。
