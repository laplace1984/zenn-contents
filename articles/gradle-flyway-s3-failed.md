---
title: "flyway gradle plugin で Amazon S3 バケットのファイルを利用したかった (失敗の記録)"
emoji: "😵‍💫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flyway", "s3", "gradle", "kotlin"]
published: true
---

## TL;DR
- flyway gradle plugin で Amazon S3 バケットのファイルを利用してマイグレーションできそうだったのでやってみた
- 結局実装できなかったので別の方法にした

## 試した環境
| 環境 | バージョン |
| :-- | :-- |
| flyway | 8.5.10 |
| gradle | 7.5.1 |
| Java (gradle実行用) | 11 |
| PostgreSQL | 14 |
| macOS | Ventura 13.2.1 |

## 事の発端
「flyway の location 指定で Amazon S3 のバケットを指定できる」[^1] という記述を見つけた

## やってみたかった理由
- 環境ごとに違うデータを作るために、flyway経由でのマイグレーションが必要だった (主にDDL)
- 環境ごとにS3バケットを作成し、その中にマイグレーションファイルをポン置きしておけばマイグレーションがラクになんじゃね？と思った

## 実際にやってみた
### gradle ファイルの中身
:::details 参考）変更前の gradle
```gradle:build.gradle.kts
plugins {
    id("org.flywaydb.flyway") version "8.5.10"
}

repositories {
    mavenCentral()
}

dependencies {
    runtimeOnly("org.postgresql:postgresql:42.3.6")
}

@OptIn(ExperimentalStdlibApi::class)
flyway {
    url = "jdbc:postgresql://localhost:5432/test-db"
    user = "postgres"
    password = "postgres"
    defaultSchema = "public"
    // jOOQ の除外対象とするため、テーブル名を固定する
    table = "flyway_schema_history"
    loggers = arrayOf("auto")

    locations = buildList {
        add("filesystem:./src/main/resources/db/migration")
    }.toTypedArray()
}
```
:::

```gradle:build.gradle.kts
plugins {
    id("org.flywaydb.flyway") version "8.5.10"
}

repositories {
    mavenCentral()
}

dependencies {
    runtimeOnly("org.postgresql:postgresql:42.3.6")
    // Amazon S3 の依存性を追加 (S3 は v2 じゃないとダメ / implementation ではダメ)
    runtimeOnly("software.amazon.awssdk:s3:2.20.18")
}

@OptIn(ExperimentalStdlibApi::class)
flyway {
    url = "jdbc:postgresql://localhost:5432/test-db"
    user = "postgres"
    password = "postgres"
    defaultSchema = "public"
    // jOOQ の除外対象とするため、テーブル名を固定する
    table = "flyway_schema_history"
    loggers = arrayOf("auto")

    locations = buildList {
        add("filesystem:./src/main/resources/db/migration")
        // ここ追加
        add("s3:test-bucket")
    }.toTypedArray()
}
```

## やってみた
### 環境変数に AWS の接続情報を入れる
```shell
export AWS_ACCESS_KEY_ID=xxxx
export AWS_SECRET_ACCESS_KEY=yyyy
# AWSへSSOしている場合は必要
export AWS_SESSION_TOKEN=zzzz
```

### flyway を動かす
```shell
./gradlew flywayMigrate
```

### エラー発生 😭
`java.lang.ClassNotFoundException: software.amazon.awssdk.core.exception.SdkClientException` が発生した[^2]

## 他に試してみたこと
- gradle の依存性を `compileOnly` / `compile` / `runtime` に変更した
- `flyway gradle plugin` のバージョンを最新版 (9.15.2) に変更した
  - 最新版に変更すると `cleanDisabled` のデフォルト値が変わった[^3]
- `gradle` のバージョンを最新版 (8.0.1) に変更した

## 結論
(おそらく) flyway gradle plugin では依存性解決の都合上解消できないっぽい?

## 結局どうしたん？
flyway の外でファイルツリーが良い感じになるように構築した

1. Amazon S3のバケットからファイルを取得する処理を追加
    - Amazon S3から取得したファイルが flyway のファイルロケーションとして正しい位置になるようにした
2. flyway のマイグレーションを実行


[^1]: https://documentation.red-gate.com/fd/locations-184127514.html
[^2]: [こんな感じのエラー](https://stackoverflow.com/q/73889019)
[^3]: https://documentation.red-gate.com/fd/release-notes-for-flyway-engine-179732572.html#:~:text=Change%20default%20of%20cleanDisabled%20to%20true.
