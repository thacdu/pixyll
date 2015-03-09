---
layout:     post
title:      Multiplayer Game Architecture (P3)
date:       2015-01-27 00:44:00
---

### Part III: Entity Interpolation


#### Giới thiệu


Trong bài viết đầu tiên, chúng ta đã giới thiệu về khái niệm _authoritative server_ và sự hữu dụng của nó trong việc chống hack/cheat. Tuy nhiên, sử dụng kỹ thuật này một cách đơn thuần có thể dẫn đến vấn đề giật lag khi chơi. Ở bài viết thứ hai, chúng ta đã đề xuất giải pháp _client-side prediction_ như một cách để giải quyết vấn đề trên.


Tổng hợp chung quy lại từ hai bài viết đầu tiên, chúng ta đã có những khái niệm và kỹ thuật cho phép người chơi điều khiển nhân vật trong game giống như là đang chơi _single-player game_, kể cả khi kết nối tới authoritative server gặp delay.


Trong bài viết này, chúng ta sẽ đi tìm hiểu những hệ quả khi có những nhân vật được điều khiển bởi những người chơi khác cùng kết nối đến một máy chủ.


#### Server time step


Trong bài viết trước, hành vi của server được miêu tả một cách khá là đơn giản - đọc dữ liệu đầu vào từ client, cập nhật trạng thái game, gửi lại cho client. Khi có nhiều client kết nối đến server, server loop (hành vi của server khi nhận dữ liệu từ client) sẽ thay đổi khác đi một chút.


Trong trường hợp này, có thể có nhiều client cùng gửi dữ liệu lên server. Việc cập nhật game world mỗi khi nhận dữ liệu đầu vào từ mỗi client rồi sau đó gửi trạng thái game cho toàn bộ các client còn lại làm tiêu tốn rất nhiều CPU và băng thông.


Một cách tiếp cận sáng sủa hơn là đưa vào hàng đợi những hành động nhận được từ client, và không thực hiện chúng ngay. Thay vào đó, game world sẽ được cập nhật một cách định kỳ với tần suất thấp, ví dụ 10 lần trên giây. Độ trễ giữa mỗi lần cập nhật, trong trường hợp này là 100 ms, được gọi là _time step_. Trong mỗi lần cập nhật, tất cả những hành động trong hàng đợi sẽ được thực thi (thường trong khoảng thời gian nhỏ hơn _time step_, làm tăng khả năng dự đoán vật lý), và trạng thái game mới sẽ được gửi cho toàn bộ client.


Tóm lại, việc cập nhật trạng thái game là độc lập với sự có mặt và số lượng hành động của client.


#### Dealing with low-frequency updates


Dưới góc nhìn của client, cách tiếp cận này hoạt động trơn tru như trước kia - client-side prediction hoạt động độc lập với với update delay, do đó nó rõ ràng cũng hoạt động tốt cùng với prediction. Tuy nhiên, vì trạng thái game được gửi đi với tần suất thấp (tiếp tục với ví dụ, mỗi 100 ms), nên client có rất ít thông tin về các thực thể khác có thể di chuyển trong game world.


Với cách tiếp cận đầu tiên, vị trí của các nhân vật khác sẽ được cập nhật khi client nhận được trạng thái cập nhật từ server. Điều này ngay lập tức dẫn đến một vấn đề đó là nhân vật di chuyển nhảy cóc, tức là, nhân vật sẽ nhảy một bước mỗi 100 ms thay vì di chuyển mượt mà.


![]({{ site.url }}/images/fpm3-01.png)


Tùy thuộc vào thể loại game bạn phát triển mà có những cách khác nhau để xử lý với vấn đề này. Nói chung, bạn càng dự đoán được đối tượng trong game của mình, bạn càng dễ dàng giải quyết được nó.


#### Dead reckoning (Đoán định vị trí)


