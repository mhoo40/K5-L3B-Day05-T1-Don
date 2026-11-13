# Bài nộp LAB5 — [Track] · Nhóm Don
**Lớp:** H202

**Bàn / zone:** **I** 

**Track:** 1

| Họ và Tên | Mã Học Viên | Việc đã làm |
|---|---|---|
| Nguyễn Long Vũ | 2A202603842 | **(1)** đọc đề, chọn lát cắt, xác định tool không đảo lại được **(2)** viết 8 kịch bản rủi ro phủ 4 lớp, khoanh 2 ca đắt nhất (#1, #4) **(3)** thiết kế câu AI nói, thông tin người dùng thấy, người được báo và loại lỗi cho 2 ca **(4)** xếp mức tự chủ cho từng tool, thêm tool `doc_anh_bao_bi`, chốt `goi_y_san_pham` **(5)** dựng prototype HTML 2 màn hình, ghi quyết định thiết kế |


## §1. Lát cắt & phạm vi

- **Lát cắt**: Trong kênh chat Hỏi dược sĩ của Pharmacity, khi khách gửi ảnh hộp thuốc kèm câu hỏi “cái này uống thế nào”, AI quyết định xác nhận tên thuốc và chỉ về nhãn, hỏi lại vì ảnh chưa đủ, chuyển dược sĩ, hay từ chối và khuyên đi khám, để khách không dùng nhầm thuốc hoặc nhầm liều.

### 1.1. Ai đang làm việc gì?
- Khách trên Zalo OA của Pharmacity, người nhắn thay hay người sẽ uống thuốc.
- Thuốc đã nằm trong tay khách và họ cần một câu trả lời ngay. Ảnh có thể mờ, cắt mất tên, hoặc chứa thông tin của người khác.

### 1.2. AI phải ra quyết định gì?
- Xác nhận và chỉ về nhãn: ảnh rõ, thuốc không kê đơn, người dùng thuốc thuộc nhóm bình thường. AI đọc tên thuốc, nói lại cho khách kiểm và chỉ vào hướng dẫn trên nhãn.
- Hỏi lại một câu: ảnh mờ, thiếu tên hoặc hàm lượng, hoặc chưa biết ai sẽ uống.
- Chuyển dược sĩ (chuyen_duoc_si): người dùng thuốc là trẻ nhỏ, người già hoặc bà bầu, hai thuốc cùng lúc, hoặc đơn thuốc có nhiều thuốc.
- Khuyên đi khám ngay (khuyen_di_kham): ảnh thuốc đi kèm dấu hiệu nguy hiểm.
- Từ chối: đòi thuốc thay thế cho thuốc kê đơn hoặc đòi liều.

### 1.3. Phân loại Tool:
- Tool thay đổi trạng thái: goi_y_san_pham, luu_ghi_chu_suc_khoe, chuyen_duoc_si.
- Hai tool không đảo lại được là goi_y_san_pham (khi thuốc đã được khách mua và dùng vào cơ thể) và ban_thuoc_ke_don (vi phạm pháp luật và quy chế dược phẩm). Đây là những điểm mà thiết kế hệ thống tuyệt đối không được phép giao quyền tự chủ hoàn toàn cho AI.

### 1.4. Nhu cầu khách đòi mà không có tool:
- Đòi liều theo cân nặng, hỏi thuốc này thay thuốc kia được không, hỏi hai thuốc uống chung được không, hỏi hạn dùng khi chữ mờ, nhờ đọc đơn rồi “tư vấn thay bác sĩ”.
- AI nói gì: nói rõ phần mình đọc được từ ảnh và phần mình không xác nhận được. Với việc ngoài thẩm quyền thì từ chối ngắn gọn, không xin lỗi rườm rà, mời gặp dược sĩ (chuyen_duoc_si) và nói dược sĩ sẽ thấy ảnh khách vừa gửi.

## §2. Kịch bản rủi ro  (chặng 2)

| # | tin người dùng | lớp | AI có thể sai thế nào | sản phẩm tốt nên làm gì | nhãn |
|---|---|---|---|---|---|
| 1 | [ảnh vỉ thuốc mờ, chỉ thấy "...cillin 500"] "Mẹ mình 70 tuổi, thuốc này uống sao em?" | bịa | Ảnh không đủ tên, AI đoán là "Amoxicillin" rồi chỉ cách dùng. Không có tool tra cứu để kiểm, nên tên thuốc là suy đoán và các câu sau đều dựa vào nó | Nói rõ chỉ đọc được "...cillin 500", không xác nhận tên. Hỏi khách chụp lại mặt có tên thuốc. Người dùng là người 70 tuổi nên chuyển dược sĩ (`chuyen_duoc_si`), không chờ ảnh mới mới chuyển | không chấp nhận được |
| 2 | [ảnh hộp Paracetamol rõ] "Con mình 3 tuổi, ngày uống mấy viên thì hết sốt nhanh?" | ngoài thẩm quyền | AI đọc đúng tên thuốc nên tạo cảm giác đáng tin, rồi trả lời liều từ trí nhớ hoặc tính theo tuổi. Vi phạm "không tự nói liều"; trẻ nhỏ sai liều là hại thật | Xác nhận tên thuốc đọc được, không nói liều, chỉ về mục hướng dẫn trên nhãn. Trẻ 3 tuổi nên chuyển dược sĩ kèm ảnh và tuổi | không chấp nhận được |
| 3 | [ảnh đơn thuốc bác sĩ: kháng sinh + thuốc huyết áp] "Có loại nào rẻ hơn thay được không?" | ngoài thẩm quyền | Gọi `goi_y_san_pham` với thuốc thay thế, hoặc nói "cùng hoạt chất dùng được". Đây là thuốc kê đơn, AI không có quyền thay | Từ chối gợi ý thay thế, giải thích ngắn, chuyển dược sĩ. Không gửi nút đặt | không chấp nhận được |
| 4 | [ảnh hộp thuốc giảm đau] "Ba mình uống cái này từ chiều mà vẫn đau ngực trái, hơi khó thở." | mất sức khoẻ ngay | AI bị ảnh kéo vào "việc đọc thuốc", trả lời về thuốc trước rồi mới nhắc đi khám, hoặc đẩy khách vào hàng chờ dược sĩ | Bỏ qua câu hỏi thuốc. Hiện `khuyen_di_kham` ngay: gọi cấp cứu, không gợi ý thuốc, không bắt chờ. Chuyển dược sĩ chạy song song, không thay thế | không chấp nhận được |
| 5 | [ảnh 2 hộp thuốc khác nhau] "Hai thuốc này uống chung được không?" | bịa | Không có tool tra tương tác. AI trả lời "được" hoặc "không" từ trí nhớ nghe rất chắc | Đọc tên hai thuốc cho khách kiểm, nói rõ mình không kết luận được tương tác, chuyển dược sĩ kèm cả hai tên thuốc | sửa được |
| 6 | [ảnh vỉ thuốc, hạn dùng in mờ] "Thuốc này còn dùng được không?" | bịa | AI "đọc" ra một hạn dùng không chắc rồi kết luận còn/hết hạn | Nói phần đọc được, nói rõ phần mờ không chắc, khuyên không dùng nếu không chắc, mời dược sĩ xem | sửa được |
| 7 | [ảnh đơn thuốc có tên, tuổi, chẩn đoán của người khác] "Lưu lại giúp mình cho lần sau nhé." | mất niềm tin | Gọi `luu_ghi_chu_suc_khoe` ngay, lưu cả thông tin người nhà (bên thứ ba) khi chưa hỏi đồng ý. Xoá được nhưng đã có rủi ro lộ | Hỏi xin đồng ý trước và nói rõ sẽ lưu gì, của ai. Mặc định không lưu thông tin sức khoẻ khi khách chưa xác nhận | sửa được |
| 8 | [ảnh vỉ thuốc] "Cái này uống sao? Với Pharmacity gần mình còn nước muối sinh lý không, mình đang bực lắm rồi." | mơ hồ | Hai yêu cầu trong một tin. AI trả lời xong việc dễ rồi bỏ quên câu hỏi thuốc, hoặc gắt lại vì khách giận | Trả lời từng việc, việc thuốc vẫn qua đủ bước sàng lọc. Giọng bình tĩnh, hỏi một câu duy nhất: ai sẽ uống? | nhẹ |

**Phủ 4 lớp:** 
1) #1, #5, #6
2) #8
3) #2, #3
4) #4, #7

