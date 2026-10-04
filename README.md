# Bài 5: Khôi phục trạng thái và đảo ngược commit (Reset vs Revert)

## 1. Mục tiêu

Bài thực hành trình bày hai cách xử lý một commit chứa mã nguồn bị lỗi:

- Dùng `git reset` để viết lại lịch sử **cục bộ**, đồng thời giữ mã nguồn trong Working Directory để sửa tiếp.
- Dùng `git revert` để tạo một commit **mới** đảo ngược commit lỗi, phù hợp với lịch sử đã được chia sẻ lên remote.

![So sánh luồng xử lý bằng Reset và Revert](./images/reset-vs-revert.svg)

> Hai trường hợp bên dưới nên được thực hành độc lập hoặc trên hai nhánh khác nhau. Nếu đã reset làm commit lỗi biến mất khỏi nhánh hiện tại thì sẽ không còn đúng bối cảnh để tiếp tục revert commit đó trên cùng một chuỗi thao tác.

---

## 2. Trường hợp 1: Sửa đổi lịch sử cục bộ bằng Reset

### Bối cảnh

Commit mới nhất ở local chứa mã nguồn bị lỗi và **chưa được push** lên remote. Mục tiêu là đưa `HEAD` lùi lại một commit nhưng vẫn giữ toàn bộ nội dung thay đổi để chỉnh sửa.

### Bước 1: Xem lịch sử hiện tại

```bash
git log --oneline -n 5
```

Ví dụ trước khi reset:

```text
a12bc34 (HEAD -> reset-demo) Add feature with bug
8de91f2 Complete previous feature
34ac891 Initial project
```

Trong ví dụ này, `a12bc34` là commit chứa lỗi.

### Bước 2: Di chuyển HEAD lùi lại một commit

```bash
git reset --mixed HEAD~1
```

`--mixed` là chế độ mặc định, vì vậy lệnh trên cũng có thể viết ngắn gọn như sau:

```bash
git reset HEAD~1
```

### Bước 3: Kiểm tra kết quả

```bash
git log --oneline -n 5
git status
```

Kết quả log mong đợi:

```text
8de91f2 (HEAD -> reset-demo) Complete previous feature
34ac891 Initial project
```

Kết quả `git status` mong đợi:

```text
Changes not staged for commit:
  modified:   <ten-tep>
```

Sau thao tác trên:

- `HEAD` và nhánh hiện tại đã lùi lại một commit.
- Commit lỗi không còn nằm ở đầu lịch sử của nhánh local.
- Nội dung mã nguồn của commit lỗi **không bị mất**.
- Các tệp thay đổi được giữ trong Working Directory ở trạng thái `Modified`.
- Các thay đổi chưa nằm trong Staging Area nên có thể tiếp tục chỉnh sửa.

Sau khi sửa lỗi, có thể kiểm tra, stage và tạo commit thay thế:

```bash
git add .
git commit -m "Fix feature implementation"
```

### Lưu ý an toàn

Không sử dụng lệnh sau cho yêu cầu của bài:

```bash
git reset --hard HEAD~1
```

`--hard` cập nhật cả lịch sử, Staging Area và Working Directory. Vì vậy, nội dung chưa được lưu ở nơi khác có thể bị mất, trái với yêu cầu phải giữ mã nguồn hiện hành.

---

## 3. Trường hợp 2: Đảo ngược commit công cộng bằng Revert

### Bối cảnh

Commit lỗi đã được push lên remote hoặc đã được các thành viên khác sử dụng. Trong trường hợp này không nên viết lại lịch sử chung bằng `reset` và force push. Thay vào đó, cần tạo một commit mới chứa thay đổi đối lập.

Giả sử lịch sử hiện tại là:

```text
d47ae21 (HEAD -> revert-demo) Update project documentation
a12bc34 Add feature with bug
8de91f2 Complete previous feature
34ac891 Initial project
```

Commit cần đảo ngược là `a12bc34`.

### Bước 1: Tạo commit đảo ngược

```bash
git revert a12bc34
```

Git sẽ mở trình soạn thảo để xác nhận commit message mặc định, thường có dạng:

```text
Revert "Add feature with bug"
```

Nếu muốn chấp nhận commit message mặc định mà không mở trình soạn thảo, có thể dùng:

```bash
git revert --no-edit a12bc34
```

### Bước 2: Kiểm tra lịch sử

```bash
git log --oneline -n 5
```

Kết quả mong đợi:

```text
f93ad82 (HEAD -> revert-demo) Revert "Add feature with bug"
d47ae21 Update project documentation
a12bc34 Add feature with bug
8de91f2 Complete previous feature
34ac891 Initial project
```

