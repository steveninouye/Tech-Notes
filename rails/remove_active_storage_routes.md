Here's how you get rid of those `active_storage` routes if you don't want them to show up when you type `bundle exec rails routes`.

1.  In your `config/application.rb` file, change the following:

```
# require 'rails/all'

require "rails"

# Include each railties manually, excluding `active_storage/engine`
%w(
  active_record/railtie
  action_controller/railtie
  action_view/railtie
  action_mailer/railtie
  active_job/railtie
  action_cable/engine
  rails/test_unit/railtie
  sprockets/railtie
).each do |railtie|
  begin
    require railtie
  rescue LoadError
  end
end
```

2.  In your `development.rb` file, remove this:
`config.active_storage.service = :local`

3. In your `application.js` file, remove this:
`//= require activestorage`

**************

In order to test using Postman we will need to disable CSRF protection. Add the line config.action_controller.default_protect_from_forgery = false right underneath the line config.load_defaults 5.2 in config/application.rb. Remember, this should only be done while we are working in development.

