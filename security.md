確保您的 Livewire 應用程式安全並且不會暴露任何應用程式漏洞是非常重要的。Livewire 具有內部安全功能來處理許多情況，然而，有時需要依賴您的應用程式代碼來確保您的元件安全。

## 授權動作引數

Livewire 的動作非常強大，然而，傳遞給 Livewire 動作的任何引數在客戶端上是可變的，應該被視為不受信任的使用者輸入。

在 Livewire 中最常見的安全陷阱之一是在將更改持久化到資料庫之前未驗證和授權 Livewire 動作呼叫。

以下是由於缺乏授權而導致的一個不安全示例：

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    // ...

    public function delete($id)
    {
        // INSECURE!

        $post = Post::find($id);

        $post->delete();
    }
}
```

```html
<button wire:click="delete({{ $post->id }})">刪除文章</button>
```

上面示例不安全的原因是 `wire:click="delete(...)"` 可以在瀏覽器中被修改以傳遞任何惡意使用者希望的文章 ID。

動作引數（像這個案例中的 `$id`）應該與來自瀏覽器的任何不受信任的輸入一樣處理。

因此，為了保證應用程式安全並防止使用者刪除其他使用者的文章，我們必須在 `delete()` 動作中添加授權。

首先，讓我們透過執行以下命令為 Post 模型創建一個 [Laravel 政策](https://laravel.com/docs/authorization#creating-policies)：

```bash
php artisan make:policy PostPolicy --model=Post
```

執行上述命令後，將在 `app/Policies/PostPolicy.php` 內創建一個新的政策。然後，我們可以更新其內容，添加一個像這樣的 `delete` 方法：

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if the given post can be deleted by the user.
     */
    public function delete(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

現在，我們可以從 Livewire 元件使用 `$this->authorize()` 方法來確保使用者在刪除之前擁有該文章：

```php
public function delete($id)
{
    $post = Post::find($id);

    // If the user doesn't own the post,
    // an AuthorizationException will be thrown...
    $this->authorize('delete', $post); // [tl! highlight]

    $post->delete();
}
```

進一步閱讀：
* [Laravel Gates](https://laravel.com/docs/authorization#gates)
* [Laravel Policies](https://laravel.com/docs/authorization#creating-policies)

## 授權公共屬性

與動作引數類似，Livewire 中的公共屬性應該被視為來自使用者的不受信任輸入。

這裡有一個關於以不安全方式撰寫的刪除文章範例，以不同方式呈現：

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        // INSECURE!

        $post = Post::find($this->postId);

        $post->delete();
    }
}
```

```html
<button wire:click="delete">刪除文章</button>
```

如您所見，我們將 `$postId` 存儲為 Livewire 元件的公共屬性，而不是將其作為參數傳遞給 `wire:click` 的 `delete` 方法。

這種方法的問題在於，任何惡意用戶都可以注入自定義元素到頁面上，例如：

```html
<input type="text" wire:model="postId">
```

這將允許他們在按下「刪除文章」之前自由修改 `$postId`。由於 `delete` 操作未授權 `$postId` 的值，用戶現在可以刪除數據庫中的任何文章，無論他們是否擁有它。

為了防範這種風險，有兩種可能的解決方案：

### 使用模型屬性

當設置公共屬性時，Livewire 對待模型與普通值（如字符串和整數）的方式不同。因此，如果我們將整個文章模型存儲為元件的屬性，Livewire 將確保 ID 永遠不會被篡改。

這是將 `$post` 屬性存儲為簡單的 `$postId` 屬性的示例：

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public Post $post;

    public function mount($postId)
    {
        $this->post = Post::find($postId);
    }

    public function delete()
    {
        $this->post->delete();
    }
}
```

```html
<button wire:click="delete">刪除文章</button>
```

這個元件現在是安全的，因為惡意用戶無法將 `$post` 屬性更改為不同的 Eloquent 模型。

### 鎖定屬性

防止屬性被設置為不需要的值的另一種方法是使用 [鎖定屬性](https://livewire.laravel.com/docs/locked)。通過應用 `#[Locked]` 屬性來鎖定屬性。現在，如果用戶嘗試篡改此值，將拋出錯誤。

請注意，具有 Locked 屬性的屬性仍然可以在後端更改，因此仍然需要注意，在您自己的 Livewire 函數中不要將不受信任的用戶輸入傳遞給屬性。

```php
<?php

use App\Models\Post;
use Livewire\Component;
use Livewire\Attributes\Locked;

class ShowPost extends Component
{
    #[Locked] // [tl! highlight]
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        $post = Post::find($this->postId);

        $post->delete();
    }
}
```

### 授權屬性

如果在您的情況中不希望使用模型屬性，當然可以退而手動授權在 `delete` 操作中刪除文章。 

--- 

