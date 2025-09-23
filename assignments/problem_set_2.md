# Problem Set 2

## Concept Questions

1. We need contexts in `NonceGeneration` concept, because while we require that in
   one context there are no repeating `String`s, we don't want this to affect other
   contexts. For example, if for a given user we want to create multiple access
   tokens, we don't want any repetition between them, but they still can overlap with
   access tokens of other users. This way, in the context of `UrlShortening` concept,
   the short URL base becomes a context, as different bases can be combined with the
   same suffix and ultimately redirect to different long URLs.
2. `NonceGeneration` concept has to store a set of used `String`s, because if it
   doesn't do that, then, with some degree of probability, the `generate` action will
   produce a `String` that it has already produced for this context, and the concept
   won't be able to tell if the produced `String` is unique or not. By keeping track
   of all unique `String`s produced for contexts, the concept can first check if the
   generated `String` is unique or not, and only then add it to the set of used
   `String`s.

   Simplest implementation of `NonceGeneration` uses a separate counter for each
   context. When `generate` action is called with a given context, the counter of
   that context is returned (initialized to 0 if not present) as `String`, and then
   the counter is incremented by 1. This way, due to
   [Peano axioms](https://en.wikipedia.org/wiki/Peano_axioms), `generate` action will
   always produce new, unique `String` corresponding to the new counter value. One
   can then write an abstraction function from the counter value $n$ to the set of
   unique generated `String`s as
   $$
   \text{AF}(n) = \{i | i \in \mathbb{N}, i < n\}.
   $$
3. From the perspective of user, the advantage of this approach is obvious: they don't
   need to type long unintuive nonce and can use common (or at least typeable) word
   to get the combination of URL base and suffix that leads them to the desired long
   URL. Howewer, the user can over-rely on the shortness of the suffix and resert
   to remembering it rather than writing it down somewhere, which can lead to
   frustration.

   To realize this idea, we can add a static set of word `String`s that can be used as
   suffixes, and when the user tries to generate a nonce, the concept randomly chooses
   one of the words from the set and then checks if it is already used. If it is,
   the concept randomly chooses another word from the set and checks again. This
   procedure is repeated until the concept finds a word that is not yet used in a
   given context. This word is then returned as the nonce and added to the set of
   used `String`s for this context.

## Synchronization Questions

1. In the description of any concept or sync, we are trying to be as abstract and
   concise as possible. Sync `generate`'s purpose is to produce a unique nonce for a
   given context, which is, as mentioned previously, a short URL base. We don't need
   information abouth the target URL to produce a nonce, so it is not included in the
   arguments. However, both target URL and short URL base become relevant in the
   `register` sync, so they both are included in the arguments.

2. We don't use the convention of omitting the names of the arguments or results if
   they are the same as the names of the parameters, because sometimes we want to
   chain actions together and use output name different from the next input name,
   and also sometimes the type of the argument is abstract, so we have to use the
   concrete value inside the invocation of the action.

3. `Request` is not included in the `setExpiry` sync, because it happens without any
   additional input from the user side and is conditioned only on the previous
   `register` sync that yields the `shortUrl`. This way, number of parameters is
   reduced and excessive conditions are avoided.

4. To remove the support of alternate domain names, we can remove the `shortUrlBase`
   argument from `generate` and `register` syncs, and implicitly use the default
   base domain (`bit.ly` or whatever is actually used) instead.

5. The sync that makes the shortened URL invalid after a certain amount of time should
   be conditioned on the **system** `expireResource` action, because it happens
   exactly when the resource (short URL in out case) is expired. When this happens,
   we want to remove the short URL and associated target URL from the set of
   `Shortening`s in the state of `UrlShortening` concept.\
   \
   **sync** `expire`\
   **when** `ExpiringResource.expireResource` () : (`shortUrl` : `String`)\
   **then** `UrlShortening.delete` (`shortUrl`)

## Extending the Design

1. To provide analytics about the usage of the certain shortened URL, and to make it
   private to the user who created it, we need to add two new concepts: `Ownership`
   and `UseAnalytics`.\
   \
   **concept** `Ownership` [`User`, `Item`]\
   **purpose** track which items are owned by which users\
   **principle** `User` can add an item and then be authorized to use it if they are\
      the owner of the item\
   **state**\
      a set of `Ownership`s with\
        an `owner` `User`\
        an `item` `Item`\
   **actions**\
      `addOwnership` (`user`: `User`, `item`: `Item`)\
         **requires** given `Item` is not owned by any `User`\
         **effects** adds a new `Ownership` with the provided `user` and `item` to\
           the set of `Ownership`s\
      `authorize` (`user`: `User`, `item`: `Item`)\
         **requires** a `user`-`item` pair is in the set of `Ownership`s\
         **effect** authorizes the `user` to use the `item`\
\
   **concept** `UseAnalytics` [`Item`]\
   **purpose** track how many times the certain item was used\
   **principle** when an item is used, its usage count is incremented and can later\
      be queried\
   **state**\
      a set of `ItemUsage`s with\
        an `item` `Item`\
        a `count` `Number`\
   **actions**\
      `addItem` (`item`: `Item`)\
         **requires** nothing\
         **effect** adds a new `ItemUsage` with the provided `item` and `count` 0 to\
            the set of `ItemUsage`s\
      `incrementUsage` (`item`: `Item`)\
         **requires** `item` is in the set of `ItemUsage`s\
         **effect** increments the usage `count` of the `item` in the set of\
            `ItemUsage`s by 1.\
      **query** `getUsage` (`item`: `Item`): (`count`: `Number`)\
         **requires** there is a `ItemUsage` with the provided `item` in the set of
            `ItemUsage`s
         **effect** returns the usage `count` of the `item` in the set of `ItemUsage`s
2. When shortening is created by the user, we want to add this `user`-`shortUrl` pair
   to the set of `Ownership`s and add this `shortUrl` to the set of `ItemUsage`s to
   keep track of the usage of this shortened URL\
   \
   **sync** `startAnalytics`\
   **when**\
      `Request.shortenUrl` (`user`)\
      `UrlShortening.register` (): (`shortUrl`)\
   **then**\
      `Ownership.addOwnership` (`user`, `shortUrl`)\
      `UseAnalytics.addItem` (`shortUrl`)\
   \
   When shortened URL is used, we want to increment the usage count of this shortened
   URL\
   \
   **sync** `incrementUsage`\
   **when** `UrlShortening.lookup` (`shortUrl`)\
   **then** `UseAnalytics.incrementUsage` (`shortUrl`)\
   \
   Finally, when a user requests the usage of a shortened URL, we need to check if
   they are the owner of the shortened URL, and if they are, then we return the usage
   count\
   \
   **sync** `getUsage`\
   **when**\
      `Request.getUsage` (`user`, `shortUrl`)\
      `Ownership.authorize` (`user`, `shortUrl`)\
   **then** `UseAnalytics.getUsage` (`shortUrl`)
3. Let's go through each feature one by one:
   - Adding the functionality of allowing users to choose their own short URL is
     straightforward, we need to add a new action to the `NonceGeneration`, called
     `addString`, which will add a new `String` to the set of used `String`s for a
      given context. We then can add a sync that for a request that actually does
      provide a suffix and URL base, and then adds this suffix to the set of used
      `String`s for this context.
   - Using words a nonces is even more straightforward, we just need to statically
     keep the set of words and use it to generate a random unique word for a given
     context in the `NonceGeneration` concept. No other changes to syncs and concepts
     are needed.
   - Getting analytics about the usage of a target URL would require changes in two
     syncs; first, in `startAnalytics` we need to add the ownership of the target URl,
     not the shortened URL, and also to start the analytics for the target URL. Then,
     in `incrementUsage` we need to increment the usage count for the target URL,
     not the shortened URL.
   - The most obvious way to make short URLs that are not easily guessed is to add
     the ability to set minimum length of the nonce. Then, we can add new action to
     the `NonceGeneration` concept, e.g. `generateMinLength`, which generates a nonce
     with a length at least the provided minimum length and which is unique for the
     given context. Other syncs should remain the same.
   - If the creator of short URL isn't registered as user, it becomes unbearably
     complicated to verify the ownership of the short URL, probably through IP address
     or some other implicit unique identifier. This would require us to add new
     concepts and syncs, so it's much more desirable to have a standard user
     authentication concept in the system.
