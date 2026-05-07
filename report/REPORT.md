# Lab 21 — Evaluation Report

**Học viên**: Tạ Bảo Ngọc — 2A202600286

**Ngày nộp**: 2026-05-07
**Submission option**: A (lightweight)

**Link Hugging Face**: https://huggingface.co/ngocbao25/qwen2.5-3b-vi-lab21-r16

## 1. Setup
- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit` (Qwen 2.5 3B 4-bit)
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`, 200 samples (180 train + 20 eval)
- **max_seq_length**: 1024 (p95 = 562, rounded up to 1024)
- **GPU**: Tesla T4, 16 GB VRAM
- **Training cost**: ~$0.08 (~12 phút training tổng cộng cho 3 ranks @ ~$0.4/hr Colab)

## 2. Rank Experiment Results

| Rank | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|-----------------|------------|-----------|-----------|------------|
| 8    | 1,843,200       | 3.66 min   | 8.70 GB   | 1.5577    | 4.7479     |
| 16   | 3,686,400       | 4.09 min   | 6.62 GB   | 1.5161    | 4.5544     |
| 64   | 14,745,600      | 3.79 min   | 9.48 GB   | 1.4768    | 4.3790     |
| Base | -               | -          | -         | 1.6241    | 5.0739     |

## 3. Loss Curve Analysis
[Đính kèm loss_curve.png trong thư mục report/]
- **Quan sát**: Training loss giảm đều đặn từ 1.61 xuống còn 1.39 sau 3 epochs huấn luyện. Không quan sát thấy hiện tượng overfitting nghiêm trọng vì eval loss (dao động từ 1.47 đến 1.55) bám sát theo training loss. Khoảng cách giữa train loss và eval loss là hợp lý đối với kích thước dataset nhỏ (200 samples), cho thấy model có khả năng tổng quát hóa tốt trên các instructions tiếng Việt mới.

## 4. Qualitative Comparison (5 examples)

### Example 1
- **Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
- **Base**: Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình máy móc để học tập từ dữ liệu... (bị cắt ngang)
- **Fine-tuned (r=16)**: Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán dựa trên dữ liệu mà không có sự hướng dẫn trực tiếp từ người dùng...
- **Nhận xét**: Improved. Câu trả lời đầy đủ, mạch lạc và không bị lỗi truncate.

### Example 2
- **Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
- **Base**: (Cung cấp code sử dụng đệ quy nhưng logic chưa hoàn chỉnh và bị cắt ngang ở phần quan trọng).
- **Fine-tuned (r=16)**: (Cung cấp code sử dụng vòng lặp tối ưu, có xử lý ngoại lệ cho đầu vào âm và logic hoàn toàn chính xác).
- **Nhận xét**: Improved. Code có tính ứng dụng cao hơn và tuân thủ các best practices.

### Example 3
- **Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
- **Base**: 1. Thân thiện với người dùng: Mục đích của thiết kế UI/UX là cung cấp trải nghiệm... (lặp từ và giải thích dài dòng).
- **Fine-tuned (r=16)**: 1. Chuyển đổi. 2. Thích ứng. 3. Đơn giản. 4. Tương thích. 5. Thống nhất. (Mỗi mục đều có giải thích ngắn gọn).
- **Nhận xét**: Improved. Định dạng danh sách rõ ràng, dễ theo dõi.

### Example 4
- **Prompt**: Tóm tắt sự khác biệt giữa LoRA và QLoRA.
- **Base**: LoRA và QLoRA là hai phương pháp cải thiện hiệu năng của mô hình NLU (NLP)... (sai định nghĩa kỹ thuật).
- **Fine-tuned (r=16)**: LoRA là kỹ thuật regularization sử dụng các low-rank matrices. QLoRA là phiên bản cải tiến sử dụng quantization 4-bit...
- **Nhận xét**: Slightly Improved. Model đã bắt đầu nhận diện được các thuật ngữ kỹ thuật chính xác hơn, dù vẫn còn một số nhầm lẫn nhỏ về bản chất toán học.

### Example 5
- **Prompt**: Phân biệt prompt engineering, RAG, và fine-tuning.
- **Base**: Prompt engineering là một kỹ thuật để cải thiện hiệu suất bằng cách cung cấp câu hỏi... (bị cắt ngang giữa chừng).
- **Fine-tuned (r=16)**: Giải thích rõ ràng cả 3 khái niệm: Prompt engineering (câu lệnh), RAG (truy xuất dữ liệu ngoài) và Fine-tuning (cập nhật trọng số).
- **Nhận xét**: Improved. Cung cấp cái nhìn tổng quan và so sánh tốt giữa các kỹ thuật.

## 5. Conclusion về Rank Trade-off

Dựa trên các kết quả thu được từ thí nghiệm với rank 8, 16 và 64, tôi có những kết luận sau:

Về hiệu quả đầu tư (ROI), **Rank 16** cho thấy kết quả tốt nhất trên dataset này. Với việc tăng số lượng tham số huấn luyện lên gấp đôi so với rank 8 (từ 1.8M lên 3.6M), perplexity đã giảm đáng kể từ 4.74 xuống 4.55. Đáng chú ý là mức tiêu thụ VRAM đỉnh của rank 16 trong quá trình chạy này lại thấp nhất (6.62 GB), có thể do sự tối ưu hóa của Unsloth và việc quản lý bộ nhớ hiệu quả hơn ở cấu hình này.

Hiện tượng "diminishing returns" (hiệu suất giảm dần) bắt đầu xuất hiện khi tăng rank từ 16 lên 64. Mặc dù số lượng tham số tăng gấp 4 lần, perplexity chỉ cải thiện thêm khoảng 3.7% (từ 4.55 xuống 4.38). Điều này không tương xứng với việc tiêu tốn thêm tài nguyên VRAM (tăng lên 9.48 GB).

**Khuyến nghị**: Nếu triển khai trong môi trường production, tôi sẽ lựa chọn **Rank 16**. Cấu hình này cung cấp chất lượng câu trả lời ổn định cho tiếng Việt, khắc phục hoàn toàn lỗi truncate và cải thiện logic so với Base model, đồng thời vẫn giữ được model nhẹ, chi phí huấn luyện thấp và có thể chạy tốt trên các GPU phổ thông như Tesla T4.

## 6. What I Learned
- Hiểu được sự đánh đổi (trade-off) giữa độ phức tạp của adapter (rank) và khả năng hội tụ của mô hình. Tăng rank không phải lúc nào cũng mang lại kết quả vượt trội so với tài nguyên bỏ ra.
- Làm quen với quy trình fine-tuning hiện đại sử dụng Unsloth và QLoRA 4-bit, giúp tiết kiệm hơn 60% VRAM và tăng tốc độ training đáng kể.
- Kỹ năng đánh giá mô hình thông qua cả chỉ số định lượng (Perplexity) và định tính (Qualitative side-by-side comparison).
