<img width="1500" height="2000" alt="image (8)" src="https://github.com/user-attachments/assets/bd9769a4-d297-4de1-99b9-1df871c2e554" />
<img width="1500" height="2000" alt="image (9)" src="https://github.com/user-attachments/assets/01ee8ce5-4f1b-4b8d-a274-b08e321339b5" />
<img width="1500" height="2000" alt="image (10)" src="https://github.com/user-attachments/assets/b400a029-4af3-461e-b820-692ef44954e5" />


These nine cards are a ServiceNow server-script cheat sheet. Here is a table of contents, then a section index you can scan.

## Table of contents

| # | Card | One-line job | Memory hook |
|---|---|---|---|
| 1 | GlideAggregate | Counts and grouped totals. Extends GlideRecord. Do not use `getRowCount` for this. | `addAggregate`, `query`, `next`, `getAggregate`. Group field is on the row. Count is a string. |
| 2 | GlideQuery | Fluent query. One terminal ends the chain. Do not mix Optional and Stream. | `selectOne` / `get` / `avg` → Optional → `orElse`. `select` → Stream → `forEach`. `count()` is a number. |
| 3 | GlideDateTime | Instants. Store UTC, display local. GlideDate is the date-only sibling. | `getValue` to store, `getDisplayValue` to show, `addDaysUTC` to shift, `subtract` for a duration. |
| 4 | GlideAjax | Client calls a client-callable Script Include. Async. Return a string. | `addParam` `sysparm_name`, `getXMLAnswer`, `AbstractAjaxProcessor`, return a string. |
| 5 | gs · GlideSystem | Server global. Logging, user, dates, properties, events. | `info` for the log, `addErrorMessage` for the human, `getUserID` for the sys_id, `nil` for empty. |
| 6 | GlideUser | The user object. Get it from `gs.getUser()`. Client twin is `g_user`. | `gs.getUserID` for the stamp, `hasRole` for the gate, `isMemberOf` for the group. `g_user` is display-only. |
| 7 | RESTMessageV2 | Outbound HTTP. A REST Message record, or an ad-hoc call. `sn_ws` scope. | `RESTMessageV2`, `execute`, `getStatusCode`, `getBody`. 2xx is success. `haveError` is only the transport. |
| 8 | GlideQuery · typical incident work | Reuse one base query. `count()` is a number. `groupBy` + `aggregate()` returns a Stream of group objects. | Which call for which job (count, list, latest, group, having, bulk). |
| 9 | GlideQuery · three paths | Start here. One terminal ends the chain. Do not mix Optional with Stream. | `selectOne` / `get` / `sum` / `avg` → Optional. `select` → Stream. `find()` is the only door back to Optional. |

## Section index

