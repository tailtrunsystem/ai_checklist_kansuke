# Kansuke AI QA Checklist Generator 🚀

Hệ thống tự động hóa tạo Checklist kiểm thử (Markdown & Excel) dành cho đội ngũ QC dự án Kansuke, sử dụng sức mạnh của AI (Cursor / Antigravity) kết hợp với phân tích mã nguồn thời gian thực.

---

## 🛠 1. Yêu cầu hệ thống (Prerequisites)

Để sử dụng công cụ này, bạn cần:
1.  **Cursor IDE**: Trình soạn thảo mã nguồn tích hợp AI.
2.  **Python 3**: Cài đặt sẵn trên máy.
3.  **Thư viện Python**:
    ```bash
    pip install openpyxl
    ```
4.  **Cấu trúc dự án**: Đảm bảo folder `.agents/` có đầy đủ `rules`, `workflows` và `CONTEXT.md`.

---

## 🚀 2. Cách sử dụng (Usage)

### Bước 1: Kích hoạt Workflow
Tại thanh chat của Cursor (Cmd+L hoặc Cmd+K), gõ lệnh sau để kích hoạt quy trình:

```text
/gen-checklist task_id=[ID_TASK], cr="[NỘI DUNG YÊU CẦU]", pvah="[PHÂN TÍCH KỸ THUẬT - OPTIONAL]"
```

**Ví dụ thực tế:**
*   **Cách 1 (Text):** 
    ```text
    /gen-checklist task_id=CANSUKE-172, cr="Lỗi vị trí focus..."
    ```
*   **Cách 2 (File - Khuyên dùng):** Nếu bạn có file `.md` mô tả chi tiết CR, hãy kéo file vào khung chat:
    ```text
    /gen-checklist task_id=CANSUKE-181, cr=@CR_Description_Task.md
    ```

### Bước 2: AI thực hiện "Deep Read" & Architecture Audit
Sau khi nhận lệnh, AI sẽ:
1.  **Requirement Extraction**: Nếu bạn dùng `@file`, AI sẽ đọc toàn bộ nội dung file để bóc tách mục tiêu, hành vi UI và các case lỗi được nhắc đến.
2.  **Architecture Audit**: Quét mã nguồn để tìm ra các rủi ro kỹ thuật ngầm (Shared Core, KINO flags...).
3.  **Generate Files**: Tạo Folder, file `.md` review và script `.xlsx` chuẩn Nhật.

---

## 📁 3. Cấu trúc đầu ra (Output Structure)

Để tránh lộn xộn, hệ thống sẽ tự động gom các file vào folder theo Task ID:

```text
/Users/tailt/AI gen checklist/
├── [TASK_ID]/                       # Folder tự động tạo theo mã Task
│   ├── KANSUKE_Checklist_[ID].md    # Bản review nội bộ (Tiếng Việt)
│   └── KANSUKE_Checklist_[ID].xlsx  # Bản gửi khách hàng (Tiếng Nhật, có Dropdown)
├── .agents/                         # "Bộ não" của hệ thống (Rules, Context, Workflows)
└── gen_xlsx.py                      # Script generator dùng chung
```

---

## 💡 4. Lưu ý quan trọng cho QC

*   **Dữ liệu đầu vào**: Nội dung CR (Change Request) càng chi tiết thì AI phân tích PVAH càng chính xác.
*   **PVAH (Optional)**: Nếu Dev đã có phân tích kỹ thuật, hãy đưa vào tham số `pvah` để AI double-check. Nếu không, AI sẽ tự độc lập quét source code để đưa ra quan điểm kiểm thử.
*   **Error Codes**: AI đã được lập trình để quét folder `assets/` và `lib/core/` nhằm trích xuất chính xác Error Message Nhật cho các case 異常系 (Negative cases).
*   **Template Excel**: File Excel tạo ra đã bao gồm sẵn các Dropdown chọn `合/否`, `Lv1~Lv4` và định dạng `ー` theo đúng tiêu chuẩn khách hàng Nhật.

---

## 🏗 5. Cấu trúc bộ não AI (.agents/)

Nếu bạn muốn điều chỉnh logic phân tích của AI, hãy chú ý các file sau:
*   `rules/kansuke-impact-auditor.md`: Quy định cách AI soi lỗi và ảnh hưởng chéo giữa 3 App.
*   `workflows/gen-checklist.md`: Định nghĩa quy trình tạo file và cấu trúc folder.
*   `CONTEXT.md`: Chứa kiến thức chuyên sâu về logic code (BLoC, BlackBoard, Lifecycle) để AI "hiểu" dự án như một member thực thụ.

---
**RS.HCM QA Team**
