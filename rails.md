# Quy chuẩn coding với Ruby on Rails
Ruby on Rails style guide
## Config chung
### Initializer
  - Để các code khởi tạo trong thư mục `config/initializers`. Các đoạn code để trong thư mục này sẽ tự động execute khi server khởi động.
  - Đối với những file code khởi tạo của các gem như carierwave.rb, active_admin.rb,  devise.rb... thì đặt tên file giống tên gem.
  - Những thiết lập cho từng môi trường development, test, production thì thiết lập trong các file tương ứng trong thư mục config/environments
  - Thiết lập cho tất cả môi trường thì viết vào config/application.rb
  - Trong trường hợp tạo môi trường mới như staging thì cố gắng thiết lập gần giống môi trường production
  - Có thể dùng gem figaro để app bảo mật hơn

## Routing
  * Khi cần phải thêm action vào RESTful resource thì sử dụng ``` member ``` và ``` collection ```

  ```ruby
  # cách viết không tốt
  get 'subscriptions/:id/unsubscribe'
  resources :subscriptions

  # cách viết tốt
  resources :subscriptions do
    get 'unsubscribe', on: :member
  end

  # cách viết không tốt
  get 'photos/search'
  resources :photos

  # cách viết tốt
  resources :photos do
    get 'search', on: :collection
  end
  ```

  * Sử dụng block để nhóm lại khi có nhiều ``` member/collection ```

  ```ruby
  resources :subscriptions do
    member do
      get 'unsubscribe'
      get 'subscribe'
    end
  end

  resources :photos do
    collection do
      get 'search'
      get 'trashes'
    end
  end
  ```

  * Sử dụng routes lồng nhau (nested routes) để thể hiện mối quan hệ của các model trong ActiveRecord

  ```ruby
  class Post < ActiveRecord::Base
    has_many :comments
  end

  class Comments < ActiveRecord::Base
    belongs_to :post
  end

  # routes.rb
  resources :posts do
    resources :comments
  end
  ```

  * Sử dụng namespace để nhóm các action liên quan

  ```ruby
  namespace :admin do
    # Directs /admin/products/* to Admin::ProductsController
    # (app/controllers/admin/products_controller.rb)
    resources :products
  end
  ```

  * Không sử dụng wild controller route trước đây

  **Lý do**

  Viết theo cách này sẽ làm cho tất cả action của mọi controller có thể bị truy cập bằng GET request

  ```ruby
  # Cách viết cực kỳ không tốt
  match ':controller(/:action(/:id(.:format)))'
  ```

## Controller

*  Cố gắng gọt giũa nội dung của controller. Trong controller chỉ nên thực hiện việc lấy những data mà bên view cần, không code business logic ở đây. (Những cái đó nên viết trong model)

* Mỗi action trong controller thì lý tưởng là 1 method initialize model , 1 method seach , 1 method thực hiện tác vụ gì đó.

* Không chia sẻ giữa controller và view từ 2 biến instance trở lên.

* Đối với biến instance biểu thị resource chính ở Controller, hãy gán vào object của resource đó. Ví dụ, đối với `@article` ở bên trong ArticlesController thì gán instance của class `Article` vào. Với `@articles` thì gán collection của nó vào.

```ruby
# Không tốt
class ArticlesController < ApplicationController
  def index
    @articles = Article.all.pluck [:id, :title]
  end

  def show
    @article = "This is an article."
  end
end

# Tốt
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find params[:id]
  end
end
```

* Controller cần xử lý ngoại lệ xuất hiện tại model. Cần phải thông báo việc xuất hiện ngoại lệ bằng cách gửi đến client code 400 trở lên.

* Để tham số của render là symbol.

```ruby
render :new
```

* Không lược bỏ action ngay cả khi action đó không xử lý gì cả mà chỉ để hiển thị view

```ruby
class HomeController < ApplicationController

  def index
  end

end
```

* Đối với những action được truy cập bằng những method HTTP khác GET thì nhất định sau khi đã xử lý xong phải redirect đến một action được truy cập bằng phương thức GET. Tuy nhiên, những trường hợp mà không truy cập trực tiếp, ví dụ như chỉ là API để trả về json thì điều này không cần thiết.

**Lý do**

Ngăn chặn việc phát sinh nhiều xử lý khi mà người dùng thao tác refresh.

* Trong hàm callback nên sử dụng tên method hoặc ```lamda```. Không được sử dụng block ở đây.


```ruby
# cách viết không tốt

  before_action{@users = User.all} # brock

# cách viết tốt

  before_action :methodname # method name

# cách viết cũng tốt

  before_action ->{@users = User.all} # lambda
```

