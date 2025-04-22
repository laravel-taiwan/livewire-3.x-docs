組件是 Livewire 應用程式的構建塊。它們結合狀態和行為，為前端創建可重複使用的 UI 片段。在這裡，我們將介紹如何創建和渲染組件。

## 創建組件

一個 Livewire 組件只是一個擴展 `Livewire\Component` 的 PHP 類別。您可以手動創建組件文件，也可以使用以下 Artisan 命令：

```shell
php artisan make:livewire CreatePost
```

如果您喜歡 kebab-case 命名，您也可以使用它們：

```shell
php artisan make:livewire create-post
```

運行此命令後，Livewire 將在您的應用程式中創建兩個新文件。第一個將是組件的類別：`app/Livewire/CreatePost.php`

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
	public function render()
	{
		return view('livewire.create-post');
	}
}
```

第二個將是組件的 Blade 視圖：`resources/views/livewire/create-post.blade.php`

```blade
<div>
	{{-- ... --}}
</div>
```

您可以使用命名空間語法或點記法在子目錄中創建您的組件。例如，以下命令將在 `Posts` 子目錄中創建一個 `CreatePost` 組件：

```shell
php artisan make:livewire Posts\\CreatePost
php artisan make:livewire posts.create-post
```

### 內聯組件

如果您的組件相當小，您可能希望創建一個 _內聯_ 組件。內聯組件是單文件 Livewire 組件，其視圖模板直接包含在 `render()` 方法中，而不是在單獨的文件中：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
	public function render()
	{
		return <<<'HTML' // [tl! highlight:4]
		<div>
		    {{-- Your Blade template goes here... --}}
		</div>
		HTML;
	}
}
```

您可以通過將 `--inline` 標誌添加到 `make:livewire` 命令來創建內聯組件：

```shell
php artisan make:livewire CreatePost --inline
```

### 省略 render 方法

為了減少組件中的樣板代碼，您可以完全省略 `render()` 方法，Livewire 將使用其自己的底層 `render()` 方法，該方法返回一個具有與您的組件對應的常規名稱的視圖：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    //
}
```

如果上面的組件在頁面上呈現，Livewire 將自動確定應使用 `livewire.create-post` 模板進行呈現。

### 自訂元件樣板

您可以透過執行以下命令來自訂 Livewire 用來產生新元件的檔案（或 _樣板_）：

```shell
php artisan livewire:stubs
```

這將在您的應用程式中建立四個新檔案：

* `stubs/livewire.stub` — 用於產生新元件
* `stubs/livewire.inline.stub` — 用於產生 _內嵌_ 元件
* `stubs/livewire.test.stub` — 用於產生測試檔案
* `stubs/livewire.view.stub` — 用於產生元件視圖

即使這些檔案存在於您的應用程式中，您仍然可以使用 `make:livewire` Artisan 命令，Livewire 將在產生檔案時自動使用您自訂的樣板。

## 設定屬性

Livewire 元件具有用於儲存資料並可以在元件類別和 Blade 視圖中輕鬆存取的屬性。本節討論了如何將屬性添加到元件並在應用程式中使用它的基本知識。

要將屬性添加到 Livewire 元件中，請在元件類別中宣告一個公共屬性。例如，讓我們在 `CreatePost` 元件中創建一個 `$title` 屬性：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    public $title = 'Post title...';

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

### 在視圖中存取屬性

元件屬性會自動提供給元件的 Blade 視圖。您可以使用標準 Blade 語法來參考它。這裡我們將顯示 `$title` 屬性的值：

```blade
<div>
    <h1>標題："{{ $title }}"</h1>
</div>
```

此元件的呈現輸出將是：

```blade
<div>
    <h1>標題："文章標題..."</h1>
</div>
```

### 與視圖分享額外資料

除了從視圖中存取屬性外，您還可以從 `render()` 方法中明確地將資料傳遞給視圖，就像您通常從控制器中做的那樣。當您想要傳遞額外資料而不必先將其存儲為屬性時，這將非常有用，因為屬性具有[特定的性能和安全性問題](/docs/properties#security-concerns)。

要在 `render()` 方法中將資料傳遞給視圖，您可以在視圖實例上使用 `with()` 方法。例如，假設您想要將文章作者的名稱傳遞給視圖。在這種情況下，文章的作者是當前驗證的使用者：

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class CreatePost extends Component
{
    public $title;

    public function render()
    {
        return view('livewire.create-post')->with([
	        'author' => Auth::user()->name,
	    ]);
    }
}
```

