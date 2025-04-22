## 創建您的第一個測試

通過將 `--test` 標誌附加到 `make:livewire` 命令，您可以生成一個測試文件以及一個組件：

```shell
php artisan make:livewire create-post --test
```

除了生成組件文件本身之外，上述命令還將生成以下測試文件 `tests/Feature/Livewire/CreatePostTest.php`：

如果您想要創建一個 [Pest PHP](https://pestphp.com/) 測試，您可以向 `make:livewire` 命令提供 `--pest` 選項：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_renders_successfully()
    {
        Livewire::test(CreatePost::class)
            ->assertStatus(200);
    }
}
```

當然，您始終可以手動創建這些文件，甚至在 Laravel 應用程序中的任何現有 PHPUnit 測試中使用 Livewire 的測試工具。

在繼續閱讀之前，您可能希望熟悉一下[Laravel 內建的測試功能](https://laravel.com/docs/testing)。

## 測試頁面包含一個組件

您可以編寫的最簡單的 Livewire 測試是斷言應用程序中的特定端點包含並成功呈現給定的 Livewire 組件。

Livewire 提供了一個 `assertSeeLivewire()` 方法，可以從任何 Laravel 測試中使用：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_component_exists_on_the_page()
    {
        $this->get('/posts/create')
            ->assertSeeLivewire(CreatePost::class);
    }
}
```

> [!tip] 這些被稱為煙霧測試
> 煙霧測試是廣泛的測試，確保應用程序中沒有災難性問題。儘管這似乎是一個不值得撰寫的測試，但這些是您可以撰寫的一些最有價值的測試，因為它們需要非常少的維護工作，並為您提供了對應用程序將成功呈現且沒有主要錯誤的基本信心水平。

## 測試視圖

Livewire 提供了一個簡單而強大的工具，用於斷言組件呈現輸出中的文本存在：`assertSee()`。

以下是使用 `assertSee()` 確保數據庫中的所有文章都顯示在頁面上的示例：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\ShowPosts;
use Livewire\Livewire;
use App\Models\Post;
use Tests\TestCase;

class ShowPostsTest extends TestCase
{
    public function test_displays_posts()
    {
        Post::factory()->make(['title' => 'On bathing well']);
        Post::factory()->make(['title' => 'There\'s no time like bathtime']);

        Livewire::test(ShowPosts::class)
            ->assertSee('On bathing well')
            ->assertSee('There\'s no time like bathtime');
    }
}
```

### 從視圖斷言數據

除了斷言呈現視圖的輸出之外，有時測試傳遞到視圖的數據也很有幫助。

這裡是相同的測試，但測試視圖數據而不是渲染輸出：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\ShowPosts;
use Livewire\Livewire;
use App\Models\Post;
use Tests\TestCase;

class ShowPostsTest extends TestCase
{
    public function test_displays_all_posts()
    {
        Post::factory()->make(['title' => 'On bathing well']);
        Post::factory()->make(['title' => 'The bathtub is my sanctuary']);

        Livewire::test(ShowPosts::class)
            ->assertViewHas('posts', function ($posts) {
                return count($posts) == 2;
            });
    }
}
```

如您所見，`assertViewHas()` 允許您控制要對指定數據進行哪些斷言。

如果您寧願進行簡單斷言，例如確保一個視圖數據與給定值匹配，則可以將該值直接作為傳遞給 `assertViewHas()` 方法的第二個參數。

例如，假設您有一個帶有名為 `$postCount` 的變數的組件被傳遞到視圖中，您可以這樣對其字面值進行斷言：

```php
$this->assertViewHas('postCount', 3)
```

## 設置已驗證用戶

大多數 Web 應用程序要求用戶在使用之前登錄。與在測試開始時手動驗證假用戶不同，Livewire 提供了一個 `actingAs()` 方法。

以下是一個示例測試，其中多個用戶都有帖子，但已驗證的用戶只能看到自己的帖子：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\ShowPosts;
use Livewire\Livewire;
use App\Models\User;
use App\Models\Post;
use Tests\TestCase;

