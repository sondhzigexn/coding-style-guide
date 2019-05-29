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

## Views
### Qui tắc chung
  - Không gọi model trực tiếp từ view.
  - Với các thao tác format phức tạp, nên để trong helper(hoặc decorator tùy project).
  - Các đoạn view hay dùng nên tạo partial views.
  - Nếu render partial view cho một mảng các object nên dùng option: `collection` thay cho vòng lăp
  - Nên dùng thẻ ``<%- -%>``
  - Nếu chèn url, dùng thẻ url_for, không dùng đường dẫn.
  - Dùng đường dẫn tuyệt đối khi gọi partial view. Ví dụ


```ruby
<%= render :partial => "/jobs/show", :locals => { :job => @job } %>
```

### Assets
  - Thư mục ''app/assets'' để lưu các custom css, javascript.
  - Thư mục ''lib/assets'' để lưu các thư viện css, javascript hay dùng giữa các project.
  - Thư mục ''vendor/assets'' để lưu các thư viện javascript như jquery, bootstrap, ...
  - Nên sử dụng các phiên bản gem của các thư viện js, css như:  `jquery-rails, jquery-ui-rails, bootstrap-sass, zurb-foundation`

## Mailer
  - Đặt tên mailer theo format SomethingMailer
  - Nên tạo 2 phiên bản text và html
  - Hạn chế thực gọi hàm gửi mail trực tiếp trong controller(gây delay khi generate view). Nên dùng các gem tạo backgound job như https://github.com/mperham/sidekiq|Sidekiq

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
