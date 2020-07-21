# Kickoff

After speaking with the OTRv4 team in regards to ChatSecure, there are some
recommendations and things to follow for implementing OTRv4 there.

## Ideas

### Deciding between Swift and Objective-C

As it can be seen in the code from the [OTR Kit on ChatSecure](https://github.com/ChatSecure/OTRKit),
the library is a wrapper of the C code, and it is written on Objective C.

Due to this, it uses wrappers to make the calls to the library itself and
to the [very different dependencies](https://github.com/ChatSecure/OTRKit/tree/master/scripts).
For OTRv4, we will need to create other scripts (for libgoldilocks and libsodium),
or use Objective C versions of them.

Notice that for libsodium:

* There is an [Objective C library](https://github.com/gabriel/NAChloride)
* There is a [Swift library](https://github.com/jedisct1/swift-sodium): this
library might be much more reliable than the prior one.

For libgcrypt:

We can try using the same script, or use this [library](https://github.com/x2on/libgcrypt-for-ios).

In order to implement it on ChatSecure, we, therefore, need to:

* Decide between Swift and Objective C: while the first is the [mostly recommeded](https://www.infoworld.com/article/2920333/swift-vs-objective-c-10-reasons-the-future-favors-swift.html), some C headers might not work with
it. This has to be tested.
* Write wrappers from the C code to either Swift or Objective C. This
  [tutorial](https://medium.com/shopify-mobile/wrapping-a-c-library-in-swift-part-1-6dd240070cef)
  gives the basic insight for it. [This](https://medium.com/@cecilia.humlelu/using-c-c-and-objective-c-frameworks-in-swift-apps-6a60e5f71c36) might be interesting to follow as well.

### Analyse the build scripts for the OTR kit


