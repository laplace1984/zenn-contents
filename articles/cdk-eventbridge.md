---
title: "EventBridge で Chatbot のメッセージを切り替える"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cdk", "aws"]
published: false
---

## TL;DR
- `AWS SES ConfigurationSet` から Slack 通知をする場合は `EventBridge` でメッセージを変換しないとダメ
- メッセージの変換をするときは `` を使うと簡単に設定できる

## 試した環境
| 環境 | バージョン |
| :-- | :-- |
| CDK | 2.189.0 |
| Node.js | 22.14.0 |
| macOS | Sonoma 14.x.x |

## 書くこと
