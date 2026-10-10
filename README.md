# Java LocalDate 学習ノート

Javaで日付を扱うための `LocalDate` と `ChronoUnit` のまとめです。
図書館の貸出システム課題（返却期限・延滞判定）で使った内容をもとにしています。

## 目次

1. [LocalDateとは](#1-localdateとは)
2. [作る](#2-作る)
3. [足す・引く](#3-足す引く)
4. [比べる](#4-比べる)
5. [取り出す](#5-取り出す)
6. [日数の差（ChronoUnit）](#6-日数の差chronounit)
7. [表示の形を変える](#7-表示の形を変える)
8. [nullとの関係](#8-nullとの関係)
9. [図書館課題での使い方](#9-図書館課題での使い方)
10. [動作確認用サンプル](#10-動作確認用サンプル)
11. [つまずきやすい点](#11-つまずきやすい点)
12. [早見表](#12-早見表)

---

## 1. LocalDateとは

- **日付だけ**（年・月・日）を持つ型。時刻は持たない
- `java.time` パッケージにある（Java 8以降）
- `new` では作れず、専用のメソッドで作る
- **イミュータブル（変更不可）**。足し算をしても元の値は変わらず、新しい `LocalDate` が返る

```java
import java.time.LocalDate;
```

## 2. 作る

```java
LocalDate today = LocalDate.now();               // 今日 → 2026-10-10
LocalDate xmas  = LocalDate.of(2026, 12, 25);    // 指定日 → 2026-12-25
LocalDate d     = LocalDate.parse("2026-10-24"); // 文字列から（形式は yyyy-MM-dd）

// LocalDate x = new LocalDate();                // ✗ コンパイルエラー
```

## 3. 足す・引く

```java
LocalDate today = LocalDate.now();   // 2026-10-10

today.plusDays(14);     // 2026-10-24
today.plusWeeks(2);     // 2026-10-24
today.plusMonths(1);    // 2026-11-10
today.plusYears(1);     // 2027-10-10
today.minusDays(3);     // 2026-10-07
```

**結果を受け取らないと意味がありません。**

```java
today.plusDays(14);                   // ✗ 結果を捨てている（todayは変わらない）
LocalDate due = today.plusDays(14);   // ○ 変数で受け取る
today = today.plusDays(14);           // ○ 同じ変数に入れ直してもよい
```

## 4. 比べる

`boolean` が返ります。

```java
LocalDate due   = LocalDate.of(2026, 10, 24);
LocalDate today = LocalDate.of(2026, 10, 27);

due.isBefore(today);    // true   dueはtodayより前
due.isAfter(today);     // false  dueはtodayより後ではない
due.isEqual(today);     // false  同じ日ではない
```

期限切れの判定：

```java
if (due.isBefore(today)) {
    System.out.println("期限切れです");
}
```

## 5. 取り出す

```java
LocalDate d = LocalDate.of(2026, 10, 10);

d.getYear();          // 2026
d.getMonthValue();    // 10（1〜12）
d.getDayOfMonth();    // 10
d.getDayOfWeek();     // SATURDAY（DayOfWeek型。表示は英語）
d.lengthOfMonth();    // 31（その月の日数）
d.isLeapYear();       // false（うるう年か）
```

## 6. 日数の差（ChronoUnit）

```java
import java.time.temporal.ChronoUnit;

LocalDate due   = LocalDate.of(2026, 10, 24);
LocalDate today = LocalDate.of(2026, 10, 27);

long late = ChronoUnit.DAYS.between(due, today);   // 3
long back = ChronoUnit.DAYS.between(today, due);   // -3（順番を逆にすると負）
```

- 戻り値は **`long`**（`int` に入れるとエラー）
- **`between(前, 後)`** の順に書く
- 後の日付のほうが新しければ正、古ければ負、同じ日なら `0`
- 単位は `DAYS` のほか `WEEKS`、`MONTHS`、`YEARS` が使える

## 7. 表示の形を変える

```java
import java.time.format.DateTimeFormatter;

LocalDate d = LocalDate.of(2026, 10, 24);

System.out.println(d);   // 2026-10-24（そのまま出すとこの形）

DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyy年M月d日");
System.out.println(d.format(f));    // 2026年10月24日

DateTimeFormatter f2 = DateTimeFormatter.ofPattern("yyyy/MM/dd");
System.out.println(d.format(f2));   // 2026/10/24
```

| 記号 | 意味 |
|---|---|
| `yyyy` | 年（4桁） |
| `M` / `MM` | 月（`MM` は0埋め：`01`〜`12`） |
| `d` / `dd` | 日（`dd` は0埋め） |

## 8. nullとの関係

```java
private LocalDate dueDate;     // フィールドで宣言だけなら null

dueDate.plusDays(14);          // ✗ null に対して呼ぶと NullPointerException
dueDate = LocalDate.now();     // ○ 先に日付を入れてから使う
```

「期限がまだない」状態を `null` で表すのは自然です。ただし、`null` のまま使わないように注意します。

## 9. 図書館課題での使い方

```java
// Book.java：借りたとき
public void setBorrowed() {
    borrowed = true;
    dueDate = LocalDate.now().plusDays(14);   // 借りた日の14日後
}

// Book.java：返したとき
public void setReturn() {
    borrowed = false;
    dueDate = null;
}

// Member.java：返却時の延滞判定（setReturn の前に行う）
LocalDate today = LocalDate.now();
if (returnBook.getDueDate().isBefore(today)) {
    long late = ChronoUnit.DAYS.between(returnBook.getDueDate(), today);
    System.out.println(late + "日遅れています");
}
returnBook.setReturn();
```

### 延滞のテスト方法

14日待てないので、一時的に `plusDays(14)` を `plusDays(-3)` に変えます。借りてすぐ返すと `3日遅れています` と出ます。**確認したら必ず14に戻します。**

## 10. 動作確認用サンプル

新しいファイルに貼って実行すると、動きを確認できます。

```java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;

public class LocalDateSample {
    public static void main(String[] args) {
        LocalDate today = LocalDate.now();
        LocalDate due = today.plusDays(14);
        LocalDate xmas = LocalDate.of(2026, 12, 25);

        System.out.println("今日: " + today);
        System.out.println("14日後: " + due);
        System.out.println("曜日: " + today.getDayOfWeek());

        System.out.println("期限は今日より前か: " + due.isBefore(today));   // false
        System.out.println("期限は今日より後か: " + due.isAfter(today));    // true

        long days = ChronoUnit.DAYS.between(today, xmas);
        System.out.println("クリスマスまで: " + days + "日");

        DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyy年M月d日");
        System.out.println("期限（日本語表記）: " + due.format(f));
    }
}
```

## 11. つまずきやすい点

1. `new LocalDate()` は書けない。`now()` か `of()` を使う
2. `plusDays` などの結果は**受け取る**
3. `between` は **`long`** で受ける
4. `between(前, 後)` の**順番**を逆にすると符号が反転する
5. `null` の `LocalDate` にメソッドを呼ぶと `NullPointerException` で落ちる
6. `dueDate = dueDate.plusDays(14)` は、`dueDate` が `null` だと落ちる。借りた瞬間は `LocalDate.now().plusDays(14)` で作る
7. 延滞判定は `setReturn()`（`null` にする）の**前**に行う

## 12. 早見表

| やりたいこと | 書き方 |
|---|---|
| 今日を作る | `LocalDate.now()` |
| 指定日を作る | `LocalDate.of(年, 月, 日)` |
| n日後 | `日付.plusDays(n)`（結果を受け取る） |
| 前か後か | `isBefore()` / `isAfter()` / `isEqual()` |
| 日数の差 | `ChronoUnit.DAYS.between(前, 後)`（`long`） |
| 表示を変える | `DateTimeFormatter.ofPattern(...)` |