- Bằng chứng: cột "AI có thể sai thế nào" trỏ về một tool hoặc một dòng dữ liệu của track.
- Quyết định: 2 kịch bản đắt nhất - vì sao đắt hơn các kịch bản còn lại; kịch bản nào suýt được chọn và vì sao không.
    + **#1**: Mọi câu trả lời sau đó đều dựa trên tên thuốc AI tự đoán, và khách nghe giọng chắc chắn nên không kiểm lại. Người uống là người già, thuốc đã uống thì không đảo được. Các kịch bản khác sai ở một câu, kịch bản này sai ở tiền đề.
    + **#4**: Sai thì mất cả thời gian cứu người, và là chỗ AI dễ chọn nhầm việc vì ảnh thuốc kéo sự chú ý.
    + **Suýt được chọn**: #2. Hại cũng không chấp nhận được, nhưng đã có luật cứng “không nói liều” và chuyển dược sĩ cho trẻ nhỏ. Lỗi này chặn bằng quy tắc, ít cần thiết kế UX hơn #1 và #4.

## §3. Khi AI sai  (chặng 3)
### 3.1 Kịch bản #1: ảnh vỉ thuốc mờ, chỉ đọc được "...cillin 500", người uống là mẹ 70 tuổi

**Đường đi:** low-confidence (2) chồng failure/không căn cứ (1). AI không có tool tra tên thuốc, nên phần không đọc được là dữ liệu *không có*.

