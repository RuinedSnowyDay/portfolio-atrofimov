# Exercise 2

## Question 1: State

In order to uniquely authenticate the `User` with correct username and password, in
addition to just the `User`s we need to store *unique* associated username strings
and password strings, but the password strings need not be unique. This way, we can
complete the state of `PasswordAuthentication` as follows:

**state**\
  a set of `User`s with\
    a `username` String\
    a `password` String

## Question 2: Actions

To register a new `User`, we just have to require that the username is unique, so the
pre- and postconditions for `register` are pretty simple:

`register` (`username`: `String`, `password`: `String`): (`user`: `User`)\
  **requires** there is no `User` with `username` in the set of `User`s\
  **effects** add a new `User` with the provided `username` and `password` to the set
   of `User`s, return the new `User`

For `authenticate`, we just need to assert that the provided `username` and `password`
match the `username` and `password` of some `User` in the set of `User`s.

`authenticate` (`username`: `String`, `password`: `String`): (`user`: `User`)\
  **requires** there is a `User` with provided `username` and `password` in the set of
  `User`s\
  **effects** return the `User` with the provided `username` and `password`,
  authenticating it

## Question 3: Invariant

As mentioned in the [first question](#question-1-state), we need to store *unique*
associated `username` Strings in order to deterministically authenticate the `User`.
This way, our invariant becomes that for any `username` String, there can be at most
one `User` with that `username`. This invariant is preserved by the `register`,
because it requires that there is no `User` with the same `username` in the set of
`User`s.

Attentive reader can notice that in principle we can make this invariant weaker by
allowing multiple `User`s with the same `username`, but different `password`s. This
won't contradict the principle of the concept, because the `User` can still be
uniquely authenticated by their unique `username` and `password` pair.  However, one
can imagine a scenario when a person tries to `register` a `User` with existing
`username` and `password`, but the precondition check implies that the user can become
knowledgeable of the existing `username`-`password` pair, allowing them to impersonate
the existing `User`, which is undesirable.

## Question 4: Email confirmation

To include the functionality of email confirmation, we might want to extend the state to
include another set of `User`s that are created, but not yet confirmed. To do this,
we augment the state with another set, but now with `SecretToken` instead of `user`

**state**\
  a set of `SecretToken`s with\
    a `username` String\
    a `password` String\
\
  a set of confirmed `User`s\
    a `username` String\
    a `password` String

To use this state, we need to add two new actions instead of just `register`:

**actions**\
  `register` (`username`: String, `password`: String): (`token`: `SecretToken`)\
    **requires** there is no `User` and `SecretToken` with the provided `username` in\
      the sets of `User`s and `SecretToken`s\
    **effects** add a new `SecretToken` with the provided `username` and `password` to\
      the set of `SecretToken`s, return the new `SecretToken`\
\
  `confirm` (`token`: `SecretToken`): (`user`: `User`)\
    **requires** there is a `SecretToken` with the provided `token` in the set of\
      `SecretToken`s\
    **effects** add a new `User` with the provided `username` and `password` to the set\
      of `User`s, remove the `SecretToken` from the set of `SecretToken`s, return the\
      new `User`
