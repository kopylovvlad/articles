# Паттерны на ruby: Шаблонный метод

**Описание:**

Данный шаблон позволяет переложить реализацию алгоритма манипулирования данными с класса-родителя, на классы потомки, которые созданы для каждого конкретного случая. Не меняя при этом входящие данные и не переписывая публичные методы.

**Пример задачи:**

Необходимо реализовать функционал генерации текстовых отчетов о товаров. Отчеты должны быть в форматах: обычный текст, json и html.

**Реализация:**

У нас есть абстрактный класс <AbstractReport> с публичным методом .create_report, который описывает реализацию создания отчета.
От него наследуются классы <TextReport>, <JSONReport> и <HTMLReport>.
Каждый из них переписывает только приватные методы класса и не трогает публичных методов.

```ruby
class Item
  attr_reader :title, :description, :price
  def initialize(arg = {})
    @title = arg[:title]
    @description = arg[:description]
    @price = arg[:price]
  end
end

class AbstractReport
  attr_reader :title, :items
  def initialize(arg = {})
    @title = arg[:title]
    @items = arg[:items]
  end

  def create_report
    main_template
      .gsub(/%items_placeholder%/, items_string_array.join(joiner))
      .gsub(/%report_title%/, title)
  end

  private

  def items_string_array
    items.map do |item|
      item_template
        .gsub(/%title%/, item.title)
        .gsub(/%description%/, item.description)
        .gsub(/%price%/, item.price)
    end
  end

  def joiner
    ''
  end

  def main_template
    raise 'Called abstract method: main_template'
  end

  def item_template
    raise 'Called abstract method: item_template'
  end
end

class TextReport < AbstractReport
  def main_template
    [
      'Text Report.',
      'Report title: %report_title%',
      'Items are:',
      '%items_placeholder%'
    ].join("\n")
  end

  def item_template
    [
      'Item:',
      "title: '%title%',",
      "description: '%description%',",
      "price: '%price%'\n"
    ].join("\n")
  end
end

class JSONReport < AbstractReport
  def main_template
    "{\"type\":\"JSON Report\",\"title\":\"%report_title%\",\"items\":[%items_placeholder%]}"
  end

  def item_template
    "{\"title\":\"%title%\",\"description\":\"%description%\",\"price\":\"%price%\"}"
  end

  def joiner
    ','
  end
end

class HTMLReport < AbstractReport;
  def main_template
    [
      '<!DOCTYPE html>',
      '<html>',
      ' <head>',
      '  <meta charset="utf-8">',
      '  <title>%report_title%</title>',
      ' </head>',
      ' <body>',
      '  <div>',
      '    %items_placeholder%',
      '  </div>',
      ' </body>',
      '</html>'
    ].join("\n")
  end

  def item_template
    [
      '<p>',
      '      <span>title: %title%</span>',
      '      <span>description: %description%</span>',
      '      <span>price: %price%</span>',
      "    <p>"
    ].join("\n")
  end
end

PARAMS = {
  title: 'Report for 3 items',
  items: [
    Item.new(title: 'Item 1', description: 'It is not nice item', price: '10'),
    Item.new(title: 'Item 2', description: 'Not bad', price: '20'),
    Item.new(title: 'Item 3', description: 'Very good item', price: '30')
  ]
}

puts TextReport.new(PARAMS).create_report
=begin
Text Report.
Report title: Report for 3 items
Items are:
Item:
title: 'Item 1',
description: 'It is not nice item',
price: '10'
Item:
title: 'Item 2',
description: 'Not bad',
price: '20'
Item:
title: 'Item 3',
description: 'Very good item',
price: '30'
=end

puts JSONReport.new(PARAMS).create_report
#{"type":"JSON Report","title":"Report for 3 items","items":[{"title":"Item 1","description":"It is not nice item","price":"10"},{"title":"Item 2","description":"Not bad","price":"20"},{"title":"Item 3","description":"Very good item","price":"30"}]}

puts HTMLReport.new(PARAMS).create_report
=begin
<!DOCTYPE html>
<html>
 <head>
  <meta charset="utf-8">
  <title>Report for 3 items</title>
 </head>
 <body>
  <div>
    <p>
      <span>title: Item 1</span>
      <span>description: It is not nice item</span>
      <span>price: 10</span>
    <p><p>
      <span>title: Item 2</span>
      <span>description: Not bad</span>
      <span>price: 20</span>
    <p><p>
      <span>title: Item 3</span>
      <span>description: Very good item</span>
      <span>price: 30</span>
    <p>
  </div>
 </body>
</html>
=end
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D1%88%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD%D0%BD%D1%8B%D0%B9-%D0%BC%D0%B5%D1%82%D0%BE%D0%B4-e1cc3d8afe9)
