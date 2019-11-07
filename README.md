# Session Timeout
Create a session inactivity timeout that warns users when their session is about to expire.

# 目的
建立一個連線逾時的 session(會話)，並在使用者的 session 超時後發出警告。

該插件提供 "session 逾時" 和 "閒置、不活動逾時" 的功能來幫助開發者實作。當使用者一進入頁面時，他們的 session 即開始。在指定的時間過後，將通知他們 session 即將逾時。 此時，他們可以選擇點擊 "Continue session" 保持登入的狀態，或透過點擊 "End session now" 登出。如果使用者在特定時間內沒有回應逾時的通知，將會被自動重導至登出頁面。

# 使用情境
* 於 session 即將逾時前提醒使用者，並給他們延長 session 的機會。
* 若使用者仍在網頁上活動(無閒置)時，應延長其 session。

# 範例
[官方範例](https://wet-boew.github.io/v4.0-ci/demos/session-timeout/session-timeout-en.html)
(需於頁面等待 30 秒後，才會看到 session timeout 提醒視窗)

# 如何實作
新增含有 `class="wb-sessto"` 類別的元素至頁面中(只需引用一次)，且必須設定
`logouturl` ，此為 session 逾時後，會自動重導的頁面。
```sh
<span class="wb-sessto" data-wb-sessto='{"logouturl": "http://app.gc.ca/logout"}'></span>
```
使用 `data-wb-sessto` 屬性配置更多選項，請參考以下：
```sh
<span class="wb-sessto" data-wb-sessto='{
	"inactivity": 1200000,
	"reactionTime": 30000,
	"sessionalive": 1200000,
	"logouturl": "./",
	"refreshCallbackUrl": "./"
	"refreshOnClick": true,
	"refreshLimit": 200000,
	"method": "POST",
	"additionalData": null}'></span>
```

# 配置選項
該插件的所有配置選項均由 `data-wb-sessto` 屬性或 `window ["wb-sessto"]` 控制。 


| Option | Description | How to configure | Values |
| ------ | ------ | ------ | ------ |
| `inactivity` | 設定閒置、不活動逾時。 一旦超時，將顯示對話框，提示使用者繼續或結束 session | 數值，時間單位非必須 | 未設定則預設 20 分鐘；單寫數值視為毫秒值；也可寫成數值加上時間單位(e.g. "100 s") |
| `reactionTime` | 設定對話框顯示後，使用者能操作的時間 | 數值，時間單位非必須 | 未設定則預設 3 分鐘；單寫數值視為毫秒值；也可寫成數值加上時間單位(e.g. "100 s") |
| `sessionalive` | 設定在發出 Ajax 請求之前，確定 session 是否仍然存在的時間。必須指定 `refreshCallbackUrl` 才有作用 | 數值，時間單位非必須 | 未設定則預設 20 分鐘；單寫數值視為毫秒值；也可寫成數值加上時間單位(e.g. "100 s") |
| `logouturl` | 設定 session 逾時後，會自動重導的頁面 | URL | 未設定則預設 URL 為 "./" |
| `refreshCallback` | 該函數用於檢查從 `refreshCallbackUrl` 收到的回應。不能在 `data-wb-sessto` 屬性上指定此參數 | 函數 | 未設定則預設該回應完全匹配字串 "true"；該函數傳入一個字串參數，該參數為回應訊息。 如果 session 處於活動狀態，則必須回傳 true，反之 false |
| `refreshCallbackUrl` | 用於執行 Ajax 請求的 URL，以確定 session 的有效性。 有關更多詳細資訊請參見 `method` 和 `refreshCallback` | URL | 未設定則表示沒有發送 Ajax 請求 |
| `refreshOnClick` | 決定點擊對話框外其他地方，是否應重置閒置超時並執 Ajax 請求(如果已指定 `refreshCallbackUrl`) | 布林值(true/false) | 未設定則預設 true |
| `refreshLimit` | 設定發出 Ajax 請求之前必須經過的時間 | 數值，時間單位非必須 | 未設定則預設 2 分鐘；單寫數值視為毫秒值；也可寫成數值加上時間單位(e.g. "100 s") |
| `method` | 設定發送 Ajax 請求的方法。雖然所有方法都可被接受，但回應必須包含訊息內容(message body)，建議使用 GET、POST | 字串 | 未設定則預設 POST |
| `additionalData` | 與請求一起發送的其他資料 | 字串、陣列、物件 或 null | 未設定則表示沒有發送其他資料 |

支援的時間單位：
* `ms`：0.001 秒
* `cs`：0.01 秒
* `ds`：0.1 秒
* `s`：1 秒
* `das`：10 秒
* `hs`：100 秒
* `ks`：1,000 秒

# 事件
| Event | Trigger | What it does |
| ------ | ------ | ------ |
| `wb-init.wb-sessto` | 手動觸發 (e.g. `$( ".wb-sessto" ).trigger( "wb-init.wb-sessto" );`) | 初始化插件並啟動 session 和閒置逾時。注意：除非頁面載入完成後有加上 `.wb-sessto` 元素，否則此插件將自動初始化 |
| `wb-ready.wb-sessto` (v4.0.5+) | 該插件初始化後自動觸發 | 用來確認該插件初始化完成的時間點 |
| `inactivity.wb-sessto` | 手動觸發 (e.g. `$( ".wb-sessto" ).trigger( "inactivity.wb-sessto" );`) | 使對話框出現，並提示使用者是否繼續或延長 session，則此事件會在閒置逾時而自動觸發 |
| `keepalive.wb-sessto` | 手動觸發 (e.g. `$( ".wb-sessto" ).trigger( "keepalive.wb-sessto" );`) | 使發出 Ajax 請求（如果已指定 `refreshCallbackUrl`）。如果 `refreshCallbackUrl` 回應不是 "true"，則會警告使用者其 session 已逾時 |
| `reset.wb-sessto` | 手動觸發 (e.g. `$( ".wb-sessto" ).trigger( "reset.wb-sessto" );`) | 重置 `inactivity` 和 `keepalive` 逾時 |
| `wb-ready.wb` (v4.0.5+) | 當所有 WET(Web Experience Toolkit) 插件都完成載入和執行後自動觸發 | 用來確認所有 WET 插件都完成載入和執行 |

##### Note
當手動觸發 `inactivity`、`keepalive` 和 `reset` 事件時，必須傳入 `data-wb-sessto` 屬性當作第二個參數
```sh
// Get a reference to the session timeout element and its data-wb-sessto attribute
var $element = $( ".wb-sessto" ),
	settings = $element.data( "wb-sessto" );

// Trigger the event on the element
$element.trigger( "reset.wb-sessto", settings );
```

##### Source code
[Session timeout source code on GitHub](https://github.com/wet-boew/wet-boew/tree/master/src/plugins/session-timeout)
