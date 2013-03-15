---
layout: post
title: "Getting OpenID working on Heroku"
date: 2013-03-14 20:13
comments: true
categories:
---

*by [Matt Rogish](http://www.mattrogish.com/) ([@MattRogish](https://twitter.com/MattRogish))*

I just spent the last few days wrestling with OpenID intermittently failing on production, but not test, development, or staging.

It took me a bit of time to fix, so I thought I'd enumerate the steps.

1. Use [Unicorn](https://blog.heroku.com/archives/2013/2/27/unicorn_rails)
1. Use [MemCachier](https://devcenter.heroku.com/articles/memcachier)
1. Use [Dalli](https://github.com/mperham/dalli)
1. Use [ruby-openid](https://github.com/openid/ruby-openid)
1. Configure OpenID to use Dalli:

(set :expires_in to taste)

{% codeblock lang:ruby %}
    ::OpenID::Consumer.new(session,
        OpenID::Store::Memcache.new(Dalli::Client.new(ENV['MEMCACHIER_SERVERS'],
                               username: ENV['MEMCACHIER_USERNAME'],
                               password: ENV['MEMCACHIER_PASSWORD'],
                               expires_in: 300)))
{% endcodeblock %}

1. If you are using [rack-openid](https://github.com/josh/rack-openid)

(set :expires_in to taste)

{% codeblock lang:ruby %}
    config.middleware.use "Rack::OpenID",
      OpenID::Store::Memcache.new(Dalli::Client.new(ENV['MEMCACHIER_SERVERS'],
                             username: ENV['MEMCACHIER_USERNAME'],
                             password: ENV['MEMCACHIER_PASSWORD'],
                             expires_in: 300))
{% endcodeblock %}

That's it!

P.S.

The reason why it worked on development and test: we only had a single Unicorn running, so memory storage (the default) worked fine. Staging is running more than one dyno, but since the load was so small it hit the same dyno more often than not, causing it to appear to work when it wasn't really.

P.P.S.

You may see guides on the internet that are a few years old suggesting to use filesystem storage:

`OpenID::Store::Filesystem.new('./tmp')`

This would only work if you use a single [dyno](https://devcenter.heroku.com/articles/dynos) as the filesystem is not shared amongst dynos. Stick with memcached!