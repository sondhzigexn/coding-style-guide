# Quy chuẩn coding với Ruby
Ruby style guide

## Các qui tắt chung
  - Dùng 2 space
  - Dùng space trước và sau các dấu +,-,x,/, {, }, =
  - Không dùng space sau dấu (, [ và trước dấu ), ]
  - Thêm dấu _ ở các số lớn. Vd: 100000 --> 100_000

## Cú pháp
  - Dùng ( ) ở khai báo hàm có truyền tham số, không dùng () trong trường hợp hàm không nhận tham số

### Các câu lệnh điều kiện
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

### Câu lệnh lặp
  - Hạn chế dùng câu for, nên dùng câu each
  - Dùng ''until'' thay cho  ''while not''

### Khai báo và gọi hàm
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

  - Tên biến, tên hàm, symbol: `a_var_name`, `get_prime`, `a_symbol`.
  - Tên class, module: SomeClass, SomeModule
  - Hằng số: SOME_CONST

## Class, Module
  - Dùng include, extend ngay sau khai báo class
  - Dùng `attr_reader`, `attr_writer`, `attr_accessor` thay cho các hàm set, get
  - Hạn chế dùng `self << class`, nên dùng `def self.a_func`

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

## References
- Based on: http://wiki.zigexn.vn/doku.php?id=coding_rule (Zigexn VeNtura internal only)
- https://github.com/rubocop-hq/ruby-style-guide
- https://github.com/github/rubocop-github/blob/master/STYLEGUIDE.md
- https://github.com/framgia/coding-standards/blob/master/vn/ruby/standard.md
