
## About Webhooks
The webhooks are a way for you to be automatically notified of real-time
events.

When a specific event occurs, the webhook is triggered and will send you
related data through an url that you provided.

The webhook will trigger for businesses with the same `org_id`.

<br>
<b>⚠️ To activate webhooks, you will need to contact your customer
success. ⚠️</b>

You can also set webhooks on our website in the settings > integrations > webhooks.

You can subscribe to each event independently.

To subscribe to an event, you need to provide the following informations:
  - An url where you will receive the event data (can be shared for multiple
  events)
  - Your `org_id`
  - The type of webhook event you wish to activate

<br>
When an event is triggered and if you configured a webhook for it, we will
make an HTTP Post request to the URL that you provided. The body of the
request will contain the payload of the event, as described in the sections
below.

Upon receiving this HTTP request, you must respond with a `200` Success HTTP
code.

If your server fails to treat the incoming request with an Internal error
code (`500` range), we will assume that the error is temporary and retry
sending the event.
If after a few attempts the error persists, we will stop retrying and the
event will be lost.

If your server fails with any other error code, we will assume that you
cannot treat the request and will not retry sending the event.

<br>

## Webhook security

A common requirement for webhook subscribers is the security aspect of validation of issuer.

Here are the possible validations that we offer for security

### Digital Signature

All webhooks contain a header `X-Partoo-Signature-<version>` (example: `X-Partoo-Signature-v1`)
The value of this header is the base64-encoded hash of payload with asymmetric encryption ed25519 algorithm.

<details>
  <summary>Production Key</summary>

  ```
  -----BEGIN PUBLIC KEY-----
  MCowBQYDK2VwAyEA0G9ciHL6XZQXuWq6W4dFLvwNEPWgcdtQgEVlBIwZWBQ=
  -----END PUBLIC KEY-----
  ```
</details>
<details>
  <summary>Sandbox Key</summary>

  ```
  -----BEGIN PUBLIC KEY-----
  MCowBQYDK2VwAyEALsyvX2yVnG3ZKRIFfEvYk2nkzanoNgAqBSqdeNub4sM=
  -----END PUBLIC KEY-----
  ```
</details>

Here are sample snippets

<details>
  <summary>Python</summary>

  ```python
  from base64 import b64decode
  import binascii
  from cryptography.hazmat.primitives import serialization
  from cryptography.hazmat.primitives.asymmetric import ed25519

  # load public key from filesystem, you may adapt depending on your secret management framework
  public_key = serialization.load_pem_public_key(open("/var/secrets/partoo.pub.pem"))

  def validate_signature(request):
    if (signature:=request.headers.get("X-Partoo-Signature-v1")) is None:
      raise ValueError("Missing signature")

    # don't trust your inputs
    # will raise a subclass of ValueError if format is invalid
    decoded_signature = b64decode(signature, validate=True)

    # validate payload's signature
    try:
      public_key.verify(decoded_signature, request.body.encode())
    except Exception as e:
      raise ValueError("Invalid signature") from e
  ```

</details>
<details>
  <summary>JavaScript</summary>


  ```javascript
  const fs = require('fs');
  const crypto = require('crypto');

  // Load public key from filesystem
  const publicKey = fs.readFileSync('/var/secrets/partoo.pub.pem', 'utf8');

  function validateSignature(request) {
    const signature = sig;
    if (!signature) {
      throw new Error('Missing signature');
    }
    // Decode the base64 signature
    let decodedSignature;
    try {
      decodedSignature = Buffer.from(signature, 'base64');
    } catch (err) {
      throw new Error('Invalid signature format');
    }
    // Verify the payload's signature
      if (!crypto.verify(null, Buffer.from(payload), publicKey, decodedSignature)) {
    throw new Error('Invalid signature');
  }
  }
  ```

</details>


### Add a secret to your webhook's url

When configuring the webhook, you may add an additional parameter that may represent some secret that only Partoo and you share.<br/>
This parameter can be either in url path, or as query string parameter.<br/>
Examples:
- `https://my.integration.io/webhooks/partoo/9fa91de19/business_update`
- `https://my.integration.io/webhooks/partoo/business_update?key=9fa91de19`

## Webhook API

You can use the API to subcribe to relevant events using the enpoints described
below.

This feature is currently in beta and the API might change before reaching the
general availability stage.

`PROVIDER` users acting for their customers are required to provide an `org_id`
query parameter.
