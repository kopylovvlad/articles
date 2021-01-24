# How to refactor long if-else condition

![Photo by NESA by Makers on Unsplash](image01.jpeg)

Sometimes you write a code with long if-else conditions. Rubocop doesn’t like it and will print offenses as:

```
Metrics/AbcSize: Assignment Branch Condition size for transform_for_machine is too high.
Metrics/MethodLength: Method has too many lines.
Metrics/PerceivedComplexity: Perceived complexity for transform is too high.
```

How to fix it? — Use Dictionary!

**Example 1**. We use if-else condition to return a value. Here is a function that takes one argument and returns a string or default string. Rubocop doesn’t like it and prints “Metrics/MethodLength: Method has too many lines. [13/10]”. We can refactor it easily.

```ruby
# Before
# Metrics/MethodLength: Method has too many lines. [13/10]
def foo1(number)
  if number == 1
    'one'
  elsif number == 2
    'two'
  elsif number == 3
    'three'
  elsif number == 4
    'four'
  elsif number == 5
    'five'
  else
    'many'
  end
end

# After
DICTIONARY = {
  1 => 'one',
  2 => 'two',
  3 => 'three',
  4 => 'four',
  5 => 'five'
}.freeze

def foo2(number)
  DICTIONARY[number] || 'many'
end
```

**Example 2**. We can use if-else condition to manipulate an object. Here is a module that implements image conversion algorithm. It is just long if-else condition for choosing a right module. And here we could use a dictionary.

```ruby
# before
module Main
  def self.convert(file, format)
    if format == 'png'
      PngGenerator.generate(file)
    elsif format == 'gif'
      GifGenerator.generate(file)
    elsif format == 'bmp'
      BmpGenerator.generate(file)
    elsif format == 'tiff'
      TiffGenerator.generate(file)
    elsif format == 'pdf'
      PdfGenerator.generate(file)
    end
  end
end

# after
module Main2
  DICTIONARY = {
    'png' => PngGenerator,
    'gif' => GifGenerator,
    'bmp' => BmpGenerator,
    'tiff' => TiffGenerator,
    'pdf' => PdfGenerator
  }.freeze

  def self.convert(file, format)
    DICTIONARY[format].generate(file)
  end
end
```

**Example 3**. If we have to refactor a difficult algorithm, we could use dictionary with procs. Here is an example of refactoring a searching algorithm.

```ruby
# Before
module SearchService1
  # Metrics/AbcSize: Assignment Branch Condition size for perform is too high.
  # Metrics/CyclomaticComplexity: Cyclomatic complexity for perform is too high.
  # Metrics/MethodLength: Method has too many lines.
  # Metrics/PerceivedComplexity: Perceived complexity for perform is too high.
  def self.perform(params = {})
    scope = ::User.active

    if params['id'].present?
      scope = scope.where(id: params['id'])
    end

    %w[first_name last_name email city gender].each do |item|
      next unless params[item].present?
      scope = scope.where(item => params[item])
    end

    if params['height_after'].present?
      scope = scope.where(height: { '$gt' => params['height_after'] })
    end

    if params['height_before'].present?
      scope = scope.where(height: { '$lt' => params['height_before'] })
    end

    if params['weight_after'].present?
      scope = scope.where(weight: { '$gt' => params['weight_after'] })
    end

    if params['weight_before'].present?
      scope = scope.where(weight: { '$lt' => params['weight_before'] })
    end

    if params['gender'].present?
      scope = scope.where(gender: params['gender'])
    end

    scope
  end
end

# After
module SearchService2
  PARAMS_MAP = {
    'id' => ->(value) { { id: value } },
    'first_name' => ->(value) { { 'first_name' => value } },
    'last_name' => ->(value) { { 'last_name' => value } },
    'email' => ->(value) { { 'email' => value } },
    'city' => ->(value) { { 'city' => value } },
    'gender' => ->(value) { { gender: value } },
    'height_after' => ->(value) { { height: { '$gt' => value } } },
    'height_before' => ->(value) { { height: { '$lt' => value } } },
    'weight_after' => ->(value) { { weight: { '$gt' => value } } },
    'weight_before' => ->(value) { { weight: { '$lt' => value } } },
  }.freeze

  def self.perform(params = {})
    where_params = {}

    PARAMS_MAP.keys.each do |item|
      next unless params[item].present?
      where_params.merge!(PARAMS_MAP[item][params[item]])
    end

    ::User.active.where(where_params)
  end
end
```

And it is all. I hope the material is useful.

[Medium](https://kopilov-vlad.medium.com/how-to-refactor-long-if-else-condition-793645afc707)
