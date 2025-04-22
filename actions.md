Livewire 操作是您組件上的方法，可以由前端互動觸發，例如點擊按鈕或提交表單。它們提供了開發者體驗，可以直接從瀏覽器調用 PHP 方法，讓您專注於應用程式的邏輯，而無需為連接應用程式的前端和後端而寫冗長的樣板代碼而感到困擾。

讓我們探索一個基本示例，調用 `CreatePost` 組件上的 `save` 操作：

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
        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect()->to('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```blade
<form wire:submit="save"> <!-- [tl! highlight] -->
    <input type="text" wire:model="title">

    <textarea wire:model="content"></textarea>

    <button type="submit">Save</button>
</form>
```

在上面的示例中，當用戶通過點擊 "Save" 提交表單時，`wire:submit`攔截 `submit` 事件並在服務器上調用 `save()` 操作。

實質上，操作是一種輕鬆將用戶互動映射到服務器端功能的方式，而無需手動提交和處理 AJAX 請求的麻煩。

## 刷新組件

有時您可能希望觸發組件的簡單 "刷新"。例如，如果您有一個組件正在檢查數據庫中某些內容的狀態，您可能希望向用戶顯示一個按鈕，讓他們可以刷新顯示的結果。

您可以使用 Livewire 的簡單 `$refresh` 操作，就像您通常引用自己的組件方法一樣：

```blade
<button type="button" wire:click="$refresh">...</button>
```

當觸發 `$refresh` 操作時，Livewire 將進行服務器往返並重新渲染您的組件，而不調用任何方法。

重要的是要注意，當組件刷新時，組件中的任何待定數據更新（例如 `wire:model` 綁定）將應用於服務器。

在內部，Livewire 使用名稱 "commit" 來指代任何時候 Livewire 組件在服務器上更新。如果您更喜歡這個術語，您可以使用 `$commit` 助手，而不是 `$refresh`。這兩者是相同的。

```blade
<button type="button" wire:click="$commit">...</button>
```

您還可以在 Livewire 組件中使用 AlpineJS 觸發組件刷新：

```blade
<button type="button" x-on:click="$wire.$refresh()">...</button>
```

通過閱讀[在 Livewire 內部使用 Alpine 的文檔](/docs/alpine)來了解更多。

## 確認操作

當允許用戶執行危險操作，例如從數據庫中刪除帖子時，您可能希望向他們顯示一個確認警報，以驗證他們希望執行該操作。

Livewire 通過提供一個名為 `wire:confirm` 的簡單指示詞來輕鬆實現這一點：

```blade
<button
    type="button"
    wire:click="delete"
    wire:confirm="Are you sure you want to delete this post?"
>
    Delete post <!-- [tl! highlight:-2,1] -->
</button>
```

當將 `wire:confirm` 添加到包含 Livewire 操作的元素中時，當用戶嘗試觸發該操作時，將顯示一個包含提供的消息的確認對話框。他們可以按“確定”來確認操作，或按“取消”或按下逃脫鍵。

欲了解更多信息，請訪問[`wire:confirm` 文檔頁面](/docs/wire-confirm)。

## 事件監聽器

Livewire 支持各種事件監聽器，允許您對各種類型的用戶交互做出響應：

| 監聽器        | 說明                               |
|-----------------|-------------------------------------------|
| `wire:click`    | 當單擊元素時觸發      |
| `wire:submit`   | 當提交表單時觸發        |
| `wire:keydown`  | 當按下鍵時觸發      |
| `wire:keyup`  | 當釋放鍵時觸發
| `wire:mouseenter`| 當鼠標進入元素時觸發 |
| `wire:*`| `wire:` 後面的任何文本將用作監聽器的事件名稱 |

由於 `wire:` 後面的事件名稱可以是任何內容，Livewire 支持您可能需要監聽的任何瀏覽器事件。例如，要監聽 `transitionend` 事件，您可以使用 `wire:transitionend`。

### 監聽特定按鍵

您可以使用 Livewire 的方便別名之一，將按鍵事件監聽器縮小到特定按鍵或按鍵組合。

例如，當用戶在搜索框中輸入後按下 `Enter` 鍵時執行搜索時，您可以使用 `wire:keydown.enter`：

```blade
<input wire:model="query" wire:keydown.enter="searchPosts">
```

您可以在第一個之後鏈接更多按鍵別名，以監聽按鍵的組合。例如，如果您想要在按下`Shift`鍵時僅監聽`Enter`鍵，您可以這樣寫：

```blade
<input wire:keydown.shift.enter="...">
```

以下是所有可用的按鍵修飾符列表：

| 修飾符         | 按鍵                          |
|---------------|------------------------------|
| `.shift`      | Shift鍵                       |
| `.enter`      | Enter鍵                       |
| `.space`      | Space鍵                       |
| `.ctrl`       | Ctrl鍵                        |
| `.cmd`        | Cmd鍵                         |
| `.meta`       | Mac上的Cmd鍵，Windows上的Windows鍵 |
| `.alt`        | Alt鍵                         |
| `.up`         | 上箭頭                         |
| `.down`       | 下箭頭                         |
| `.left`       | 左箭頭                         |
| `.right`      | 右箭頭                         |
| `.escape`     | Escape鍵                      |
| `.tab`        | Tab鍵                         |
| `.caps-lock`  | Caps Lock鍵                   |
| `.equal`      | 等於鍵，`=`                   |
| `.period`     | 句號鍵，`.`                   |
| `.slash`      | 正斜線鍵，`/`                |

### 事件處理程序修飾符

Livewire還包括一些有用的修飾符，使常見的事件處理任務變得簡單。

例如，如果您需要在事件監聽器內部調用`event.preventDefault()`，您可以在事件名後面加上`.prevent`：

```blade
<input wire:keydown.prevent="...">
```

以下是所有可用的事件監聽器修飾符及其功能的完整列表：

| 修飾符            | 按鍵                                                     |
|------------------|---------------------------------------------------------|
| `.prevent`       | 等效於調用`.preventDefault()`                           |
| `.stop`          | 等效於調用`.stopPropagation()`                          |
| `.window`        | 監聽`window`對象上的事件                                 |
| `.outside`       | 僅監聽元素“外部”的點擊事件                             |
| `.document`      | 監聽`document`對象上的事件                               |
| `.once`          | 確保監聽器僅被調用一次                                   |
| `.debounce`      | 默認情況下將處理程序延遲250毫秒                         |
| `.debounce.100ms`| 將處理程序延遲指定時間                                 |
| `.throttle`      | 至少每250毫秒調用一次處理程序                           |
| `.throttle.100ms`| 在自定義持續時間內調用處理程序                         |
| `.self`          | 只有在事件源於此元素而非子元素時才調用監聽器              |
| `.camel`         | 將事件名轉換為駝峰命名法（`wire:custom-event` -> "customEvent"） |
| `.dot`           | 將事件名轉換為點記號表示法（`wire:custom-event` -> "custom.event"） |
| `.passive`       | `wire:touchstart.passive`不會阻止滾動效能                |
| `.capture`       | 在“捕獲”階段監聽事件                                   |

因為 `wire:` 在底層使用 [Alpine's](https://alpinejs.dev) `x-on` 指示詞，這些修飾符可以透過 Alpine 讓您使用。若要瞭解應何時使用這些修飾符，請參考 [Alpine 事件文件](https://alpinejs.dev/essentials/events)。

### 處理第三方事件

Livewire 也支援監聽由第三方庫觸發的自定義事件。

例如，假設您在專案中使用 [Trix](https://trix-editor.org/) 富文本編輯器，並且想要監聽 `trix-change` 事件以捕獲編輯器的內容。您可以使用 `wire:trix-change` 指示詞來實現這一點：

```blade
<form wire:submit="save">
    <!-- ... -->

    <trix-editor
        wire:trix-change="setPostContent($event.target.value)"
    ></trix-editor>

    <!-- ... -->
</form>
```

在這個例子中，每當觸發 `trix-change` 事件時，將調用 `setPostContent` 動作，並使用 Trix 編輯器的當前值更新 Livewire 元件中的 `content` 屬性。

> [!info] 您可以使用 `$event` 存取事件物件
> 在 Livewire 事件處理程序中，您可以透過 `$event` 存取事件物件。這對於參考事件上的資訊很有用。例如，您可以透過 `$event.target` 存取觸發事件的元素。

> [!warning]
> 上面的 Trix 示範程式碼是不完整的，僅用於展示事件監聽器。如果直接使用，將在每次按鍵時發送網路請求。更有效率的實作方式是：
>
> ```blade
> <trix-editor
>    x-on:trix-change="$wire.content = $event.target.value"
>></trix-editor>
> ```

### Listening for dispatched custom events

If your application dispatches custom events from Alpine, you can also listen for those using Livewire:

```blade
<div wire:custom-event="...">

    <!-- 深度嵌套在此元件中： -->
    <button x-on:click="$dispatch('custom-event')">...</button>

</div>

When the button is clicked in the above example, the `custom-event` event is dispatched and bubbles up to the root of the Livewire component where `wire:custom-event` catches it and invokes a given action.

If you want to listen for an event dispatched somewhere else in your application, you will need to wait instead for the event to bubble up to the `window` object and listen for it there. Fortunately, Livewire makes this easy by allowing you to add a simple `.window` modifier to any event listener:

```blade
<div wire:custom-event.window="...">
    <!-- ... -->
</div>

<!-- 在元件外的頁面某處發送： -->
<button x-on:click="$dispatch('custom-event')">...</button>

### Disabling inputs while a form is being submitted

Consider the `CreatePost` example we previously discussed:

```blade
<form wire:submit="save">
    <input wire:model="title">

    <textarea wire:model="content"></textarea>

    <button type="submit">儲存</button>
</form>

When a user clicks "Save", a network request is sent to the server to call the `save()` action on the Livewire component.

But, let's imagine that a user is filling out this form on a slow internet connection. The user clicks "Save" and nothing happens initially because the network request takes longer than usual. They might wonder if the submission failed and attempt to click the "Save" button again while the first request is still being handled.

In this case, there would be two requests for the same action being processed at the same time.

To prevent this scenario, Livewire automatically disables the submit button and all form inputs inside the `<form>` element while a `wire:submit` action is being processed. This ensures that a form isn't accidentally submitted twice.

To further lessen the confusion for users on slower connections, it is often helpful to show some loading indicator such as a subtle background color change or SVG animation.

Livewire provides a `wire:loading` directive that makes it trivial to show and hide loading indicators anywhere on a page. Here's a short example of using `wire:loading` to show a loading message below the "Save" button:

```blade
<form wire:submit="save">
    <textarea wire:model="content"></textarea>

```blade
<button type="submit">儲存</button>

<span wire:loading>儲存中...</span> <!-- [tl! highlight] -->
</form>

`wire:loading` is a powerful feature with a variety of more powerful features. [Check out the full loading documentation for more information](/docs/wire-loading).

## Passing parameters

Livewire allows you to pass parameters from your Blade template to the actions in your component, giving you the opportunity to provide an action additional data or state from the frontend when the action is called.

For example, let's imagine you have a `ShowPosts` component that allows users to delete a post. You can pass the post's ID as a parameter to the `delete()` action in your Livewire component. Then, the action can fetch the relevant post and delete it from the database:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete($id)
    {
        $post = Post::findOrFail($id);

        $this->authorize('delete', $post);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button> <!-- [tl! highlight] -->
        </div>
    @endforeach
</div>
```

For a post with an ID of 2, the "Delete" button in the Blade template above will render in the browser as:

```blade
<button wire:click="delete(2)">刪除</button>

When this button is clicked, the `delete()` method will be called and `$id` will be passed in with a value of "2".

> [!warning] Don't trust action parameters
> Action parameters should be treated just like HTTP request input, meaning action parameter values should not be trusted. You should always authorize ownership of an entity before updating it in the database.
>
> For more information, consult our documentation regarding [security concerns and best practices](/docs/actions#security-concerns).


As an added convenience, you may automatically resolve Eloquent models by a corresponding model ID that is provided to an action as a parameter. This is very similar to [route model binding](/docs/components#using-route-model-binding). To get started, type-hint an action parameter with a model class and the appropriate model will automatically be retrieved from the database and passed to the action instead of the ID:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete(Post $post) // [tl! highlight]
    {
        $this->authorize('delete', $post);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}

## Dependency injection

You can take advantage of [Laravel's dependency injection](https://laravel.com/docs/controllers#dependency-injection-and-controllers) system by type-hinting parameters in your action's signature. Livewire and Laravel will automatically resolve the action's dependencies from the container:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Repositories\PostRepository;

class ShowPosts extends Component
{
    public function delete(PostRepository $posts, $postId) // [tl! highlight]
    {
        $posts->deletePost($postId);
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button> <!-- [tl! highlight] -->
        </div>
    @endforeach
</div>
```

In this example, the `delete()` method receives an instance of `PostRepository` resolved via [Laravel's service container](https://laravel.com/docs/container#main-content) before receiving the provided `$postId` parameter.

## Calling actions from Alpine

Livewire integrates seamlessly with [Alpine](https://alpinejs.dev/). In fact, under the hood, every Livewire component is also an Alpine component. This means you can take full advantage of Alpine within your components to add JavaScript powered client-side interactivity.

To make this pairing even more powerful, Livewire exposes a magic `$wire` object to Alpine that can be treated as a JavaScript representation of your PHP component. In addition to [accessing and mutating public properties via `$wire`](/docs/properties#accessing-properties-from-javascript), you can call actions. When an action is invoked on the `$wire` object, the corresponding PHP method will be invoked on your backend Livewire component:

```blade
<button x-on:click="$wire.save()">儲存文章</button>

Or, to illustrate a more complex example, you might use Alpine's [`x-intersect`](https://alpinejs.dev/plugins/intersect) utility to trigger a `incrementViewCount()` Livewire action when a given element is visible on the page:

```blade
<div x-intersect="$wire.incrementViewCount()">...</div>

### Passing parameters

Any parameters you pass to the `$wire` method will also be passed to the PHP class method. For example, consider the following Livewire action:

```php
public function addTodo($todo)
{
    $this->todos[] = $todo;
}

Within your component's Blade template, you can invoke this action via Alpine, providing the parameter that should be given to the action:

```blade
<div x-data="{ todo: '' }">
    <input type="text" x-model="todo">

    <button x-on:click="$wire.addTodo(todo)">新增待辦事項</button>
</div>

```php
use App\Models\Post;

public function getPostCount()
{
    return Post::count();
}
```

```blade
<span x-init="$el.innerHTML = await $wire.getPostCount()"></span>
```

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public $bookmarked = false;

    public function mount()
    {
        $this->bookmarked = $this->post->bookmarkedBy(auth()->user());
    }

    public function bookmarkPost()
    {
        $this->post->bookmark(auth()->user());

        $this->bookmarked = $this->post->bookmarkedBy(auth()->user());
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

```blade
<button x-on:click="$wire.$js.bookmark()">Bookmark</button>
```

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    public $title = '';

    public function save()
    {
        // ...

        $this->js('onPostSaved'); // [tl! highlight]
    }
}
```

```blade
<button wire:click="$parent.removePost({{ $post->id }})">Remove</button>
```

```blade
<button wire:click="$set('query', '')">Reset Search</button>
```

```blade
<button wire:click="$refresh">Refresh</button>
```

```blade
<button wire:click="$toggle('sortAsc')">
    Sort {{ $sortAsc ? 'Descending' : 'Ascending' }}
</button>
```

```blade
<button type="submit" wire:click="$dispatch('post-deleted')">Delete Post</button>
```

```blade
<input type="text" wire:keydown.enter="search($event.target.value)">
```

```blade
<button x-on:click="$wire.$refresh()">Refresh</button>
```

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Renderless;
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    #[Renderless] // [tl! highlight]
    public function incrementViewCount()
    {
        $this->post->incrementViewCount();
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    public function incrementViewCount()
    {
        $this->post->incrementViewCount();

        $this->skipRender(); // [tl! highlight]
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}

## Security concerns

Remember that any public method in your Livewire component can be called from the client-side, even without an associated `wire:click` handler that invokes it. In these scenarios, users can still trigger the action from the browser's DevTools.

Below are three examples of easy-to-miss vulnerabilities in Livewire components. Each will show the vulnerable component first and the secure component after. As an exercise, try spotting the vulnerabilities in the first example before viewing the solution.

If you are having difficulty spotting the vulnerabilities and that makes you concerned about your ability to keep your own applications secure, remember all these vulnerabilities apply to standard web applications that use requests and controllers. If you use a component method as a proxy for a controller method, and its parameters as a proxy for request input, you should be able to apply your existing application security knowledge to your Livewire code.

### Always authorize action parameters

Just like controller request input, it's imperative to authorize action parameters since they are arbitrary user input.

Below is a `ShowPosts` component where users can view all their posts on one page. They can delete any post they like using one of the post's "Delete" buttons.

Here is a vulnerable version of the component:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete($id)
    {
        $post = Post::find($id);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>
```

Remember that a malicious user can call `delete()` directly from a JavaScript console, passing any parameters they would like to the action. This means that a user viewing one of their posts can delete another user's post by passing the un-owned post ID to `delete()`.

To protect against this, we need to authorize that the user owns the post about to be deleted:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete($id)
    {
        $post = Post::find($id);

        $this->authorize('delete', $post); // [tl! highlight]

        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}

### Always authorize server-side

Like standard Laravel controllers, Livewire actions can be called by any user, even if there isn't an affordance for invoking the action in the UI.

Consider the following `BrowsePosts` component where any user can view all the posts in the application, but only administrators can delete a post:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        $post = Post::find($id);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            @if (Auth::user()->isAdmin())
                <button wire:click="deletePost({{ $post->id }})">Delete</button>
            @endif
        </div>
    @endforeach
</div>
```

As you can see, only administrators can see the "Delete" button; however, any user can call `deletePost()` on the component from the browser's DevTools.

To patch this vulnerability, we need to authorize the action on the server like so:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) { // [tl! highlight:2]
            abort(403);
        }
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}
```

```php
        $post = Post::find($id);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}

With this change, only administrators can delete a post from this component.

### Keep dangerous methods protected or private

Every public method inside your Livewire component is callable from the client. Even methods you haven't referenced inside a `wire:click` handler. To prevent a user from calling a method that isn't intended to be callable client-side, you should mark them as `protected` or `private`. By doing so, you restrict the visibility of that sensitive method to the component's class and its subclasses, ensuring they cannot be called from the client-side.

Consider the `BrowsePosts` example that we previously discussed, where users can view all posts in your application, but only administrators can delete posts. In the [Always authorize server-side](/docs/actions#always-authorize-server-side) section, we made the action secure by adding server-side authorization. Now imagine we refactor the actual deletion of the post into a dedicated method like you might do in order to simplify your code:

```php
// 警告：此片段示範了不應該做的事情...
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) {
            abort(403);
        }

        $this->delete($id); // [tl! highlight]
    }

    public function delete($postId)  // [tl! highlight:5]
    {
        $post = Post::find($postId);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="deletePost({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>
```

As you can see, we refactored the post deletion logic into a dedicated method named `delete()`. Even though this method isn't referenced anywhere in our template, if a user gained knowledge of its existence, they would be able to call it from the browser's DevTools because it is `public`.

To remedy this, we can mark the method as `protected` or `private`. Once the method is marked as `protected` or `private`, an error will be thrown if a user tries to invoke it:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) {
            abort(403);
        }

        $this->delete($id);
    }

    protected function delete($postId) // [tl! highlight]
    {
        $post = Post::find($postId);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}

<!--
## Applying middleware

By default, Livewire re-applies authentication and authorization related middleware on subsequent requests if those middleware were applied on the initial page load request.

For example, imagine your component is loaded inside a route that is assigned the `auth` middleware and a user's session ends. When the user triggers another action, the `auth` middleware will be re-applied and the user will receive an error.

If there are specific middleware that you would like to apply to a specific action, you may do so with the `#[Middleware]` attribute. For example, we could apply a `LogPostCreation` middleware to an action that creates posts:

```php
<?php

namespace App\Livewire;

use App\Http\Middleware\LogPostCreation;
use Livewire\Component;

class CreatePost extends Component
{
    public $title;

    public $content;

    #[Middleware(LogPostCreation::class)] // [tl! highlight]
    public function save()
    {
        // Create the post...
    }

    // ...
}
```

現在，`LogPostCreation` 中介層將僅應用於 `createPost` 行為，確保只有在用戶創建新文章時才會記錄活動。

-->
