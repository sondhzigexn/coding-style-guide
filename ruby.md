# Quy chuẩn coding với Ruby
Ruby style guide

## Các qui tắt chung
  - Căn lề dùng 2 space
  - Dùng space trước và sau các dấu +,-,x,/, {, }, =
  - Không dùng space sau dấu (, [ và trước dấu ), ]
  - Thêm dấu _ ở các số lớn. Vd: 100000 --> 100_000
  - Nếu class, method không có nội dung thì dùng inline format
 
    ```ruby
    # bad
    class FooError < StandardError
    end

    # good
    class FooError < StandardError; end

    # good
    FooError = Class.new(StandardError)
    ```
  - Thêm một dòng trống giữa các phương thức, và các nhóm xử lý logic. 
  
    ```ruby
    def some_method
      data = initialize(options)

      data.manipulate!

      data.result
    end

    def some_method
      result
    end
    ```
  - Sau ký tự comment out `#` đặt 1 khoảng trắng
    ```ruby
    #this is bad comment

    # this is good comment
    ```
  - Tránh dùng khoảng trắng thừa (thường gặp ở cuối dòng)
  - Cuối file nên có thêm một dòng trống

## Cú pháp
  - Chỉ dùng () ở khai báo hàm có truyền tham số
  - Nếu block chỉ làm một việc thì ưu tiên dùng shorthand
  
    ```ruby
    # bad
    names.map { |name| name.upcase }

    # good
    names.map(&:upcase)
    ```
  - Nếu block 1 dòng thì dùng `{...}`, nếu nhiều dòng thì dùng `do...end`
   
    ```ruby
    # bad
    names.each do |name|
    puts name
    end

    # good
    names.each { |name| puts name }

    # bad
    names.select do |name|
    name.start_with?('S')
    end.map { |name| name.upcase }

    # good
    names.select { |name| name.start_with?('S') }.map(&:upcase)
    ```
  - Trong trường hợp có thể bỏ được return thì bỏ
    ```ruby
    # bad
    def some_method(some_arr)
      return some_arr.size
    end

    # good
    def some_method(some_arr)
      some_arr.size
    end
    ```
  - Dùng kiểu gán rút gọn khi có thể
    ```ruby
    # bad
    x = x + y
    x = x * y
    x = x**y
    x = x / y
    x = x || y
    x = x && y

    # good
    x += y
    x *= y
    x **= y
    x /= y
    x ||= y
    x &&= y
    ```
  - Ưu tiên dùng map hơn collect, find hơn detect, select hơn find_all, reduce hơn inject và size hơn length
  - Đừng dùng count để thay cho size. Với đối tượng Enumerable hay là Array, nó sẽ duyệt qua từng phần tử để đếm số lượng phần tử, sẽ rất tốn thời gian
   
    ```ruby
    # bad
    some_hash.count

    # good
    some_hash.size
    ```
  - Dùng flat_map thay cho map + flatten. Cái này không áp dụng cho mảng sâu hơn 2 cấp
    
    ```ruby
    # bad
    all_songs = users.map(&:songs).flatten.uniq

    # good
    all_songs = users.flat_map(&:songs).uniq
    ```
  - Ưu tiên dùng reverse_each hơn reverse.each vì một số lớp mà include Enumerable sẽ hoạt động hiệu quả hơn
  
    ```ruby
    # bad
    array.reverse.each { ... }

    # good
    array.reverse_each { ... }
    ```
    
#### Các câu lệnh điều kiện
  - Không dùng and, or. Dùng &&, ||
  - Dùng ''unless'' thay cho ''if not''
  - Không dùng ''unless ... else''
  - Không dùng () ở điều kiện trong câu if, unless
  - Dùng các hàm số có sẵn như ''x.even?, x.odd?, x.nil?, x.zero?'' thay cho các câu so sánh ''x % 2 == 0, x % 2 == 1, x == nil, x == 0''
  - Trong trường hợp câu if/unless/while/until chỉ có một dòng, dùng inline format. Ví dụ

    ```ruby
    do_something if some_condition

    thay cho

    if some_condition  
      do_something  
    end
    ```
  - Dùng toán tử 3 ngôi thay cho if/else/end
  
    ```ruby
    # bad
    result = if some_condition then something else something_else end

    # good
    result = some_condition ? something : something_else
    ```
  - Khi kiểm tra điều kiện thì không cần kiểm tra với nil trừ khi làm với boolean.
    ```ruby
    # bad
    do_something if !something.nil?
    do_something if something != nil

    # good
    do_something if something

    # good - dealing with a boolean
    def value_set?
      !@some_boolean.nil?
    end
    ```
    
#### Câu lệnh lặp
  - Hạn chế dùng câu for, nên dùng câu each
  - Dùng ''until'' thay cho  ''while not''

#### Khai báo và gọi hàm
  - Bỏ () khi gọi các hàm không tham số
  - Nếu hàm có tham số, để các tham số trong (). Ví dụ: `foo(param1, param2)`.
  - Không cần dùng return ở  dòng cuối cùng trong khai báo hàm. Ví dụ:

