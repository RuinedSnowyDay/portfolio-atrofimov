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