**Lúc AI không chắc, nó làm gì?** Trả lời kèm giới hạn **và** chuyển dược sĩ ngay, hỏi khách chụp lại chạy song song.

| Phương án | Quyết định | Vì sao |
|---|---|---|
| Chỉ hỏi lại | Loại | Bệnh nhân 70 tuổi thuộc nhóm bắt buộc chuyển dược sĩ dù ảnh rõ. Hỏi lại một mình chỉ kéo dài thời gian chờ |
| Trả lời kèm giới hạn, không chuyển | Loại | Dễ trượt thành "có thể là Amoxicillin, dùng như sau...". Tên thuốc đoán sẽ kéo theo mọi câu sau |
| Dừng, im lặng chuyển người | Loại | Khách không biết chuyện gì xảy ra, bỏ đi hoặc tự uống |
| Kèm giới hạn + chuyển dược sĩ + xin ảnh mới | Chọn| Khách biết AI đọc được gì, không đọc được gì, và đã có người nhận |

**AI nói (nguyên văn):**
> Mình là trợ lý AI của Pharmacity. Trong ảnh, mình chỉ đọc được "...cillin 500", chưa đủ để biết chính xác đây là thuốc gì, nên mình chưa hướng dẫn cách dùng. Vì người uống là mẹ bạn (70 tuổi), mình đã chuyển cuộc chat và ảnh này cho dược sĩ. Bạn chụp lại mặt hộp hoặc vỉ có tên thuốc giúp mình nhé. Nếu mẹ chưa uống, bạn chờ dược sĩ xác nhận trước. Nếu mẹ đã uống và có dấu hiệu bất thường như nổi mẩn, sưng môi, khó thở, hãy đi khám ngay.

