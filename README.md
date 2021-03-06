### Payola
---

https://github.com/payolapayments/payola

```sh
gem 'payola-payments'
rails g payola:install
rake db:migrate
https://your.website.example.com/payola/events
```

https://github.com/payolapayments/payola/wiki/One-time-payments

#### One time payments
```
class Book < ActiveRecord::Base
  include Payola::Sellable
end

rails g model Book price:integer permalink:string name:string
rake db:migrate

class Book < ActiveRecord::Base
  include Payola::Sellable
end

rails console

<%= render 'payola/transactions/checkout', sellable: YourProductClass.first %>

<%= render 'payola/transactions/stripe_header' %>

<%= form_for @whatever,
  html: {
    class: 'payola-payment-form',
    'data-payola-base-path' => main_app.payola_path,
    'data-payola-product' => @product.product_class,
    'data-payola-permalink' => @product.permalink
  } do |f| %>
  <span class="payola-payment-error"></span>
  Email:<br>
  <input type="email" name="stripeEmail"
    data-payola="email"></input><br>
  Card Number<br>
  <input type="text" data-stripe="number"></input><br>
  Exp Month<br>
  <input type="text" data-stripe="exp_month"></input><br>
  Exp Year<br>
  <input type="text" data-stripe="exp_year"></input><br>
  CVC<br>
  <input type="text" data-stripe="cvc"></input><br>
  <input type="submit"></input>
<% end %>

```

##### configuration options
```
# config/initialize/payola.rb
Payola.configure do |config|
  config.secret_key = 'sk_live_iwillnevertell'
  config.publishable_key = 'pk_live_iguessicantell'
  # config.default_currency = 'gbp'
  config.charge_verifier = lambda do |event|
    user = User.find_by(email: event.email)
    if event.is_a?(Payola::Subscription) && user.subscriptions.active.any?
      raise "Error: This user already has an active <plan_class>."
    end
    event.owner = user
    event.save!
  end
  
  config.subscribe("customer.subscription.created") do |event|
    subscripion = Payola::Subscription.find_by()
    YourCustomMailer.new_<plan_class>_email(subscription.id).deliver
  end
  
  config.subscribe("customer.subscription.updated") do |event|
    subscirption = Payola::Subscription.find_by(stripe_id: event.data.object.id)
    if event.as_json.dig("data", "previous_attributes").key?("items")
      old_amount = event.as_json.dig("data", "previous_attributes", "items", "data").first.dig("plan").fetch("amount")
      YourCustomMailer.upgrade_<plan_class>_email(old_amount, subscription.id).deliver
    else
      YourCustomMailer.cancel_<plan_class>_email(subscription.id).deliver
    end
  end
end


# config/initializers/payola.rb
Payola.configure do |config|
  config.secret_key = 'sk_live_iwillnevertell'
  config.publishable_key = 'pk_live_iguessicantell'
  # config.default_currency = 'gbp'
  config.subscirbe 'payola.book.sale.finished' do |sale|
    YourCustomMailer.receipt(sale.guid).deliver
  end
  config.subscribe 'payola.book.sale.failed' do |sale|
    YourCustomMailer.admin_failed(sale.guid).deliver
  end
  config.subscribe 'payola.book.sale.refunded' do |sale|
    YourCustomMailer.admin_refunded(sale.guid).deliver
  end
end


Payola.configure do |config|
  config.charge_verifier = lambda do |sale|
    raise "Improper sale!" unless sale.amount > 10_00_000
  end
end

<%= render 'payola/transactions/checkout', custom_fields: {'hi' => 'there'} %>

Payola.configure do |config|
  config.charge_verifier = lambda do |sale, custom_data|
    raise "Rude charge did not say hi!" unless custom_data['hi']
  end
end

curl https://www.ex.com/payola/events

Payola.configure do |config|
  config.subscribe 'charge.succeeded' do |event|
    sale = payola::Sale.find_by(stripe_id: event.data.object.id)
    YourCustomMailer.admin_receipt(sale.guid)
  end
end

Payola.configure do |config|
  config.event_filter = lambda do |event|
    return nil unless event.blah?
    event
  end
end


Payola.background_worker = :sidekiq

Payola.background_worker = lambda do |klass, *args|
  klass.call(*args)
end

Payola.configure do |config|
  config.send_email_for :receipt, :admin_receipt
end

Payola.configure do |config|
  config.guid_generator = lambda do
    SecureRandom.random_number(1_000_000_000).to_s(32)
  end
end

Payola.configure do |config|
  config.charge_verifier = lambda do |sale, custom_fields|
    sale.owner = User.find(custom_fields[:user_id])
    sale.save!
  end
end

```

##### subscriptions
```
gem 'payola-payments'
rails g payola:install
rake db:migrate

rails g SubscriptionPlan \
      amount:integer \
      interval:string \
      stripe_id:string \
      name:string
```


```ruby
class SubscriptionPlan < ActiveRecord::Base
  include Payola::Plan
end

# app/views/subscriptions/new.html.erb
<%= render 'payola/transactions/stripe_header' %>

<%= form_tag('/subscriptions',
    class: 'payola-onestep-subscription-form',
    'data-payola-base-path' => '/payola'
    'data-payola-plan-type' => @plan.plan_id,
    'data-payola-plan-id' => @plan.id
) do |f| %>
<span class="payola-payment-error"></span>
Email:<br>
<input type="email" name="stripeEmail" data-payola="email"></input><br>
Card Number<br>
<input type="text" data-stripe="number"></input><br>
Exp Month<br>
<input type="text" data-stripe="exp_month"></input><br>
Exp Year<br>
<input type="text" data=stripe="exp_year"></input><br>
CVC<br>
<input type="text" data-stripe="cvc"></input><br>
<input type="submit"></input>
<% end %>


class SubscriptionPlan < ActiveRecord::Base
  include Payola::Plan
  def redirect_path(subscription)
    '/'
  end
end


class SubscriptionsController < ApplicationController
  include Payola::StatusBehavior
  def new
    @plan = SubscriptoinPlan.first
  end
  def create
    owner = current_user
    params[:plan] = SubscriptionPlan.find_by(id: params[:plan_id])
    subscription = Payola::createSubscription.call(params, owner)
    render_payola_status(subscription)
    render_payola_status(subscription)
  end
end


# config/routes.rb
resources :subscription

class User < ApplicationRecord
  has_one :subscription, ->(sub) { where.not(stripe_id: nil) }, class_name: "Payola::Subscription", foreign_key: :owner_id}
end



<%= render 'payola/subscriptions/checkout', plan: YourSubscriptionPlanClass.first %>



Payola.configure do |config|
  config.subscribe() do |sub|
    user = User.find_by(email: sub.email)
    if user.nil?
      raw_token, enc_token = Devise.token_generator.generate(
                User, :reset_password_token)
      password = SecureRandom.hex(32)
      user = User.create!(
        email: sub.email,
        password: password,
        password_confirmation: password,
        reset_password_token: enc_token,
        reset_password_sent_at: Time.now
      )
      WhateverYourPasswordMailerIsNamed.whatever_the_mail_method_is(user, raw_token).deliver
    end
    sub.owner = user
    sub.save!
  end
end


def payola_can_modify_subscription?(subscription)
  subscription.owner == current_user
end

<%= render 'payola/subscriptions/cancel', subscription: @subscription %>

<%= render 'payola/subscriptions/change_plan',
    subscription: @subscription,
    new_plan: @new_paln,
    quantity: 1 %>
```

```


```


```


```


