# NTLM authentication with Ruby and Faraday

<!-- Photo by <a href="https://unsplash.com/@philberndt?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Philipp Berndt</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a> -->

![Photo by Philipp Berndt on Unsplash](image01.jpg)

Everyone knows how HTTP basic authentication works. A lot of services use it often. Once I happened upon with NTLM auth. Getting access was challenging, but I coped with it. After working on the task I would like to share the solution how to work with NTLM and Faraday library.

For instance, with `curl` you can specify the user name and password for server authentication. The command looks like this: `curl -u <USERNAME>:<PASSWORD> https://resource_with_ntlm_auth.com/path_to_resource.xml -I`. Unfortunately it doesn't work with NTLM. The response would be:

```
HTTP/1.1 401 Unauthorized
Connection: Keep-Alive
Content-Length: 1292
Date: Tue, 01 Nov 2021 12:00:00 GMT
Content-Type: text/html
Server: Microsoft-IIS/10.0
WWW-Authenticate: Negotiate
WWW-Authenticate: NTLM
X-Powered-By: ASP.NET
```

In order to authenticate by curl, we have to use --ntlm flag. `curl -u <USERNAME>:<PASSWORD> --ntlm https://resource_with_ntlm_auth.com/path_to_resource.xml -I`

The response is:

```
HTTP/1.1 401 Unauthorized
Connection: Keep-Alive
Content-Length: 341
Date: Tue, 01 Nov 2021 12:00:00 GMT
Content-Type: text/html; charset=us-ascii
Server: Microsoft-HTTPAPI/2.0
WWW-Authenticate: NTLM <SECRET_STRING>

HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Length: 321981
Date: Tue, 01 Nov 2021 12:00:00 GMT
Content-Type: text/xml
ETag: "d783128d33d5d22"
Server: Microsoft-IIS/10.0
Accept-Ranges: bytes
Last-Modified: Tue, 01 Nov 2021 10:00:00 GMT
Persistent-Auth: true
X-Powered-By: ASP.NET
```

Noticed that we have two responses? 😅 It's the feature of NTLM.

So, how to authenticate via Ruby, not using curl?

There is a library on GitHub [ruby-ntlm](https://github.com/macks/ruby-ntlm). Looking into the code you can see an [example](https://github.com/macks/ruby-ntlm/blob/master/examples/http2.rb) of how to authenticate. Even so, I don't like [the solution](https://github.com/macks/ruby-ntlm/blob/master/lib/ntlm/http.rb) because it monkey patches `Net::HTTP` class.

It's better to see another [example](https://github.com/macks/ruby-ntlm/blob/master/examples/http.rb) that uses `Net::HTTP#start`. Looking into the source code, we have noticed that NTLM auth is tricky because it requires that the connection must be keep-alive.

```ruby
require 'ntlm'
require 'net/http'

Net::HTTP.start('www.example.com') do |http|
  request = Net::HTTP::Get.new('/')
  request['authorization'] = 'NTLM ' + NTLM.negotiate.to_base64

  response = http.request(request)

  # The connection must be keep-alive!

  challenge = response['www-authenticate'][/NTLM (.*)/, 1].unpack('m').first
  request['authorization'] = 'NTLM ' + NTLM.authenticate(challenge, 'User', 'Domain', 'Password').to_base64

  response = http.request(request)

  p response
  print response.body
end
```

`Net::HTTP` is good. Even so, how to authenticate with popular gem, for instance [Faraday](https://github.com/lostisland/faraday)? I started to figure out how to implement the protocol with Faraday 🤔

As the NTLM auth algorithm requires to keep the connection alive, we have to use the gem [net-http-persistent](https://github.com/drbrain/net-http-persistent). Also we have to use the [ruby-ntlm](https://github.com/macks/ruby-ntlm) gem because coding/encoding [logic](https://github.com/macks/ruby-ntlm/blob/master/lib/ntlm/message.rb) is tricky.

After spending some time I have found the solution. The solution is here:

```ruby
require 'faraday'
require 'ntlm'
# It requires gems 'ruby-ntlm' and 'net-http-persistent'

connection = Faraday.new('https://resource_with_ntlm_auth.com') do |f|
  # The connection must be keep-alive
  f.adapter :net_http_persistent
end
response = connection.get do |req|
  req.url '/path_to_resource.xml'
  req.headers['Content-Type'] = 'application/xml'
  req.headers['authorization'] = 'NTLM ' + NTLM::Message::Negotiate.new({}).to_base64
end
puts response.status # we received 401 status

challenge = response['www-authenticate'][/NTLM (.*)/, 1].unpack1('m')
response = connection.get do |req|
  req.url '/path_to_resource.xml'
  req.headers['Content-Type'] = 'application/xml'
  req.headers['authorization'] = 'NTLM ' + NTLM.authenticate(challenge, ENV['USER'], nil, ENV['PASSWORD']).to_base64
end
puts response.status # we have got 200 status 🥳
```

That's all! I hope the article will be useful for you.

[dev.to](https://dev.to/kopylov_vlad/ntlm-authentication-with-ruby-and-faraday-3lin)
