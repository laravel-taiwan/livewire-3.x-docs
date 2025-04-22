屬性在 Livewire 元件內部存儲和管理資料。它們被定義為元件類別上的公共屬性，可以在伺服器端和客戶端上訪問和修改。

## 初始化屬性

您可以在元件的 `mount()` 方法中為屬性設置初始值。

考慮以下示例：

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class TodoList extends Component
{
    public $todos = [];

    public $todo = '';

    public function mount()
    {
        $this->todos = Auth::user()->todos; // [tl! highlight]
    }

    // ...
}
```

在這個示例中，我們定義了一個空的 `todos` 陣列，並將其初始化為已驗證使用者現有的 todos。現在，當元件首次呈現時，資料庫中的所有現有 todos 都會顯示給使用者。

## 批量賦值

有時在 `mount()` 方法中初始化許多屬性可能會感覺冗長。為了幫助解決這個問題，Livewire 提供了一種方便的方式來通過 `fill()` 方法一次性賦值多個屬性。通過傳遞屬性名稱和它們對應值的關聯陣列，您可以同時設置多個屬性，從而減少在 `mount()` 中重複的程式碼行數。

例如：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class UpdatePost extends Component
{
    public $post;

    public $title;

    public $description;

    public function mount(Post $post)
    {
        $this->post = $post;

        $this->fill( // [tl! highlight]
            $post->only('title', 'description'), // [tl! highlight]
        ); // [tl! highlight]
    }

    // ...
}
```

因為 `$post->only(...)` 返回一個基於您傳入的名稱的模型屬性和值的關聯陣列，所以 `$title` 和 `$description` 屬性將被初始設置為資料庫中 `$post` 模型的 `title` 和 `description`，而無需逐個設置。

## 資料綁定

Livewire 通過 `wire:model` HTML 屬性支持雙向資料綁定。這使您可以輕鬆同步元件屬性和 HTML 輸入之間的資料，保持用戶界面和元件狀態同步。

讓我們使用 `wire:model` 指示詞將 `TodoList` 元件中的 `$todo` 屬性綁定到基本輸入元素：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class TodoList extends Component
{
    public $todos = [];

    public $todo = '';

    public function add()
    {
        $this->todos[] = $this->todo;

        $this->todo = '';
    }

    // ...
}
```

```blade
<div>
    <input type="text" wire:model="todo" placeholder="Todo..."> <!-- [tl! highlight] -->

    <button wire:click="add">Add Todo</button>

    <ul>
        @foreach ($todos as $todo)
            <li>{{ $todo }}</li>
        @endforeach
    </ul>
</div>
```

在上面的示例中，當單擊“新增 Todo”按鈕時，文字輸入的值將與伺服器上的 `$todo` 屬性同步。

這僅僅是 `wire:model` 的一部分。要深入了解資料綁定，請查看我們有關[表單](/docs/forms)的文件。

## 重置屬性

有時候，在使用者執行動作後，您可能需要將屬性重置回初始狀態。在這些情況下，Livewire 提供了一個 `reset()` 方法，該方法接受一個或多個屬性名稱並將它們的值重置為初始狀態。

在下面的示例中，我們可以通過使用 `$this->reset()` 來避免程式碼重複，以在點擊“新增待辦事項”按鈕後重置 `todo` 欄位：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class ManageTodos extends Component
{
    public $todos = [];

    public $todo = '';

    public function addTodo()
    {
        $this->todos[] = $this->todo;

        $this->reset('todo'); // [tl! highlight]
    }

    // ...
}
```

在上面的示例中，當使用者點擊“新增待辦事項”後，剛剛新增的待辦事項的輸入欄位將被清除，讓使用者可以輸入新的待辦事項。

> [!warning] `reset()` 不適用於在 `mount()` 中設置的值
> `reset()` 將屬性重置為 `mount()` 方法調用之前的狀態。如果您在 `mount()` 中將屬性初始化為不同的值，則需要手動重置該屬性。

## 提取屬性

