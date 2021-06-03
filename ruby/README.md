# Ruby

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