Kết quả này chứng minh rằng:

- Commit lỗi `a12bc34` vẫn còn trong lịch sử.
- Git tạo thêm commit mới `f93ad82` ở đầu nhánh.
- Tiêu đề commit mới bắt đầu bằng `Revert`.
- Nội dung của commit mới đảo ngược các thay đổi do commit lỗi tạo ra.

Sau khi kiểm tra mã nguồn, có thể đẩy commit revert lên remote:

```bash
git push origin <ten-nhanh>
```

Ví dụ với nhánh `main`:

```bash
git push origin main
```

### Trường hợp phát sinh conflict

Nếu mã nguồn đã thay đổi nhiều kể từ commit lỗi, `git revert` có thể phát sinh conflict. Khi đó cần sửa các tệp bị conflict, đưa chúng vào Staging Area rồi tiếp tục:

```bash
git add <tep-da-xu-ly>
git revert --continue
```

Nếu muốn hủy toàn bộ lần revert đang thực hiện:

```bash
git revert --abort
```

---

## 4. So sánh Reset và Revert

| Tiêu chí | `git reset` | `git revert` |
|---|---|---|
| Cách hoạt động | Di chuyển `HEAD` và con trỏ nhánh | Tạo commit mới có thay đổi đối lập |
| Có viết lại lịch sử không? | Có | Không |
| Có tạo commit mới không? | Không | Có |
| Commit lỗi còn trong log không? | Có thể không còn trên nhánh hiện tại | Vẫn còn nguyên |
| Trường hợp phù hợp | Commit local, chưa chia sẻ | Commit đã push hoặc nằm trên nhánh chung |
| Mức độ an toàn khi làm việc nhóm | Không an toàn nếu lịch sử đã được chia sẻ | An toàn và minh bạch |
| Giữ mã nguồn để sửa tiếp | Có với `--mixed` hoặc `--soft` | Không theo cách đó; Git áp dụng bản vá đối lập |
| Rủi ro chính | Mất dữ liệu nếu dùng nhầm `--hard`; gây lệch lịch sử nếu force push | Có thể phát sinh conflict |

---

## 5. Khác biệt khi làm việc cá nhân và làm việc nhóm

### Làm việc cá nhân

Khi commit lỗi chỉ tồn tại ở local và chưa được push, có thể dùng:

```bash
git reset --mixed HEAD~1
```

Cách này giúp loại bỏ commit chưa phù hợp khỏi đầu nhánh nhưng giữ nguyên mã nguồn trong Working Directory để sửa và commit lại. Đây là cách làm gọn lịch sử cá nhân trước khi chia sẻ.

### Làm việc nhóm trên nhánh chung

Khi commit đã được push, các thành viên khác có thể đã dựa trên lịch sử đó để phát triển tiếp. Nếu dùng `reset` rồi force push, lịch sử trên remote bị thay đổi và có thể gây lệch nhánh, conflict hoặc làm mất commit của người khác.

Trong trường hợp này nên dùng:

```bash
git revert <commit-id>
```

`git revert` giữ nguyên lịch sử đã chia sẻ và bổ sung một commit mới ghi lại rõ ràng hành động đảo ngược. Các thành viên khác vẫn có thể pull hoặc merge theo quy trình thông thường.

---

## 6. Kết luận

- Dùng `git reset --mixed HEAD~1` khi commit lỗi chỉ tồn tại ở local và cần giữ mã nguồn để sửa lại.
- Không dùng `git reset --hard` khi cần bảo toàn thay đổi trong Working Directory.
- Dùng `git revert <commit-id>` khi commit đã được chia sẻ hoặc nằm trên nhánh chung.
- `reset` phù hợp để chỉnh sửa lịch sử cá nhân; `revert` phù hợp để sửa lỗi an toàn, minh bạch khi làm việc nhóm.

---

## 7. Ảnh chụp minh chứng Revert

Sau khi thực hành thật, chạy:

```bash
git log --oneline -n 5
```

Chụp màn hình sao cho thấy commit mới có tiêu đề bắt đầu bằng `Revert`, lưu ảnh tại:

```text
images/revert-commit-history.png
```

Sau đó bỏ dấu `<!--` và `-->` ở đoạn dưới để ảnh xuất hiện trong báo cáo:

```markdown
<!--
![Lịch sử commit sau khi thực hiện revert](./images/revert-commit-history.png)
-->
```

> Hiện chưa chèn ảnh lịch sử commit vì bài được soạn dưới dạng nội dung và không thực thi thao tác Git. Ảnh minh chứng phải được chụp từ kết quả thực hành thực tế, không sử dụng kết quả giả lập.
