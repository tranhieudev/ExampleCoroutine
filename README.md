# ExampleCoroutine

Thực hiện các cuộc gọi mạng không đồng bộ với Coroutine Kotlin trong Android.
Coroutine được giới thiệu với kotlin v1.1 trong năm 2017 và từ đó chúng ta có sự trải nghiệm bất đồng bộ một cách tốt nhất
Coroutines mang đến một loạt các tính năng đầy sức mạnh cho trò chơi và hầu hết chúng ta đã chứng kiến điều đó cho đến bây giờ.
Trong khi phát triển ứng dụng Android, bạn sẽ bắt gặp nhiều tình huống trong đó các coroutines có thể được thực hiện. Phổ biến nhất là trong khi thực hiện nhiều cuộc gọi mạng không đồng bộ.
Trong bài báo này, chúng tôi không nói những điều cơ bản về Coroutines nhưng  thảo luận về một số tình hướng mà chúng ta hay thường gặp phải trong khi xứ lý các cuộc gọi mạng không đồng bộ và làm thể nào chúng ta có thể thực hiện nó một cách dễ dàng với sự trợ giúp của Coroutines
Điều kiện tiên quyết:
Một số kiến thức cơ bản và nguyên tắc cơ bản của Kotlin và Coroutines

Mục tiêu:
Đây là bài báo nói về xử lý với một số tình huông cơ bản gặp phải trong thực hiện gọi đa phương mạng bất đồng bộ. Một số khá đơn giản , và t một số yêu cầu xử lý đặc biệt.

Những kich bản như sau:

### 1. Fire-and-forget network calls.
### 2. Hủy các cuộc gọi mạng khác nếu ít nhất một cuộc gọi thất bại
### 3. Tiếp tục thực hiện các cuộc gọi mạng khác ngày cả khi có 1 cuộc gọi thất bại
### 4 Hủy các cuộc gọi khác chỉ khi gặp một số điều kiện lỗi nhất định

Hãy cùng chúng tôi đi sau vào kịch bản.
## 1. Fire-and-forget(FAF) network calls.  
Những cuộc gọi này được thực hiện dưới background (chạy ngầm) trong khi bỏ qua các phản hồi. Hãy xem xét một kịch bản trong đó thực hiện 10 cuộc gọi mạng và không chặn giao diện người dung ứng dụng. Điều này có thể làm được với phần mở rộng  của CorountinScope .  Nếu nó trả về một đối tượng công việc có thể cung cấp cho chúng tôi thông tin về công việc cụ thể nào đó. Một cái gì đó như thế này 
![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv1.png)

Để cho đơn giản, chúng ta thực hiện 10 cuộ gọi mạng trong vòng lặp joinALL được gọi khi tất cả các công việc coroutine đã hoàn thành.

![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv2.png)

Chú ý rằng Id cuộc gọi mạng không xuất tăng dần,Kết quả được hiện thị như trên là do corountine là luồng dữ liệu rất nhỏ . Ở đây mỗi cuộc gọi mạng là đồng thời và độc lập lẫn nhau

## Hủy các cuộc gọi mạng khác nếu có ít nhất một cuộc gọi thất bại.
Xem xét một kịch bản trog đó tất cả các cuộc gọi mạng khác sẽ bị hủy và bỏ qua ngay nếu có một cuộc gọi thất bại. Điều này có thể do bạn đang điều hướng khỏi trang và tất các các cuộc gọi mạng còn lại không còn quan trọng nữa.
Trong ví dụ sau đây. Chúng tôi đã sử dụng async mở rộng để có có thể kết thúc các đối tượng. Tuy nhiên chức năng tương tự cũng có thể đạt được bằng cách khởi chạy.

![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv3.png)
![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv4.png)

Chú ý rằng chúng ta sử dụng “ CoroutineExceptionHandel”. Nếu nó đã được sử dụng như một khối bắt chung cho ErroHandingScope. Nếu một công việc nào đó bị ném lỗi ngoại lệ. Nó sẽ bị bắt bởi CorountineExeptionHander, một tín hiệu hủy sẽ được gửi đến tất các các công việc con khác và hành đồng cần thiết có thể thực hiện được. Để mô tả một kịch bản lỗi , chúng tôi gọi hàm throwCustomExeption trong lần lập thứ 4 

