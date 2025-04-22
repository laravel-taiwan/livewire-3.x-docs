Livewire 允許您透過 `wire:stream` API 在請求完成之前將內容流式傳送到網頁。這對於像是 AI 聊天機器人這樣在生成時即時流式回應的功能非常有用。

> [!warning] 不兼容 Laravel Octane
> Livewire 目前不支援在 Laravel Octane 中使用 `wire:stream`。

為了展示 `wire:stream` 的最基本功能，以下是一個簡單的倒數計時組件，當按下按鈕時，會向使用者顯示從 "3" 到 "0" 的倒數：

```php
use Livewire\Component;

class CountDown extends Component
{
    public $start = 3;

    public function begin()
    {
        while ($this->start >= 0) {
            // Stream the current count to the browser...
            $this->stream(  // [tl! highlight:4]
                to: 'count',
                content: $this->start,
                replace: true,
            );

            // Pause for 1 second between numbers...
            sleep(1);

            // Decrement the counter...
            $this->start = $this->start - 1;
        };
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            <button wire:click="begin">Start count-down</button>

            <h1>Count: <span wire:stream="count">{{ $start }}</span></h1> <!-- [tl! highlight] -->
        </div>
        HTML;
    }
}
```

當使用者按下 "開始倒數" 按鈕時，以下是從使用者角度看到的情況：
* 頁面上顯示 "Count: 3"
* 他們按下 "開始倒數" 按鈕
* 一秒過去，顯示 "Count: 2"
* 這個過程持續進行，直到顯示 "Count: 0"

在所有這些過程中，只有一個網路請求發送到伺服器。

當按下按鈕時，系統的運作如下：
* 發送請求給 Livewire 以調用 `begin()` 方法
* 調用 `begin()` 方法並開始 `while` 迴圈
* 呼叫 `$this->stream()` 並立即開始向瀏覽器發送 "流式回應"
* 瀏覽器接收到一個帶有指示的流式回應，指示瀏覽器在組件中尋找具有 `wire:stream="count"` 的元素，並用接收到的有效負載（在第一個流式數字的情況下為 "3"）替換其內容
* `sleep(1)` 方法導致伺服器暫停一秒
* 重複 `while` 迴圈，並持續每秒流式傳送一個新數字的過程，直到 `while` 條件為 false
* 當 `begin()` 完成運行並所有計數已流式傳送到瀏覽器時，Livewire 完成其請求生命週期，呈現組件並將最終回應發送到瀏覽器

## 流式傳送聊天機器人回應

`wire:stream` 的常見用例是在從支援流式回應的 API（例如 [OpenAI's ChatGPT](https://chat.openai.com/)）接收到時即時流式傳送聊天機器人回應。

以下是使用 `wire:stream` 實現類似 ChatGPT 介面的示例：

```php
use Livewire\Component;

class ChatBot extends Component
{
    public $prompt = '';

    public $question = '';

    public $answer = '';

    function submitPrompt()
    {
        $this->question = $this->prompt;

        $this->prompt = '';

        $this->js('$wire.ask()');
    }

    function ask()
    {
        $this->answer = OpenAI::ask($this->question, function ($partial) {
            $this->stream(to: 'answer', content: $partial); // [tl! highlight]
        });
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            <section>
                <div>ChatBot</div>

                @if ($question)
                    <article>
                        <hgroup>
                            <h3>User</h3>
                            <p>{{ $question }}</p>
                        </hgroup>

                        <hgroup>
                            <h3>ChatBot</h3>
                            <p wire:stream="answer">{{ $answer }}</p> <!-- [tl! highlight] -->
                        </hgroup>
                    </article>
                @endif
            </section>

            <form wire:submit="submitPrompt">
                <input wire:model="prompt" type="text" placeholder="Send a message" autofocus>
            </form>
        </div>
        HTML;
    }
}
```

上面示例中的操作如下：
* 用戶在標記為“發送消息”的文本字段中輸入問題以向聊天機器人提問。
* 他們按下 [Enter] 鍵。
* 發送網絡請求到服務器，將消息設置為 `$question` 屬性，並清除 `$prompt` 屬性。
* 將響應發送回瀏覽器並清除輸入。因為調用了 `$this->js('...')`，將觸發新的請求到服務器調用 `ask()` 方法。
* `ask()` 方法調用 ChatBot API 並通過回調中的 `$partial` 參數接收流式響應片段。
* 每個 `$partial` 都流式傳送到頁面上的 `wire:stream="answer"` 元素中，逐步向用戶展示答案。
* 當接收到整個響應時，Livewire 請求完成，用戶接收完整的響應。

## 替換 vs. 附加

在使用 `$this->stream()` 將內容流式傳送到元素時，您可以告訴 Livewire 是替換目標元素的內容還是將其附加到現有內容中。

根據情況，替換或附加都可能是理想的。例如，從聊天機器人流式傳送響應時，通常希望附加（因此是默認值）。但是，當顯示倒計時等內容時，替換更適合。

您可以通過將 `replace:` 參數傳遞給 `$this->stream` 並設置布爾值來配置替換或附加：

```php
// Append contents...
$this->stream(to: 'target', content: '...');

// Replace contents...
$this->stream(to: 'target', content: '...', replace: true);
```

附加/替換也可以在目標元素級別指定，方法是添加或刪除 `.replace` 修飾符：

```blade
// Append contents...
<div wire:stream="target">

// Replace contents...
<div wire:stream.replace="target">
```
