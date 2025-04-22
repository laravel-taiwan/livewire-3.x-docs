## 全域元件掛勾

在您想要將功能或行為新增至應用程式中的每個元件的情況下，您可以使用 Livewire 的 "元件掛勾"。

元件掛勾允許您定義一個單一類別，具有權限外部連接到 Livewire 元件的生命週期（不是在元件類別本身中，也不是在特徵中）。

在我們查看實際使用它們的範例之前，這裡是一個通用的元件掛勾類別，展示您可以在其中使用的每個可用方法：

```php
use Livewire\ComponentHook;

class MyComponentHook extends ComponentHook
{
    public static function provide()
    {
        // Runs once at application boot.
        // Can be used to register any services you may need.
    }

    public function mount($params, $parent)
    {
        // Called when a component is "mounted"
        // 
        // $params: Array of parameters passed into the component
        // $parent: The parent component object if this is a nested component
    }

    public function hydrate($memo)
    {
        // Called when a component is "hydrated"
        //
        // $memo: An associative array of the "dehydrated" metadata for this component
    }

    public function boot()
    {
        // Called when the component boots
    }

    public function update($property, $path, $value)
    {
        // Called before the component updates...

        return function () {
            // Called after the component property has updated...
        };
    }

    public function call($method, $params, $returnEarly)
    {
        // Called before a method on the component is called...

        return function ($returnValue) {
            // Called after a method is called
        };
    }

    public function render($view, $data)
    {
        // Called after "render" is called but before the Blade has been rendered...
        return function ($html) {
            // Called after the component's view has been rendered
        };
    }

    public function dehydrate($context)
    {
        // Called when a component "dehydrates"
    }

    public function exception($e, $stopPropagation)
    {
        // Called if an exception is thrown within a component...
    }
}
```

您可以像這樣從服務提供者註冊元件掛勾，例如您的 `App\Providers\AppServiceProvider`：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Livewire\Livewire;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Livewire::componentHook(MyComponentHook::class);
    }

    // ...
}
```

現在您已經看到了元件掛勾的廣泛概述，這裡有一個更實際的範例，展示如何使用它們為您的應用程式提供有用的功能。

假設您想要支援從任何 Livewire 操作返回 CSV 的功能，並且它將自動觸發檔案下載。例如，您可以從名為 `save` 的方法內部的 `CreatePost` 元件中返回 Csv：

```php
use Livewire\Component;

class CreateUser extends Component
{
    public $username = '';

    public $email = '';

    public function something()
    {
        return new Csv();
    }

    // ...
}
```


```php
<?php

namespace App;

use Livewire\ComponentHook;

class SupportCsvDownloads extends ComponentHook
{
    public function call($method, $params, $returnEarly)
    {
        // Called before a method on the component is called...

        return function ($returnValue) {
            if ($returnValue instanceof Csv) {
                // do something
            }
        };
    }
}
```