*原始連結：* [https://laravel.com/docs/8.x](https://laravel.com/docs/8.x)

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        $post = Post::find($this->postId);

        $this->authorize('delete', $post); // [tl! highlight]

        $post->delete();
    }
}
```

```html
<button wire:click="delete">刪除文章</button>
```

現在，即使惡意使用者仍然可以自由修改 `$postId` 的值，當調用 `delete` 操作時，如果使用者不擁有該文章，`$this->authorize()` 將拋出一個 `AuthorizationException`。

進一步閱讀：
* [Laravel Gates](https://laravel.com/docs/authorization#gates)
* [Laravel Policies](https://laravel.com/docs/authorization#creating-policies)

## 中介層

當一個 Livewire 元件在包含路由級別的[授權中介層](https://laravel.com/docs/authorization#via-middleware)的頁面上加載時，如下所示：

```php
Route::get('/post/{post}', App\Livewire\UpdatePost::class)
    ->middleware('can:update,post'); // [tl! highlight]
```

Livewire 將確保這些中介層被重新應用於後續的 Livewire 網絡請求。這在 Livewire 核心中被稱為“持久中介層”。

持久中介層保護您免受授權規則或使用者權限在初始頁面加載後發生變化的情況。

以下是這種情況的更深入示例：

```php
Route::get('/post/{post}', App\Livewire\UpdatePost::class)
    ->middleware('can:update,post'); // [tl! highlight]
```

```php
<?php

use App\Models\Post;
use Livewire\Component;
use Livewire\Attributes\Validate;

class UpdatePost extends Component
{
    public Post $post;

    #[Validate('required|min:5')]
    public $title = '';

    public $content = '';

    public function mount()
    {
        $this->title = $this->post->title;
        $this->content = $this->post->content;
    }

    public function update()
    {
        $this->post->update([
            'title' => $this->title,
            'content' => $this->content,
        ]);
    }
}
```

正如您所看到的，`can:update,post` 中介層應用在路由級別。這意味著沒有權限更新文章的使用者無法查看該頁面。

然而，考慮一個情況，一個使用者：
* 加載頁面
* 在頁面加載後失去更新權限
* 嘗試在失去權限後更新文章

因為 Livewire 已經成功加載了頁面，您可能會問自己：“當 Livewire 發出後續請求來更新文章時，`can:update,post` 中介層會被重新應用嗎？還是未經授權的使用者將能夠成功更新文章？”

由於 Livewire 具有重新應用原始端點中介層的內部機制，您在這種情況下是受到保護的。

### 配置持久中介層

默認情況下，Livewire會在網絡請求之間保留以下中間件：

```php
\Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
\Laravel\Jetstream\Http\Middleware\AuthenticateSession::class,
\Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
\Illuminate\Routing\Middleware\SubstituteBindings::class,
\App\Http\Middleware\RedirectIfAuthenticated::class,
\Illuminate\Auth\Middleware\Authenticate::class,
\Illuminate\Auth\Middleware\Authorize::class,
```

如果上述任何中間件應用於初始頁面加載，它們將被保留（重新應用）到任何未來的網絡請求中。

但是，如果您在初始頁面加載時應用了來自應用程序的自定義中間件，並希望在Livewire請求之間保留它，您需要將它添加到應用程序的[Service Provider](https://laravel.com/docs/providers#main-content)中的此列表中，如下所示：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Livewire;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Livewire::addPersistentMiddleware([ // [tl! highlight:2]
            App\Http\Middleware\EnsureUserHasRole::class,
        ]);
    }
}
```

如果在使用`EnsureUserHasRole`中間件的頁面上加載了Livewire組件，則現在將保留並重新應用該中間件到該Livewire組件的任何未來網絡請求中。

> [!warning] 不支持中間件引數
> Livewire目前不支持持久性中間件定義的中間件引數。
>
> ```php
> // Bad...
> Livewire::addPersistentMiddleware(AuthorizeResource::class.':admin');
>
> // Good...
> Livewire::addPersistentMiddleware(AuthorizeResource::class);
> ```


### Applying global Livewire middleware

Alternatively, if you wish to apply specific middleware to every single Livewire update network request, you can do so by registering your own Livewire update route with any middleware you wish:

```php
Livewire::setUpdateRoute(function ($handle) {
	return Route::post('/livewire/update', $handle)
        ->middleware(App\Http\Middleware\LocalizeViewPaths::class);
});
```

發送到服務器的任何Livewire AJAX/fetch請求將使用上述端點，並在處理組件更新之前應用`LocalizeViewPaths`中間件。

在[安裝頁面上自定義更新路由](https://livewire.laravel.com/docs/installation#configuring-livewires-update-endpoint)以瞭解更多。

## 快照校驗和校驗和校驗

在每個Livewire請求之間，Livewire會對Livewire組件進行快照並將其發送到瀏覽器。此快照用於在下一個服務器往返期間重建組件。

[在Hydration文檔中瞭解有關Livewire快照的更多信息。](https://livewire.laravel.com/docs/hydration#the-snapshot)

由於瀏覽器中的fetch請求可能會被攔截並篡改，Livewire會為每個快照生成一個“校驗和”以隨附。

然後，在下一個網絡請求中使用此校驗和來驗證快照在任何方面都沒有更改。

如果 Livewire 發現檢查碼不符，它將拋出 `CorruptComponentPayloadException` 並且請求將失敗。

這可防範任何形式的惡意篡改，否則將導致用戶有權執行或修改不相關的程式碼。
