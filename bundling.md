每次 Livewire 中的組件更新都會觸發一個網路請求。預設情況下，當多個組件同時觸發更新時，它們會被打包成一個請求。

這導致與伺服器的網路連線減少，可以大幅減少伺服器負載。

除了性能提升外，這也內部解鎖了需要多個組件協作的功能（[反應性屬性](/docs/nesting#reactive-props)、[可建模屬性](/docs/nesting#binding-to-child-data-using-wiremodel) 等）。

然而，有時出於性能原因希望禁用此打包功能。以下頁面概述了在 Livewire 中自定義此行為的各種方法。

## 隔離組件請求

通過使用 Livewire 的 `#[Isolate]` 類屬性，您可以將一個組件標記為“隔離”。這意味著每當該組件進行伺服器往返時，它將嘗試與其他組件請求隔離開來。

如果更新很昂貴，您寧願將此組件的更新與其他更新並行執行，這將非常有用。例如，如果多個組件正在使用 `wire:poll` 或在頁面上聆聽事件，您可能希望隔離特定組件，其更新很昂貴，否則將阻礙整個請求。

```php
use Livewire\Attributes\Isolate;
use Livewire\Component;

#[Isolate] // [tl! highlight]
class ShowPost extends Component
{
    // ...
}
```

通過添加 `#[Isolate]` 屬性，此組件的請求將不再與其他組件更新打包。

## 懶惰組件默認為隔離狀態

當單個頁面上有許多“懶惰”加載的組件（使用 `#[Lazy]` 屬性）時，通常希望它們的請求被隔離並且並行發送。因此，Livewire 默認隔離懶惰更新。

如果您希望禁用此行為，您可以將 `isolate: false` 參數傳遞給 `#[Lazy]` 屬性，如下所示：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Lazy;

#[Lazy(isolate: false)] // [tl! highlight]
class Revenue extends Component
{
    // ...
}
```

現在，如果同一頁面上有多個 `Revenue` 組件，所有十個更新將被打包並作為單個、懶惰加載的網路請求發送到伺服器。
