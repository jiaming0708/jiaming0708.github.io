---
title: Supabase 初體驗
date: 2025-09-04 17:05:15
updated: 2025-09-04 17:05:15
categories:
- Database
tags:
- Database
- Saas
thumbnail: /images/supabase-logo-wordmark--light.svg
---

最近因為一個案子，開始碰觸到了 Supabase 這個資料庫，接下來想要來介紹 Supabase 我所用到的部分。

<!-- more -->

Supabase 是一個基於 Postgres 資料庫所延伸的 saas 服務，在沒有付費的情況下有限度的使用，蠻適合當做 side project 使用，當然如果是一個正式的服務，本身也是很適合，只是要記得付費XD

註冊非常的簡單，可以透過 Github 做 OAuth 就可以開始享受服務。

> 如果服務一段時間內都沒有用到，則會先被官方所暫停，主要是避免他們的資源浪費吧。

## 本機開發環境

那如果只是要先嘗試看看的話，也有提供 cli 讓你在本機的環境中建置起來，以下介紹一些 cli 的作法

> 如果要在本機開發，官方是採用 container 的方式將環境建置起來，要記得先準備好 Docker 或是 Podman 這類的工具

首先是要初始化，並且起環境，可以參考 [官方文件](https://supabase.com/docs/guides/local-development)

```shell
npx supabase init
npx supabase start
```

第一次執行的話，因為包含 image pull 所以會花比較多時間，完成的話會看到類似這樣的資訊

```shell
supabase local development setup is running.

         API URL: http://127.0.0.1:54321
     GraphQL URL: http://127.0.0.1:54321/graphql/v1
          DB URL: postgresql://postgres:postgres@127.0.0.1:54322/postgres
      Studio URL: http://127.0.0.1:54323
    Inbucket URL: http://127.0.0.1:54324
      JWT secret: super-secret-jwt-token-with-at-least-32-characters-long
        anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0
service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImV4cCI6MTk4MzgxMjk5Nn0.EGIM96RAZx35lJzdJsyH-qQwv8Hdp7fsn3W0YpN81IU
```

那就可以透過 http://127.0.0.1:54323 來看到整個本機環境，其實就跟在官網所可以操作的介面是一樣的。

## 前端操作資料庫

朋友介紹這個資料庫給我的時候特別強調，可以不用後端主機，前端就可以直接連接資料庫，但我第一個反應是安全性(?)，這段等等來看，先看如何在前端直接操作資料庫。

有一個表叫做 `articles`

```sql
CREATE TABLE IF NOT EXISTS public.articles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(200) NOT NULL,
    slug VARCHAR(200) NOT NULL UNIQUE,
    content TEXT NOT NULL,
    excerpt TEXT,
    featured_image_url TEXT,
    status VARCHAR(20) DEFAULT 'draft',
    published_at TIMESTAMPTZ,
    author_id UUID REFERENCES auth.users(id),
    category_id UUID REFERENCES public.article_categories(id),
    tags TEXT[] DEFAULT '{}',
    view_count INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

需要根據 `文章 id` 和 `作者 id` 取得資料，可以採用以下的寫法。

```typescript
const { data } = await supabase
      .from("articles")
      .select("status, published_at")
      .eq("id", id)
      .eq("author_id", userId)
      .single();
```

前端只要透過這樣的寫法，就可以取得 `articles` 的資料，可以省下寫後端 API 的時間。
有些比較複雜的邏輯，可以採用 RPC 也就是 db function 的概念，等於取代了 API function。

## 安全性控管 (RLS)

前面有提到前端直接呼叫，那就會有一些寫入資料的安全性問題，你的服務可以寫入資料通常也是有登入功能，所以我們可以透過登入的使用者來進行權限的限縮。
這個權限的設定是根據 `table`，以下是一些設定的條件，例如所有人都可以 select，作者可以更新自己的資料，admin 就沒有限制。

```sql
CREATE POLICY "所有人可以查看已發布的文章" ON public.articles
    FOR SELECT TO anon, authenticated
    USING (status = 'published');

CREATE POLICY "管理員可以查看所有文章" ON public.articles
    FOR SELECT TO authenticated
    USING (public.is_admin());

CREATE POLICY "管理員可以管理文章" ON public.articles
    FOR ALL TO authenticated
    USING (public.is_admin());

CREATE POLICY "作者可以查看自己的文章" ON public.articles
    FOR SELECT TO authenticated
    USING (author_id = auth.uid());

CREATE POLICY "作者可以更新自己的草稿文章" ON public.articles
    FOR UPDATE TO authenticated
    USING (author_id = auth.uid() AND status = 'draft');
```

可以去呼叫方法來作檢查，這樣就不會被限制在只有一張表的資料，而是可以有一些邏輯。

```sql
CREATE FUNCTION public.is_admin()
RETURNS BOOLEAN AS $$
  SELECT EXISTS (
    SELECT 1 FROM admin_users
    WHERE id = auth.uid() AND is_active = true
  );
$$ LANGUAGE sql SECURITY DEFINER;
```

## 結論

當初用到這套是因為 Netlify 有和 Supabase 整合，因此就使用了這套，市面上也有其他類似的服務可以使用。這種服務出來大幅的降低營運的成本，以往要 *前端+後端+db* 三套式才能架一個服務起來，現在只要 *前端+db* 即可，對於維運上來說，甚至是開發上來說都是輕鬆許多。
