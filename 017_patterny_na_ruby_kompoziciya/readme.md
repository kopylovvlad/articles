# Паттерны на ruby: Композиция

**Описание:**

Данный паттерн решает задачу манипулированием вложенными объектами как одним объектом.

**Пример задачи:**

Посчитать сколько весят папки с файлами и другими папками в сумме.

**Реализация:**

Во-первых надо понять какие объекты являются конечными, а какие могут включать в себя другие.
В нашем варианте, конечным объектом являеться Файл, а объект Папка может хранить в себе другие папки и файлы.
Для каждого из них описываем поведение в абстрактных классах <AbstractLeaf> (для файла) и <AbstractComposite> (для папки).
У обоих из них есть метод .size который, в случае с <AbstractLeaf> возвращает указанный размер, а с <AbstractComposite> возвращает сумму вызовов .size всех вложенных элементов.
От них наследуем классы <FileItem> и <Folder> соответственно.

```ruby
class AbstractLeaf
  def initialize; end
  def size
    raise 'Called abstract method: size'
  end
end

class AbstractComposite
  attr_reader :subgroup
  def initialize(args = {})
    @subgroup = []
    post_initialize(args)
  end

  def add(child)
    subgroup.push(child)
    self
  end

  def remove(child)
    subgroup.remove(child)
    self
  end

  def size
    subgroup.map { |item| item.size }.reduce(:+) || 0
  end

  # subclasses may override
  def post_initialize(args={})
    nil
  end
end

class FileItem < AbstractLeaf
  attr_reader :size, :title
  def initialize(args = {})
    @size = args[:size]
    @title = args[:title]
  end
end

class Folder < AbstractComposite
  attr_reader :title
  def post_initialize(args={})
    @title = args[:title]
  end
end


file1 = FileItem.new(title: 'file1', size: 2)
file2 = FileItem.new(title: 'file1', size: 3)

puts file1.size
# 2
puts file2.size
# 3

folder1 = Folder.new(title: 'folder1')
puts folder1.size
# 0
folder1.add(file1)
puts folder1.size
# 2
folder1.add(file2)
puts folder1.size
# 5

folder2 = Folder.new(title: 'folder2')
puts folder2.size
# 0
folder2.add(FileItem.new(title: 'anotherfile', size: 10))
puts folder2.size
# 10
folder2.add(folder1)
puts folder2.size
# 15

folder3 = Folder.new(title: 'main folder')
puts folder3.size
# 0
folder3.add(folder1).add(folder2)
puts folder3.size
# 20
```


[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%B7%D0%B8%D1%86%D0%B8%D1%8F-21a8ff9e2075)
