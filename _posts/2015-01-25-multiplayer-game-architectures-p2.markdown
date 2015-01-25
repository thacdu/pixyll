---
layout:     post
title:      Multiplayer Game Architecture (P2)
date:       2015-01-23 17:30:00
---

### Part II: Client-Side Prediction and Server Reconciliation


#### Giới thiệu


Trong phần trước, chúng ta đã tìm hiểu mô hình client-server với authoritative server và dumb client với nhiệm vụ gửi hành động cho server và thể hiện trạng thái đã cập nhật được server gửi lại.


Mô hình này dẫn đến việc delay giữa mệnh lệnh của người chơi và sự thay đổi trên màn hình. Ví dụ, khi người chơi ấn phím mũi tên sang phải, phải nửa giây sau nhân vật mới bắt đầu di chuyển. Đó là bởi vì mệnh lệnh của người chơi cần được chuyển đến server, sau đó server xử lý, tính toán trạng thái mới của game và khi client nhận được trạng thái này thì sự thay đổi mới được thể hiện.


![Effect of network delays]({{ site.url }}/images/fpm2-01.png)


Trong môi trường mạng diện rộng như Internet, khi độ trễ (delay) có thể lên đến phần mười giây, game của chúng ta có thể không đáp ứng được tốt nhất, hoặc trong trường hợp xấu nhất, game không thể chơi được. Trong bài viết này, chúng ta sẽ đi tìm hiểu phương pháp tối giản hoặc có thể loại bỏ vấn đề trên.


#### Client-side prediction


Mặc dù có thể có một bài người chơi gian lận, nhưng phần lớn thời gian game server sẽ xử lý những yêu cầu hợp lệ (từ những người chơi không gian lận hoặc những thời điểm họ không gian lận). Điều đó có nghĩa trong phần lớn thời gian, trạng thái game sẽ được cập nhật đúng như mong muốn của chúng ta. Tức là, nếu nhân vật của bạn đang ở vị trí (10, 10) and bạn ấn phím mũi tên sang phải, trạng thái mới sẽ là (11, 10).


Chúng ta có thể coi đó là mặt tích cực, nếu game world của chúng ta là xác định (deterministic) (tức là, với trạng thái hiện tại và hành động đầu vào, chúng ta hoàn toàn có thể dự đoán được chính xác kết quả).


Giả sử lag đang là 100 ms, và animation di chuyển nhân vật từ ô này sang ô kế tiếp cần 100 ms. Vậy thì, tổng thời gian của hành động sẽ là 200 ms.



![]({{ site.url }}/images/fpm2-02.png)


Vì game world là deterministic nên chúng ta giả sử hành động mà chúng ta gửi lên server sẽ được thực thi thành công. Khi đó client có thể dự đoán được trạng thái của game world sau khi thực thi, và phần lớn là nó sẽ đúng.


Thay vì gửi mệnh lệnh và chờ nhận trạng thái mới để bắt đầu rendering thì chúng ta có thể gửi mệnh lệnh và bắt đầu rendering trạng thái kết quả, trong khi chờ server gửi trạng thái thực của game - thường chính là trạng thái mà chúng ta dự đoán.


![]({{ site.url }}/images/fpm2-03.png)


Và bây giờ hoàn toàn không có delay giữa hành động của người chơi và sự thể hiện trên màn hình, trong khi đó server vẫn là authoritative (nếu hacked client gửi hành động không hợp lệ, chỉ có sự thể hiện trên màn hình của họ là không chính xác, còn trạng thái của server sẽ không bị ảnh hưởng, và những gì người chơi khác thấy vẫn là chính xác).


Vấn đề về việc đồng hộ hóa


Trong ví dụ trên, chúng ta đã chọn những con số để mọi thứ diễn ra một cách đúng đắn. Tuy nhiên, thử thay đổi một chút như sau: chúng ta có lag là 250 ms, và thời gian thực hiện animation là 100 ms. Người chơi ấn phím mũi tên sang phải 2 lần, để di chuyển sang phải 2 ô. Sử dụng kĩ thuật vừa nêu trên, điều gì sẻ xảy ra:


![]({{ site.url }}/images/fpm2-04.png)


Có một vấn đề khá thú vị tại thời điểm **t=250 ms**, khi trạng thái mới game đến. Trạng thái dự đoán tại client là **x = 12**, nhưng server lại nói rằng trạng thái mới là **x = 11**. Vì server là authoritative nên client bắt buộc phải di chuyển nhân vật tới vị trí **x = 11**. Tuy nhiên, trạng thái mới tại thời điểm **t = 350** là **x = 12**, nên nhân vật lại di chuyển quay lại vị trí **x = 12**


