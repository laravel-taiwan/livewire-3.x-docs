Livewire 旨在使用戶輸入的驗證和提供反饋盡可能愉快。通過在 Laravel 的驗證功能基礎上構建，Livewire 利用您現有的知識，同時為您提供了強大的額外功能，如實時驗證。

這裡是一個示例 `CreatePost` 元件，演示了 Livewire 中最基本的驗證工作流程：

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
        $validated = $this->validate([ // [tl! highlight:3]
			'title' => 'required|min:3',
			'content' => 'required|min:3',
        ]);

		Post::create($validated);

		return redirect()->to('/posts');
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
    <div>@error('title') {{ $message }} @enderror</div>

	<textarea wire:model="content"></textarea>
    <div>@error('content') {{ $message }} @enderror</div>

	<button type="submit">Save</button>
</form>
```

如您所見，Livewire 提供了一個 `validate()` 方法，您可以調用該方法來驗證元件的屬性。它返回經過驗證的數據集，然後您可以安全地將其插入到數據庫中。

在前端，您可以使用 Laravel 現有的 Blade 指令來向用戶顯示驗證消息。

有關更多信息，請參閱[Laravel 在 Blade 中呈現驗證錯誤的文檔](https://laravel.com/docs/blade#validation-errors)。

## 驗證屬性

如果您更喜歡將元件的驗證規則與屬性直接放在一起，您可以使用 Livewire 的 `#[Validate]` 屬性。

通過使用 `#[Validate]` 將驗證規則與屬性關聯起來，Livewire 將在每次更新之前自動運行屬性的驗證規則。但是，在將數據持久化到數據庫之前，您仍應運行 `$this->validate()`，以便對未更新的屬性進行驗證。

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required|min:3')] // [tl! highlight]
	public $title = '';

    #[Validate('required|min:3')] // [tl! highlight]
    public $content = '';

    public function save()
    {
        $this->validate();

		Post::create([
            'title' => $this->title,
            'content' => $this->content,
		]);

		return redirect()->to('/posts');
    }

    // ...
}
```

> [!info] 驗證屬性不支持 Rule 對象
> PHP 屬性僅限於某些語法，如純字符串和數組。如果您希望使用類似 Laravel 的 Rule 對象（`Rule::exists(...)`）的運行時語法，您應該在您的元件中[定義一個 `rules()` 方法](#defining-a-rules-method)。
>
> 在[使用 Laravel Rule 對象與 Livewire](#using-laravel-rule-objects)的文檔中了解更多。

如果您更喜歡在何時驗證屬性方面擁有更多控制權，您可以將 `onUpdate: false` 參數傳遞給 `#[Validate]` 屬性。這將禁用任何自動驗證，而是假設您希望使用 `$this->validate()` 方法手動驗證屬性：

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required|min:3', onUpdate: false)]
	public $title = '';

    #[Validate('required|min:3', onUpdate: false)]
    public $content = '';

    public function save()
    {
        $validated = $this->validate();

		Post::create($validated);

		return redirect()->to('/posts');
    }

    // ...
}
```

### 自訂屬性名稱

如果您希望自訂注入驗證訊息中的屬性名稱，您可以使用 `as: ` 參數來進行設定：

```php
use Livewire\Attributes\Validate;

#[Validate('required', as: 'date of birth')]
public $dob;
```

在上述程式碼片段中，當驗證失敗時，Laravel 將使用 "date of birth" 代替 "dob" 作為驗證訊息中欄位的名稱。生成的訊息將是 "The date of birth field is required" 而不是 "The dob field is required"。

### 自訂驗證訊息

若要繞過 Laravel 的驗證訊息並替換為自訂訊息，您可以在 `#[Validate]` 屬性中使用 `message: ` 參數：

```php
use Livewire\Attributes\Validate;

#[Validate('required', message: 'Please provide a post title')]
public $title;
```

現在，當此屬性的驗證失敗時，訊息將是 "Please provide a post title" 而不是 "The title field is required"。

如果您希望為不同的規則添加不同的訊息，您可以簡單地提供多個 `#[Validate]` 屬性：

```php
#[Validate('required', message: 'Please provide a post title')]
#[Validate('min:3', message: 'This title is too short')]
public $title;
```

### 不使用本地化

預設情況下，Livewire 的規則訊息和屬性是使用 Laravel 的翻譯輔助程式 `trans()` 進行本地化的。

