Livewire 提供了各種生命週期鉤子，允許您在元件生命週期的特定時刻執行程式碼。這些鉤子使您能夠在特定事件之前或之後執行操作，例如初始化元件、更新屬性或渲染模板。

以下是所有可用的元件生命週期鉤子列表：

| 鉤子方法         | 說明                                                                           |
|------------------|---------------------------------------------------------------------------------|
| `mount()`        | 當元件被建立時調用                                                              |
| `hydrate()`      | 在後續請求開始時重新將元件重新填充時調用                                       |
| `boot()`         | 在每個請求的開頭調用。包括初始和後續請求                                      |
| `updating()`     | 在更新元件屬性之前調用                                                         |
| `updated()`      | 在更新屬性後調用                                                               |
| `rendering()`    | 在調用 `render()` 之前調用                                                     |
| `rendered()`     | 在調用 `render()` 後調用                                                       |
| `dehydrate()`    | 在每個元件請求結束時調用                                                       |
| `exception($e, $stopPropagation)` | 當拋出異常時調用                      |

## Mount

在標準的 PHP 類中，建構子 (`__construct()`) 接受外部參數並初始化物件的狀態。然而，在 Livewire 中，您使用 `mount()` 方法來接受參數並初始化元件的狀態。

Livewire 元件不使用 `__construct()`，因為 Livewire 元件在後續網路請求中會被 _重新構建_，我們只希望在首次建立元件時初始化元件一次。

以下是使用 `mount()` 方法來初始化 `UpdateProfile` 元件的 `name` 和 `email` 屬性的範例：

```php
use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class UpdateProfile extends Component
{
    public $name;

    public $email;

    public function mount()
    {
        $this->name = Auth::user()->name;

        $this->email = Auth::user()->email;
    }

    // ...
}
```

如前所述，`mount()` 方法接收作為方法參數傳遞到元件的數據：

```php
use Livewire\Component;
use App\Models\Post;

class UpdatePost extends Component
{
    public $title;

    public $content;

    public function mount(Post $post)
    {
        $this->title = $post->title;

        $this->content = $post->content;
    }

    // ...
}
```

