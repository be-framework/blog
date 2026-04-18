---
title: "What Tests Don't Know"
date: 2026-04-18
---

> You write `get` after `delete`. Halfway through typing, you wonder why.

## Asking Twice

```php
$repo->delete($id);
$this->assertNull($repo->get($id));
```

`find` after `save`. `get` after `update`. The person who wrote it can't quite explain why the second call is needed.

"Just to be sure," they say. "For confirmation," they say.

## The Silence of `void`

`delete($id): void`. It deleted. It doesn't say how many.

SQL knows. `affected_rows = 1`. But as long as the return is `void`, that number is born and dies inside the function.

A test sitting outside has no way to know what happened within. So it calls another witness. It asks `get` to answer "nothing."

Indirect evidence. Not direct.

## The Observer Is Outside

A test watches from outside. An observer.

The observer has two choices.

- Expose the internal feedback so it can be seen — break encapsulation
- Verify by other means — duplicate the test

The small contradiction TDD has carried. As long as the test is outside, what happened inside can only be written as hearsay.

## Evidence Held Within

In Be Framework, deletion becomes one stage of metamorphosis.

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

Declare `int` as the interface return type, and `Ray.MediaQuery` returns `affected_rows` directly.

Nothing leaks outside. Inside, the fact `affectedRows: 1` is written into `$been`.

```php
$result = $becoming(new ArticleDeleteRequest($id));

$this->assertInstanceOf(ArticleDeleted::class, $result);
$this->assertSame(1, $result->been->last()->affectedRows);
```

No `get` call. The object itself holds the grounds for being `ArticleDeleted`.

## Not a Record, a Condition

A log usually writes down what happened, after the fact.

`$been` is different. Without the trace, `ArticleDeleted` cannot exist. `affectedRows: 1` is not an after-the-fact record. It is the condition of existence.

From outside, all you can do is ask twice.