您可以通過將 `translate: false` 參數傳遞給 `#[Validate]` 屬性來選擇不使用本地化：

```php
#[Validate('required', message: 'Please provide a post title', translate: false)]
public $title;
```

### 自訂鍵

當直接將驗證規則應用於屬性時，Livewire 假設驗證鍵應該是屬性本身的名稱。但是，有時您可能希望自訂驗證鍵。

例如，您可能希望為陣列屬性及其子屬性提供獨立的驗證規則。在這種情況下，您可以將一組鍵值對的陣列而不是驗證規則作為 `#[Validate]` 屬性的第一個參數：

```php
#[Validate([
    'todos' => 'required',
    'todos.*' => [
        'required',
        'min:3',
        new Uppercase,
    ],
])]
public $todos = [];
```

現在，當使用者更新 `$todos` 或呼叫 `validate()` 方法時，這兩個驗證規則將被應用。

## 表單物件

隨著 Livewire 元件添加更多屬性和驗證規則，可能會開始感覺過於擁擠。為了減輕這種痛苦並為代碼重用提供有用的抽象，您可以使用 Livewire 的 *表單物件* 來存儲您的屬性和驗證規則。

以下是相同的 `CreatePost` 範例，但現在屬性和規則已經提取到一個專用的表單物件中，名為 `PostForm`：

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:3')]
	public $title = '';

    #[Validate('required|min:3')]
    public $content = '';
}
```

上面的 `PostForm` 現在可以被定義為 `CreatePost` 元件的一個屬性：

```php
<?php

namespace App\Livewire;

use App\Livewire\Forms\PostForm;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public PostForm $form;

    public function save()
    {
		Post::create(
    		$this->form->all()
    	);

		return redirect()->to('/posts');
    }

    // ...
}
```

如您所見，我們可以使用表單物件上的 `->all()` 方法檢索所有屬性值，而不是逐個列出每個屬性。

此外，在模板中引用屬性名稱時，您必須在每個實例前加上 `form.`：

```blade
<form wire:submit="save">
	<input type="text" wire:model="form.title">
    <div>@error('form.title') {{ $message }} @enderror</div>

	<textarea wire:model="form.content"></textarea>
    <div>@error('form.content') {{ $message }} @enderror</div>

	<button type="submit">Save</button>
</form>
```

使用表單物件時，當更新屬性時，`#[Validate]` 屬性驗證將運行。但是，如果您通過在屬性上指定 `onUpdate: false` 來禁用此行為，則可以使用 `$this->form->validate()` 手動運行表單物件的驗證：

```php
public function save()
{
    Post::create(
        $this->form->validate()
    );

    return redirect()->to('/posts');
}
```

表單物件是一個對於大多數較大數據集和各種其他功能都很有用的抽象。欲了解更多信息，請查看全面的 [表單物件文檔](/docs/forms#extracting-a-form-object)。

## 實時驗證

實時驗證是指在用戶填寫表單時對其輸入進行驗證，而不是等待表單提交時進行驗證。

通過直接在 Livewire 屬性上使用 `#[Validate]` 屬性，每當發送網絡請求以更新屬性值時，提供的驗證規則將被應用。

這意味著為了為用戶提供在特定輸入上的實時驗證體驗，不需要額外的後端工作。唯一需要的是使用 `wire:model.live` 或 `wire:model.blur` 指示 Livewire 在填寫字段時觸發網絡請求。

在下面的示例中，已將 `wire:model.blur` 添加到文本輸入中。現在，當用戶在字段中輸入文本，然後切換標籤或點擊離開字段時，將觸發一個網絡請求，其中包含更新後的值，並運行驗證規則：

```blade
<form wire:submit="save">
    <input type="text" wire:model.blur="title">

    <!-- -->
</form>
```

如果您使用 `rules()` 方法來為屬性聲明驗證規則，而不是使用 `#[Validate]` 屬性，您仍然可以包含一個不帶參數的 `#[Validate]` 屬性，以保留實時驗證行為：

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate] // [tl! highlight]
	public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => 'required|min:5',
            'content' => 'required|min:5',
        ];
    }

    public function save()
    {
        $validated = $this->validate();

		Post::create($validated);

		return redirect()->to('/posts');
    }
```

現在，在上面的示例中，即使 `#[Validate]` 是空的，它也會告訴 Livewire 在每次更新屬性時運行由 `rules()` 提供的字段驗證。

