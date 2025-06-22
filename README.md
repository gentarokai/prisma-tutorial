# Prisma Tutorial

Prismaの基本的な概念と実装方法を学習するためのリポジトリです。

## 📖 学習内容

### 1. Prismaとは
Prismaは現代的なデータベースアクセスツールキットで、以下の特徴があります。
- **Type-safe database client**: TypeScriptと完全に統合されたタイプセーフなクライアント
- **Database migrations**: データベーススキーマの変更を管理
- **Introspection**: 既存のデータベースからPrismaスキーマを生成
- **Prisma Studio**: データベースの視覚的な管理ツール

### 2. 主要コンポーネント

#### Prisma Schema
```prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int     @id @default(autoincrement())
  name  String
  email String  @unique
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  content  String?
  author   User   @relation(fields: [authorId], references: [id])
  authorId Int
}
```

#### Prisma Client
```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// Create
await prisma.user.create({
  data: {
    name: 'Alice',
    email: 'alice@example.com'
  }
})

// Read
const users = await prisma.user.findMany({
  include: {
    posts: true
  }
})

// Update
await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' }
})

// Delete
await prisma.user.delete({
  where: { id: 1 }
})
```

### 3. セットアップ手順

#### 初期設定
```bash
# Prismaのインストール
npm install prisma @prisma/client

# Prismaの初期化
npx prisma init

# スキーマからクライアント生成
npx prisma generate

# データベースマイグレーション
npx prisma migrate dev --name init
```

#### 環境変数設定
```env
# .env
DATABASE_URL="postgresql://username:password@localhost:5432/mydb"
```

### 4. 主要な機能

#### リレーション
- **1対多**: `User`と`Post`の関係
- **多対多**: 中間テーブルを使用
- **1対1**: `@unique`制約を使用

#### クエリ機能
- **フィルタリング**: `where`句による条件指定
- **ソート**: `orderBy`による並び替え
- **ページネーション**: `skip`と`take`による制限
- **集約**: `count`、`sum`、`avg`等

#### トランザクション
```typescript
await prisma.$transaction([
  prisma.user.create({ data: { name: 'Alice', email: 'alice@example.com' } }),
  prisma.post.create({ data: { title: 'Hello', authorId: 1 } })
])
```

### 5. ベストプラクティス

#### スキーマ設計
- 適切な型定義の使用
- インデックスの最適化
- 命名規則の統一

#### パフォーマンス
- N+1問題の回避（`include`や`select`の活用）
- 適切なページネーション
- コネクションプールの設定

#### セキュリティ
- 環境変数による接続情報の管理
- 入力値の検証
- SQLインジェクション対策（Prismaは自動で対応）

## 🚀 使用方法

```bash
# 依存関係のインストール
npm install

# データベースマイグレーション
npx prisma migrate dev

# Prisma Studio起動（GUI管理ツール）
npx prisma studio

# 開発サーバー起動
npm run dev
```

## 📚 参考リンク

- [Prisma公式ドキュメント](https://www.prisma.io/docs)
- [Prisma Examples](https://github.com/prisma/prisma-examples)
- [TypeScript with Prisma](https://www.prisma.io/docs/concepts/overview/what-is-prisma/prisma-in-your-stack)

## 📝 学習メモ

このリポジトリは[Zenn記事](https://zenn.dev/hayato94087/books/e9c2721ff22ac7)を参考にしたPrismaの学習記録です。基本的なCRUD操作からリレーション、トランザクションまでの実装例を含んでいます。
