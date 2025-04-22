因為表單是大多數網頁應用程式的支柱，Livewire 提供了許多有用的工具來建立它們。從處理簡單的輸入元素到複雜的事情，如即時驗證或檔案上傳，Livewire 提供了簡單、有文件記錄的工具，讓您的生活更輕鬆，並讓您的使用者感到愉快。

讓我們開始吧。

## 提交表單

讓我們首先看一下在 `CreatePost` 元件中的一個非常簡單的表單。這個表單將有兩個簡單的文字輸入框和一個提交按鈕，以及後端的一些程式碼來管理表單的狀態和提交：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
        Post::create(
            $this->only(['title', 'content'])
        );

        session()->flash('status', 'Post successfully updated.');

        return $this->redirect('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```blade
<form wire:submit="save">
    <input type="text" wire:model="title">

    <input type="text" wire:model="content">

    <button type="submit">Save</button>
</form>
```

正如您所看到的，我們在上面的表單中使用 `wire:model` 來「綁定」公共的 `$title` 和 `$content` 屬性。這是 Livewire 最常用且功能強大的功能之一。

除了綁定 `$title` 和 `$content`，我們還使用 `wire:submit` 來捕獲當點擊「儲存」按鈕時的 `submit` 事件，並調用 `save()` 動作。這個動作將把表單輸入持久化到資料庫中。

在資料庫中創建新文章後，我們將用戶重新導向到 `ShowPosts` 元件頁面，並向他們顯示一條「快閃」訊息，告訴他們新文章已經創建。

### 添加驗證

為了避免存儲不完整或危險的使用者輸入，大多數表單需要某種形式的輸入驗證。

Livewire 讓驗證表單變得非常簡單，只需在要驗證的屬性上方添加 `#[Validate]` 屬性即可。

一旦一個屬性有一個附加了 `#[Validate]` 屬性的標記，驗證規則將應用於該屬性的值，每次在伺服器端更新時都會進行驗證。

讓我們在 `CreatePost` 元件中的 `$title` 和 `$content` 屬性中添加一些基本的驗證規則：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Validate; // [tl! highlight]
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required')] // [tl! highlight]
    public $title = '';

    #[Validate('required')] // [tl! highlight]
    public $content = '';

    public function save()
    {
        $this->validate(); // [tl! highlight]

        Post::create(
            $this->only(['title', 'content'])
        );

        return $this->redirect('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

我們還將修改 Blade 模板以在頁面上顯示任何驗證錯誤。

```blade
<form wire:submit="save">
    <input type="text" wire:model="title">
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror <!-- [tl! highlight] -->
    </div>

    <input type="text" wire:model="content">
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror <!-- [tl! highlight] -->
    </div>

    <button type="submit">Save</button>
</form>
```

現在，如果用戶嘗試提交表單而沒有填寫任何字段，他們將看到驗證訊息，告訴他們在保存文章之前需要填寫哪些字段。

Livewire 提供了更多的驗證功能。欲瞭解更多資訊，請參閱我們的[驗證專用文件頁面](/docs/validation)。

### 提取表單物件

如果您正在處理一個大型表單，並且希望將其所有屬性、驗證邏輯等提取到一個獨立的類別中，Livewire 提供了表單物件。

表單物件允許您在組件之間重複使用表單邏輯，並提供了一種很好的方式，通過將所有與表單相關的代碼分組到一個獨立的類別中，來保持您的組件類別更加清晰。

您可以手動創建一個表單類別，或者使用方便的 artisan 命令：

```shell
php artisan livewire:form PostForm
```

上述命令將創建一個名為 `app/Livewire/Forms/PostForm.php` 的文件。

讓我們重寫 `CreatePost` 組件以使用 `PostForm` 類：

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';
}
```

```php
<?php

namespace App\Livewire;

use App\Livewire\Forms\PostForm;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public PostForm $form; // [tl! highlight]

    public function save()
    {
        $this->validate();

        Post::create(
            $this->form->only(['title', 'content']) // [tl! highlight]
        );

        return $this->redirect('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```blade
<form wire:submit="save">
    <input type="text" wire:model="form.title">
    <div>
        @error('form.title') <span class="error">{{ $message }}</span> @enderror
    </div>

    <input type="text" wire:model="form.content">
    <div>
        @error('form.content') <span class="error">{{ $message }}</span> @enderror
    </div>

    <button type="submit">Save</button>
</form>
```

如果您希望，您也可以將發文創建邏輯提取到表單物件中，如下所示：

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    public function store() // [tl! highlight:5]
    {
        $this->validate();

        Post::create($this->only(['title', 'content']));
    }
}
```

現在您可以從組件中調用 `$this->form->store()`：

```php
class CreatePost extends Component
{
    public PostForm $form;

    public function save()
    {
        $this->form->store(); // [tl! highlight]

        return $this->redirect('/posts');
    }

    // ...
}
```

如果您希望將此表單物件用於創建和更新表單，您可以輕鬆地適應它以處理這兩種用例。

以下是使用相同表單物件為 `UpdatePost` 組件並填充初始數據的方式：

```php
<?php

namespace App\Livewire;

use App\Livewire\Forms\PostForm;
use Livewire\Component;
use App\Models\Post;

class UpdatePost extends Component
{
    public PostForm $form;

    public function mount(Post $post)
    {
        $this->form->setPost($post);
    }

    public function save()
    {
        $this->form->update();

        return $this->redirect('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;
use App\Models\Post;

class PostForm extends Form
{
    public ?Post $post;

    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    public function setPost(Post $post)
    {
        $this->post = $post;

        $this->title = $post->title;

        $this->content = $post->content;
    }

    public function store()
    {
        $this->validate();

        Post::create($this->only(['title', 'content']));
    }

    public function update()
    {
        $this->validate();

        $this->post->update(
            $this->only(['title', 'content'])
        );
    }
}
```

如您所見，我們在 `PostForm` 物件中添加了一個 `setPost()` 方法，以選擇性地允許填充表單以及將發文存儲在表單物件中以供以後使用。我們還添加了一個 `update()` 方法來更新現有的發文。

在使用 Livewire 時，並不需要表單物件，但它們確實提供了一個很好的抽象，以避免組件中重複的樣板代碼。

### 重置表單字段

如果您使用表單物件，可能希望在提交後重置表單。這可以通過調用 `reset()` 方法來完成：

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    // ...

    public function store()
    {
        $this->validate();

        Post::create($this->only(['title', 'content']));

        $this->reset(); // [tl! highlight]
    }
}
```

您也可以通過將屬性名稱傳遞給 `reset()` 方法來重置特定屬性：

```php
$this->reset('title');

// Or multiple at once...

$this->reset(['title', 'content']);
```

### 提取表單字段

或者，您可以使用 `pull()` 方法來一次性檢索表單的屬性並重置它們。

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    // ...

    public function store()
    {
        $this->validate();

        Post::create(
            $this->pull() // [tl! highlight]
        );
    }
}
```

您還可以通過將屬性名稱傳遞給 `pull()` 方法來提取特定屬性：

```php
// Return a value before resetting...
$this->pull('title');

 // Return a key-value array of properties before resetting...
$this->pull(['title', 'content']);
```

### 使用規則對象

如果您有更複雜的驗證場景，需要 Laravel 的 `Rule` 對象，您可以替代地定義一個 `rules()` 方法來聲明您的驗證規則，如下所示：

```php
<?php

namespace App\Livewire\Forms;

use Illuminate\Validation\Rule;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    public ?Post $post;

    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => [
                'required',
                Rule::unique('posts')->ignore($this->post), // [tl! highlight]
            ],
            'content' => 'required|min:5',
        ];
    }

    // ...

    public function update()
    {
        $this->validate();

        $this->post->update($this->only(['title', 'content']));

        $this->reset();
    }
}
```

當使用 `rules()` 方法而不是 `#[Validate]` 時，Livewire 只會在您調用 `$this->validate()` 時運行驗證規則，而不是每次更新屬性時都運行。

如果您正在使用實時驗證或任何其他場景，希望 Livewire 在每次請求後驗證特定字段，您可以像這樣使用 `#[Validate]`，而不提供任何規則：

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Illuminate\Validation\Rule;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    public ?Post $post;

    #[Validate] // [tl! highlight]
    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => [
                'required',
                Rule::unique('posts')->ignore($this->post),
            ],
            'content' => 'required|min:5',
        ];
    }

    // ...

    public function update()
    {
        $this->validate();

        $this->post->update($this->only(['title', 'content']));

        $this->reset();
    }
}
```

現在，如果在提交表單之前更新了 `$title` 屬性—例如在使用 [`wire:model.blur`](/docs/wire-model#updating-on-blur-event) 時—將運行對 `$title` 的驗證。

### 顯示加載指示器

默認情況下，Livewire 將在表單提交時自動禁用提交按鈕並將輸入標記為 `readonly`，防止用戶在處理第一次提交時再次提交表單。

但是，對於用戶來說，要在應用程序的 UI 中提供額外的提示來檢測這種“加載”狀態可能有些困難。

以下是通過 `wire:loading` 將一個小的加載旋轉器添加到“保存”按鈕的示例，以便用戶了解表單正在提交：

```blade
<button type="submit">
    Save

    <div wire:loading>
        <svg>...</svg> <!-- SVG loading spinner -->
    </div>
