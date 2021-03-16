---
layout: post
title: "ModSecurity 設定"
date: 2017-02-15 21:52 +0800
tags:
  - Security
  - SysAdmin
---

ModSecurity 是一套 WAF，你可以把它裝在 Apache 上面，因為計中叫我弄，就稍微研究一下如何設定它好惹。

{% include toc.html %}

## 概要

- 使用環境為 CentOS
- ModSecurity 預設不會做任何事，都要自己寫規則
- 安全可能犧牲效能甚至擋掉合法的使用者


## 設定檔


主要設定檔
- /etc/httpd/conf.d/mod_security.conf


規則設定檔會被主要設定檔 include

- /etc/httpd/mod_security.d/\*.conf
- /etc/httpd/mod_security.d/activated_rules/\*.conf



## 試試你的第一條規則

「不准別人用 curl 看我們的網站」

1.  編輯 /etc/httpd/mod_security.d/NO-MORE-CURL.conf
    ```
    SecRule REQUEST_HEADERS:User-Agent "@contains curl" phase:1,id:87,deny,status:403
    ```

    Syntax: `SecRule <variables> <operator> <actions>`

    - `REQUEST_HEADERS:User-Agent` 檢查 Request Header 中的 User-Agent 欄位
    - `@contains curl` 比對若此參數含有 "curl" 則執行下方動作
    - `phase:1,deny,id:87` 將此規則放到 phase 1 (Phase Request Headers)；拒絕請求；規則的 id 設為 87（每條規則一定要指定 id）

2. 重啟 Apache
3. 實際測試

```
$ curl localhost:8080
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /
on this server.</p>
</body></html>
```


## Processing Phases

如果 rule 設在前面的 phase 有可能因為 Apache 還未取得資料而無法比對。例如 phase:1 的 rule 比對 RESPONSE_HEADERS 時。分成這麼多階段是為了可以精準過濾髒東西，不讓髒東西跑掉下一個階段處理。每個 rule 要在 actions 指定自己的 phase，如 `phase:2` 。

1. Phase Request Headers（此時可以比對請求檔頭）
2. Phase Request Body（此時可以比對請求的 Body，以此類推）
3. Phase Response Headers（此時還沒送出回應，但是有些參數已經確定了，無法改，例如 status）
4. Phase Response Body（此時已經送出回應，若要存取內容必須 `SecResponseBodyAccess On`）
5. Phase Logging

## 試試你的第二條規則

隨便過濾一下資料庫注入關鍵字，把壞蛋導到 FBI 的網站。

```
SecRule ARGS \
\ select|\ union|#|\/\*|\*\/|\(|\)|'|\"|--|\ or|\ and \
t:lowercase,phase:1,id:87,redirect:https://www.fbi.gov/investigate/cyber
```

- 中間如果沒有設定 `@contains` 之類的 operator 就是默認用正規表達式
- `t:lowercase` 會先把要比對的東西轉成小寫

## 常用指令（Derictives）

先介紹幾個一定要會的。

### SecRuleEngine

開啟或關閉過濾功能，語法：`SecRuleEngine On|Off|DetectionOnly`
- DetectionOnly: 開啟，只紀錄不擋掉請求

### SecRule

已介紹，語法：`SecRule VARIABLES OPERATOR [ACTIONS]`

例如：
```
SecRule ARGS "@rx attack" "phase:1,log,deny,id:1" 
```

- `ARGS` 參數，包含 GET, POST 的參數

### SecDefaultAction

設定預設動作
語法：`SecDefaultAction "action1,action2,...,actionN"`

例如：
```
SecDefaultAction "phase:2,log,auditlog,deny,status:404"
```

- `phase:2` 針對 Phase Request Body 中的規則
- `log` 讓 Apache 紀錄檔記下這筆
- `auditlog` 寫進 ModSecurity 的 audit log
- `deny` 設定預設的破壞性動作為 deny，讓這個請求的流程不再進行下去
- `status:404` 傳回 404 Not Found（這個是跟 deny 一起的）

接下來的規則也可以覆寫這些預設值

### SecAction

相當於無條件的 SecRule，語法：`SecAction "action1,action2,...,actionN"`

例如：
```
SecDefaultAction phase:2,deny,status:403
SecAction id:8787,block
```
會根據預設的破壞性動作「無條件」deny 掉請求並回應 403。

