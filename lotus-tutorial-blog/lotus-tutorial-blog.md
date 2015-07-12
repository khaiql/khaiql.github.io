# Lotus tutorial - Part 1: First look at Lotus

Như trong [bài viết trước](http://blog.siliconstraits.vn/rails-sang-lotus/), lotus là một web framework rất mới và cũng rất tiềm năng. Hôm nay chúng ta sẽ bắt tay vào làm thử một web application với Lotus để xem nó *lạ* như thế nào :D.

### Prerequisite (hay còn gọi là yêu cầu cơ bản)

Để có thể theo được chuỗi bài viết về lotus này, các bạn cần có kiến thức cơ bản về lập trình, biết ngôn ngữ Ruby và biết thêm Rails thì càng tốt. Environment mà các bạn cần phải setup bao gồm:

* Hệ điều hành: *nix, có thể chọn Linux hoặc MacOS tùy các bạn, nếu dùng Windows thì mình không chịu trách nhiệm về mọi sự mầu nhiệm có thể xảy ra trong quá trình thực hiện :D
* DBMS: bạn có thể chọn postgresql hay mysql gì cũng được, nhưng trong bài viết này mình sử dụng postgresql.
* git
* Terminal, web browser và text editor :troll-face:

### Bước 0 - Install lotus

Mở terminal lên và chạy lệnh `gem install lotusrb`. Tại thời điểm viết bài thì Lotus đang ở phiên bản 0.4.1. Hên xui sau này lotus đổi app structure một lần nữa như từ 0.3 lên 0.4 thì... bác sỹ bó tay :D

### Bước 1 - Tạo Lotus application

Nếu đã từng làm qua rails thì tạo mới một lotus application cũng tương tự: `lotus new [app_name]`. Ứng dụng của mình tên là  **lotus-tutorial** nên mình sẽ chạy lệnh: `lotus new lotus-tutorial`, bên cạnh đó mình thường sử dụng **Rspec** nên mình sẽ có thêm param khi khởi tạo app như sau: `lotus new lotus-tutorial --test=rspec`

Sử dụng text-editor để mở application folder được tạo ra bởi lotus. Ở đây mình sử dụng Sublime Text. Về cơ bản thì lotus cũng là ruby application nên nó có  nhiều chỗ tương đồng với rails, bạn sẽ vẫn có Gemfile, vẫn có bundle, rake, blah blah. Tuy nhiên, chỉ giống chừng đó thôi, còn lại thì khá là khác.

*Thường thì mấy bài tutorial chỉ tới đây sẽ kêu bạn chạy `lotus server` để xem cái hình dáng méo tròn của nó như nào, nhưng mà mình thì không, bạn tự làm đi, :)))))*

## Những khác nhau cơ bản giữa Lotus và Rails application
#### 1. App structure
![Lotus folder structure](/Users/sss/Documents/Cloud/OneDrive/Working/Silicon Straits Saigon/lotus-tutorial-blog/folder-structure.png)
* Folder **apps**: Một lotus application có thể chứa nhiều web apps, các web apps được đặt trong folder `apps`. Với cách tổ chức này, core team của lotus hi vọng sẽ giúp cho lotus app của bạn ready hơn với kiến trúc micro-services. Tức là, giả sử sau này app của bạn bự lên, cần phải scale nhiều thì bạn chỉ cần đơn giản copy cái app folder đó và bỏ sang một application khác, host ở nơi khác dễ dàng hơn. Mặc định, lotus sẽ tạo ra một web app đầu tiên giúp bạn và đặt tên là **web** :))

* Folder **lib**: khác với Rails, lib sẽ là nơi chứa các business logic của bạn, kể cả model. Chi tiết mình sẽ giới thiệu sau.

* Các file `.env`, `.env.development`, `.env.test`. Rails thường có xu hướng sử dụng các file YAML để config, còn các anh trong team phát triển lotus thì lại có xu hướng tránh sử dụng YAML. Nếu ở Rails bạn có file `database.yml` để cấu hình database thì ở lotus sẽ không có. Thay vào đó bạn phải sử dụng database url và đặt trong các file .env tương ứng với từng environment của bạn. Ví dụ .env.test là các biến environment sẽ được export khi chạy ở môi trường test.