</button>
```

現在，當用戶按下“保存”時，將顯示一個小的內嵌旋轉器。

Livewire 的 `wire:loading` 功能還有更多功能。訪問[加載文檔以了解更多。](/docs/wire-loading)

## 即時更新欄位

預設情況下，Livewire 只會在表單提交時（或調用任何其他[操作](/docs/actions)時）發送網絡請求，而不是在填寫表單時。

以 `CreatePost` 元件為例。如果您希望確保「標題」輸入欄位與後端的 `$title` 屬性同步，當用戶輸入時，您可以像這樣將 `.live` 修飾符添加到 `wire:model`：

```blade
<input type="text" wire:model.live="title">
```

現在，當用戶在此欄位中輸入時，將向服務器發送網絡請求以更新 `$title`。這對於實時搜索等情況非常有用，當用戶在搜索框中輸入時，數據集將被過濾。

## 只在 _失焦_ 時更新欄位

對於大多數情況，`wire:model.live` 對於實時更新表單字段是可以接受的；但是，對於文本輸入框，它可能會過度消耗網絡資源。

如果您希望在用戶輸入時不發送網絡請求，而是在用戶「切換」出文本輸入框時才發送請求（也稱為文本框「失焦」），則可以改用 `.blur` 修飾符：

```blade
<input type="text" wire:model.blur="title" >
```

現在，直到用戶按下 Tab 鍵或從文本輸入框點擊到其他位置，元件類別才會在服務器上更新。

## 實時驗證

有時，您可能希望在用戶填寫表單時顯示驗證錯誤。這樣，他們可以及早得知有問題，而不必等到整個表單填寫完畢。

Livewire 會自動處理這類情況。通過在 `wire:model` 上使用 `.live` 或 `.blur`，Livewire 將在用戶填寫表單時發送網絡請求。每個網絡請求都會在更新每個屬性之前運行適當的驗證規則。如果驗證失敗，則不會在服務器上更新屬性，並向用戶顯示驗證消息：

```blade
<input type="text" wire:model.blur="title">

