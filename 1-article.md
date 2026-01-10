About a year ago, I wrote about freezing time when testing Laravel's temporary storage URLs.
{% embed https://dev.to/tegos/freezing-time-testing-laravel-temporary-storage-urls-13n1 %}
Guess what? I ran into the same problem again, just from a different angle. It was a good reminder: controlling time in your tests isn't just a nice-to-have, it's essential.

## The Problem

I had this test running smoothly on my machine. But then, on CI, it failed randomly:

```php
public function test_order_item_cancel(): void
{
    $user = UserFixture::createUser();
    $this->actingAsFrontendUser($user);

    $order = OrderFixture::create($user);
    $orderItem = OrderItemFactory::new()->for($order)->for($user)->create();

    $response = $this->put(route('api-v2:order.order-items.cancel', ['uuid' => $orderItem->uuid]));

    $response->assertNoContent();

    $this->assertDatabaseHas(OrderItem::class, [
        'uuid' => $orderItem->uuid,
        'canceled_at' => Date::now(),
    ]);
}
```

Sometimes I'd get this error:

```
Failed asserting that a row in the table [order_items] matches the attributes {
    "canceled_at": "2026-01-09T10:24:52.008406Z"
}.

Found: [
    {
        "canceled_at": "2026-01-09 12:24:51"
    }
].
```

At first, I just shrugged and hit retry, like everyone does, right? 😅 But after reading [The Flaky Test Chronicles VI](https://deligoz.me/articles/the-flaky-test-chronicles-vi-the-reference/#time-randomness), I realized I needed to actually pay attention. Was this a real bug, or just a flaky test?

## Why This Happens

The problem's pretty simple: `Date::now()` gets called twice, but not at the same time.

First, when the controller sets `canceled_at`.
Then, again when the test checks the value.

Even a tiny delay, maybe just a millisecond, can make those two timestamps different. And CI is usually slower, so it happens more often there.

## The Fix

Just freeze time before you make the request:

```php

// Option 1
$this->freezeTime();
// Option 2
$now = Date::now();
Date::setTestNow($now);

$response = $this->put(route('api-v2:order.order-items.cancel', ['uuid' => $orderItem->uuid]));

$this->assertDatabaseHas(OrderItem::class, [
    'uuid' => $orderItem->uuid,
    'canceled_at' => $now,
]);
```

`$this->freezeTime()` is just a convenient wrapper around Date::setTestNow(), scoped to the test lifecycle.

Now both the controller and the test share the exact same timestamp. No more missmatches.

## Another Way

If you don't care about the exact timestamp and just want to make sure the field isn't empty, go with this:

```php

$this->assertDatabaseMissing(OrderItem::class, [
    'uuid' => $orderItem->uuid,
    'canceled_at' => null,
]);
```

## Final thoughts

If your tests depend on time, take control of it. When test passes locally but fails in CI, freeze time using `Date::setTestNow()` or `$this->freezeTime()`. Make your tests reliable by controlling what you're testing. Build it right. Keep it deterministic. Trust your tests.

## Author's Note

Thanks for sticking around!
Find me on [dev.to](https://dev.to/tegos), [linkedin](https://www.linkedin.com/in/ivan-mykhavko/), or you can check out my work on [github](https://github.com/tegos).

**Notes from real-world Laravel.**