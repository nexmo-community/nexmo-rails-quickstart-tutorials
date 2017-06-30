# How to Receive SMS Messages with Ruby on Rails

_This is the third article in a series of "Getting Started with Nexmo and Ruby on Rails" tutorials._

In the previous article, you set up your Rails application to be publicly accessible by Nexmo, and then received a **Delivery Receipt** for a sent message. In this article you will learn how to receive an inbound SMS by implementing a similar webhook endpoint in Ruby on Rails.

[View the source code on GitHub](https://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/inbound_sms_controller.rb)

## Prerequisites

For this tutorial I assume you will:

- Have a basic understanding of Ruby, and Rails
- Have [Rails](http://rubyonrails.org/) installed on your machine
- Have followed our previous tutorial on Receiving a Delivery Receipt with Ruby on Rails

## What is an "Inbound SMS"?

When someone sends an SMS message to the Nexmo Number that you purchased in the first tutorial it will be received by Nexmo, and we will then pass the content of that message on to a webhook in your application.

To receive this webhook you will need to set up a webhook endpoint and tell Nexmo where to find it.

![DLR flow](sms-receive/diagram-receive.png)

In the previous tutorial I already covered how to set up [Ngrok](http://ngrok.io) for your application to allow it to be accessible even in a development environment.

## Set the Webhook Endpoint with Nexmo

There are 2 ways to set up your number for inbound SMS. The first way is to install the [Nexmo CLI tool](https://github.com/nexmo/nexmo-cli) (a NodeJS command line interface) and then run the following commands to bind your number to an inbound SMS webhook:

```sh
$ nexmo link:sms 12015555522 http://abc123.ngrok.io/inbound_sms
Number updated
```

Alternatively, head over to the [Settings page](https://dashboard.nexmo.com/settings) on the Nexmo Dashboard and scroll down to **API settings** to configure the **Webhook URL for Inbound Message**.

[`dashboard.nexmo.com/settings`](https://dashboard.nexmo.com/settings)

![Global inbound SMS configuration](sms-receive/settings.png)

_Note: This sets the webhook endpoint for inbound SMS at an account level. You can also set up unique webhook endpoints for each virtual number._

## Handle an Inbound SMS WebHook

The hard part is really done again at this point. When an SMS has been sent to your virtual Nexmo number, Nexmo will notify your application by sending a webhook. A typical payload for an inbound SMS will look something like this.

```json
{
    "msisdn": "12025555511",
    "to": "12015555522",
    "messageId": "0B00000057CC7BC7",
    "text": "Hello World!",
    "type": "text",
    "keyword": "HELLO",
    "message-timestamp": "2017-06-29 00:00:00"
}
```

In this payload the sending number is identified by the `msisdn` parameter, and your Nexmo number is the `to` parameter. Let's add a new controller to process this payload and store a new SMS record.

```ruby
# app/controllers/inbound_sms_controller.rb
class InboundSmsController < ApplicationController
  skip_before_action :verify_authenticity_token

  def create
    sms = Sms.create(
      to: params[:to],

      from: params[:msisdn],
      text: params[:text],
      message_id: params[:messageId],
      is_inbound: true
    )

    # Let's send a reply as well?
    reply sms
    head :ok
  end
end
```

Finally, let's immediately reply back to the sender by reversing the message they sent and sending it back to them.

```ruby
# app/controllers/inbound_sms_controller.rb
private

def reply sms
  nexmo = Nexmo::Client.new(
    key: 'YOUR-API-KEY',
    secret: 'YOUR-API-SECRET'
  )

  nexmo.send_message(
    from: sms.to,
    to: sms.from,
    text: sms.text.reverse
  )
end
```

Ok, now start your server, ensure you have something like Ngrok running, and send an SMS to your Nexmo number.

![Reversed SMS result](sms-receive/reverse.png)

## To sum things up

That's it for this tutorial. We've set up our Rails application to receive inbound SMS webhooks, informed Nexmo of where to find our server, processed an incoming SMS webhook, and replied back to the sender.

You can view the [code used in this tutorial](ttps://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/inbound_sms_controller.rb) on GitHub.

## Next steps

In the next tutorial we will move on from SMS and start looking at making our first phone call from a Rails application.
