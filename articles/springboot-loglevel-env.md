---
title: "環境変数で SpringBoot のログレベルを変更する"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SpringBoot", "コンテナ"]
published: true
---

## TL;DR
- SpringBoot のログレベルはアプリケーションプロパティファイル (`application.yml` / `application.properties`) でも設定できる
  - `logback-spring.xml`[^1] は要らない
- アプリケーションプロパティファイルの値はキーを一定のルールで変換すれば、環境変数で上書きできる
- ログレベルも環境変数で上書きできる

## 試した環境
1. SpringBoot 2.7.5
2. Gradle 7.5.1
3. Java 11 (Gradle用)
4. Kotlin 1.7.10
5. logback

## 始めに
SpringBoot のログ設定の **一部**[^2] はアプリケーションプロパティファイル (`application.yml` / `application.properties`) で設定できます
よく使うログ設定と言えばログレベルで、ログレベルはアプリケーションプロパティファイルで設定できます[^3]

### 設定例
```yaml:application.yml
logging:
  level:
    # 全体に影響するログレベル
    ROOT: INFO
    # 個別のパッケージのログレベル
    org.springframework.boot: INFO
```

## アプリケーションプロパティファイルの値を環境変数で上書きする
SpringBoot の設定はアプリケーションプロパティファイルを用いて設定できますが、それ以外の方法でも設定することが可能[^4]です
特にコンテナ環境だとよく使用するのは `環境変数` だと思います

注釈のリンク先に優先順が記載されていて、環境変数の方が後なのでアプリケーションプロパティファイルよりも優先することが可能です (要するに環境変数で上書きできる)

環境変数で上書きする場合は設定のキーをこのように変換します (YAMLの場合 / properties の場合は3以降が不要)
1. 全部大文字にする
2. `.` や `-` を `_` に置換する
3. `:` を `_` にする
4. 一行にまとめる

### 環境変数の上書きの例
1. この設定を `default` に上書きしたい
```yaml:application.yml
spring:
  profiles:
    active: development
```
2. 以下の環境変数をセットする
```shell
export SPRING_PROFILES_ACTIVE=default
```
3. アプリケーションを実行する

## ログレベルを環境変数で上書きするやり方
アプリケーションプロパティファイルの値を環境変数で上書きできるということは、ログレベルの設定も上書きできます[^5]

### 例： `org.springframework.web` パッケージのログレベルを `DEBUG` に変更したい
1. 以下の環境変数をセットする
```shell
export LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG
```
2. アプリケーションを実行する

ね？簡単でしょ？

## この方法のユースケース
1. 特定の環境の調査目的でログを詳細化したい
    - アプリケーションプロパティファイルを変更しなくても良いので、環境変数を追加してコンテナを再起動すればログレベルを切り替えられる
2. 環境ごとにアプリケーションプロパティファイルを用意したくない

[^1]: `logback.xml` でも良いですが、 `-spring` サフィックスが推奨されています https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-1.3-Release-Notes#spring-specific-configuration
[^2]: デコレータとかの細かい設定はできない
[^3]: https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.core.logging.level
[^4]: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config
[^5]: アプリケーション設定ファイルに書いてなくても変更できます (`logback-spring.xml` がいる場合は未検証)