## 所有指令 

指令太多，要用的時候再自己看文件就好了：
https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual

- SecAction
- SecArgumentSeparator
- SecAuditEngine, SecAuditLog, SecAuditLog2, SecAuditLogDirMode, SecAuditLogFormat, SecAuditLogFileMode, SecAuditLogParts, SecAuditLogRelevantStatus, SecAuditLogStorageDir, SecAuditLogType
- SecCacheTransformations
- SecChrootDir
- SecCollectionTimeout
- SecComponentSignature
- SecConnEngine
- SecContentInjection
- SecCookieFormat, SecCookieV0Separator
- SecDataDir
- SecDebugLog, SecDebugLogLevel
- SecDefaultAction
- SecDisableBackendCompression
- SecHashEngine, SecHashKey, SecHashParam, SecHashMethodRx, SecHashMethodPm
- SecGeoLookupDb, SecGsbLookupDb
- SecGuardianLog
- SecHttpBlKey
- SecInterceptOnError
- SecMarker
- SecPcreMatchLimit, SecPcreMatchLimitRecursion
- SecPdfProtect, SecPdfProtectMethod, SecPdfProtectSecret, SecPdfProtectTimeout, SecPdfProtectTokenName
- SecReadStateLimit
- SecConnReadStateLimit
- SecSensorId
- SecWriteStateLimit
- SecConnWriteStateLimit
- SecRemoteRules, SecRemoteRulesFailAction
- SecRequestBodyAccess, SecRequestBodyInMemoryLimit, SecRequestBodyLimit, SecRequestBodyNoFilesLimit, SecRequestBodyLimitAction
- SecResponseBodyLimit, SecResponseBodyLimitAction, SecResponseBodyMimeType, SecResponseBodyMimeTypesClear, SecResponseBodyAccess
- SecRule, SecRuleInheritance, SecRuleEngine, SecRulePerfTime, SecRuleRemoveById, SecRuleRemoveByMsg, SecRuleRemoveByTag, SecRuleScript, SecRuleUpdateActionById, SecRuleUpdateTargetById, SecRuleUpdateTargetByMsg, SecRuleUpdateTargetByTag
- SecServerSignature
- SecStatusEngine
- SecStreamInBodyInspection, SecStreamOutBodyInspection
- SecTmpDir
- SecUnicodeMapFile, SecUnicodeCodePage
- SecUploadDir, SecUploadFileLimit, SecUploadFileMode, SecUploadKeepFiles
- SecWebAppId
- SecXmlExternalEntity


## 常用變數（Variables）

### ARGS
請求參數，例如 GET, POST 的參數。

另有：
- ARGS_COMBINED_SIZE（參數長度）
- ARGS_GET
- ARGS_GET_NAMES
- ARGS_NAMES
- ARGS_POST
- ARGS_POST_NAMES

### ENV
可以存取由 ModSecurity 設定的環境變數，有可以設定環境變數的 action，叫做「setenv」，語法如下：
```
SecAction setenv:tag=suspicious
SecRule ENV:tag suspicious id:16
```

### PERF_COMBINED
ModSecurity 花了多久時間處理這個 Request？單位 ms。

```
SecRule PERF_COMBINED "@ge 1000" ....
```

- `@ge 1000` "(G)reater than or (E)qual to" 1000

另有：
- PERF_GC
- PERF_LOGGING
- PERF_PHASE1
- PERF_PHASE2
- PERF_PHASE3
- PERF_PHASE4
- PERF_PHASE5
- PERF_RULES
- PERF_SREAD
- PERF_SWRITE

### QUERY_STRING
還沒有處理過的，`?` 後面那一堆。

### REMOTE_ADDR
TCP 連線對方的 IP 位址。另有：
- REMOTE_HOST
- REMOTE_USER
- REMOTE_PORT

### REQUEST_BODY
生的 request body。另有：
- REQBODY_ERROR
- REQBODY_ERROR_MSG
- REQBODY_PROCESSOR
- REQUEST_BODY
- REQUEST_BODY_LENGTH

