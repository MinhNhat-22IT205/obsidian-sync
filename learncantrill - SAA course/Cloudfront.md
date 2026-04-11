#cloudfront-related
Invalidate CloudFront" là gì?
**Invalidate = xóa cache đã lưu trên các edge server**

👉 Khi bạn deploy FE mới:

- File mới đã nằm ở S3 ✅
- Nhưng CloudFront vẫn giữ **bản cũ trong cache** ❌

➡️ User vẫn thấy version cũ