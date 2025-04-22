> [!warning] 先熟悉 Livewire
> 在使用 Volt 之前，我們建議您先熟悉標準的基於類別的 Livewire 使用方式。這將使您能夠快速將對 Livewire 的知識轉移到使用 Volt 的功能性 API 撰寫元件。

Volt 是 Livewire 的優雅設計的功能性 API，支持單文件元件，允許元件的 PHP 邏輯和 Blade 模板共存於同一文件中。在幕後，功能性 API 編譯為 Livewire 類別元件並與同一文件中的模板相關聯。

一個簡單的 Volt 元件如下所示：

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

?>

<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

## 安裝

要開始使用，請使用 Composer 套件管理器將 Volt 安裝到您的專案中：

```bash
composer require livewire/volt
```

安裝 Volt 後，您可以執行 `volt:install` Artisan 命令，該命令將把 Volt 的服務提供者文件安裝到您的應用程序中。此服務提供者指定了 Volt 將搜索單文件元件的掛載目錄：

```bash
php artisan volt:install
```

## 創建元件

您可以通過將具有 `.blade.php` 擴展名的文件放置在任何 Volt 掛載目錄中來創建 Volt 元件。默認情況下，`VoltServiceProvider` 會掛載 `resources/views/livewire` 和 `resources/views/pages` 目錄，但您可以在 Volt 服務提供者的 `boot` 方法中自定義這些目錄。

為了方便起見，您可以使用 `make:volt` Artisan 命令來創建新的 Volt 元件：

```bash
php artisan make:volt counter
```