* Những thứ lạ lẫm còn lại các bạn có thể tự tìm hiểu, nói chung là cũng vui á :D

#### 2. Architecture

Lotus được phát triển sau Rails, do đó đã nhìn thấy và khắc phục được một vài nhược điểm của Rails. Một trong những điểm mới được cộng đồng đánh giá rất tích cực đó là:

* `Lotus::Action`, bây giờ mỗi action sẽ tương ứng với một class chứ không còn là một `method` giống như bên Rails nữa. Điều này sẽ giúp các bạn tránh khỏi những controller rất là bự, với hàng đống action cũng như business logic lung tung ben trong một file controller nữa. Ngoài ra thì các tổ chức này cũng giúp cho việc testing trở nên đơn giản hơn. Ví dụ:

```
require 'spec_helper'
require_relative '../../../../apps/web/controllers/home/index'

describe Web::Controllers::Home::Index do
  let(:action) { Web::Controllers::Home::Index.new }
  let(:params) { Hash[] }

  it "is successful" do
    response = action.call(params)
    expect(response[0]).to eq 200
  end
end

```

Đoạn code trên được dùng để test `Index action` của `Home controller`. Ta có thể thấy việc gọi đến action rất đơn giản, giống như việc bạn test các unit bình thường khác. +1 cho lotus.

