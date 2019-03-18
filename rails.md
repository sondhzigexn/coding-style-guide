# Quy chuẩn coding với Ruby on Rails
Ruby on Rails style guide
## Config chung
### Initializer
  - Để các code khởi tạo trong thư mục `config/initializers`. Các đoạn code để trong thư mục này sẽ tự động execute khi server khởi động.
  - Để các code khởi tạo thành các file riêng theo gem. Ví dụ: carrierwave.rb, devise.rb...

### Global variable
  - Các biến global như email dùng để gửi mail, hostname, ... nên để trong các file `config/enviroments/development.rb`, `config/enviroments/production.rb`
  - Trường hợp các biến dùng cho tất cả môi trường thì định nghĩa trong file `application.rb`
  - Có thể dùng gem figaro

## Routing and links
### Routes.rb

  - Không dùng `match`, cần chỉ cụ thể phương thức như `post`, `get`, `put`, ...
  - Dùng `link_to "Display text, name_route_url` thay cho `link_to "Display text, controller: 'name', action: 'route'`.

## Models
### Migration file
  - Đặt timestamps ngay sau cột Primary Key
  - Đặt index trên các cột Foreign Key

### ActiveRecord
  - Hạn chế đổi các giá trị mặc định của Model như tên bảng, tên cột khóa chính(id).
  - Thứ tự các phần trong một model:
    - `default_scope`, `scope`.
    - Các khai báo hằng số.
    - `attr_accessor`, `attr_accessible`, ...
    - Các hàm quan hệ(`has_many`, `belongs_to`, `has_one`, ...).
    - Các hàm `validates`.
    - Các hàm callback(`before_save`, ...).
    - Các hàm macro của các gem(như các macro của Devise).
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
<%=render :partial => "/jobs/show", :locals => { :job => @job } %>
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
