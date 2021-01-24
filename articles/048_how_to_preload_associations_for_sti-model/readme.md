# How to preload associations for STI-model

![image01](image01.webp)

Single table inheritance (STI) is useful pattern. It helps organise codebase and split logic into different classes. Many programmers use it and like the pattern, but it has one disadvantage: you can’t preload associations by standard `preload` method. Now, I show you the way to handle it.

Imagine, you have an Instagram style application where users are able to upload information and create posts. There are different types of post:

* post with one image
* multi images post
* and post with video

You codebase is based by STI pattern

```ruby
# base class
class Post < ApplicationRecord
end

# standard post with one image
class ImagePost < Post
  has_one :image, dependent: :destroy, foreign_key: :post_id
end

# multi images post
class MultiImagePost < Post
  has_many :images, dependent: :destroy, foreign_key: :post_id
end


# post with video
class VideoPost < Post
  has_one :video, dependent: :destroy, foreign_key: :post_id
end
```

Also `Image` class has relation to `Asset`.

```ruby
class Image < ApplicationRecord
  belongs_to :post, foreign_key: :post_id
  has_one :asset
end

class Asset < ApplicationRecord
  belongs_to :image
end
```

You have different feeds for each post type, and your application works perfectly.

```ruby
ImagePost.preload(image: :asset).each do |item|
  render 'image_post', item: item
end
# database queries:
# ImagePost Load (0.8ms)  SELECT "posts".* FROM "posts" WHERE "posts"."type" = ?  [["type", "ImagePost"]]
#  Image Load (1.4ms)  SELECT "images".* FROM "images" WHERE "images"."post_id" IN (?, ?)  [["post_id", 21], ["post_id", 22]]
#  Asset Load (0.9ms)  SELECT "assets".* FROM "assets" WHERE "assets"."image_id" IN (?, ?)  [["image_id", 5], ["image_id", 6]]

MultiImagePost.preload(images: :asset).each do |item|
  render 'multi_image_post', item: item
end
# database queries:
# MultiImagePost Load (0.7ms)  SELECT "posts".* FROM "posts" WHERE "posts"."type" = ?  [["type", "MultiImagePost"]]
# Image Load (1.1ms)  SELECT "images".* FROM "images" WHERE "images"."post_id" IN (?, ?)  [["post_id", 20], ["post_id", 25]]
# Asset Load (1.0ms)  SELECT "assets".* FROM "assets" WHERE "assets"."image_id" IN (?, ?, ?, ?)  [["image_id", 3], ["image_id", 4], ["image_id", 7], ["image_id", 8]]

VideoPost.preload(:video).each do |item|
  render 'video_post', item: item
end
# database queries:
# VideoPost Load (0.9ms)  SELECT "posts".* FROM "posts" WHERE "posts"."type" = ?  [["type", "VideoPost"]]
# Video Load (0.9ms)  SELECT "videos".* FROM "videos" WHERE "videos"."post_id" IN (?, ?)  [["post_id", 23], ["post_id", 24]]
```

The other day, a customer asked you to render all types of post in one feed. But there is one problem: you can’t use standard `.preload`. `ActiveRecord::AssociationNotFoundError (Association named 'image' was not found on MultiImagePost; perhaps you misspelled it?)`

What should you do? You can write your custom preload logic for new feed using `ActiveRecord::Associations::Preloader` You should create new instance of the class and then use `preload` method. It takes two parameters:

* Collection of AR-items
* Hash with relations that you want to preload. The code doesn’t look like rails-magic and you have to describe all relations explicitly, but it works perfectly.

```ruby
posts = Post.all
preloader = ActiveRecord::Associations::Preloader.new
preloader.preload(posts.select{ |i| i.type == 'ImagePost' }, image: :asset)
preloader.preload(posts.select{ |i| i.type == 'MultiImagePost' }, images: :asset)
preloader.preload(posts.select{ |i| i.type == 'VideoPost' }, :video)

posts.each |item|
  render 'post', item: item
end
```

Look into logs, `Preloader` sends one query for each association, and you there aren’t N+1 queries.

```ruby
# 1) take posts from DB
Post Load (0.4ms)  SELECT "posts".* FROM "posts"
# 2) load images for all ImagePosts
Image Load (1.5ms)  SELECT "images".* FROM "images" WHERE "images"."post_id" IN (?, ?)  [["post_id", 21], ["post_id", 22]]
# 3) load assets for them
Asset Load (1.0ms)  SELECT "assets".* FROM "assets" WHERE "assets"."image_id" IN (?, ?)  [["image_id", 5], ["image_id", 6]]
# 4) load images for all MultiImagePost
Image Load (0.4ms)  SELECT "images".* FROM "images" WHERE "images"."post_id" IN (?, ?)  [["post_id", 20], ["post_id", 25]]
# 4) load assets for them
Asset Load (0.2ms)  SELECT "assets".* FROM "assets" WHERE "assets"."image_id" IN (?, ?, ?, ?)  [["image_id", 3], ["image_id", 4], ["image_id", 7], ["image_id", 8]]
# 5) load videos for all VideoPosts
Video Load (0.4ms)  SELECT "videos".* FROM "videos" WHERE "videos"."post_id" IN (?, ?)  [["post_id", 23], ["post_id", 24]]
```

[dev.to](https://dev.to/kopylov_vlad/how-to-preload-associations-for-sti-model-422a)
