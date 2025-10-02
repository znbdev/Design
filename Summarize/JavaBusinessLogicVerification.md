Java Business Logic Verification
=====

**业务逻辑相关的校验清单**

**业务逻辑校验一览表**（列表形式），可以直接作为开发/设计/测试时的参考。

---

# 📝 Java 业务逻辑校验一览

| 大分类    | 中分类   | 小分类      | 例子 (Java 说明)                                                                                               |
|--------|-------|----------|------------------------------------------------------------------------------------------------------------|
| 值校验    | 空值校验  | 非空校验     | `Objects.requireNonNull(name, "name不能为空")`                                                                 |
| 值校验    | 默认值校验 | 缺省赋值     | `String status = (input == null) ? "NEW" : input;`                                                         |
| 值校验    | 等值校验  | 必须为指定值   | `if (!"Y".equals(flag)) throw new IllegalArgumentException("flag必须为Y");`                                   |
| 范围校验   | 数值范围  | 最大值校验    | `if (age > 120) throw new IllegalArgumentException("年龄不能大于120");`                                          |
| 范围校验   | 数值范围  | 最小值校验    | `if (age < 0) throw new IllegalArgumentException("年龄不能小于0");`                                              |
| 范围校验   | 日期范围  | 开始小于结束   | `if (startDate.isAfter(endDate)) throw new IllegalArgumentException("开始时间不能大于结束时间");`                      |
| 范围校验   | 日期范围  | 不超过今天    | `if (birthDate.isAfter(LocalDate.now())) throw new IllegalArgumentException("出生日期不能是未来");`                 |
| 长度校验   | 字符串长度 | 最大长度校验   | `if (name.length() > 50) throw new IllegalArgumentException("姓名长度不能超过50");`                                |
| 长度校验   | 字符串长度 | 最小长度校验   | `if (password.length() < 8) throw new IllegalArgumentException("密码至少8位");`                                 |
| 长度校验   | 集合元素数 | 最多N个元素   | `if (list.size() > 100) throw new IllegalArgumentException("最多只能100个元素");`                                 |
| 格式校验   | 字符串模式 | 邮箱校验     | `if (!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) throw new IllegalArgumentException("邮箱格式不正确");`            |
| 格式校验   | 字符串模式 | 手机号校验    | `if (!phone.matches("^1[3-9]\\d{9}$")) throw new IllegalArgumentException("手机号不正确");`                      |
| 格式校验   | 字符串模式 | 日期格式     | `LocalDate.parse(dateStr, DateTimeFormatter.ofPattern("yyyy-MM-dd"));`                                     |
| 枚举校验   | 枚举合法性 | 值必须在枚举中  | `if (!EnumSet.allOf(Status.class).contains(status)) throw new IllegalArgumentException("状态非法");`           |
| 关系校验   | 字段关系  | 依赖字段     | `if ("CREDIT".equals(payType) && creditCardNo == null) throw new IllegalArgumentException("信用卡支付必须提供卡号");` |
| 关系校验   | 字段关系  | 互斥字段     | `if (email != null && phone != null) throw new IllegalArgumentException("邮箱和手机号只能填一个");`                   |
| 唯一性校验  | 数据唯一  | 用户名不能重复  | `if (userRepo.existsByUsername(username)) throw new IllegalArgumentException("用户名已存在");`                   |
| 业务规则校验 | 逻辑规则  | 金额必须为正   | `if (amount.compareTo(BigDecimal.ZERO) <= 0) throw new IllegalArgumentException("金额必须大于0");`               |
| 业务规则校验 | 状态规则  | 状态流转合法   | `if (order.getStatus() == SHIPPED && newStatus == CREATED) throw new IllegalArgumentException("状态流转非法");`  |
| 业务规则校验 | 配额/限制 | 每天最多提交N次 | `if (countToday >= 5) throw new IllegalArgumentException("每天最多提交5次");`                                     |

---

👉 这样分类后，大概覆盖了：

* **值校验**（空、默认、指定值）
* **范围校验**（数值、日期）
* **长度校验**（字符串、集合）
* **格式校验**（正则、日期解析）
* **枚举校验**（合法枚举）
* **关系校验**（依赖、互斥、组合）
* **唯一性校验**（数据库或缓存检查）
* **业务规则校验**（逻辑、状态机、配额）