![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv5.png)

Chú ý rằng thuộc tính “isCancelled” cho tất cả các cuộc gọi là đúng (Tất cả các cuộc gọi đều bị hủy). Tuy nhiên, đây có thể bị sai vì một vài job có tồn tại kiến trúc “completed” state khi đẵ được nén loại ngoại lệ.  
## Tiếp tục cuộc gọi những API khác khi một sự kiện gọi API thất bại
Xem xét kịch bản, trong đó một cuộc gọi mạng thất bại có thể bị bỏ qua và giao diện người dùng được cập nhập tương ứng. Trong những công việc khác, scope bố mẹ không nên bị ảnh hưởng bởi các công việc con và những công việc con không bị ảnh hưởng lẫn nhau.
Phạm vi giám sát có thể được sử dụng để đạt được kịch bản này theo các cách sau.  
![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv6.png)
![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv7.png)

Ở đây , “SupervisorScope” được overrides từ Job() to SupervisorJob().điều này là cần thiết  vì công việc bố không nên bị hủy khi mà công việc con bị hủy.
Thay vào đó, chúng tao nên tạo một scope với “SupervisorJob” thay cho Job()
Do đó loại bỏ sự cần thiết của supervisorScope. Mặc dù chúng ta mong muốn cần tạo ra cái gì đó chắc chắn kiểu như là sử dụng Coroutine ExeptionHandler trongtrường hợp “launch” giống như chú ý trong document
Chúng ta gọi joinALL()  thay cho awaitAll() vì để  ném một ngoại lệ ngay khi công việc con nén ngoại lệ. Tuy nhiên, allJob() chỉ được gọi khi tất cả công việc đã được thực thi thành công viec. isCompleted is true
Chú ý rằng “GetCompletionExemptionOrNull” is một hàm thực nghiệm với phiên bản hiện hành của coroutines.

![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv8.png)

Chú ý rằng chỉ một công việc fail không ảnh hưởng tới các công việc khác.

## Hủy các cuộc gọi mạng khác chỉ khi gặp điều kiện lỗi nhất định
Xem xép về trường hợp tùy thuộc vào phản hồi của API, chung ta sẽ quyết định việc thực hiện các cuộc gọi mạng khác sẽ tiêc tục hay hủy bỏ. Nói cách khác, dựa trên một điều kiện cụ thể, chúng ta quyết định liệu chúng ta muốn tiếp tục hay hủy bỏ các cuộc gọi con khác

![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv9.png)
![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv10.png)
Ở đây chúng ta có 2 toán tử lỗi, 1 là CustomExeption, cái mà có thể bị bỏ qua và những công việc khác có thể thực thi trong khi 2 là hủy toán tử đó có thể hủy parent scope.

Để đơn giản. lần lặp thứ 4 ném CustomExeption và lần lập thứ 7 sẻ hủy phạm vi của cha. Chúng ta sử dụng awaitAll() vì chúng ta muốn xử lý ngoại lệ khi nó được ném ra
![](https://github.com/tranhieudev/ExampleCoroutine/blob/master/imv11.png)

Chú ý rằng khi lần lặp thứ 4 bị gọi thất bại, tất cả công việc đang chờ xử lý được hỗ trọ để tiếp tục thực hiện và khi  lần lặp thứ 7 được gọi, phạm vi parent được hủy. do đó hủy bỏ các con của nó.
Tuy nhiên, như đã thấy trong ouput, không phải tất cả các công việc đều bị hủy và đó là do các cuộc gọi mạng này không được thực hiện theo cùng thứ tự mà chúng đã được gọi.Tại thời điểm hủy phạm vi của bố mẹ, chỉ có những công việc đã bị hủy mà vẫn chưa thực hiện/
Trong khi coroutine cung cấp giải pháp mạnh để kích hoạt chúng để chúng ta viết mã đồng thời, cần chú ý đặc biết đến xử lý ngoại lệ đế tránh crash app

Tác giả : Kunal Chaubal : Dịch : Trần Hiếu
16/5/2020- 21/5/2020
Bạn có thể tìm kiến kho lưu tữ đầy đủ trên github
[](https://github.com/KunalChaubal/CoroutinesSample)

document : [](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html)

