# Паттерны на ruby: Посетитель

**Описание:** Данный паттерн хорош, если вам нужно реализовать простой интерфейс позволяющий проводить множество действий над разными объектами.

**Пример задачи:** Для rails-приложения нужно написать логику создания отчетов в разные форматы для разных классов моделей. Например для модели <User> и <Order> необходимы выгрузки в XML и JSON форматах.

**Реализация:** Для классов <User> и <Order>, необходимо написать метод .accept который вызывает нужный метод класса-посетителя. Для логики выгрузки в форматы XML и JSON необходимо написать классы <XMLExportVisitor> и <JSONExportVisitor> у которых будет по одному методу, на каждую посещенную модель.

```ruby
module Visitable
  def accept(visitor)
    raise 'does not implement error'
  end
end

# class that accept any visitor
class User < ApplicationRecord
  include Visitable

  # some code

  def accept(visitor)
    visitor.visit_user(self)
  end
end

# class that accept any visitor
class Order < ApplicationRecord
  include Visitable

  # some code

  def accept(visitor)
    visitor.visit_order(self)
  end
end


# main class for any visitors
class AbstractVisitor
  def visit_user(user)
    raise 'does not implement error'
  end

  def visit_order(order)
    raise 'does not implement error'
  end

  # here we can add some method for any classes
end


class XMLExportVisitor < AbstractVisitor
  def visit_user(user)
    # export logic user to XML is here
  end

  def visit_order(order)
    # export logic order to XML is here
  end
end

class JSONExportVisitor < AbstractVisitor
  def visit_user(user)
    # export logic user to JSON is here
  end

  def visit_order(order)
    # export logic order to JSON is here
  end
end



# and controller in rails-app
module Api
  module Users
    class ExportsController < ApplicationController
      before_action :find_user

      def xml_export
        @report = @user.accept(XMLExportVisitor.new)
      end

      def json_export
        @report = @user.accept(JSONExportVisitor.new)
      end

      private

      def find_user
        @user = User.find(params[:id])
      end
    end
  end
end
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%BF%D0%BE%D1%81%D0%B5%D1%82%D0%B8%D1%82%D0%B5%D0%BB%D1%8C-630c152e34f9)
