Livewire 提供了一個強大的事件系統，您可以使用它在頁面上的不同組件之間進行通信。因為它在底層使用瀏覽器事件，您也可以使用 Livewire 的事件系統來與 Alpine 組件或甚至純粹的、原生的 JavaScript 進行通信。

要觸發事件，您可以在組件內的任何地方使用 `dispatch()` 方法，並在頁面上的任何其他組件中聆聽該事件。

## 發送事件

要從 Livewire 組件中發送事件，您可以調用 `dispatch()` 方法，並將事件名稱和您想要隨事件一起發送的任何額外數據作為參數傳遞。

以下是從 `CreatePost` 組件中發送 `post-created` 事件的示例：

```php
use Livewire\Component;

class CreatePost extends Component
{
    public function save()
    {
		// ...

		$this->dispatch('post-created'); // [tl! highlight]
    }
}
```

在這個示例中，當調用 `dispatch()` 方法時，將發送 `post-created` 事件，並且頁面上正在聆聽此事件的每個其他組件將收到通知。

您可以通過將數據作為第二個參數傳遞給 `dispatch()` 方法來傳遞事件的附加數據：

```php
$this->dispatch('post-created', title: $post->title);
```

## 聆聽事件

要在 Livewire 組件中聆聽事件，請在您希望在給定事件被發送時調用的方法上方添加 `#[On]` 屬性：

> [!warning] 確保您導入屬性類
> 確保您導入任何屬性類。例如，下面的 `#[On()]` 屬性需要以下導入 `use Livewire\Attributes\On;`。

```php
use Livewire\Component;
use Livewire\Attributes\On; // [tl! highlight]

class Dashboard extends Component
{
	#[On('post-created')] // [tl! highlight]
    public function updatePostList($title)
    {
		// ...
    }
}
```

現在，當從 `CreatePost` 發送 `post-created` 事件時，將觸發網絡請求並調用 `updatePostList()` 操作。

正如您所看到的，隨事件發送的附加數據將作為其第一個參數提供給操作。

### 聆聽動態事件名稱

有時，您可能希望在運行時使用來自您的組件的數據動態生成事件監聽器名稱。

例如，如果您想要將事件監聽器範圍限定為特定的 Eloquent 模型，您可以在發送時將模型的 ID 附加到事件名稱中，如下所示：

```php
use Livewire\Component;

class UpdatePost extends Component
{
    public function update()
    {
        // ...

        $this->dispatch("post-updated.{$post->id}"); // [tl! highlight]
    }
}
```

然後監聽該特定模型：

```php
use Livewire\Component;
use App\Models\Post;
use Livewire\Attributes\On; // [tl! highlight]

class ShowPost extends Component
{
    public Post $post;

	#[On('post-updated.{post.id}')] // [tl! highlight]
    public function refreshPost()
    {
		// ...
    }
}
```

如果上述 `$post` 模型的 ID 為 `3`，`refreshPost()` 方法將只會被名為：`post-updated.3` 的事件觸發。

### 監聽特定子元件的事件

Livewire 允許您直接在 Blade 模板中的個別子元件上監聽事件，如下所示：

```blade
<div>
    <livewire:edit-post @saved="$refresh">

    <!-- ... -->
</div>
```

在上述情況下，如果 `edit-post` 子元件發送了一個 `saved` 事件，父元件的 `$refresh` 將被調用並刷新父元件。

您可以傳遞任何您通常會傳遞給像 `wire:click` 這樣的方法，而不是傳遞 `$refresh`。這裡是一個調用 `close()` 方法的示例，可能會執行類似關閉模態對話框的操作：

```blade
<livewire:edit-post @saved="close">
```

如果子元件隨著請求一起發送參數，例如 `$this->dispatch('saved', postId: 1)`，您可以使用以下語法將這些值傳遞給父元件方法：

```blade
<livewire:edit-post @saved="close($event.detail.postId)">
```

## 使用 JavaScript 與事件互動

當您從應用程式內部的 JavaScript 與 Livewire 的事件系統互動時，Livewire 的事件系統變得更加強大。這使得應用程式中的任何其他 JavaScript 都能與頁面上的 Livewire 元件進行通信。

### 在元件腳本內部監聽事件

您可以輕鬆地從 `@script` 指令中監聽元件模板內的 `post-created` 事件，如下所示：

```html
@script
<script>
    $wire.on('post-created', () => {
        //
    });
</script>
@endscript
```

上述片段將從其註冊的元件中監聽 `post-created` 事件。如果該元件不再在頁面上，則事件監聽器將不再被觸發。

