---
layout:     post
title:      Multiplayer Game Architecture (P1)
date:       2015-01-23 17:30:00
---

#### Lời nói đầu


Tôi mới quay lại chơi Dota 2 trong vài tháng gần đây (trước đó tôi đã bỏ Dota khá là lâu rồi). Điều tôi thực sự ấn tượng với Dota 2 là trải nghiệm mà Valve mang lại cho người chơi. Thực sự là rất tuyệt vời.


Ngoài các vấn đề về tính năng, thứ làm tôi ấn tượng chính là tôi có thể chơi game rất tốt với ping xấp xỉ 100 ms, điều mà trước đây khi tôi chơi Dota trên GG chắc chắn không bao giờ có được. Đôi khi tôi tự hỏi Valve làm thế nào mà hay vậy, tuy nhiên bản tính tôi lười nên bẵng cái là tôi lại quên ngay. À thực ra tôi có mượn công ty quyển _"Massively Multiplayer Game Development"_ về tìm hiểu xem sao, tuy nhiên quyển sách đó không có những thứ tôi cần tìm hiểu nên tôi gác nó vào một xó và cũng quên luôn cả vụ Valve với Dota 2. ![]({{ site.url }}/images/emo/shame.gif)


Gần đây, sếp tôi giao cho tôi tìm hiểu về kiến trúc hệ thống game trực tuyến nhiều người chơi. Chắc thấy tôi mượn quyển sách kia về nên nghĩ tôi học hỏi được gì đây mà ![]({{ site.url }}/images/emo/cry.gif). Thôi không sao, cũng là dịp để tôi tìm hiểu lại vấn đề tôi từng phân vân, challenge accepted! ![]({{ site.url }}/images/emo/boss.gif)


