Livewire 提供了在組件中強大的文件上傳支持。

首先，將 `WithFileUploads` trait 添加到您的組件中。一旦將此 trait 添加到您的組件中，您可以在文件輸入上使用 `wire:model`，就像它們是任何其他輸入類型一樣，Livewire 將為您處理其餘部分。

這裡是一個處理上傳照片的簡單組件示例：

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\WithFileUploads;
use Livewire\Attributes\Validate;

class UploadPhoto extends Component
{
    use WithFileUploads;

    #[Validate('image|max:1024')] // 1MB Max
    public $photo;

    public function save()
    {
        $this->photo->store(path: 'photos');
    }
}
```

```blade
<form wire:submit="save">
    <input type="file" wire:model="photo">

    @error('photo') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save photo</button>
</form>
```

> [!warning] "upload" 方法已被保留
> 請注意上面的示例使用了 "save" 方法而不是 "upload" 方法。這是一個常見的 "坑"。術語 "upload" 被 Livewire 保留。您不能在組件中將其用作方法或屬性名稱。

從開發人員的角度來看，處理文件輸入與處理任何其他輸入類型沒有任何不同：將 `wire:model` 添加到 `<input>` 標籤中，其他所有事情都會為您處理。

然而，在 Livewire 中使文件上傳工作時，底層發生了更多事情。以下是當用戶選擇要上傳的文件時發生的情況：

1. 當選擇新文件時，Livewire 的 JavaScript 會向服務器上的組件發出初始請求，以獲取臨時的“簽名”上傳 URL。
2. 一旦收到 URL，JavaScript 會將實際的“上傳”操作執行到簽名 URL，將上傳存儲在 Livewire 指定的臨時目錄中，並返回新臨時文件的唯一哈希 ID。
3. 一旦文件上傳完成並生成了唯一的哈希 ID，Livewire 的 JavaScript 會向服務器上的組件發出最終請求，告訴它將所需的公共屬性“設置”為新的臨時文件。
4. 現在，公共屬性（在這種情況下是 `$photo`）被設置為臨時文件上傳，並準備隨時存儲或驗證。

## 存儲上傳的文件

前面的示例演示了最基本的存儲方案：將臨時上傳的文件移動到應用程序默認文件系統磁碟上的“photos”目錄中。

但是，您可能希望自定義存儲文件的文件名，甚至指定特定的存儲“磁碟”以保存文件（例如 S3）。

> [!tip] 原始檔案名稱
> 您可以透過呼叫其 `->getClientOriginalName()` 方法來存取臨時上傳的原始檔案名稱。

Livewire 遵循 Laravel 用於儲存上傳檔案的相同 API，因此請隨時參考 [Laravel 的檔案上傳文件](https://laravel.com/docs/filesystem#file-uploads)。不過，以下是一些常見的儲存情境和範例：

```php
public function save()
{
    // Store the file in the "photos" directory of the default filesystem disk
    $this->photo->store(path: 'photos');

    // Store the file in the "photos" directory in a configured "s3" disk
    $this->photo->store(path: 'photos', options: 's3');

    // Store the file in the "photos" directory with the filename "avatar.png"
    $this->photo->storeAs(path: 'photos', name: 'avatar');

    // Store the file in the "photos" directory in a configured "s3" disk with the filename "avatar.png"
    $this->photo->storeAs(path: 'photos', name: 'avatar', options: 's3');

    // Store the file in the "photos" directory, with "public" visibility in a configured "s3" disk
    $this->photo->storePublicly(path: 'photos', options: 's3');

    // Store the file in the "photos" directory, with the name "avatar.png", with "public" visibility in a configured "s3" disk
    $this->photo->storePubliclyAs(path: 'photos', name: 'avatar', options: 's3');
}
```

## 處理多個檔案

Livewire 通過檢測 `<input>` 標籤上的 `multiple` 屬性，自動處理多個檔案上傳。

例如，下面是一個具有名為 `$photos` 的陣列屬性的元件。通過將 `multiple` 添加到表單的檔案輸入中，Livewire 將自動將新檔案附加到此陣列中：

```php
use Livewire\Component;
use Livewire\WithFileUploads;
use Livewire\Attributes\Validate;

class UploadPhotos extends Component
{
    use WithFileUploads;

    #[Validate(['photos.*' => 'image|max:1024'])]
    public $photos = [];

    public function save()
    {
        foreach ($this->photos as $photo) {
            $photo->store(path: 'photos');
        }
    }
}
```

```blade
<form wire:submit="save">
    <input type="file" wire:model="photos" multiple>

    @error('photos.*') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save photo</button>
