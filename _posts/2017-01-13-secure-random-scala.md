---
layout: post
title: Secure alphanumeric random in Scala
---

Random values in Scala are usually generated using the `scala.util.Random` class. This implementation may be fine for some use cases, but it is definitely not recommended for security-critical purposes. If you need to generate a unique token for your API, to generate a unique URL for your users or any other use case that require a [CSPRNG](https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator), you have to resort to Java's `SecureRandom`.

Here is a simple Scala object to generate alphanumeric strings. It is based on the implementation taken from [here](http://stackoverflow.com/a/41156/848330):

```
import java.math.BigInteger
import java.security.SecureRandom

object RandomUtil {
  private val random = new SecureRandom()

  def alphanumeric(nrChars: Int = 24): String = {
    new BigInteger(nrChars * 5, random).toString(32)
  }
}
```

Note that `new SecureRandom()` is preferable over `SecureRandom.getInstanceStrong` because it uses `/dev/urandom` instead of `/dev/random`. The former is non-blocking and [equally secure](https://tersesystems.com/2015/12/17/the-right-way-to-use-securerandom).
