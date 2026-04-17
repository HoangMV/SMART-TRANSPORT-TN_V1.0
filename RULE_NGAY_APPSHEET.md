# QUY TẮC XỬ LÝ NGÀY TỪ APPSHEET

## 1. Nguyên tắc cơ bản

AppSheet field type **Date** luôn trả về dạng **`MM/DD/YYYY`**:
```
04/15/2026 → tháng 04, ngày 15, năm 2026
```

---

## 2. Quy tắc hiển thị (chuẩn VN)

| Thành phần | Quy tắc | Ví dụ |
|---|---|---|
| Ngày | Luôn 2 chữ số | `01`, `02`, `15`, `31` |
| Tháng 1, 2 | Có số 0 | `01`, `02` |
| Tháng 3-12 | Không có số 0 | `3`, `4`, `10`, `12` |
| Năm | Đủ 4 chữ số | `2026` |

Kết quả: `15/4/2026`, `01/01/2026`, `31/12/2026`

---

## 3. Hàm dùng chung — copy vào mỗi trang

```javascript
// Parse MM/DD/YYYY từ AppSheet Date field
function parseMDY(str) {
    const p = String(str).trim().match(/^(\d{1,2})\/(\d{1,2})\/(\d{4})(?:\s.*)?$/);
    if (!p) return null;
    const month = Number(p[1]), day = Number(p[2]), year = Number(p[3]);
    const d = new Date(year, month - 1, day);
    return (d.getFullYear() === year && d.getMonth() === month - 1 && d.getDate() === day) ? d : null;
}

// Format sang DD/MM/YYYY chuẩn VN
function formatDate(dateValue) {
    if (!dateValue) return '';
    const date = parseMDY(dateValue);
    if (!date) return '';
    const day = String(date.getDate()).padStart(2, '0');
    const m = date.getMonth() + 1;
    const month = (m <= 2) ? String(m).padStart(2, '0') : String(m);
    return `${day}/${month}/${date.getFullYear()}`;
}
```

---

## 4. Cách dùng

```javascript
// Tất cả field Date từ AppSheet đều qua formatDate()
document.getElementById('ngay-tu').textContent  = formatDate(data.NgayCap);
document.getElementById('ngay-den').textContent = formatDate(data.NgayHetHan);
document.getElementById('ngay-cap').textContent = formatDate(data.NgayCap_SoGPLienVanQT);
```

---

## 5. KHÔNG làm

```javascript
// SAI - hiển thị thẳng → ra MM/DD sai
element.textContent = data.NgayCap;

// SAI - phụ thuộc trình duyệt
element.textContent = new Date(data.NgayCap).toLocaleDateString();

// KHÔNG CẦN - xử lý trong code là đủ
// Không cần tạo virtual field _DinhDang trong AppSheet
```

---

## 6. Lưu ý

- Field có thêm **thời gian** (vd: `"04/17/2026 16:07:54"`) vẫn parse được vì regex có `(?:\s.*)?` bỏ qua phần giờ.
- Nếu field trống hoặc null → `formatDate()` trả về chuỗi rỗng `''`, không báo lỗi.
