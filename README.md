# nikkileaks

A distributed Dead Man's Switch.


**WARNING: This is highly experimental software. I provide no guarantees. Use at your own risk. You've been warned**

# Requirements

You need Oasis CLI (tested on 0.2.0, toolchain 20.18). As for now, you can
install this with `curl --proto '=https' --tlsv1.2 -sSL https://get.oasis.dev |
python - --toolchain 20.18`

# Setup

- Run `oasis chain` on your machine.
- In this directory, do `yarn test` to run tests. (Note: You may need to `yarn add -D -W @types/node`).

# What is this?

A Dead Man's Switch releases information at some future date---unless the user takes some proactive action.

Cases include whistleblowing, compliance, last will and testament, etc.

## Example ##

See `app/src/main.ts` for example usage.

Given an [Oasis](https://www.oasislabs.com/) gateway `gw`:

```ts
 const service = await Release.deploy(gw, {
    description: 'My big secret',
    message: 'I love kimchi',
    messageReleaseTime: BigInt(moment().add(2, 'minutes').unix()),
});
```

This `message` will remain confidential (i.e., no one can access it) for the next two minutes. After
two minutes, it will become public. The `description` will always be public.

Once you have your service, you can call:

`service.message()` - This will retrieve your message. If the message has not yet been released, the promise will reject and give you an error message. (This is true even if you wrote the message; see Possible Extensions, below).

`service.changeReleaseTime({newTime: unixTime})` - This will change the release
time of your message. Typically, you will probably be calling this to extend the
release time of your message (i.e., making it release later than it would
otherwise. However, you can also change the release date to be sooner. You can
even set the release date to be in the past, effectively making the message
public. Note, however, that you can**not** change the release time of an
already-released message. (See FAQ).

## Using the Oasis platform ##

This Dead Man's Switch uses the [Oasis](https://www.oasislabs.com/) platform to
provide confidential state over trustless, distributed system.

Future release dates are set by specifying a [UNIX
time](https://en.wikipedia.org/wiki/Unix_time). The Oasis smart contract returns
a trusted "block time." This time iscurl --proto '=https' --tlsv1.2 -sSL https://get.oasis.dev | python - --toolchain 20.18 possible to manipulate, but its tampering
becomes increasingly unlikely as the number of honest validators increases.
(After all, if 3/5 of the validators on the network spoof time to release your
secret, the security assumptions that undergird Oasis are already violated).

# FAQ

*Why can't I change the release time of already-released messages?*

This is for caller's own safety. Allowing the caller to update the release time
of an already-released message to be in the future may give the caller a false
sense of security. Once a message has been released, we assume it is no longer
confidential.

*I'm trying to release a PDF/.zip/video. How do I do that?*

You'll want to encrypt your file symmetrically with something like VeraCrypt.
Post the encrypted file wherever you like (e.g., IPFS). Take the symmetric key
and publish it as a message in your release. As the description, list a link to
the file (e.g., the IPFS hash), along with any other metadata.

A managed workflow is coming here soon. If you're not 100% sure what's going on
here, don't use this tool yet.

# Possible extensions

**Delegate others to extend time?**

Currently, only the release's author can extend the time at which a release becomes public.

In the future, an author could delegate other addresses to also extend the time.

This could be useful in some circumstances, for example, if the secret is being
jointly controlled by a small group of people. On the other hand, delegating
increases the attack surface area, as it increases the number of people who need
to perform operational security. Besides, a group of people who need to manage a
shared secret may do better to manage a single wallet, as this provides them
with the same functionality.

User testing can help to evaluate the risks and rewards of this possible feature.

**Let author read messages anytime?**

Currently, no one can read a message that hasn't been released yet---even the
release's author. Should authors be able to read still-secret messages? On one
hand, this allows authors to access their own confidential data. On the other
hand, it could enable new types of operational security mistakes, including
social engineering attacks. Besides, users can simply keep track of their secret
off the chain for their own personal use, making them less likely to use the
contract to manage this process (the contract should probably only be used for
releasing secrets to others**.

Again, user testing can help to evaluate the risks and rewards of this possible feature.

**Contract emits an event when a change occurs?**

There's no need to an emit an event when a message becomes public - that time is
just in the contract. However, it may be helpful to emit an event when the
release time is updated. This could allow subscribers to keep tabs on a secret.
At the same time, this updates happen semi-infrequently, so it's unclear what the usecase would be.

Again, user testing can help us determine if there's user need around this feature.

# Contributing
The smart contract is in `services/src/bin/greeter.rs`.
After modifying it, run `oasis build` to regenerate the `app/service-clients/greeter.ts` client.

Then, you can test the client with `yarn test`.

# License

See LICENSE.