---

**業務ロジック検証の日本語一覧**
チェックリスト形式**（列：大分類 / 中分類 / 小分類 / 例 / 適用可否 / 優先度）
---

# 📝 Java 業務ロジック検証一覧（日本語版）

| 大分類       | 中分類       | 小分類          | 例 (Javaコード)                                                                                                          |
|-----------|-----------|--------------|----------------------------------------------------------------------------------------------------------------------|
| 値検証       | Null検証    | 非Null検証      | `Objects.requireNonNull(name, "nameは必須です");`                                                                         |
| 値検証       | デフォルト値検証  | 未入力時に初期値を設定  | `String status = (input == null) ? "NEW" : input;`                                                                   |
| 値検証       | 等値検証      | 指定値であること     | `if (!"Y".equals(flag)) throw new IllegalArgumentException("flagはYでなければなりません");`                                     |
| 範囲検証      | 数値範囲      | 最大値検証        | `if (age > 120) throw new IllegalArgumentException("年齢は120以下でなければなりません");`                                           |
| 範囲検証      | 数値範囲      | 最小値検証        | `if (age < 0) throw new IllegalArgumentException("年齢は0以上でなければなりません");`                                               |
| 範囲検証      | 日付範囲      | 開始日 < 終了日    | `if (startDate.isAfter(endDate)) throw new IllegalArgumentException("開始日は終了日以前でなければなりません");`                         |
| 範囲検証      | 日付範囲      | 今日以前であること    | `if (birthDate.isAfter(LocalDate.now())) throw new IllegalArgumentException("誕生日は未来日であってはなりません");`                   |
| 長さ検証      | 文字列長さ     | 最大長さ検証       | `if (name.length() > 50) throw new IllegalArgumentException("名前は50文字以内でなければなりません");`                                 |
| 長さ検証      | 文字列長さ     | 最小長さ検証       | `if (password.length() < 8) throw new IllegalArgumentException("パスワードは8文字以上でなければなりません");`                            |
| 長さ検証      | コレクション要素数 | 最大要素数        | `if (list.size() > 100) throw new IllegalArgumentException("要素数は100以内でなければなりません");`                                  |
| フォーマット検証  | 文字列形式     | メール形式        | `if (!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) throw new IllegalArgumentException("メール形式が不正です");`                   |
| フォーマット検証  | 文字列形式     | 電話番号形式       | `if (!phone.matches("^1[3-9]\\d{9}$")) throw new IllegalArgumentException("電話番号が不正です");`                             |
| フォーマット検証  | 文字列形式     | 日付フォーマット     | `LocalDate.parse(dateStr, DateTimeFormatter.ofPattern("yyyy-MM-dd"));`                                               |
| 列挙値検証     | 列挙値の有効性   | 列挙値に含まれていること | `if (!EnumSet.allOf(Status.class).contains(status)) throw new IllegalArgumentException("不正なステータスです");`               |
| 関連性検証     | 項目依存関係    | 依存項目必須       | `if ("CREDIT".equals(payType) && creditCardNo == null) throw new IllegalArgumentException("クレジット決済の場合、カード番号は必須です");` |
| 関連性検証     | 相互排他      | 同時入力禁止       | `if (email != null && phone != null) throw new IllegalArgumentException("メールと電話番号は同時に入力できません");`                     |
| 一意性検証     | データ一意性    | ユーザー名重複禁止    | `if (userRepo.existsByUsername(username)) throw new IllegalArgumentException("ユーザー名は既に存在します");`                      |
| ビジネスルール検証 | 論理ルール     | 金額は正の数       | `if (amount.compareTo(BigDecimal.ZERO) <= 0) throw new IllegalArgumentException("金額は正の値でなければなりません");`                |
| ビジネスルール検証 | 状態遷移      | 状態遷移の妥当性     | `if (order.getStatus() == SHIPPED && newStatus == CREATED) throw new IllegalArgumentException("不正な状態遷移です");`         |
| ビジネスルール検証 | 制限/回数     | 1日N回まで       | `if (countToday >= 5) throw new IllegalArgumentException("1日に5回までしか申請できません");`                                       |

---