<div>
    @error('title') <span class="error">{{ $message }}</span> @enderror
</div>
```

```php
#[Validate('required|min:5')]
public $title = '';
```

現在，如果用戶只在「標題」輸入欄位中輸入三個字符，然後點擊表單中的下一個輸入欄位，將向他們顯示驗證消息，指示該字段需要至少五個字符。

## 即時表單保存

如果您希望在用戶填寫表單時自動保存表單，而不是等到用戶點擊“提交”，您可以使用 Livewire 的 `updated()` 鉤子：

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class UpdatePost extends Component
{
    public Post $post;

    #[Validate('required')]
    public $title = '';

    #[Validate('required')]
    public $content = '';

    public function mount(Post $post)
    {
        $this->post = $post;
        $this->title = $post->title;
        $this->content = $post->content;
    }

    public function updated($name, $value) // [tl! highlight:5]
    {
        $this->post->update([
            $name => $value,
        ]);
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```blade
<form wire:submit>
    <input type="text" wire:model.blur="title">
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror
    </div>

    <input type="text" wire:model.blur="content">
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror
    </div>
</form>
```

在上面的示例中，當用戶完成一個字段（通過點擊或切換到下一個字段）時，將發送網絡請求以更新組件上的該屬性。在類中更新屬性後，將為該特定屬性名稱調用 `updated()` 鉤子及其新值。

我們可以使用此鉤子僅更新數據庫中的該特定字段。

此外，由於我們將 `#[Validate]` 屬性附加到這些屬性，因此在更新屬性並調用 `updated()` 鉤子之前將運行驗證規則。

