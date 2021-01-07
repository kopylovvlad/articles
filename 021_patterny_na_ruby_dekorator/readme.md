# Паттерны на ruby: Декоратор

**Описание:**

Данный паттерн предназначенный для динамического подключения дополнительного поведения к объекту.
Декоратор предоставляет гибкую альтернативу практике создания подклассов с целью расширения функциональности.

**Пример задачи:**

Есть класс <Robot> со своим ограниченным набором атрибутов.
В различных частях приложения от него требуются некоторые дополнительные возможности. При этом класс <Robot> переписать нельзя.

**Реализация:**

Для реализации данной задачи создадим два класса-декоратора <RobotFirstDecorator>, <RobotSecondDecorator> и наследуем их от <SimpleDelegator>. Благодаря чему мы может в любой момент подключать дополнительное поведения для класса <Robot>.

```ruby
class Robot
  attr_reader :model_name, :id, :release_date
  def initialize(model_name:, id:, release_date:)
    @model_name = model_name
    @id = id
    @release_date = release_date
  end
end

class RobotFirstDecorator < SimpleDelegator
  def age
    ((Time.now - release_date) / 31557600).floor
  end
end

class RobotSecondDecorator < SimpleDelegator
  def full_name
    "#{id} => #{model_name}"
  end
end

robot = Robot.new(
  release_date: Time.new(2015, 3, 2, 3, 0, "+03:00"),
  id: "934kawrr22",
  model_name: "IOI"
)

robot_decorator = RobotFirstDecorator.new(robot)
robot_decorator = RobotSecondDecorator.new(robot_decorator)

puts robot_decorator.class
# RobotSecondDecorator
puts robot_decorator.id
# 934kawrr22
puts robot_decorator.model_name
# IOI
puts robot_decorator.full_name
# 934kawrr22 => IOI
puts robot_decorator.release_date
# 2015-03-02 03:00:03 +0300
puts robot_decorator.age
# 2
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%B4%D0%B5%D0%BA%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80-db3623a5c2d3)
