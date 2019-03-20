
# Git flow 

## Naming
**I. Tên branch:**
1. Feature tổng: **feature_[mã issue github]_[tóm tắt nội dung]**
2. Task: **task_[mã ticket redmine]_[tóm tắt nội dung]**
3. Bug: **bug_[mã_ticket_redmine]_[tóm tắt nội dung]**

**II. Commit message:**
1. Feature: **Feature #[mã_issue] [tóm_tắt_nội_dung_code]**
2. Task: **Task #[mã_ticket_redmine] [tóm_tắt_nội_dung_code]**
3. Bug: **Bug #[mã_ticket_redmine] [tóm_tắt_nội_dung_code]**

## Workflow
**I. Khi có chức năng mới**
1. Pull code mới nhất từ master về
2. Leader checkout nhánh feature tổng từ master
  
**II. Khi có task/bug mới**
1. Pull code mới nhất từ nhánh tổng
2. Dev checkout nhánh task/bug từ nhánh feature tổng
3. Coding trên nhánh task/bug, tạo pull request đến nhánh feature tổng
4. Dev review chéo rồi approve, leader review, approve và **merge squash** vào nhánh tổng,<br>

**III. Khi release**
1. Leader **merge squash** nhánh feature tổng vào nhánh master,<br>
commit message "**Feature #[mã_issue] [tóm_tắt_nội_dung_code]**"

**Chú ý:**
- Không push trực tiếp code lên nhánh feature tổng
- Luôn squash để dễ dàng review commit message trong nhánh tổng và master
- Khi xảy ra conflict giữa nhánh tổng và nhánh master, leader sẽ là người fix conflict
- Bất kể dev JP hay VN, đều thực hiện theo mục **Workflow/II** khi thực hiện task

