應該放棄使用"hydrate"和"dehydrate"這兩個術語嗎？"Hydrate"通常用於將前端行為從凍結狀態恢復。嗯，我們只是將其應用於後端。"Serialize"和"unserialize"是替代方案，也有"sleep"和"wakeup"。

* 更多範例
* 最佳實踐
	* [https://github.com/michael-rubel/livewire-best-practices](https://github.com/michael-rubel/livewire-best-practices)
* 性能最佳實踐
* 安全風險
	* [https://forum.archte.ch/livewire/t/advanced-livewire-a-better-way-of-working-with-models](https://forum.archte.ch/livewire/t/advanced-livewire-a-better-way-of-working-with-models)
* 更易讀的字體
* "Livewire工作原理"
* Laravel訓練營風格教程
* 小型示例應用
* 

## 大綱

快速入門
* 安裝 Livewire
* 創建您的第一個元件（創建文章，而不是計數器）
* 添加屬性
* 添加行為
* 在瀏覽器中呈現元件
* 測試

升級指南

基礎知識：
* 安裝
	* Composer命令
	* 發布配置
	* 禁用資產自動注入
	* 配置Livewire的更新端點
	* 配置Livewire的JavaScript端點
	* 發布和托管Livewire的JavaScript
* 元件
	* 創建元件
	* 渲染方法
		* 返回Blade視圖
		* 返回模板字符串
	       * artisan make --inline
	- 渲染單個元件
		- 傳遞參數
		- 接收參數
	- 渲染元件路由
		* 配置佈局
		* 路由參數
		* 路由模型綁定
* 屬性
	* 簡介
	* 在mount方法中初始化屬性
	* 批量分配屬性（$this->fill()）
	* 重置屬性（$this->reset()）
	* 數據綁定（基本介紹，連結到其他文件頁面）
	* 支持的屬性類型
		* （簡要解釋hydration/dehydration以及為什麼不支持每種可能的類型）
		* 基本類型（字符串，int，布爾值等...）
		* 常見的PHP類型（Collection，DateTime等...）
		* 支持自定義類型（解釋用戶如何為特定應用程序添加類型支持）
			* 使用Wireables
			* 使用Synthesizers
	* 使用$wire 
	    * 在您的元件內部使用Alpine的$wire訪問屬性
	    * 操作屬性
	    * 使用$wire.get和$wire.set
	* 安全問題
		* 不要信任屬性
			* 授權屬性
			* 使用"locked"屬性
		* 請注意，Livewire會公開屬性元數據，如Eloquent模型類名
	* "Computed"屬性（使用->getPostProperty()語法）

* 操作
	* 安全性考量
	* 參數
	* 事件修飾符
		* 按鍵修飾符
	* 魔法操作
	 * 可連線操作
 * 資料綁定
	* 即時綁定
	* 懶惰綁定
	* 延遲綁定
	* 限制綁定
	* 綁定巢狀資料
	* 綁定至 Eloquent 模型
* 嵌套元件
* 事件
	* 基本範例
	* 安全性考量
	* 觸發事件
	* 監聽器
	* 傳遞參數
	* 事件範圍
		* 父級 / 名稱 / 自身
	* JavaScript 監聽器
	* 分派瀏覽器事件
* 生命週期鉤子
	* 類別鉤子
		* 安裝
		* 水合
		* 啟動
		* 脫水
		* 更新
* 測試
	* 基本測試
        * `artisan make: --test`
	* 創建測試
	* 測試存在性
	* 傳遞元件資料
	* 傳遞查詢字串參數
	* 可用指令
	* 可用斷言
* AlpineJS
	* ...
* Eloquent 模型
	* 設定為屬性
	* 效能影響
	* 綁定至屬性
	* 模型集合

表單:
* 表單提交
* 表單輸入
* 輸入驗證
* 檔案上傳

功能:
* 載入狀態
* 分頁
* 內嵌腳本
* 快閃訊息
* 查詢字串
* 重新導向
* 輪詢
* 授權
* 髒狀態
* 檔案下載
* 離線狀態
* 計算屬性

深入了解:
* Livewire 如何運作
* 合成器

JavaScript 全域
	* 生命週期鉤子
元件抽象
Artisan 指令
疑難排解
安全性（內部和使用者端）
擴充
- 自訂可連線
套件開發
部署
發佈存根
Laravel Echo
參考

V3:
* 懶載入
* 單頁應用模式


* 這真的是我想要訂購的地方嗎？
* 媽媽喜歡什麼樣的花？
* 我應該花多少錢買花？
* 什麼時候應該送花？
* 我需要打電話給爸爸確認她會喜歡嗎？


為什麼寫文件這麼困難？

### 詞語很難

是的，但不僅如此。同意詞語很難，但似乎我可以毫不費力地寫一篇部落格文章。

這是為讀者最低共同分母而設的無聊言論

是的，就像我感覺自己完全沒有發言權一樣。就像我平常喜歡寫的笑話、閒聊和大膽陳述在這裡不合適。這裡只能追求清晰度。

### 組織內容很困難
存在著「雞」和「蛋」問題：某些內容依賴於另一部分，但該部分又依賴於原始部分。哪一個應該先來？

存在著細粒度問題：我們要如何拆分這個內容？

存在著排序問題：「我是從簡單到複雜嗎？還是從現實世界到非現實世界？」 

存在著重複問題：在多個頁面中是否要重複自己？還是將一個功能隔離到單個文件中？這個問題已經有答案了：重複自己。

### 代碼示例很難
要包含多少非關鍵上下文？（包括 < ?php 包含類？命名空間？使用？渲染？）

如何使其現實世界化，但又迎合正確的機制？「hello world」和「counter」組件很有幫助，但不是現實生活。但現實生活往往不夠簡單。

你要堅持相同的領域嗎？整個時間都是「CreatePost」？是不是頻繁跳來跳去太刺耳，還是堅持一個例子整個時間太可預測、受限和單調？

### 這樣的內容太多了
這只是太多了，感覺壓倒性

幽默：
* 浴缸
* 愚蠢的橡皮筋
* 馬達加斯加的企鵝
* Ed Bassmaster
* 海綿寶寶