現在您可以從元件的 Blade 視圖中訪問 `$author` 屬性：

```blade
<div>
	<h1>Title: {{ $title }}</h1>

	<span>Author: {{ $author }}</span>
</div>
```

### 在 `@foreach` 迴圈中添加 `wire:key`

在 Livewire 模板中使用 `@foreach` 迴圈遍歷數據時，您必須為迴圈渲染的根元素添加一個唯一的 `wire:key` 屬性。

在 Blade 迴圈中沒有 `wire:key` 屬性存在時，Livewire 將無法正確將舊元素與其新位置匹配，當迴圈變化時，這可能導致應用程序中許多難以診斷的問題。

例如，如果您正在遍歷一個帖子數組，您可以將 `wire:key` 屬性設置為帖子的 ID：

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}"> <!-- [tl! highlight] -->
            <!-- ... -->
        </div>
    @endforeach
</div>
```

如果您正在遍歷渲染 Livewire 元件的數組，您可以將鍵設置為元件屬性 `:key` 或在使用 `@livewire` 指令時將鍵作為第三個參數傳遞。

```blade
<div>
    @foreach ($posts as $post)
        <livewire:post-item :$post :key="$post->id">

        @livewire(PostItem::class, ['post' => $post], key($post->id))
    @endforeach
</div>
```

### 將輸入綁定到屬性

Livewire 最強大的功能之一是「數據綁定」：自動將頁面上的表單輸入與屬性同步。

讓我們使用 `wire:model` 指令將 `CreatePost` 元件中的 `$title` 屬性綁定到文本輸入：

```blade
<form>
    <label for="title">Title:</label>

    <input type="text" id="title" wire:model="title"> <!-- [tl! highlight] -->
</form>
```

對文本輸入進行的任何更改都將自動與 Livewire 元件中的 `$title` 屬性同步。

> [!warning] "為什麼我的組件在我輸入時沒有即時更新？"
> 如果您在瀏覽器中嘗試此操作並且對於標題為何不自動更新感到困惑，那是因為 Livewire 只有在提交「操作」時才會更新組件，例如按下提交按鈕，而不是當用戶輸入字段時。這有助於減少網絡請求並提高性能。要啟用用戶輸入時的「即時」更新，您可以改用 `wire:model.live`。[了解更多關於數據綁定的資訊](/docs/properties#data-binding)。

Livewire 屬性非常強大，是一個重要的概念需要理解。欲獲取更多信息，請查看 [Livewire 屬性文件](/docs/properties)。

## 呼叫行為

行為是您的 Livewire 元件內的方法，用於處理使用者互動或執行特定任務。它們通常用於回應頁面上按鈕點擊或表單提交。

要了解更多關於行為的資訊，讓我們在 `CreatePost` 元件中新增一個 `save` 行為：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title;

    public function save() // [tl! highlight:8]
    {
		Post::create([
			'title' => $this->title
		]);

		return redirect()->to('/posts')
			 ->with('status', 'Post created!');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

接下來，讓我們從元件的 Blade 視圖中呼叫 `save` 行為，方法是在 `<form>` 元素中新增 `wire:submit` 指示詞：

```blade
<form wire:submit="save"> <!-- [tl! highlight] -->
    <label for="title">Title:</label>

    <input type="text" id="title" wire:model="title">

	<button type="submit">Save</button>
