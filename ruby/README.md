# Ruby


## Rails

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
