# Run the Microsoft Graph Ruby REST quick start sample

Thank you for downloading the Microsoft Graph Ruby REST quick start sample. Follow these instructions to run and try out the sample.

> **Note:** These instructions apply if you downloaded the sample via the [Microsoft Graph Quick Start](https://developer.microsoft.com/graph/quick-start) page. In addition to the quick start, we also provide the following alternatives to get started with Microsoft Graph and Ruby:
>
> - This sample is also available on [GitHub](https://github.com/microsoftgraph/ruby-connect-rest-sample). If you downloaded the sample from GitHub, please follow the instructions in the README file.
> - If you prefer to build your own app from scratch, see our [Microsoft Graph Ruby on Rails tutorial](/concepts/ruby.md).

## Prerequisites

Before you follow the instructions in this document, you'll need to make sure you have the following:

- [Ruby 2.5.1 or later](https://www.ruby-lang.org/)
- An Outlook.com account, or a Microsoft work or school account

## Build and run the sample

1. Run the following command to install [Bundler](http://bundler.io/).

    ```Shell
    gem install bundler
    ```

1. Run the following command to install the application's dependencies.

    ```Shell
    bundle install
    ```

1. Run the following command to start the application.

    ```Shell
    rackup -p 3000
    ```

## Use the sample application

1. Open a browser and navigate to `http://localhost:3000`.

    ![A screenshot of the sample application home page](images/ruby-rest/app-homepage.PNG)

1. Choose the **Connect to Microsoft Graph** button to sign in to your Outlook.com or Microsoft work or school account.

1. Review and accept the requested permissions. The browser redirects back to the application.

    ![A screenshot of the sample application after the user signs in](images/ruby-rest/app-signed-in.PNG)

1. Choose the **Send mail** button to send a message from the signed-in user to the signed-in user.

    ![A screenshot of the status message after successfully sending the message](images/ruby-rest/app-success-message.PNG)

1. Choose the **Disconnect** button to sign out and return to the home page.

1. Check the user's mailbox for the message sent from the app. The subject of the message is `Welcome to Microsoft Graph development with Ruby on Rails`.

## Explore the code

Let's take a look at some of the key parts of the sample. We'll also point out where you can add your own code to change the sample.

### Authentication

The sample uses [OmniAuth](https://github.com/omniauth/omniauth) with the [omniauth-microsoft_v2_auth](https://github.com/cbales/omniauth-microsoft_graph) gem to handle the [Azure AD authorization code flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-code) for getting an access token.

- The requested scopes (or permissions) are defined in the **./config/environment.rb** file. If you want to modify the sample to use the Microsoft Graph for something other than sending mail, you can add the [applicable permissions](/concepts/permissions_reference.md) there.

    ```ruby
    ENV['OAUTH_SCOPE'] =
      'openid ' \
      'email ' \
      'profile ' \
      'User.Read ' \
      'Mail.Send'
    ```

- The callback for the OAuth flow is implemented in the `callback` function in **./app/controllers/pages_controller.rb**.

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      # and extract the auth token, user name, and email
      data = request.env['omniauth.auth']

      @email = data[:extra][:raw_info][:userPrincipalName]
      @name = data[:extra][:raw_info][:displayName]

      # Associate token/user values to the session
      session[:access_token] = data['credentials']['token']
      session[:name] = @name
      session[:email] = @email

      # Debug logging
      logger.info "Name: #{@name}"
      logger.info "Email: #{@email}"
      logger.info "[callback] - Access token: #{session[:access_token]}"
    end
    ```

### Microsoft Graph calls

The app uses the Microsoft Graph to [send an email from the user](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_sendmail). The API call for this is in the `send_mail` function in **./app/controllers/pages_controller.rb**.

- The body of the request is constructed as a JSON string.

    ```ruby
    email_message = "{
            Message: {
            Subject: '#{email_subject}',
            Body: {
                ContentType: 'HTML',
                Content: '#{email_body}'
            },
            ToRecipients: [
                {
                    EmailAddress: {
                        Address: '#{@recipient}'
                    }
                }
            ]
            },
            SaveToSentItems: true
            }"
    ```

- The request is sent via HTTP POST, with the access token in the `Authorization` header.

    ```ruby
    response = http.post(
      SENDMAIL_ENDPOINT,
      email_message,
      'Authorization' => "Bearer #{session[:access_token]}",
      'Content-Type' => content_type
    )
    ```

If you want to modify the sample to use the Microsoft Graph for something other than sending mail, you could modify the `send_mail` function or create a new function modeled after the `send_mail` function. Keep in mind that if you create a new function, you would also need to:

- Configure a route for the new function in **./config/routes.rb**.
- Change the form action to your new route in **./app/views/pages/callback.html.erb**.

## Troubleshooting

If you run into any problems using this quick start, please check the [issues page on GitHub](https://github.com/microsoftgraph/ruby-connect-rest-sample/issues). If you can't find an answer to your problem in the existing issues there, please create a new issue, providing as much detail as possible.

## Next steps

- Learn more about [Azure AD and OAuth](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).
- Learn more about [what's possible in the Microsoft Graph](/concepts/overview.md).
- Explore the [Microsoft Graph API reference](/concepts/v1-overview.md).