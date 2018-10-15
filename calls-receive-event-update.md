# How to Receive Voice Call Events for a Call In Progress with Ruby on Rails

_This is the second article in a series of "Getting Started with Nexmo Voice APIs and Ruby on Rails" tutorials. It continues the "Getting Started with Nexmo SMS and Ruby on Rails" series._

In our previous tutorial, I showed you how to make a text-to-speech call using the Nexmo API and the Nexmo Ruby gem in a Rails application. What we haven't looked at though is how to know when a call has connected or completed. In this tutorial, we will look at how we can listen for Call Events from Nexmo to update the status of an Call in our application.

[View the source code on GitHub](https://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/call_events_controller.rb)

## Prerequisites

For this tutorial I assume you will:

- Have a basic understanding of Ruby, and Rails
- Have [Rails](http://rubyonrails.org/) installed on your machine
- Have [NPM](https://www.npmjs.com/) installed for the purpose of our CLI
- Have followed our previous tutorial on Making a Text-To-Speech Call with Ruby on Rails

## What Does "Connected" Mean?

When you make a successful Voice Call request to Nexmo, the API returns a status for your call, often this will be the initial state of `started`. In the next step, Nexmo will route your call and start ringing the phone of the recipient, and when we do so we can notify your Rails application of the change in status using a **Call Event Webhook**.

To receive this webhook in your application, you will need to set up a webhook endpoint, telling Nexmo where to forward these receipts to. Lucky for

## Set the Webhook Endpoint with Nexmo

To receive a webhook we need 2 things, firstly we need to set up our server so that Nexmo can make a HTTP call to it. If you are developing on a local machine this might be hard, which is where tools like [Ngrok](http://ngrok.io) come in. I won't go too much into detail, but with Ngrok you can make your local Rails server available within seconds to the outside world.

```sh
# forwarding port 3000 to an externally accessible URL
$ ngrok http 3000

Session Status                online
Account                       Cristiano Betta
Version                       2.2.4
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://6382942c.ngrok.io -> localhost:3000
Forwarding                    https://6382942c.ngrok.io -> localhost:3000
```

With this in place, you can set up this URL as your `event_url` webhook address on your Nexmo Application. Lucky for us, we already did this when we created the Nexmo Application in the previous tutorial.

```sh
$ nexmo app:create "My Voice App" http://abc123.ngrok.io/inbound_calls http://abc123.ngrok.io/call_events --keyfile private.key --answer_method POST --event_method POST
Application created: abcd1234-ancd-abcd-abcd-abcd1234abcd
Private Key saved to: private.key
```

If you need to change the URLs somehow, you can do so easily using a pretty similar command.

```sh
$ nexmo app:update abcd1234-ancd-abcd-abcd-abcd1234abcd "My Voice App" http://abc123.ngrok.io/inbound_calls http://abc123.ngrok.io/call_events --answer_method POST --event_method POST
Application updated
```

## Handle a Call Event WebHook

The hard part is done at this point really. When a Call has been initiated Nexmo will notify your application of any changes in the call by sending a webhook. A typical payload will look something like this.

```json
{
  "uuid": "abcd1234-ancd-abcd-abcd-abcd1234abcd",
  "conversation_uuid": "CON-defg2345-defg-defg-defg-defg2345abcd",
  "status": "ringing",
  "direction": "outbound"
}
```

We can extend the example from our previous tutorial and update the Call record we stored then with the new status.

```ruby
# app/controllers/call_events_controller.rb
class CallEventsController < ApplicationController
  # We disable CSRF for this webhook call
  skip_before_action :verify_authenticity_token

  def create
    if params[:uuid]
      Call.where(uuid: params[:uuid])
          .first_or_create
          .update(
            status: params[:status],
            conversation_uuid: params[:conversation_uuid]
          )
    end

    head :ok
  end
end
```

In this example, we find the Call record with the `uuid` provided, and then update its status with the given `status`, in this case `"ringing"`.

## To sum things up

That's it for this tutorial. We've set up our Rails application to receive webhooks, informed Nexmo where to find our server, and processed an incoming webhook with a Delivery Receipt.

You can view the [code used in this tutorial](https://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/call_events_controller.rb) on GitHub.

## Next steps

In the next tutorial, we will look at receiving inbound Voice Calls into our application.
