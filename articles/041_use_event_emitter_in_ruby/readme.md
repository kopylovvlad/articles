# Use Event emitter in Ruby

![Photo by Nathan Dumlao on Unsplash](image01.jpeg)

## What is Event emitter?

Event emitter is so close to the observer pattern. It works the same: something emits an event, another object listens the event and execute the code. Event emitter triggers an event which everyone can listen. Various libraries have different implementation, but the basic idea is a tool for issuing events and subscribing to them.

How to work with it?

There is an simple implementation of code. Just a module that can be included to any class.

```ruby
# the module can be included to any class
module EventEmitterCore
  # turn on the event
  # @param event_name [String, Symbol]
  def on(event_name)
    events[event_name.to_sym] ||= []
  end

  # turn off the event
  # @param event_name [String, Symbol]
  def off(event_name)
    events.delete(event_name.to_sym)
  end

  # subscribe to event
  # @param event_name [String, Symbol]
  # @param handler_proc [Proc]
  #   Proc with [Symbol, Object]
  def subscribe(event_name, handler_proc)
    handler_id = "#{event_name}_#{handler_proc.object_id}"
    events[event_name.to_sym]&.push(
      { id: handler_id, proc: handler_proc }
    )
    handler_id
  end

  # unsubscribe to event
  # @param event_name [String, Symbol]
  # @param handler [Proc]
  #   Proc with [Symbol, User]
  def unsubscribe(event_name, handler_id)
    events[event_name.to_sym]&.reject! do |item|
      item[:id] == handler_id
    end
  end

  # emit the event
  # @param event_name [String, Symbol]
  def emit(event_name)
    events[event_name.to_sym]&.each do |h|
      h[:proc].call(event_name.to_sym, self)
    end
  end

  # get array of existing events
  # @return [Array<Symbols>]
  def all_events
    events.keys
  end

  # get array of existing events with stat
  # @return [Array<Symbols, Fixnum>]
  def all_events_with_stat
    events
      .map { |name, arr| [name, arr.size] }
      .flatten
  end

  private

  def events
    @events ||= {}
  end
end
```

Here the easiest example. We implement code that will print the line ‘ring ring ring’, then we create an event and subscribe code to the event.

```ruby
require_relative './event_emitter_core'

class EventEmitter
  include EventEmitterCore
end

event_emitter = EventEmitter.new

# the code should be executed when event will be triggered
ring_bell = lambda do |event_name, object|
  puts 'ring ring ring'
end

# create an event 'doorOpen'
event_emitter.on(:doorOpen)

# and subscribe code to event
event_emitter.subscribe(:doorOpen, ring_bell)

# to trigger event
event_emitter.emit(:doorOpen)
# 'ring ring ring'
```

## Where should I use the pattern?

Everywhere when you have subscribing feature. For example: chat application (many users wait up new messages); answers and questions web-sites (many users subscribe to questions); real-time application where people subscribe to updates; notifications about software update.

Here the example of chat app implementation with Event emitter.

```ruby
require_relative './event_emitter_core'

# class for users
class User
  attr_reader :first_name

  def initialize(first_name)
    @first_name = first_name
  end
end

# service object for notification about new messages
class Notificator
  def self.call(message, user)
    # notify user about new message
    puts "new message '#{message}' for user #{user.first_name}"
  end
end

# class for chats
class Chat
  include EventEmitterCore
  def initialize(title, creator)
    @title = title
    @messages = []
    on(:new_message)

    add_subscriber(creator)
  end

  # to add a subscriber to the chat
  def add_subscriber(user)
    notify_user = lambda do |event_name, chat_object|
      Notificator.call(chat_object.last_message, user)
    end
    subscribe(:new_message, notify_user)
  end

  # to delete a subscriber from the chat
  def del_subscriber(user, hander_id)
    unsubscribe(:new_message, hander_id)
    new_message("#{user.first_name} left the chat", nil)
  end

  # to add new messages to the chat
  def new_message(message, user)
    @messages << { text: message, from: user&.first_name }
    emit(:new_message)
  end

  # to get the last message
  def last_message
    @messages.last[:text]
  end
end

# user Sasha create new chat
sasha = User.new('sasha')
chat = Chat.new("invitation for Sasha's birthday", sasha)
chat.new_message('test', sasha)

# users Ivan and Vova subscribe to the chat
ivan = User.new('ivan')
vova = User.new('vova')
ivan_handler_id = chat.add_subscriber(ivan)
vova_handler_id = chat.add_subscriber(vova)

# Sasha write a message
chat.new_message('hello, everyone. I invite you to my birthday party', sasha)

# unfortunately, Vova leave the chat
chat.del_subscriber(vova, vova_handler_id)
chat.new_message('Ohhh, Vova left the chat :(', sasha)

#### output is
# new message 'test' for user sasha
# new message 'hello, everyone. I invite you to my birthday party' for user sasha
# new message 'hello, everyone. I invite you to my birthday party' for user ivan
# new message 'hello, everyone. I invite you to my birthday party' for user vova
# new message 'vova left the chat' for user sasha
# new message 'vova left the chat' for user ivan
# new message 'Ohhh, Vova left the chat :(' for user sasha
# new message 'Ohhh, Vova left the chat :(' for user ivan
```

Of course, you don’t need to implement it by yourself. There are some great implementations in Github. If you want to have some practice, you could write it from scratch. It is so funny.

* [event_emitter](https://github.com/shokai/event_emitter)
* [emittr](https://github.com/talyssonoc/emittr)

[Medium](https://kopilov-vlad.medium.com/use-event-emitter-in-ruby-6b289fe2e7b4)
