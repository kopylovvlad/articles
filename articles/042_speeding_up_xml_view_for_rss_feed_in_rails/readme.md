# Speeding up xml view for RSS feed in Rails

Recently, I tried to make a RSS feed faster in my Rails application. I had a Rails application with standard xml.builder view wrote with [builder gem](https://github.com/jimweirich/builder).

Example:

```ruby
xml.instruct! :xml, :version => "1.0"
xml.rss :version => "2.0" do
  xml.channel do
    xml.title "My Company Blog1"
    xml.description "This is a blog by My Company"
    xml.link '/'
    @topics.each do |i|
      xml.item do
        xml.title i.title
        xml.subtitle i.subtitle
        xml.description i.description
        xml.big_text i.big_text
        xml.rubric_id i.rubric_id
        xml.tag_id i.tag_id
        xml.publication_at i.publication_at
        xml.one_more_time i.one_more_time
      end
    end
  end
end
```

It is absolutely standard xml view for all RSS feeds in Rails. I began an experiment to rewrite it from xml.bulder to xml.erb and it made render faster. For my app, it made it twice as fast as before.

Example:

```erb
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
  <channel>
  <title><%= 'My Company Blog1' %></title>
  <description><%= 'This is a blog by My Company' %></description>
  <link><%= '/' %></link>
  <% @topics.each do |i| %>
    <item>
      <title><%= i.title %></title>
      <subtitle><%= i.subtitle %></subtitle>
      <description><%= i.description %></description>
      <big_text><%= i.big_text %></big_text>
      <rubric_id><%= i.rubric_id %></rubric_id>
      <tag_id><%= i.tag_id %></tag_id>
      <publication_at><%= i.publication_at %></publication_at>
      <one_more_time><%= i.one_more_time %></one_more_time>
    </item>
  <% end %>
</channel>
</rss>
```

Almost the same result we can reach to replace gem builder by [ox](https://github.com/ohler55/ox) (the fastest optimized XML parser and generator). But, unfortunately, it has quirky syntax and we don’t have action_view’s template out the box.

In conclusion, there is an easy way to decrease rss response time. I prepared a test application in order to check it out and uploaded it to Github. There is the link to test application with the result — [github](https://github.com/kopylovvlad/rails_rss_app).

[Medium](https://kopilov-vlad.medium.com/speeding-up-xml-view-for-rss-feed-in-rails-f70f65024626)
