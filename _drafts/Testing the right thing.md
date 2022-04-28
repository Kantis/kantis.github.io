With the advent of test driven development (TDD), many were taught the red-green-refactor cycle. Write the failing (red) test, write the code to make it pass (green), then refactor.

In my experience, a lot of code written in the steps of TDD got stuck half-way. Tests got written (sometimes before the actual code), but in most cases we get stuck with tests that will only work as long as you _don't_ refactor.

## Example:

Assume we have an online checkout system. The customer puts some items in a basket, proceeds to checkout, pays and items get shipped. Typical tests might look like:

```kotlin

test("add item to basket") {
  val basket = Basket()
  val item = Item("Foo", price = 10.usd)
  basket.add(item)

  basket.items shouldContain item
}


test("Checking out a basket should deduct money from the customer") {
  val wallet = mock<Wallet>()
  val basket = basketOf { Item("Foo", price = 10.usd) }
  
  
  checkout(basket)
}


```


When does this break?