</form>
```

## 檔案驗證

正如我們所討論的，使用 Livewire 驗證檔案上傳與處理標準 Laravel 控制器中的檔案上傳相同。

> [!warning] 確保 S3 正確配置
> 與檔案相關的許多驗證規則需要存取該檔案。當[直接上傳到 S3](#uploading-directly-to-amazon-s3) 時，如果 S3 檔案物件不是公開訪問的，這些驗證規則將失敗。

有關檔案驗證的更多資訊，請參考 [Laravel 的檔案驗證文件](https://laravel.com/docs/validation#available-validation-rules)。

## 臨時預覽 URL

在使用者選擇檔案後，通常應在提交表單並儲存檔案之前向他們顯示該檔案的預覽。

Livewire 通過在上傳的檔案上使用 `->temporaryUrl()` 方法來實現這一點。

> [!info] 臨時 URL 限制於圖片
> 出於安全原因，臨時預覽 URL 僅支援具有圖片 MIME 類型的檔案。

讓我們探索一個帶有圖片預覽的檔案上傳範例：

```php
use Livewire\Component;
use Livewire\WithFileUploads;
use Livewire\Attributes\Validate;

class UploadPhoto extends Component
{
    use WithFileUploads;

    #[Validate('image|max:1024')]
    public $photo;

    // ...
}
```

```blade
<form wire:submit="save">
    @if ($photo) <!-- [tl! highlight:2] -->
        <img src="{{ $photo->temporaryUrl() }}">
    @endif

    <input type="file" wire:model="photo">

    @error('photo') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save photo</button>
</form>
```

正如先前討論的，Livewire 將臨時檔案存儲在非公開目錄中；因此，通常沒有簡單的方法向您的使用者公開臨時的公共 URL 以供圖片預覽。

然而，Livewire 通過提供一個臨時的簽名 URL 來解決這個問題，該 URL 假裝是上傳的圖片，因此您的頁面可以向用戶顯示圖片預覽。

這個 URL 受保護，防止顯示臨時目錄上方的文件。而且，由於它是簽名的，用戶無法濫用此 URL 來預覽系統中的其他文件。

> [!tip] S3 臨時簽名 URL
> 如果您已配置 Livewire 使用 S3 進行臨時文件存儲，調用 `->temporaryUrl()` 將直接生成一個臨時的簽名 URL 到 S3，以便圖片預覽不是從您的 Laravel 應用伺服器加載。

## 測試文件上傳

您可以使用 Laravel 現有的文件上傳測試輔助工具來測試文件上傳。

以下是使用 Livewire 測試 `UploadPhoto` 元件的完整示例：

```php
<?php

namespace Tests\Feature\Livewire;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use App\Livewire\UploadPhoto;
use Livewire\Livewire;
use Tests\TestCase;

class UploadPhotoTest extends TestCase
{
    public function test_can_upload_photo()
    {
        Storage::fake('avatars');

        $file = UploadedFile::fake()->image('avatar.png');

        Livewire::test(UploadPhoto::class)
            ->set('photo', $file)
            ->call('upload', 'uploaded-avatar.png');

        Storage::disk('avatars')->assertExists('uploaded-avatar.png');
    }
}
```

以下是使前面的測試通過所需的 `UploadPhoto` 元件示例：

```php
use Livewire\Component;
use Livewire\WithFileUploads;

class UploadPhoto extends Component
{
    use WithFileUploads;

    public $photo;

    public function upload($name)
    {
        $this->photo->storeAs('/', $name, disk: 'avatars');
    }

    // ...
}
```

有關測試文件上傳的更多信息，請參考 [Laravel 的文件上傳測試文件](https://laravel.com/docs/http-tests#testing-file-uploads)。

## 直接上傳到 Amazon S3

如前所述，Livewire 將所有文件上傳存儲在臨時目錄中，直到開發人員永久存儲文件。

默認情況下，Livewire 使用默認的文件系統磁碟配置（通常是 `local`）並將文件存儲在 `livewire-tmp/` 目錄中。

因此，即使您稍後選擇將上傳的文件存儲在 S3 存儲桶中，文件上傳始終在使用您的應用伺服器。

如果您希望繞過應用伺服器，而是將 Livewire 的臨時上傳存儲在 S3 存儲桶中，您可以在應用程式的 `config/livewire.php` 配置文件中配置該行為。首先，將 `livewire.temporary_file_upload.disk` 設置為 `s3`（或使用 `s3` 驅動程序的其他自定義磁碟）：

```php
return [
    // ...
    'temporary_file_upload' => [
        'disk' => 's3',
        // ...
    ],
];
```

現在，當用戶上傳文件時，該文件將永遠不會實際存儲在您的伺服器上。相反，它將直接上傳到您的 S3 存儲桶中的 `livewire-tmp/` 子目錄中。

> [!info] 發佈 Livewire 的組態檔案
> 在自訂檔案上傳磁碟之前，您必須先將 Livewire 的組態檔案發佈到您應用程式的 `/config` 目錄中，方法是執行以下命令：
> ```shell
> php artisan livewire:publish --config
> ```

### Configuring automatic file cleanup

Livewire's temporary upload directory will fill up with files quickly; therefore, it's essential to configure S3 to clean up files older than 24 hours.

To configure this behavior, run the following Artisan command from the environment that is utilizing an S3 bucket for file uploads:

```shell
php artisan livewire:configure-s3-upload-cleanup

