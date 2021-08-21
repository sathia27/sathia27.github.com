---
layout: post
title: Rails - Diving into CSRF
date: '2021-08-21'
categories: [security, posts]
published: true
tags: [ruby, rails, csrf, security, owasp]
---

### Cross-Site Request Forgery (CSRF)

Consider you have visited both Genuine and attacker websites. Attacker tricks you to submit information to a Genuine website from an attacker website. A CSRF attack works because browser requests automatically include all cookies including session cookies. Therefore, if the user is authenticated to the site, the site cannot distinguish between genuine user requests an attacker requests.

By doing this, attacker can change password or update any critical information of the genuine website.

![alt Attacker-Genuine](/img/blogs/rails/attacker_genuine.png "Attacker vs Genuine")

### CSRF token and rails

CSRF tokens should be generated on the server-side. They can be generated once per user session or for each request. Whenever a form is submitted, the server must verify submitted token and session is matching.

Rails generate csrf token **per session** instead of each request. Rails took this decision to **avoid usability** issues. Let’s say if the user clicked the back button, and try to submit the form then the application throws an error because of the in-valid token.

Rails 5 comes up with **per-form csrf-token** which can be enabled in application level or controller level.

```ruby
class ActivitiesController < ApplicationController
  self.per_form_csrf_tokens = true
end
```

```ruby
config.action_controller.per_form_csrf_tokens = true
```

### Authenticity token

Rails form inserts authenticity_token in every form instead of csrf_token, every time we reload the page, the authenticity token will change. This could be easy to monitor if a user is trying to submit the form with the same authenticity multiple times.

Authenticity token is generated based on csrf_token. As csrf_token is generated per session, authenticity_token will expire only when user log out.

#### Form with authenticity_token

```html
<form method="post" action="http://www.travel.com/activities">
  <input type="hidden" name="authenticity_token" value="HTyAhRpOcZ1OyKjmq12hH4RYiMrFY4cVD2J54uPnCrE4qblDCqqcMSwAR59QtvthPWPd3BxG_MFAybc_HnipvA">
</form>
```

#### How authenticity_token is generated

Below is the logic for generating an authenticity token

```
Base64::encoding( one_time_pad + (one_time_pad XOR session[:_csrf_token]) )
```

[one-time](https://en.wikipedia.org/wiki/One-time_pad) pad is the reason for new authenticty_token generated every time. whenever a form is submitted to the server, rails verifies authenticity_token which is generated and verifies if it has valid csrf-token.

This makes the attacker’s job difficult, as he cannot submit the form without knowing the user’s csrf_token.


Below is rails code for the same ([link](https://github.com/rails/rails/blob/18707ab17fa492eb25ad2e8f9818a320dc20b823/actionpack/lib/action_controller/metal/request_forgery_protection.rb#L361))

```ruby
def valid_authenticity_token?(session, encoded_masked_token) # :doc:
  if encoded_masked_token.nil? || encoded_masked_token.empty? || !encoded_masked_token.is_a?(String)
    return false
  end

  begin
    masked_token = decode_csrf_token(encoded_masked_token)
  rescue ArgumentError # encoded_masked_token is invalid Base64
    return false
  end

  # See if it's actually a masked token or not. In order to
  # deploy this code, we should be able to handle any unmasked
  # tokens that we've issued without error.

  if masked_token.length == AUTHENTICITY_TOKEN_LENGTH
    # This is actually an unmasked token. This is expected if
    # you have just upgraded to masked tokens, but should stop
    # happening shortly after installing this gem.
    compare_with_real_token masked_token, session

  elsif masked_token.length == AUTHENTICITY_TOKEN_LENGTH * 2
    csrf_token = unmask_token(masked_token)

    compare_with_global_token(csrf_token, session) ||
    compare_with_real_token(csrf_token, session) ||
    valid_per_form_csrf_token?(csrf_token, session)
  else
    false # Token is malformed.
  end
end

def unmask_token(masked_token) # :doc:
  # Split the token into the one-time pad and the encrypted
  # value and decrypt it.
  one_time_pad = masked_token[0...AUTHENTICITY_TOKEN_LENGTH]
  encrypted_csrf_token = masked_token[AUTHENTICITY_TOKEN_LENGTH..-1]
  xor_byte_strings(one_time_pad, encrypted_csrf_token)
end
```
