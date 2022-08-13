---
title: Asymmetric Key Encryption (ECDSA and RSA) 
description: Webhook security Asymmetric Keys 
--- 

{% table %}
---
* **Complexity**
* - High
---
* **Pros**
* - Extends HMAC with Non-Repudiation (ensures webhook calls can be sent only by the webhook provider)
---
* **Caveats**
* - Additional deployment complexity (compared to HMAC)
  - Additional operational complexity to issue, renew, and rotate keys
---
* **Examples**
* - [Sendgrid](https://docs.sendgrid.com/for-developers/tracking-events/getting-started-event-webhook-security-features)
  - [PayPal](https://developer.paypal.com/docs/api-basics/notifications/webhooks/notification-messages/#event-headers)
{% /table %}
---

In our research, we found a couple of providers — PayPal and SendGrid — using asymmetric encryption for webhook security. In this method, the webhook provider uses a private key (only known to the provider) to sign requests, while the listener uses a public key with a verifier to validate webhook calls. At a conceptual level, the process works as follows:

1. On webhook requests, the provider signs the webhook message using its private key. The signature is included in the webhook request as a header.
2. The webhook listener receives the request and runs an ECDSA or RSA verifier with the public key, the request body, and the signed value from the provider. If the verifier returns positive, it means that the signature could only be created by the private key, and the request is considered legit.

The big difference between HMAC and asymmetric key encryption is in how the verification process is carried out. In HMAC, webhook listeners generate the same signature from the provider using the same key to validate the request. In asymmetric encryption, listeners cannot generate the same signature. Instead, they use a verifier with the public key to confirm if the request is legit.

This difference in verification logic is used to deliver non-repudiation. In HMAC, anyone with access to the private key can generate a webhook request. With asymmetric keys, only the webhook provider — in possession of the private key — can generate legit signatures. 

Webhook implementations with asymmetric keys can also use the same techniques as HMAC providers to add forward compatibility and replay prevention. SendGrid, for example, implements a timestamp header ( `X-Twilio-Email-Event-Webhook-Timestamp` ) within the payload to mitigate replay attacks.

However, asymmetric encryption comes with drawbacks. The most compelling: asymmetric encryption is harder to implement than HMAC and will introduce many different methods for shipping and rotating public keys

PayPal sends the public key with the webhook request for verification (in the `PAYPAL-CERT-URL` header). SendGrid informs the public key in its settings page.