**業務ロジックに必要なチェック一覧（日文版）**を「大分類| 中分類| 小分類| 例子（サンプル）」の形式で整理しました。

---

# ✅ Java 業務ロジックチェック一覧（サンプル）

| 大分類       | 中分類        | 小分類        | 例子（サンプル）                                   |
|-----------|------------|------------|--------------------------------------------|
| 値チェック     | 必須チェック     | 未入力チェック    | 名前が空文字でないこと name == null OR name.isEmpty() | 
| 値チェック     | 固定値チェック    | 許可値チェック    | ステータスが「NEW」「DONE」のみ許容                      | 
| 値チェック     | Null許可チェック | Null禁止     | 金額は必ず入力必須                                  | 
| 値チェック     | ブールチェック    | 真偽値        | フラグは true/false のみ許容                       | 
| 範囲チェック    | 数値範囲       | 最小値チェック    | 年齢 >= 18                                   | 
| 範囲チェック    | 数値範囲       | 最大値チェック    | 金額 <= 1000000                              | 
| 範囲チェック    | 日付範囲       | 開始日 <= 終了日 | 契約開始日 < 契約終了日                              | 
| 範囲チェック    | 日付範囲       | 未来日禁止      | 誕生日は現在日付より未来不可                             | 
| 範囲チェック    | 日付範囲       | 過去日禁止      | 有効期限は過去日不可                                 | 
| 範囲チェック    | 件数範囲       | リスト最大件数    | 申請書の明細は最大50件まで                             | 
| 長さチェック    | 文字列長       | 最大長チェック    | 氏名は100文字以内                                 | 
| 長さチェック    | 文字列長       | 最小長チェック    | パスワードは8文字以上                                | 
| 長さチェック    | 文字列長       | 固定長チェック    | 郵便番号は7桁固定                                  | 
| 形式チェック    | 文字種        | 半角数字のみ     | 電話番号は半角数字のみ許可                              | 
| 形式チェック    | 文字種        | 全角のみ       | 氏名カナは全角カタカナのみ                              | 
| 形式チェック    | 正規表現       | メール形式      | `aaa@bbb.com` の形式であること                     | 
| 形式チェック    | コード形式      | 郵便番号形式     | 「NNN-NNNN」形式                               | 
| 相関チェック    | クロスチェック    | 開始・終了比較    | 勤務開始時刻 < 勤務終了時刻                            | 
| 相関チェック    | 複数項目依存     | セット必須      | 住所入力時は郵便番号必須                               | 
| 相関チェック    | 一意性        | DB重複チェック   | ユーザIDが既存と重複しないこと                           | 
| 相関チェック    | 組み合わせ制約    | 禁止組合せ      | 商品Aと商品Bは同時申込不可                             | 
| 業務ルールチェック | 条件分岐       | 年齢制限       | 未成年は申込不可                                   | 
| 業務ルールチェック | 依存条件       | ステータス遷移    | 「申請中」→「承認済」以外の遷移禁止                         | 
| 業務ルールチェック | 制御値        | 金額計算       | 合計金額 = 明細合計であること                           | 
| 業務ルールチェック | 重複防止       | 多重申請       | 同じ申請は24時間以内に重複不可                           | 

---

📌 この一覧は **Java の業務アプリ開発でよく使うチェック項目を整理** したものです。
もしご希望なら、**Javaのコード例付きの一覧** も作成できます。

了解しました！
では、先ほどの一覧をさらに細かく分類した **「Java 業務ロジックチェック一覧（日文・詳細版）」** を作ります。
できるだけ「Empty」「Null」「境界値」「依存条件」「禁止条件」などを網羅して、実際の設計書レベルで使えるように整理します。

---

# ✅ Java 業務ロジックチェック一覧（日文・詳細版）

### 🔹 値チェック

### 🔹 数値チェック

### 🔹 文字列チェック

### 🔹 日付チェック

### 🔹 相関チェック

### 🔹 業務ルールチェック