* **View và Template**: ở Rails, view và template được gộp vào làm một, chính vì vậy đôi khi bạn sẽ thấy trong file view lại chứa business logic để render. Sử dụng view helper cũng là một cách nhưng vẫn chưa phải là giải pháp được cộng đồng đánh giá là tốt ([xem thêm](http://nicksda.apotomo.de/2011/10/rails-misapprehensions-helpers-are-shit/)). Với Lotus, view là một class và tất cả các methods của class sẽ available trong template. Có thể nói view class như là một presentor vậy. Template sẽ chỉ có một nhiệm vụ duy nhất đó là hiển thị nội dung, sẽ không có câu `if-else` nào nằm trong template nữa (trừ khi bạn cố tình làm vậy). Và tất nhiên, một trong những benefit cực kỳ to lớn mà cách làm này mang lại, một lần nữa chính là testing, dé dé.

Nói chung phần trên mình chỉ giới thiệu những khác nhau cơ bản và nổi bật nhất giữa 2 frameworks, để có cái nhìn cụ thể và đầy đủ, các bạn có thể theo dõi tại [github](https://github.com/lotus/) hoặc trên [trang chính thức của lotus](http://lotusrb.org/).

*mới viết đó mà sao nhiều chữ quá -_-*

### Bước 1.5 - Action đầu tiên

Lotus cũng cung cấp một vài command giúp generate nhanh chóng như Rails. Để tạo một action mới, chúng ta có lệnh `generate action` với syntax như sau:

`lotus generate action [web-application-name] [controller-name]#[action-name]`

Từ version 0.4.1, các anh core team của lotus cho phép bạn thay dấu `#` bằng dấu `/` giữa `controller-name` và `action-name`. Nói chung là dùng cách nào cũng được.

*Lưu ý*: bạn nào xài `zsh` thì có thể đến bước này sẽ bị báo lỗi `lotus command not found`, lúc này hoặc là thay dấu `#` bằng dấu `/`, hoặc là chạy lệnh `exec bash` để quay trở lại sử dụng bash shell trước, rồi chạy lại lệnh `generate` là sẽ được. Nếu không được thì tắt cái tab terminal và lặp lại bước trên thì chắc được :D.

Ok, nói lang mang quá, giờ mình muốn tạo action `index` cho `home controller`, nên câu lệnh của mình sẽ là:

`lotus generate action web home#index`

Sau khi chạy lệnh, bạn sẽ thấy những dòng sau trên terminal:

```
insert  apps/web/config/routes.rb
create  spec/web/controllers/home/index_spec.rb
create  apps/web/controllers/home/index.rb
create  apps/web/views/home/index.rb
create  apps/web/templates/home/index.html.erb
create  spec/web/views/home/index_spec.rb
```

Tác dụng:
* Add thêm một path nữa vào router để trỏ vào action bạn vừa tạo.
* Tạo action class và spec tương ứng
* Tạo view class và spec tương ứng
* Tạo template file

*Tips*: nếu bạn không muốn sử dụng `erb` mà muốn sử dụng `haml`, `slim`, hay cái gì khác, bạn chỉ cần install gem tương ứng, sau đó mở file: `.lotusrc`, chỉnh lại value cho `template`, ví dụ:
```
architecture=container
test=rspec
template=slim
```
Từ lúc này trở đi, lotus sẽ generate file template sử dụng template engine mà bạn đã set.

### Bước 2 - Application layout

Mình qua bên bootstrap và lựa được  [layout này](http://getbootstrap.com/examples/jumbotron/), thấy cũng hợp hợp với  requirement của bài tut này nên sử dụng luôn (requirement sẽ được nói rõ trong bài sau :D).

Đầu tiên thì bạn phải download bootstrap về, để lần lượt các file trong folder `css`, `js` vào các folder `stylesheets` và `javascripts` tương ứng trong folder `public` của `web` app. Copy tiếp folder `fonts` của bootstrap bỏ vào `public` luôn. Download và lưu file stylesheet  `jumbotron` vào folder `public/stylesheets` luôn.

Tiếp theo, mở file `application.html.erb`, copy và paste html source của theme đã chọn ở trên vào đây. Chỉnh lại đường dẫn cho `css` và `javascripts`, ví dụ: `css/bootstrap.min.css` -> `stylesheets/bootstrap.min.css`. Viết thì thấy nhiều cơ mà không bao nhiêu, bạn nào siêng thì làm, không thì copy hẳn từ [github](https://github.com/siliconstraits/lotus-tutorial/blob/master/apps/web/templates/application.html.erb) bỏ vào cũng không sao :)).

### Chạy thử

Giờ mới đến lúc chạy lệnh `lotus server` để xem mặt mũi lotus nó như nào nè (nhớ chạy lệnh ở folder root của lotus app nha). Mặc định lotus server sử dụng port 2300, cho nên mấy bạn mở browser lên, nhập vào `localhost:2300`. Kết quả là...

![Not found T_T](/Users/sss/Documents/Cloud/OneDrive/Working/Silicon Straits Saigon/lotus-tutorial-blog/not-found-root.png)

Bây giờ mấy bạn mở file `routes.rb` lên sẽ thấy:
```
get '/home', to: 'home#index'
# Configure your routes here
# See: http://www.rubydoc.info/gems/lotus-router/#Usage
```

Tức là nó chỉ có route trỏ vào `/home` thôi, chứ chưa có route nào trỏ vào `/` hết, giờ có hai cách, hoặc là bạn nhập đường dẫn chính xác: `localhost:2300/home`, hoặc là thêm một dòng nữa vào file `routes.rb` như sau:
```
get '/home', to: 'home#index'
get '/', to: 'home#index'
```

Rồi, bây giờ refresh lại trang bạn sẽ thấy cái bạn muốn thấy :D.
![Hello world nè](/Users/sss/Documents/Cloud/OneDrive/Working/Silicon Straits Saigon/lotus-tutorial-blog/hello-word.png)

### Tạm kết

Nói chung tâm lý của mình đọc mấy bài tut mà dài quá + toàn chữ là chữ thì hơi ngán nên chắc mọi người cũng vậy, hôm nay dừng ở đây được rồi :D. Mấy bạn có thể vào [github repo](https://github.com/siliconstraits/lotus-tutorial) để lấy source code về xem trước. Nếu thấy thích thì subscribe blog để được notify khi có part 2 nha :D. Chào thân ái và quyết thắng!

P.S: đừng quên `git commit` nha.