Now, any temporary files older than 24 hours will be cleaned up by S3 automatically.

> [!info]
> If you are not using S3 for file storage, Livewire will handle file cleanup automatically and there is no need to run the command above.

## Loading indicators

Although `wire:model` for file uploads works differently than other `wire:model` input types under the hood, the interface for showing loading indicators remains the same.

You can display a loading indicator scoped to the file upload like so:

```blade
<input type="file" wire:model="photo">

<div wire:loading wire:target="photo">上傳中...</div>

Now, while the file is uploading, the "Uploading..." message will be shown and then hidden when the upload is finished.

For more information on loading states, check out our comprehensive [loading state documentation](/docs/wire-loading).

## Progress indicators

Every Livewire file upload operation dispatches JavaScript events on the corresponding `<input>` element, allowing custom JavaScript to intercept the events:

Event | Description
--- | ---
`livewire-upload-start` | Dispatched when the upload starts
`livewire-upload-finish` | Dispatched if the upload is successfully finished
`livewire-upload-cancel` | Dispatched if the upload was cancelled prematurely
`livewire-upload-error` | Dispatched if the upload fails
`livewire-upload-progress` | An event containing the upload progress percentage as the upload progresses

Below is an example of wrapping a Livewire file upload in an Alpine component to display an upload progress bar:

```blade
<form wire:submit="save">
    <div
        x-data="{ uploading: false, progress: 0 }"
        x-on:livewire-upload-start="uploading = true"
        x-on:livewire-upload-finish="uploading = false"
        x-on:livewire-upload-cancel="uploading = false"
        x-on:livewire-upload-error="uploading = false"
        x-on:livewire-upload-progress="progress = $event.detail.progress"
    >
        <!-- 檔案輸入 -->
        <input type="file" wire:model="photo">

        <!-- 進度條 -->
        <div x-show="uploading">
            <progress max="100" x-bind:value="progress"></progress>
        </div>
    </div>

    <!-- ... -->
</form>

## Cancelling an upload

If an upload is taking a long time, a user may want to cancel it. You can provide this functionality with Livewire's `$cancelUpload()` function in JavaScript.

Here's an example of creating a "Cancel Upload" button in a Livewire component using `wire:click` to handle the click event:

```blade
<form wire:submit="save">
    <!-- 檔案輸入 -->
    <input type="file" wire:model="photo">

    <!-- 取消上傳按鈕 -->
    <button type="button" wire:click="$cancelUpload('photo')">取消上傳</button>

    <!-- ... -->
</form>

When "Cancel upload" is pressed, the file upload will request will be aborted and the file input will be cleared. The user can now attempt another upload with a different file.

Alternatively, you can call `cancelUpload(...)` from Alpine like so:

```blade
<button type="button" x-on:click="$wire.cancelUpload('photo')">取消上傳</button>

## JavaScript upload API

Integrating with third-party file-uploading libraries often requires more control than a simple `<input type="file" wire:model="...">` element.

For these scenarios, Livewire exposes dedicated JavaScript functions.

These functions exist on a JavaScript component object, which can be accessed using Livewire's convenient `$wire` object from within your Livewire component's template:

```blade
@script
<script>
    let file = $wire.el.querySelector('input[type="file"]').files[0]

    // 上傳檔案...
    $wire.upload('photo', file, (uploadedFilename) => {
        // 成功回呼...
    }, () => {
        // 錯誤回呼...
    }, (event) => {
        // 進度回呼...
        // event.detail.progress 包含一個介於 1 到 100 之間的數字，代表上傳進度
    }, () => {
        // 取消回呼...
    })

    // 上傳多個檔案...
    $wire.uploadMultiple('photos', [file], successCallback, errorCallback, progressCallback, cancelledCallback)

```php
// 從多個上傳的文件中刪除單個文件...
$wire.removeUpload('photos', uploadedFilename, successCallback)

// 取消上傳...
$wire.cancelUpload('photos')
</script>
@endscript

## Configuration

Because Livewire stores all file uploads temporarily before the developer can validate or store them, it assumes some default handling behavior for all file uploads.

### Global validation

By default, Livewire will validate all temporary file uploads with the following rules: `file|max:12288` (Must be a file less than 12MB).

If you wish to customize these rules, you can do so inside your application's `config/livewire.php` file:

```php
'temporary_file_upload' => [
    // ...
    'rules' => 'file|mimes:png,jpg,pdf|max:102400', // (最大100MB，僅接受PNG、JPEG和PDF格式)
],

### Global middleware

The temporary file upload endpoint is assigned a throttling middleware by default. You can customize exactly what middleware this endpoint uses via the following configuration option:

```php
'temporary_file_upload' => [
    // ...
    'middleware' => 'throttle:5,1', // 每位用戶每分鐘僅允許上傳5次
],

### Temporary upload directory

Temporary files are uploaded to the specified disk's `livewire-tmp/` directory. You can customize this directory via the following configuration option:

```php
'temporary_file_upload' => [
    // ...
    'directory' => 'tmp',
],
```
