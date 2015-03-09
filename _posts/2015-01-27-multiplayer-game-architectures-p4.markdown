---
layout:     post
title:      Multiplayer Game Architecture (P4)
date:       2015-01-27 15:10:00
---

### Part IV: Headshot! (AKA Lag Compensation)


#### Recap


Tóm tắt lại 3 bài viết trước:


* Server nhận dữ liệu đầu vào từ tất cả client tại những mốc thời gian.


* Server thực thi các yêu cầu và cập nhật trạng thái game world


* Server gửi trạng thái game cho toàn bộ client


* Client gửi hành động và mô phỏng hành động


* Client nhận trạng thái cập nhật của game


	* Đồng bộ trạng thái dự đoán với trạng thái thực


	* Nội suy trạng thái cũ của những người chơi khác


Dưới góc nhìn của người chơi, có hai hệ quả tất yếu như sau:


* Người chơi thấy nhân vật của họ tại thời điểm hiện tại


* Người chơi thấy nhân vật của những người chơi khác tại thời điểm trước đó


Nói chung cách tiếp cận này hoạt động ổn định, nhưng nó lại gặp phải những vấn đề đối với những sự kiện đòi hỏi độ chính xác cao về mặt không gian và thời gian, ví dụ như bắn vào đầu đối thủ!


#### Lag Compensation


Giả sử bạn ngắm rất chuẩn vào đầu của mục tiêu bằng khẩu sniper rifle của bạn. Bạn bắn - chắc chắn là không thể trượt được rồi.


Nhưng... TRƯỢT.


Tại sao lại như vậy?


Vì với kiến trúc client-server được miêu tả như trên, bạn đã ngắm vào đầu của mục tiêu ở thời điểm 100 ms trước khi bắn, chứ không phải tại thời điểm bắn.


Trường hợp này giống như là khi bạn chơi game trong một thế giới mà tốc độ ánh sáng ở đó là rất rất chậm. Bạn bắn vào vị trí của đối phương, nhưng hắn đã đi mất trong khoảng thời gian đạn bay.


May mắn thay, có một giải pháp tương đối đơn giản để giải quyết vấn đề này, nó làm cho người chơi thấy hài lòng trong phần lớn thời gian (với một trường hợp ngoại lệ sẽ được thảo luận dưới đây)


Giải pháp là như sau:


* Khi bạn bắn, client gửi sự kiện đó tới server với đầy đủ thông tin: mốc thời gian chính xác khi bắn, vị trí ngắm chính xác


* **Đây chính là bước quyết định**. Vì server nhận toàn bộ dữ liệu trong từng mốc thời gian, nên server có thể xây dựng lại game world tại bất kỳ thời điểm nào trước đó. Đặc biệt, server có thể xây dựng lại game world giống y hệt như game world của bất kỳ client nào, tại bất kỳ thời điểm nào.


* Điều đó có nghĩa là server có thể biết chính xác vị trí ngắm bắn của bạn là vào đâu. Đó là vị trí trước đó của đầu đối phương, nhưng server biết được đó lại là vị trí hiện tại dưới góc nhìn của bạn.


* Server thực hiện nhát bắn vào đầu đối phương, và cập nhật trạng thái tới toàn bộ client.


Và tất cả mọi người đều cảm thấy hài lòng!


Server cảm thấy hài lòng, vì server là server. Server lúc nào cũng thấy hài lòng.


Bạn cảm thấy hài lòng vì bạn ngắm vào đầu của đối thủ, bắn, và bạn được một pha headshot!


Đối phương là người duy nhất cảm thấy không vui một chút nào. Nếu anh ấy đứng yên tại vị trí mà anh ấy bị bắn, thì đó là lỗi của anh ấy thôi. Nếu anh ấy di chuyển... wow, bạn thực sự là một sniper tuyệt vời.


Nhưng nếu anh ấy đang ở vị trí trống, rồi núp vào sau bức tường, và bị bắn, trong khoảng dưới 1s sau đó, khi anh ấy nghĩ rằng anh ấy đã an toàn?


Đúng, trường hợp này có thể xảy ra. Đó là sự đánh đổi.


Điều đó có hơi chút bất công, tuy nhiên giải pháp này được nhiều người tán thành nhất. Còn hơn là việc bắn trượt những phát bắn không thể nào trượt được.


#### Kết luận


Đây là phần kết cho loạt bài Game nhiều người chơi tốc độ cao (Fast-paced Multiplayer). Vấn đề này thực sự là khó để làm cho đúng, nhưng với sự hiểu biết rõ ràng về những gì đang diễn ra thì nó cũng không phải quá là khó.


Mặc dù độc giả của loạt bài này là những nhà làm game, thì những game thủ cũng nên quan tâm đến những vấn đề này. Dưới góc nhìn của một game thủ, thực sự cũng khá thú vị để hiểu lý do tại sao một số sự việc lại xảy ra theo cách chúng xảy ra.


##### Đọc thêm


Một số nguồn mà tác giả khuyến khích đọc thêm


[What Every Programmer Needs to Know About Game Networking](http://gafferongames.com/networking-for-game-programmers/what-every-programmer-needs-to-know-about-game-networking/)


[Latency Compensating Methods in Client/Server In-game Protocol Design and Optimization](https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization)


### Sample Code and Live Demo


Phần này tôi sẽ không dịch mà dẫn link đến website của tác giả.


[Live Demo](http://www.gabrielgambetta.com/fpm_live.html)


_Theo [Gabriel Gambetta](http://www.gabrielgambetta.com/fast_paced_multiplayer.html)_