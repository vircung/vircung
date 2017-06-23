---
layout: post
title:  "Rails support for UUID"
cover: https://static.pexels.com/photos/1173/animal-dog-pet-cute.jpg
tags:
  - en
---

[PostgreSQL][pg-gem-github] is almost popular as [SQLite][sqlite-gem-github] as a database backend for Rails applications. Some of them uses [uuid-ossp][uuid-ossp-extension-github] extension to change type of column `id` to `uuid`. It's common knowledge that `uuid` breaks `references` column type when you want to reflect table relations in migration files. Actually it's not.

<!-- more -->

## Missing knowledge

When you [search][rails-uuid-primary-key-search] for articles that are about integrating `uuid` type into `id` column. [Almost](http://blog.arkency.com/2014/10/how-to-start-using-uuid-in-activerecord-with-postgresql/) every [one](http://labria.github.io/2013/04/28/rails-4-postgres-uuid-pk-guide/) of [them](http://rny.io/rails/postgresql/2013/07/27/use-uuids-in-rails-4-with-postgresql.html) either say that this will break `references` column type in migrations or says nothing about it.

There's workaround to use `t.uuid :something_else_id` thing. And it's working great. I don't mind. I even used it in my code. But the brilliant idea came to mi that i might fix it in Rails code. And i started digging.

## Actually it works

I started to go through Rails code to search The Place. It's hard to navigate though it. Especially when you go into large codebase for first time. So i stumbled upon migration tests and found this [little gem][uuid-reference-type-github]. It works! But even Rails documentation says nothing about this. But hopefully it's going to be [fixed][rails-uuid-doc-pr] :)

Therefore you can fix `t.references :something` to work with `uuid`

{% highlight ruby %}
# db/migrate/20150418012400_create_blog.rb
def change
  create_table :posts, id: :uuid
end

create_table :comments, id: :uuid do |t|
  # t.belongs_to :post, type: :uuid
  t.references :post, type: :uuid
end

# app/models/post.rb
class Post < ActiveRecord::Base
  has_many :comments
end

# app/models/comment.rb
class Comment < ActiveRecord::Base
  belongs_to :post
end
{% endhighlight %}

It would be lovely that migration mechanism would detect referenced column type automatically. I think i can be done. Just need to find proper place to start hacking things.


[pg-gem-github]:https://github.com/search?l=ruby&q=gem+pg&ref=searchresults&type=Code&utf8=%E2%9C%93
[sqlite-gem-github]:https://github.com/search?l=ruby&q=gem+sqlite3&ref=searchresults&type=Code&utf8=%E2%9C%93
[uuid-ossp-extension-github]:https://github.com/search?utf8=%E2%9C%93&q=enable_extension+uuid-ossp&type=Code

[rails-uuid-doc-pr]:https://github.com/rails/rails/pull/19806
[rails-uuid-primary-key-search]:https://www.google.pl/search?q=rails+uuid+primary+key
[uuid-reference-type-github]:https://github.com/rails/rails/blob/master/activerecord/test/cases/adapters/postgresql/uuid_test.rb#L268