或者，您可以使用 `pull()` 方法來在一個操作中重置並檢索值。

以下是與上面相同的示例，但使用 `pull()` 簡化：

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class ManageTodos extends Component
{
    public $todos = [];

    public $todo = '';

    public function addTodo()
    {
        $this->todos[] = $this->pull('todo'); // [tl! highlight]
    }

    // ...
}
```

上面的示例正在提取單個值，但 `pull()` 也可以用於重置和檢索（作為鍵值對）所有或某些屬性：

```php
// The same as $this->all() and $this->reset();
$this->pull();

// The same as $this->only(...) and $this->reset(...);
$this->pull(['title', 'content']);
```

## 支援的屬性類型

由於 Livewire 在管理伺服器請求之間的元件資料時採用獨特的方法，因此 Livewire 支援有限的屬性類型。

Livewire 中的每個屬性都在請求之間序列化或“脫水”，然後從 JSON 轉換回 PHP 以供下一個請求使用。

這種雙向轉換過程具有某些限制，限制了 Livewire 可以處理的屬性類型。

### 原始類型

Livewire 支援原始類型，如字串、整數等。這些類型可以輕鬆地在 JSON 和 PHP 之間轉換，使它們成為 Livewire 元件中屬性的理想選擇。

Livewire 支援以下原始屬性類型：`Array`、`String`、`Integer`、`Float`、`Boolean` 和 `Null`。 

--- 

*本翻譯遵循嚴格的翻譯規則，請務必查看原始文件以獲取更多詳細資訊。*

```php
class TodoList extends Component
{
    public $todos = []; // Array

    public $todo = ''; // String

    public $maxTodos = 10; // Integer

    public $showTodos = false; // Boolean

    public $todoFilter; // Null
}
```

### 常見的 PHP 類型

除了原始類型外，Livewire 還支持 Laravel 應用程序中常用的 PHP 物件類型。但是，重要的是要注意這些類型將被 _脫水_ 成 JSON，並在每個請求中 _水合_ 回 PHP。這意味著屬性可能無法保留運行時值，如閉包。此外，物件的信息，如類名，可能會暴露給 JavaScript。

支持的 PHP 類型：
| 類型 | 完整類名 |
|------|-----------------|
| BackedEnum | `BackedEnum` |
| Collection | `Illuminate\Support\Collection` |
| Eloquent Collection | `Illuminate\Database\Eloquent\Collection` |
| Model | `Illuminate\Database\Eloquent\Model` |
| DateTime | `DateTime` |
| Carbon | `Carbon\Carbon` |
| Stringable | `Illuminate\Support\Stringable` |

> [!warning] Eloquent 集合和模型
> 在 Livewire 屬性中存儲 Eloquent 集合和模型時，額外的查詢約束，如 select(...)，將不會在後續請求中重新應用。
>
> 詳情請參見 [Eloquent 約束在請求之間不會保留](#eloquent-constraints-arent-preserved-between-requests)。

這裡有一個快速示例，展示如何將屬性設置為這些不同類型：

```php
public function mount()
{
    $this->todos = collect([]); // Collection

    $this->todos = Todos::all(); // Eloquent Collection

    $this->todo = Todos::first(); // Model

    $this->date = new DateTime('now'); // DateTime

    $this->date = new Carbon('now'); // Carbon

    $this->todo = str(''); // Stringable
}
```

### 支持自定義類型

Livewire 允許您的應用程序通過兩種強大的機制來支持自定義類型：

* Wireables
* Synthesizers

Wireables 對於大多數應用程序來說是簡單易用的，因此我們將在下面探討它們。如果您是高級用戶或套件作者，想要更靈活性，[Synthesizers 是更好的選擇](/docs/synthesizers)。

#### Wireables

Wireables 是您的應用程序中實現 `Wireable` 接口的任何類。

例如，假設您的應用程序中有一個 `Customer` 物件，其中包含有關客戶的主要數據：

```php
class Customer
{
    protected $name;
    protected $age;

