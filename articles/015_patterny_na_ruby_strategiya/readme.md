# Паттерны на ruby: Стратегия

**Описание:**

Данный паттерн применяется в случае если входные данные/представление/вывод данных — одни и те же; логика обработки данных перед отдачей в представление — разное.
Краткий смысл паттерна — поместить алгоритмы/логику в отдельные объекты.

**Пример задачи:**

Нужно реализовать функционал шифрования текстового сообщения разными алгоритмами.

**Реализация:**

Создадим класс для текстового сообщения <TextMessage> который при инициализации принимает текстовое сообщение.
Сделаем этому классу атрибут .encryptor, с возможностью менять его значение налету.
Плюс добавим метод .encrypt_me для шифрования, который будет возвращать строку. Этот метод будет проверяет есть ли у значения атрибута .encryptor возможности вызова метода .encrypt, и если есть то вызывает его и передает наше сообщение.
В качестве алгоритмов шифрования создадим классы <SimpleEncryptor>, <Base64Encryptor> и <AESEncryptor> у каждого из которых будет метод .encrypt.

```ruby
require 'base64'
require 'aes'

class TextMessage
  attr_accessor :encryptor
  attr_reader :original_message

  def initialize(original_message, encryptor)
    @original_message = original_message
    @encryptor = encryptor
  end
  def encrypt_me
    encryptor.encrypt(self)
  end
end

class SimpleEncryptor
  ALPHABET = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
  ENCODING = 'MOhqm0PnycUZeLdK8YvDCgNfb7FJtiHT52BrxoAkas9RWlXpEujSGI64VzQ31w'
  def encrypt(context)
    context.original_message.tr(ALPHABET, ENCODING)
  end
end

class Base64Encryptor
  def encrypt(context)
    Base64.encode64(context.original_message)
  end
end

class AESEncryptor
  KEY = 'dsfi434n534df0v0bn23324dfgdfgdf4353454'
  def encrypt(context)
    AES.encrypt(context.original_message, KEY)
  end
end

message = TextMessage.new('my secret secret message', SimpleEncryptor.new)
puts message.encrypt_me
# eb vmhYmD vmhYmD emvvMPm

message.encryptor = Base64Encryptor.new
puts message.encrypt_me
# bXkgc2VjcmV0IHNlY3JldCBtZXNzYWdl

message.encryptor = AESEncryptor.new
puts message.encrypt_me
# 0kr326hFWodI9Xzd42xKBg==$eb6N+Xz4yJngCndUPYVJ7CIEHngqNTG2lFDL1vIocEw=
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D1%81%D1%82%D1%80%D0%B0%D1%82%D0%B5%D0%B3%D0%B8%D1%8F-dc4917887fe8)
