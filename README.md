websocket-actioncable-client
=======================
Simple WebSocket Client for Ruby. It can work with ActionCable server

- https://github.com/shokai/websocket-client-simple
- https://rubygems.org/gems/websocket-client-simple


Installation
------------

    gem install websocket-client-simple


Usage
-----
```ruby
require 'rubygems'
require 'websocket-client-simple'
require 'json'

ws = WebSocket::Client::Simple.connect 'ws://example.com:3000/cable'

ws.on :message do |msg|
  puts msg.data
end

ws.on :open do
  # Send command to subscribe to ActionCable's specific channel after connect to socket
  ws.send ({"command":"subscribe","identifier":"{\"channel\":\"LocationChannel\",\"request_id\":\"b03e65bc-994c-d1e6-cd71-51b7a86242d8\"}"}.to_json)
end

ws.on :close do |e|
  p e
  exit 1
end

ws.on :error do |e|
  p e
end

loop do
  ws.send STDIN.gets.strip
end
```

`connect` runs a given block before connecting websocket

```ruby
WebSocket::Client::Simple.connect 'ws://example.com:8888' do |ws|
  ws.on :open do
    puts "connect!"
  end

  ws.on :message do |msg|
    puts msg.data
  end
end
```



## The Action Cable Protocol

There really isn't that much to this gem. :-)

1. Connect to the Action Cable URL
2. After the connection succeeds, send a subscribe message
  - The subscribe message JSON should look like this
    - `{"command":"subscribe","identifier":"{\"channel\":\"MeshRelayChannel\"}"}`
  - You should receive a message like this:
    - `{"identifier"=>"{\"channel\":\"MeshRelayChannel\"}", "type"=>"confirm_subscription"}`
3. Once subscribed, you can send messages.
  - Make sure that the `action` string matches the data-handling method name on your ActionCable server.
  - Your message JSON should look like this:
    - `{"command":"message","identifier":"{\"channel\":\"MeshRelayChannel\"}","data":"{\"to\":\"user1\",\"message\":\"hello from user2\",\"action\":\"chat\"}"}`
    - Received messages should look about the same

4. Notes:
  - Every message sent to the server has a `command` and `identifier` key.
  - Ping messages from the action cable server look like:
    - `{ "type" => "ping", "message" =>  1461845503 }`
  - The channel value must match the `name` of the channel class on the ActionCable server.
  - `identifier` and `data` are redundantly jsonified. So, for example (in ruby):
```ruby
payload = {
  command: 'command text',
  identifier: { channel: 'MeshRelayChannel' }.to_json,
  data: { to: 'user', message: 'hi', action: 'chat' }.to_json
}.to_json
```


Contributing
------------
1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
