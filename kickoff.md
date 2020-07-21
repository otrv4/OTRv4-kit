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

As it was noted, there are some build scripts for the dependencies of OTR.
There can be found [here](https://github.com/ChatSecure/OTRKit/tree/master/scripts).
They should work the same.

### Create an API

Basically, the way it used to work is as follows (there should be a main file,
like [this](https://github.com/ChatSecure/OTRKit/blob/master/OTRKit/OTRKit.h):

```
@protocol OTRKitDelegate <NSObject>
@required
/**
 *  This method **MUST** be implemented or OTR will not work. All outgoing messages
 *  should be sent first through OTRKit encodeMessage and then passed from this delegate
 *  to the appropriate chat protocol manager to send the actual message.
 *
 *  @param otrKit      reference to shared instance
 *  @param message     message to be sent over the network. may contain ciphertext.
 *  @param recipient   intended recipient of the message
 *  @param accountName your local account name
 *  @param protocol    protocol for account name such as "xmpp"
 *  @param tag optional tag to attached to message. Only used locally.
 */
- (void) otrKit:(OTRKit*)otrKit
  injectMessage:(NSString*)message
       username:(NSString*)username
    accountName:(NSString*)accountName
       protocol:(NSString*)protocol
            tag:(id)tag;

/**
 *  All outgoing messages should be sent to the OTRKit encodeMessage method before being
 *  sent over the network.
 *
 *  @param otrKit      reference to shared instance
 *  @param message     plaintext message
 *  @param sender      buddy who sent the message
 *  @param accountName your local account name
 *  @param protocol    protocol for account name such as "xmpp"
 *  @param tag optional tag to attach additional application-specific data to message. Only used locally.
 */
- (void) otrKit:(OTRKit*)otrKit
 encodedMessage:(NSString*)encodedMessage
       username:(NSString*)username
    accountName:(NSString*)accountName
       protocol:(NSString*)protocol
            tag:(id)tag
          error:(NSError*)error;


/**
 *  All incoming messages should be sent to the OTRKit decodeMessage method before being
 *  processed by your application. You should only display the messages coming from this delegate method.
 *
 *  @param otrKit      reference to shared instance
 *  @param message     plaintext message
 *  @param tlvs        OTRTLV values that may be present.
 *  @param sender      buddy who sent the message
 *  @param accountName your local account name
 *  @param protocol    protocol for account name such as "xmpp"
 *  @param tag optional tag to attach additional application-specific data to message. Only used locally.
 */
- (void) otrKit:(OTRKit*)otrKit
 decodedMessage:(NSString*)decodedMessage
           tlvs:(NSArray*)tlvs
       username:(NSString*)username
    accountName:(NSString*)accountName
       protocol:(NSString*)protocol
            tag:(id)tag;

/**
 *  When the encryption status changes this method is called
 *
 *  @param otrKit      reference to shared instance
 *  @param messageState plaintext, encrypted or finished
 *  @param username     buddy whose state has changed
 *  @param accountName your local account name
 *  @param protocol    protocol for account name such as "xmpp"
 */
- (void)    otrKit:(OTRKit*)otrKit
updateMessageState:(OTRKitMessageState)messageState
          username:(NSString*)username
       accountName:(NSString*)accountName
          protocol:(NSString*)protocol;
...
```

To encode a message:

```
/**
 * Encodes a message and optional array of OTRTLVs, splits it into fragments,
 * then injects the encoded data via the injectMessage: delegate method.
 * @param message The message to be encoded
 * @param tlvs Array of OTRTLVs, the data length of each TLV must be smaller than UINT16_MAX or it will be ignored.
 * @param recipient The intended recipient of the message
 * @param accountName Your account name
 * @param protocol the protocol of accountName, such as @"xmpp"
 *  @param tag optional tag to attach additional application-specific data to message. Only used locally.
 */
- (void)encodeMessage:(NSString*)message
                 tlvs:(NSArray*)tlvs
             username:(NSString*)username
          accountName:(NSString*)accountName
             protocol:(NSString*)protocol
                  tag:(id)tag;
```

To decode a message:

```
/**
 *  All messages should be sent through here before being processed by your program.
 *
 *  @param message     Encoded or plaintext incoming message
 *  @param sender      account name of buddy who sent the message
 *  @param accountName your account name
 *  @param protocol    the protocol of accountName, such as @"xmpp"
 *  @param tag optional tag to attach additional application-specific data to message. Only used locally.
 */
- (void)decodeMessage:(NSString*)message
             username:(NSString*)username
          accountName:(NSString*)accountName
             protocol:(NSString*)protocol
                  tag:(id)tag;
```



