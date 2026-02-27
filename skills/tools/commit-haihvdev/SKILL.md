---
name: commit
description: Tạo git commit với message tiếng Việt theo quy tắc trong CLAUDE.md
disable-model-invocation: true
---

Phân tích các thay đổi trong git (`git diff --staged` và `git status`) rồi tạo commit message theo đúng quy tắc trong CLAUDE.md:

- Định dạng: `<type>: <mô tả ngắn gọn>`
- Viết bằng tiếng Việt có dấu
- Type giữ nguyên tiếng Anh: feat, fix, refactor, docs, style, test, chore
- Với commit phức tạp (nhiều thay đổi, refactor lớn): BẮT BUỘC thêm body giải thích chi tiết

Sau khi tạo commit message, hỏi người dùng xác nhận trước khi thực hiện commit.

Nếu chưa có file nào được staged, hãy thông báo để người dùng chạy `git add` trước.
