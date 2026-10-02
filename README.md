## Mục tiêu

Bài thực hành minh họa hai cách xử lý một commit có mã nguồn bị lỗi:

- `git reset --mixed`: lùi lịch sử cục bộ nhưng vẫn giữ mã nguồn để tiếp tục sửa.
- `git revert`: tạo một commit mới đảo ngược commit lỗi, phù hợp với nhánh đã chia sẻ.

## Chuẩn bị

Kiểm tra trạng thái và lịch sử trước khi thao tác:

```bash
git status
git log --oneline -n 5
```

Trong bài này, commit `feat: introduce faulty subtraction` đóng vai trò là commit chứa lỗi.

## Trường hợp 1: Sửa đổi cục bộ bằng `reset`

Khi commit lỗi chỉ tồn tại ở local và chưa được đẩy lên remote, thực hiện:

```bash
git reset --mixed HEAD~1
```

Có thể viết ngắn gọn vì `--mixed` là chế độ mặc định:

```bash
git reset HEAD~1
```

Kết quả:

- `HEAD` và nhánh hiện tại lùi lại một commit.
- Commit lỗi không còn xuất hiện ở đầu lịch sử local.
- Nội dung của commit lỗi vẫn được giữ trong Working Directory.
- Các tệp bị ảnh hưởng xuất hiện dưới dạng `modified`, chưa nằm trong Staging Area.

Kiểm tra:

```bash
git log --oneline -n 5
git status --short
```

Kết quả minh họa của `git status --short`:

```text
 M calculator.js
```

Sau đó có thể sửa lại mã nguồn, đưa tệp vào Staging Area và tạo commit mới:

```bash
git add calculator.js
git commit -m "fix: correct subtraction logic"
```

> Không dùng `git reset --hard HEAD~1` trong tình huống này vì lệnh đó xóa cả thay đổi trong Working Directory và có thể làm mất phần mã đang cần sửa.

## Trường hợp 2: Sửa đổi công cộng bằng `revert`

Khi commit lỗi đã được đẩy lên remote hoặc đã xuất hiện trên nhánh chung, không viết lại lịch sử. Trước tiên, tìm mã commit cần đảo ngược:

```bash
git log --oneline -n 5
```

Sau đó tạo commit đối lập:

```bash
git revert <commit-id-cua-commit-loi>
```

Nếu trình soạn thảo được mở, giữ thông điệp mặc định rồi lưu và đóng. Git tạo một commit mới có tiêu đề dạng:

```text
Revert "feat: introduce faulty subtraction"
```

Kiểm tra kết quả:

```bash
git log --oneline -n 5
git status
```

Commit lỗi vẫn còn trong lịch sử, nhưng thay đổi của nó đã được commit `Revert ...` đảo ngược. Working Directory trở về trạng thái sạch và commit mới có thể được đẩy lên remote an toàn:

```bash
git push origin main
```

## Ảnh minh chứng

Ảnh dưới đây ghi lại kết quả thật của lệnh `git log --oneline -n 5` sau khi tạo commit revert:

![Lịch sử commit sau khi revert](images/revert-log.png)

## So sánh `reset` và `revert`

| Tiêu chí | `git reset --mixed HEAD~1` | `git revert <commit-id>` |
|---|---|---|
| Cách hoạt động | Di chuyển con trỏ nhánh và `HEAD` về commit trước | Tạo commit mới có nội dung đối lập |
| Lịch sử | Viết lại lịch sử local | Giữ nguyên lịch sử đã có |
| Mã nguồn hiện hành | Được giữ lại dưới dạng chưa stage | Được đảo ngược và ghi nhận trong commit mới |
| Phạm vi phù hợp | Commit local, chưa chia sẻ | Commit đã push hoặc nhánh dùng chung |
| Mức độ an toàn khi làm nhóm | Không nên dùng để sửa lịch sử đã chia sẻ | An toàn vì không thay đổi commit cũ |
| Cách đẩy lên remote | Có thể cần force push nếu lịch sử đã từng được đẩy, vì vậy nên tránh | Push bình thường |
