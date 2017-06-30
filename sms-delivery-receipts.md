# How to receive an SMS Delivery Receipt from a Mobile Carrier with Ruby on Rails

_This is the second article in a series of "Getting Started with Nexmo and Ruby on Rails" tutorials._

In our previous tutorial I showed you how to send an SMS using the Nexmo API and the Nexmo Ruby gem in a Rails application. What we haven't looked at though is how to know when a message has been delivered. In this tutorial we will look at what it means for a message to be delivered, and how we can listen for Delivery Receipts from Nexmo to update the status of an SMS in our application.

[View the source code on GitHub](https://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/sms_delivery_receipts_controller.rb)

## What Does "Delivered" Mean?

When you make a successful SMS request to Nexmo, the API returns an array of `message` objects, ideally with a status of `0` for "Success". At this moment the SMS has not been delivered yet, rather it's been queued for delivery with Nexmo.

In the next step, Nexmo find the best carrier to deliver your SMS to the recipient, and when they do so they notify Nexmo of the delivery with a **Delivery Receipt (DLR)**.

To receive this DLR in your application, you will need to set up a webhook endpoint, telling Nexmo where to forward these receipts to.

![DLR flow](sms-delivery-receipts/diagram-dlr.png)

## Set the Webhook Endpoint with Nexmo

To receive a webhook we need 2 things, firstly we need to set up our server so that Nexmo can make a HTTP call to it. If you are developing on a local machine this might be hard, which is where tooks like [Ngrok](http://ngrok.io) come in. I won't go too much into detail, but with Ngrok you can make your local Rails server available within seconds to the outside world.

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

Secondly, we need to make sure our server has an endpoint in place that return a nice and clean HTTP 200 response when called. Let's add a new controller with an empty response.

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :sms_delivery_receipts, only: [:create]
end

# app/controllers/sms_delivery_receipts_controller.rb
class SmsDeliveryReceiptsController < ApplicationController
  skip_before_action :verify_authenticity_token

  def create
    head :ok
  end
end
```

With this in place, you can set up this URL as your webhook address on your Nexmo account. Head over to the settings page on the Nexmo Dashboard and scroll down the **API Settings** and fill in the following 2 details.

- Set the **Webook URL for Delivery Receipts** to the Ngrok URL, e.g. `http://abc123.ngrok.io/sms_delivery_receipts`
- Ensure the **HTTP Method** is set to `POST` for this tutorial

![Webhook Endpoint Configuration](sms-delivery-receipts/endpoint.png)

Finally, save the form. You might see an error appear after a few seconds if your server can not be reached, or the endpoint did not return a HTTP 200 response. In this case head over to the [Ngrok Web Interface](http://127.0.0.1:4040) to inspect the request and response made.

## Handle a Delivery Receipt WebHook

The hard part is done at this point really. When an SMS has been sent and delivered, the carrier will notify Nexmo, and we will in return notify your application by sending a webhook. A typical DLR will look something like this.

```json
{
  "msisdn": "12015555522",
  "to": "12025555511",
  "network-code": "310090",
  "messageId": "02000000FEA5EE9B",
  "price": "0.00570000",
  "status": "delivered",
  "scts": "1208121359",
  "err-code": "0",
  "message-timestamp": "2017-06-29 22:40:30"
}
```

We can extend the example from our previous tutorial and update the SMS record we stored then with the new status.

```ruby
# app/controllers/sms_delivery_receipts_controller.rb
def create
  Sms.where(message_id: params[:messageId])
     .update_all(status: params[:status]) if params[:messageId]

  head :ok
end
```

In this example we find the SMS record with the `messageId` provided, and then update its status with the given status, in this case `"delivered"`.

_Note: Some US carriers do not support the feature. Also, if you are sending SMS to a Google Voice number, you will not get any receipt. We do not provide reach to other virtual number providers due to fraud prevention purposes. If you have any particular business case where you would like to be able to reach virtual numbers, please [contact our Support team](https://www.nexmo.com/contact-sales)!_

## To sum things up

That's it for this tutorial. We've set up our Rails application to receive webhooks, informed Nexmo of where to find our server, and processed an incoming webhook with a Delivery Receipt.

You can view the [code used in this tutorial](ttps://github.com/workbetta/nexmo-rails-quickstart/blob/master/app/controllers/sms_delivery_receipts_controller.rb) on GitHub.

## Next steps

In the next tutorial we will look at receiving inbound SMS messages into our application.
