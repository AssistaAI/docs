---
sidebar_position: 1
---

# Credentials

- [Credentials](#credentials)
  - [Authorize user](#authorize-user)
    - [Setup Omniauth provider](#setup-omniauth-provider)
    - [Building and saving the credential](#building-and-saving-the-credential)
  - [Create the credential in N8N](#create-the-credential-in-n8n)
  - [Save the credential in our database](#save-the-credential-in-our-database)
  - [What's next?](#whats-next)

## Authorize user

### Setup Omniauth provider
- Usually you can find a gem that helps with this. For example for Gmail we ca use this gem [omniauth-google-oauth2](https://github.com/zquestz/omniauth-google-oauth2).
- Any extra Omniauth provider can be setup in `config/initializers/omniauth.rb`
```ruby title="config/initializers/omniauth.rb"
Rails.application.config.middleware.use OmniAuth::Builder do
  # (...other providers...)

  provider :google_oauth2, ENV['GOOGLE_CLIENT_ID'], ENV['GOOGLE_CLIENT_SECRET'], {
    scope: [
      'https://www.googleapis.com/auth/gmail.addons.current.message.action',
      ...
    ],
    access_type: 'offline',
    prompt: 'consent',
    include_granted_scopes: true,
    name: 'gmail_authorize'
  }
end
```
- Now you can use the name of the provider in a `button_to` form
:::note
The query param `state` given.
:::
```erb
<%= button_to "/auth/gmail_authorize?state=gmail", data: { turbo: false } do %>
  Authorize Gmail
<% end %>
```
- Than we need to handle the redirect back from the provider (in our case Google).
```ruby title="app/controllers/sessions/omniauth_controller.rb"
# ...
return authorize_gmail if params['state'] == 'gmail'
# ...

def authorize_gmail
  save_gmail_data
  redirect_to agents_path, notice: 'Successfully Authorized'
end

# ...
def save_gmail_data
  @user.save_uid(omniauth_params) if @user.uid.nil?
  @user.save_credential(omniauth)
end
```

### Building and saving the credential
The `omniauth` object passed earlier in the `@user.save_credential(omniauth)` should have all the tokens and data needed for N8N credential.
- Firstly we should `find_or_create` a credential by name associated with the current user. In our case we will use the name given by the omniauth `gmailOAuth2`
:::note
Ensure you add the new name in the `app/models/credential.rb` **ALLOWED_NAMES** array
:::
```
credential = credentials.find_or_create_by(name: 'gmailOAuth2')
```
- Secondly We have to update the credential with the data from the omniauth object
> **_`email:`_** will be used in n8n as the credential name

> **_`schema:`_** is a jsonb attribute and ca be different for each type of credential
```rb
credential.update(email: omniauth.dig('info', 'email'),
                  schema: {
                    "access_token": omniauth.dig('credentials', 'token'),
                    "expires_in": Time.at(omniauth.dig('credentials', 'expires_at')) - Time.now,
                    "callbackQueryString": {
                      "scope": omniauth.dig('credentials', 'scope')
                    }
                  })
```









## Create the credential in N8N
tbc
## Save the credential in our database
tbc

## What's next?

- Read the [official documentation](https://docusaurus.io/)