Dưới góc nhìn của người chơi, họ ấn 2 lần phím mũi tên phải; nhân vật di chuyển 2 lần sang phải, đứng yên tại chỗ 50 ms rồi nhảy ngược lại sang trái 1 ô, đứng tại chỗ 100 ms, rồi lại nhảy sang phải 1 ô. Điều này tất nhiên là không thể chấp nhận được rồi.


#### Máy chủ điều phối (Server reconciliation)


Chìa khóa để giải quyết vấn đề là client thể hiện game world tại thời điểm hiện tại, nhưng vì lag nên cập nhật từ server gửi về lại là trạng thái game tại thời điểm trước đó. Và tại thời điểm server gửi trạng thái cập nhật đó, server vẫn chưa thực hiện tất cả những mệnh lệnh được gửi từ client.


Điều này không quá khó để giải quyết. Đầu tiên, client thêm sequence number vào mỗi request; trong ví dụ của chúng ta, hành động ấn phím đầu tiên là request #1, hành động thứ 2 là request #2. Sau đó, server sẽ thêm sequence number vào phản hồi tương ứng với mệnh lệnh từ client.


![]({{ site.url }}/images/fpm2-05.png)


Giờ đây, tại thời điểm **t = 250**, server nói rằng "dựa vào những gì tôi nhận được từ request #1, vị trí của bạn là x = 11". Vì server là authoritative nên vị trí của nhân vật là **x = 11**. Giả sử client giữ lại bản sao của những request đã được gửi lên server. Dựa vào trạng thái mới của game, client biết server đã thực hiện request #1, client sẽ xóa bỏ bản sao đó. Nhưng client cũng biết rằng server vẫn chưa gửi lại kết quả của request #2, do vậy client thực hiện dự đoán lại vị trí hiện tại của game dựa trên trạng thái cuối cùng nhận được từ server, và những hành động mà server chưa thực thi.


Vậy, tại thời điểm **t = 250**, client nhận được "**x = 11, request thực thi cuối cùng = #1**". Client xóa bỏ các bản sao cho đến request #1 - nhưng giữ lại bản sao của #2 (chưa được server ghi nhận). Client cập nhật trạng thái trong game dựa vào những gì server gửi về, **x = 11**, và sau đó tính toán lại các trạng thái dự đoán - trong trường hợp này, hành động #2, "di chuyển sang phải". Kết quả cuối cùng, **x = 12**, đúng với dự đoán từ trước.


Tiếp tục với ví dụ trên, tại thời điểm **t = 350** trạng thái game mới được nhận từ server; nói rằng "x = 12, request thực thi cuối cùng = #2". Client xóa bỏ bảo sao request #2, không còn hành động nào chưa thực thi, client tính toán và đưa ra kết quả chính xác.


#### Lặt vặt


Ví dụ đưa ra phía trên nói về việc di chuyển, tuy nhiên nguyên tắc này có thể áp dụng với hầu hết các hành động khác. Ví dụ, trong một game turn-based, khi một người chơi tấn công người chơi khác, bạn hoàn toàn có thể hiển thị máu (blood) và con số thể hiện sức tấn công (damage), nhưng bạn không thể cập nhật máu của (health) nhân vật cho tới khi nhận thông tin từ server.


Vì sự phức tạp của trạng thái game, bạn sẽ muốn tránh việc nhận vật bị giết trước khi server thông báo, dù rằng trạng thái máu của nhân vật đã ở dưới 0 tại client (điều gì sẽ xảy ra nếu nhân vật kia sử dụng hồi máu ngay trước khi bị bạn đánh chết nhưng server lại chưa thông báo với bạn?)


Vấn đề này thực sự rất thú vị - dù mọi thứ có thể dự đoán được và không hề có người chơi nào gian lận, thì vẫn có khả năng xảy ra việc trạng thái được dự đoán từ client và trạng thái được gửi từ server không trùng khớp nhau sau khi server điều phối. Trường hợp này không thể xảy ra khi chỉ có một người chơi duy nhất, tuy nhiên nó rất dễ xảy ra khi có nhiều người chơi cùng kết nối đến một server. Chúng ta sẽ nói đến chủ đề này tại bài viết tiếp theo.


#### Tóm tắt


Khi sử dụng authoritative server, bạn cần mang lại cho người chơi những thay đổi, đáp ứng giả, trong khi bạn chờ đợi server thực sự thực thi hành động từ người chơi. Để làm vậy, client cần mô phỏng kết quả của các hành động. Khi trạng thái cập nhật được nhận từ server, trạng thái đã dự đoán tại client sẽ được tính toán lại dựa vào trạng thái cập nhật, và những hành động mà server chưa ghi nhận.


Theo [Gabriel Gambetta](http://www.gabrielgambetta.com/fpm2.html)