在生成元件時添加 `--test` 指令，將同時生成相應的測試文件。如果您希望相關測試使用 [Pest](https://pestphp.com/)，您應該使用 `--pest` 標誌：

```bash
php artisan make:volt counter --test --pest
```

通過添加 `--class` 指令，它將生成基於類別的 volt 元件。

```bash
php artisan make:volt counter --class
```

## API 風格

通過使用 Volt 的功能性 API，我們可以通過導入的 `Livewire\Volt` 函數來定義 Livewire 元件的邏輯。然後 Volt 將轉換和編譯功能性代碼為傳統的 Livewire 類別，使我們能夠利用 Livewire 的廣泛功能，同時減少樣板代碼。

Volt的API會自動將其使用的閉包綁定到底層元件。因此，在任何時候，動作、計算屬性或監聽器都可以使用`$this`變數參考該元件：

```php
use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

// ...
```

### 基於類別的Volt元件

如果您想要享受Volt的單文件元件功能，同時仍然編寫基於類別的元件，我們已為您準備好。要開始，定義一個匿名類別，該類別擴展`Livewire\Volt\Component`。在類別內部，您可以使用傳統的Livewire語法利用Livewire的所有功能：

```blade
<?php

use Livewire\Volt\Component;

new class extends Component {
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }
} ?>

<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

#### 類別屬性

與典型的Livewire元件一樣，Volt元件支持類別屬性。當使用匿名PHP類別時，類別屬性應在`new`關鍵字之後定義：

```blade
<?php

use Livewire\Attributes\{Layout, Title};
use Livewire\Volt\Component;

new
#[Layout('layouts.guest')]
#[Title('Login')]
class extends Component
{
    public string $name = '';

    // ...
```

#### 提供額外的視圖資料

當使用基於類別的Volt元件時，渲染的視圖是同一文件中存在的模板。如果您需要在每次渲染時將額外的資料傳遞給視圖，您可以使用`with`方法。這些資料將被傳遞到視圖中，除了元件的公共屬性：

```blade
<?php

use Livewire\WithPagination;
use Livewire\Volt\Component;
use App\Models\Post;

new class extends Component {
    use WithPagination;

    public function with(): array
    {
        return [
            'posts' => Post::paginate(10),
        ];
    }
} ?>

<div>
    <!-- ... -->
</div>
```

#### 修改視圖實例

有時，您可能希望直接與視圖實例互動，例如，使用翻譯字串設置視圖的標題。為了實現這一點，您可以在元件上定義一個`rendering`方法：

```blade
<?php

use Illuminate\View\View;
use Livewire\Volt\Component;

new class extends Component {
    public function rendering(View $view): void
    {
        $view->title('Create Post');

        // ...
    }

    // ...
```

## 渲染和掛載元件

與典型的Livewire元件一樣，Volt元件可以使用Livewire的標籤語法或`@livewire` Blade指令來渲染：

```blade
<livewire:user-index :users="$users" />
```

要聲明元件接受的屬性，您可以使用`state`函式：

```php
use function Livewire\Volt\{state};

state('users');

// ...
```

如果需要，您可以通過向`state`函式提供閉包來攔截傳遞給元件的屬性，從而與並修改給定的值：

```php
use function Livewire\Volt\{state};

state(['count' => fn ($users) => count($users)]);
```

`mount` 函數可用於定義 Livewire 元件的 "mount" [生命週期鉤子](/docs/lifecycle-hooks)。提供給元件的參數將被注入此方法。mount 鉤子所需的任何其他參數將由 Laravel 的服務容器解析：

```php
use App\Services\UserCounter;
use function Livewire\Volt\{mount};

mount(function (UserCounter $counter, $users) {
    $counter->store('userCount', count($users));

    // ...
});
```

### 全頁元件

您可以選擇將 Volt 元件呈現為全頁元件，方法是在應用程式的 `routes/web.php` 檔案中定義一個 Volt 路由：

```php
use Livewire\Volt\Volt;

Volt::route('/users', 'user-index');
```

預設情況下，元件將使用 `components.layouts.app` 佈局呈現。您可以使用 `layout` 函數自訂此佈局檔案：

```php
use function Livewire\Volt\{layout, state};

state('users');

layout('components.layouts.admin');

// ...
```

您也可以使用 `title` 函數自訂頁面的標題：

```php
use function Livewire\Volt\{layout, state, title};

state('users');

layout('components.layouts.admin');

title('Users');

// ...
```

如果標題依賴於元件狀態或外部依賴項，您可以將閉包傳遞給 `title` 函數：

```php
use function Livewire\Volt\{layout, state, title};

state('users');

layout('components.layouts.admin');

title(fn () => 'Users: ' . $this->users->count());
```

## 屬性

Volt 屬性，如 Livewire 屬性一樣，可以方便地在視圖中訪問並在 Livewire 更新之間保留。您可以使用 `state` 函數定義屬性：

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

?>

<div>
    {{ $count }}
</div>
```

如果狀態屬性的初始值依賴於外部依賴項，例如資料庫查詢、模型或容器服務，則其解析應該封裝在閉包中。這可以防止在絕對必要時解析值：

```php
use App\Models\User;
use function Livewire\Volt\{state};

state(['count' => fn () => User::count()]);
```

如果狀態屬性的初始值是通過[Laravel Folio](https://github.com/laravel/folio)的路由模型綁定注入的，也應該封裝在閉包中：

```php
use App\Models\User;
use function Livewire\Volt\{state};

state(['user' => fn () => $user]);
```

當然，屬性也可以在不明確指定其初始值的情況下聲明。在這種情況下，它們的初始值將是 `null` 或將根據呈現元件時傳遞的屬性設置：

### 鎖定屬性

Livewire 提供了一個功能，可以通過啟用"鎖定"來保護屬性，從而防止在客戶端進行任何修改。要使用 Volt 實現這一點，只需在要保護的狀態上鏈接 `locked` 方法：

```php
state(['id'])->locked();
```

### 反應式屬性

在使用嵌套組件時，您可能會遇到一種情況，即您需要將父組件的屬性傳遞給子組件，並且當父組件更新屬性時，子組件會[自動更新](/docs/nesting#reactive-props)。

要使用 Volt 實現這一點，您可以在要使其具有反應性的狀態上鏈接 `reactive` 方法：

```php
state(['todos'])->reactive();
```

### 模型化屬性

在您不想使用反應式屬性的情況下，Livewire 提供了一個[可模型化的功能](/docs/nesting#binding-to-child-data-using-wiremodel)，您可以使用 `wire:model` 直接在子組件上共享父組件和子組件之間的狀態。

要使用 Volt 實現這一點，只需在要模型化的狀態上鏈接 `modelable` 方法：

```php
state(['form'])->modelable();
```

### 計算屬性

Livewire 還允許您定義[計算屬性](/docs/computed-properties)，這對於延遲提取組件所需信息非常有用。計算屬性的結果被"記憶化"，或者在單個 Livewire 請求生命週期中被緩存。

要定義計算屬性，您可以使用 `computed` 函數。變量的名稱將決定計算屬性的名稱：

```php
<?php

use App\Models\User;
use function Livewire\Volt\{computed};

$count = computed(function () {
    return User::count();
});

?>

<div>
    {{ $this->count }}
</div>
```

您可以通過在計算屬性定義上鏈接 `persist` 方法來將計算屬性的值持久化到應用程序的快取中：

```php
$count = computed(function () {
    return User::count();
})->persist();
```

默認情況下，Livewire 將計算屬性的值緩存 3600 秒。您可以通過向 `persist` 方法提供所需的秒數來自定義此值：

```php
$count = computed(function () {
    return User::count();
})->persist(seconds: 10);
```

## 操作

Livewire [actions](/docs/actions) 提供了一種方便的方式來監聽頁面互動並在您的元件上調用相應的方法，從而重新渲染元件。通常，操作是在用戶點擊按鈕時調用的。

要使用 Volt 定義 Livewire 操作，您只需要定義一個閉包。包含閉包的變數名將決定操作的名稱：

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

?>

<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

在閉包內，`$this` 變數綁定到底層的 Livewire 元件，使您能夠訪問元件上的其他方法，就像在典型的 Livewire 元件中一樣：

```php
use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = function () {
    $this->dispatch('count-updated');

    //
};
```

您的操作也可以從 Laravel 的服務容器接收引數或依賴：

```php
use App\Repositories\PostRepository;
use function Livewire\Volt\{state};

state(['postId']);

$delete = function (PostRepository $posts) {
    $posts->delete($this->postId);

    // ...
};
```

### 無渲染操作

在某些情況下，您的元件可能聲明一個不執行任何操作的操作，這不會導致元件的渲染 Blade 模板發生變化。如果是這種情況，您可以在 `action` 函數內封裝操作並在其定義上鏈接 `renderless` 方法來[跳過 Livewire 生命週期的渲染階段](/docs/actions#skipping-re-renders)：

```php
use function Livewire\Volt\{action};

$incrementViewCount = action(fn () => $this->viewCount++)->renderless();
```

### 受保護的輔助函式

默認情況下，所有 Volt 操作都是“公共”的，可以由客戶端調用。如果您希望創建一個[僅從您的操作內部訪問的函式](/docs/actions#keep-dangerous-methods-protected-or-private)，您可以使用 `protect` 函數：

```php
use App\Repositories\PostRepository;
use function Livewire\Volt\{protect, state};

state(['postId']);

$delete = function (PostRepository $posts) {
    $this->ensurePostCanBeDeleted();

    $posts->delete($this->postId);

    // ...
};

$ensurePostCanBeDeleted = protect(function () {
    // ...
});
```

## 表單

Livewire 的[表單](/docs/forms)提供了一種方便的方式來處理表單驗證和提交在單個類中。要在 Volt 元件中使用 Livewire 表單，您可以使用 `form` 函數：

```php
<?php

use App\Livewire\Forms\PostForm;
use function Livewire\Volt\{form};

form(PostForm::class);

$save = function () {
    $this->form->store();

    // ...
};

?>

<form wire:submit="save">
    <input type="text" wire:model="form.title">
    @error('form.title') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save</button>
</form>
```

如您所見，`form` 函數接受 Livewire 表單類別的名稱。一旦定義，您可以在元件內透過 `$this->form` 屬性存取該表單。

如果您想要為您的表單使用不同的屬性名稱，您可以將名稱作為 `form` 函數的第二個引數傳遞：

```php
form(PostForm::class, 'postForm');

$save = function () {
    $this->postForm->store();

    // ...
};
```

## 監聽器

Livewire 的全域 [事件系統](/docs/events) 可以讓元件之間進行通訊。如果頁面上存在兩個 Livewire 元件，它們可以透過事件和監聽器進行通訊。在使用 Volt 時，監聽器可以使用 `on` 函數來定義：

```php
use function Livewire\Volt\{on};

on(['eventName' => function () {
    //
}]);
```

如果您需要為事件監聽器分配動態名稱，例如基於已驗證使用者或傳遞給元件的資料，您可以將閉包傳遞給 `on` 函數。這個閉包可以接收任何元件參數，以及透過 Laravel 服務容器解析的額外依賴：

```php
on(fn ($post) => [
    'event-'.$post->id => function () {
        //
    }),
]);
```

為了方便起見，在使用「點」表示法定義監聽器時，也可以參考元件資料：

```php
on(['event-{post.id}' => function () {
    //
}]);
```

## 生命週期鉤子

Livewire 有各種[生命週期鉤子](/docs/lifecycle-hooks)，可用於在元件生命週期的各個時刻執行程式碼。使用 Volt 的便利 API，您可以使用相應的函數定義這些生命週期鉤子：

```php
use function Livewire\Volt\{boot, booted, ...};

boot(fn () => /* ... */);
booted(fn () => /* ... */);
mount(fn () => /* ... */);
hydrate(fn () => /* ... */);
hydrate(['count' => fn () => /* ... */]);
dehydrate(fn () => /* ... */);
dehydrate(['count' => fn () => /* ... */]);
updating(['count' => fn () => /* ... */]);
updated(['count' => fn () => /* ... */]);
```

## 懶載入占位符

在渲染 Livewire 元件時，您可以將 `lazy` 參數傳遞給 Livewire 元件，以延遲其載入，直到初始頁面完全載入。預設情況下，Livewire 會在 DOM 中插入 `<div></div>` 標籤，用於載入元件的位置。

如果您想要自訂在初始頁面載入時顯示在元件占位符中的 HTML，您可以使用 `placeholder` 函數：

```php
use function Livewire\Volt\{placeholder};

placeholder('<div>Loading...</div>');
```

## 確認

Livewire 提供了簡單訪問 Laravel 強大的 [驗證功能](/docs/validation)。使用 Volt 的 API，您可以使用 `rules` 函數定義組件的驗證規則。與傳統的 Livewire 組件一樣，這些規則將應用於您的組件數據，當您調用 `validate` 方法時：

```php
<?php

use function Livewire\Volt\{rules};

rules(['name' => 'required|min:6', 'email' => 'required|email']);

$submit = function () {
    $this->validate();

    // ...
};

?>

<form wire:submit.prevent="submit">
    //
</form>
```

如果您需要動態定義規則，例如基於已驗證用戶或來自數據庫的信息的規則，您可以向 `rules` 函數提供一個閉包：

```php
rules(fn () => [
    'name' => ['required', 'min:6'],
    'email' => ['required', 'email', 'not_in:'.Auth::user()->email]
]);
```

### 錯誤消息和屬性

要修改驗證期間使用的驗證消息或屬性，您可以將 `messages` 和 `attributes` 方法鏈接到您的 `rules` 定義上：

```php
use function Livewire\Volt\{rules};

rules(['name' => 'required|min:6', 'email' => 'required|email'])
    ->messages([
        'email.required' => 'The :attribute may not be empty.',
        'email.email' => 'The :attribute format is invalid.',
    ])->attributes([
        'email' => 'email address',
    ]);
```

## 文件上傳

使用 Volt 時，由於 Livewire 的支持，[上傳和存儲文件](/docs/uploads)變得更加容易。要在您的功能性 Volt 組件上包含 `Livewire\WithFileUploads` 特性，您可以使用 `usesFileUploads` 函數：

```php
use function Livewire\Volt\{state, usesFileUploads};

usesFileUploads();

state(['photo']);

$save = function () {
    $this->validate([
        'photo' => 'image|max:1024',
    ]);

    $this->photo->store('photos');
};
```

## URL 查詢參數

有時在組件狀態更改時，[更新瀏覽器的 URL 查詢參數](/docs/url)是很有用的。在這些情況下，您可以使用 `url` 方法指示 Livewire 將 URL 查詢參數與一個組件狀態同步：

```php
<?php

use App\Models\Post;
use function Livewire\Volt\{computed, state};

state(['search'])->url();

$posts = computed(function () {
    return Post::where('title', 'like', '%'.$this->search.'%')->get();
});

?>

<div>
    <input wire:model.live="search" type="search" placeholder="Search posts by title...">

    <h1>Search Results:</h1>

    <ul>
        @foreach($this->posts as $post)
            <li wire:key="{{ $post->id }}">{{ $post->title }}</li>
        @endforeach
    </ul>
</div>
```

Livewire 還支持的其他 URL 查詢參數選項，例如 URL 查詢參數別名，也可以提供給 `url` 方法：

```php
use App\Models\Post;
use function Livewire\Volt\{state};

state(['page' => 1])->url(as: 'p', history: true, keep: true);

// ...
```

## 分頁

Livewire 和 Volt 也完全支持 [分頁](/docs/pagination)。要在您的功能性 Volt 組件上包含 Livewire 的 `Livewire\WithPagination` 特性，您可以使用 `usesPagination` 函數：

```php
<?php

use function Livewire\Volt\{with, usesPagination};

usesPagination();

with(fn () => ['posts' => Post::paginate(10)]);

?>

<div>
    @foreach ($posts as $post)
        //
    @endforeach

    {{ $posts->links() }}
</div>
```

與 Laravel 一樣，Livewire 的默認分頁視圖使用 Tailwind 類進行樣式設置。如果您在應用程序中使用 Bootstrap，您可以在調用 `usesPagination` 函數時指定所需的主題以啟用 Bootstrap 分頁主題：

```php
usesPagination(theme: 'bootstrap');
```

## 自訂特性和介面

要在您的功能性 Volt 元件上包含任意特性或介面，您可以使用 `uses` 函式：

```php
use function Livewire\Volt\{uses};

use App\Contracts\Sorting;
use App\Concerns\WithSorting;

uses([Sorting::class, WithSorting::class]);
```

## 匿名元件

有時，您可能希望將頁面的一小部分轉換為 Volt 元件，而不將其提取到單獨的檔案中。例如，想像一個返回以下視圖的 Laravel 路由：

```php
Route::get('/counter', fn () => view('pages/counter.blade.php'));
```

視圖的內容是一個典型的 Blade 模板，包括版面定義和區塊。但是，通過將視圖的一部分包裹在 `@volt` Blade 指示詞中，我們可以將該視圖的這部分轉換為一個完全功能的 Volt 元件：

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

?>

<x-app-layout>
    <x-slot name="header">
        Counter
    </x-slot>

    @volt('counter')
        <div>
            <h1>{{ $count }}</h1>
            <button wire:click="increment">+</button>
        </div>
    @endvolt
</x-app-layout>
```

#### 傳遞資料給匿名元件

在呈現包含匿名元件的視圖時，所有提供給視圖的資料也將可用於匿名 Volt 元件：

```php
use App\Models\User;

Route::get('/counter', fn () => view('users.counter', [
    'count' => User::count(),
]));
```

當然，您可以將這些資料聲明為您的 Volt 元件上的 "state"。當從視圖代理到元件的資料初始化狀態時，您只需要聲明狀態變數的名稱。Volt 將使用代理視圖資料自動填充狀態的預設值：

```php
<?php

use function Livewire\Volt\{state};

state('count');

$increment = function () {
    // Store the new count value in the database...

    $this->count++;
};

?>

<x-app-layout>
    <x-slot name="header">
        Initial value: {{ $count }}
    </x-slot>

    @volt('counter')
        <div>
            <h1>{{ $count }}</h1>
            <button wire:click="increment">+</button>
        </div>
    @endvolt
</x-app-layout>
```

## 測試元件

要開始測試一個 Volt 元件，您可以調用 `Volt::test` 方法，提供元件的名稱：

```php
use Livewire\Volt\Volt;

it('increments the counter', function () {
    Volt::test('counter')
        ->assertSee('0')
        ->call('increment')
        ->assertSee('1');
});
```

在測試 Volt 元件時，您可以利用標準 [Livewire 測試 API](/docs/testing) 提供的所有方法。

如果您的 Volt 元件是巢狀的，您可以使用 "點" 表示法來指定要測試的元件：

```php
Volt::test('users.stats')
```

在測試包含匿名 Volt 元件的頁面時，您可以使用 `assertSeeVolt` 方法來斷言元件是否已呈現：

```php
$this->get('/users')
    ->assertSeeVolt('stats');
```