Giả sử bạn đang làm một game đua xe. Chúng ta hoàn toàn có thể dự đoán được khi chiếc xe đang ở tốc độ cao - ví dụ, nếu nó đang chạy ở vận tốc 100 m/s, một giây sau đó vị trí của chiếc xe là khoảng 100 mét về phía trước của nơi mà nó vừa đứng.


Tại sao lại là "khoảng"? Bởi vì trong một giây đó chiếc xe có thể tăng tốc hoặc giảm tốc một chút, hoặc có thể rẽ trái hoặc rẽ phải một chút - từ khóa ở đây chính là "một chút". Khi ở tốc độ cao, vị trí của chiếc xe tại bất kì thời điểm nào đều phụ thuộc rất nhiều và vị trí trước đó, tốc độ và hướng di chuyển, bất kể hành động thực sự của người chơi. Nói cách khác, một chiếc xe đua không thể quay ngoắt 180 độ ngay lập tức được.


Vậy game sẽ hoạt động ra sao nếu server gửi cập nhật mỗi 100 ms? Client nhận được thông tin về tốc độ và hướng di chuyển của mỗi chiếc xe trong cuộc đua; trong 100 ms tiếp theo đó nó sẽ không nhận thêm thông tin gì. Tuy vậy, client vẫn phải thể hiện được cuộc đua đang diễn ra. Cách làm đơn giản nhất là giả định hướng và gia tốc của mỗi xe là không thay đổi trong suốt 100 ms, và việc di chuyển sẽ được tính toán ngay tại client với các thông số hiện tại. Sau đó, 100 ms sau, khi nhận được trạng thái cập nhật từ server, vị trí của những chiếc xe sẽ được cập nhật lại cho đúng.


Sự chính xác có thể nhiều hay ít tùy thuộc vào nhiều yếu tố. Nếu người chơi giữ cho xe di chuyển thẳng về phía trước với vận tốc không đổi, vị trí dự đoán sẽ chính là vị trí thực của chiếc xe. Mặt khác, nếu người chơi va phải vật gì đó trên đường, vị trí được dự đoán sẽ rất là sai lầm.


Chú ý rằng dead reckoning có thể được áp dụng cho các trường hợp tốc độ thấp - ví dụ như battleships. Trên thực tế, thuật ngữ "dead reckoning" có nguồn gốc trong ngành hàng hải.


#### Entity interpolation (Nội suy thực thể)


Có những trường hợp mà dead reckoning hoàn toàn không thể áp dụng được - đặc biệt trong những hoàn cảnh mà hướng và tốc độ của người chơi có thể thay đổi ngay tức khắc. Ví dụ, trong game bắn súng 3D, người chơi luôn luôn chạy, dừng lại, quay góc ở tốc độ rất cao. Điều này khiến cho dead reckoning về cơ bản là vô dụng, vị trí và tốc độ không thể dự đoán được dựa vào dữ liệu trước đó nữa.


Bạn không thể chỉ cập nhật vị trí của người chơi khi server gửi dữ liệu độc quyền. Bạn chắc chắn sẽ không muốn người chơi dịch chuyển tức thời trong khoảng cách ngắn mỗi 100 ms, điều này làm cho game không thể chơi được.


Bạn chỉ có dữ liệu về vị trí của người chơi mỗi 100 ms; làm thể nào để cho người chơi thấy được những gì diễn ra trong khoảng thời gian đó. Giải pháp ở đây là cho những người chơi khác thấy lại những gì mà người chơi đã thực hiện.


Giả sử bạn nhận dữ liệu tại thời điểm **t = 1000**. Bạn đã nhận được dữ liệu tại thời điểm **t = 900**, do vậy bạn biết được vị trí của người chơi tại thời điểm **t = 900** và **t = 1000**. Vậy từ **t = 1000** đến **t = 1100**, bạn hiển thị những gì mà những người chơi khác đã làm từ thời điểm ***t = 900* đến **t = 1000**. Với cách này, bạn sẽ luôn luôn hiển thị được chính xác việc di chuyển của người chơi, tuy nhiên bạn hiển thị nó trễ đi 100 ms.


