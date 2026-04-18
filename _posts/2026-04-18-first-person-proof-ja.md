---
title: "テストが知らないこと"
date: 2026-04-18
lang: ja
---

> `delete`の後に`get`を書く。書いていて、これ要るかと思う。

## 二度尋ねる

```php
$repo->delete($id);
$this->assertNull($repo->get($id));
```

`save`の後に`find`。`update`の後に`get`。書いた本人も、なぜ二度目が要るのか説明できない。

「念のため」と言います。「確認のため」と言います。

## voidの沈黙

`delete($id): void`. 削除した。何件消えたかは言わない。

SQLは知っています。`affected_rows = 1`。けれど戻り値が`void`である限り、その数は関数の中で生まれて消えます。

外にいるテストは、内側で何が起きたかを知る術を持ちません。だから別の証人を呼ぶ。`get`に「無い」と答えさせる。

間接証拠です。直接証拠ではない。

## 観察者は外にいる

テストは外から見ます。観察者です。

観察者には選択肢が二つあります。

- 内部の手応えを取り出して見えるようにする——カプセル化を壊す
- 別の方法で確かめる——テストが二重になる

TDDが小さく抱えてきた矛盾。テストが外にいる限り、内で起きたことは伝聞でしか書けない。

## 内側に証拠を持つ

Be Frameworkでは、削除は変容の一段になります。

```php
#[Be([ArticleDeleted::class])]
final readonly class ArticleDeleteRequest
{
    public function __construct(
        #[Input] public string $id,
    ) {}
}

final readonly class ArticleDeleted
{
    public function __construct(
        #[Input] string $id,
        #[Inject] ArticleRepository $repo,
    ) {
        $affectedRows = $repo->delete($id);
        $this->been->with(new DeletedContext(id: $id, affectedRows: $affectedRows->count));
    }
}
```

`Ray.MediaQuery`はインターフェースの戻り値を`int`にするだけで、`affected_rows`を直接届けます。

外には晒しません。中で、`affectedRows: 1`という事実を`$been`に書き留める。

```php
$result = $becoming(new ArticleDeleteRequest($id));

$this->assertInstanceOf(ArticleDeleted::class, $result);
$this->assertSame(1, $result->been->last()->affectedRows);
```

`get`は呼びません。オブジェクト自身が、自分が`ArticleDeleted`である根拠を持っている。

## 記録ではなく条件

ログは普通、起きたことを後から書き留めます。

`$been`は違います。証跡がなければ`ArticleDeleted`は成立しない。`affectedRows: 1`は事後の記録ではなく、存在の条件です。

外から見るかぎり、二度尋ねるしかない。
