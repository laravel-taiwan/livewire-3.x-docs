Livewire 中的檔案下載與 Laravel 本身的運作方式非常相似。通常情況下，您可以在 Livewire 元件中使用任何 Laravel 下載工具，並且應該能正常運作。

然而，在幕後，檔案下載的處理方式與標準的 Laravel 應用程式有所不同。在使用 Livewire 時，檔案的內容會被 Base64 編碼，然後傳送到前端，並在前端解碼為二進位制以直接從客戶端下載。

## 基本用法

在 Livewire 中觸發檔案下載就像返回標準的 Laravel 下載回應一樣簡單。

以下是一個包含「下載」按鈕以下載發票 PDF 的 `ShowInvoice` 元件的示例：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Invoice;

class ShowInvoice extends Component
{
    public Invoice $invoice;

    public function mount(Invoice $invoice)
    {
        $this->invoice = $invoice;
    }

    public function download()
    {
        return response()->download( // [tl! highlight:2]
            $this->invoice->file_path, 'invoice.pdf'
        );
    }

    public function render()
    {
        return view('livewire.show-invoice');
    }
}
```

```blade
<div>
    <h1>{{ $invoice->title }}</h1>

    <span>{{ $invoice->date }}</span>
    <span>{{ $invoice->amount }}</span>

    <button type="button" wire:click="download">Download</button> <!-- [tl! highlight] -->
</div>
```

就像在 Laravel 控制器中一樣，您也可以使用 `Storage` 門面來啟動下載：

```php
public function download()
{
    return Storage::disk('invoices')->download('invoice.csv');
}
```

## 流式下載

Livewire 也可以進行流式下載；但是，它們並非真正的流式。下載不會觸發，直到檔案的內容被收集並傳送到瀏覽器：

```php
public function download()
{
    return response()->streamDownload(function () {
        echo '...'; // Echo download contents directly...
    }, 'invoice.pdf');
}
```

## 測試檔案下載

Livewire 還提供了一個 `->assertFileDownloaded()` 方法，用於輕鬆測試是否已下載具有特定名稱的檔案：

```php
use App\Models\Invoice;

public function test_can_download_invoice()
{
    $invoice = Invoice::factory();

    Livewire::test(ShowInvoice::class)
        ->call('download')
        ->assertFileDownloaded('invoice.pdf');
}
```

您還可以使用 `->assertNoFileDownloaded()` 方法來測試確保未下載檔案：

```php
use App\Models\Invoice;

public function test_does_not_download_invoice_if_unauthorised()
{
    $invoice = Invoice::factory();

    Livewire::test(ShowInvoice::class)
        ->call('download')
        ->assertNoFileDownloaded();
}
```
