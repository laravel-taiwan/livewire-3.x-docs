* 元件
    * 計數器元件
* 渲染元件
    * 掛載
        * 新建類別
        * 解構狀態
        * 嵌入 HTML 內
        * 返回 HTML
* 在 JS 中初始化元件
    * 尋找 wire:id 元素
    * 提取 id 和快照
    * 新建物件
* 發送更新
    * 註冊事件監聽器
    * 發送帶有更新和快照的取得請求
* 接收更新
    * 將快照轉換為元件 (解構)
    * 應用更新
    * 渲染元件
    * 返回 HTML 和新快照
* 處理更新
    * 用新快照取代
    * 用新 HTML 取代 HTML
        * 變形

## 元件

```php
<?php

use Livewire\Component;

class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

```blade
<div>
    <button wire:click="increment">Increment</button>

    <span>{{ $count }}</span>
</div>
```

## 渲染元件

```blade
<livewire:counter />
```

```php
<?php echo Livewire::mount('counter'); ?>
```

```php
public function mount($name)
{
    $class = Livewire::getComponentClassByName();

    $component = new $class;

    $id = str()->random(20);

    $component->setId($id);

    $data = $component->getData();

    $view = $component->render();

    $html = $view->render($data);

    $snapshot = [
        'data' => $data,
        'memo' => [
            'id' => $component->getId(),
            'name' => $component->getName(),
        ]
    ];

    return Livewire::embedSnapshotInsideHtml($html, $snapshot);
}
```

```blade
<div wire:id="123456789" wire:snapshot="{ data: { count: 0 }, memo: { 'id': '123456789', 'name': 'counter' }">
    <button wire:click="increment">Increment</button>

    <span>1</span>
</div>
```

## JavaScript 初始化

```js
let el = document.querySelector('wire\\:id')

let id = el.getAttribute('wire:id')
let jsonSnapshot = el.getAttribute('wire:snapshot')
let snapshot = JSON.parse(jsonSnapshot)

let component = { id, snapshot }

walk(el, el => {
    el.hasAttribute('wire:click') {
        let action = el.getAttribute('wire:click')

        el.addEventListener('click', e => {
            updateComponent(el, component, action)
        })
    }
})

function updateComponent(el, component, action) {
    let response fetch('/livewire/update', {
        body: JSON.stringify({
            "snapshot": snapshot,
            "calls": [
                ["method": action, "params": []],
            ]
        })
    })

    // To be continued...
}
```

## 接收更新

```php
Route::post('/livewire/update', function () {
    $snapshot = request('snapshot');
    $calls = request('calls');

    $component = Livewire::fromSnapshot($snapshot);

    foreach ($calls as $call) {
        $component->{$call['method']}(...$call['params']);
    }

    [$html, $snapshot] = Livewire::snapshot($component);

    return [
        'snapshot' => $snapshot,
        'html' => $html,
    ];
});
```

## 處理更新

```js
function updateComponent(el, component, action) {
    fetch('/livewire/update', {
        body: JSON.stringify({
            "snapshot": snapshot,
            "calls": [
                ["method": action, "params": []],
            ]
        })
    }).then(i => i.json()).then(response => {
        let { html, snapshot } = response

        component.snapshot = snapshot

        el.outerHTML = html
    })
}
```
