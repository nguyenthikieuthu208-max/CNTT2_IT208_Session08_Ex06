***BÀI6: RÀ SOÁT SCRIPT BẢO TRÌ HỆ THỐNG***

Phần 1: Phân tích mã nguồn AI (Auditing)

1\. Điểm cực kỳ nguy hiểm trong đoạn mã AI gợi ý:

Đoạn mã do AI cung cấp ở trên thực chất là một "lệnh hủy diệt" hệ thống. Nếu chạy lệnh này với quyền Quản trị viên (Administrator), máy chủ của "Trang trại Lan thông minh" sẽ bị xóa sạch toàn bộ hệ điều hành ngay lập tức.

2\. Giải thích ý nghĩa của từng tham số:

\-Path "C:\\\*": Chỉ định đối tượng tác động là tất cả mọi tệp tin và thư mục nằm ở gốc của ổ đĩa hệ thống C:\\ (bao gồm cả các thư mục cốt lõi của hệ điều hành như Windows, Program Files, Users).

\-Recurse: Lệnh xóa đệ quy. Nó ra lệnh cho hệ thống quét sâu vào tận cùng của tất cả các thư mục con, thư mục cháu bên trong ổ C:\\ để xóa sạch sành sanh không chừa một file nào.

\-Force: Ép buộc hệ thống thực thi lệnh xóa mà không cần hỏi lại người dùng (bỏ qua mọi cảnh báo an toàn). Nó cũng tự động xóa luôn các tệp thuộc dạng ẩn hoặc tệp chỉ đọc (Read-only).

3\. Hậu quả thực tế:

Máy chủ sẽ bị sập hoàn toàn trong vòng vài giây, xuất hiện lỗi màn hình xanh chết chóc (BSOD). Khi khởi động lại, máy sẽ báo lỗi không tìm thấy hệ điều hành do các tệp tin hạt nhân hệ thống đã bị xóa sổ vĩnh viễn, gây gián đoạn toàn bộ hoạt động của Trang trại Lan và làm mất trắng toàn bộ dữ liệu quan trọng.

4\. Áp dụng 3 Nguyên tắc "Vàng" thẩm định mã AI:

Nguyên tắc 1 \- Không bao giờ chạy code khi chưa hiểu rõ từng tham số: AI có thể hiểu nhầm từ "tệp nhật ký cũ" thành "tất cả các tệp" và đưa ra đường dẫn C:\\\* vô tội vạ. Lập trình viên phải luôn tra cứu tài liệu (bằng lệnh Get-Help) trước khi chạy.

Nguyên tắc 2 \- Kiểm tra phạm vi tác động (Scope): Code dọn dẹp hệ thống bắt buộc phải được "khoanh vùng" trong một thư mục đích cụ thể (ví dụ: C:\\project\\logs), tuyệt đối không được sử dụng ký tự đại diện nguy hiểm bao phủ toàn bộ ổ đĩa (\*).

Nguyên tắc 3 \- Thử nghiệm trên môi trường giả lập (Sandbox): Tuyệt đối không chạy thử code bảo trì do AI viết trực tiếp trên máy chủ Production (máy chủ đang vận hành thực tế) mà phải test trước trên máy ảo hoặc môi trường Staging an toàn.

Phần 2: Triển khai mã nguồn an toàn (safe\_cleanup.ps1)

Áp dụng đúng Nguyên tắc khoanh vùng đường dẫn, dưới đây là kịch bản bảo trì hệ thống chuẩn mực và an toàn tuyệt đối cho trang trại:

powershell  
Tên kịch bản: safe\_cleanup.ps1  
Công dụng: Bảo trì thư mục dự án, lọc lỗi WARNING và dọn file tạm

Khai báo đường dẫn thư mục dự án cụ thể (Khoanh vùng phạm vi an toàn)  
\$ThuMucDuAn \= "D:\\smart\_farm\_project"

Bước 1: Di chuyển đến thư mục làm việc của dự án  
Set-Location \-Path \$ThuMucDuAn

Bước 2: Đi sâu vào thư mục logs để lọc các dòng WARNING từ system.log ra tệp issues.txt  
Write-Host "\[\>\] Dang trich xuat cac canh bao tu system.log..." \-ForegroundColor Yellow  
Select-String \-Path ".\\logs\\system.log" \-Pattern "WARNING" | Add-Content \-Path ".\\logs\\issues.txt" \-Encoding utf8

Bước 3: Chỉ xóa các tệp .tmp bên trong thư mục dự án (Không chạm vào phân vùng hệ thống)  
Write-Host "\[\>\] Dang don dep cac tep tin tam thoi (.tmp)..." \-ForegroundColor Yellow

Kiểm tra xem có file .tmp nào không trước khi xóa để tránh báo lỗi thừa  
if (Test-Path ".\\\*.tmp") {  
    Remove-Item \-Path ".\\\*.tmp" \-Force  
    Write-Host "\[OK\] Da xoa sach cac tep .tmp trong thu muc dự án." \-ForegroundColor Green  
} else {  
    Write-Host "\[\*\] Khong co tep .tmp nao can xoa." \-ForegroundColor Cyan  
}

Write-Host "\[HOAN THANH\] Quy trinh bao tri he thong ket thuc an toan\!" \-ForegroundColor Green

Phần 3: Kiểm thử và Xác nhận kết quả (Verify)

Để kiểm tra xem đoạn script an toàn trên có hoạt động chính xác hay không, mình sẽ mở PowerShell tại thư mục dự án và gõ tuần tự bộ 3 câu lệnh kiểm thử sau:

1. Test-Path \-Path ".\\logs\\issues.txt"  
   * *Mục đích:* Kiểm tra xem tệp báo cáo lỗi đã được tạo thành công hay chưa.  
   * *Kết quả mong đợi:* Hệ thống trả về giá trị True, chứng tỏ kịch bản đã chạy và xuất tệp tin đúng vị trí chỉ định.  
2. Get-Content \-Path ".\\logs\\issues.txt" \-Tail 5  
   * *Mục đích:* Đọc nội dung bên trong tệp issues.txt để xác nhận xem hệ thống có lọc đúng các dòng chứa chữ WARNING hay không.  
   * *Kết quả mong đợi:* Trả về các dòng văn bản có chứa từ khóa WARNING, phông chữ hiển thị chuẩn xác, không bị lỗi rác ký tự.  
3. Get-ChildItem \-Filter "\*.tmp"  
   * *Mục đích:* Quét lại thư mục dự án một lần nữa để xem còn tệp tin tạm nào không.  
   * *Kết quả mong đợi:* Màn hình Terminal trống rỗng (không liệt kê ra file nào), điều này xác nhận toàn bộ các file .tmp cũ đã bị câu lệnh Remove-Item quét sạch thành công.

