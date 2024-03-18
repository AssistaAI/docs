---
sidebar_position: 2
---

# External Service

- [External Service](#external-service)
  - [What is this?](#what-is-this)
    - [The `Base` class](#the-base-class)
    - [The Action class](#the-action-class)
  - [Let's break that down](#lets-break-that-down)
  - [What's next?](#whats-next)

## What is this?
We need a way to call the external service from Assista.

### The `Base` class
- This class `Google::Base` serves as a foundation for interacting with Google APIs.
- It includes methods for generating request headers containing the OAuth2 access token and refreshing the access token when necessary.
- Additionally, it relies on environment variables (GOOGLE_CLIENT_ID and GOOGLE_CLIENT_SECRET) for Google API client credentials.
```ruby title="services/google/base.rb"
module Google
  class Base
    TOKEN_URL = 'https://oauth2.googleapis.com/token'

    def initialize(user)
      @user = user
    end

    private

    def headers
      {
        'Authorization' => "Bearer #{refresh_access_token}",
        'Content-Type' => 'application/json'
      }
    end
  end
end
```

### The Action class
This class `Google::Gmail` is designed to facilitate interactions with the Gmail API for sending emails. It provides a method send_email to send emails, and internally utilizes the create_message method to generate the MIME-encoded message required by the Gmail API.

> **_NOTE:_**  We want to use this class as `Google::Gmail.new(user).some_action(some_argument)`

1. Initializing the action class
```ruby title="services/google/gmail.rb"
module Google
  class Gmail < Base
    URL = 'https://gmail.googleapis.com/gmail/v1'

    def initialize(user)
      @user = user
      @credential = @user.credentials.find_by(name: 'gmailOAuth2')
    end
  end
end
```
2. Calling the action method
```ruby title="services/google/gmail.rb"
# Send Email method
def send_email(to:, subject:, body:)
  message = create_message(to, subject, body)

  response = HTTParty.post(
    "#{URL}/users/me/messages/send",
    headers:, body: { raw: message }.to_json
  )

  raise StandardError, 'There was an issue sending the email. :(' unless response.success?

  response
end
```

## Let's break that down
1. Creating the message body and encoding it in the way Google requires it
```ruby title="services/google/gmail.rb"
  def send_email(to:, subject:, body:)
    # highlight-start
    message = create_message(to, subject, body)
    # highlight-end

    response = HTTParty.post(
      "#{URL}/users/me/messages/send",
      headers:, body: { raw: message }.to_json
    )

    raise StandardError, 'There was an issue sending the email. :(' unless response.success?

    response
  end

  private

  # highlight-start
  def create_message(to, subject, body)
    email = <<~END_EMAIL
      From: #{@user.email}
      To: #{to}
      Subject: #{subject}
      MIME-Version: 1.0
      Content-Type: text/plain; charset="UTF-8"

      #{body}
    END_EMAIL

    Base64.urlsafe_encode64(email)
  end
    # highlight-end
```
2. Making the request - We need to make a post request to the specific endpoint to send an email
```ruby title="services/google/gmail.rb"
# Send Email method
def send_email(to:, subject:, body:)
  message = create_message(to, subject, body)

  # highlight-start
  response = HTTParty.post(
    "#{URL}/users/me/messages/send",
    headers:, body: { raw: message }.to_json
  )
  # highlight-end

  raise StandardError, 'There was an issue sending the email. :(' unless response.success?

  response
end
```
3. Raise an error if anything goes wrong
```ruby title="services/google/gmail.rb"
def send_email(to:, subject:, body:)
  message = create_message(to, subject, body)
  
  response = HTTParty.post(
    "#{URL}/users/me/messages/send",
    headers:, body: { raw: message }.to_json
  )

  # highlight-start
  raise StandardError, 'There was an issue sending the email. :(' unless response.success?
  # highlight-end

  response
end
```
3. Returning the response is always a good practice even if we're not using anything yet from the response
```ruby title="services/google/gmail.rb"
def send_email(to:, subject:, body:)
  message = create_message(to, subject, body)
  
  response = HTTParty.post(
    "#{URL}/users/me/messages/send",
    headers:, body: { raw: message }.to_json
  )
  
  raise StandardError, 'There was an issue sending the email. :(' unless response.success?

  # highlight-start
  response
  # highlight-end
end
```



## What's next?

- Read the [official documentation](https://docusaurus.io/)