| 大分類       | 中分類     | 小分類         | 例子（サンプル）              | 
|-----------|---------|-------------|-----------------------|
| 値チェック     | 必須チェック  | Null禁止      | ユーザIDが `null` でないこと   | 
| 値チェック     | 必須チェック  | 空文字禁止       | 名前が `""` でないこと        | 
| 値チェック     | 必須チェック  | 空白文字禁止      | 入力が `"   "` のみは禁止     | 
| 値チェック     | 固定値チェック | 列挙型チェック     | 性別は「M」「F」のみ許可         | 
| 値チェック     | 固定値チェック | 定数一致        | 権限は「ADMIN」「USER」のみ    | 
| 値チェック     | 真偽値     | Booleanチェック | フラグは true/false のみ許可  | 
| 値チェック     | Null許容  | オプション入力     | 備考は空でもOK              |
| 数値チェック    | 境界値     | 最小値チェック     | 年齢 >= 18              | 
| 数値チェック    | 境界値     | 最大値チェック     | 得点 <= 100             | 
| 数値チェック    | 境界値     | 範囲内チェック     | 金額 1000 ～ 1,000,000   | 
| 数値チェック    | 符号チェック  | 正数のみ        | 数量 > 0                | 
| 数値チェック    | 符号チェック  | 負数のみ        | 残高 < 0 の場合のみ警告        | 
| 数値チェック    | ゼロ判定    | ゼロ禁止        | 割り算の除数 ≠ 0            |
| 文字列チェック   | 長さチェック  | 最大長チェック     | 氏名は100文字以内            | 
| 文字列チェック   | 長さチェック  | 最小長チェック     | パスワードは8文字以上           | 
| 文字列チェック   | 長さチェック  | 固定長チェック     | 郵便番号は7桁固定             | 
| 文字列チェック   | 文字種チェック | 半角数字のみ      | 電話番号は半角数字のみ           | 
| 文字列チェック   | 文字種チェック | 全角カタカナのみ    | フリガナは全角カタカナ限定         | 
| 文字列チェック   | 形式チェック  | 正規表現        | メールは `aaa@bbb.com` 形式 | 
| 文字列チェック   | 形式チェック  | 禁止文字        | 特殊文字 `<>&` は禁止        |
| 日付チェック    | 必須チェック  | Null禁止      | 契約開始日は必須              | 
| 日付チェック    | 境界値     | 過去日禁止       | 有効期限は今日以降             | 
| 日付チェック    | 境界値     | 未来日禁止       | 誕生日は未来不可              | 
| 日付チェック    | 範囲比較    | 開始日 < 終了日   | 契約開始 < 契約終了           | 
| 日付チェック    | 範囲比較    | 当日含む判定      | イベント日 >= 今日           | 
| 日付チェック    | 期間制限    | 最大期間        | 予約期間は90日以内            |
| 相関チェック    | 項目依存    | セット必須       | 住所を入力したら郵便番号必須        | 
| 相関チェック    | 項目依存    | 条件付き必須      | 法人番号入力時は会社名必須         | 
| 相関チェック    | 相互比較    | 大小関係        | 勤務開始時刻 < 終了時刻         | 
| 相関チェック    | 相互比較    | 金額一致        | 合計金額 = 明細合計           | 
| 相関チェック    | 一意性     | DB重複チェック    | ユーザIDが既存と重複不可         | 
| 相関チェック    | 組合せ禁止   | 排他条件        | 商品Aと商品Bは同時申込不可        | 
| 相関チェック    | 組合せ制約   | 依存条件        | 支払方法「クレジット」時はカード番号必須  |
| 業務ルールチェック | 条件分岐    | 年齢制限        | 未成年は申込不可              | 
| 業務ルールチェック | 条件分岐    | 金額制限        | 法人契約は金額100万円以上        | 
| 業務ルールチェック | 状態遷移    | 許可遷移        | 「申請中」→「承認済」は許可        | 
| 業務ルールチェック | 状態遷移    | 禁止遷移        | 「承認済」→「申請中」は禁止        | 
| 業務ルールチェック | 重複防止    | 多重申請        | 同じユーザは24時間以内に再申請不可    | 
| 業務ルールチェック | 集計制約    | 件数制限        | 申請書の明細は最大50件          | 
| 業務ルールチェック | 集計制約    | 合計値制御       | 申込合計金額 = 各項目合計        | 

---

📌 これで **Empty/Null/日付比較/相関チェック/禁止条件** を含めた **より細かい分類表** になりました。
📌 設計書にそのまま流用できるレベルを意識しています。
