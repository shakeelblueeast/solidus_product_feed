# Solidus Product Feed

[![CircleCI](https://circleci.com/gh/solidusio-contrib/solidus_product_feed.svg?style=svg)](https://circleci.com/gh/solidusio-contrib/solidus_product_feed)

An extension that provides an RSS feed for products. Google Merchant Feed attributes are also
implemented.

## Installation

Add `solidus_product_feed` to your Gemfile:

```ruby
gem 'solidus_product_feed'
```

Bundle your dependencies and run the installation generator:

```shell
bundle
bundle exec rails g solidus_product_feed:install
```

You're done! You can now see your RSS feed at `/products.rss`.

## Usage

The feed ships with sensible defaults for your products, but customization is very easy.

### Headers

You can easily change the feed's headers by putting the following in your Rails initializer:

```ruby
SolidusProductFeed.configure do |config|
  config.title = 'My Awesome Store'
  config.link = 'https://www.awesomestore.com'
  config.description = 'Find out about new products on https://www.awesomestore.com first!'
  config.language = 'en-us'
end
```

Note that you can also pass a Proc for each of these options. The Proc will be passed the view
context as its only argument, so that you can use all your helpers:

```ruby
SolidusProductFeed.configure do |config|
  config.title = -> (view) { view.current_store.name }
  config.link = -> (view) { "http://#{view.current_store.url}" }
  config.description = -> (view) { "Find out about new products on http://#{view.current_store.url} first!" }
  config.language = -> (view) { view.lang_from_store(current_store.language) }
end
```

### Item schema

If you need to alter the XML schema of a product (e.g. to add/remove a tag), you can do it by
subclassing `Spree::FeedProduct` in your app and overriding the `schema` method:

```ruby
module AwesomeStore
  class FeedProduct < Spree::FeedProduct
    def schema
      super.merge('g:brand' => 'Awesome Store Inc.')
    end
  end
end
```

Then set your custom class in an initializer:

```ruby
SolidusProductFeed.configure do |config|
  config.feed_product_class = 'AwesomeStore::FeedProduct'
end
```

If you want to change the value of an existing tag, you can also simply override the corresponding
tag method (`link`, `price` etc.). Check the [source code](https://github.com/solidusio-contrib/solidus_product_feed/blob/master/app/models/spree/feed_product.rb)
for more details.

### Making your feed discoverable

If you want to make your feed discoverable when visiting your store, simply add the following to
your storefront's `head`:

```html
<%= auto_discovery_link_tag(:rss, products_path(format: :rss), title: "My Store's Products") %>
```

## Development

### Testing the extension

First bundle your dependencies, then run `bin/rake`. `bin/rake` will default to building the dummy
app if it does not exist, then it will run specs. The dummy app can be regenerated by using
`bin/rake extension:test_app`.

```shell
bundle
bin/rake
```

To run [Rubocop](https://github.com/bbatsov/rubocop) static code analysis run

```shell
bundle exec rubocop
```

When testing your application's integration with this extension you may use its factories.
Simply add this require statement to your spec_helper:

```ruby
require 'solidus_product_feed/factories'
```

### Running the sandbox

To run this extension in a sandboxed Solidus application, you can run `bin/sandbox`. The path for
the sandbox app is `./sandbox` and `bin/rails` will forward any Rails commands to
`sandbox/bin/rails`.

Here's an example:

```shell
$ bin/rails server
=> Booting Puma
=> Rails 6.0.2.1 application starting in development
* Listening on tcp://127.0.0.1:3000
Use Ctrl-C to stop
```

### Releasing new versions

Your new extension version can be released using `gem-release` like this:

```shell
bundle exec gem bump -v VERSION --tag --push --remote upstream && gem release
```

## License

Copyright (c) 2011 Joshua Nussbaum and other contributors, released under the New BSD License.
