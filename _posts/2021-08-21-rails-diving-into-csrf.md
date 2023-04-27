---
layout: post
title: Rails - Diving into CSRF
date: '2021-08-21'
categories: [security, posts]
published: true
tags: [ruby, rails, csrf, security, owasp]
---

### Cross-Site Request Forgery (CSRF)

A Cross-Site Request Forgery (CSRF) attack is a form of attack that exploits a vulnerability where a malicious website tricks an authenticated user into unknowingly submitting sensitive information to a legitimate website. This is made possible because the user's browser automatically includes all cookies, including session cookies, when making requests to a website. Therefore, the legitimate website is unable to differentiate between a genuine user request and a fraudulent one, allowing the attacker to obtain access to confidential information.

A CSRF attack can also be executed through a webpage that includes an automatic form submission. In this scenario, the attacker creates a webpage with a form that includes sensitive information, like a money transfer or a password change, and is programmed to submit automatically. The attacker then deceives the user into visiting the webpage, triggering the automatic submission of the form, which ultimately leads to the successful execution of the attacker's action on the targeted legitimate website.

![alt Attacker-Genuine](/img/blogs/rails/attacker_genuine.png "Attacker vs Genuine")

### CSRF token and rails

To mitigate CSRF attacks, anti-CSRF tokens can be implemented. These tokens are distinct, unpredictable values that are linked to a user's session. They are incorporated into forms and requests and verified by the website before any sensitive actions are executed. By doing so, only requests that contain a valid token are authorized, safeguarding the website from CSRF attacks.

To prevent CSRF attacks, websites can utilize anti-CSRF tokens. These tokens are exclusive and unpredictable values that correspond to each user's session. They are added to forms and requests and verified by the website before executing any sensitive actions. As a result, only requests that contain an authentic token are permitted, providing a shield against CSRF attacks.

After creation, the anti-CSRF token is linked to the user's session and integrated into forms and requests. Prior to executing any sensitive actions, the server-side application validates the token to verify that only legitimate requests that contain a valid token are accepted. It's essential to ensure that the CSRF token is unique for each request and form, and that it is not stored on the client-side. Furthermore, the token should be changed frequently to maintain its effectiveness.

The Rails web application framework produces a CSRF token once per session as opposed to for every request to evade usability concerns. If a user clicked the back button and submitted a form, an error would arise because of the invalid token, making it imperative to avoid such situations. Therefore, Rails' approach helps to ensure that the CSRF token remains consistent throughout the user's session, enabling smooth navigation without triggering errors.

Rails 5 has introduced a new feature that allows per-form CSRF token generation. This feature can be activated either at the application level or the controller level, which provides more precise security control.

```ruby
class ActivitiesController < ApplicationController
  self.per_form_csrf_tokens = true
end
```

```ruby
config.action_controller.per_form_csrf_tokens = true
```

### Authenticity token

In Rails, the term "authenticity_token" is employed in forms instead of "csrf_token", and the token is renewed every time the page is refreshed. This helps to keep track of any attempts to submit a form multiple times using the same token, which increases security against CSRF attacks.

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
