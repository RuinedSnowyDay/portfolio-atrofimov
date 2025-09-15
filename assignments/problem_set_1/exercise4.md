# Exercise 4

## URL Shortener

**concept** `URLShortener`\
**purpose** shorten long URLs to make them more concise and easier to share\
**principle**\
  User provides a long URL, and the system generates a short URL;\
  User opens the short URL, and the system redirects to the long URL;\
  Alternatively, user can provide a unique short URL suffix, and this suffix composed\
  with the domain name forms a short URL, redirecting to the long URL.\
**state**\
  a set of `Link`s with\
    a `suffix` String\
    a `longURL` String\
**actions**\
  `shorten` (`longURL`: `String`): (`shortURL`: `String`)\
    **requires** nothing\
    **effects** generates a new `Link` with the provided `longURL` and a new unique\
      `suffix` and adds it to the set of `Link`s. Returns the new `shortURL` `String`\
      by combining domain name and the generated `suffix`\
\
  `shortenSuffix` (`longURL`: `String`, `suffix`: `String`): (`shortURL`: `String`)\
    **requires** there is no `Link` with the provided `suffix` in the set of `Link`s\
    **effects** generates a new `Link` with the provided `longURL` and the provided\
      `suffix` and adds it to the set of `Link`s. Returns the new `shortURL` `String`\
      by combining domain name and the provided `suffix`\
\
  `redirect` (`shortURL`: `String`): (`longURL`: `String`)\
    **requires** there is a `Link` with the `suffix` extracted from the provided\
      `shortURL` in the set of `Link`s\
    **effecrs** returns the `longURL` of the `Link` associated with the extracted\
      `suffix` `String`

In this concept, the main invariant is that `Link`s have unique `suffix` `Strings`,
because otherwise the shortened URL would lead to ambiguous long URL. `longURL` values
need not be unique, and many short URLs can lead to the same long URL.

## Billable Hours Tracking

**concept** `HoursTracker[User, Time]`\
**purpose** track the activity of the user on different projects\
**principle**\
  Employee user starts working some project, they start their session;\
  When employee user stops working, they stop their session;\
  If employee user forgot to stop their session, they can manually enter the time when\
  they stopped working.\
**state**\
  a set of `EndedSession`s with\
    a `project` `String`\
    a `start` `Time`\
    an `end` `Time`\
    a `contributor` `User`\
  a set of `ActiveSession`s with\
    a `project` `String`\
    a `start` `Time`\
    a `contributor` `User`\

**actions**\
  `startSession` (`project`: `String`, `contributor`: `User`): (`session`: `ActiveSession`)\
    **requires** there are no `ActiveSession`s with the provided `contributor` in the\
      set of `ActiveSession`s\
    **effects** adds a new `ActiveSession` with the provided `project`,\
      `contributor`, and current time as `start` to the set of `ActiveSession`s\
\
  `endSession` (`session`: `ActiveSession`)\
    **requires** nothing\
    **effects** removes the `ActiveSession` from the set of `ActiveSession`s\
      and adds a new `EndedSession` with the associated `project`, `start`, current\
      time as `end`, and `contributor` to the set of `EndedSession`s
\
  `endSessionRetroactively` (`session`: `ActiveSession`, `end`: `Time`)\
    **requires** `end` is after `start` of the `ActiveSession` but before current time\
    **effects** removes the `ActiveSession` from the set of `ActiveSession`s\
      and adds a new `EndedSession` with the associated `project`, `start`, and\
      `contributor` and provided `end` time as `end` to the set of `EndedSession`s

Here the important invariant is that a person can't start a new session if he is in
another active session.

## Conference Room Booking

**concept** `RoomBooking[User, Time, Room]`\
**purpose** users can book a conference room for a specific time period\
**principle**\
  User books an available conference room for a specific time period;\
  User uses the room for the specified time period;\
  User can cancel a booking.

**state**\
  a set of `Booking`s with\
    a `room` `Room`\
    a `start` `Time`\
    an `end` `Time`\
    a `user` `User`\

**actions**\
  `bookRoom` (`room`: `Room`, `start`: `Time`, `end`: `Time`, `user`: `User`): (`booking`: `Booking`)\
    **requires** there is no `Booking` with the provided `room` and overlapping time\
      period\
    **effects** adds a new `Booking` with the provided `room`, `start`, `end`, and\
      `user` to the set of `Booking`s\
\
  `cancelBooking` (`booking`: `Booking`)\
    **requires** nothing\
    **effects** removes the `Booking` from the set of `Booking`s

[Back to main](main.md)