Lần này tôi quyết định làm bạn với Google, bỏ thời gian tìm hiểu chính quy từ sách thì quả thật là vừa tốn công có khi lại không thu được kết quả như mong muốn. Và may mắn là tôi đã tìm thấy series của [Gabriel Gambetta](http://www.gabrielgambetta.com/fast_paced_multiplayer.html). Đây thực sự là thứ tôi đang tìm kiếm - series về các kiến trúc client-server cho game nhiều người chơi với tần suất cập nhật lớn (high update rates). Các bài viết trong series tập trung nói về việc làm thế nào để xử lý được các vấn đề liên quan đến lag, đồng bộ giữa các client và server. Ngoài ra, trong series còn có **Sample Code** và **Live Demo**. Quá tuyệt vời!! ![]({{ site.url }}/images/emo/beauty.gif)


Và tôi thấy rằng mình cần chia sẻ điều thú vị này cho các game dev khác nên tôi quyết định sẽ dịch series này. Thực ra tôi định viết lại theo ý hiểu của tôi, nhưng tôi thấy bài viết gốc của tác giả dẫn dắt người đọc đã quá hay rồi nên thôi tôi dịch. Tránh nhiều người lại lấy blog của tôi để thay cho thuốc ngủ ![]({{ site.url }}/images/emo/sweat.gif)


Tôi sẽ cố gắng dịch một số khái niệm và kèm theo bản gốc, tuy nhiên một số tôi không thể dịch hoặc thấy dịch ra thì không được hay nên tôi sẽ giữ nguyên.


### Part I: Giới thiệu


Đây là bài đầu tiên trong series các bài viết tìm hiểu về kĩ thuật và thuật toán trong game nhiều người chơi tốc độ cao (fast-paced multiplayer game). Nếu bạn đã quen thuộc với các khái niệm trong multiplayer games, bạn hoàn toàn có thể bỏ qua bài viết này để đến với bài viết tiếp theo trong series.


#### Gian lận


Khi làm game một người chơi (single-player game), nhà phát triển thường không quan tâm đến vấn đề hack/cheat của người chơi, bởi vì hành động đó không làm ảnh hưởng đến bất kì ai, ngoài chính họ. Người chơi gian lận sẽ không trải nghiệm game theo đúng cách mà bạn sẽ định ra, tuy nhiên đó là trò chơi của riêng họ, và họ có quyền được trải nghiệm trò chơi theo cách mà họ muốn.


Tuy nhiên, trò chơi nhiều người chơi (multiplayer-games) thì lại khác. Trong bất kì sự cạnh tranh nào, người gian lận không chỉ tạo ra lợi thế cho chính họ, mà còn tạo ra sự bất lợi cho người chơi khác. Là một nhà phát triển game, chúng ta chắc chắn không muốn điều này xảy ra, nếu không người chơi sẽ bỏ game của chúng ta mà thôi.


Có rất nhiều cách để ngăn chặn việc gian lận của người chơi, nhưng chung quy lại điều quan trọng nhất chính là: **_"Đừng bao giờ tin vào người chơi"_**. Hay nói cách khác, người chơi sẽ luôn tìm cách để gian lận.


#### Authoritative servers and dumb clients


Một giải pháp rất đơn giản đó là, mọi sự kiện trong game của bạn đều xảy ra đưới sự kiểm soát của server và khi đó client chỉ có quyền theo dõi diễn biến của game. Nói cách khác, game client gửi hành động (ví dụ như hành động bấm phím) lên cho server, server thực hiện hành động và gửi lại kết quả cho các clients. Việc này được gọi là sử dụng _máy chủ quyền năng_ (authoritative server), bởi vì thứ duy nhất có quyền quyết định bất chấp mọi thứ xảy ra trên thế giới chính là server.


Tất nhiên, server của bạn có thể bị khai thác từ các lỗ hổng bảo mật, tuy nhiên vấn đề đó nằm ngoài phạm vi của bài viết. Việc sử dụng authoritative server sẽ ngăn chặn được phần lớn gian lận. Ví dụ, khi bạn không tin vào thông tin về "máu" của player được cung cấp từ client: một client có thể gian lận thông tin bằng cách chỉnh sửa các giá trị và nói rằng player đó có 10000% máu, nhưng server biết rằng nó chỉ là 10% - player đó sẽ chết khi bị tấn công, không cần quan tâm tới bất cứ thông tin gì mà client gian lận gửi lên.


Bạn sẽ không được phép tin tưởng vào client thông tin về vị trí của người chơi trên bản đồ. Nếu không, một client gian lận sẽ nói với server **"Tôi đang ở vị trí (10, 10)"** và 1s sau đó **"Tôi đang ở vị trí (20, 10)"**, điều đó có thể dẫn đến việc đi xuyên qua tường hay đi nhanh hơn những người chơi khác. Thay vào đó, server biết rằng người chơi ở vị trí (10, 10), client nói với server rằng **"Tôi muốn di chuyển một bước sang phải"**, server cập nhật trạng thái game với vị trí mới của người chơi là (11, 10), và sau đó trả lời người chơi **"Bạn ở vị trí (11, 10)"**: 


![]({{ site.url }}/images/fpm1-01.png)


Tóm lại, trạng thái của game sẽ chỉ được quản lý bởi server. Clients gửi hành động của họ cho server. Server cập nhật trạng thái của game theo định kỳ, và gửi trạng thái mới cho clients để hiển thị trên màn hình.


#### Xử lý với vấn đề của mạng


Giải pháp dumb client hoạt động ổn đối với game slow turn based, ví dụ game chiến thuật hoặc poker, và hoạt động tốt trong hệ thống LAN, khi tốc độ truyền dữ liệu là tức thời. Tuy nhiên, phương án này sẽ gặp vấn đề khi áp dụng cho những game tốc độ nhanh (fast-paced) trên mạng internet diện rộng.


Giả sử bạn đang ở San Francisco, kết nối tới server đặt tại New York. Khoảng cách xấp xỉ 4,000 km. Không gì có thể di chuyển nhanh hơn ánh sáng, kể cả bytes trên Internet. Ánh sáng truyền đi với vận tốc xấp xỉ 300,000 km/s, tức là cần 13 ms để di chuyển 4,000 km.


Nghe có vẻ khá là nhanh, tuy nhiên đây chỉ là giả định trong trường hợp tốt nhất - khi dữ liệu được truyền với tốc độ của ánh sáng trên đường thẳng - trường hợp hầu như không thể xảy ra. Trong thực tế, dữ liệu được truyền thông qua rất nhiều bước nhảy (được gọi là _hops_) từ router này đến router khác, mà phần lớn trong đó không phải là tốc độ ánh sáng. Ngoài ra, chính routers cũng tạo ra một chút delay, khi mà gói tin phải được sao chép (copy), kiểm tra (inspect), chuyển tiếp (reroute).


Với tất cả những yếu tố trên, giả sử dữ liệu cần 50 ms để di chuyển từ client tới server. Đây gần như là trường hợp tốt nhất trên thực tế. Vậy điều gì sẻ xảy ra khi bạn ở NY và kết nối tới server đặt tại Tokyo? Nếu mạng bị nghẽn vì một lý do nào đó? Delay sẽ là 100, 200 hoặc có thể là 500 ms.


Quay trở lại ví dụ của chúng ta, client gửi hành động lên server ("**Tôi đã ấn phím mũi tên sang phải**"). Server nhận được sau đó 50 ms. Giả sử server thực hiện yêu cầu và gửi lại trạng thái đã cập nhật ngay lập tức, client nhận được trạng thái mới của game ("**Vị trí hiện tại của bạn là (1, 0)**") sau 50 ms.


Bạn sẽ cảm thấy như thế nào khi mà bạn ấn phím mũi tên sang trái nhưng không có hiện tượng gì xảy ra trong phần mười giây. Rồi sau đó, nhân vật của bạn di chuyển một bước sang bên phải. Sự trễ (lag) giữa hành động và kết quả của bạn có thể không nhiều, nhưng nó cũng đủ để nhận ra - và tất nhiên, điều đó thực sự làm cho game của chúng ta không thể chơi được.


#### Tóm tắt


Game trực tuyến nhiều người chơi thực sự rất thú vị, nhưng cũng hứa hẹn nhiều tầng lớp vấn đề cần giải quyết. Kiến trúc authoritative server là rất tốt để ngăn chặn phần lớn gian lận, nhưng có thể làm cho game không thể đáp ứng được người chơi.


Trong những bài viết tiếp theo, chúng ta sẽ tìm hiểu làm thế nào để xây dựng một hệ thống dựa trên authoritative server, trong khi vẫn tối giản được độ trễ (delay), để người chơi được trải nghiệm như là đang chơi một single player game.

_Theo [Gabriel Gambetta](http://www.gabrielgambetta.com/fast_paced_multiplayer.html)_