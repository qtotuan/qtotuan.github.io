---
layout: post
title:  "Sending Emails With Rails Application"
date:   2017-06-24 12:11:03 +0200
categories: rails
---
You've set up your Rails app and want to send out emails to your users, for example welcome emails for sign ups or thank you emails when they completed an order.

In order to do that you'll need Rails' ActionMailer class and a third party email service, for example SendGrid.

In the following is an example of how we set it up for our FlatironShop website.

First of, you create a Mailer controller by inheriting from ActionMailer::Base and put the file in 'app/mailers'.
This is an example for FlatironShop, where we specify a method called order_email:

``` ruby
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@flatironshop.com"
  layout 'mailer'
end

# app/mailers/user_mailer.rb
class UserMailer < ApplicationMailer
  default from: 'notifications@flatironshop.com'

  def order_email(user)
    @user = user
    @url  = 'http://flatironschool.com/login'
    mail(to: @user.email, subject: 'Thanks for your order')
  end
end
```

Like a controller, UserMailer needs a view to render. So we need to create order_email.html.erb which will serve as the template:

``` ruby
# app/views/user_mailer/order_email.html.erb

<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1>Thanks for ordering, <%= @user.name %>!</h1>
    <p>Your package will be soon on its way.</p>
    <p>This is what you ordered:</p>
      <ul>
        <% @order.items.each do |item| %>
          <li><%= item.name %> from <%= item.seller.name %></li>
        <% end %>
      </ul>
    <p>Thanks again and have an awesome day!</p>
  </body>
</html>
```


Now you can use the ActionMailer methods in your controllers to render the view, i.e. send the email, just like any other method. We are calling our order_email method on the UserMailer class that we created above, pass in the arguments and call 'deliver_now'. This is how it looks like in the orders_controller:

``` ruby
# app/controllers/orders_controller.rb

def create
  @order = Order.new(order_params)
  @order.buyer_id = current_user.id
  current_cart.each do |item_id|
    @order.items << Item.find(item_id)
  end
  if @order.items.size < 1
    return flash[:danger] = "Your cart is currently empty."
  end

  if @order.valid?
    @order.save
    session[:cart] = []
    UserMailer.order_email(current_user, @order).deliver_now
    redirect_to order_path(@order)
  else
    flash[:danger] << @order.errors.full_messages.first
    redirect_to cart_path
  end
end
```


And that's it for the MVC part.

Now, assuming that your app is deployed with Heroku, we need to set up a SendGrid account so that we can retrieve the API key and set the ActionMailer's SMTP settings to use the SendGrid credentials.

After you've set up your account (you can choose the free version) run in the console:

$ heroku config:get SENDGRID_USERNAME  
$ heroku config:get SENDGRID_PASSWORD

To get the API key, log into your SendGrid account, click the “Create API Key” button, and a dropdown menu will appear allowing you to choose the type of API Key you would like to create.

![sendgrid]({{ site.url }}/assets/170624_sendgrid.png)
<br>

Set the environment in the console:

$ heroku config:set SENDGRID_API_KEY=xxxx_api_key_xxxx


Now that the API key has been set we need to include them in the Gemfile (don't forget to run 'bundle'):

$ gem 'sendgrid-ruby'

The last step is to configure the ActionMailer to use SendGrid:

``` ruby
# config/environement.rb

ActionMailer::Base.smtp_settings = {
  :user_name => ENV['SENDGRID_USERNAME'],
  :password => ENV['SENDGRID_PASSWORD'],
  :domain => 'flatironschool.com',
  :address => 'smtp.sendgrid.net',
  :port => 587,
  :authentication => :plain,
  :enable_starttls_auto => true
}
```

And voila! That's it. Try it out!

Happy emailing!


![email]({{ site.url }}/assets/170624_email.gif)
<br><br>



There is great documentation for ActionMailer and SendGrid:  
[ActionMailer](http://guides.rubyonrails.org/action_mailer_basics.html)  
[SendGrid with Heroku](https://devcenter.heroku.com/articles/sendgrid)
