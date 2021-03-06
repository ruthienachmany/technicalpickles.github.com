--- 
title: Shoulda can automatically load custom macros
tags: 
- rails
- ruby
- shoulda
- testing
layout: post
---
I wrote about [creating your own shoulda macros awhile back](/posts/shoulda-macros-allows-you-to-embrace-your-inner-slacker). At that point, I was suggesting to place these in your `test/test_helper.rb`.

Now it's even easier, thanks to a [semi recent change](http://thoughtbot.lighthouseapp.com/projects/5807/tickets/62).

You can now create put `.rb` files in `test/shoulda_macros`, and [Shoulda](http://www.thoughtbot.com/projects/shoulda) will automatically pick them up. Let's try making a [will\_paginate](http://github.com/mislav/will_paginate/) macro, and stick it out in `test/shoulda_macros/will_paginate_macros.rb`:

{% highlight ruby %}
module WillPaginateMacros
  def should_have_per_page(count)
    klass = self.name.gsub(/Test$/, '').constantize
    context "#{klass}" do
      should "respond to per_page" do
        assert klass.respond_to?(:per_page), "#{klass} does not respond to :per_page"
      end
      
      should "have #{count} per page" do
        assert_equal count, klass.per_page
      end
    end
  end
end

class Test::Unit::TestCase
  extend WillPaginateMacros
end
{% endhighlight %}

In the test of an imaginary `Post` model, which I'm sure you can picture, we can do:

{% highlight ruby %}
require File.dirname(__FILE__) + '/../test_helper'

class UserTest < ActiveSupport::TestCase
  # ...
  should_have_per_page 10
end
{% endhighlight %}

[BOOYAH!](http://www.poetv.com/video.php?vid=25029)

For plugin and gem authors, `vendor/gems/*/shoulda_macros/*.rb` and `vendor/plugins/*.rb` will also be automatically picked up.