Câu này không hứa điều AI không làm được: không nói tên thuốc, không nói liều, không hứa thời gian dược sĩ trả lời.

**Người dùng thấy gì ngoài chữ:**
- Nhãn "Trợ lý AI" ở đầu mỗi tin.
- Thẻ ảnh có hai vùng: "Đọc được: ...cillin 500" và "Không đọc được: tên thuốc, hạn dùng". Đây là căn cứ để khách tự kiểm.
- Nhãn độ chắc: "Chưa xác nhận tên thuốc".
- Trạng thái: "Đã chuyển dược sĩ", kèm nút "Chụp lại" và "Nhập tên thuốc".
- Không có nút đặt hàng và không có dòng cách dùng.

**Người dùng làm gì tiếp:**
- Chụp lại, hoặc gõ tên thuốc nếu đọc được từ hộp.
- Sửa AI: bấm "Không đúng" ở vùng đọc được, hoặc nhắn "thuốc này là X". AI ghi nhận là *"khách cho biết"*, không dùng làm căn cứ gợi ý, chỉ đính kèm cho dược sĩ.
- Gọi hotline nếu không muốn chờ.
- Nếu không làm gì ca vẫn nằm ở dược sĩ kèm ảnh. Không có hành động không đảo được nào xảy ra.

**Ai được báo, sau bao lâu:** dược sĩ trực, **ngay lúc `chuyen_duoc_si` chạy thành công**. Họ thấy: ảnh gốc, phần AI đọc được, phần không đọc được, người dùng thuốc "mẹ, 70 tuổi", thông tin còn thiếu (thuốc đang dùng, bệnh nền, dị ứng, đã uống chưa), và ghi chú "AI chưa nói gì về cách dùng". Nếu không dược sĩ nào nhận trong khoảng thời gian nhóm tự đặt (ví dụ 10 phút), khách thấy nút gọi hotline.

**Loại lỗi:** lỗi **ngữ cảnh** - đọc ảnh chạy đúng thiết kế nhưng ảnh không đủ rõ. Nếu AI đoán tên thuốc thì thành lỗi **hệ thống**. Cách sửa nằm ở thiết kế: AI không được đoán tên thuốc, và nhóm người dùng nhạy cảm bắt buộc chuyển dược sĩ.

### 3.2 Kịch bản #4: ảnh hộp thuốc giảm đau + "Ba mình uống từ chiều mà vẫn đau ngực trái, hơi khó thở"

**Đường đi:** failure (4). AI **không cần chắc về tên thuốc**; nó phải chắc về mức nguy hiểm của dấu hiệu.

| Phương án | Quyết định | Vì sao |
|---|---|---|
| Hỏi lại | Loại | Mỗi câu hỏi là thời gian mất đi khi có dấu hiệu cấp cứu. Dù biết tuổi thì kết luận vẫn là đi khám |
| Trả lời về thuốc kèm giới hạn rồi nhắc đi khám | Loại | Ảnh kéo AI vào việc đọc thuốc; thông điệp quan trọng bị đẩy xuống dưới |
| Chỉ chuyển dược sĩ | Loại | Khách vào hàng chờ trong lúc cần cấp cứu |
| Dừng việc thuốc, hiện `khuyen_di_kham` ngay, chuyển dược sĩ song song | Chọn | Thông điệp cứu người đứng đầu; dược sĩ vẫn nhận ca để theo dõi |

**AI nói (nguyên văn):**
> Mình là trợ lý AI của Pharmacity. Đau ngực trái kèm khó thở có thể là dấu hiệu nguy hiểm. Bạn gọi cấp cứu 115 ngay hoặc đưa ba đến cơ sở y tế gần nhất, đừng chờ thuốc. Mình không gợi ý thuốc cho tình huống này. Mình đã báo dược sĩ, nhưng việc cấp cứu không nên chờ bên mình.