要了解有關“updated”生命週期鉤子和其他鉤子的更多信息，[請查看生命週期鉤子文件](/docs/lifecycle-hooks)。

## 顯示未保存指示器

在上面討論的即時保存方案中，當一個字段尚未持久化到數據庫時，向用戶指示可能是有幫助的。

例如，如果用戶訪問一個 `UpdatePost` 頁面並開始修改帖子標題的文本輸入，對於他們來說，當標題實際上在數據庫中更新時可能不太清楚，特別是如果表單底部沒有“保存”按鈕。

Livewire 提供了 `wire:dirty` 指示詞，允許您在輸入值與服務端組件不一致時切換元素或修改類：

```blade
<input type="text" wire:model.blur="title" wire:dirty.class="border-yellow">
```

在上面的示例中，當用戶在輸入字段中輸入時，該字段周圍將出現黃色邊框。當用戶切換時，將發送網絡請求，邊框將消失；向他們表明輸入已被持久化，不再是“未保存”的。

如果要切換整個元素的可見性，您可以通過將 `wire:dirty` 與 `wire:target` 結合使用來實現。`wire:target` 用於指定要監視“未保存”狀態的數據。在這種情況下，是“title”字段：

```blade
<input type="text" wire:model="title">

<div wire:dirty wire:target="title">未儲存...</div>
```

## 輸入防彈跳

當在文字輸入框上使用 `.live` 時，您可能希望對發送網絡請求的頻率進行更細緻的控制。默認情況下，輸入框應用了 "250ms" 的防彈跳；但是，您可以使用 `.debounce` 修飾符來自定義此行為：

```blade
<input type="text" wire:model.live.debounce.150ms="title" >
```

現在在該字段中添加了 `.debounce.150ms`，在處理此字段的輸入更新時將使用較短的 "150ms" 防彈跳。換句話說，當用戶輸入時，只有在用戶停止輸入至少 150 毫秒時才會發送網絡請求。

## 輸入節流

如前所述，當對字段應用輸入防彈跳時，直到用戶停止輸入一段時間後才會發送網絡請求。這意味著如果用戶繼續輸入長消息，則直到用戶完成輸入為止才會發送網絡請求。

有時這不是所需的行為，您可能希望在用戶輸入時發送請求，而不是等到他們完成或休息。

在這些情況下，您可以改用 `.throttle` 來表示發送網絡請求的時間間隔：

```blade
<input type="text" wire:model.live.throttle.150ms="title" >
```

在上面的示例中，當用戶在 "title" 字段中持續輸入時，每 150 毫秒將發送一次網絡請求，直到用戶完成為止。

## 將輸入字段提取到 Blade 元件中

即使在像我們討論的 `CreatePost` 範例這樣的小組件中，我們最終會重複很多表單字段的樣板，如驗證消息和標籤。

