# Ruby

## Run CLI

```bash
ruby <<EOF
  puts 'test'
EOF

cat <<EOF | ruby
	puts 'test'
EOF

ruby - "$@" << EOF
	puts ARGV
EOF

ruby -e 'code' -- argument
```

## Statement

### Heredoc

```ruby
data = <<-EOF
hello world
EOF

data = %Q(
hello world
)

# \n, #{} 등을 escape
data = <<-'EOF'
hello\nworld
EOF

# squiggly
# output: "hello world\nwow\n"
data = <<~EOF
  hello world
  wow
EOF

# strip
data = <<-EOF.strip
...
EOF
```

### Pattern Matching

ruby >= 2.7

```ruby
case [variable or expression]
in 0 | 1
  ...
in [pattern]
  ...
in [pattern] if [expression] then ...
in [pattern] => var then ...
in [String, Integer => i] then ...
in -> i { i == 0 } then ... # using proc
in {a: Integer} then ...
else
  ...
end

case [0,1,2]
in *a   # a is [0,1,2]
in *a, 1, 2   # a is [0]
in [0, *a]   # a is [1, 2]
in *   # true
end

case {a:3}
in a: 3
in a: /3/
in **a  # a is {a:3}
in a:, **_
end

case [21, 1, 10, 23]
in [20..21, 1..12, 1..31, 0..23]
end

case object
in method1: 0..10, method2: String
end


# oneline pattern (ruby >= 3.0)
{name: 'foo', num: 99} => {name: name, num: num}
puts name, num

_ : ignore value (_a 같은 형태도 가능)
^ : pin operator - 매칭하는 값으로 assign 하지 않고 패턴으로 사용한다
    - [1, 2, 2] => [a, b, b^]
* : multiple values..
```


## Rails

### Nokogiri

```ruby
html = Nokogiri::HTML(response.body)
elem = html.search('.awesome-class').first

puts elem.attr('id')

# xpah
html.xpath('//h1[@class="title"]')
```

### ActiveRecord invalid record

```ruby
begin SomeTable.first.update! name: 'invalid';
rescue ActiveRecord::RecordInvalid => i; puts i.record.errors.each{|e|puts e}; end
```

### Dump to JSON

```ruby
File.write(
  Rails.root.join('output/dump.json'),
  JSON.pretty_generate(
    @query_result.as_json(only: %i[id title metadata created_at])
  )
)
```

### 함수 정의 위치 찾기

```ruby
method(:get).source_location
ClassName.new.method(:method_symbol).source_location
```

### Turbolinks

- https://qiita.com/morrr/items/54f4be21032a45fd4fe9
- https://gist.github.com/saboyutaka/8727377

- head에 추가된건 내용물이 같은 태그가 없으면 새로 추가한다
  - 한번 읽히면 이후로 내용물에 diff가 생기지 않는한 다시 돌지 않는다
- body 안의 내용물은 매번 동작한다
  - addEventListener를 body에 추가하면 tubolinks:load가 동작할 때마다 계속 추가한다
  - script를 body에 쓰는건 비권장

## Rails - Test

- https://guides.rubyonrails.org/testing.html

### Factorybot

- https://github.com/thoughtbot/factory_bot
- https://devhints.io/factory_bot

```ruby
# need include class
include FactoryBot::Syntax::Methods

# define
FactoryBot.define do
  factory :tab, class: Tab do
    _tabs { build(:owner) }
    sequence(:key) { |n| "tab-#{n}" }
    name { 'TAB_NAME' }

    transient do
      or_tags { [] }
    end

    trait :corona do
      name { '코로나' }
      or_tags { %w[코로나] }
      before(:create) do |tab, evaluator|
        tab.tags << evaluator.or_tags.map  do |name|
          build(:tag, name: name, metadata: {})
        end
      end
    end
  end
end

# can use
create(:tab)
create(:tab, :corona)
create(:tab, :corona, or_tags: %w[코로나 백신])
```

```ruby
# DB Mocking이 아니라 값이 지정된 Object처럼 사용하기
FactoryBot.define do
  factory :user_data, class: String do
    skip_create

    filename { nil }

    initialize_with do
      JSON.parse(File.read("test/data/#{filename.to_s}.json"))
    end
  end
end

create :user_data, filename: 'user_01'
```

```ruby
# factory 안에 factory를 정의하면 상속받은 새 factory로 정의할 수 있다
# trait과 다르게 별도의 factory로 사용하게 된다
FactoryBot.define do
  factory :user do
    name { '' }
    type { :normal }
    created_at { DateTime.now }

    factory :guest do
      type { :guest }
    end
  end
end

create :user
create :guest
```

### ActionDispatch::IntegrationTest

```ruby
controller.instance_variable_get(:@page)

response.body

# .item
#   .title= blabla
# .item
#   .title= asdf
assert_select '.item' do |elems|
  assert_select elems, '.name', 'blabla'

  puts elems.first.text # elem is Nokogiri::Element
end
```
