---
layout: post
title: Jsonless Json Schema
---
As a Rails programmer developing a number of APIs, it is no surprise that there needs to be a way have confidence in what your are writing and your code will always conform to the relative Api Schema.

The best resource I have found on the subject is from the awesome guys at Thoughtbot and their [blog post](http://robots.thoughtbot.com/validating-json-schemas-with-an-rspec-matcher) on the subject.

The solution they provide is excellent and provides you with a foundation on which to test your JSON response. The TLDR of which is, by using the [JSON-Schema](https://github.com/ruby-json-schema/json-schema) gem, you can validate your response against a Json schema.

However I am Ruby programmer first and foremost. So I would rather not spend all day writing JSON schema files like the following

```javascript
{
  "type": "object",
  "properties": {
    "name" : {
      "type": "string"
    },
    "super_power": {
      "type": "string"
    },
    "weaknesses": {
      "type": "array"
        "items": {
          "type": "string"
        },
    }
    "times_saved_the_world": {
      "type": "number"
    }
  }
}
```

Which frankly when your spend most of your time writing and reading ruby, this is hard to mentally parse, verbose , and is annoying to write.

However there is a nicer way. The JSON Schema gem can take ruby hashes as well as JSON schema strings. Which means we can rewrite it in Ruby (Boo Yah).

```ruby
{
  type: :object,
  properties: {
    name: {
      type: :string
    },
    superpower: {
      type: :string
    },
    weaknesses: {
      type: :array
        items: {
          type: :string
        }
    },
    times_saved_the_world: {
      type: :number
    }
  }
}
```


Already the schema definition is nicer to read (at least to a ruby-ist). However we have no structure to use it in our project. A hash by itself is not very useful.

In spec/support/json\_schema/super\_hero\_schema.rb

```ruby
module JsonSchema
  def self.super_hero
    {
      type: :object,
      properties: {
        name: {
          type: :string
        },
        superpower: {
          type: :string
        },
        weaknessess: {
          type: :array,
          items: {
            type: :string
          }
        },
        times_saved_the_world: {
          type: :number
        }
      }
    }
  end
end
```

With this we can now access the schema hash definition simply with

```ruby
JsonSchema.super_hero
```
By using modules to name space it we can create matcher using the JSON schema Gem by adapting the example by Thoughtbot.
With our new spec/support/api\_schema\_matcher.rb to use a hash object for schema definitions

```ruby
RSpec::Matchers.define :match_response_schema do |schema|
  match do |response|
    JSON::Validator.validate!(schema, response, strict: true)
  end
end

```

And in our spec file

```ruby
describe SuperHeroSerializer do
  subject { described_class.new(super_hero).to_json }
  it "matches its repsonse schema" do
    expect(subject).to match_response_schema(JsonSchema.super_hero)
  end
end
```
We now have a spec to run to ensure that our serialized object matches what is expected as defined by the response schema.

I like this solution a lot better with one annoying problem. The continual use of type hashes to define strings, number, etc. So lets DRY it up a little.

In /spec/support/json\_schema.rb

```ruby
module JsonSchema
  def self.string
    { type: :string }
  end

  def self.number
    { type: :number }
  end
end
```
With this we can update /spec/support/json\_schema/super\_hero\_schema.rb to:

```ruby
 {
    type: :object,
    properties: {
      name: JsonSchema.string,
      super_power: JsonSchema.string,
      weaknessess: {
        type: :array,
          items: JsonSchema.string
      },
      times_save_the_world: JsonSchema.number
    }
 }
```

Which is much more succinct.

Thanks comments and improvements welcome.

[Follow me maybe ](http://twitter.com/MattlesHunter)
Thank you and goodnight.






