# Ghi chú phân tích · Day 04

Họ và tên:
MSSV:

## 1. Case được chọn

- ID case: E1
- Câu trả lời của agent:Agent đã gọi thành công tool get_status với provider là github, component là actions. Kết quả trả về cho biết GitHub Actions đang hoạt động bình thường, không có sự cố đang diễn ra. Agent cũng liệt kê 5 incident gần đây, tất cả đều có trạng thái resolved, đồng thời cung cấp đường dẫn GitHub Status làm nguồn tham khảo. Tuy nhiên, câu trả lời chưa làm rõ mốc thời gian của snapshot và chưa phân biệt rõ thời điểm cập nhật incident với thời điểm incident xảy ra.
- Điểm ở lần chạy 1: 75

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```
{
  "event": "tool",
  "step": 1,
  "name": "get_status",
  "arguments": {
    "provider": "github",
    "component": "actions"
  },
  "result": {
    "captured_at": "2026-11-09T03:50:47.065020+00:00",
    "overall": {
      "indicator": "none",
      "description": "All Systems Operational"
    },
    "components": [
      {
        "name": "Actions",
        "status": "operational"
      }
    ],
    "incidents": [
      {
        "id": "pfbmpxxfq1cy",
        "name": "Degraded performance for multiple GitHub services",
        "status": "resolved",
        "updated_at": "2026-11-06T00:20:49.369Z"
      },
      {
        "id": "t89g2q1800fr",
        "name": "Incident with Actions",
        "status": "resolved",
        "updated_at": "2026-11-05T17:42:35.455Z"
      }
    ]
  }
}

```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Thêm cơ chế kiểm tra (validation) và tự sửa lỗi (self-correction) cho URL nguồn trực tiếp vào vòng lặp của hàm run_agent trong file agent.py
- Vì sao chọn thay đổi này: Dù System Prompt đã yêu cầu "Dẫn nguồn từ tool", model vẫn thường xuyên có xu hướng tóm tắt thông tin và lược bỏ URL chi tiết của incident. Thay vì tiếp tục tinh chỉnh prompt với rủi ro không ổn định, việc dùng code logic để ép model tuân thủ contract sẽ đảm bảo giải quyết triệt để lỗi thiếu nguồn đối với các incident đã được gọi chi tiết.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 76.59 |  76.36|
| Điểm case đã chọn | 75 | 65 |
| Số model calls | 2 steps |  2 steps |
| Token, chi phí nếu có |  |  |

- Lỗi đã giải quyết:
- Lỗi còn lại hoặc mới xuất hiện:

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.

- Giới hạn của điểm judge: LLM Judge (GPT-4o-mini) tự đánh giá câu trả lời nên mang tính chủ quan và có độ dao động (variance) cao. Cùng một nội dung mang ý nghĩa tương đương (như ở case E1), judge có thể đưa ra mức điểm và lý do trừ điểm khác nhau giữa các lần chạy, dẫn đến điểm tổng thể bị thiên lệch.   

- Giới hạn của snapshot: Dữ liệu cung cấp qua tools là dữ liệu tĩnh và bị giới hạn ở tối đa 50 incident gần nhất. Do đó, kết quả trả về không chứng minh được tính toàn vẹn của lịch sử hệ thống. 

- Chỉ chạy một lần mỗi phiên bản: Theo tài liệu hướng dẫn, chỉ một lần chạy cho mỗi phiên bản không đủ thống kê để chứng minh độ ổn định của prompt hay agent loop, do bản chất sinh ngẫu nhiên của mô hình ngôn ngữ lớn. Cần chạy nhiều lần để tính trung bình mới có kết luận chính xác về hiệu quả của code cải tiến.   
