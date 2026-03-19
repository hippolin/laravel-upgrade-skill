# Laravel 升級注意事項

各版本升級時最容易踩坑、最值得特別留意的地方。
完整 breaking changes 請參考 Laravel 官方文件。

---

## v6 → v7（約 15 分鐘）

**日期序列化格式改變（高影響）**
`toArray()` / `toJson()` 輸出的日期格式從 `2019-12-02 20:01:00` 變成 `2019-12-02T20:01:00.283041Z`（ISO-8601 含時區與毫秒）。如果 API 消費者或前端依賴日期格式，這是破壞性變更。可在 Model 覆寫 `serializeDate()` 還原舊格式。

**Exception Handler 改用 `Throwable`（高影響）**
`App\Exceptions\Handler` 的四個方法（`report`、`shouldReport`、`render`、`renderForConsole`）型別提示從 `Exception` 改成 `Throwable`。Queue job 的 `failed()` 方法也一樣。

**Artisan Command 必須回傳整數**
`handle()` 回傳 `true` 要改成回傳 `0`，否則 Symfony Console 會報錯。

**`Blade::component` 更名**
改為 `Blade::aliasComponent`，直接搜尋取代即可。

---

## v7 → v8（約 15 分鐘）

**Model Factory 完全重寫（高影響）**
舊的 closure-based factory 與新版不相容。可安裝 `laravel/legacy-factories` 過渡，暫時不需改任何程式碼。若要遷移新版，Model 需加 `HasFactory` trait，測試裡的 `factory(User::class)->create()` 改成 `User::factory()->create()`。

**Seeder 需要加命名空間（高影響）**
`database/seeds/` 目錄改名為 `database/seeders/`，所有 Seeder 加上 `namespace Database\Seeders;`，`composer.json` 的 autoload 從 `classmap` 改為 `psr-4`，記得跑 `composer dump-autoload`。

**Pagination 預設改為 Tailwind CSS（高影響）**
若使用 Bootstrap，在 `AppServiceProvider::boot()` 加一行 `Paginator::useBootstrap()`。

**Queue 屬性更名（高影響）**
`retryAfter` → `backoff`，`timeoutAt` → `retryUntil`。Job、Mailer、Notification、Listener 都要檢查。

---

## v8 → v9（約 30 分鐘）

> 這是歷次升級中改動最多的一版，建議用 [Laravel Shift](https://laravelshift.com/) 輔助。

**Symfony Mailer 取代 SwiftMailer（高影響）**
這版最大的工程。`withSwiftMessage` → `withSymfonyMessage`，`getSwiftMailer()` → `getSymfonyTransport()`，低階的 `setBody` / `addPart` → `html()` / `text()`。SMTP 設定的 `stream` 巢狀結構也被攤平。有客製化 mailer 邏輯的專案要仔細清查。

**Flysystem 3.x（高影響）**
行為全面改變：`put`/`write` 不再拋例外而是回傳 `false`（需加 `'throw' => true` 設定才會拋）；讀不存在的檔案回傳 `null` 而非拋例外；`cached adapter` 直接移除。自訂 filesystem driver 的回傳型別也改了。

**HTTP Client 新增預設 30 秒 timeout**
原本沒有 timeout，升級後原本會「永遠等待」的請求會在 30 秒後拋例外，注意長時間 API 呼叫。

**`when` / `unless` closure 行為改變（中影響）**
以前傳 closure 給第一個參數永遠執行（因為 closure 物件恆為 truthy），現在會執行 closure 並用其回傳值判斷。全面搜尋 `->when(function` 和 `->unless(function` 確認邏輯是否正確。

**`password` 驗證規則更名**
`'password'` 改為 `'current_password'`。

---

## v9 → v10（約 10 分鐘）

**`$dates` 屬性移除（中影響）**
Eloquent Model 上的 `$dates` 已正式移除，全面改用 `$casts`。搜尋 `protected $dates` 逐一替換。

**`DB::raw` 不能再 cast 成字串**
`(string) DB::raw(...)` 要改成 `$expr->getValue(DB::connection()->getQueryGrammar())`。主要影響有直接操作 raw expression 的程式碼或套件。

**`MocksApplicationServices` 測試 trait 移除**
`expectsEvents` / `expectsJobs` / `expectsNotifications` 要換成 `Event::fake()` / `Bus::fake()` / `Notification::fake()`。

---

## v10 → v11（約 15 分鐘）

**`->change()` 必須重新宣告所有欄位屬性（高影響）**
這版最容易踩的坑。以前 `->change()` 會靜默保留欄位原有屬性（`unsigned`、`default`、`comment` 等），現在沒有明確寫出的屬性會直接被刪掉。全面搜尋 `->change()` 逐一補齊屬性，或先跑 `php artisan schema:dump` 壓縮既有 migrations 省事。

**Sanctum / Passport / Cashier / Telescope 不再自動載入 migration**
升級後必須手動 `php artisan vendor:publish --tag=sanctum-migrations`（以及其他用到的套件），否則新功能的 migration 不會跑。

**Carbon 3 的 `diffIn*` 回傳值改變（中影響）**
從永遠正整數變成可能為負數的浮點數（負數表示方向）。全面審查所有日期差值比較邏輯，需要正數的地方加 `abs()`。

**浮點欄位型別語法改變**
`$table->double('col', 15, 2)` 的 `$total` / `$places` 參數移除，直接用 `$table->double('col')`。`unsignedDouble` / `unsignedFloat` 等方法也移除，改鏈式呼叫 `->unsigned()`。

---

## v11 → v12（約 5 分鐘）

**`HasUuids` 改用 UUIDv7（中影響）**
唯一需要主動決策的地方。原本是 UUIDv4，現在換成時間排序的 UUIDv7。如果 API、前端、或測試有依賴 UUID 格式（例如排序邏輯、格式驗證），要評估是否需要換回 `HasVersion4Uuids`。

**`Storage::disk('local')` 預設根目錄改變**
從 `storage/app` 改為 `storage/app/private`。只影響沒有在 `config/filesystems.php` 明確設定 `local` disk 的專案。

**`image` 驗證規則不再接受 SVG**
有檔案上傳用 `image` rule 的地方，確認是否需要加 `allow_svg` 參數。

---

## v12 → v13（約 10 分鐘）⚠️ Pre-release，文件可能變動

**CSRF Middleware 更名並加強安全性（高影響）**
`VerifyCsrfToken` 更名為 `PreventRequestForgery`，同時新增 `Sec-Fetch-Site` header 的 origin 驗證。測試裡常見的 `->withoutMiddleware([VerifyCsrfToken::class])` 要一起更新，否則測試可能靜默失效。

**Cache `serializable_classes` 安全加固（中影響）**
預設改為不允許反序列化 PHP 物件，防止 `APP_KEY` 外洩時的 deserialization 攻擊。如果有把 Eloquent Model 或其他物件存進 cache（例如 `Cache::put('key', $model)`），升級後會壞掉，需要在 `config/cache.php` 的 `serializable_classes` 明確列出允許的 class。

**`JobAttempted` 事件屬性改變**
`$event->exceptionOccurred`（bool）→ `$event->exception`（Throwable|null）。檢查所有監聽此事件的 listener。

**`QueueBusy` 事件屬性改名**
`$event->connection` → `$event->connectionName`。