## Models

  * Có thể sử dụng model không cần dựa trên ActiveRecord

  * Cố gắng đặt tên ngắn, dễ hiểu nhưng không giản lược quá mức.

  * Sử dụng gem [ActiveAttr](https://github.com/cgriego/active_attr) khi cần có những thao tác của ActiveRecord giống như validation trong model.

  ```ruby
  class Message
    include ActiveAttr::Model

    attribute :name
    attribute :email
    attribute :content
    attribute :priority

    attr_accessible :name, :email, :content

    validates_presence_of :name
    validates_format_of :email, :with => /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i
    validates_length_of :content, :maximum => 500
  end
  ```

### ActiveRecord
  - Hạn chế đổi các giá trị mặc định của Model như tên bảng, tên cột khóa chính(id).
  * Nhóm các macro lại, Và đặt các constant của class lên đầu tiên. Các macro cùng loại ( như là belongs_to và has_many ) hoặc là cùng macro nhưng tham số khác nhau ( ví dụ như validates ) thì sắp xếp theo thứ tự từ điển. Các callback thì sắp xếp theo thứ tự thời gian.
  - Thứ tự các phần trong một model:
    ```ruby
    class User < ActiveRecord::Base
      # keep the default scope first (if any)
      default_scope { where(active: true) }

      # constants come up next
      COLORS = %w(red green blue)

      # afterwards we put attr related macros
      attr_accessor :formatted_date_of_birth

      attr_accessible :login, :first_name, :last_name, :email, :password

      # Rails4+ enums after attr macros, prefer the hash syntax
      enum gender: { female: 0, male: 1 }

      # followed by association macros
      belongs_to :country

      has_many :authentications, dependent: :destroy

      # and validation macros
      validates :email, presence: true
      validates :username, presence: true
      validates :username, uniqueness: { case_sensitive: false }
      validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
      validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true }

      # next we have callbacks
      before_save :cook
      before_save :update_username_lower

      # scopes
      scope :active, ->{where(active: true)}
  
      # other macros (like devise's) should be placed after the callbacks

      ...
    end
    ```
  - Nên sử dụng `has_many :through` thay cho `has_and_belongs_to_many`
  - Đặt các custom validator trong thư mục app/validators
  - Nên dùng name scope
  - Chú ý hàm `update_attribute` không check validation, hàm `update_attributes` có check validation
  - Khi gọi hàm ActiveRecord#count phải gắn `:select => 'id'`.
  - Dùng hàm validates thay cho các hàm `validates_presence_of`, ... Ví dụ:

    ```ruby
      # bad
      validates_presence_of :email

      # good
      validates :email, presence: true
    ```
  - Dùng find_each để duyệt qua một danh sách các ActiveRecord. Lặp qua một collection các record từ CSDL (vd: dùng phương thức all) rất kém hiệu quả, nguyên do là mỗi lần nó sẽ tạo ra một đối tượng. Thay vào đó, hãy dùng find_each để gom chúng vào và làm một lần.
  
    ```ruby
    # bad
    Person.all.each do |person|
      person.do_awesome_stuff
    end

    Person.where('age > 21').each do |person|
      person.party_all_night!
    end

    # good
    Person.find_each do |person|
      person.do_awesome_stuff
    end

    Person.where('age > 21').find_each do |person|
      person.party_all_night!
    end
    ```
  - Định nghĩa tùy chọn dependent cho quan hệ has_many và has_one
  
    ```ruby
    # bad
    class Post < ActiveRecord::Base
      has_many :comments
    end

    # good
    class Post < ActiveRecord::Base
      has_many :comments, dependent: :destroy
    end
    ```

#### Scope
* Đặt tên scope thể hiện việc lấy một tập hợp con trong tập hợp bản ghi cha. 
* Hãy đặt tên scope sao cho có thể hiểu được một cách tự nhiên như sau 
`[số nhiều của tên model] có đặc tính [tên scope]`. 
(Ví dụ: với việc đặt tên scope là `active` trong model user có thể hiểu 
            Hãy lấy ra các `[users] có đặc tính [active]`.)
* Trong trường hợp có đối số, hãy kết hợp tên scope và đối số sao cho thật tự nhiên và dễ hiểu. 
* Cố gắng tránh việc đặt tên scope có bao gồm tên model. 

```ruby
# Không tốt
class User < ActiveRecord::Base
  scope :active_users, ->{where activated: true}
end
class Post < ActiveRecord::Base
  scope :by_author, ->author{where author_id: author.id}
end

# Tốt
class User < ActiveRecord::Base
  scope :active, ->{where activated: true}
end
class Post < ActiveRecord::Base
  scope :posted_by, ->author{where author_id: author.id}
end
```

* scope thì viết theo cách ngắn gọn của lambda. Nếu trong 1 dòng mà quá 80 kí tự thì nên cắt xuống dòng mới thích hợp để cho 1 dòng chỉ nên ít hơn 80 kí tự.

* Một khi đã dùng `has_many` hoặc `has_one` đối với một model thì nhất định phải khai báo `belongs_to` với model tương ứng.

### ActiveRecord Queries

* <a name="avoid-interpolation"></a>
  Tránh nhúng biến vào String của câu truy vấn, nó sẽ dễ dẫn đến SQL injection.
<sup>[[link](#avoid-interpolation)]</sup>

  ```Ruby
  # bad - param will be interpolated unescaped
  Client.where("orders_count = #{params[:orders]}")

  # good - param will be properly escaped
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Nếu câu truy vấn có nhiều hơn một tham số, ưu tiên dùng phương án
  tham-số-là-tên (named placeholders) thay vì dùng `?`.
  Việc này sẽ giúp bạn tránh việc nhầm lẫn, sai sót khi các tham số không
  được đặt đúng thứ tự.
<sup>[[link](#named-placeholder)]</sup>

  ```Ruby
  # okish
  Client.where(
    'created_at >= ? AND created_at <= ?',
    params[:start_date], params[:end_date]
  )

  # good
  Client.where(
    'created_at >= :start_date AND created_at <= :end_date',
    start_date: params[:start_date], end_date: params[:end_date]
  )
  ```

* <a name="find"></a>
  Khi cần query một record bằng `id`, ưu tiên dùng `find` hơn `where`
<sup>[[link](#find)]</sup>

  ```Ruby
  # bad
  User.where(id: id).take

  # good
  User.find(id)
  ```

* <a name="find_by"></a>
  Khi cần query một record bằng một vài attribute, ưu tiên dùng `find_by`
  hơn `where` và `find_by_attribute`.
<sup>[[link](#find_by)]</sup>

  ```Ruby
  # bad
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # bad
  User.find_by_first_name_and_last_name('Bruce', 'Wayne')

  # good
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```

* <a name="where-not"></a>
  Ưu tiên dùng `where.not` hơn là SQL thuần.
<sup>[[link](#where-not)]</sup>

  ```Ruby
  # bad
  User.where("id != ?", id)

  # good
  User.where.not(id: id)
  ```
* <a name="squished-heredocs"></a>
  Khi dùng query trực tiếp bằng `find_by_sql`, dùng heredocs cùng với `squish`.
  Bạn sẽ format code theo SQL tốt hơn (xuống dòng, thụt đầu dòng, tô màu từ khóa)
  và được hiển thị tốt ở nhiều tools như GitHub, Atom, and RubyMine.
<sup>[[link](#squished-heredocs)]</sup>

  ```Ruby
  User.find_by_sql(<<SQL.squish)
    SELECT
      users.id, accounts.plan
    FROM
      users
    INNER JOIN
      accounts
    ON
      accounts.user_id = users.id
    # further complexities...
  SQL
  ```

  [`String#squish`](http://apidock.com/rails/String/squish) sẽ tự bỏ thụt đầu dòng
  hay ký hiệu xuống dòng, nên khi log ra sẽ rất khó đọc, kiểu như này:
  ```
  SELECT\n    users.id, accounts.plan\n  FROM\n    users\n  INNER JOIN\n    acounts\n  ON\n    accounts.user_id = users.id
  ```


## Migration

* Quản lý phiên bản của ``` schema.rb ``` （hoặc là ``` structure.sql ```）
* Dùng rake db:schema:load thay vì rake db:migrate để khởi tạo CSDL rỗng.
* Sử dụng ``` rake db:test:prepare ``` để tạo database phục vụ cho test
* Nếu cần thiết lập giá trị mặc định, thì không thiết lập tại tầng ứng dụng mà thiết lập thông qua migration

```ruby
# không tốt - gán giá trị mặc định tại tầng ứng dụng
def amount
  self[:amount] or 0
end
```

Việc thiết lập giá trị mặc định của các bảng chỉ trong ứng dụng là cách làm tạm bợ, có thể sinh ra lỗi trong ứng dụng. Hơn nữa, ngoại trừ những ứng dụng khá nhỏ thì hầu như các ứng dụng đều chia sẻ database với các ứng dụng khác, thế nên nếu chỉ thiết lập trong ứng dụng thì tính nhất quán của dữ liệu sẽ không còn được đảm bảo.

* Quy định ràng buộc khoá ngoài. Mặc dù ActiveRecord không hỗ trợ điều này nhưng mà có thể dùng gem của bên thứ 3 như [schema_plus](https://github.com/lomba/schema_plus).

* Để thay đổi cấu trúc bảng như thêm column thì viết theo cách mới của Rails 3.1. Tóm lại, không sử dụng ``` up ``` hoặc ``` down ``` mà sử dụng ``` change ```. Rails nó tự biết để  `revert`, `rollback`. Kiểm tra kĩ các trường hợp phức tạp

```ruby
# cách viết cũ
class AddNameToPerson < ActiveRecord::Migration
  def up
    add_column :persons, :name, :string
  end

  def down
    remove_column :person, :name
  end
end

# cách viết mới tiện hơn
class AddNameToPerson < ActiveRecord::Migration
  def change
    add_column :persons, :name, :string
  end
end
```

* Không sử dụng class của model trong migration. Tại vì model class thì rất dễ bị thay đổi, khi đó xử lý của migration trước đây có thể bị ảnh hưởng.

## Views
### Qui tắc chung
  - Không gọi model trực tiếp từ view
  - Tuy nhiên vẫn có ngoại lệ cho phép trực tiếp gọi Model trong View như một master cho các thẻ như thẻ Select
  - Với các thao tác format phức tạp, nên để trong helper(hoặc decorator tùy project)
  - Các đoạn view hay dùng nên tạo partial views
  - Nếu render partial view cho một mảng các object nên dùng option: `collection` thay cho vòng lăp
  - Thêm 1 space bên trong các ``` <% ``` , ``` <%= ``` và ``` %> ```
    ```ruby
    #Cách viết không tốt
    <%foo%>
    <% bar%>
    <%=bar%>
    <%=bar %>

    #Cách viết tốt
    <% foo %>
    <%= bar %>
    ```
  - Nếu chèn url, dùng thẻ url_for, không dùng đường dẫn
  - Không sử dụng form_tag khi mà có thể sử dụng form_for
  - Dùng đường dẫn tuyệt đối khi gọi partial view. Ví dụ

    ```ruby
    <%= render :partial => "/jobs/show", :locals => { :job => @job } %>
    ```

### Assets
  - Thư mục ''app/assets'' để lưu các custom css, javascript.
  - Thư mục ''lib/assets'' để lưu các thư viện css, javascript hay dùng giữa các project.
  - Thư mục ''vendor/assets'' để lưu các thư viện javascript như jquery, bootstrap, ...
  - Nên sử dụng các phiên bản gem của các thư viện js, css như:  `jquery-rails, jquery-ui-rails, bootstrap-sass, zurb-foundation`
  - Trong CSS khi viết url thì dùng asset_url

## Mailer
  - Đặt tên mailer theo format SomethingMailer
  - Nên tạo 2 phiên bản text và html
  - Hạn chế thực gọi hàm gửi mail trực tiếp trong controller(gây delay khi generate view). Nên dùng các gem tạo backgound job như https://github.com/mperham/sidekiq|Sidekiq
  
## Active Support Core Extensions

* <a name="try-bang"></a>
  Ưu tiên dùng toán tử truy xuất an toàn của Ruby 2.3 `&.` hơn `ActiveSupport#try!`.
<sup>[[link](#try-bang)]</sup>

```ruby
# bad
obj.try! :fly

# good
obj&.fly
```

* <a name="active_support_aliases"></a>
  Ưu tiên dùng các hàm trong thư viện chuẩn của Ruby hơn
  các bí danh của `ActiveSupport`.
<sup>[[link](#active_support_aliases)]</sup>

```ruby
# bad
'the day'.starts_with? 'th'
'the day'.ends_with? 'ay'

# good
'the day'.start_with? 'th'
'the day'.end_with? 'ay'
```

* <a name="active_support_extensions"></a>
  Ưu tiên dùng thư viện chuẩn của Ruby hơn các extension của  `ActiveSupport`
  không thông dụng.
<sup>[[link](#active_support_extensions)]</sup>

```ruby
# bad
(1..50).to_a.forty_two
1.in? [1, 2]
'day'.in? 'the day'

# good
(1..50).to_a[41]
[1, 2].include? 1
'the day'.include? 'day'
```

* <a name="inquiry"></a>
  Ưu tiên dùng toán tử so sánh của `ActiveSupport` hơn `Array#inquiry`,
  `Numeric#inquiry` và `String#inquiry`.
<sup>[[link](#inquiry)]</sup>

```ruby
# bad - String#inquiry
ruby = 'two'.inquiry
ruby.two?

# good
ruby = 'two'
ruby == 'two'

# bad - Array#inquiry
pets = %w(cat dog).inquiry
pets.gopher?

# good
pets = %w(cat dog)
pets.include? 'cat'

# bad - Numeric#inquiry
0.positive?
0.negative?

# good
0 > 0
0 < 0
```

## Time

* <a name="tz-config"></a>
  Cấu hình `timezone` trong `application.rb`.
<sup>[[link](#tz-config)]</sup>

  ```Ruby
  config.time_zone = 'Eastern European Time'
  # optional - note it can be only :utc or :local (default is :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Không dùng `Time.parse`, thay vào đó hãy dùng `Time.zone.parse`
  `Time.parse` sẽ lấy `timezone` của hệ thống.
<sup>[[link](#time-parse)]</sup>

  ```Ruby
  # bad
  Time.parse('2015-03-02 19:05:37')

  # good
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Không dùng `Time.now`.
  `Time.now` sẽ lấy `timezone` của hệ thống.
<sup>[[link](#time-now)]</sup>

  ```Ruby
  # bad
  Time.now

  # good
  Time.zone.now # => Fri, 12 Mar 2014 22:04:47 EET +02:00
  Time.current # Tương tự nhưng viết ngắn hơn
  ```

## Bundle
  - Đặt gem theo group tùy vào môi trường.
  - Nên kèm theo phiên bản.

## Gem

### Gem chuẩn dùng trong các dự án JP/VN

  - https://docs.google.com/spreadsheets/d/1TkPkn0G_TaOXMO_Zzz7QS-lGx_Aj7gVlfNVp0YBm1mQ|Recommended gems list

### Gem thường dùng
  - https://github.com/gregbell/active_admin|active_admin: tạo giao diện admin, quản lý tìm kiếm, thêm xóa sửa database.
  - https://github.com/charliesome/better_errors|better_errors: hiện thông báo lỗi chi tiết, dễ nhìn hơn. Nên cài kèm theo gem https://github.com/banister/binding_of_caller|binding_of_caller.
  - https://github.com/flyerhzm/bullet|bullet: Kiểm tra các query, thông báo khi có vấn đề http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations|(N + 1) query
  - https://github.com/railsbp/rails_best_practices|rails_best_practices: Gem giúp tự động kiểm tra các best pracice của Rails.
  - https://github.com/CanCanCommunity/cancancan|cancancan: authorization gem, nếu cần gem ít chức năng và đơn giản hơn có thể dùng https://github.com/elabs/pundit|pundit
  - https://github.com/jnicklas/capybara|capybara: giúp test view.
  - https://github.com/jnicklas/carrierwave|carrierwave: gem hỗ trợ upload file. Nhiều chức năng và dễ customize hơn paperclip.
  - https://github.com/jnicklas/turnip|turnip: integrated test.
  - https://github.com/plataformatec/devemise|devemise: authentication gem.
  - https://github.com/thoughtbot/factory_girl|factory_girl: tạo data mẫu dùng để test.
  - https://github.com/EmmanuelOga/ffaker|ffaker: tạo dummy data.
  - https://github.com/indirect/haml-rails|haml: Haml gem.
  - https://github.com/amatsuda/kaminari|kaminari: giúp paging. Dễ customize hơn `will_paginate`.
  - https://github.com/rspec/rspec-rails|Rspec: unit test.
  - https://github.com/mperham/sidekiq|sidekiq: tạo background job.
  - https://github.com/plataformatec/simple_form|simple_form: enhance default `form_for`.

### Gem không nên dùng
  - http://rmagick.rubyforge.org/|rmagick: tốn bộ nhớ. Nên dùng https://github.com/probablycorey/mini_magick|mini_magick để thay thế.
  - http://www.zenspider.com/ZSS/Products/ZenTest/|autotest : tự động chạy test. Nên dùng https://github.com/guard/guard|guard và https://github.com/mynyml/watchr|watchr.
  - https://github.com/relevance/rcov|rcov: Không tương thích với ruby 1.9. Dùng https://github.com/colszowka/simplecov|simplecov thay thế.
  - https://github.com/cowboyd/therubyracer|therubyracer: tốn nhiều bộ nhớ(Đặc biệt ở môi trường production). Nên dùng `node.js`. Tham khảo tại http://stackoverflow.com/a/7481391| install node.js　https://devcenter.heroku.com/articles/rails-asset-pipeline#therubyracer|Heroku

## References
- Based on: http://wiki.zigexn.vn/doku.php?id=rails_coding_standard (Zigexn VeNtura internal only)
- https://github.com/rubocop-hq/rails-style-guide
- https://github.com/framgia/coding-standards/blob/master/vn/rails/standard.md