![]({{ site.url }}/images/fpm3-02.png)


Thông tin vị trí bạn sử dụng để nội suy từ **t = 900** đến **t = 1000** tùy thuộc vào game của bạn. Thông thường nội suy hoạt động rất tốt. Nếu không, server gửi thêm thông tin chi tiết về việc di chuyển trong mỗi cập nhật - ví dụ, một chuỗi các đoạn thẳng mô tả đường di chuyển của nhân vật, hoặc lấy mẫu vị trí mỗi 10 ms làm cho việc nội suy tốt hơn.


Chú ý rằng việc sử dụng phương pháp này sẽ làm cho mỗi người chơi thấy sự thể hiện của game world có đôi chút khác nhau, vì mỗi người chơi thấy được bản thân mình tại thời điểm hiện tại nhưng lại thấy những người chơi khác ở thời điểm trong quá khứ. Mặc dù game là tốc độ cao, tuy nhiên việc trông thấy những thực thể khác sau 100 ms nói chung là không thể nhận ra được sự khác biệt.


Cũng có những trường hợp ngoại lệ - khi bạn cần rất nhiều sự chính xác về mặt không gian và thời gian, chẳng hạn như khi một người chơi bắn người chơi khác. Vì những người chơi khác được thấy là thời điểm trong quá khứ nên bạn đang ngắm với độ trễ 100 ms - nghĩa là, bạn đang bắn mục tiêu của bạn ở 100 ms trước. Chúng ta sẽ xử lý vấn đề này trong bài viết tiếp theo.


#### Tóm tắt


Trong mô hình client-server với authoritative server, infrequent updates và network delay, bạn vẫn sẽ phải tạo ra những giả tưởng cho người chơi về tính liên tục và chuyển động trơn tru của game. Trong phần 2 của loạt bài này, chúng ta đã tìm hiểu về giải pháp để thể hiện sự chuyển động của người chơi trong thời gian thực bằng cách sử dụng client-side prediction và server reconciliation; điều này đảm bảo cho hành động của người chơi được thực hiện ngay tức khắc trên thiết bị của họ, loại bỏ độ trễ làm game chúng ta không thể chơi được.


Tuy nhiên, vẫn còn những vấn đề khác. Trong bài này chúng ta đã tìm hiểu thêm 2 phương pháp để xử lý với chúng.


Thứ nhất, _dead reckoning_, áp dụng đối với những loại đối tượng mà vị trí có thể được ước tính bằng những dữ liệu trước đó, ví dụ như vị trí, tốc độ và gia tốc. Cách tiếp cận này không thành công khi những điều kiện này không được đáp ứng.


Thứ hai, _entity interpolation_, không hề dự đoán vị trí tương lai, kỹ thuật này chỉ sử dụng dữ liệu thực được cung cấp bởi server, do vậy khi thấy các đối tượng khác sẽ có đôi chút chậm trễ.


Tác dụng là nhân vật của người chơi được nhìn thấy là tại thời điểm hiện tại còn những đối tượng khác được nhìn thấy là tại thời điểm trong quá khứ. Điều này thường tạo ra một trải nghiệm vô cùng liền mạch.


Tuy nhiên, nếu không thực hiện thêm gì khác, những giả lập sẽ bị phá vỡ khi một sự kiện cần độ chính xác cao về mặt không gian và thời gian, chẳng hạn như bắn vào một mục tiêu di động: vị trí mà Client 2 render Client 1 không khớp với của server hay vị trí của Client 1, do vậy không thể headshot! Vì game không thể hoàn thiện nếu thiếu headshot, nên chúng ta sẽ xử lý với vấn đề này trong bài viết tiếp theo.


_Theo [Gabriel Gambetta](http://www.gabrielgambetta.com/fast_paced_multiplayer.html)_