    public function __construct($name, $age)
    {
        $this->name = $name;
        $this->age = $age;
    }
}
```

嘗試將此類的實例設置為 Livewire 組件屬性將導致錯誤，告訴您 `Customer` 屬性類型不受支持：```

然而，您可以通過實現 `Wireable` 介面並在您的類別中添加 `toLivewire()` 和 `fromLivewire()` 方法來解決這個問題。這些方法告訴 Livewire 如何將此類型的屬性轉換為 JSON，然後再轉換回來：

現在您可以自由地在 Livewire 元件上設置 `Customer` 物件，Livewire 將知道如何將這些物件轉換為 JSON，然後再轉換回 PHP。

正如之前提到的，如果您想更全面且更強大地支持類型，Livewire 提供了 Synthesizers，這是 Livewire 用於處理不同屬性類型的高級內部機制。[了解更多關於 Synthesizers](/docs/synthesizers)。

## 從 JavaScript 存取屬性

由於 Livewire 屬性也可以通過 JavaScript 在瀏覽器中使用，您可以從 [AlpineJS](https://alpinejs.dev/) 存取並操作它們的 JavaScript 表示。

Alpine 是 Livewire 隨附的輕量級 JavaScript 函式庫。Alpine 提供了一種在 Livewire 元件中輕鬆構建輕量級交互的方式，而無需進行完整的伺服器往返。

在內部，Livewire 的前端是建立在 Alpine 之上的。事實上，每個 Livewire 元件實際上都是 Alpine 元件的底層實現。這意味著您可以自由地在 Livewire 元件中使用 Alpine。

本頁其餘部分假設您對 Alpine 有基本的了解。如果您對 Alpine 不熟悉，[請查看 Alpine 文件](https://alpinejs.dev/docs)。

### 存取屬性

Livewire 將一個神奇的 `$wire` 物件暴露給 Alpine。您可以從 Livewire 元件內的任何 Alpine 表達式中存取 `$wire` 物件。

`$wire` 物件可以被視為您的 Livewire 元件的 JavaScript 版本。它具有與您的元件的 PHP 版本相同的所有屬性和方法，但還包含一些專用方法來在模板中執行特定功能。

例如，我們可以使用 `$wire` 來顯示 `todo` 輸入欄位的即時字元計數：

```blade
<div>
    <input type="text" wire:model="todo">

    Todo character length: <h2 x-text="$wire.todo.length"></h2>
</div>
```

當用戶在欄位中輸入時，將即時顯示並更新當前待辦事項的字符長度，而無需向伺服器發送網路請求。

如果您喜歡，您可以使用更明確的 `.get()` 方法來完成相同的事情：

```blade
<div>
    <input type="text" wire:model="todo">

    Todo character length: <h2 x-text="$wire.get('todo').length"></h2>
</div>
```

### 操作屬性

同樣地，您可以使用 `$wire` 在 JavaScript 中操作 Livewire 元件的屬性。

例如，讓我們在 `TodoList` 元件中添加一個 "清除" 按鈕，讓用戶僅使用 JavaScript 重置輸入欄位：

```blade
<div>
    <input type="text" wire:model="todo">

    <button x-on:click="$wire.todo = ''">Clear</button>
</div>
```

用戶點擊 "清除" 後，輸入將被重置為空字符串，而無需向伺服器發送網路請求。

在後續請求中，伺服器端的 `$todo` 值將被更新並同步。

如果您喜歡，您也可以使用更明確的 `.set()` 方法來在客戶端設置屬性。但是，您應該注意，使用 `.set()` 默認會立即觸發網路請求並將狀態與伺服器同步。如果需要這樣做，那麼這是一個很好的 API：

```blade
<button x-on:click="$wire.set('todo', '')">清除</button>
```

為了在不向伺服器發送網路請求的情況下更新屬性，您可以傳遞第三個布爾參數。這將延遲網路請求，並在後續請求中，在伺服器端同步狀態：
```blade
<button x-on:click="$wire.set('todo', '', false)">清除</button>
```

## 安全性考量

雖然 Livewire 屬性是一個強大的功能，但在使用它們之前，您應該了解一些安全性考量。

簡而言之，始終將公共屬性視為用戶輸入 — 就像它們是來自傳統端點的請求輸入一樣。基於這一點，重要的是在將屬性持久化到資料庫之前驗證和授權屬性 — 就像在控制器中處理請求輸入時一樣。

### 不要信任屬性值

為了演示忽略授權和驗證屬性如何在應用程式中引入安全漏洞，以下 `UpdatePost` 元件容易受到攻擊：

一開始看起來，這個元件可能看起來完全正常。但是，讓我們看看攻擊者如何使用這個元件在您的應用程式中執行未經授權的操作。

因為我們將文章的 `id` 存儲為元件上的公共屬性，就像 `title` 和 `content` 屬性一樣，它可以在客戶端被操縱。

我們沒有使用 `wire:model="id"` 寫入輸入並不重要。一個惡意用戶可以輕鬆地使用他們的瀏覽器開發工具更改視圖為以下內容：

```blade
<form wire:submit="update">
    <input type="text" wire:model="id"> <!-- [tl! highlight] -->
    <input type="text" wire:model="title">
    <input type="text" wire:model="content">

    <button type="submit">Update</button>
</form>
```

現在，惡意用戶可以將 `id` 輸入更新為不同文章模型的 ID。當表單提交並調用 `update()` 時，`Post::findOrFail()` 將返回並更新一篇用戶不是擁有者的文章。

為了防止這種攻擊，我們可以使用以下一種或兩種策略：

* 授權輸入
* 鎖定屬性以防止更新

#### 授權輸入

因為 `$id` 可以像在控制器中一樣在客戶端被操縱，我們可以使用 [Laravel 的授權](https://laravel.com/docs/authorization) 來確保當前用戶可以更新文章：

```php
public function update()
{
    $post = Post::findOrFail($this->id);

    $this->authorize('update', $post); // [tl! highlight]

    $post->update(...);
}
```

如果一個惡意用戶變更了 `$id` 屬性，添加的授權將捕捉到並拋出錯誤。

#### 鎖定屬性

Livewire 還允許您“鎖定”屬性以防止在客戶端修改屬性。您可以使用 `#[Locked]` 屬性來從客戶端操縱中“鎖定”屬性：

```php
use Livewire\Attributes\Locked;
use Livewire\Component;

class UpdatePost extends Component
{
    #[Locked] // [tl! highlight]
    public $id;

    // ...
}
```

現在，如果用戶嘗試在前端修改 `$id`，將拋出錯誤。

通過使用 `#[Locked]`，您可以假設此屬性在您元件類別之外的任何地方都沒有被操縱。

有關鎖定屬性的更多信息，請參考 [鎖定屬性文件](/docs/locked)。

#### Eloquent 模型和鎖定

當將 Eloquent 模型分配給 Livewire 元件屬性時，Livewire 將自動鎖定屬性並確保 ID 不會更改，因此您可以免受這類攻擊的影響：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class UpdatePost extends Component
{
    public Post $post; // [tl! highlight]
    public $title;
    public $content;

    public function mount(Post $post)
    {
        $this->post = $post;
        $this->title = $post->title;
        $this->content = $post->content;
    }

    public function update()
    {
        $this->post->update([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        session()->flash('message', 'Post updated successfully!');
    }

    public function render()
    {
        return view('livewire.update-post');
    }
}
```

### 屬性將系統資訊暴露給瀏覽器

另一個重要的事情要記住是，Livewire 屬性在傳送到瀏覽器之前會被序列化或“脫水化”。這意味著它們的值會被轉換為一種可以通過網路傳送並被 JavaScript 理解的格式。這種格式可以將關於應用程式的資訊暴露給瀏覽器，包括屬性的名稱和類別名稱。

例如，假設您有一個定義了名為 `$post` 的公共屬性的 Livewire 元件。這個屬性包含來自您的資料庫的 `Post` 模型的一個實例。在這種情況下，傳送到瀏覽器的這個屬性的脫水化值可能看起來像這樣：

```json
{
    "type": "model",
    "class": "App\Models\Post",
    "key": 1,
    "relationships": []
}
```

正如您所看到的，`$post` 屬性的脫水化值包括模型的類別名稱（`App\Models\Post`），以及已經被急切載入的 ID 和任何關聯。

如果您不想暴露模型的類別名稱，您可以使用 Laravel 的“morphMap”功能從服務提供者中為模型類別名稱分配別名：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Eloquent\Relations\Relation;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Relation::morphMap([
            'post' => 'App\Models\Post',
        ]);
    }
}
```

現在，當 Eloquent 模型被“脫水化”（序列化）時，原始的類別名稱將不會被暴露，只會顯示“post”別名：

```json
{
    "type": "model",
    "class": "App\Models\Post", // [tl! remove]
    "class": "post", // [tl! add]
    "key": 1,
    "relationships": []
}
```

### Eloquent 約束在請求之間不會被保留

通常情況下，Livewire 能夠在請求之間保留並重新創建伺服器端屬性；然而，在某些情況下，保留值在請求之間是不可能的。

例如，當將 Eloquent 集合存儲為 Livewire 屬性時，額外的查詢約束如 `select(...)` 將不會在後續請求中重新應用。

為了演示，考慮以下 `ShowTodos` 元件，其中對 `Todos` Eloquent 集合應用了 `select()` 約束：

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class ShowTodos extends Component
{
    public $todos;

    public function mount()
    {
        $this->todos = Auth::user()
            ->todos()
            ->select(['title', 'content']) // [tl! highlight]
            ->get();
    }

    public function render()
    {
        return view('livewire.show-todos');
    }
}
```

當此元件初始加載時，`$todos` 屬性將設置為用戶的待辦事項的 Eloquent 集合；然而，只有每個資料庫行的 `title` 和 `content` 字段會被查詢並載入到每個模型中。

當 Livewire 在後續請求中將此屬性的 JSON 資料重新轉換為 PHP 時，選擇約束將會遺失。

為了確保 Eloquent 查詢的完整性，我們建議您改用[計算屬性](/docs/computed-properties)來替代屬性。

計算屬性是您元件中標記有 `#[Computed]` 屬性的方法。它們可以被當作動態屬性來存取，並不會被存儲為元件狀態的一部分，而是在需要時即時評估。

以下是使用計算屬性重新撰寫上述範例：

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

class ShowTodos extends Component
{
    #[Computed] // [tl! highlight]
    public function todos()
    {
        return Auth::user()
            ->todos()
            ->select(['title', 'content'])
            ->get();
    }

    public function render()
    {
        return view('livewire.show-todos');
    }
}
```

這裡是如何從 Blade 視圖中存取這些 _todos_：

```blade
<ul>
    @foreach ($this->todos as $todo)
        <li>{{ $todo }}</li>
    @endforeach
</ul>
```

請注意，在您的視圖中，您只能像這樣在 `$this` 物件上存取計算屬性：`$this->todos`。

您也可以在類別內部從 `$todos` 存取。例如，如果您有一個 `markAllAsComplete()` 行為：

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

class ShowTodos extends Component
{
    #[Computed]
    public function todos()
    {
        return Auth::user()
            ->todos()
            ->select(['title', 'content'])
            ->get();
    }

    public function markAllComplete() // [tl! highlight:3]
    {
        $this->todos->each->complete();
    }

    public function render()
    {
        return view('livewire.show-todos');
    }
}
```

您可能會想為什麼不直接在需要時直接調用 `$this->todos()` 方法呢？為什麼要一開始就使用 `#[Computed]`？

原因在於計算屬性具有性能優勢，因為它們在單個請求期間的第一次使用後會自動被快取。這意味著您可以在元件內部自由存取 `$this->todos`，並確保實際方法只會被調用一次，這樣您就不會在同一請求中多次運行昂貴的查詢。

欲瞭解更多資訊，請參閱[計算屬性文件](/docs/computed-properties)。