```ruby
def some_method(some_arr)
  some_arr.size
end
```
  - Không dùng space giữa tên hàm và dấu (
  - Dùng -> thay cho từ khóa lambda
  - Dùng các câu inline if thay cho các câu if lồng nhau. Ví dụ:

```ruby
# Bad
def compute_thing(thing)
  if thing[:foo]
    update_with_bar(thing)
      if thing[:foo][:bar]
        partial_compute(thing)
      else
        re_compute(thing)
      end
  end
end

# Good
def compute_thing(thing)
  return unless thing[:foo]
  update_with_bar(thing[:foo])
  return re_compute(thing) unless thing[:foo][:bar]
  partial_compute(thing)
end
```

  - Khi khai báo biến array, hash, dùng cấu trúc: `a_arr = [], a_hash = {}`  thay cho `a_arr = Array.new, a_hash = Hash.new`

## Qui tắc đặt tên

  - Tên biến, tên hàm, symbol: `a_var_name`, `get_prime`, `a_symbol`
  - Tên class, module: SomeClass, SomeModule
  - Hằng số: SOME_CONST
  - Phương thức trả về boolean thì nên có thêm `?` đằng sau (vd: `Array#empty?`)
  - Tránh đặt tiền tố cho tên phương thức với các động từ bổ trợ như is, does, hay can. Những từ này không cần thiết và nghĩa quá chung chung, không đồng nhất với các phương thức boolean của ngôn ngữ: empty? hay include?
    
    ```ruby
    # bad
    class Person
      def is_tall?
        true
      end

      def can_play_basketball?
        false
      end

      def does_like_candy?
        true
      end
    end

    # good
    class Person
      def tall?
        true
      end

      def basketball_player?
        false
      end

      def likes_candy?
        true
      end
    end
    ```
  - Tên của những phương thức *nguy hiểm* (vd: xóa dữ liệu DB, ghi vào file, sửa self, raise exception...) nên kết thúc bằng một dấu `!` 
  
    ```Ruby
    # bad - there is no matching 'safe' method
    class Person
      def update!
      end
    end

    # good
    class Person
      def update
      end
    end

    # good
    class Person
      def update!
      end

      def update
      end
    end
    ```
  - Nếu có phương thức *nguy hiểm* thì nên có thêm phương thức *an toàn*
  
    ```Ruby
    class Array
      def flatten_once!
        res = []

        each do |e|
          [*e].each { |f| res << f }
        end

        replace(res)
      end

      def flatten_once
        dup.flatten_once!
      end
    end
    ```
  
## Class, Module
  - Dùng include, extend ngay sau khai báo class
  - Dùng `attr_reader`, `attr_writer`, `attr_accessor` thay cho các hàm set, get
  - Tránh sử dụng biến class `@@` trừ khi thực sự cần thiết
  - Viết protected methods trước private methods. Lúc đó, khi định nghĩa các protected, private method, căn lề trùng với public method và đặt dòng trắng trên các protected, private methods này, không đặt dòng trắng ở bên dưới
    ```ruby
    class SomeClass
      def public_method
        # ...
      end

      protected
      def protected_method
        # ...
      end

      private
      def private_method
        # ...
      end
    end
    ```

## Collection
  - Dùng %w để khai báo mảng các chuỗi
  - Dùng %i để khai báo mảng các symbol
  - Nên dùng symbol làm key trong hash. Ví dụ: { :one => 1, :two => 2 }

## String
  - Dùng `#{}` thay cho nối chuỗi. Vd: `email_with_name = "#{ user.name } <#{ user.email }>"`
  - Dùng %Q để khai báo chuỗi có ', ", #{}
  - Dùng + để cắt khi chuỗi quá dài. Ví dụ:

    ```ruby
    long_str = 'a very long long long long long long' + 
               'long long long long string'
    ```
  - Khi sử dụng here document, delimiter được căn lề thẳng với lệnh gán.
    ```ruby
    module AttrComparable
      module ClassMethods
        def attr_comparable *attrs
          class_eval <<-DELIM
            attrs.each do |attr|
              define_method(attr.to_s<<'?'){|param| self.send(attr) == param }
            end
          DELIM
        end
      end
    #...
    ```
## Comparison
  - Khi so sánh biến số với một giá trị khác như số thực hay hằng số thì viết biến số sang bên phải
    ```ruby
    greeting = "Hello!"

    # bad
    if greeting == "Hola!"
      ...
    end

    # good
    if "Hola!" == greeting
      ...
    end
    ```

## References
- Based on: http://wiki.zigexn.vn/doku.php?id=coding_rule (Zigexn VeNtura internal only)
- https://github.com/rubocop-hq/ruby-style-guide
- https://github.com/github/rubocop-github/blob/master/STYLEGUIDE.md
- https://github.com/framgia/coding-standards/blob/master/vn/ruby/standard.md
