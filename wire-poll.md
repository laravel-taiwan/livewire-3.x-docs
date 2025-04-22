輪詢是 Web 應用程式中使用的一種技術，用於定期向伺服器發送請求以獲取更新。這是一種簡單的方式，可以使頁面保持最新，而無需像 [WebSockets](/docs/events#real-time-events-using-laravel-echo) 這樣更複雜的技術。

## 基本用法

在 Livewire 中使用輪詢就像在元素中添加 `wire:poll` 一樣簡單。

以下是一個名為 `SubscriberCount` 的元件示例，顯示用戶的訂閱者數量：

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class SubscriberCount extends Component
{
    public function render()
    {
        return view('livewire.subscriber-count', [
            'count' => Auth::user()->subscribers->count(),
        ]);
    }
}
```

```blade
<div wire:poll> <!-- [tl! highlight] -->
    訂閱者數量：{{ $count }}
</div>
```

通常，此元件將顯示用戶的訂閱者數量，並且在頁面刷新之前永遠不會更新。但是，由於在元件模板上使用了 `wire:poll`，此元件現在將每 `2.5` 秒刷新一次，保持訂閱者數量最新。

您還可以通過將值傳遞給 `wire:poll` 來指定在輪詢間隔上觸發的動作：

```blade
<div wire:poll="refreshSubscribers">
    訂閱者數量：{{ $count }}
</div>
```

現在，元件上的 `refreshSubscribers()` 方法將每 `2.5` 秒被調用一次。

## 時間控制

輪詢的主要缺點是可能會消耗資源。如果在使用輪詢的頁面上有一千位訪客，將每 `2.5` 秒觸發一千個網路請求。

在這種情況下減少請求的最佳方式是簡單地延長輪詢間隔。

您可以通過將所需的持續時間附加到 `wire:poll` 來手動控制元件輪詢的頻率，如下所示：

```blade
<div wire:poll.15s> <!-- 以秒為單位... -->

<div wire:poll.15000ms> <!-- 以毫秒為單位... -->
```

## 背景節流

為了進一步減少伺服器請求，Livewire 在頁面處於背景時會自動節流輪詢。例如，如果用戶在不同的瀏覽器分頁中保持頁面開啟，Livewire 將將輪詢請求數量減少 95%，直到用戶重新訪問該分頁。

如果您希望退出此行為並保持連續輪詢，即使分頁處於背景中，您可以將 `.keep-alive` 修飾符添加到 `wire:poll` 中。

```blade
<div wire:poll.keep-alive>
```

##  視口節流

另一個可以採取的措施是僅在必要時進行輪詢，方法是將 `.visible` 修飾符添加到 `wire:poll`。`.visible` 修飾符指示 Livewire 只在組件在頁面上可見時進行輪詢：

```blade
<div wire:poll.visible>
```

如果使用 `wire:visible` 的組件位於長頁面的底部，直到用戶將其滾動到視口中，它才會開始輪詢。當用戶滾動離開時，它將再次停止輪詢。
```