## 自定義錯誤消息

在 Laravel 中，提供了合理的驗證消息，例如如果 `$title` 屬性附加了 `required` 規則，則會顯示類似 "The title field is required." 的消息。

但是，您可能需要自定義這些錯誤消息的語言，以更好地適應您的應用程序和用戶。

### 自定義屬性名稱

有時，您正在驗證的屬性具有不適合顯示給用戶的名稱。例如，如果您的應用程序中有一個名為 `dob`（代表 "Date of birth"）的數據庫字段，您希望向用戶顯示 "The date of birth field is required"，而不是 "The dob field is required"。

Livewire 允許您使用 `as: ` 參數為屬性指定替代名稱：

```php
use Livewire\Attributes\Validate;

#[Validate('required', as: 'date of birth')]
public $dob = '';
```

現在，如果 `required` 驗證規則失敗，錯誤消息將顯示為 "The date of birth field is required."，而不是 "The dob field is required."。

### 自定義消息

如果自定義屬性名稱不夠，您可以使用 `message: ` 參數來自定義整個驗證消息：

```php
use Livewire\Attributes\Validate;

#[Validate('required', message: 'Please fill out your date of birth.')]
public $dob = '';
```

如果有多個規則需要自定義消息，建議為每個規則使用完全獨立的 `#[Validate]` 屬性，如下所示：

```php
use Livewire\Attributes\Validate;

#[Validate('required', message: 'Please enter a title.')]
#[Validate('min:5', message: 'Your title is too short.')]
public $title = '';
```

如果您想使用 `#[Validate]` 屬性的陣列語法，您可以像這樣指定自訂屬性和訊息：

```php
use Livewire\Attributes\Validate;

#[Validate([
    'titles' => 'required',
    'titles.*' => 'required|min:5',
], message: [
    'required' => 'The :attribute is missing.',
    'titles.required' => 'The :attribute are missing.',
    'min' => 'The :attribute is too short.',
], attribute: [
    'titles.*' => 'title',
])]
public $titles = [];
```

## 定義 `rules()` 方法

作為 Livewire 的 `#[Validate]` 屬性的替代方案，您可以在組件中定義一個名為 `rules()` 的方法，並返回一個字段列表和相應的驗證規則。如果您嘗試使用 PHP 屬性不支持的運行時語法，例如 Laravel 規則對象如 `Rule::password()`，這將非常有幫助。

當您在組件內運行 `$this->validate()` 時，這些規則將被應用。您還可以定義 `messages()` 和 `validationAttributes()` 函數。

這是一個示例：

```php
use Livewire\Component;
use App\Models\Post;
use Illuminate\Validation\Rule;

class CreatePost extends Component
{
	public $title = '';

    public $content = '';

    protected function rules() // [tl! highlight:6]
    {
        return [
            'title' => Rule::exists('posts', 'title'),
            'content' => 'required|min:3',
        ];
    }

    protected function messages() // [tl! highlight:6]
    {
        return [
            'content.required' => 'The :attribute are missing.',
            'content.min' => 'The :attribute is too short.',
        ];
    }

    protected function validationAttributes() // [tl! highlight:6]
    {
        return [
            'content' => 'description',
        ];
    }

    public function save()
    {
        $this->validate();

		Post::create([
            'title' => $this->title,
            'content' => $this->content,
		]);

		return redirect()->to('/posts');
    }

    // ...
}
```

> [!warning] `rules()` 方法不會在數據更新時進行驗證
> 當通過 `rules()` 方法定義規則時，Livewire 僅在運行 `$this->validate()` 時使用這些驗證規則來驗證屬性。這與標準的 `#[Validate]` 屬性不同，後者在每次通過 `wire:model` 等方式更新字段時應用。要將這些驗證規則應用到每次更新屬性時，仍然可以使用沒有額外參數的 `#[Validate]`。

> [!warning] 不要與 Livewire 的機制衝突
> 在使用 Livewire 的驗證工具時，您的組件 **不應該** 有名為 `rules`、`messages`、`validationAttributes` 或 `validationCustomValues` 的屬性或方法，除非您正在自定義驗證過程。否則，這些將與 Livewire 的機制衝突。

## 使用 Laravel 規則對象

Laravel 的 `Rule` 對象是向表單添加高級驗證行為的一種非常強大的方式。