</form>
```

當點擊 "Save" 按鈕時，Livewire 元件中的 `save()` 方法將被執行，並且您的元件將重新渲染。

要繼續學習有關 Livewire 行為的資訊，請參閱[行為文件](/docs/actions)。

## 渲染元件

在頁面上渲染 Livewire 元件有兩種方式：

1. 將其包含在現有的 Blade 視圖中
2. 直接將其指定為路由的全頁元件

讓我們先談談第一種渲染元件的方式，因為比第二種方式更簡單。

您可以使用 `<livewire:component-name />` 語法在 Blade 模板中包含 Livewire 元件：

```blade
<livewire:create-post />
```

如果元件類別位於 `app/Livewire/` 目錄中的更深層次，您可以使用 `.` 字元來指示目錄的巢狀結構。例如，如果我們假設一個元件位於 `app/Livewire/EditorPosts/CreatePost.php`，我們可以這樣渲染它：

```blade
<livewire:editor-posts.create-post />
```

> [!warning] 您必須使用 kebab-case
> 如上面的片段所示，您必須使用元件名稱的 _kebab-case_ 版本。使用 _StudlyCase_ 版本的名稱 (`<livewire:CreatePost />`) 是無效的，Livewire 將無法識別。

### 傳遞資料至元件

要將外部資料傳遞到 Livewire 元件中，您可以在元件標籤上使用屬性。這在您想要使用特定資料初始化元件時非常有用。

要將初始值傳遞給 `CreatePost` 元件的 `$title` 屬性，您可以使用以下語法：

```blade
<livewire:create-post title="初始標題" />
```

如果您需要將動態值或變數傳遞給組件，您可以在組件屬性中使用 PHP 表達式，方法是在屬性前加上冒號：

```blade
<livewire:create-post :title="$initialTitle" />
```

通過 `mount()` 生命週期鉤子作為方法參數接收傳遞到組件中的數據。在這種情況下，要將 `$title` 參數分配給屬性，您將編寫如下的 `mount()` 方法：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    public $title;

    public function mount($title = null)
    {
        $this->title = $title;
    }

    // ...
}
```

在這個例子中，`$title` 屬性將被初始化為值 "Initial Title"。

您可以將 `mount()` 方法視為類構造函數。它在組件的初始加載時運行，但不會在頁面內的後續請求中運行。您可以在 [生命週期文檔](/docs/lifecycle-hooks) 中了解有關 `mount()` 和其他有用的生命週期鉤子的更多信息。

為了減少組件中的樣板代碼，您可以選擇省略 `mount()` 方法，Livewire 將自動使用與傳遞值匹配的任何屬性來設置您組件上的屬性：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    public $title; // [tl! highlight]

    // ...
}
```

這與在 `mount()` 方法內部分配 `$title` 的效果相同。

> [!warning] 這些屬性默認情況下不是反應式的
> 如果在初始頁面加載後更改外部 `:title="$initialValue"`，則 `$title` 屬性將不會自動更新。這是在使用 Livewire 時常見的困惑點，特別是對於已使用 Vue 或 React 等 JavaScript 框架並假設這些 "參數" 行為類似於這些框架中的 "反應式 props" 的開發人員。但是，不用擔心，Livewire 允許您選擇 [使您的 props 變成反應式](/docs/nesting#reactive-props)。

## 全頁面組件

Livewire 允許您將組件直接分配給 Laravel 應用程序中的路由。這些被稱為 "全頁面組件"。您可以使用它們來構建具有邏輯和視圖的獨立頁面，完全封裝在 Livewire 組件中。

要創建一個全頁面組件，在您的 `routes/web.php` 文件中定義一個路由，並使用 `Route::get()` 方法將組件直接映射到特定 URL。例如，假設您想要在專用路由 `/posts/create` 上呈現 `CreatePost` 組件。

您可以通過將以下行添加到您的 `routes/web.php` 文件來完成此操作：

```php
use App\Livewire\CreatePost;

Route::get('/posts/create', CreatePost::class);
```

現在，當您在瀏覽器中訪問 `/posts/create` 路徑時，`CreatePost` 元件將作為全頁元件呈現。

### 佈局文件

請記住，全頁元件將使用您應用程序的佈局，通常在 `resources/views/components/layouts/app.blade.php` 文件中定義。

如果該文件不存在，您可以運行以下命令來創建它：

```shell
php artisan livewire:layout
```

此命令將生成一個名為 `resources/views/components/layouts/app.blade.php` 的文件。

請確保您在此位置創建了一個 Blade 文件並包含一個 `{{ $slot }}` 佔位符：

```blade
<!-- resources/views/components/layouts/app.blade.php -->

<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? 'Page Title' }}</title>
    </head>
    <body>
        {{ $slot }}
    </body>
</html>
```

#### 全局佈局配置

要在所有元件中使用自定義佈局，您可以在 `config/livewire.php` 中設置 `layout` 鍵為您自定義佈局的路徑，相對於 `resources/views`。例如：

```php
'layout' => 'layouts.app',
```

通過上述配置，Livewire 將在佈局文件 `resources/views/layouts/app.blade.php` 中呈現全頁元件。

#### 每個元件的佈局配置

要為特定元件使用不同的佈局，您可以在元件的 `render()` 方法上方放置 Livewire 的 `#[Layout]` 屬性，將其設置為您自定義佈局的相對視圖路徑：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Layout;
use Livewire\Component;

