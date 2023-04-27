---
layout: post
title: Rails - Diving into CSRF
date: '2021-08-21'
categories: [security, posts]
published: true
tags: [ruby, rails, csrf, security, owasp]
---

### Cross-Site Request Forgery (CSRF)

A CSRF attack is a type of attack where a malicious website tricks the user into submitting sensitive information to a legitimate website, while the user is authenticated. This is possible because the browser automatically includes all cookies, including session cookies, in requests made to a website. As a result, the legitimate website cannot distinguish between a genuine user request and an attacker request, allowing the attacker to gain access to sensitive information.

Another example of a CSRF attack is when an attacker creates a webpage with a form that automatically submits sensitive information, such as a password change or a money transfer, when the user visits the page. The attacker then tricks the user into visiting the page, which will automatically submit the form and execute the attacker's desired action on the legitimate website.

![alt Attacker-Genuine](/img/blogs/rails/attacker_genuine.png "Attacker vs Genuine")

### CSRF token and rails

CSRF attacks can be mitigated by using anti-CSRF tokens, which are unique, unpredictable values that are associated with each user's session. These tokens are included in forms and requests, and are checked by the website before any sensitive action is taken. This ensures that only requests that contain a valid token are accepted, protecting the website from CSRF attacks.

CSRF attacks can be mitigated by using anti-CSRF tokens, which are unique, unpredictable values that are associated with each user's session. These tokens are included in forms and requests, and are checked by the website before any sensitive action is taken. This ensures that only requests that contain a valid token are accepted, protecting the website from CSRF attacks.

Once generated, the token is associated with the user's session and included in forms and requests. The token is then checked by the server-side application before any sensitive action is taken to ensure that only requests that contain a valid token are accepted. It's important to note that the CSRF token should be unique for each form and request, and should not be stored client-side. It also should be rotated regularly.

Rails, a web application framework, generates a CSRF token once per session instead of for every request, in order to prevent usability issues. For instance, if a user clicks the back button and submits a form, an error would occur due to the invalid token. This is to avoid such situations.

In Rails 5, an option for generating a per-form CSRF token has been introduced. This can be enabled on the application level or on the controller level, providing more granular control and security.

```ruby
class ActivitiesController < ApplicationController
  self.per_form_csrf_tokens = true
end
```

```ruby
config.action_controller.per_form_csrf_tokens = true
```

### Authenticity token

Rails uses "authenticity_token" instead of "csrf_token" in forms, and regenerates the token each time the page is reloaded. This allows for easy monitoring of attempts to submit a form multiple times with the same token, increasing security against CSRF attacks.

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
