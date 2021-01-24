# Паттерны на ruby: Команда

**Описание:**

Есть какой то объект с которым производятся манипуляции.
Нужно предоставить функционал ставить действия по манипуляции в очередь, логировать их или откатывать.
В таких случаях полезен паттерн Команда так как он превращает операцию в самостоятельный объект.

**Пример задачи:**

Приложение по манипуляции журналом сотрудников. С журналом можно делать следующие операции: добавления сотрудника,
удаления сотрудника, изменение адреса сотрудника, поиска и выгрузки всех данных. В случае ошибки, операции можно откатить.

**Реализация:**

Есть класс <EmployeeJournal> — это синглтон который может проводить необходимые манипуляции.
Есть абстрактный класс <AbstractCommand> от которого наследуются команды для какого то конкретного случая.
У каждого наследника <AbstractCommand> есть методы .execute, .unexecute для запуска манипуляции и отката назад.

```ruby
require 'singleton'

class Employee
  attr_accessor :name, :number, :address

  def initialize(args = {})
    @name = args[:name]
    @number = args[:number]
    @address = args[:address]
  end

  def to_s
    "Employee: name: #{name} number: #{number} address: #{address}"
  end

  def to_json
    {
      name: name,
      number: number,
      address: address
    }
  end
end

class EmployeeJournal
  include Singleton
  attr_reader :employees

  def initialize
    @employees = {}
  end

  def add_employee(e)
    @employees[e.number] = e
  end

  def change_address(number, address)
    employee = @employees[number]
    raise 'No such employee' unless employee
    employee.address = address
  end

  def delete_employee(number)
    @employees.delete(number)
  end

  def find_employee(number)
    @employees[number]
  end

  def import_employees
    @employees.map { |_k, v| v }
  end
end

class AbstractCommand
  attr_reader :system

  def initialize(args = {}, system = EmployeeJournal.instance)
    @system = system
    post_initialize(args)
  end

  private

  # children may overwrite it
  def post_initialize(args = {})
    nil
  end
end

class AddEmployee < AbstractCommand
  attr_reader :employee

  def execute
    system.add_employee(employee)
  end

  def unexecute
    system.delete_employee(employee.number)
  end

  private

  def post_initialize(args = {})
    @employee = args[:employee]
  end
end

class DeleteEmployee < AbstractCommand
  attr_reader :number

  def execute
    @employee_hash = system.find_employee(number).to_json
    system.delete_employee(number)
  end

  def unexecute
    raise 'No such employee' if @employee_hash.nil?
    employee = Employee.new(
      name: @employee_hash[:name],
      number: @employee_hash[:number],
      address: @employee_hash[:address]
    )
    system.add_employee(employee)
  end

  private

  def post_initialize(args = {})
    @number = args[:number]
    @employee_hash = nil
  end
end

class ChangeAddress < AbstractCommand
  attr_reader :number, :address
  def execute
    @old_addres = system.find_employee(number).address
    system.change_address(number, address)
  end

  def unexecute
    raise 'No such employee' if @old_addres.nil?
    system.change_address(number, @old_addres)
    @old_addres = nil
  end

  private

  def post_initialize(args={})
    @number = args[:number]
    @address = args[:address]
    @old_addres = nil
  end
end

class FindEmployee < AbstractCommand
  attr_reader :number

  def execute
    system.find_employee(number)
  end

  def unexecute
    nil
  end

  private

  def post_initialize(args = {})
    @number = args[:number]
  end
end

# performing

john = Employee.new(
  name: 'John',
  number: 1,
  address: '19368 Bins Meadow'
)
coby = Employee.new(
  name: 'Coby',
  number: 2,
  address: '8837 Ratke Meadow'
)
karianne = Employee.new(
  name: 'Karianne',
  number: 3,
  address: '8940 Nadia Cliffs'
)
elvie = Employee.new(
  name: 'Elvie',
  number: 4,
  address: '723 Ewald Glens'
)
santos = Employee.new(
  name: 'Santos',
  number: 5,
  address: '78087 Porter Pine'
)

add_john = AddEmployee.new(employee: john)
add_coby = AddEmployee.new(employee: coby)
add_karianne = AddEmployee.new(employee: karianne)
add_elvie = AddEmployee.new(employee: elvie)
add_santos = AddEmployee.new(employee: santos)
delete_elvie = DeleteEmployee.new(number: elvie.number)
find_number5 = FindEmployee.new(number: 5)
change_number5 = ChangeAddress.new(number: 5, address: '6999 Koepp Plains')

EmployeeJournal.instance.import_employees # 0 employees

add_john.execute
add_coby.execute
add_karianne.execute

EmployeeJournal.instance.import_employees # 3 employees

add_karianne.unexecute

EmployeeJournal.instance.import_employees # 2 employees

add_elvie.execute
add_santos.execute

EmployeeJournal.instance.import_employees # 4 employees

delete_elvie.execute

EmployeeJournal.instance.import_employees # 3 employees

delete_elvie.unexecute

EmployeeJournal.instance.import_employees # 4 employees

find_number5.execute
# #<Employee:0x007fd905038ac8 @name="Santos", @number=5, @address="78087 Porter Pine">
change_number5.execute

find_number5.execute
# #<Employee:0x007fd905038ac8 @name="Santos", @number=5, @address="6999 Koepp Plains">

find_number5.unexecute # nil


# or run it in queue
[
  add_john, add_coby, add_karianne, add_elvie, add_santos,
  delete_elvie, change_number5
].each do |command|
  command.execute
end
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D0%B0-d68de249a3d8)