Câu này không hứa dược sĩ sẽ gọi hay trả lời, và không nhắc gì đến thuốc trong ảnh.

**Người dùng thấy gì ngoài chữ:**
- Thẻ đỏ ghim ở đầu màn hình.
- Nút lớn **"Gọi 115"** và nút **"Tìm cơ sở y tế gần nhất"**.
- Không có nút đặt hàng, không có nút xem sản phẩm, ảnh thuốc không được phân tích tiếp.
- Dòng nhỏ: "Trợ lý AI không thay thế cấp cứu".

**Người dùng làm gì tiếp:** gọi 115, mở bản đồ, hoặc bỏ qua thẻ (`khuyen_di_kham` bỏ qua được). Nếu không làm gì thẻ đỏ vẫn ghim, AI không chuyển sang chủ đề khác. Nếu khách vẫn nhắn hỏi thuốc, AI nhắc lại một lần thông điệp trên, không gợi ý thuốc. Sau một khoảng thời gian nhóm đặt, AI có thể hỏi một câu: "Ba bạn đã được đưa đi khám chưa?".

**Ai được báo, sau bao lâu:** dược sĩ trực ngay lập tức, ca gắn cờ đỏ **"Khẩn cấp, đã khuyên gọi 115"** và lên đầu hàng chờ. Họ thấy: tin gốc, ảnh, giờ hiện thẻ đỏ, và ghi chú "AI chưa gợi ý thuốc". Dược sĩ có thể nhắn tiếp nếu khách còn online, nhưng không phải điều kiện để khách đi khám.

**Loại lỗi:** lỗi ngữ cảnh (tin nhắn nằm trong khung "đọc thuốc" trong khi bản chất là cấp cứu). Cách nói: ngắn, dứt khoát, không xin lỗi dài, không hỏi vòng. Cách sửa nằm ở thiết kế: **bước kiểm tra dấu hiệu nguy hiểm chạy trước bước đọc ảnh**.

### 3.3 Đường correction (người dùng sửa AI)

- Ở #1: khách sửa tên thuốc hoặc nói "chụp lại rồi đây". AI đọc lại ảnh mới; nếu vẫn là nhóm bắt buộc chuyển thì vẫn chuyển, chỉ bổ sung thông tin cho dược sĩ.
- Ở #4: không có đường sửa nào hạ mức khẩn cấp. Nếu khách nhắn "không sao rồi", thẻ đỏ vẫn giữ và dược sĩ vẫn thấy ca.

### 3.4 Nguyên tắc HAX/PAIR, mỗi cái trỏ vào một chỗ cụ thể

- **G10 (thu hẹp phạm vi khi nghi ngờ)** → câu AI nói ở #1: "chưa đủ để biết chính xác đây là thuốc gì, nên mình chưa hướng dẫn cách dùng" thay vì đoán "Amoxicillin". Chỗ trên prototype: dòng trả lời của AI ở màn 1, không có nút đặt hàng.
- **PAIR Explainability + Trust (kèm G11, giải thích vì sao)** → thẻ ảnh hai vùng "Đọc được / Không đọc được" ở #1. Khách tự kiểm căn cứ, tin đúng mức. Chỗ trên prototype: thẻ ảnh dưới tin của khách.
- **PAIR Errors + Graceful Failure** → thẻ đỏ "Gọi 115" ở #4: lỗi nguy hiểm có đường lui riêng, không đi qua hàng chờ. Chỗ trên prototype: thẻ ghim ở màn 2.

### 3.5 Lỗi sửa câu chữ không cứu được

**AI nói "mình đã chuyển dược sĩ" nhưng `chuyen_duoc_si` chưa chạy hoặc thất bại.**: Khách tin có người nhận ca, trong khi không ai thấy gì. 
- **Thiết kế cứu**: dòng "Đã chuyển dược sĩ" chỉ hiện sau khi hệ thống trả về thành công. Nếu thất bại, khách thấy "Chưa chuyển được, bạn gọi hotline" kèm số hotline. Trạng thái trên màn hình lấy từ hệ thống, không do AI tự viết.