class ShowPostsTest extends TestCase
{
    public function test_user_only_sees_their_own_posts()
    {
        $user = User::factory()
            ->has(Post::factory()->count(3))
            ->create();

        $stranger = User::factory()
            ->has(Post::factory()->count(2))
            ->create();

        Livewire::actingAs($user)
            ->test(ShowPosts::class)
            ->assertViewHas('posts', function ($posts) {
                return count($posts) == 3;
            });
    }
}
```

## 測試屬性

Livewire 還提供了有助於在組件內設置和斷言屬性的測試工具。

組件屬性通常在用戶與包含 `wire:model` 的表單輸入進行交互時在應用程序中更新。但是，因為測試通常不會在實際瀏覽器中輸入，Livewire 允許您使用 `set()` 方法直接設置屬性。

以下是使用 `set()` 更新 `CreatePost` 組件的 `$title` 屬性的示例：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_can_set_title()
    {
        Livewire::test(CreatePost::class)
            ->set('title', 'Confessions of a serial soaker')
            ->assertSet('title', 'Confessions of a serial soaker');
    }
}
```

### 初始化屬性

通常，Livewire 組件從父組件或路由參數傳入數據。由於 Livewire 組件是獨立測試的，您可以使用 `Livewire::test()` 方法的第二個參數手動將數據傳遞給它們：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\UpdatePost;
use Livewire\Livewire;
use App\Models\Post;
use Tests\TestCase;

class UpdatePostTest extends TestCase
{
    public function test_title_field_is_populated()
    {
        $post = Post::factory()->make([
            'title' => 'Top ten bath bombs',
        ]);

        Livewire::test(UpdatePost::class, ['post' => $post])
            ->assertSet('title', 'Top ten bath bombs');
    }
}
```

正在測試的基礎組件（`UpdatePost`）將通過其 `mount()` 方法接收 `$post`。讓我們查看 `UpdatePost` 的源代碼，以更清晰地了解此功能：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class UpdatePost extends Component
{
	public Post $post;

    public $title = '';

	public function mount(Post $post)
	{
		$this->post = $post;

		$this->title = $post->title;
	}

	// ...
}
```

### 設置URL參數

如果您的 Livewire 元件依賴於頁面 URL 中特定的查詢參數，您可以使用 `withQueryParams()` 方法來手動設置測試中的查詢參數。

以下是一個基本的 `SearchPosts` 元件，它使用 [Livewire 的 URL 功能](/docs/url) 來在查詢字串中存儲和追踪當前的搜索查詢：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Url;
use App\Models\Post;

class SearchPosts extends Component
{
    #[Url] // [tl! highlight]
    public $search = '';

    public function render()
    {
        return view('livewire.search-posts', [
            'posts' => Post::search($this->search)->get(),
        ]);
    }
}
```

如您所見，上面的 `$search` 屬性使用 Livewire 的 `#[Url]` 屬性來表示其值應存儲在 URL 中。

以下是一個示例，展示如何模擬在具有特定 URL 查詢參數的頁面上加載此元件的情況：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\SearchPosts;
use Livewire\Livewire;
use App\Models\Post;
use Tests\TestCase;

class SearchPostsTest extends TestCase
{
    public function test_can_search_posts_via_url_query_string()
    {
        Post::factory()->create(['title' => 'Testing the first water-proof hair dryer']);
        Post::factory()->create(['title' => 'Rubber duckies that actually float']);

        Livewire::withQueryParams(['search' => 'hair'])
            ->test(SearchPosts::class)
            ->assertSee('Testing the first')
            ->assertDontSee('Rubber duckies');
    }
}
```

### 設置Cookie

如果您的 Livewire 元件依賴於 Cookie，您可以使用 `withCookie()` 或 `withCookies()` 方法來手動設置測試中的 Cookie。

以下是一個基本的 `Cart` 元件，在加載時從 Cookie 中加載折扣標記：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Url;
use App\Models\Post;

class Cart extends Component
{
    public $discountToken;

    public mount()
    {
        $this->discountToken = request()->cookie('discountToken');
    }
}
```

如您所見，上面的 `$discountToken` 屬性從請求中的 Cookie 中獲取其值。

以下是一個示例，展示如何模擬在具有 Cookie 的頁面上加載此元件的情況：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\Cart;
use Livewire\Livewire;
use Tests\TestCase;

class CartTest extends TestCase
{
    public function test_can_load_discount_token_from_a_cookie()
    {
        Livewire::withCookies(['discountToken' => 'CALEB2023'])
            ->test(Cart::class)
            ->assertSet('discountToken', 'CALEB2023');
    }
}
```

## 呼叫動作

Livewire 動作通常是從前端使用類似 `wire:click` 的方式調用的。

由於 Livewire 元件測試不使用實際的瀏覽器，您可以使用 `call()` 方法在測試中觸發動作。

以下是一個示例，展示了一個 `CreatePost` 元件使用 `call()` 方法觸發 `save()` 動作：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use App\Models\Post;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_can_create_post()
    {
        $this->assertEquals(0, Post::count());

        Livewire::test(CreatePost::class)
            ->set('title', 'Wrinkly fingers? Try this one weird trick')
            ->set('content', '...')
            ->call('save');

        $this->assertEquals(1, Post::count());
    }
}
```