> [!tip] 您可以在所有掛勾方法中使用依賴注入
> Livewire 允許您通過在生命週期掛勾的方法參數上進行型別提示，從 [Laravel 的服務容器](https://laravel.com/docs/container#automatic-injection) 解析依賴。

`mount()` 方法是使用 Livewire 的重要部分。以下文檔提供了使用 `mount()` 方法完成常見任務的進一步示例：

* [初始化屬性](/docs/properties#initializing-properties)
* [從父元件接收數據](/docs/nesting#passing-props-to-children)
* [訪問路由參數](/docs/components#accessing-route-parameters)

## 啟動

儘管 `mount()` 非常有用，但它僅在每個元件生命週期運行一次，您可能希望在每次為給定元件向服務器發送請求時運行邏輯。

對於這些情況，Livewire 提供了一個 `boot()` 方法，您可以在其中編寫您打算在每次組件類別啟動時運行的組件設置代碼：無論是在初始化時還是在後續請求時。

`boot()` 方法可用於初始化受保護的屬性等任務，這些任務在請求之間不會保留。以下是初始化受保護屬性作為 Eloquent 模型的示例：

```php
use Livewire\Attributes\Locked;
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    #[Locked]
    public $postId = 1;

    protected Post $post;

    public function boot() // [tl! highlight:3]
    {
        $this->post = Post::find($this->postId);
    }

    // ...
}
```

您可以使用此技術完全控制 Livewire 元件中屬性的初始化。

> [!tip] 大多數情況下，您可以改用計算屬性
> 上述使用的技術很強大；但通常更好的做法是使用 [Livewire 的計算屬性](/docs/computed-properties) 來解決這個用例。

> [!warning] 始終鎖定敏感的公共屬性
> 如上所示，我們在 `$postId` 屬性上使用了 `#[Locked]` 屬性。在像上面這樣的情況下，您希望確保 `$postId` 屬性不會被用戶在客戶端篡改，重要的是在使用之前授權屬性的值或在屬性上添加 `#[Locked]` 以確保它永遠不會更改。
>
> 欲瞭解更多信息，請查看有關 [鎖定屬性](/docs/locked) 的文檔。

## 更新

客戶端用戶可以通過多種方式更新公共屬性，最常見的是通過在其上使用 `wire:model` 來修改輸入。

Livewire 提供了方便的鉤子來攔截公共屬性的更新，這樣您就可以在設置之前驗證或授權值，或者確保屬性以特定格式設置。

以下是使用 `updating` 來防止修改 `$postId` 屬性的示例。

值得注意的是，對於這個特定示例，在實際應用中，您應該使用 [`#[Locked]` 屬性](/docs/locked) 來代替，就像上面的示例一樣。

```php
use Exception;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId = 1;

    public function updating($property, $value)
    {
        // $property: The name of the current property being updated
        // $value: The value about to be set to the property

        if ($property === 'postId') {
            throw new Exception;
        }
    }

    // ...
}
```

上面的 `updating()` 方法在屬性更新之前運行，允許您捕獲無效輸入並防止屬性更新。以下是使用 `updated()` 來確保屬性值保持一致的示例：

```php
use Livewire\Component;

class CreateUser extends Component
{
    public $username = '';

    public $email = '';

    public function updated($property)
    {
        // $property: The name of the current property that was updated

        if ($property === 'username') {
            $this->username = strtolower($this->username);
        }
    }

    // ...
}
```

現在，每當客戶端更新 `$username` 屬性時，我們將確保該值始終為小寫。

由於在使用更新鉤子時通常會針對特定屬性，Livewire 允許您直接將屬性名稱指定為方法名的一部分。以下是從上面相同示例重新編寫並利用此技術：

```php
use Livewire\Component;

class CreateUser extends Component
{
    public $username = '';

    public $email = '';

    public function updatedUsername()
    {
        $this->username = strtolower($this->username);
    }

    // ...
}
```

當然，您也可以將此技術應用於 `updating` 鉤子。

### 陣列

陣列屬性有一個額外的 `$key` 參數傳遞給這些函數，以指定正在更改的元素。

請注意，當更新陣列本身而不是特定鍵時，`$key` 參數為空。

```php
use Livewire\Component;

class UpdatePreferences extends Component
{
    public $preferences = [];

    public function updatedPreferences($value, $key)
    {
        // $value = 'dark'
        // $key   = 'theme'
    }

    // ...
}
```

## 賦值 & 反賦值

賦值和反賦值是較少知名且較少使用的鉤子。但是，在特定情況下，它們可以非常有用。

“反賦值”和“賦值”這兩個術語指的是 Livewire 組件被序列化為 JSON 供客戶端使用，然後在後續請求中反序列化回 PHP 物件。

我們通常在 Livewire 的代碼庫和文檔中使用“賦值”和“反賦值”這兩個術語來指代這個過程。如果您想更清楚地了解這些術語，可以通過[查閱我們的賦值文檔](/docs/hydration)來獲取更多信息。

讓我們看一個示例，該示例同時使用 `mount()`、`hydrate()` 和 `dehydrate()` 來支持使用自定義 [資料傳輸物件 (DTO)](https://en.wikipedia.org/wiki/Data_transfer_object) 而不是 Eloquent 模型來存儲組件中的文章數據：

```php
use Livewire\Component;

class ShowPost extends Component
{
    public $post;

    public function mount($title, $content)
    {
        // Runs at the beginning of the first initial request...

        $this->post = new PostDto([
            'title' => $title,
            'content' => $content,
        ]);
    }

    public function hydrate()
    {
        // Runs at the beginning of every "subsequent" request...
        // This doesn't run on the initial request ("mount" does)...

        $this->post = new PostDto($this->post);
    }

    public function dehydrate()
    {
        // Runs at the end of every single request...

        $this->post = $this->post->toArray();
    }

    // ...
}
```

現在，在組件內的操作和其他地方，您可以訪問 `PostDto` 物件，而不是原始數據。

上面的示例主要展示了 `hydrate()` 和 `dehydrate()` 鉤子的功能和性質。然而，建議您改用 [Wireables 或 Synthesizers](/docs/properties#supporting-custom-types) 來完成這個任務。

## 渲染

如果您想要鉤入渲染組件 Blade 視圖的過程，您可以使用 `rendering()` 和 `rendered()` 鉤子來實現：

```php
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function render()
    {
        return view('livewire.show-posts', [
            'post' => Post::all(),
        ])
    }

    public function rendering($view, $data)
    {
        // Runs BEFORE the provided view is rendered...
        //
        // $view: The view about to be rendered
        // $data: The data provided to the view
    }

    public function rendered($view, $html)
    {
        // Runs AFTER the provided view is rendered...
        //
        // $view: The rendered view
        // $html: The final, rendered HTML
    }

    // ...
}
```

## 例外

有時截取並捕獲錯誤可能很有幫助，例如：自定義錯誤消息或忽略特定類型的異常。`exception()` 鉤子允許您做到這一點：您可以對 `$error` 執行檢查，並使用 `$stopPropagation` 參數來捕獲問題。
這也為您提供了強大的模式，當您想要停止代碼的進一步執行時（提前返回），這就是內部方法如 `validate()` 的工作方式。

```php
use Livewire\Component;

class ShowPost extends Component
{
    public function mount() // [tl! highlight:3]
    {
        $this->post = Post::find($this->postId);
    }

    public function exception($e, $stopPropagation) {
        if ($e instanceof NotFoundException) {
            $this->notify('Post is not found');
            $stopPropagation();
        }
    }

    // ...
}
```

## 在 trait 內使用鉤子

Trait 是在組件之間重複使用代碼或從單個組件中提取代碼的有用方式。

為了避免在聲明生命週期鉤子方法時多個 trait 之間發生衝突，Livewire 支持使用當前聲明它們的 trait 的 _駝峰式_ 名稱作為鉤子方法的前綴。

這樣，您可以有多個 trait 使用相同的生命週期鉤子，並避免衝突的方法定義。

以下是一個組件引用名為 `HasPostForm` 的 trait 的示例：

```php
use Livewire\Component;

class CreatePost extends Component
{
    use HasPostForm;

    // ...
}
```

現在這裡是實際的 `HasPostForm` trait，其中包含所有可用的前綴鉤子：

```php
trait HasPostForm
{
    public $title = '';

    public $content = '';

    public function mountHasPostForm()
    {
        // ...
    }

    public function hydrateHasPostForm()
    {
        // ...
    }

    public function bootHasPostForm()
    {
        // ...
    }

    public function updatingHasPostForm()
    {
        // ...
    }

    public function updatedHasPostForm()
    {
        // ...
    }

    public function renderingHasPostForm()
    {
        // ...
    }

    public function renderedHasPostForm()
    {
        // ...
    }

    public function dehydrateHasPostForm()
    {
        // ...
    }

    // ...
}
```

Please paste the Markdown content you need to be translated into traditional Chinese.