### 其他 REQUEST 系列
- REQUEST_BASENAME（請求的檔名，例如 `index.html`）
- REQUEST_COOKIES
- REQUEST_COOKIES_NAMES
- REQUEST_FILENAME（拿掉參數的 URL，例如 `/index.html`）
- REQUEST_HEADERS
- REQUEST_HEADERS_NAMES
- REQUEST_LINE（整個請求檔頭）
- REQUEST_METHOD（例如 GET, POST）
- REQUEST_PROTOCOL（如 HTTP, HTTPS）
- REQUEST_URI（不含 Hostname 如 `/index.php?p=X`）
- REQUEST_URI_RAW（沒有經過 urldecode 的 REQUEST_URI）

例如：Cookie 中沒有有某欄位名稱就滾
```
SecRule &REQUEST_COOKIES:bad "@eq 0" id:1487,block
```
- 想像 REQUEST_COOKIES 是個 hash，bad 是一個鍵名，REQUEST_COOKIES:bad 是在取值
- `&` 符號是取得變數數量，沒有就是 0
- 不等於零可以這樣寫 `!@eq 0`

### SERVER 系列

- SERVER_ADDR（自己的位址）
- SERVER_NAME（自己的 Hostname）
- SERVER_PORT（自己的 Port）

舉例：
```
SecRule SERVER_ADDR "@ipMatch 192.168.1.100" "id:67"
SecRule SERVER_NAME "hostname\.com$" "id:68"
SecRule SERVER_PORT "^80$" "id:69"
```

- `@ipMatch` 是一個用來比對位址的 operator
- 如果沒有特別寫 operator 就是正規表達式，跟 `@rx ...` 效果一樣

### SESSION

必須先用 REQUEST_COOKIES 拿到 PHPSESSID 在用 setsid 設定 session id：
```
SecRule REQUEST_COOKIES:PHPSESSID !^$ "phase:2,id:70,nolog,pass,setsid:%{REQUEST_COOKIES.PHPSESSID}"
```
甚至可以修改 session 的值，用 session 來計數：
```
SecRule REQUEST_URI "^/cgi-bin/finger$" "phase:2,id:71,t:none,t:lowercase,t:normalizePath,pass,setvar:SESSION.score=+10" 
SecRule SESSION:score "@gt 50" "phase:2,id:72,pass,setvar:SESSION.blocked=1"
SecRule SESSION:blocked "@eq 1" "phase:2,id:73,deny,status:403"
```

- `t:lowercase` 是一個 transformation function，會先轉換要比對的東西，還有很多其他 transformation function。
- `setvar:SESSION.score=+10` 會把 session 裡面的 score 變數加上 10

### TIME
時間（hour:min:sec），還有：
- TIME_DAY
- TIME_EPOCH
- TIME_HOUR
- TIME_MIN
- TIME_MON
- TIME_SEC
- TIME_WDAY
- TIME_YEAR

### TX
拿來暫存的變數，等到這個請求處理完之後就消失。
```
SecRule ARGS attack "phase:2,id:82,nolog,pass,setvar:TX.score=+5"
```

注意此處 `TX.score` 寫法跟 SecRule 的第一個參數不一樣，如果要比對 `TX.score` 在 SecRule 第一個參數要寫成 `TX:score` ，大小寫沒差

## 轉換函式（Transformation functions）
- base64Decode, sqlHexDecode, base64DecodeExt, base64Encode
- cmdLine
- compressWhitespace
- cssDecode, htmlEntityDecode, jsDecode
- escapeSeqDecode
- hexDecode, hexEncode
- length
- lowercase
- none（移除目前規則內所有的轉換函式）
- normalisePath, normalisePathWin
- parityEven7bit, parityOdd7bit, parityZero7bit
- removeNulls, removeWhitespace, removeCommentsChar, removeComments
- replaceComments, replaceNulls
- urlDecode, urlDecodeUni, urlEncode, utf8toUnicode
- sha1, md5
- trimLeft, trimRight, trim

## 常用動作（Actions）

### 破壞性動作（Disruptive actions）
讓 ModSecurity 有所行動的動作。例如 deny, block, redirect, allow。如果 SecRuleEngine 不是 On，破壞性動作不會有效果。一個 rule 只能有一個此類型的 action。