在上面的測試中，我們斷言調用 `save()` 會在資料庫中創建一個新的文章。

您也可以通過將額外參數傳遞給 `call()` 方法來將參數傳遞給動作：

```php
->call('deletePost', $postId);
```

### 確認

要測試是否拋出了驗證錯誤，您可以使用 Livewire 的 `assertHasErrors()` 方法：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_title_field_is_required()
    {
        Livewire::test(CreatePost::class)
            ->set('title', '')
            ->call('save')
            ->assertHasErrors('title');
    }
}
```

如果您想測試特定的驗證規則失敗，您可以傳遞一個規則陣列：

```php
$this->assertHasErrors(['title' => ['required']]);
```

或者，如果您寧願斷言驗證訊息存在，您也可以這樣做：

```php
$this->assertHasErrors(['title' => ['The title field is required.']]);
```

### 授權

在 Livewire 元件中授權依賴不受信任的輸入的操作是[重要的](/docs/properties#authorizing-the-input)。Livewire 提供 `assertUnauthorized()` 和 `assertForbidden()` 方法來確保身份驗證或授權檢查失敗：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\UpdatePost;
use Livewire\Livewire;
use App\Models\User;
use App\Models\Post;
use Tests\TestCase;

class UpdatePostTest extends TestCase
{
    public function test_cant_update_another_users_post()
    {
        $user = User::factory()->create();
        $stranger = User::factory()->create();

        $post = Post::factory()->for($stranger)->create();

        Livewire::actingAs($user)
            ->test(UpdatePost::class, ['post' => $post])
            ->set('title', 'Living the lavender life')
            ->call('save')
            ->assertUnauthorized();

        Livewire::actingAs($user)
            ->test(UpdatePost::class, ['post' => $post])
            ->set('title', 'Living the lavender life')
            ->call('save')
            ->assertForbidden();
    }
}
```

如果您喜歡，您也可以測試元件中的操作可能觸發的明確狀態碼，使用 `assertStatus()`：

```php
->assertStatus(401); // 未經授權
->assertStatus(403); // 禁止
```

### 重新導向

您可以使用 `assertRedirect()` 方法來測試 Livewire 操作是否執行了重新導向：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_redirected_to_all_posts_after_creating_a_post()
    {
        Livewire::test(CreatePost::class)
            ->set('title', 'Using a loofah doesn\'t make you aloof...ugh')
            ->set('content', '...')
            ->call('save')
            ->assertRedirect('/posts');
    }
}
```

作為一種便利，您可以斷言用戶被重新導向到特定頁面元件，而不是硬編碼的 URL。

```php
->assertRedirect(CreatePost::class);
```

### 事件

要斷言從您的元件內部發送了一個事件，您可以使用 `->assertDispatched()` 方法：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_creating_a_post_dispatches_event()
    {
        Livewire::test(CreatePost::class)
            ->set('title', 'Top 100 bubble bath brands')
            ->set('content', '...')
            ->call('save')
            ->assertDispatched('post-created');
    }
}
```

通常有助於測試兩個元件之間可以通過發送和監聽事件來溝通。使用 `dispatch()` 方法，讓我們模擬一個 `CreatePost` 元件發送一個 `create-post` 事件。然後，我們將斷言一個監聽該事件的 `PostCountBadge` 元件適當地更新其帖子計數：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\PostCountBadge;
use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class PostCountBadgeTest extends TestCase
{
    public function test_post_count_is_updated_when_event_is_dispatched()
    {
        $badge = Livewire::test(PostCountBadge::class)
            ->assertSee("0");

        Livewire::test(CreatePost::class)
            ->set('title', 'Tear-free: the greatest lie ever told')
            ->set('content', '...')
            ->call('save')
            ->assertDispatched('post-created');

        $badge->dispatch('post-created')
            ->assertSee("1");
    }
}
```

有時斷言一個事件被發送時帶有一個或多個參數可能很有用。讓我們看一下一個名為 `ShowPosts` 的元件，它發送一個名為 `banner-message` 的事件，帶有一個名為 `message` 的參數：

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\ShowPosts;
use Livewire\Livewire;
use Tests\TestCase;

class ShowPostsTest extends TestCase
{
    public function test_notification_is_dispatched_when_deleting_a_post()
    {
        Livewire::test(ShowPosts::class)
            ->call('delete', postId: 3)
            ->assertDispatched('notify',
                message: 'The post was deleted',
            );
    }
}
```