**1. GlideAggregate**
- Count, no groups — `setGroup(false)` if you only want the grand total
- Group by, several aggregates — `groupBy`, `COUNT` / `AVG` / `MAX`, `orderByAggregate`, `addHaving`
- Aggregate names — `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `STDDEV`, `COUNT(DISTINCT)`, `addTrend`
- vs GlideQuery — Aggregate returns a string; GlideQuery `count()` is already a number
- Traps — parse the string; empty `AVG` is null not 0; `addHaving` compares the aggregate; still call `query()` and `next()`; do not mix `setLimit` with groups

**2. GlideQuery**
- One record · Optional — `selectOne` / `get` / `orElse` / `isPresent` / `ifPresent`. No `orElseThrow`; `get()` throws.
- Many · Stream — `limit` before `select`, then `forEach`. `sys_id` always returned.
- Numbers — `count()` is a number; `avg` / `sum` / `min` / `max` are Optional. Shape: `row.group.*`, `row.count`, `row.avg.*`
- Also returns Optional — `get`, `getBy`, `insert`, `update`, `updateMultiple`, `whereNotNull` / `whereNull` / `orWhere`, `IN`
- Do not mix — `select().orElse` and `selectOne().forEach` are broken. `find()` bridges Stream to Optional. Flags: `$DISPLAY` vs `$SYSID`. Immutable; reuse the base query.
- When to pick it — new server code and fail-fast field checks. Stay on GlideRecord for encoded queries and joins (`addJoinQuery` / `addEncodedQuery`).

**3. GlideDateTime**
- Now, parse, format — `getValue` (UTC), `getDisplayValue` (user TZ), `getNumericValue` (ms). Empty constructor is now. GlideDate is date-only.
- Move it — `addDaysUTC`, `addDaysLocalTime`, `addSeconds`, `addMonthsUTC`, `addWeeksLocalTime`. UTC methods do not shift with DST.
- Compare — `before` / `after` / `compareTo` / `equals`. `subtract` returns a GlideDuration.
- Last 30 days — pass the GlideDateTime object into the query. Do not string-concat it. `gs.daysAgo(30)` is a date string in the user TZ.
- Traps — display vs store; `addDaysUTC(-30)` from now is not midnight 30 days ago; `isValid()` before you trust a parse; invalid string is empty, not now.

**4. GlideAjax**
- Client — catalog, portal, UI script. `addParam('sysparm_name', …)`, `getXMLAnswer`. Do not use `getXMLWait`.
- Server — Script Include, Client callable checked. `extends AbstractAjaxProcessor`. Return a string; `JSON.stringify` if you need more than one value.
- Client read of JSON — `JSON.parse(answer)`. Never return a GlideRecord.
- Parameters — `sysparm_name` is the method; `sysparm_*` are yours. `getParameter` is always a string or null. Sanitize.
- Traps — client callable must be checked or the call 404s. ACLs still apply. Not for server-to-server (use RESTMessageV2). OnChange: pass `newValue`; do not reread the form after the async gap.

**5. gs · GlideSystem**
- Log and tell the user — `info` / `warn` / `error` / `debug` go to the log. `addInfoMessage` / `addErrorMessage` show on the next form or list paint.
- Who is running — `getUserID`, `getUserName`, `getUser`, `hasRole`, `getSession`. In a scheduled job, `getUserID()` is the job’s run-as user. `hasRole('admin')` is true for admin even without the role row.
- Dates, the short ones — `nowDateTime`, `daysAgo`, `beginningOfToday`, `endOfToday`, `beginningOfThisMonth`, `monthsAgo`. These return strings, not GlideDateTime.
- Null, message, property — `nil` (`0` is a value; `nil(0)` is false). Prefer `getProperty(name, fallback)`. `setProperty` is rare and needs rights.
- Events and includes — `eventQueue` (parm1 and parm2 are strings, current is the GlideRecord). `include` is the older pattern. Do not queue inside a loop without a reason.
- Where gs is different — client has a smaller gs. Client uses `g_user`, `g_form`, `g_scratchpad`. No GlideRecord on the client except in some portal server scripts. `sleep` stalls the thread.

**6. GlideUser**
- Server — `getID`, `getName`, `getDisplayName`, `getEmail`, `hasRole`, `isMemberOf`, `getCompanyID`, `getDepartmentID`. `hasRoleExactly` is the strict check.
- Preferences — `getPreference` / `savePreference` / `setPreference`. Save writes `sys_user_preference`. Timezone: `getTZ()`.
- Client `g_user` — `userID`, `userName`, `hasRole`, `hasRoleExactly`, `getFullName`. Role checks are UX only. No `getCompanyID`; Ajax it if you need it.
- Impersonation and jobs — `isImpersonating`, `getImpersonatingUserName`. A scheduled job often runs as system. Do not decide security from the client.
- What to call — reference stamp: `gs.getUserID()`. Role: `hasRole` or `current.canWrite()`. Group: `isMemberOf`. Manager: query `sys_user`, not GlideUser.

**7. RESTMessageV2**
- Named message — the one you want in prod. Name and method match the REST Message and HTTP Method records. `setStringParameterNoEscape` if the value has `&` or quotes.
- Ad hoc, no record — `setEndpoint`, `setHttpMethod`, headers, `setRequestBody`, `execute`. Fine for a spike. A REST Message record gives auth, MID, and retry in one place.
- Auth — basic, authentication profile, or bearer header. Prefer a profile. Do not hardcode secrets.
- Response — `getStatusCode` (int), `getBody`, `getHeader`, `getErrorMessage` (transport only), `haveError`. HTTP 500 is not `haveError()`.
- Traps — `execute()` is synchronous and holds the transaction. Do not call it from a before-query business rule. Outbound from a MID: set the MID on the record, or `setEccParameter` / `setMIDServer`. Inbound is Scripted REST, not this API. Set a timeout. Never log the Authorization header.

**8. GlideQuery · typical incident work**
- Base — P1 incidents opened in the last 30 days, assigned to the same caller. No `SAMEAS` operator; match by the same sys_id.
- 1 How many — `count()` is a plain number. `avg` needs `orElse(0)`.
- 2 List them — `orderByDesc`, `limit`, `select`, `forEach`. `$DISPLAY` is the display value; sys_id always comes back.
- 3 Latest one — `selectOne` ignores extra rows. `orElse` if the window is empty.
- 4 Several aggregates at once — by assignment group. Row shape: `row.group`, `row.count`, `row.avg`, `row.max`.
- 5 Callers with 3 or more P1s — `having('count', '>=', 3)` before `select`.
- 6 Their open incidents, display names — `caller_id$DISPLAY`, `assigned_to$DISPLAY`.
- 7 Bulk touch — only after a count. `updateMultiple`. One-row `update` returns Optional.

**9. GlideQuery · three paths**

| Path | Terminal | You get | Next methods |
|---|---|---|---|
| 1 One record | `selectOne` → Optional | The object, or your default | `get` throws if empty. `orElse`, `isPresent` / `isEmpty`, `ifPresent`, `map`, `flatMap`. No `orElseThrow`. |
| 2 Many records | `select` → Stream | One row at a time | `forEach`, `map`, `flatMap`, `filter`, `find` (back to Optional), `reduce`, `toArray`, `some` / `every`. No `orElse`. `limit` before `select`. |
| 3 Aggregate | `count` / `sum` / `avg` | A number, or Optional | `count()` is already a number. `sum` / `avg` / `min` / `max` return Optional — always `orElse(0)`. |

Same name, different type: `select().orElse` is broken (still a Stream). `selectOne().forEach` is broken (still Optional). `select().limit(1)` is still a Stream. `find().orElse()` is the bridge.

Copy-paste recipes on that card: Exists?, One field or null, First of any, Average safe, Collect a few, Nested lookup.
<img width="1500" height="2000" alt="image (1)" src="https://github.com/user-attachments/assets/344138a1-96c2-4098-b346-ce72f233bb54" />
<img width="1500" height="2000" alt="image (2)" src="https://github.com/user-attachments/assets/9c200546-9dd0-49fb-b6e9-1e3d9455f315" />
<img width="1500" height="2000" alt="image (3)" src="https://github.com/user-attachments/assets/252825aa-5c51-46ce-824c-f7c609eb83ae" />
<img width="1500" height="2000" alt="image (4)" src="https://github.com/user-attachments/assets/6cc1e997-4ba3-41bb-9759-8fefe4bcb433" />
<img width="1500" height="2000" alt="image (5)" src="https://github.com/user-attachments/assets/b5088a29-5b05-43bc-a3aa-bbff069a138a" />
<img width="1500" height="2000" alt="image (6)" src="https://github.com/user-attachments/assets/130fc181-39a0-4ad5-8dd2-1d44a2e3db1a" />
<img width="1500" height="2000" alt="image (7)" src="https://github.com/user-attachments/assets/2cd13645-791b-4c69-b181-6699ed1bc830" />
<img width="1354" height="2000" alt="7lWCB" src="https://github.com/user-attachments/assets/cedc65d9-a6d3-41a3-b1f5-178413b4616e" />
<img width="1523" height="2000" alt="8rt31" src="https://github.com/user-attachments/assets/0c3ca232-a738-4a7a-ba2b-8cf1bbad50c9" />
