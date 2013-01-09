# Strong Parameters

With this plugin Action Controller parameters are forbidden to be used in Active Model mass assignments until they have been whitelisted. This means you'll have to make a conscious choice about which attributes to allow for mass updating and thus prevent accidentally exposing that which shouldn't be exposed.

In addition, parameters can be marked as required and flow through a predefined raise/rescue flow to end up as a 400 Bad Request with no effort.

```ruby
class PeopleController < ActionController::Base
  # This will raise an ActiveModel::ForbiddenAttributes exception because it's using mass assignment
  # without an explicit permit step.
  def create
    Person.create(params[:person])
  end

  # This will pass with flying colors as long as there's a person key in the parameters, otherwise
  # it'll raise a ActionController::MissingParameter exception, which will get caught by
  # ActionController::Base and turned into that 400 Bad Request reply.
  def update
    person = current_account.people.find(params[:id])
    person.update(person_params)
    redirect_to person
  end

  private
    # Using a private method to encapsulate the permissible parameters is just a good pattern
    # since you'll be able to reuse the same permit list between create and update. Also, you
    # can specialize this method with per-user checking of permissible attributes.
    def person_params
      params.require(:person).permit(:name, :age)
    end
end
```

You can also use permit on nested parameters, like:

```ruby
params.permit(:name, friends: [ :name, { family: [ :name ] }])
```

Thanks to Nick Kallen for the permit idea!

## Installation

In Gemfile:

    gem 'strong_parameters'

and then run `bundle`. To activate the strong parameters, you need to include this module in
every model you want protected.

```ruby
class Post < ActiveRecord::Base
  include ActiveModel::ForbiddenAttributesProtection
end
```

If you want to now disable the default whitelisting that occurs in later versions of Rails, change the `config.active_record.whitelist_attributes` property in your `config/application.rb`:

```ruby
config.active_record.whitelist_attributes = false
```

This will allow you to remove / not have to use `attr_accessible` and do mass assignment inside your code and tests.

## Compatibility

This plugin is only fully compatible with Rails versions 3.0, 3.1 and 3.2 but not 4.0+, as it is part of Rails Core in 4.0.
