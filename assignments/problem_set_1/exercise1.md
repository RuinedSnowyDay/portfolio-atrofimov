# Problem Set 1. Exercise 1

## Question 1: Invariants

The most obvious invariant is that the count of required `Item`s in a `Request` can't
be less than 0, as the purchasers shouldn't be able to buy more items than originally
requested. This invariant holds because `purchase` action requires that the `Item`
count in a `Request` is at least the count of purchased items, so the decrementing of
the request count can't lead to a negative count.

Another invariant is more tricky, but can still be observed if we consider the
constituent parts of `Request` and `Purchase`. `Request` contains the requested `Item`
and its count, but most importantly, it contains the set of `Purchase`s that should
correspond to this `Registry` and this particular `Item`. Each `Purchase` contains a
`User` purchaser, `Item` purchased, and the count of purchased items. This way, the
invariant we want to hold is that for some `Request` in a `Registry`, every `Purchase`
in the corresponding set of `Purchase`s should have the same `Item` as the `Item` in
the `Request`.

The second invariant is more important, because it ensures that the `Purchase`s are
consistent with corresponding `Request`s. Expectedly, `purchase` action is the most
affected by this invariant, and it preserves it by requiring that there is a request
with the same `Item` as the input `Item`, creating a new `Purchase` with the provided
`Item`, and adding this `Purchase` to the corresponding set of `Purchase`s in the
matching `Request`.

## Question 2: Fixing an action

The only action other than `purchase` that can affect the `Request`s and the
corresponding `Purchase`s is `removeItem`. One can imagine the following scenario:

+ Owner creates `Registry`, adds `Item` with some count, thus creating a `Request` for
  this `Item` with the given count; + Purchaser creates `Purchase` for this `Item`,
   thus adding this `Purchase` to the set of `Purchases` of the corresponding
   `Request`;
+ Owner decides that they don't need this `Item` anymore, so they remove the `Request`
  for this `Item` from the `Registry`. `Purchase` that was made by the purchaser is
  dangling, attached to this detached `Request`.
+ Owner feels mercurial and decides to add the `Item` back to the `Registry`, but when
  they do that, the new `Request` can't know about the `Purchase` that was made by the
  purchaser, so even though the `Purchase` is still valid and has the correct count,
  it is not reflected by the new `Request`.

To prevent this, we can act prohibitvely and add another precondition to the
`removeItem`: `removeItem` should also require that the set of `Purchase`s associated
with the `Request` of given `Item` is empty.

## Question 3: Inferring behavior

Yes, the `Registry` can be opened and closed repeatedly, it doesn't affect the related
pieces of state, because this opening and closing is performed by switching the `Flag`
that indicates whether the `Registry` is active or not. One can imagine scenario when
the owner satiates the requests for some set of `Item`s, closes the `Registry`
thinking that they are done, but then realizes that they can sap more gifts from the
purchaser(s), so they open the `Registry` again, add new `Item`s or increment the
count of the existing `Item`s, and allow the purchaser(s) to purchase more items.

## Question 4: Registry deletion

There is no action for deleting the `Registry`, but this wouldn't matter in practice,
because the old `Registry` can just exist as a done deal, which is accessible, but not
modifiable due to being inactive, and nothing stops the `User` from creating a new
`Registry` with new requests.

## Question 5: Queries

As an owner, we might want to know whether there is a `Request` for a given `Item`
in the `Registry`, what is the current count of a `Request` for this `Item`, and
what are the `Purchase`s for this `Request`. The owner needs to know this to be able
to determine whether they can or can't remove the `Request` for this `Item`, or
whether they will add new `Request` or just increment the count of the existing one
when performing the `addItem` action.

As a purchaser, we just want to know the number of `Item`s that are still available
for purchase, so that we satisfy the precondition of the `purchase` action. To do
this, we need to query the `count` of the `Request` for the given `Item`.

## Question 6: Hiding purchases

To make the `Purchase`s hidden for the owner, we can add another field to the
`Registry`, which is a `Flag` that indicates whether the `Purchase`s are visible or
not. This flag would then prevent any queries from the owner to see the counts
and `Purchase`s of the `Request`s.

We need to modify the `create` action to reflect the new flag\
`create` (`owner`: `User`, `visible`: `Flag`): (`registry`: `Registry`)\
  **effects** create a new `Registry` with this `owner`, `active` set to `false`,\
  `visible` set to the provided `visible` flag, and no requests

We also can add two new actions for toggling the visibility of the `Purchase`s
in the `Registry`:\
`makeVisible` (`registry`: `Registry`)\
  **requires** `registry` is hidden\
  **effects** make the `Registry` visible

`makeHidden` (`registry`: `Registry`)\
  **requires** `registry` is visible\
  **effects** make the `Registry` hidden

## Question 7: Generic types

We might want to use SKU codes for `Item`s instead of names, prices, or descriptions,
because these codes are unique by their nature, so there can be no confusion between
owner and purchaser, because the purchaser will now exactly what to purchase, given
that the owner provided the correct SKU code. Also, prices and descriptions can be
mutable, making them very inconvenient to store in the state.
