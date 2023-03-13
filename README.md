# ZIO Cheat Sheet

- This is based on [ZIO](https://github.com/zio/zio) 2.0.X (in particular 2.0.10).
- For simplicity, ZIO environment has been omitted but all the functions also work with the form `ZIO[R, E, A]`.
- Function arguments are usually by name, but that has (mostly) been ignored for simplicity.
- For many functions there are several (unmentioned) related functions that are conceptually similar but differ in some detail.
- Important ZIO types other than the functional effect type `ZIO[R, E, A]` have been left out. For example: `ZStream[R, E, A]`, `ZLayer[RIn, E, ROut]`, `Fiber[E, A]` and `Ref[A]`.

## Aliases

| Alias        | Full Form                |
| ------------ | ------------------------ |
| `UIO[A]`     | `ZIO[Any, Nothing, A]`   |
| `IO[E, A]`   | `ZIO[Any, E, A]`         |
| `Task[A]`    | `ZIO[Any, Throwable, A]` |
| `RIO[R, A]`  | `ZIO[R, Throwable, A]`   |
| `URIO[R, A]` | `ZIO[R, Nothing, A]`     |

## Creating effects

| Name                                     | Given                                                                                   | To                                |
| ---------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------- |
| ZIO.succeed                              | `=> A`                                                                                  | `IO[Nothing, A]`                  |
| ZIO.fail                                 | `=> E`                                                                                  | `IO[E, Nothing]`                  |
| ZIO.interrupt                            |                                                                                         | `IO[Nothing, Nothing]`                  |
| ZIO.die                                  | `Throwable`                                                                             | `IO[Nothing, Nothing]`            |
| ZIO.attempt                              | `=> A`                                                                                  | `IO[Throwable, A]`                |
| ZIO.async                                | `((IO[E, A] => Unit) => Unit)` <br> FiberId                                             | `IO[E, A]`                        |
| ZIO.fromEither                           | `Either[E, A]`                                                                          | `IO[E, A]`                        |
| ZIO.left                                 | `A`                                                                                     | `IO[Nothing, Either[A, Nothing]]` |
| ZIO.right                                | `A`                                                                                     | `IO[Nothing, Either[Nothing, A]]` |
| ZIO.fromFiber                            | `Fiber[E, A]`                                                                           | `IO[E, A]`                        |
| ZIO.fromFuture                           | `ExecutionContext => Future[A]`                                                         | `IO[Throwable, A]`                |
| ZIO.fromOption                           | `Option[A]`                                                                             | `IO[Option[Nothing], A]`                     |
| ZIO.none                                 |                                                                                         | `IO[Nothing, Option[Nothing]]`    |
| ZIO.some                                 | `A`                                                                                     | `IO[Nothing, Option[A]]`          |
| ZIO.fromTry                              | `Try[A]`                                                                                | `IO[Throwable, A]`                |
| ZIO.acquireReleaseWith                   | `IO[E, A]` (acquire) <br> `A => IO[Nothing, Any]` (release) <br> `A => IO[E, B]` (use)  | `IO[E, B]`                        |
| ZIO.when                                 | `Boolean` <br> `IO[E, A]`                                                               | `IO[E, Option[A]]`                      
| ZIO.whenZIO                              | `IO[E, Boolean]` <br> `IO[E, A]`                                                        | `IO[E, Option[A]]`                     |
| ZIO.whenCase                             | `A` <br> `PartialFunction[A, IO[E, B]]`                                                 | `IO[E, Option[B]]`                     |
| ZIO.whenCaseZIO                          | `IO[E, A]` <br> `PartialFunction[A, IO[E, B]]`                                          | `IO[E, Option[B]]`                     |

## Transforming effects

| Name               | From                     | Given                                   | To                         |
| ------------------ | ------------------------ | --------------------------------------- | -------------------------- |
| map                | `IO[E, A]`               | `A => B`                                | `IO[E, B]`                 |
| as                 | `IO[E, A]`               | `B`                                     | `IO[E, B]`                 |
| orElseFail         | `IO[E, A]`               | `E2`                                    | `IO[E2, A]`                |
| unit               | `IO[E, A]`               |                                         | `IO[E, Unit]`              |
| flatMap            | `IO[E, A]`               | `A => IO[E1, B]`                        | `IO[E1, B]`                |
| flatten            | `IO[E, IO[E1, A]]`       |                                         | `IO[E1, A]`                |
| mapBoth            | `IO[E, A]`               | `E => E2`<br>`A => B`                   | `IO[E2, B]`                |
| mapError           | `IO[E, A]`               | `E => E2`                               | `IO[E2, A]`                |
| mapErrorCause      | `IO[E, A]`               | `Cause[E] => Cause[E2]`                 | `IO[E2, A]`                |
| flatMapError       | `IO[E, A]`               | `E => IO[Nothing, E2]`                  | `IO[E2, A]`                |
| sandbox            | `IO[E, A]`               |                                         | `IO[Cause[E], A]`          |
| flip               | `IO[E, A]`               |                                         | `IO[A, E]`                 |
| tap                | `IO[E, A]`               | `A => IO[E1, _]`                        | `IO[E1, A]`                |
| tapBoth            | `IO[E, A]`               | `E => IO[E1, _]`<br>`A => IO[E1, _]`    | `IO[E1, A]`                |
| tapError           | `IO[E, A]`               | `E => IO[E1, _]`                        | `IO[E1, A]`                |
| absolve            | `IO[E, Either[E, A]]`    |                                         | `IO[E, A]`                 |
| some               | `IO[Nothing, Option[A]]` |                                         | `IO[Option[E], A]`         |
| head               | `IO[E, List[A]]`         |                                         | `IO[Option[E], A]`         |
| toFuture           | `IO[Throwable, A]`       |                                         | `IO[Nothing, Future[A]]`   |
| filterOrDie        | `IO[E, A]`               | `A => Boolean`<br>`Throwable`           | `IO[E, A]`                 |
| filterOrDieMessage | `IO[E, A]`               | `A => Boolean`<br>`String`              | `IO[E, A]`                 |
| filterOrElse       | `IO[E, A]`               | `A => Boolean`<br>`A => ZIO[R2, E2, B]` | `ZIO[R2, E2, B]`           |
| filterOrFail       | `IO[E, A]`               | `A => Boolean`<br>`E2`                  | `IO[E2, A]`                |


## Recover from errors

| Name          | From       | Given                                       | To                          |
| ------------- | ---------- | ------------------------------------------- | --------------------------- |
| either        | `IO[E, A]` |                                             | `IO[Nothing, Either[E, A]]` |
| option        | `IO[E, A]` |                                             | `IO[Nothing, Option[A]]`    |
| ignore        | `IO[E, A]` |                                             | `IO[Nothing, Unit]`         |
| exit          | `IO[E, A]` |                                             | `IO[Nothing, Exit[E, A]]`   |
| `<>` (orElse) | `IO[E, A]` | `IO[E2, A1]`                                | `IO[E2, A1]`                |
| orElseEither  | `IO[E, A]` | `IO[E2, B]`                                 | `IO[E2, Either[A, B]]`      |
| fold          | `IO[E, A]` | `E => B`<br>`A => B`                        | `IO[Nothing, B]`            |
| foldZIO       | `IO[E, A]` | `E => IO[E2, B]`<br>`A => IO[E2, B]`        | `IO[E2, B]`                 |
| foldCauseZIO  | `IO[E, A]` | `Cause[E] => IO[E2, B]`<br>`A => IO[E2, B]` | `IO[E2, B]`                 |
| catchAll      | `IO[E, A]` | `E => IO[E2, A1]`                           | `IO[E2, A1]`                |
| catchAllCause | `IO[E, A]` | `Cause[E] => IO[E2, A1]`                    | `IO[E2, A1]`                |
| catchSome     | `IO[E, A]` | `PartialFunction[E, IO[E1, A1]]`            | `IO[E1, A1]`                |
| retry         | `IO[E, A]` | `Schedule[E, S]`                            | `IO[E, A]`                  |
| eventually    | `IO[E, A]` |                                             | `IO[Nothing, A]`            |

## Terminate fiber with errors

| Name              | From               | Given                                           | To                          |
| ----------------- | ------------------ | ----------------------------------------------- | --------------------------- |
| orDie             | `IO[Throwable, A]` |                                                 | `IO[Nothing, A]`            |
| orDieWith         | `IO[E, A]`         | `E => Throwable`                                | `IO[Nothing, A]`            |
| refineOrDie       | `IO[Throwable, A]` | `PartialFunction[Throwable, E1]`                | `IO[E1, A]`                 |
| refineOrDieWith   | `IO[E, A]`         | `PartialFunction[E, E1]`<br> `E => Throwable`   | `IO[E1, A]`                 |


## Combining effects + parallelism

| Name                | From       | Given                                            | To                               |
| ------------------- | ---------- | ------------------------------------------------ | -------------------------------- |
| ZIO.foldLeft        |            | `Iterable[A]` <br> `S` <br> `(S, A) => IO[E, S]` | `IO[E, S]`                       |
| ZIO.foreach         |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 |
| ZIO.foreachPar      |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 |
| ZIO.forkAll         |            | `Iterable[IO[E, A]]`                             | `IO[Nothing, Fiber[E, List[A]]]` |
| fork                | `IO[E, A]` |                                                  | `IO[Nothing, Runtime[E, A]]`     |
| `<*>` (zip)         | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 |
| `*>` (zipRight)     | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      |
| `<*` (zipLeft)      | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      |
| `<&>` (zipPar)      | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 |
| `&>` (zipParRight)  | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      |
| `<&` (zipParLeft)   | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      |
| race                | `IO[E, A]` | `IO[E1, A1]`                                     | `IO[E1, A1]`                     |
| raceAll             | `IO[E, A]` | `Iterable[IO[E1, A1]]`                           | `IO[E1, A1]`                     |
| raceEither          | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, Either[A, B]]`           |

## Finalizers

| Name          | From       | Given                      | To         |
| ------------- | ---------- | -------------------------- | ---------- |
| ensuring      | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` |
| onError       | `IO[E, A]` | `Cause[E] => UIO[_]`       | `IO[E, A]` |
| onInterrupt   | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` |
| onTermination | `IO[E, A]` | `Cause[Nothing] => UIO[_]` | `IO[E, A]` |

## Timing

| Name      | From       | Given            | To                             |
| --------- | ---------- | ---------------- | ------------------------------ |
| ZIO.never |            |                  | `IO[Nothing, Nothing]`         |
| ZIO.sleep |            | `Duration`       | `IO[Nothing, Unit]`            |
| delay     | `IO[E, A]` | `Duration`       | `IO[E, A]`                     |
| timeout   | `IO[E, A]` | `Duration`       | `IO[E, Option[A]]`             |
| timed     | `IO[E, A]` |                  | `IO[E, (Duration, A)]`         |
| forever   | `IO[E, A]` |                  | `IO[E, Nothing]`               |
| repeat    | `IO[E, A]` | `Schedule[A, B]` | `IO[E, B]`                     |
