---
description: Chain multiple functions together
---

# Send result to another service

A common use case of serverless functions is to act as the bridge between several web services or messaging queues. It receives a request from a source, and then send the return value onto the next service.

The way to accomplish that is to return a JSON string from the function. If the JSON object contains a `callback` object, the FaaS would strip it from the return value, and then send the rest of the return value to the HTTP endpoint defined in the callback.

In the following example, the `say()` function returns a JSON `callback`, which requests SendGrid to send the hello messages as an email. The Rust function is as follows.

```text
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn say(context: &str, s: &str) -> String {
  let r = String::from("hello ");
  let ret = r#"
    {
      'callback': {
        'method': 'POST',
        'hostname': 'api.sendgrid.com',
        'port': 443,
        'path': '/v3/mail/send',
        'headers': {
          'Content-Type': 'application/json',
          'authorization': 'Bearer {}'
        },
        'maxRedirects': 20
      },
      'personalizations': {
        [{
          'to':[{'email':'{}','name':''}],
          'subject':'{}'
        }],
        'from':{'email':'{}','name':''}
      }
    }
  "#
  
  return format!(ret, "auth_token_123", "dev@secondstate.io", r + &s, "alice@secondstate.io");
}
```

The `callback` object in the return value is as follows. It conforms to the Node.js request options specification.

```text
{
  'method': 'POST',
  'hostname': 'api.sendgrid.com',
  'port': 443,
  'path': '/v3/mail/send',
  'headers': {
    'Content-Type': 'application/json',
    'authorization': 'Bearer <<YOUR_API_KEY>>'
  },
  'maxRedirects': 20
}
```

The HTTP body sent to SendGrid is as follows.

```text
{'personalizations':
  [{
    'to':[{'email':"dev@example.com","name":""}],
    'subject':'hello email'
  }],
  'from':{'email':'alice@secondstate.io','name':''}
}
```

The recipient email address and auth token are currently hardcoded in the Rust source code. But you can use the stateful context in the previous article to configure them! Just set a JSON configuration object in the state. Try it!

You can see that the `callback` could direct the result from one function to another, and hence chaining multiple functions together.