class CreatePost extends Component
{
	// ...

	#[Layout('layouts.app')] // [tl! highlight]
	public function render()
	{
	    return view('livewire.create-post');
	}
}
```

或者，如果您喜歡，您可以在類聲明上方使用此屬性：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Layout;
use Livewire\Component;

#[Layout('layouts.app')] // [tl! highlight]
class CreatePost extends Component
{
	// ...
}
```

PHP 屬性僅支持文字值。如果您需要傳遞動態值，或者更喜歡此替代語法，您可以在元件的 `render()` 方法中使用流暢的 `->layout()` 方法：

```php
public function render()
{
    return view('livewire.create-post')
	     ->layout('layouts.app'); // [tl! highlight]
}
```

此外，Livewire 支持使用傳統的 Blade 佈局文件與 `@extends`。 

給定以下佈局文件：

```blade
<body>
    @yield('content')
</body>
```

您可以配置 Livewire 使用 `->extends()` 而不是 `->layout()` 來引用它：

```php
public function render()
{
    return view('livewire.show-posts')
        ->extends('layouts.app'); // [tl! highlight]
}
```

如果您需要為組件配置 `@section`，您也可以使用 `->section()` 方法進行配置：

```php
public function render()
{
    return view('livewire.show-posts')
        ->extends('layouts.app')
        ->section('body'); // [tl! highlight]
}
```

### 設置頁面標題

為應用程序中的每個頁面分配唯一的頁面標題對用戶和搜索引擎都很有幫助。

要為完整頁面組件設置自定義頁面標題，首先確保您的佈局文件包含一個動態標題：

```blade
<head>
    <title>{{ $title ?? 'Page Title' }}</title>
</head>
```

接下來，在 Livewire 組件的 `render()` 方法之前，添加 `#[Title]` 屬性並傳遞您的頁面標題：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Title;
use Livewire\Component;

class CreatePost extends Component
{
	// ...

	#[Title('Create Post')] // [tl! highlight]
	public function render()
	{
	    return view('livewire.create-post');
	}
}
```

這將為 `CreatePost` Livewire 組件設置頁面標題。在此示例中，當組件呈現時，頁面標題將為 "Create Post"。

如果您希望，您可以在類聲明之前使用此屬性：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Title;
use Livewire\Component;

#[Title('Create Post')] // [tl! highlight]
class CreatePost extends Component
{
	// ...
}
```

如果您需要傳遞動態標題，例如使用組件屬性的標題，您可以在組件的 `render()` 方法中使用 `->title()` 流暢方法：

```php
public function render()
{
    return view('livewire.create-post')
	     ->title('Create Post'); // [tl! highlight]
}
```

### 設置額外的佈局文件插槽

