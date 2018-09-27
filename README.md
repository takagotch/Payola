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


```

```