[閱讀更多關於在 Livewire 元件中使用 JavaScript →](/docs/javascript#using-javascript-in-livewire-components)

### 從元件腳本中發送事件

此外，您可以像這樣從元件的 `@script` 內部發送事件：

當上述 `@script` 被執行時，`post-created` 事件將被派發到其所定義的元件內。

要僅將事件派發到腳本所在的元件，而不是頁面上的其他元件（防止事件“冒泡”），您可以使用 `dispatchSelf()`：

```js
$wire.dispatchSelf('post-created');
```

您可以通過將對象作為 `dispatch()` 的第二個參數來傳遞任何額外的參數給事件：

```html
@script
<script>
    $wire.dispatch('post-created', { refreshPosts: true });
</script>
@endscript
```

您現在可以從您的 Livewire 類別以及其他 JavaScript 事件監聽器中訪問這些事件參數。

以下是在 Livewire 類別中接收 `refreshPosts` 參數的示例：

```php
use Livewire\Attributes\On;

// ...

#[On('post-created')]
public function handleNewPost($refreshPosts = false)
{
    //
}
```

您還可以從 JavaScript 事件監聽器的事件 `detail` 屬性中訪問 `refreshPosts` 參數：

```html
@script
<script>
    $wire.on('post-created', (event) => {
        let refreshPosts = event.detail.refreshPosts

        // ...
    });
</script>
@endscript
```

[閱讀更多有關在 Livewire 元件中使用 JavaScript 的資訊 →](/docs/javascript#using-javascript-in-livewire-components)

### 在全域 JavaScript 中聆聽 Livewire 事件

或者，您可以使用 `Livewire.on` 在應用程式中的任何腳本中全域聆聽 Livewire 事件：

```html
<script>
    document.addEventListener('livewire:init', () => {
       Livewire.on('post-created', (event) => {
           //
       });
    });
</script>
```

上面的片段將聆聽從頁面上的任何元件派發的 `post-created` 事件。

如果出於任何原因希望刪除此事件監聽器，您可以使用返回的 `cleanup` 函數來執行：

```html
<script>
    document.addEventListener('livewire:init', () => {
        let cleanup = Livewire.on('post-created', (event) => {
            //
        });

        // Calling "cleanup()" will un-register the above event listener...
        cleanup();
    });
</script>
```

## Alpine 中的事件

因為 Livewire 事件在幕後實際上是普通的瀏覽器事件，您可以使用 Alpine 來聆聽它們或甚至派發它們。

### 在 Alpine 中聆聽 Livewire 事件

例如，我們可以輕鬆使用 Alpine 來聆聽 `post-created` 事件：

```blade
<div x-on:post-created="..."></div>
```

上面的片段將聆聽任何 Livewire 元件發送的 `post-created` 事件，這些元件是分配了 `x-on` 指令的 HTML 元素的子元素。

要從頁面上的任何 Livewire 元件聆聽事件，您可以將 `.window` 添加到監聽器中：

```blade
<div x-on:post-created.window="..."></div>
```

如果您想要存取隨事件一起發送的額外資料，可以使用 `$event.detail` 這樣做：

```blade
<div x-on:post-created="notify('新文章：' + $event.detail.title)"></div>
```

Alpine 文件提供了有關[監聽事件](https://alpinejs.dev/directives/on)的進一步資訊。

### 從 Alpine 發送 Livewire 事件

從 Alpine 發送的任何事件都可以被 Livewire 元件攔截。

例如，我們可以輕鬆地從 Alpine 發送 `post-created` 事件：

```blade
<button @click="$dispatch('post-created')">...</button>
```

類似於 Livewire 的 `dispatch()` 方法，您可以通過將資料作為方法的第二個參數傳遞來傳遞事件的附加資料：

```blade
<button @click="$dispatch('post-created', { title: '文章標題' })">...</button>
```

要了解如何使用 Alpine 發送事件的更多資訊，請參考[Alpine 文件](https://alpinejs.dev/magics/dispatch)。

> [!tip] 您可能不需要事件
> 如果您正在使用事件從子元件調用父元件的行為，您可以在 Blade 模板中直接從子元件使用 `$parent` 調用動作。例如：
>
> ```blade
> <button wire:click="$parent.showCreatePostForm()">Create Post</button>
> ```
>
> [Learn more about $parent](/docs/nesting#directly-accessing-the-parent-from-the-child).

## Dispatching directly to another component

If you want to use events for communicating directly between two components on the page, you can use the `dispatch()->to()` modifier.

Below is an example of the `CreatePost` component dispatching the `post-created` event directly to the `Dashboard` component, skipping any other components listening for that specific event:

```php
use Livewire\Component;

class CreatePost extends Component
{
    public function save()
    {
		// ...

		$this->dispatch('post-created')->to(Dashboard::class);
    }
}

## Dispatching a component event to itself

Using the `dispatch()->self()` modifier, you can restrict an event to only being intercepted by the component it was triggered from:

```php
use Livewire\Component;

class CreatePost extends Component
{
    public function save()
    {
		// ...

		$this->dispatch('post-created')->self();
    }
}

## Dispatching events from Blade templates

You can dispatch events directly from your Blade templates using the `$dispatch` JavaScript function. This is useful when you want to trigger an event from a user interaction, such as a button click:

```blade
<button wire:click="$dispatch('show-post-modal', { id: {{ $post->id }} })">
    編輯文章
</button>

In this example, when the button is clicked, the `show-post-modal` event will be dispatched with the specified data.

If you want to dispatch an event directly to another component you can use the `$dispatchTo()` JavaScript function:

```blade
<button wire:click="$dispatchTo('posts', 'show-post-modal', { id: {{ $post->id }} })">
    編輯文章
</button>

In this example, when the button is clicked, the `show-post-modal` event will be dispatched directly to the `Posts` component.

## Testing dispatched events

To test events dispatched by your component, use the `assertDispatched()` method in your Livewire test. This method checks that a specific event has been dispatched during the component's lifecycle:

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Livewire\CreatePost;
use Livewire\Livewire;

class CreatePostTest extends TestCase
{
    use RefreshDatabase;

    public function test_it_dispatches_post_created_event()
    {
        Livewire::test(CreatePost::class)
            ->call('save')
            ->assertDispatched('post-created');
    }
}

In this example, the test ensures that the `post-created` event is dispatched with the specified data when the `save()` method is called on the `CreatePost` component.

### Testing Event Listeners

To test event listeners, you can dispatch events from the test environment and assert that the expected actions are performed in response to the event:

```php
<?php
```

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Livewire\Dashboard;
use Livewire\Livewire;

class DashboardTest extends TestCase
{
    use RefreshDatabase;

    public function test_it_updates_post_count_when_a_post_is_created()
    {
        Livewire::test(Dashboard::class)
            ->assertSee('Posts created: 0')
            ->dispatch('post-created')
            ->assertSee('Posts created: 1');
    }
}

In this example, the test dispatches the `post-created` event, then checks that the `Dashboard` component properly handles the event and displays the updated count.

## Real-time events using Laravel Echo

Livewire pairs nicely with [Laravel Echo](https://laravel.com/docs/broadcasting#client-side-installation) to provide real-time functionality on your web-pages using WebSockets.

> [!warning] Installing Laravel Echo is a prerequisite
> This feature assumes you have installed Laravel Echo and the `window.Echo` object is globally available in your application. For more information on installing echo, check out the [Laravel Echo documentation](https://laravel.com/docs/broadcasting#client-side-installation).

### Listening for Echo events

Imagine you have an event in your Laravel application named `OrderShipped`:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public Order $order;

    public function broadcastOn()
    {
        return new Channel('orders');
    }
}

You might dispatch this event from another part of your application like so:

```php
use App\Events\OrderShipped;

OrderShipped::dispatch();

If you were to listen for this event in JavaScript using only Laravel Echo, it would look something like this:

```js
Echo.channel('orders')
    .listen('OrderShipped', e => {
        console.log(e.order)
    })

Assuming you have Laravel Echo installed and configured, you can listen for this event from inside a Livewire component.

Below is an example of an `OrderTracker` component that is listening for the `OrderShipped` event in order to show users a visual indication of a new order:

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\On; // [tl! highlight]
use Livewire\Component;

class OrderTracker extends Component
{
    public $showNewOrderNotification = false;

    #[On('echo:orders,OrderShipped')]
    public function notifyNewOrder()
    {
        $this->showNewOrderNotification = true;
    }

    // ...
}

If you have Echo channels with variables embedded in them (such as an Order ID), you can define listeners via the `getListeners()` method instead of the `#[On]` attribute:

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\On; // [tl! highlight]
use Livewire\Component;
use App\Models\Order;

class OrderTracker extends Component
{
    public Order $order;

    public $showOrderShippedNotification = false;

    public function getListeners()
    {
        return [
            "echo:orders.{$this->order->id},OrderShipped" => 'notifyShipped',
        ];
    }

    public function notifyShipped()
    {
        $this->showOrderShippedNotification = true;
    }

    // ...
}

Or, if you prefer, you can use the dynamic event name syntax:

```php
#[On('echo:orders.{order.id},OrderShipped')]
public function notifyNewOrder()
{
    $this->showNewOrderNotification = true;
}
```

```php
#[On('echo:orders.{order.id},OrderShipped')]
public function notifyNewOrder($event)
{
    $order = Order::find($event['orderId']);

    //
}
```

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class OrderTracker extends Component
{
    public $showNewOrderNotification = false;

    public function getListeners()
    {
        return [
            // Public Channel
            "echo:orders,OrderShipped" => 'notifyNewOrder',

            // Private Channel
            "echo-private:orders,OrderShipped" => 'notifyNewOrder',

            // Presence Channel
            "echo-presence:orders,OrderShipped" => 'notifyNewOrder',
            "echo-presence:orders,here" => 'notifyNewOrder',
            "echo-presence:orders,joining" => 'notifyNewOrder',
            "echo-presence:orders,leaving" => 'notifyNewOrder',
        ];
    }

    public function notifyNewOrder()
    {
        $this->showNewOrderNotification = true;
    }
}
```