如果您的元件發送一個事件，其中參數值必須有條件地被斷言，您可以將閉包作為第二個參數傳遞給 `assertDispatched` 方法，如下所示。它接收事件名稱作為第一個參數，以及包含參數的陣列作為第二個參數。請確保閉包返回一個布林值。

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\ShowPosts;
use Livewire\Livewire;
use Tests\TestCase;

class ShowPostsTest extends TestCase
{
    public function test_notification_is_dispatched_when_deleting_a_post()
    {
        Livewire::test(ShowPosts::class)
            ->call('delete', postId: 3)
            ->assertDispatched('notify', function($eventName, $params) {
                return ($params['message'] ?? '') === 'The post was deleted';
            })
    }
}
```

## 所有可用的測試工具

Livewire 提供了許多更多的測試工具。以下是您可以使用的每個測試方法的詳細清單，並簡要描述了它的預期用法：

### 設定方法
| 方法                                                  | 說明                                                                                                      |
|---------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `Livewire::test(CreatePost::class)`                      | 測試 `CreatePost` 元件 |
| `Livewire::test(UpdatePost::class, ['post' => $post])`                      | 測試帶有 `post` 參數的 `UpdatePost` 元件（通過 `mount()` 方法接收） |
| `Livewire::actingAs($user)`                      | 將提供的使用者設置為會話的驗證使用者 |
| `Livewire::withQueryParams(['search' => '...'])`                      | 將測試的 `search` URL 查詢參數設置為提供的值（例如 `?search=...`）。通常在使用 Livewire 的 [`#[Url]` 屬性](/docs/url) 的屬性上下文中 |
| `Livewire::withCookie('color', 'blue')`                      | 將測試的 `color` cookie 設置為提供的值（`blue`）。 |
| `Livewire::withCookies(['color' => 'blue', 'name' => 'Taylor])`                      | 將測試的 `color` 和 `name` cookies 設置為提供的值（`blue`、`Taylor`）。 |
| `Livewire::withHeaders(['X-COLOR' => 'blue', 'X-NAME' => 'Taylor])`                      | 將測試的 `X-COLOR` 和 `X-NAME` 標頭設置為提供的值（`blue`、`Taylor`）。 |
| `Livewire::withoutLazyLoading()`                      | 在此測試下禁用延遲加載以及所有子元件。 |

### 與元件互動
| 方法                                                  | 說明                                                                                                      |
|---------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `set('title', '...')`                      | 將 `title` 屬性設置為提供的值 |
| `set(['title' => '...', ...])`                      | 使用關聯陣列設置多個元件屬性 |
| `toggle('sortAsc')`                      | 在 `sortAsc` 屬性之間切換 `true` 和 `false`  |
| `call('save')`                      | 呼叫 `save` 行為 / 方法 |
| `call('remove', $post->id)`                      | 呼叫 `remove` 方法並將 `$post->id` 作為第一個參數傳遞（也接受後續參數） |
| `refresh()`                      | 觸發元件重新渲染 |
| `dispatch('post-created')`                      | 從元件發送 `post-created` 事件  |
| `dispatch('post-created', postId: $post->id)`                      | 使用 `$post->id` 作為附加參數發送 `post-created` 事件（從 Alpine 的 `$event.detail`） |

