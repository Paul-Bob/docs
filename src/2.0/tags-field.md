# Tags field

[[toc]]

<div class="rounded-md bg-blue-50 p-4">
  <div class="flex">
    <div class="flex-shrink-0">
      <svg class="h-5 w-5 text-blue-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z" clip-rule="evenodd" />
      </svg>
    </div>
    <div class="ml-3 flex-1 md:flex md:justify-between">
      <div class="text-sm leading-5 text-blue-700">
        This is a <a href="https://avohq.io/purchase/pro" target="_blank" class="underline">pro</a> feature
      </div>
    </div>
  </div>
</div>

**Requires v2.5.0**

## Basic info

Adding a list of things to a record is something we need to do pretty frequently, that's why having the `tags` field is really useful.

```ruby
field :skills, as: :tags
```

<img :src="$withBase('/assets/img/fields/tags-field/basic.gif')" alt="Avo tags field" class="border mb-4" />

## Suggestions

You can pass suggestions to your users to pick from. The `suggestions` option can be an array of strings, an object with the keys `value`, `label`, and (optionally) `avatar`, or a block that returns an array of that type of object. The block is a [`RecordHost`](evaluation-hosts.html#recordhost), so it has access to the `record`.

```ruby
# app/avo/resource/course_resource.rb
class CourseResource < Avo::BaseResource
  field :skills, as: :tags, suggestions: -> { record.skill_suggestions }
end

# app/models/course.rb
class Course < ApplicationRecord
  def skill_suggestions
    ['example suggestion', 'example tag', self.name]
  end
end
```

<img :src="$withBase('/assets/img/fields/tags-field/suggestions.gif')" alt="Avo tags field" class="border mb-4" />

The suggestions will be displayed to the user as a dropdown under the field.

## Enforce suggestions

You might only want to allow the user to select from a pre-configured list of items. You can use `enforce_suggestions` to do that. Now the user won't be able to add anything else than what you posted in the `suggestions` option.

```ruby
# app/avo/resource/course_resource.rb
class CourseResource < Avo::BaseResource
  field :skills, as: :tags, suggestions: %w(one two three), enforce_suggestions: true
end
```

<img :src="$withBase('/assets/img/fields/tags-field/enforce_suggestions.gif')" alt="Avo tags field" class="border mb-4" />

## Blacklist

The `blacklist` param works similarly to `suggestions`. Use it to prevent the user from adding specific values.

```ruby
# app/avo/resource/course_resource.rb
class CourseResource < Avo::BaseResource
  field :skills, as: :tags, blacklist: ['not', 'that']
end
```

<img :src="$withBase('/assets/img/fields/tags-field/blacklist.gif')" alt="Avo tags field" class="border mb-4" />

## Delimiters

By default, the delimiter that cuts off the content when the user inputs data is a comma `,`. You can customize that using the `delimiters` option.

```ruby
# app/avo/resource/course_resource.rb
class CourseResource < Avo::BaseResource
  field :skills, as: :tags, delimiters: [',', ' ']
end
```

<img :src="$withBase('/assets/img/fields/tags-field/delimiters.gif')" alt="Avo tags field" class="border mb-4" />

Valid values are comma `,` and space ` `.

## Close on select

If you have `suggestions` enabled, the dropdown with the options will keep open after the user selects an option. You can choose to close it after a selection using `close_on_select`.

```ruby
# app/avo/resource/post_resource.rb
class PostResource < Avo::BaseResource
  field :items, as: :tags, suggestions: -> { Post.tags_suggestions }, close_on_select: true
end

# app/models/post.rb
class Post < ApplicationRecord
  def self.tags_suggestions
    [
      {
        value: 1,
        label: 'one',
        avatar: 'https://images.unsplash.com/photo-1560363199-a1264d4ea5fc?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&w=256&h=256&fit=crop',
      },
      {
        value: 2,
        label: 'two',
        avatar: 'https://images.unsplash.com/photo-1567254790685-6b6d6abe4689?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&w=256&h=256&fit=crop',
      },
      {
        value: 3,
        label: 'three',
        avatar: 'https://images.unsplash.com/photo-1560765447-da05a55e72f8?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&w=256&h=256&fit=crop',
      },
    ]
  end
end
```

<img :src="$withBase('/assets/img/fields/tags-field/close_on_select.gif')" alt="Avo tags field" class="border mb-4" />

## PostgreSQL array fields

You can use the tags field with the PostgreSQL array field.

```ruby{9}
# app/avo/resource/course_resource.rb
class CourseResource < Avo::BaseResource
  field :skills, as: :tags
end

# db/migrate/add_skills_to_courses.rb
class AddSkillsToCourses < ActiveRecord::Migration[6.0]
  def change
    add_column :courses, :skills, :text, array: true, default: []
  end
end
```

## Array fields

We haven't tested all the possibilities, but the tags field should play nicely with any array fields provided by Rails.

```ruby{8-10,12-14}
# app/avo/resource/post_resource.rb
class PostResource < Avo::BaseResource
  field :items, as: :tags
end

# app/models/post.rb
class Post < ApplicationRecord
  def items=(items)
    puts ["items->", items].inspect
  end

  def items
    %w(1 2 3 4)
  end
end
```

## Acts as taggable on

One very popular gem used for tagging is [`acts-as-taggable-on`](https://github.com/mbleigh/acts-as-taggable-on). The tags field integrates very well with it.

You need to add `gem 'acts-as-taggable-on', '~> 9.0'` in your `Gemfile`, add it to your model `acts_as_taggable_on :tags`, and use `acts_as_taggable_on` on the field.

```ruby{5}
# app/avo/resource/post_resource.rb
class PostResource < Avo::BaseResource
  field :tags,
    as: :tags,
    acts_as_taggable_on: :tags,
    close_on_select: false,
    placeholder: 'add some tags',
    suggestions: -> { Post.tags_suggestions },
    enforce_suggestions: true,
    help: 'The only allowed values here are `one`, `two`, and `three`'
end

# app/models/post.rb
class Post < ApplicationRecord
  acts_as_taggable_on :tags
end
```

That will let Avo know which attribute should be used to fill with the user's tags.