以下是在 Livewire 的 `rules()` 方法中使用規則對象來實現更複雜驗證的示例：

```php
<?php

namespace App\Livewire;

use Illuminate\Validation\Rule;
use App\Models\Post;
use Livewire\Form;

class UpdatePost extends Form
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

    public function mount()
    {
        $this->title = $this->post->title;
        $this->content = $this->post->content;
    }

    public function update()
    {
        $this->validate(); // [tl! highlight]

        $this->post->update($this->all());

        $this->reset();
    }

    // ...
}
```

## 手動控制驗證錯誤

Livewire 的驗證工具應該處理最常見的驗證場景；但是，有時您可能希望在組件中對驗證訊息有完全控制。

以下是在 Livewire 元件中操作驗證錯誤的所有可用方法：

方法 | 說明
--- | ---
`$this->addError([key], [message])` | 手動將驗證訊息添加到錯誤包中
`$this->resetValidation([?key])` | 重置提供的鍵的驗證錯誤，或者如果未提供鍵，則重置所有錯誤
`$this->getErrorBag()` | 檢索 Livewire 元件中使用的基礂 Laravel 錯誤包

> [!info] 使用 `$this->addError()` 與表單物件
> 當在表單物件內部使用 `$this->addError` 手動添加錯誤時，鍵將自動以父元件中分配給表單的屬性名稱作為前綴。例如，如果在您的元件中將表單分配給名為 `$data` 的屬性，鍵將變為 `data.key`。

## 存取驗證器實例

有時您可能希望存取 Livewire 在 `validate()` 方法中內部使用的驗證器實例。這可以使用 `withValidator` 方法實現。您提供的閉包將接收完全構建的驗證器作為參數，使您能夠在實際評估驗證規則之前調用其任何方法。

以下是拦截 Livewire 內部驗證器以手動檢查條件並添加額外驗證訊息的示例：

```php
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required|min:3')]
	public $title = '';

    #[Validate('required|min:3')]
    public $content = '';

    public function boot()
    {
        $this->withValidator(function ($validator) {
            $validator->after(function ($validator) {
                if (str($this->title)->startsWith('"')) {
                    $validator->errors()->add('title', 'Titles cannot start with quotations');
                }
            });
        });
    }

    public function save()
    {
		Post::create($this->all());

		return redirect()->to('/posts');
    }

    // ...
}
```

## 使用自訂驗證器

如果您希望在 Livewire 中使用自己的驗證系統，這並不是問題。Livewire 將捕獲元件內部拋出的任何 `ValidationException` 異常，並將錯誤提供給視圖，就像您使用 Livewire 自己的 `validate()` 方法一樣。

以下是 `CreatePost` 元件的示例，但是不使用 Livewire 的驗證功能，而是創建並應用完全自定義的驗證器到元件屬性：

```php
use Illuminate\Support\Facades\Validator;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
	public $title = '';

    public $content = '';

    public function save()
    {
        $validated = Validator::make(
            // Data to validate...
            ['title' => $this->title, 'content' => $this->content],

            // Validation rules to apply...
            ['title' => 'required|min:3', 'content' => 'required|min:3'],

            // Custom validation messages...
            ['required' => 'The :attribute field is required'],
         )->validate();

		Post::create($validated);

		return redirect()->to('/posts');
    }

    // ...
}
```

## 驗證測試

Livewire 提供了有用的驗證場景測試工具，例如 `assertHasErrors()` 方法。

以下是一個基本的測試案例，確保如果未為 `title` 屬性設置任何輸入，則會拋出驗證錯誤：

除了測試錯誤的存在之外，`assertHasErrors` 還允許您通過將要對抗的規則作為方法的第二個參數傳遞來將斷言範圍縮小到特定規則：

您還可以同時斷言多個屬性的驗證錯誤的存在：

有關 Livewire 提供的其他測試工具的更多信息，請查看[測試文件](/docs/testing)。

## 已棄用的 `[#Rule]` 屬性

當 Livewire v3 首次推出時，它使用 "Rule" 一詞而不是 "Validate" 來表示其驗證屬性 (`#[Rule]`)。

由於與 Laravel 規則對象的命名衝突，這已經更改為 `#[Validate]`。在 Livewire v3 中支持這兩種，但建議您將所有 `#[Rule]` 的出現更改為 `#[Validate]` 以保持最新。
