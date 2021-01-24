# Паттерны на ruby: Одиночка

**Описание:**

Данный паттерн гарантирует что у класса будет только один экземпляр. Чаще всего это полезно для доступа к какому-то общему ресурсу, например, базе данных.

```ruby
class Database
  @@instance = Database.new

  def self.instance
    return @@instance
  end

  private_class_method :new
end

d1 = Database.instance
d2 = Database.instance
d2.object_id == d1.object_id # true
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%BE%D0%B4%D0%B8%D0%BD%D0%BE%D1%87%D0%BA%D0%B0-7879ae1ece3)