如果您的[佈局文件](#layout-files)除了 `$slot` 外還有任何具名插槽，您可以在 Blade 視圖中通過在根元素外定義 `<x-slot>` 來設置它們的內容。例如，如果您想要能夠為每個組件單獨設置頁面語言，您可以在佈局文件的開頭 HTML 標籤中添加一個動態 `$lang` 插槽：

```blade
<!-- resources/views/components/layouts/app.blade.php -->

<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', $lang ?? app()->getLocale()) }}"> <!-- [tl! highlight] -->
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? 'Page Title' }}</title>
    </head>
    <body>
        {{ $slot }}
    </body>
</html>
```

然後，在您的組件視圖中，在根元素外定義一個 `<x-slot>` 元素：

```blade
<x-slot:lang>fr</x-slot> // This component is in French <!-- [tl! highlight] -->

<div>
    // French content goes here...
</div>
```


### 訪問路由參數

在使用完整頁面組件時，您可能需要在 Livewire 組件內部訪問路由參數。

為了演示，首先在您的 `routes/web.php` 文件中定義帶有參數的路由：

```php
use App\Livewire\ShowPost;

Route::get('/posts/{id}', ShowPost::class);
```

在這裡，我們定義了一個帶有 `id` 參數的路由，該參數代表一篇文章的 ID。

接下來，更新您的 Livewire 元件以在 `mount()` 方法中接受路由參數：

```php
<?php

namespace App\Livewire;

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public Post $post;

    public function mount($id) // [tl! highlight]
    {
        $this->post = Post::findOrFail($id);
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

在這個例子中，因為參數名稱 `$id` 符合路由參數 `{id}`，如果訪問 `/posts/1` URL，Livewire 將把值 "1" 作為 `$id` 傳遞。

### 使用路由模型繫結

Laravel 的路由模型繫結允許您從路由參數自動解析 Eloquent 模型。

在您的 `routes/web.php` 檔案中定義具有模型參數的路由後：

```php
use App\Livewire\ShowPost;

Route::get('/posts/{post}', ShowPost::class);
```

您現在可以通過元件的 `mount()` 方法接受路由模型參數：

```php
<?php

namespace App\Livewire;

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public Post $post;

    public function mount(Post $post) // [tl! highlight]
    {
        $this->post = $post;
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

Livewire 知道要使用 "路由模型繫結"，因為在 `mount()` 中的 `$post` 參數前面有 `Post` 型別提示。

與之前一樣，您可以通過省略 `mount()` 方法來減少樣板代碼：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post; // [tl! highlight]

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

`$post` 屬性將自動分配給通過路由的 `{post}` 參數綁定的模型。

### 修改回應

在某些情況下，您可能希望修改回應並設置自定義回應標頭。您可以通過在視圖上調用 `response()` 方法並使用閉包來修改回應對象：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Illuminate\Http\Response;

class ShowPost extends Component
{
    public function render()
    {
        return view('livewire.show-post')
            ->response(function(Response $response) {
                $response->header('X-Custom-Header', true);
            });
    }
}
```

## 使用 JavaScript

有許多情況下，內建的 Livewire 和 Alpine 工具不足以在 Livewire 元件內實現您的目標。

幸運的是，Livewire 提供了許多有用的擴展點和工具來與自定義 JavaScript 互動。您可以從 [JavaScript 文件頁面](/docs/javascript) 上詳盡的參考中學習。但現在，這裡有一些在 Livewire 元件內使用自己的 JavaScript 的有用方法。 

### 執行腳本

Livewire 提供了一個有用的 `@script` 指令，當包裹在 `<script>` 元素中時，將在頁面上初始化您的元件時執行給定的 JavaScript。

這裡是一個簡單的 `@script` 範例，使用 JavaScript 的 `setInterval()` 每兩秒刷新您的元件：

```blade
@script
<script>
    setInterval(() => {
        $wire.$refresh()
    }, 2000)
</script>
@endscript
```

您會注意到我們在 `<script>` 內部使用了一個名為 `$wire` 的物件來控制元件。Livewire 會自動將這個物件提供給任何 `@script`。如果您對 `$wire` 不熟悉，您可以在以下文件中了解更多關於 `$wire` 的資訊：
* [從 JavaScript 存取屬性](/docs/properties#accessing-properties-from-javascript)
* [從 JS/Alpine 呼叫 Livewire 動作](/docs/actions#calling-actions-from-alpine)
* [$wire 物件參考](/docs/javascript#the-wire-object)

### 載入資源檔

除了一次性的 `@script` 外，Livewire 還提供了一個有用的 `@assets` 工具，可以輕鬆載入頁面上的任何腳本/樣式相依性。

它還確保提供的資源檔只在每個瀏覽器頁面中載入一次，不像 `@script`，每次初始化該 Livewire 元件的新實例時都會執行。

這裡是使用 `@assets` 載入一個名為 [Pikaday](https://github.com/Pikaday/Pikaday) 的日期選擇庫並在您的元件內使用 `@script` 初始化它的範例：

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

> [!info] 在 Blade 元件中使用 `@verbatim@script@endverbatim` 和 `@verbatim@assets@endverbatim`
> 如果您正在使用 [Blade 元件](https://laravel.com/docs/blade#components) 來提取您標記的部分，您也可以在其中使用 `@verbatim@script@endverbatim` 和 `@verbatim@assets@endverbatim`；即使在同一個 Livewire 元件內有多個 Blade 元件。但是，`@verbatim@script@endverbatim` 和 `@verbatim@assets@endverbatim` 目前僅在 Livewire 元件的上下文中受支援，這意味著如果您完全在 Livewire 之外使用給定的 Blade 元件，這些腳本和資源檔將不會載入頁面。