- allow 結束比對，讓 apache 把這個 request 走完
- block 執行前一個 SecDefaultAction 中定義的破壞性動作
- deny 結束比對，終止此 request 的處理
- drop 直接斷線
- pass 比對成功也繼續進行下一個規則的比對
- pause:TIME 終止一段時間，單位 ms
- proxy\:PROXT 轉送這個 request 給其他 web server
- redirect:URL 重導向

### 非破壞性動作（Non-disruptive actions）
不會影響流程，例如設定變數。

- setenv:, setvar:, setuid:, setrsc:, setsid:
- log, logdata:TEXT, auditlog, nolog, noauditlog
- sanitiseArg, sanitiseMatched, sanitiseMatchedBytes, sanitiseRequestHeader, sanitiseResponseHeader
- t:TF（指定轉換函式）
- append:TEXT（在回應最後面加文字）, prepend:TEXT（前面）
- capture（用正規表達式時，把抓到的存到 `TX.1` 等變數）
- ctl:（修改設定）
- deprecatevar:（設定讓變數的值隨時間減少）, expirevar:TIME（設定變數生存時間）
- initcol, multiMatch, exec（執行 lua 腳本）

### 流程動作（Flow actions）
會改變流程，例如 skip, skipAfter。

- chain
    - 串連下一條規則，類似邏輯 AND
    - 會改變流程的動作只能寫在第一個規則
    - 可以串連 SecAction, SecRule, SecRuleScript
- skip:N（如果此規則比對成功就跳過 N 條規則）
- skipAfter（看 SecMarker）


chain 後面的 rule 會繼承前面的 action
```
SecRule ARGS select phase:1,id:1,chain,block
    SecRule ARGS union
```


### 資料定義動作（Meta-data actions）
提供更多關於規則的資訊，例如 id, rev, severity, msg。

- tag\:TAG（分類）
- ver:VER（版本）
- id:, accuracy:（判斷精準度）, maturity:（成熟度）, severity:（嚴重度）
- msg:MESSAGE（MESSAGE 會被寫進 log 裡面）
- phase:N（把這條規則放到第 N 階段去比對）

### 附帶動作（Data actions）
例如 status 指定擋掉之後的狀態。

- status:N（指定 deny 或 redirect 的回應狀態）
- xmlns

## 運算子（Operators）
要加 `@`，例如 `@within GET,POST,HEAD`

- beginsWith, endsWith
- contains, containsWord, within, noMatch
- detectSQLi, detectXSS
- fuzzyHash
- geoLookup, gsbLookup
- inspectFile
- ipMatch, ipMatchF, ipMatchFromFile
- eq, ge, le, gt, lt
- pm, pmf, pmFromFile
- rbl, rsub
- rx
- streq, strmatch, unconditionalMatch
- validateByteRange, validateDTD, validateHash, validateSchema, validateUrlEncoding, validateUtf8Encoding
- verifyCC, verifyCPF, verifySSN

## 巨集（Macro）

- `%{VARIABLE}`
- `%{COLLECTION.VARIABLE}`

## 簡單用 Docker 測試規則

用 docker 不用怕把環境搞爛

1. 先抓 CentOS 開個 container
   ```
   docker pull centos
   docker run -it centos bash
   ```

2. 在 container 安裝需要的東西
   ```
   yum update -y
   yum install -y httpd mod_security vim
   ```

3. 另外開一個終端機把環境做成 image 存起來
   ```
   docker ps
   docker commit 760d7e9c342c modsec
   ```

4. 啟動 apache
   ```
   docker run -itp 8080:80 modsec bash
   httpd -k start
   ```


## OWASP Core Rule Set (CRS)

現成的規則設定檔，支援：
- SQLi, XSS, LFI, RFI, RCE
- PHP Code Injection
- HTTP Protocol Violations
- Shellshock
- Session Fixation (?)
- Scanner Detection
- Metadata/Error Leakages
- Project Honey Pot Blacklist (?)
- GeoIP Country Blocking (?)

### 安裝

```
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
cd owasp-modsecurity-crs
mv crs-setup.conf.example crs-setup.conf
ln -s `pwd` /etc/httpd/modsecurity.d/crs
```

編輯 conf.d/mod_security.conf

```apache
    ...
    Include modsecurity.d/crs/crs-setup.conf
</IfModule>
```

我寫不下去了，試下的自己看文件好嗎？

