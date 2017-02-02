# Try Paperclip!!
### Step-to-Rails-Expert.rb [@two_sann](https://twitter.com/two_sann)
### Jan.31th 2017

---

## 自己紹介

- IT系の会社で社内SEやってます。

- ずっとフロントエンド中心にやってきた感じです。  
（Java歴が長くてPHPは少しかじった程度）

- Ruby歴は３年くらいです。（結構経ったな・・・）

- holidaysっていうgemをメンテしてます。  
よかったら使ってください。:)
    - github: <https://github.com/holidays/holidays>

---

## Paperclipってなに？

- 有名なファイルアップロード用のgem。  
一応file attachment librariesとのことなので画像以外も  
添付できるはず。

- Github => <https://github.com/thoughtbot/paperclip>  
  8282 stars at Jan.6.2017

- 公式サイト？ => <https://thoughtbot.com/tools>  
 Thoughtbot社（Factory_girlとか作った会社）なので  
 メンテは続きそう。

---

# Requirements

--

### Ruby and Rails
- Ruby >= 2.1
- Rails >= 4.2

--

### imagemagick
- 画像処理（多分リサイズやサムネイル作成等）で必要。  
今回はテーマが画像なので必須。  
- 公式サイト<https://www.imagemagick.org/script/index.php>

--

### File
- content_typeのチェックにfileコマンドの機能が必要。  
  Unix系のOSなら特に考慮する必要なさそう。  
  Windows系はRubyinstallerのdevelopment-kitが必要。

---

# Installation

--

- include rails gems
~~~  
gem "paperclip”  
bundle install
~~~  

- imagimagick
~~~
brew install imagemagick
~~~
  - sierraにしたせいかinstallに失敗したけど  
  brew updateして無事解決。

---

# Quickstart

--
1.railsプロジェクト作成。<!-- .element: class="fragment grow" data-fragment-index="1" -->
~~~Ruby
rails new picture
~~~
2.適当にModel作る。<!-- .element: class="fragment grow" data-fragment-index="2" -->
~~~Ruby
rails g scaffold user name:string password:string
~~~
3.avatarという名前でpaperclip用のカラムを作る。<!-- .element: class="fragment grow" data-fragment-index="3" -->
~~~Ruby
rails generate paperclip user avatar
~~~
4.paperclip用に以下の４カラムが追加される。  
avatar_file_name  
avatar_file_size  
avatar_content_type  
avatar_updated_at  

--

5.model修正
~~~Ruby
class User < ActiveRecord::Base
      has_attached_file :avatar, styles: { medium: "300x300>", thumb: "100x100>" },  
      default_url: "/images/:style/missing.png"      
　　    validates_attachment_content_type :avatar, content_type: /\Aimage\/.*\z/  
end
~~~
6._form.html.erbの適当な位置にfile_upload用のフィールドを追加
~~~
<div class="field">      
	<%= f.label :avatar %><br>      
	<%= f.file_field :avatar %>  
</div>
~~~

---

# File upload execution

--


- アップロードしたファイルは以下の場所に配置されている。一つのファイルに対して縮小画像とサムネイル画像を同時に作るらしい。
~~~
public
└── system
    └── users
        └── avatars
            └── 000
                └── 000
                    └── 001
                        ├── medium
                        │   └── example_image1.jpg
                        ├── original
                        │   └── example_image1.jpg
                        └── thumb
                             └── example_image1.jpg
~~~
- なお画像以外のファイルでもimagemagickの処理は通るみたいで
同様のフォルダ構成になる。<!-- .element: class="fragment fade-up" data-fragment-index="3" -->

---

# validation

--

- validationは３種類用意(size,content_type,必須)されている。
    - validates_attachment_size
    - validates_attachment_presence
    - alidates_attachment_content_type

- ‘validates_attachment’を使えば複数のチェックを１カラムにまとめて設定することが可能だ。
~~~Ruby
validates_attachment :avatar,presence: true,  content_type: { content_type: "image/jpeg" },  size: { in: 0..10.kilobytes }
~~~

---

# resizing

--

- has_attached_fileのstyle optionで指定する。
~~~Ruby
class User < ActiveRecord::Base
      has_attached_file :avatar, styles: { medium: "300x300>", thumb: "100x100>" }, default_url: "/images/:style/missing.png"      
　　validates_attachment_content_type :avatar, content_type: /\Aimage\/.*\z/  
end
~~~

- view側でそれぞれのサイズを取得することもできる。
~~~
<%= image_tag @user.photo.url %>
<%= image_tag @user.photo.url(:thumb) %>
<%= image_tag @user.photo.url(:medium) %>
~~~

---

# Prevent thumnail generation for invalid uploads.

--

- ’ before_post_process’を指定すれば、paperclipの処理（サムネイル作成処理等）の前に実行するチェックして処理を止めることも可能。（指定しないとinvalidになってもサムネイル作成処理が実行されるらしい）
~~~Ruby
before_post_process :check_file_size
def check_file_size
	valid?
	errors[:image_file_size].blank?
end
~~~

---

# Hashing

--

- ファイル名にハッシュ値を使うことができる。 defaultではハッシュ値を使わないので、URLにファイル名がそのまま表示される。
~~~Ruby
# config/initializers/paperclip_defaults.rb

  Paperclip::Attachment.default_options.update({
    url: "/system/:class/:attachment/:id_partition/:style/:hash.:extension",
    hash_secret: Rails.application.secrets.secret_key_base
  })
~~~
* initializerに以下のファイルを置いてサーバ再起動すればOK.

---

# 最後に

- 導入から利用開始まで非常に簡単。  
触ったことのない方は是非一度試して見てください。   

- ここでは取り上げていませんが、File storageとして  
S3をサポートしているみたいです。ちなみにHerokuも  
File storageにはS３を推奨してるっぽいです。  
<https://devcenter.heroku.com/articles/paperclip-s3>

- Imagemagick optionを使ってサムネイルの最適化もできるので、  
使い方はかなり多岐に渡りそう。  
<https://github.com/thoughtbot/paperclip/wiki/Thumbnail-Generation#optimizing-thumbnails-for-the-web>

---

# 参考

- <https://github.com/thoughtbot/paperclip/wiki>

- <http://qiita.com/kakipo/items/4e70428e46f60c74ea63>

- <http://ruby-rails.hatenadiary.com/entry/20140716/1405443484>
