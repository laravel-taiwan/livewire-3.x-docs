在包含表單的傳統 HTML 頁面中，只有在使用者按下「提交」按鈕時才會提交表單。

然而，Livewire 能夠實現比傳統表單提交更多的功能。您可以即時驗證表單輸入，甚至在使用者輸入時保存表單。

在這些「即時」更新情況下，當表單或表單的子集已更改但尚未保存到資料庫時，向使用者發出信號可能會有所幫助。

當表單包含未保存的輸入時，該表單被視為「未清除」。只有在觸發網路請求以將伺服器狀態與客戶端狀態同步時，它才會變為「清除」。

## 基本用法

Livewire 允許您使用 `wire:dirty` 指示詞輕鬆切換頁面上的視覺元素。

通過將 `wire:dirty` 添加到元素中，您正在指示 Livewire 只在客戶端狀態與伺服器端狀態不一致時顯示該元素。

為了演示，這裡是一個包含視覺「未保存更改...」提示的 `UpdatePost` 表單的示例，該提示向使用者發出表單包含尚未保存的輸入的信號：

```blade
<form wire:submit="update">
    <input type="text" wire:model="title">

    <!-- ... -->

    <button type="submit">Update</button>

    <div wire:dirty>Unsaved changes...</div> <!-- [tl! highlight] -->
</form>
```

因為將 `wire:dirty` 添加到「未保存更改...」訊息中，該訊息將默認隱藏。當使用者開始修改表單輸入時，Livewire 將自動顯示該訊息。

當使用者提交表單時，該訊息將再次消失，因為伺服器/客戶端資料重新同步。

### 移除元素

通過將 `.remove` 修飾符添加到 `wire:dirty`，您可以將一個元素設為默認顯示，並僅在元件具有「未清除」狀態時隱藏它：

```blade
<div wire:dirty.remove>資料已同步...</div>
```

## 定位屬性更新

假設您正在使用 `wire:model.blur` 在使用者離開輸入欄位後立即更新伺服器上的屬性。在這種情況下，您可以通過將 `wire:target` 添加到包含 `wire:dirty` 指示詞的元素來僅為該屬性提供「未清除」指示。

這是一個僅在標題屬性更改時顯示「未清除」指示的示例：

```blade
<form wire:submit="update">
    <input wire:model.blur="title">

    <div wire:dirty wire:target="title">Unsaved title...</div> <!-- [tl! highlight] -->

    <button type="submit">Update</button>
</form>
```

## 切換類別

通常，您可能希望在輸入狀態為「已更改」時，而不是切換整個元素，而是切換單個 CSS 類別。

以下是一個示例，當用戶在輸入欄中輸入時，邊框變為黃色，表示為「未保存」狀態。然後，當用戶從欄位切換時，邊框被移除，表示狀態已在伺服器上保存：

```blade
<input wire:model.blur="title" wire:dirty.class="border-yellow-500">
```