將這些重複的 UI 元素提取到專用的 [Blade 元件](https://laravel.com/docs/blade#components) 中以在應用程序中共享可能會很有幫助。

例如，以下是來自 `CreatePost` 組件的原始 Blade 模板。我們將把以下兩個文字輸入框提取到專用的 Blade 元件中：

```blade
<form wire:submit="save">
    <input type="text" wire:model="title"> <!-- [tl! highlight:3] -->
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror
    </div>

    <input type="text" wire:model="content"> <!-- [tl! highlight:3] -->
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror
    </div>

    <button type="submit">Save</button>
</form>
```

在提取一個可重複使用的 Blade 元件 `<x-input-text>` 後，模板將如下所示：

```blade
<form wire:submit="save">
    <x-input-text name="title" wire:model="title" /> <!-- [tl! highlight] -->

    <x-input-text name="content" wire:model="content" /> <!-- [tl! highlight] -->

    <button type="submit">Save</button>
</form>
```

接下來，這是 `x-input-text` 元件的原始碼：

```blade
<!-- resources/views/components/input-text.blade.php -->

@props(['name'])

<input type="text" name="{{ $name }}" {{ $attributes }}>

<div>
    @error($name) <span class="error">{{ $message }}</span> @enderror
</div>
```

正如您所看到的，我們將重複的 HTML 放入了一個專用的 Blade 元件中。

在大多數情況下，Blade 元件僅包含從原始元件中提取的 HTML。但是，我們添加了兩個內容：

* `@props` 指示詞
* 在輸入框上的 `{{ $attributes }}` 敘述

讓我們討論這兩個新增內容：

通過使用 `@props(['name'])` 將 `name` 指定為一個 "prop"，我們告訴 Blade：如果在此元件上設置了名為 "name" 的屬性，則將其值提取並在此元件內部作為 `$name` 可用。

對於沒有明確目的的其他屬性，我們使用 `{{ $attributes }}` 敘述。這用於 "屬性轉發"，或者換句話說，將寫在 Blade 元件上的任何 HTML 屬性轉發到元件內的元素。

這確保 `wire:model="title"` 和任何其他額外屬性，如 `disabled`、`class="..."` 或 `required` 仍然轉發到實際的 `<input>` 元素。

### 自訂表單控制項

在上面的示例中，我們將一個輸入元素包裹到一個可重複使用的 Blade 元件中，我們可以像使用本機 HTML 輸入元素一樣使用它。

這種模式非常有用；但是，可能會有一些情況，您希望從頭開始創建整個輸入元件（而不是基於本機輸入元素），但仍然能夠使用 `wire:model` 將其值綁定到 Livewire 屬性。

例如，假設您想要創建一個 `<x-input-counter />` 元件，這是一個簡單的在 Alpine 中編寫的 "計數器" 輸入。

在創建 Blade 元件之前，讓我們首先查看一個簡單的、純 Alpine 的 "計數器" 元件作為參考：

```blade
<div x-data="{ count: 0 }">
    <button x-on:click="count--">-</button>

    <span x-text="count"></span>

    <button x-on:click="count++">+</button>
</div>
```

現在，讓我們想像我們想要將這個元件提取為一個名為 `<x-input-counter />` 的 Blade 元件，我們可以在元件中這樣使用：

```blade
<x-input-counter wire:model="quantity" />
```

創建這個元件通常很簡單。我們將計數器的 HTML 放入一個 Blade 元件模板中，例如 `resources/views/components/input-counter.blade.php`。

然而，要使其與 `wire:model="quantity"` 一起運作，以便您可以輕鬆地將數據從 Livewire 元件綁定到此 Alpine 元件內的 "count"，需要額外的一個步驟。

這是該元件的原始碼：

```blade
<!-- resources/view/components/input-counter.blade.php -->

<div x-data="{ count: 0 }" x-modelable="count" {{ $attributes}}>
    <button x-on:click="count--">-</button>

    <span x-text="count"></span>

    <button x-on:click="count++">+</button>
</div>
```

正如您所看到的，這個 HTML 的唯一不同之處是 `x-modelable="count"` 和 `{{ $attributes }}`。

`x-modelable` 是 Alpine 中的一個實用工具，告訴 Alpine 使某個數據片段可以從外部綁定。[Alpine 文件中有關於這個指示詞的更多信息。](https://alpinejs.dev/directives/modelable)

`{{ $attributes }}`，正如我們之前探討的，將從外部傳入的任何屬性傳遞到 Blade 元件中。在這種情況下，`wire:model` 指令將被傳遞。

由於 `{{ $attributes }}`，當 HTML 在瀏覽器中呈現時，`wire:model="quantity"` 將與 `x-modelable="count"` 一起呈現在 Alpine 元件的根 `<div>` 上，如下所示：

```blade
<div x-data="{ count: 0 }" x-modelable="count" wire:model="quantity">
```

`x-modelable="count"` 告訴 Alpine 尋找任何 `x-model` 或 `wire:model` 陳述並使用 "count" 作為要將其綁定到的數據。

由於 `x-modelable` 同時適用於 `wire:model` 和 `x-model`，您也可以在 Livewire 和 Alpine 之間互換地使用這個 Blade 元件。以下是在純粹的 Alpine 上下文中使用這個 Blade 元件的示例：

```blade
<x-input-counter x-model="quantity" />
```

在應用程式中創建自定義輸入元素非常強大，但需要更深入地了解 Livewire 和 Alpine 提供的實用工具以及它們如何相互作用。

Please paste the Markdown content you need to be translated into traditional Chinese.