### 斷言
| 方法                                                | 說明                                                                                                                                                                          |
|-------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `assertSet('title', '...')`                           | 斷言 `title` 屬性設置為提供的值                                                                                                                        |
| `assertNotSet('title', '...')`                        | 斷言 `title` 屬性未設置為提供的值                                                                                                                    |
| `assertSetStrict('title', '...')`                     | 使用嚴格比較斷言 `title` 屬性設置為提供的值                                                                                                                        |
| `assertNotSetStrict('title', '...')`                  | 使用嚴格比較斷言 `title` 屬性未設置為提供的值                                                                                                                  |
| `assertReturned('...')`                               | 斷言先前的 `->call(...)` 返回了給定值
| `assertCount('posts', 3)`                             | 斷言 `posts` 屬性是一個類似陣列的值，其中有 `3` 個項目                                                                                                         |
| `assertSnapshotSet('date', '08/26/1990')`             | 斷言 `date` 屬性的原始 / 脫水值（來自 JSON）設置為 `08/26/1990`。在 `date` 的情況下，與針對脫水後的 `DateTime` 實例進行斷言的替代方法 |
| `assertSnapshotNotSet('date', '08/26/1990')`          | 斷言 `date` 的原始 / 脫水值不等於提供的值                                                                                                       |
| `assertSee($post->title)`                             | 斷言元件的渲染 HTML 包含提供的值                                                                                                           |
| `assertDontSee($post->title)`                         | 斷言渲染的 HTML 不包含提供的值                                                                                                                    |
| `assertSeeHtml('<div>...</div>')`                     | 斷言提供的字串文字在渲染的 HTML 中，不對 HTML 字元進行轉義（與 `assertSee` 不同，後者默認會對提供的字元進行轉義） |
| `assertDontSeeHtml('<div>...</div>')`                 | 斷言提供的字串在渲染的 HTML 中                                                                                                                         |
| `assertSeeText($post->title)`                         | 斷言提供的字串包含在渲染的 HTML 文本中。在進行斷言之前，渲染的內容將傳遞給 `strip_tags` PHP 函數                                                                                          |
| `assertDontSeeText($post->title)`                     | 斷言提供的字串不包含在渲染的 HTML 文本中。在進行斷言之前，渲染的內容將傳遞給 `strip_tags` PHP 函數                                                                                |
| `assertSeeInOrder(['...', '...'])`                    | 斷言提供的字串按順序出現在元件的渲染 HTML 輸出中                                                                                        |
| `assertSeeHtmlInOrder([$firstString, $secondString])` | 斷言提供的 HTML 字串按順序出現在元件的渲染輸出中                                                                                        |
| `assertDispatched('post-created')`                    | 斷言元件已發送給定事件                                                                                                                     |
| `assertNotDispatched('post-created')`                 | 斷言元件未發送給定事件                                                                                                                 |
| `assertHasErrors('title')`                            | 斷言 `title` 屬性的驗證失敗                                                                                                                           |
| `assertHasErrors(['title' => ['required', 'min:6']])`   | 斷言提供的驗證規則對 `title` 屬性失敗                                                                                                            |
| `assertHasNoErrors('title')`                          | 斷言 `title` 屬性沒有驗證錯誤                                                                                                                |
| `assertHasNoErrors(['title' => ['required', 'min:6']])` | 斷言提供的驗證規則對 `title` 屬性沒有失敗                                                                                                      |
| `assertRedirect()`                                    | 斷言從元件內觸發了重定向                                                                                                                  |
| `assertRedirect('/posts')`                            | 斷言元件觸發了重定向到 `/posts` 端點                                                                                                                   |
| `assertRedirect(ShowPosts::class)`                    | 斷言元件觸發了重定向到 `ShowPosts` 元件                                                                                                          |
| `assertRedirectToRoute('name', ['parameters'])`       | 斷言元件觸發了重定向到給定路由                                                                                                                |
| `assertNoRedirect()`                                  | 斷言未觸發任何重定向                                                                                                                                           |
| `assertViewHas('posts')`                              | 斷言 `render()` 方法已將 `posts` 項目傳遞給視圖數據                                                                                                         |
| `assertViewHas('postCount', 3)`                       | 斷言已將 `postCount` 變數傳遞給視圖，值為 `3`                                                                                                   |
| `assertViewHas('posts', function ($posts) { ... })`   | 斷言 `posts` 視圖數據存在，並且通過提供的回調函數聲明的任何斷言                                                                                                         |
| `assertViewIs('livewire.show-posts')`                 | 斷言元件的渲染方法返回了提供的視圖名                                                                                                            |
| `assertFileDownloaded()`                              | 斷言已觸發文件下載                                                                                                                                       |
| `assertFileDownloaded($filename)`                     | 斷言已觸發與提供的文件名匹配的文件下載                                                                                                       |
| `assertNoFileDownloaded()`                            | 斷言未觸發任何文件下載                                                                                                                                       |
| `assertUnauthorized()`                                | 斷言在元件內拋出了授權異常（狀態碼：401）                                                                                       |
| `assertForbidden()`                                   | 斷言已觸發帶有狀態碼 403 的錯誤響應                                                                                                                |
| `assertStatus(500)`                                   | 斷言最新的回應與提供的狀態碼匹配                                                                                                                     |

Please paste the Markdown content you need to be translated into traditional Chinese.
