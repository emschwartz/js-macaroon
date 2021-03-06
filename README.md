# js-macaroon
> Better than cookies! :cookie:

[Macaroons](http://theory.stanford.edu/~ataly/Papers/macaroons.pdf) are "Cookies with Contextual Caveats for Decentralized Authorization in the Cloud". Basically, that means bearer tokens that can be given limited scopes or expirations, and which can be limited further by users or 3rd party services themselves.

This JavaScript implementation is compatible with the [Go](http://github.com/go-macaroon/macaroon), [Python, and C ](https://github.com/rescrv/libmacaroons) implementations. It also includes functionality to interact with third party caveat dischargers implemented by the [Go macaroon bakery](http://github.com/go-macaroon-bakery/macaroon-bakery).

## Installation

`npm install --save macaroon`

## Usage

Try out the code in the [examples](./examples)!

```js
const crypto = require('crypto')
const Macaroon = require('..')

// The server creates a macaroon to give to their user
const rootKey = crypto.randomBytes(32)
const original = Macaroon.newMacaroon({
  rootKey,
  identifier: 'user12345',
  location: 'https://some-website.example'
})
const givenToUser = original.exportBinary()


// The user can add additional caveats that will be enforced by the server
// (The server can support whatever type of specific caveats you want)
const userMacaroon = Macaroon.importMacaroon(givenToUser)
userMacaroon.addFirstPartyCaveat('time < ' + new Date(Date.now() + 1000).toISOString())
const givenToThirdParty = userMacaroon.exportBinary()



// The user can give out the more limited caveat to some 3rd party service
// and the server will enforce the specified conditions when the 3rd party goes to authenticate

// This is an example of how the server might verify the macaroon's caveats
function verifier (caveat) {
  if (caveat.startsWith('time <')) {
    const expiry = Date.parse(caveat.replace('time < ', ''))
    if (expiry <= Date.now()) {
      throw new Error('macaroon is expired')
    }
  } else {
    // It's generally a good idea to reject macaroons with caveats you don't understand
    throw new Error('unsupported caveat: ' + caveat)
  }
}

const fromThirdParty = Macaroon.importMacaroon(givenToThirdParty)
fromThirdParty.verify(rootKey, verifier)

// In this example, the macaroon will expire after 1 second
setTimeout(function () {
  try {
    fromThirdParty.verify(rootKey, verifier)
  } catch (err) {
    console.log('...but if we wait for too long the server won\'t accept it anymore')
  }
}, 1001)
```
