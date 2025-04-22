因為 Livewire 元件在請求之間被脫水化（序列化）為 JSON，然後再被 PHP 元件水合化（反序列化），所以它們的屬性需要是可 JSON 序列化的。

在原生情況下，PHP 可將大多數基本值輕鬆序列化為 JSON。然而，為了讓 Livewire 元件支援更複雜的屬性類型（如模型、集合、Carbon 實例和可字串化對象），需要一個更強大的系統。

因此，Livewire 提供了一個名為「合成器」的擴展點，允許用戶支援他們希望的任何自定義屬性類型。

> [!tip] 確保您首先了解水合化
> 在使用合成器之前，充分了解 Livewire 的水合化系統是有幫助的。您可以通過閱讀 [水合化文檔](/docs/hydration) 來了解更多。

## 理解合成器

在探索創建自定義合成器之前，讓我們首先看一下 Livewire 用於支援 [Laravel 可字串化對象](https://laravel.com/docs/strings) 的內部合成器。

假設您的應用程序包含以下 `CreatePost` 元件：

```php
class CreatePost extends Component
{
    public $title = '';
}
```

在請求之間，Livewire 可能將此元件的狀態序列化為以下 JSON 對象：

```js
state: { title: '' },
```

現在，考慮一個更高級的示例，其中 `$title` 屬性值是一個可字串化對象，而不是一個普通字符串：

```php
class CreatePost extends Component
{
    public $title = '';

    public function mount()
    {
        $this->title = str($this->title);
    }
}
```

現在，代表此元件狀態的脫水 JSON 包含一個 [元數組](/docs/hydration#deeply-nested-tuples) 而不是一個普通的空字符串：

```js
state: { title: ['', { s: 'str' }] },
```

Livewire 現在可以使用此元數組在下一個請求中將 `$title` 屬性水合為一個可字串化對象。

既然您已經看到了合成器的外部效果，這裡是 Livewire 內部可字串化合成器的實際源代碼：

```php
use Illuminate\Support\Stringable;

class StringableSynth extends Synth
{
    public static $key = 'str';

    public static function match($target)
    {
        return $target instanceof Stringable;
    }

    public function dehydrate($target)
    {
        return [$target->__toString(), []];
    }

    public function hydrate($value)
    {
        return str($value);
    }
}
```

讓我們一步步來分解這個代碼。

首先是 `$key` 屬性：

```php
public static $key = 'str';
```

每個合成器必須包含一個靜態 `$key` 屬性，Livewire 使用它將像 `['', { s: 'str' }]` 這樣的 [元數組](/docs/hydration#deeply-nested-tuples) 轉換回一個可字串化對象。正如您可能注意到的，每個元數組都有一個引用此鍵的 `s` 鍵。

相反地，當 Livewire 在對屬性進行脫水時，它將使用 synth 的靜態 `match()` 函數來識別是否這個特定的合成器是脫水當前屬性的良好候選者（`$target` 為屬性的當前值）：

```php
public static function match($target)
{
    return $target instanceof Stringable;
}
```

如果 `match()` 返回 true，將使用 `dehydrate()` 方法將屬性的 PHP 值作為輸入並返回可轉換為 JSON 的[元數據](/docs/hydration#deeply-nested-tuples)元組：

```php
public function dehydrate($target)
{
    return [$target->__toString(), []];
}
```

現在，在下一個請求的開始時，當這個合成器已被元組中的 `{ s: 'str' }` 鍵匹配時，將調用 `hydrate()` 方法並傳遞屬性的原始 JSON 表示，期望它返回完整的 PHP 兼容值以分配給屬性。

```php
public function hydrate($value)
{
    return str($value);
}
```

## 註冊自定義合成器

為了演示如何編寫自己的合成器以支持自定義屬性，我們將使用以下 `UpdateProperty` 組件作為示例：

```php
class UpdateProperty extends Component
{
    public Address $address;

    public function mount()
    {
        $this->address = new Address();
    }
}
```

這是 `Address` 類的原始碼：

```php
namespace App\Dtos\Address;

class Address
{
    public $street = '';
    public $city = '';
    public $state = '';
    public $zip = '';
}
```

為了支持類型為 `Address` 的屬性，我們可以使用以下合成器：

```php
use App\Dtos\Address;

class AddressSynth extends Synth
{
    public static $key = 'address';

    public static function match($target)
    {
        return $target instanceof Address;
    }

    public function dehydrate($target)
    {
        return [[
            'street' => $target->street,
            'city' => $target->city,
            'state' => $target->state,
            'zip' => $target->zip,
        ], []];
    }

    public function hydrate($value)
    {
        $instance = new Address;

        $instance->street = $value['street'];
        $instance->city = $value['city'];
        $instance->state = $value['state'];
        $instance->zip = $value['zip'];

        return $instance;
    }
}
```

要使其在應用程序中全局可用，您可以使用 Livewire 的 `propertySynthesizer` 方法在服務提供者的啟動方法中註冊合成器：

```php
class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Livewire::propertySynthesizer(AddressSynth::class);
    }
}
```

## 支持數據綁定

使用上面的 `UpdateProperty` 示例，您可能希望直接支持 `wire:model` 綁定到 `Address` 對象的屬性。合成器允許您使用 `get()` 和 `set()` 方法來支持此功能：

```php
use App\Dtos\Address;

class AddressSynth extends Synth
{
    public static $key = 'address';

    public static function match($target)
    {
        return $target instanceof Address;
    }

    public function dehydrate($target)
    {
        return [[
            'street' => $target->street,
            'city' => $target->city,
            'state' => $target->state,
            'zip' => $target->zip,
        ], []];
    }

    public function hydrate($value)
    {
        $instance = new Address;

        $instance->street = $value['street'];
        $instance->city = $value['city'];
        $instance->state = $value['state'];
        $instance->zip = $value['zip'];

        return $instance;
    }

    public function get(&$target, $key) // [tl! highlight:8]
    {
        return $target->{$key};
    }

    public function set(&$target, $key, $value)
    {
        $target->{$key} = $value;
    }
}
```
