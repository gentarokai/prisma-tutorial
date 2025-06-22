# Prisma Tutorial

Prismaの基本的な概念と実装方法を体系的に学習するためのリポジトリです。

## 📚 学習内容

### はじめに
Prismaは現代的なデータベースアクセスツールキットで、TypeScriptと完全に統合されたタイプセーフなデータベースクライアントです。このチュートリアルでは、セットアップから実際のクエリ操作まで段階的に学習していきます。

---

## 第１章 セットアップ ⚙️

### 空のプロジェクトを作成
新しいNode.jsプロジェクトの初期化から始めます。

```bash
mkdir prisma-tutorial
cd prisma-tutorial
npm init -y
```

### Prisma をインストール
Prismaとその関連パッケージをインストールします。

```bash
# Prisma CLI と Client のインストール
npm install prisma @prisma/client
npm install -D typescript ts-node @types/node

# Prismaの初期化
npx prisma init
```

### VS Code の設定
開発効率を向上させるためのVS Code拡張機能と設定。

- **Prisma拡張機能**: シンタックスハイライトとオートコンプリート
- **設定例**: `.vscode/settings.json`の推奨設定

---

## 第２章 ローカルデータベース 🗄️

### Supabase CLI でローカルデータベースを構築
ローカル開発環境でのデータベースセットアップ。

```bash
# Supabase CLIのインストール
npm install -g supabase

# ローカルSupabaseの起動
supabase init
supabase start
```

### Supabase Studio
WebベースのデータベースGUIツールでのデータ管理。

- **アクセス**: `http://localhost:54323`
- **機能**: テーブル作成、データ編集、SQL実行

### 環境変数を設定
データベース接続設定の管理。

```env
# .env
DATABASE_URL="postgresql://postgres:postgres@localhost:54322/postgres"
```

---

## 第３章 スキーマファイル 📝

### テーブル定義を記述
Prismaスキーマでのデータモデル定義。

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
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### スキーマをフォーマット
コードの可読性向上のためのフォーマット。

```bash
npx prisma format
```

### スキーマを検証
スキーマファイルの構文チェック。

```bash
npx prisma validate
```

---

## 第４章 マイグレーション 🔄

### Prisma で 1回目のマイグレーション
初期スキーマのデータベースへの適用。

```bash
npx prisma migrate dev --name init
```

### Prisma で 2回目のマイグレーション
スキーマ変更時のマイグレーション実行。

```bash
# スキーマを変更後
npx prisma migrate dev --name add_user_profile
```

---

## 第５章 Prisma Studio 🖥️

### Prisma でシードデータを作成
初期データの投入設定。

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@example.com',
      posts: {
        create: [
          {
            title: 'Hello World',
            content: 'This is my first post!',
            published: true,
          },
        ],
      },
    },
  })
  console.log({ user })
}

main()
  .catch((e) => {
    console.error(e)
    process.exit(1)
  })
  .finally(async () => {
    await prisma.$disconnect()
  })
```

```bash
# シードデータの実行
npx prisma db seed
```

### Prisma Studio
視覚的なデータベース管理ツール。

```bash
npx prisma studio
```

---

## 第６章 クエリ 🔍

### Create
新しいデータの作成操作。

```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// ユーザー作成
const user = await prisma.user.create({
  data: {
    name: 'Bob',
    email: 'bob@example.com'
  }
})

// リレーションと一緒に作成
const userWithPost = await prisma.user.create({
  data: {
    name: 'Carol',
    email: 'carol@example.com',
    posts: {
      create: [
        { title: 'My First Post', content: 'Hello World!' }
      ]
    }
  }
})
```

### Read
データの読み取り操作。

```typescript
// 全ユーザー取得
const allUsers = await prisma.user.findMany()

// 条件付き検索
const user = await prisma.user.findUnique({
  where: { email: 'alice@example.com' },
  include: { posts: true }
})

// 複雑な検索
const publishedPosts = await prisma.post.findMany({
  where: { published: true },
  include: { author: true },
  orderBy: { createdAt: 'desc' }
})
```

### Update
既存データの更新操作。

```typescript
// 単一レコード更新
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Smith' }
})

// 複数レコード更新
const updatedPosts = await prisma.post.updateMany({
  where: { published: false },
  data: { published: true }
})
```

### Delete
データの削除操作。

```typescript
// 単一レコード削除
const deletedUser = await prisma.user.delete({
  where: { id: 1 }
})

// 複数レコード削除
const deletedPosts = await prisma.post.deleteMany({
  where: { published: false }
})
```

### Transaction
複数操作の一括実行。

```typescript
// Interactive transaction
const result = await prisma.$transaction(async (prisma) => {
  const user = await prisma.user.create({
    data: { name: 'David', email: 'david@example.com' }
  })
  
  const post = await prisma.post.create({
    data: {
      title: 'Transaction Post',
      authorId: user.id
    }
  })
  
  return { user, post }
})

// Sequential operations
const [deletedPosts, deletedUsers] = await prisma.$transaction([
  prisma.post.deleteMany({ where: { published: false } }),
  prisma.user.deleteMany({ where: { posts: { none: {} } } })
])
```

---

## 🚀 使用方法

```bash
# 依存関係のインストール
npm install

# データベースマイグレーション
npx prisma migrate dev

# Prisma Clientの生成
npx prisma generate

# シードデータの投入
npx prisma db seed

# Prisma Studio起動
npx prisma studio

# 開発サーバー起動
npm run dev
```

## 📚 参考リンク

- [Prisma公式ドキュメント](https://www.prisma.io/docs)
- [Supabase公式ドキュメント](https://supabase.com/docs)
- [学習元記事](https://zenn.dev/hayato94087/books/e9c2721ff22ac7)

## 📝 学習メモ

このリポジトリは段階的にPrismaの概念と実装を学習できるよう構成されています。各章で学んだ内容を実際のコードで確認しながら進めることで、Prismaの基本的な使い方から応用まで習得できます。
