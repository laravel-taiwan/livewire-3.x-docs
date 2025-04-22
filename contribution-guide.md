嗨，歡迎來到 Livewire 貢獻指南。在這個指南中，我們將看看您如何通過提交新功能、修復失敗的測試或解決問題來貢獻 Livewire。

## 在本地設置 Livewire 和 Alpine
要進行貢獻，最簡單的方法是確保 Livewire 和 Alpine 存儲庫在您的本地機器上設置。這將使您能夠輕鬆進行更改並運行測試套件。

### 複製和克隆存儲庫
要開始，第一步是複製和克隆存儲庫。最簡單的方法是使用 [GitHub CLI](https://cli.github.com/) 進行此操作，但您也可以通過在 GitHub [存儲庫頁面](https://github.com/livewire/livewire) 上點擊 "Fork" 按鈕來手動執行這些步驟。

```shell
# Fork and clone Livewire
gh repo fork livewire/livewire --default-branch-only --clone=true --remote=false -- livewire

# Switch the working directory to livewire
cd livewire

# Install all composer dependencies
composer install

# Ensure Dusk is correctly configured
vendor/bin/dusk-updater detect --no-interaction
```

要設置 Alpine，請確保已安裝 [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)，然後運行以下命令。如果您更喜歡手動複製，可以訪問 [存儲庫頁面](https://github.com/alpinejs/alpine)。

```shell
# Fork and clone Alpine
gh repo fork alpinejs/alpine --default-branch-only --clone=true --remote=false -- alpine

# Switch the working directory to alpine
cd alpine

# Install all npm dependencies
npm install

# Build all Alpine packages
npm run build

# Link all Alpine packages locally
cd packages/alpinejs && npm link && cd ../../
cd packages/anchor && npm link && cd ../../
cd packages/collapse && npm link && cd ../../
cd packages/csp && npm link && cd ../../
cd packages/docs && npm link && cd ../../
cd packages/focus && npm link && cd ../../
cd packages/history && npm link && cd ../../
cd packages/intersect && npm link && cd ../../
cd packages/mask && npm link && cd ../../
cd packages/morph && npm link && cd ../../
cd packages/navigate && npm link && cd ../../
cd packages/persist && npm link && cd ../../
cd packages/sort && npm link && cd ../../
cd packages/ui && npm link && cd ../../

# Switch the working directory back to livewire
cd ../livewire

# Link all packages
npm link alpinejs @alpinejs/anchor @alpinejs/collapse @alpinejs/csp @alpinejs/docs @alpinejs/focus @alpinejs/history @alpinejs/intersect @alpinejs/mask @alpinejs/morph @alpinejs/navigate @alpinejs/persist @alpinejs/sort @alpinejs/ui

# Build Livewire
npm run build
```

## 貢獻一個失敗的測試

如果您遇到一個 bug，並且不確定如何解決它，特別是考慮到 Livewire 核心的複雜性，您可能會想知道從哪裡開始。在這種情況下，最簡單的方法是貢獻一個失敗的測試。這樣，具有更多經驗的人可以幫助識別並修復該 bug。儘管如此，我們建議您也探索核心，以更好地了解 Livewire 的運作方式。

讓我們逐步進行。

#### 1. 確定在哪裡添加您的測試
Livewire 核心分為不同的文件夾，每個文件夾對應特定的 Livewire 功能。例如：

```shell
src/Features/SupportAccessingParent
src/Features/SupportAttributes
src/Features/SupportAutoInjectedAssets
src/Features/SupportBladeAttributes
src/Features/SupportChecksumErrorDebugging
src/Features/SupportComputed
src/Features/SupportConsoleCommands
src/Features/SupportDataBinding
//...
```

嘗試找到與您遇到的 bug 相關的功能。如果找不到適當的文件夾，或者不確定應選擇哪個文件夾，您可以簡單地選擇一個並在拉取請求中提到您需要幫助將測試放在正確的功能集中。

#### 2. 確定測試類型
Livewire 測試套件包含兩種類型的測試：

1. **單元測試**：這些測試專注於 Livewire 的 PHP 實作。
2. **瀏覽器測試**：這些測試在真實瀏覽器中運行一系列步驟並斷言正確結果。它們主要專注於 Livewire 的 Javascript 實作。

如果您不確定要選擇哪種測試類型，或者對於為 Livewire 撰寫測試感到陌生，您可以從瀏覽器測試開始。實現您在應用程式和瀏覽器中執行以重現錯誤的步驟。

單元測試應該添加到 `UnitTest.php` 檔案中，而瀏覽器測試應該添加到 `BrowserTest.php`。如果這兩個檔案中的一個或兩個不存在，您可以自行建立它們。

**單元測試**

```php
use Tests\TestCase;

class UnitTest extends TestCase
{
    public function test_livewire_can_run_action(): void
    {
       // ...
    }
}
```

**瀏覽器測試**

```php
use Tests\BrowserTestCase;

class BrowserTest extends BrowserTestCase
{
    public function test_livewire_can_run_action()
    {
        // ...
    }
}
```

> [!tip] 不確定如何撰寫測試？
> 通過探索現有的單元測試和瀏覽器測試，您可以學習如何撰寫測試。即使只是複製和貼上現有的測試也是撰寫自己測試的絕佳起點。

#### 3. 準備您的拉取請求分支
當您完成功能或失敗測試後，就該提交您的拉取請求（PR）到 Livewire 存儲庫。首先，確保將更改提交到一個獨立的分支（避免使用 `main`）。要創建一個新分支，您可以使用 `git` 命令：

```shell
git checkout -b my-feature
```

您可以將分支命名為任何您想要的名稱，但為了將來參考，最好使用反映您功能或失敗測試的描述性名稱。

接下來，將更改提交到您的分支。您可以使用 `git add .` 將所有更改加入暫存區，然後使用 `git commit -m "Add my feature"` 提交所有更改並附上描述性的提交訊息。

但是，您的分支目前僅在本機機器上可用。要創建拉取請求，您需要將分支推送到您的派生 Livewire 存儲庫使用 `git push`。

```shell
git push origin my-feature

Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.

To github.com:Username/livewire.git
 * [new branch]        my-feature -> my-feature
```

#### 4. 提交您的拉取請求
我們快要完成了！打開您的網頁瀏覽器，並導航到您的派生 Livewire 存儲庫（`https://github.com/<your-username>/livewire`）。在螢幕中央，您將看到一條新通知："**my-feature had recent pushes 1 minute ago**" 以及一個按鈕，上面寫著 "**Compare & pull request**"。點擊該按鈕以打開拉取請求表單。

在表單中，提供一個描述您拉取請求的標題，然後繼續到描述部分。文本區域已包含預定義模板。嘗試回答每個問題：

```
Review the contribution guide first at: https://livewire.laravel.com/docs/contribution-guide

1️⃣ Is this something that is wanted/needed? Did you create a discussion about it first?
Yes, you can find the discussion here: https://github.com/livewire/livewire/discussions/999999

2️⃣ Did you create a branch for your fix/feature? (Main branch PR's will be closed)
Yes, the branch is named `my-feature`

3️⃣ Does it contain multiple, unrelated changes? Please separate the PRs out.
No, the changes are only related to my feature.

4️⃣ Does it include tests? (Required)
Yes

5️⃣ Please include a thorough description (including small code snippets if possible) of the improvement and reasons why it's useful.

These changes will improve memory usage. You can see the benchmark results here:

// ...

```

一切就緒了嗎？點擊 **建立拉取請求** 🚀 恭喜！您已成功創建了您的第一個貢獻 🎉

維護者將審查您的 PR，可能會提供反饋或要求更改。請努力盡快解決任何反饋。

感謝您為 Livewire 做出貢獻！