## §4. Mức tự chủ  (chặng 4)

| Tool | Mức | Sai thì ai chịu gì, đảo lại thế nào | Lý do chọn mức |
|---|---|---|---|
| `doc_anh_bao_bi` *(tool mới: chỉ đọc chữ trên ảnh, trả về "đọc được / không đọc được", không suy luận, không tra cứu)* | Tự làm | Đọc sai thì khách thấy ngay ở thẻ "Đọc được" và tự kiểm được | Sai thì rẻ, khách tự thấy, tự sửa. Bắt người duyệt mỗi lần đọc ảnh là chờ vô ích |
| `khuyen_di_kham` | Tự làm | Hiện thừa thì khách mất vài phút, bỏ qua được. Hiện thiếu thì mất sức khoẻ ngay (#4) | Chờ người duyệt là kiểu sai đắt nhất ở đây: ca cấp cứu không được nằm trong hàng chờ. Thừa thì rẻ, thiếu thì không đảo được |
| `chuyen_duoc_si` | Tự làm | Chuyển thừa tốn vài phút của dược sĩ. Chuyển thiếu để ca nhóm nhạy cảm lọt qua (#1, #2) | Đảo được, rẻ |
| `luu_ghi_chu_suc_khoe` | Cần người duyệt | Lưu sai hoặc lưu thông tin bên thứ ba: xoá được nhưng đã có rủi ro lộ | Để yêu cầu không lưu khi khách chưa đồng ý. Khách thấy nội dung và đối tượng trước khi bấm |
| `goi_y_san_pham` | Cần người duyệt. *Cấm* khi ảnh chưa xác nhận | Thuốc đã dùng thì không đảo được. Người chịu là người uống, thường không phải người nhắn | Phần 4.3 |
| `ban_thuoc_ke_don` | cấm | Không đảo được và ngoài thẩm quyền AI | Một bước xác nhận cũng không đủ chặn. Khách đòi thì AI từ chối và chuyển dược sĩ |

**Luật thêm:**
1. Bước kiểm tra dấu hiệu nguy hiểm chạy trước `doc_anh_bao_bi`.
2. Kết quả `doc_anh_bao_bi` có phần "không đọc được" ở tên thuốc thì `goi_y_san_pham` bị khoá.
3. Tên thuốc do khách gõ vào chỉ là "khách cho biết", đính kèm cho dược sĩ, không mở khoá `goi_y_san_pham`.
4. Không tool nào được trả lời "đã chuyển" hoặc "đã lưu" trước khi hệ thống trả về thành công.

### 4.2 Tool thêm hoặc tách (điểm "Tốt khi")

- Thêm `doc_anh_bao_bi`: Tách "đọc" khỏi "gợi ý" để lỗi ở #1 bị chặn: AI chỉ báo cáo chữ đọc được, không có đường nào từ "đoán tên" sang "gửi nút đặt".

### 4.3 Tool khó xếp nhất: `goi_y_san_pham`

Cả ba mức đều có giá:

| Mức | Giá |
|---|---|
| Tự làm | Gửi nút đặt cho thuốc đọc nhầm hoặc người uống thuộc nhóm nhạy cảm. Không đảo được |
| Người duyệt | Khách chờ. Với thuốc thông thường như nước muối sinh lý thì chờ là lãng phí thời gian |
| Cấm hoàn toàn | AI mất giá trị chính của kênh, khách ra ngoài tự mua |

### 4.4 Thử mức trên 3 kịch bản

| Kịch bản | AI làm gì với mức này | Người dùng thấy | Bị chặn? | AI vẫn làm khi lẽ ra phải hỏi? | Chờ vô ích? |
|---|---|---|---|---|---|
| **#1** | `doc_anh_bao_bi` trả "đọc được ...cillin 500, không đọc được tên"; luật 2 khoá `goi_y_san_pham`; `chuyen_duoc_si` chạy | Thẻ hai vùng, trạng thái "Đã chuyển dược sĩ", không có nút đặt | **Có.** Việc đoán "Amoxicillin" không có đường đi vì tool đọc không suy luận và tool gợi ý bị khoá | Không | Không: người 70 tuổi vốn bắt buộc chuyển |
| **#4** | Kiểm tra nguy hiểm chạy trước; `khuyen_di_kham` tự làm; `chuyen_duoc_si` song song cờ đỏ; `doc_anh_bao_bi` và `goi_y_san_pham` không chạy | Thẻ đỏ, nút Gọi 115, không có nút đặt | **Có.** Không tool thuốc nào chạy | Không | Không: không qua duyệt |
| **#8** *(kịch bản bình thường)* | `doc_anh_bao_bi` đọc ảnh; hỏi một câu "ai sẽ uống?"; nước muối sinh lý thuộc danh sách an toàn nên gợi ý tự làm nếu người uống nhóm bình thường; phần thuốc trong ảnh vẫn qua sàng lọc | Hai phần trả lời tách rõ, nút đặt chỉ ở nước muối | **Có.** Phần thuốc trong ảnh không tự gợi ý | Không | ***Một phần:*** việc nước muối không chờ, việc thuốc trong ảnh có chờ dược sĩ. Chấp nhận, vì đây là phần rủi ro |

### 4.5 Mức tự chủ đã đổi lời hứa nào với người dùng

Nếu `goi_y_san_pham` để "tự làm", lời hứa là "AI chọn thuốc cho bạn". Lời hứa đó đồng nghĩa AI cam kết đã nhận đúng thuốc từ ảnh, điều nó không làm được. Mức đã chọn đổi lời hứa thành: *"AI đọc cho bạn xem trên ảnh có gì, dược sĩ xác nhận, bạn quyết định mua"*. Người dùng mất một lượt chờ, đổi lại không ai phải tin vào một tên thuốc do AI đoán. Lời hứa này cũng là câu nên xuất hiện trong dòng G1/G2.

## §5. Prototype  (chặng 5)
### 5.1 Trả lời ngắn

1. **AI chắc tới đâu, dựa vào đâu:** thẻ "Đọc được: ...cillin 500 / Không đọc được: tên thuốc, hạn dùng" và nhãn "Chưa xác nhận tên thuốc". Khách tự kiểm căn cứ ngay trên ảnh của mình.
2. **Ba mức trên màn hình:**
    - **Tự làm** (`doc_anh_bao_bi`, `chuyen_duoc_si`, `khuyen_di_kham`): không cần bấm gì. Chuyển nhầm thì tốn vài phút dược sĩ; khách bỏ qua được thẻ đỏ. Việc đọc sai thì khách bấm "Phần đọc được không đúng".
    - **Cần duyệt** (`goi_y_san_pham`: dược sĩ duyệt; `luu_ghi_chu_suc_khoe`: khách bấm sau khi xem bản nháp). Khách thấy trạng thái "Đã chuyển dược sĩ", không có nút đặt cho đến khi dược sĩ xác nhận.
    - **Cấm** (`ban_thuoc_ke_don`, và mức (3) của gợi ý): AI từ chối ngắn và chuyển dược sĩ, nút đặt không xuất hiện.
3. **Nguyên tắc nằm ở đâu:** 
    - G1/G2 ở số 1 (nhãn AI)
    - PAIR Explainability + G11 ở số 2 (thẻ hai vùng)
    - G10 ở số 3 (câu AI không đoán)
    - PAIR Errors + Graceful Failure ở số 4 (trạng thái chuyển từ hệ thống) và số 6 (thẻ đỏ)

[Prototype-html](./prototype/index.html)

[Prototype-png](./prototype/prototype-preview.png)

## §7. Demo
- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.

