# How to Handle Inbound PPhoen Calls with Ruby on Rails

_This is the third article in a series of "Getting Started with Nexmo Voice APIs and Ruby on Rails" tutorials. It continues the "Getting Started with Nexmo SMS and Ruby on Rails" series._

In the previous article, you set up your Rails application to be publicly accessible by Nexmo, and then received a **Call Event Update** for a call in progress. In this article you will learn how to receive an inbound Call by implementing a similar webhook endpoint in Ruby on Rails.

[View the source code on GitHub](https://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/inbound_calls_controller.rb)

## Prerequisites

For this tutorial I assume you will:

- Have a basic understanding of Ruby, and Rails
- Have [Rails](http://rubyonrails.org/) installed on your machine
- Have [NPM](https://www.npmjs.com/) installed for the purpose of our CLI
- Have followed our previous tutorial on Receiving Call Event Updates with Ruby on Rails

## What is an "Inbound Call"?

When someone calls the Nexmo Number that you purchased in the first tutorial it will be received by Nexmo, and we will then make a HTTP call to the `answer_url` for the Nexmo Application associated to your number.

To receive this webhook you will need to set up a webhook endpoint and tell Nexmo where to find it. In the previous tutorial I already covered how to set up [Ngrok](http://ngrok.io) for your application to allow it to be accessible even in a development environment.

## Set the Webhook Endpoint with Nexmo

The first step is to use the [Nexmo CLI tool](https://github.com/nexmo/nexmo-cli) to link the Nexmp Application we created in the previous tutorial to your Nexmo number. We pass in the phone number, and the application's UUID.

```sh
$ nexmo link:app 12015555522 abcd1234-ancd-abcd-abcd-abcd1234abcd
Number updated
```

This command tells Nexmo to make a HTTP call to the `answer_url` of the Nexmo Application every time the Nexmo Number receives an inbound call. We already set the `answer_url` in the first tutorial of this series, but if you need to update it you can do so as follows.

```sh
$ nexmo app:update abcd1234-ancd-abcd-abcd-abcd1234abcd "My Voice App" http://abc123.ngrok.io/inbound_calls http://abc123.ngrok.io/call_events --answer_method POST --event_method POST
Application updated
```

## Handle an Incoming Call WebHook

The hard part is really done again at this point. When a call comes in on your Nexmo number, Nexmo will notify your application by sending a webhook to the `answer_url`. A typical payload for this webhook will look something like this.

```json
{
    "from": "12025555511",
    "to": "12015555522",
    "conversation_uuid": "CON-defg2345-defg-defg-defg-defg2345abcd",
}
```

In this payload the sending conversation is identified by the `conversation_uuid` parameter, and the `from` and `to` specify the caller and the Nexmo number called. Let's add a new controller to process this payload and store a new SMS record.

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :inbound_calls,  only: [:create]
end

# app/controllers/inbound_calls_controller.rb
class InboundCallsController < ApplicationController
  # We disable CSRF for this webhook call
  skip_before_action :verify_authenticity_token

  def create
    Call.where(conversation_uuid: params[:conversation_uuid])
        .first_or_create
        .update_attributes(
          to: params[:to],
          from: params[:from]
        )

    render json: [
      {
        action: 'talk',
        voiceName: 'Jennifer',
        text: 'Hello, thank you for calling. This is Jennifer from Nexmo. Ciao.'
      }
    ]
  end
end
```

Although storing and updating the call details is not really necessary, it's useful to keep track of current call statuses, durations, and any other information that might benefit to your application. This action returns a new [Nexmo Call Control Object (NCCO)](https://docs.nexmo.com/voice/voice-api#ncco) will play back a simple voice message to the recipient as specified. There are many more actions you can specify in the [NCCO](https://docs.nexmo.com/voice/voice-api#ncco), have a play with them if you want.

Ok, now start your server, ensure you have something like [Ngrok](http://ngrok.io) running, and make a voice call to your Nexmo Number! Can you hear Jennifer?

## To sum things up

That's it for this tutorial. We've set up our Rails application to receive an inbound Voice Call webhook, informed Nexmo of where to find our server, processed an incoming webhook, and provided instructions to Nexmo to play back a message.

You can view the [code used in this tutorial](https://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/inbound_calls_controller.rb) on GitHub.

## Next steps

That's it for this series for now. What else would you like to see next?
