# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ HỢP ĐỒNG

## 📋 TỔNG QUAN

Chức năng Quản lý Hợp đồng là một module quan trọng của hệ thống QLNS, cho phép Admin/HR quản lý các hợp đồng lao động của nhân viên, bao gồm thêm mới, chỉnh sửa, xóa hợp đồng và theo dõi trạng thái hợp đồng.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── QuanLyHopDongActivity.java           # Activity chính - Danh sách hợp đồng
├── ThemHopDongActivity.java             # Activity thêm/sửa hợp đồng  
├── HopDongAdapter.java                  # Adapter hiển thị danh sách
└── database/DatabaseHelper.java         # Xử lý database

app/src/main/res/layout/
├── activity_quan_ly_hop_dong.xml        # Layout danh sách hợp đồng
├── activity_them_hop_dong.xml           # Layout form thêm/sửa
└── item_hop_dong.xml                    # Layout item trong ListView

Lưu ý: Module này không có model class riêng, sử dụng trực tiếp Cursor
```

## 📊 NGHIỆP VỤ QUẢN LÝ HỢP ĐỒNG

### 1. Quy trình nghiệp vụ:
- **Xem danh sách**: Hiển thị tất cả hợp đồng với thông tin chi tiết
- **Tìm kiếm**: Tìm theo mã hợp đồng hoặc mã nhân viên
- **Thêm mới**: Tạo hợp đồng mới với các loại khác nhau
- **Cập nhật**: Chỉnh sửa thông tin hợp đồng hiện có
- **Xóa**: Xóa hợp đồng khỏi hệ thống (có xác nhận)
- **Gợi ý lương**: Tự động gợi ý mức lương theo chức vụ
- **Quản lý trạng thái**: Theo dõi trạng thái hợp đồng

### 2. Loại hợp đồng:
- **Thử việc**: Hợp đồng thử việc
- **Có thời hạn (1 năm)**: Hợp đồng có thời hạn 1 năm
- **Có thời hạn (3 năm)**: Hợp đồng có thời hạn 3 năm
- **Không thời hạn**: Hợp đồng không thời hạn

### 3. Phân quyền:
- **Admin**: Full quyền tất cả chức năng
- **HR**: Full quyền tất cả chức năng  
- **Manager**: Chỉ xem danh sách (nếu được cấp quyền)
- **Employee**: Không có quyền truy cập

---

## 📱 CHI TIẾT CÁC FILE
## 1️⃣ ACTIVITY CHÍNH - QuanLyHopDongActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/QuanLyHopDongActivity.java`

### Mục đích:
Activity chính quản lý danh sách hợp đồng, hiển thị thông tin, tìm kiếm và điều hướng đến các chức năng khác.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.content.Intent;                                             // Import Intent để chuyển Activity
import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.text.Editable;                                              // Import Editable để xử lý text thay đổi
import android.text.TextWatcher;                                           // Import TextWatcher để lắng nghe thay đổi text
import android.view.View;                                                  // Import View để xử lý UI components
import android.widget.EditText;                                            // Import EditText widget
import android.widget.ImageButton;                                         // Import ImageButton widget
import android.widget.ListView;                                            // Import ListView widget
import android.widget.TextView;                                            // Import TextView widget

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
```

#### Khai báo thuộc tính class:
```java
public class QuanLyHopDongActivity extends AppCompatActivity {             // Khai báo class kế thừa AppCompatActivity

    private ListView lvHopDong;                                            // ListView hiển thị danh sách hợp đồng
    private EditText etSearch;                                             // EditText cho chức năng tìm kiếm
    private ImageButton btnAdd;                                            // ImageButton thêm hợp đồng mới
    private TextView tvEmpty;                                              // TextView hiển thị khi không có dữ liệu
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private HopDongAdapter adapter;                                        // Adapter cho ListView
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_quan_ly_hop_dong);                // Set layout cho Activity

        initViews();                                                       // Khởi tạo các view components
        setupEvents();                                                     // Thiết lập các sự kiện
        loadData();                                                        // Tải dữ liệu hợp đồng
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        lvHopDong = findViewById(R.id.lv_hop_dong);                        // Ánh xạ ListView danh sách hợp đồng
        etSearch = findViewById(R.id.et_search_hop_dong);                  // Ánh xạ EditText tìm kiếm
        btnAdd = findViewById(R.id.btn_add_hop_dong);                      // Ánh xạ ImageButton thêm hợp đồng
        tvEmpty = findViewById(R.id.tv_empty);                             // Ánh xạ TextView hiển thị khi rỗng
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
    }
```

#### Method setupEvents:
```java
    private void setupEvents() {                                           // Method thiết lập các sự kiện cho UI components
        btnAdd.setOnClickListener(v -> {                                   // Set listener cho button thêm hợp đồng
            Intent intent = new Intent(QuanLyHopDongActivity.this, ThemHopDongActivity.class); // Tạo Intent chuyển đến ThemHopDongActivity
            startActivity(intent);                                         // Start Activity thêm hợp đồng
        });

        lvHopDong.setOnItemClickListener((parent, view, position, id) -> { // Set listener cho click item trong ListView
            Cursor cursor = (Cursor) adapter.getItem(position);            // Lấy Cursor tại vị trí được click
            String maHD = cursor.getString(cursor.getColumnIndexOrThrow("MaHopDong")); // Lấy mã hợp đồng từ Cursor
            Intent intent = new Intent(QuanLyHopDongActivity.this, ThemHopDongActivity.class); // Tạo Intent chuyển đến ThemHopDongActivity
            intent.putExtra("MaHopDong", maHD);                            // Truyền mã hợp đồng qua Intent để chỉnh sửa
            startActivity(intent);                                         // Start Activity chỉnh sửa hợp đồng
        });

        etSearch.addTextChangedListener(new TextWatcher() {                // Thêm listener lắng nghe thay đổi text tìm kiếm
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {} // Method được gọi trước khi text thay đổi (không sử dụng)

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {     // Method được gọi khi text đang thay đổi
                searchHopDong(s.toString());                               // Gọi method tìm kiếm với text hiện tại
            }

            @Override
            public void afterTextChanged(Editable s) {}                    // Method được gọi sau khi text thay đổi (không sử dụng)
        });
    }
```

#### Method loadData:
```java
    private void loadData() {                                              // Method tải dữ liệu hợp đồng từ database
        Cursor cursor = dbHelper.getAllHopDong();                          // Gọi method database để lấy tất cả hợp đồng
        if (cursor != null && cursor.getCount() > 0) {                     // Kiểm tra cursor không null và có dữ liệu
            adapter = new HopDongAdapter(this, cursor, this::confirmDelete); // Tạo adapter mới với context, cursor và callback xóa
            lvHopDong.setAdapter(adapter);                                 // Set adapter cho ListView
            tvEmpty.setVisibility(View.GONE);                              // Ẩn TextView thông báo rỗng
        } else {                                                           // Nếu không có dữ liệu
            tvEmpty.setVisibility(View.VISIBLE);                           // Hiển thị TextView thông báo rỗng
        }
    }
```

#### Method confirmDelete:
```java
    private void confirmDelete(String maHD) {                              // Method hiển thị dialog xác nhận xóa hợp đồng
        new androidx.appcompat.app.AlertDialog.Builder(this)               // Tạo AlertDialog builder
            .setTitle("Xác nhận xóa")                                      // Set tiêu đề dialog
            .setMessage("Bạn có chắc chắn muốn xóa hợp đồng " + maHD + " không?") // Set message xác nhận với mã hợp đồng
            .setPositiveButton("Xóa", (dialog, which) -> {                 // Set button "Xóa" với action
                if (dbHelper.deleteHopDong(maHD)) {                        // Gọi method database xóa hợp đồng
                    android.widget.Toast.makeText(this, "Đã xóa hợp đồng", android.widget.Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                    loadData();                                            // Tải lại dữ liệu sau khi xóa
                }
            })
            .setNegativeButton("Hủy", null)                                // Set button "Hủy" không có action
            .show();                                                       // Hiển thị dialog
    }
```

#### Method searchHopDong:
```java
    private void searchHopDong(String query) {                             // Method tìm kiếm hợp đồng theo từ khóa
        Cursor cursor = dbHelper.searchHopDong(query);                     // Gọi method database tìm kiếm với từ khóa
        if (adapter != null) {                                             // Kiểm tra adapter không null
            adapter.swapCursor(cursor);                                    // Thay đổi cursor của adapter với kết quả tìm kiếm
        }
    }
```

#### Method onResume:
```java
    @Override
    protected void onResume() {                                            // Method được gọi khi Activity quay lại foreground
        super.onResume();                                                  // Gọi method của class cha
        loadData();                                                        // Tải lại dữ liệu để cập nhật thay đổi mới nhất
    }
}
```

---

## 2️⃣ ACTIVITY THÊM/SỬA - ThemHopDongActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ThemHopDongActivity.java`

### Mục đích:
Activity xử lý thêm mới và chỉnh sửa thông tin hợp đồng, bao gồm chọn nhân viên, loại hợp đồng và mức lương.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.DatePickerDialog;                                       // Import DatePickerDialog để chọn ngày
import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.widget.ArrayAdapter;                                        // Import ArrayAdapter cho Spinner
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.SimpleCursorAdapter;                                 // Import SimpleCursorAdapter cho Spinner với Cursor
import android.widget.Spinner;                                             // Import Spinner widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database

import java.util.Calendar;                                                 // Import Calendar để xử lý ngày tháng
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
```

#### Khai báo thuộc tính class:
```java
public class ThemHopDongActivity extends AppCompatActivity {               // Khai báo class kế thừa AppCompatActivity

    private EditText etMaHD, etNgayBD, etNgayKT, etMucLuong;               // EditText cho mã hợp đồng, ngày bắt đầu, ngày kết thúc, mức lương
    private Spinner spNhanVien, spLoaiHD;                                  // Spinner chọn nhân viên và loại hợp đồng
    private Button btnSave, btnCancel;                                     // Button lưu và hủy
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private String editMaHD = null;                                        // Mã hợp đồng để chỉnh sửa (null nếu thêm mới)
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_them_hop_dong);                   // Set layout cho Activity

        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        editMaHD = getIntent().getStringExtra("MaHopDong");               // Lấy mã hợp đồng từ Intent (nếu có) để chỉnh sửa

        initViews();                                                       // Khởi tạo các view components
        setupSpinners();                                                   // Thiết lập các Spinner
        setupDatePickers();                                                // Thiết lập date picker cho ngày
        setupEvents();                                                     // Thiết lập các sự kiện

        if (editMaHD != null) {                                            // Nếu có mã hợp đồng (chế độ chỉnh sửa)
            loadEditData();                                                // Tải dữ liệu để chỉnh sửa
        }
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        etMaHD = findViewById(R.id.et_ma_hd);                              // Ánh xạ EditText mã hợp đồng
        etNgayBD = findViewById(R.id.et_ngay_bd);                          // Ánh xạ EditText ngày bắt đầu
        etNgayKT = findViewById(R.id.et_ngay_kt);                          // Ánh xạ EditText ngày kết thúc
        etMucLuong = findViewById(R.id.et_muc_luong);                      // Ánh xạ EditText mức lương
        spNhanVien = findViewById(R.id.sp_nhan_vien);                      // Ánh xạ Spinner chọn nhân viên
        spLoaiHD = findViewById(R.id.sp_loai_hd);                          // Ánh xạ Spinner chọn loại hợp đồng
        btnSave = findViewById(R.id.btn_save_hd);                          // Ánh xạ Button lưu
        btnCancel = findViewById(R.id.btn_cancel);                         // Ánh xạ Button hủy
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
    }
```

#### Method setupSpinners:
```java
    private void setupSpinners() {                                         // Method thiết lập các Spinner
        // Spinner Nhân viên
        Cursor cursor = dbHelper.getEmployeeListForSpinner();              // Gọi method database lấy danh sách nhân viên cho Spinner
        String[] from = {"DisplayName"};                                   // Tên cột để hiển thị trong Spinner
        int[] to = {android.R.id.text1};                                   // ID của TextView trong layout item
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(this, android.R.layout.simple_spinner_item, cursor, from, to, 0); // Tạo SimpleCursorAdapter
        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
        spNhanVien.setAdapter(adapter);                                    // Set adapter cho Spinner nhân viên

        // Lắng nghe sự kiện chọn nhân viên để gợi ý lương
        spNhanVien.setOnItemSelectedListener(new android.widget.AdapterView.OnItemSelectedListener() { // Set listener cho Spinner nhân viên
            @Override
            public void onItemSelected(android.widget.AdapterView<?> parent, android.view.View view, int position, long id) { // Method được gọi khi chọn item
                Cursor c = (Cursor) parent.getItemAtPosition(position);    // Lấy Cursor tại vị trí được chọn
                String maNV = c.getString(c.getColumnIndexOrThrow("_id")); // Lấy mã nhân viên từ Cursor
                double lươngChucVu = dbHelper.getSalaryByEmployee(maNV);   // Lấy mức lương theo chức vụ của nhân viên
                etMucLuong.setText(String.valueOf((long)lươngChucVu));     // Set mức lương gợi ý vào EditText (cast về long để bỏ phần thập phân)
            }

            @Override
            public void onNothingSelected(android.widget.AdapterView<?> parent) {} // Method được gọi khi không chọn gì (không sử dụng)
        });

        // Spinner Loại hợp đồng
        String[] loaiHDs = {"Thử việc", "Có thời hạn (1 năm)", "Có thời hạn (3 năm)", "Không thời hạn"}; // Mảng các loại hợp đồng
        ArrayAdapter<String> loaiAdapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, loaiHDs); // Tạo ArrayAdapter với mảng loại hợp đồng
        loaiAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
        spLoaiHD.setAdapter(loaiAdapter);                                  // Set adapter cho Spinner loại hợp đồng
    }
```

#### Method setupDatePickers:
```java
    private void setupDatePickers() {                                      // Method thiết lập date picker cho các EditText ngày
        etNgayBD.setOnClickListener(v -> showDatePicker(etNgayBD));        // Set listener cho EditText ngày bắt đầu
        etNgayKT.setOnClickListener(v -> showDatePicker(etNgayKT));        // Set listener cho EditText ngày kết thúc
    }
```

#### Method showDatePicker:
```java
    private void showDatePicker(EditText editText) {                       // Method hiển thị DatePickerDialog
        Calendar calendar = Calendar.getInstance();                        // Lấy instance Calendar hiện tại
        new DatePickerDialog(this, (view, year, month, dayOfMonth) -> {    // Tạo DatePickerDialog với callback
            String date = String.format(Locale.getDefault(), "%d-%02d-%02d", year, month + 1, dayOfMonth); // Format ngày thành yyyy-MM-dd
            editText.setText(date);                                        // Set ngày đã chọn vào EditText
        }, calendar.get(Calendar.YEAR), calendar.get(Calendar.MONTH), calendar.get(Calendar.DAY_OF_MONTH)).show(); // Hiển thị dialog với ngày hiện tại
    }
```
#### Method setupEvents:
```java
    private void setupEvents() {                                           // Method thiết lập các sự kiện cho button
        btnCancel.setOnClickListener(v -> finish());                      // Set listener cho button hủy, đóng Activity

        btnSave.setOnClickListener(v -> {                                  // Set listener cho button lưu
            String maHD = etMaHD.getText().toString().trim();              // Lấy mã hợp đồng từ EditText và trim khoảng trắng
            String ngayBD = etNgayBD.getText().toString().trim();          // Lấy ngày bắt đầu từ EditText và trim khoảng trắng
            String ngayKT = etNgayKT.getText().toString().trim();          // Lấy ngày kết thúc từ EditText và trim khoảng trắng
            String mucLuongStr = etMucLuong.getText().toString().trim();   // Lấy mức lương từ EditText và trim khoảng trắng

            if (maHD.isEmpty() || ngayBD.isEmpty() || mucLuongStr.isEmpty()) { // Kiểm tra các trường bắt buộc
                Toast.makeText(this, "Vui lòng nhập đầy đủ thông tin bắt buộc", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                return;                                                    // Thoát method
            }

            // Lấy mã nhân viên từ Spinner
            Cursor cursor = (Cursor) spNhanVien.getSelectedItem();         // Lấy Cursor của item được chọn trong Spinner
            String maNV = cursor.getString(cursor.getColumnIndexOrThrow("_id")); // Lấy mã nhân viên từ Cursor
            String loaiHD = spLoaiHD.getSelectedItem().toString();         // Lấy loại hợp đồng được chọn
            double mucLuong = Double.parseDouble(mucLuongStr);             // Parse mức lương từ string thành double

            boolean success;                                               // Biến lưu kết quả thành công/thất bại
            if (editMaHD != null) {                                        // Nếu là chế độ chỉnh sửa
                success = dbHelper.updateHopDong(maHD, maNV, loaiHD, ngayBD, ngayKT, mucLuong, "Hiệu lực"); // Gọi method update database
            } else {                                                       // Nếu là chế độ thêm mới
                success = dbHelper.insertHopDong(maHD, maNV, loaiHD, ngayBD, ngayKT, mucLuong); // Gọi method insert database
            }

            if (success) {                                                 // Nếu thành công
                Toast.makeText(this, editMaHD != null ? "Cập nhật thành công" : "Thêm hợp đồng thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                finish();                                                  // Đóng Activity
            } else {                                                       // Nếu thất bại
                Toast.makeText(this, "Thao tác thất bại", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });
    }
```

#### Method loadEditData:
```java
    private void loadEditData() {                                          // Method tải dữ liệu để chỉnh sửa hợp đồng
        TextView tvTitle = findViewById(android.R.id.content).getRootView().findViewById(R.id.tv_title); // Tìm TextView tiêu đề (có thể null nếu không có ID)
        // Let's just find by type or ignore title change for now, or add ID to title.
        
        etMaHD.setText(editMaHD);                                          // Set mã hợp đồng vào EditText
        etMaHD.setEnabled(false);                                          // Disable EditText mã hợp đồng (không cho sửa)
        btnSave.setText("CẬP NHẬT HỢP ĐỒNG");                              // Đổi text button thành "CẬP NHẬT HỢP ĐỒNG"

        Cursor cursor = dbHelper.getHopDongById(editMaHD);                 // Gọi method database lấy thông tin hợp đồng theo mã
        if (cursor != null && cursor.moveToFirst()) {                      // Kiểm tra cursor không null và có dữ liệu
            String maNV = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));     // Lấy mã nhân viên từ cursor
            String loaiHD = cursor.getString(cursor.getColumnIndexOrThrow("LoaiHopDong")); // Lấy loại hợp đồng từ cursor
            String ngayBD = cursor.getString(cursor.getColumnIndexOrThrow("NgayBatDau"));  // Lấy ngày bắt đầu từ cursor
            String ngayKT = cursor.getString(cursor.getColumnIndexOrThrow("NgayKetThuc")); // Lấy ngày kết thúc từ cursor
            double mucLuong = cursor.getDouble(cursor.getColumnIndexOrThrow("MucLuong"));  // Lấy mức lương từ cursor

            etNgayBD.setText(ngayBD);                                      // Set ngày bắt đầu vào EditText
            etNgayKT.setText(ngayKT);                                      // Set ngày kết thúc vào EditText
            etMucLuong.setText(String.valueOf((long)mucLuong));            // Set mức lương vào EditText (cast về long để bỏ phần thập phân)

            // Select Employee in Spinner
            for (int i = 0; i < spNhanVien.getCount(); i++) {              // Vòng lặp duyệt qua tất cả item trong Spinner nhân viên
                Cursor c = (Cursor) spNhanVien.getItemAtPosition(i);       // Lấy Cursor tại vị trí i
                if (c.getString(c.getColumnIndexOrThrow("_id")).equals(maNV)) { // Kiểm tra mã nhân viên có khớp không
                    spNhanVien.setSelection(i);                            // Set selection cho Spinner tại vị trí i
                    break;                                                 // Thoát vòng lặp
                }
            }

            // Select Contract Type
            for (int i = 0; i < spLoaiHD.getCount(); i++) {                // Vòng lặp duyệt qua tất cả item trong Spinner loại hợp đồng
                if (spLoaiHD.getItemAtPosition(i).toString().equals(loaiHD)) { // Kiểm tra loại hợp đồng có khớp không
                    spLoaiHD.setSelection(i);                              // Set selection cho Spinner tại vị trí i
                    break;                                                 // Thoát vòng lặp
                }
            }
            cursor.close();                                                // Đóng cursor để giải phóng bộ nhớ
        }
    }
}
```

---

## 3️⃣ ADAPTER - HopDongAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/HopDongAdapter.java`

### Mục đích:
Adapter quản lý hiển thị danh sách hợp đồng trong ListView, xử lý sự kiện xóa và định dạng tiền tệ.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.content.Context;                                            // Import Context để truy cập resources và services
import android.database.Cursor;                                            // Import Cursor để xử lý dữ liệu database
import android.view.LayoutInflater;                                        // Import LayoutInflater để inflate layout
import android.view.View;                                                  // Import View để xử lý UI components
import android.view.ViewGroup;                                             // Import ViewGroup để quản lý layout container
import android.widget.BaseAdapter;                                         // Import BaseAdapter làm base class
import android.widget.TextView;                                            // Import TextView widget

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database

import java.text.NumberFormat;                                             // Import NumberFormat để định dạng số
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
```

#### Khai báo thuộc tính class và interface:
```java
public class HopDongAdapter extends BaseAdapter {                          // Khai báo class kế thừa BaseAdapter
    private Context context;                                               // Context của Activity sử dụng adapter
    private Cursor cursor;                                                 // Cursor chứa dữ liệu hợp đồng
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private OnHopDongActionListener listener;                              // Listener cho các hành động trên hợp đồng

    public interface OnHopDongActionListener {                             // Interface định nghĩa callback cho hành động xóa
        void onDelete(String maHD);                                        // Method callback khi xóa hợp đồng
    }
```

#### Constructor:
```java
    public HopDongAdapter(Context context, Cursor cursor, OnHopDongActionListener listener) { // Constructor khởi tạo adapter
        this.context = context;                                            // Gán context
        this.cursor = cursor;                                              // Gán cursor chứa dữ liệu
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
        this.listener = listener;                                          // Gán listener cho callback
    }
```

#### Method getCount:
```java
    @Override
    public int getCount() {                                                // Method trả về số lượng item trong cursor
        return cursor != null ? cursor.getCount() : 0;                    // Trả về count của cursor, nếu null thì trả về 0
    }
```

#### Method getItem:
```java
    @Override
    public Object getItem(int position) {                                  // Method trả về item tại vị trí position
        if (cursor != null && cursor.moveToPosition(position)) {           // Kiểm tra cursor không null và di chuyển đến vị trí position
            return cursor;                                                 // Trả về cursor tại vị trí đó
        }
        return null;                                                       // Trả về null nếu không thể di chuyển đến vị trí
    }
```

#### Method getItemId:
```java
    @Override
    public long getItemId(int position) {                                  // Method trả về ID của item tại vị trí position
        return position;                                                   // Trả về chính position làm ID
    }
```

#### Method getView:
```java
    @Override
    public View getView(int position, View convertView, ViewGroup parent) { // Method tạo và trả về View cho item tại vị trí position
        if (convertView == null) {                                         // Kiểm tra convertView có null không (chưa được tạo)
            convertView = LayoutInflater.from(context).inflate(R.layout.item_hop_dong, parent, false); // Inflate layout item_hop_dong
        }

        if (cursor != null && cursor.moveToPosition(position)) {           // Kiểm tra cursor không null và di chuyển đến vị trí position
            TextView tvMaHD = convertView.findViewById(R.id.tv_ma_hd);      // Ánh xạ TextView mã hợp đồng
            TextView tvTenNV = convertView.findViewById(R.id.tv_ten_nv);    // Ánh xạ TextView tên nhân viên
            TextView tvLoaiHD = convertView.findViewById(R.id.tv_loai_hd);  // Ánh xạ TextView loại hợp đồng
            TextView tvThoiGian = convertView.findViewById(R.id.tv_thoi_gian); // Ánh xạ TextView thời gian hợp đồng
            TextView tvTrangThai = convertView.findViewById(R.id.tv_trang_thai); // Ánh xạ TextView trạng thái
            TextView tvMucLuong = convertView.findViewById(R.id.tv_muc_luong); // Ánh xạ TextView mức lương
            android.widget.ImageButton btnDelete = convertView.findViewById(R.id.btn_delete_hd); // Ánh xạ ImageButton xóa

            String maHD = cursor.getString(cursor.getColumnIndexOrThrow("MaHopDong"));         // Lấy mã hợp đồng từ cursor
            String maNV = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));        // Lấy mã nhân viên từ cursor
            String loaiHD = cursor.getString(cursor.getColumnIndexOrThrow("LoaiHopDong"));     // Lấy loại hợp đồng từ cursor
            String ngayBD = cursor.getString(cursor.getColumnIndexOrThrow("NgayBatDau"));      // Lấy ngày bắt đầu từ cursor
            String ngayKT = cursor.getString(cursor.getColumnIndexOrThrow("NgayKetThuc"));     // Lấy ngày kết thúc từ cursor
            double mucLuong = cursor.getDouble(cursor.getColumnIndexOrThrow("MucLuong"));      // Lấy mức lương từ cursor
            String trangThai = cursor.getString(cursor.getColumnIndexOrThrow("TrangThai"));    // Lấy trạng thái từ cursor

            btnDelete.setOnClickListener(v -> {                            // Set listener cho button xóa
                if (listener != null) {                                    // Kiểm tra listener không null
                    listener.onDelete(maHD);                               // Gọi callback onDelete với mã hợp đồng
                }
            });

            // Lấy tên nhân viên
            String tenNV = dbHelper.getEmployeeNameByMa(maNV);             // Gọi method database lấy tên nhân viên từ mã

            tvMaHD.setText("Mã HD: " + maHD);                              // Set text hiển thị mã hợp đồng
            tvTenNV.setText("Nhân viên: " + (tenNV != null ? tenNV : maNV)); // Set text hiển thị tên nhân viên (hoặc mã nếu không có tên)
            tvLoaiHD.setText("Loại: " + loaiHD);                           // Set text hiển thị loại hợp đồng
            
            String thoiGian = "Từ " + ngayBD + (ngayKT != null && !ngayKT.isEmpty() ? " đến " + ngayKT : " - Không thời hạn"); // Tạo text thời gian hợp đồng
            tvThoiGian.setText(thoiGian);                                  // Set text hiển thị thời gian hợp đồng
            
            tvTrangThai.setText(trangThai);                                // Set text hiển thị trạng thái
            
            NumberFormat formatter = NumberFormat.getCurrencyInstance(new Locale("vi", "VN")); // Tạo formatter tiền tệ Việt Nam
            tvMucLuong.setText("Mức lương: " + formatter.format(mucLuong)); // Set text hiển thị mức lương với định dạng VNĐ
        }

        return convertView;                                                // Trả về View đã được setup
    }
```

#### Method swapCursor:
```java
    public void swapCursor(Cursor newCursor) {                             // Method thay đổi cursor của adapter
        if (cursor != null) cursor.close();                               // Đóng cursor cũ nếu không null
        cursor = newCursor;                                                // Gán cursor mới
        notifyDataSetChanged();                                            // Thông báo adapter dữ liệu đã thay đổi để refresh UI
    }
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 4️⃣ LAYOUT DANH SÁCH - activity_quan_ly_hop_dong.xml

**Đường dẫn**: `app/src/main/res/layout/activity_quan_ly_hop_dong.xml`

### Mục đích:
Layout chính cho Activity quản lý danh sách hợp đồng, bao gồm header gradient, tìm kiếm và ListView.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Khai báo LinearLayout root với namespace Android -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Khai báo namespace app cho custom attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent (full height) -->
    android:orientation="vertical"                                         <!-- Orientation dọc (các child xếp từ trên xuống) -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt (#f5f5f5) -->
```

#### Header Gradient Section:
```xml
    <!-- Header Gradient -->
    <LinearLayout                                                          <!-- LinearLayout chứa phần header với gradient -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="180dp"                                      <!-- Chiều cao cố định 180dp -->
        android:background="@drawable/header_gradient"                     <!-- Background gradient custom từ drawable -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:padding="20dp">                                            <!-- Padding 20dp cho tất cả các cạnh -->

        <LinearLayout                                                      <!-- LinearLayout chứa tiêu đề và button thêm -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:gravity="center_vertical">                             <!-- Căn giữa theo chiều dọc -->

            <TextView                                                      <!-- TextView hiển thị tiêu đề -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:layout_weight="1"                                  <!-- Weight 1 để chiếm không gian còn lại -->
                android:text="QUẢN LÝ HỢP ĐỒNG"                            <!-- Text tiêu đề hiển thị -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textSize="22sp"                                    <!-- Kích thước font 22sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:gravity="center"                                   <!-- Căn giữa text -->
                android:layout_marginStart="40dp" />                       <!-- Margin start 40dp -->

            <ImageButton                                                   <!-- ImageButton thêm hợp đồng mới -->
                android:id="@+id/btn_add_hop_dong"                         <!-- ID để truy cập từ Java code -->
                android:layout_width="40dp"                                <!-- Chiều rộng cố định 40dp -->
                android:layout_height="40dp"                               <!-- Chiều cao cố định 40dp -->
                android:background="?attr/selectableItemBackgroundBorderless" <!-- Background với ripple effect -->
                android:src="@android:drawable/ic_input_add"               <!-- Icon thêm từ Android system -->
                app:tint="@color/white" />                                 <!-- Tint màu trắng cho icon -->
        </LinearLayout>

        <EditText                                                          <!-- EditText cho chức năng tìm kiếm -->
            android:id="@+id/et_search_hop_dong"                           <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="50dp"                                   <!-- Chiều cao cố định 50dp -->
            android:layout_marginTop="30dp"                                <!-- Margin top 30dp -->
            android:background="@drawable/search_background"               <!-- Background custom từ drawable -->
            android:drawableStart="@android:drawable/ic_menu_search"       <!-- Icon search ở đầu (bên trái) -->
            android:drawablePadding="10dp"                                 <!-- Padding giữa icon và text 10dp -->
            android:hint="Tìm theo mã nhân viên hoặc mã hợp đồng..."       <!-- Hint text hiển thị khi rỗng -->
            android:padding="10dp"                                         <!-- Padding 10dp cho tất cả các cạnh -->
            android:textSize="14sp" />                                     <!-- Kích thước font 14sp -->
    </LinearLayout>
```

#### CardView Container Section:
```xml
    <androidx.cardview.widget.CardView                                     <!-- CardView chứa ListView với bo góc -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="match_parent"                               <!-- Chiều cao bằng parent -->
        android:layout_marginTop="-20dp"                                   <!-- Margin top âm để overlap với header -->
        app:cardCornerRadius="25dp"                                        <!-- Bo góc 25dp -->
        app:cardElevation="5dp">                                           <!-- Elevation 5dp tạo shadow -->

        <ListView                                                          <!-- ListView hiển thị danh sách hợp đồng -->
            android:id="@+id/lv_hop_dong"                                  <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="match_parent"                           <!-- Chiều cao bằng parent -->
            android:divider="@android:color/transparent"                   <!-- Divider trong suốt -->
            android:dividerHeight="10dp"                                   <!-- Chiều cao divider 10dp -->
            android:padding="16dp"                                         <!-- Padding 16dp cho tất cả các cạnh -->
            android:clipToPadding="false" />                               <!-- Không clip content theo padding -->
            
        <TextView                                                          <!-- TextView hiển thị khi không có dữ liệu -->
            android:id="@+id/tv_empty"                                     <!-- ID để truy cập từ Java code -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Chưa có hợp đồng nào"                            <!-- Text hiển thị khi rỗng -->
            android:layout_gravity="center"                                <!-- Căn giữa trong CardView -->
            android:visibility="gone"/>                                    <!-- Visibility gone (ẩn mặc định) -->
    </androidx.cardview.widget.CardView>

</LinearLayout>
```

---

## 5️⃣ LAYOUT THÊM/SỬA - activity_them_hop_dong.xml

**Đường dẫn**: `app/src/main/res/layout/activity_them_hop_dong.xml`

### Mục đích:
Layout form thêm mới và chỉnh sửa hợp đồng với các trường nhập liệu và validation.

### Chi tiết code:

#### Khai báo XML và ScrollView chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"    <!-- ScrollView root để có thể cuộn khi nội dung dài -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace app cho custom attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt -->

    <LinearLayout                                                          <!-- LinearLayout chính chứa tất cả form elements -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:padding="20dp">                                            <!-- Padding 20dp cho tất cả các cạnh -->
```

#### Header Title Section:
```xml
        <TextView                                                          <!-- TextView tiêu đề form -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="THÊM HỢP ĐỒNG MỚI"                               <!-- Text tiêu đề -->
            android:textSize="22sp"                                        <!-- Kích thước font 22sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="30dp" />                          <!-- Margin bottom 30dp -->
```

#### Mã Hợp Đồng Section:
```xml
        <TextView                                                          <!-- Label cho trường mã hợp đồng -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Mã hợp đồng"                                     <!-- Text label -->
            android:textStyle="bold" />                                    <!-- Style chữ đậm -->
        <EditText                                                          <!-- EditText nhập mã hợp đồng -->
            android:id="@+id/et_ma_hd"                                     <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="50dp"                                   <!-- Chiều cao cố định 50dp -->
            android:hint="Nhập mã hợp đồng (ví dụ: HD001)"                 <!-- Hint text hướng dẫn -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:padding="10dp"                                         <!-- Padding 10dp cho tất cả các cạnh -->
            android:layout_marginBottom="15dp" />                          <!-- Margin bottom 15dp -->
```

#### Chọn Nhân Viên Section:
```xml
        <TextView                                                          <!-- Label cho trường chọn nhân viên -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Chọn nhân viên"                                  <!-- Text label -->
            android:textStyle="bold" />                                    <!-- Style chữ đậm -->
        <Spinner                                                           <!-- Spinner chọn nhân viên -->
            android:id="@+id/sp_nhan_vien"                                 <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="50dp"                                   <!-- Chiều cao cố định 50dp -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:layout_marginBottom="15dp" />                          <!-- Margin bottom 15dp -->
```

#### Loại Hợp Đồng Section:
```xml
        <TextView                                                          <!-- Label cho trường loại hợp đồng -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Loại hợp đồng"                                   <!-- Text label -->
            android:textStyle="bold" />                                    <!-- Style chữ đậm -->
        <Spinner                                                           <!-- Spinner chọn loại hợp đồng -->
            android:id="@+id/sp_loai_hd"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="50dp"                                   <!-- Chiều cao cố định 50dp -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:layout_marginBottom="15dp" />                          <!-- Margin bottom 15dp -->
```

#### Ngày Bắt Đầu Section:
```xml
        <TextView                                                          <!-- Label cho trường ngày bắt đầu -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Ngày bắt đầu"                                    <!-- Text label -->
            android:textStyle="bold" />                                    <!-- Style chữ đậm -->
        <EditText                                                          <!-- EditText chọn ngày bắt đầu -->
            android:id="@+id/et_ngay_bd"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="50dp"                                   <!-- Chiều cao cố định 50dp -->
            android:hint="Chọn ngày bắt đầu"                               <!-- Hint text hướng dẫn -->
            android:focusable="false"                                      <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:padding="10dp"                                         <!-- Padding 10dp cho tất cả các cạnh -->
            android:layout_marginBottom="15dp" />                          <!-- Margin bottom 15dp -->
```

#### Ngày Kết Thúc Section:
```xml
        <TextView                                                          <!-- Label cho trường ngày kết thúc -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Ngày kết thúc (nếu có)"                          <!-- Text label với ghi chú tùy chọn -->
            android:textStyle="bold" />                                    <!-- Style chữ đậm -->
        <EditText                                                          <!-- EditText chọn ngày kết thúc -->
            android:id="@+id/et_ngay_kt"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="50dp"                                   <!-- Chiều cao cố định 50dp -->
            android:hint="Chọn ngày kết thúc"                              <!-- Hint text hướng dẫn -->
            android:focusable="false"                                      <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:padding="10dp"                                         <!-- Padding 10dp cho tất cả các cạnh -->
            android:layout_marginBottom="15dp" />                          <!-- Margin bottom 15dp -->
```

#### Mức Lương Section:
```xml
        <TextView                                                          <!-- Label cho trường mức lương -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Mức lương thỏa thuận"                            <!-- Text label -->
            android:textStyle="bold" />                                    <!-- Style chữ đậm -->
        <EditText                                                          <!-- EditText nhập mức lương -->
            android:id="@+id/et_muc_luong"                                 <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="50dp"                                   <!-- Chiều cao cố định 50dp -->
            android:hint="Nhập mức lương (ví dụ: 10000000)"                <!-- Hint text hướng dẫn với ví dụ -->
            android:inputType="number"                                     <!-- Input type chỉ cho phép nhập số -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:padding="10dp"                                         <!-- Padding 10dp cho tất cả các cạnh -->
            android:layout_marginBottom="30dp" />                          <!-- Margin bottom 30dp -->
```

#### Action Buttons Section:
```xml
        <Button                                                            <!-- Button lưu hợp đồng -->
            android:id="@+id/btn_save_hd"                                  <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="55dp"                                   <!-- Chiều cao cố định 55dp -->
            android:text="LƯU HỢP ĐỒNG"                                    <!-- Text button -->
            android:backgroundTint="#2196F3"                               <!-- Màu background xanh dương -->
            android:textColor="@color/white"                               <!-- Màu chữ trắng -->
            android:textStyle="bold" />                                    <!-- Style chữ đậm -->

        <Button                                                            <!-- Button hủy -->
            android:id="@+id/btn_cancel"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="55dp"                                   <!-- Chiều cao cố định 55dp -->
            android:text="HỦY"                                             <!-- Text button -->
            android:backgroundTint="#f44336"                               <!-- Màu background đỏ -->
            android:textColor="@color/white"                               <!-- Màu chữ trắng -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:layout_marginTop="10dp" />                             <!-- Margin top 10dp -->

    </LinearLayout>
</ScrollView>
```

---

## 6️⃣ LAYOUT ITEM - item_hop_dong.xml

**Đường dẫn**: `app/src/main/res/layout/item_hop_dong.xml`

### Mục đích:
Layout cho từng item hợp đồng trong ListView, hiển thị thông tin chi tiết và button xóa.

### Chi tiết code:

#### Khai báo XML và CardView chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android" <!-- CardView root với namespace Android -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace app cho custom attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ chứa content -->
    app:cardCornerRadius="12dp"                                            <!-- Bo góc 12dp -->
    app:cardElevation="3dp"                                                <!-- Elevation 3dp tạo shadow -->
    android:layout_margin="4dp"                                            <!-- Margin 4dp cho tất cả các cạnh -->
    android:descendantFocusability="blocksDescendants">                    <!-- Block focus của các child để ListView item có thể click -->

    <LinearLayout                                                          <!-- LinearLayout chính chứa tất cả content -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:padding="16dp">                                            <!-- Padding 16dp cho tất cả các cạnh -->
```

#### Header Row (Mã HĐ + Trạng Thái):
```xml
        <LinearLayout                                                      <!-- LinearLayout chứa mã hợp đồng và trạng thái -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:gravity="center_vertical"                              <!-- Căn giữa theo chiều dọc -->
            android:layout_marginBottom="8dp">                             <!-- Margin bottom 8dp -->

            <TextView                                                      <!-- TextView hiển thị mã hợp đồng -->
                android:id="@+id/tv_ma_hd"                                 <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:layout_weight="1"                                  <!-- Weight 1 để chiếm không gian còn lại -->
                android:text="Mã HD: HD001"                                <!-- Text mẫu hiển thị -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#2196F3"                                <!-- Màu chữ xanh dương -->
                android:textSize="16sp" />                                 <!-- Kích thước font 16sp -->

            <TextView                                                      <!-- TextView hiển thị trạng thái hợp đồng -->
                android:id="@+id/tv_trang_thai"                            <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Hiệu lực"                                    <!-- Text mẫu hiển thị -->
                android:background="@drawable/status_background"           <!-- Background custom từ drawable -->
                android:paddingHorizontal="8dp"                            <!-- Padding ngang 8dp -->
                android:paddingVertical="2dp"                              <!-- Padding dọc 2dp -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textSize="12sp" />                                 <!-- Kích thước font 12sp -->
        </LinearLayout>
```

#### Employee Information Section:
```xml
        <TextView                                                          <!-- TextView hiển thị tên nhân viên -->
            android:id="@+id/tv_ten_nv"                                    <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Nhân viên: Nguyễn Văn A"                         <!-- Text mẫu hiển thị -->
            android:textColor="#333"                                       <!-- Màu chữ xám đậm -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->
```

#### Contract Type Section:
```xml
        <TextView                                                          <!-- TextView hiển thị loại hợp đồng -->
            android:id="@+id/tv_loai_hd"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Loại: Không thời hạn"                            <!-- Text mẫu hiển thị -->
            android:textColor="#666" />                                    <!-- Màu chữ xám -->
```

#### Contract Duration Section:
```xml
        <TextView                                                          <!-- TextView hiển thị thời gian hợp đồng -->
            android:id="@+id/tv_thoi_gian"                                 <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Từ 01/01/2024 - Không xác định"                  <!-- Text mẫu hiển thị -->
            android:textColor="#666" />                                    <!-- Màu chữ xám -->
```

#### Divider Section:
```xml
        <View                                                              <!-- View làm đường phân cách -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="1dp"                                    <!-- Chiều cao 1dp -->
            android:background="#eee"                                      <!-- Background màu xám nhạt -->
            android:layout_marginVertical="8dp" />                         <!-- Margin dọc 8dp -->
```

#### Footer Row (Lương + Button Xóa):
```xml
        <LinearLayout                                                      <!-- LinearLayout chứa mức lương và button xóa -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:gravity="bottom">                                      <!-- Căn dưới -->

            <TextView                                                      <!-- TextView hiển thị mức lương -->
                android:id="@+id/tv_muc_luong"                             <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:layout_weight="1"                                  <!-- Weight 1 để chiếm không gian còn lại -->
                android:text="Mức lương: 10,000,000 VNĐ"                   <!-- Text mẫu hiển thị -->
                android:textColor="#E91E63"                                <!-- Màu chữ hồng -->
                android:textStyle="bold" />                                <!-- Style chữ đậm -->

            <ImageButton                                                   <!-- ImageButton xóa hợp đồng -->
                android:id="@+id/btn_delete_hd"                            <!-- ID để truy cập từ Java code -->
                android:layout_width="35dp"                                <!-- Chiều rộng cố định 35dp -->
                android:layout_height="35dp"                               <!-- Chiều cao cố định 35dp -->
                android:background="@drawable/btn_danger"                  <!-- Background custom màu đỏ từ drawable -->
                android:src="@android:drawable/ic_menu_delete"             <!-- Icon xóa từ Android system -->
                app:tint="@android:color/white"                            <!-- Tint màu trắng cho icon -->
                android:padding="8dp"                                      <!-- Padding 8dp cho tất cả các cạnh -->
                android:focusable="false"                                  <!-- Không cho phép focus -->
                android:focusableInTouchMode="false" />                    <!-- Không cho phép focus trong touch mode -->
        </LinearLayout>

    </LinearLayout>
</androidx.cardview.widget.CardView>
```

---

## 🔗 TÍCH HỢP VÀ ĐIỀU HƯỚNG

### 1. Tích hợp với Dashboard:
- **Quyền truy cập**: Admin/HR có full quyền, Manager có thể xem (tùy cấu hình)
- **Button điều hướng**: "QUẢN LÝ HỢP ĐỒNG" trong DashboardActivity
- **Intent navigation**: Sử dụng Intent để chuyển giữa các Activity

### 2. Tích hợp với Database:
- **DatabaseHelper methods**: getAllHopDong(), searchHopDong(), insertHopDong(), updateHopDong(), deleteHopDong()
- **Employee integration**: getEmployeeListForSpinner(), getEmployeeNameByMa(), getSalaryByEmployee()
- **Data validation**: Kiểm tra trùng lặp mã hợp đồng, validation ngày tháng

### 3. Navigation Flow:
```
DashboardActivity → QuanLyHopDongActivity → ThemHopDongActivity
                                        ↓
                                   HopDongAdapter (ListView items)
```

---

## 📋 CÁC PHƯƠNG THỨC DATABASE LIÊN QUAN

### 1. Contract Management Methods:
- `getAllHopDong()`: Lấy tất cả hợp đồng với JOIN employee và position
- `searchHopDong(String query)`: Tìm kiếm theo mã hợp đồng hoặc mã nhân viên
- `insertHopDong()`: Thêm hợp đồng mới với validation
- `updateHopDong()`: Cập nhật thông tin hợp đồng
- `deleteHopDong()`: Xóa hợp đồng khỏi database
- `getHopDongById()`: Lấy thông tin hợp đồng theo mã để chỉnh sửa

### 2. Employee Integration Methods:
- `getEmployeeListForSpinner()`: Lấy danh sách nhân viên cho Spinner
- `getEmployeeNameByMa()`: Lấy tên nhân viên từ mã
- `getSalaryByEmployee()`: Lấy mức lương cơ bản theo chức vụ để gợi ý

---

## 🎯 QUY TẮC NGHIỆP VỤ

### 1. Validation Rules:
- **Mã hợp đồng**: Bắt buộc, không trùng lặp
- **Nhân viên**: Bắt buộc chọn từ danh sách có sẵn
- **Ngày bắt đầu**: Bắt buộc, format yyyy-MM-dd
- **Ngày kết thúc**: Tùy chọn, nếu có phải sau ngày bắt đầu
- **Mức lương**: Bắt buộc, phải là số dương

### 2. Business Logic:
- **Auto-suggest salary**: Tự động gợi ý mức lương theo chức vụ khi chọn nhân viên
- **Contract types**: 4 loại hợp đồng cố định
- **Status management**: Trạng thái mặc định "Hiệu lực"
- **Currency formatting**: Hiển thị lương theo định dạng VNĐ

### 3. User Experience:
- **Real-time search**: Tìm kiếm ngay khi gõ
- **Date picker**: Chọn ngày bằng DatePickerDialog
- **Confirmation dialogs**: Xác nhận trước khi xóa
- **Toast notifications**: Thông báo kết quả thao tác

---

## 🎨 THIẾT KẾ UI/UX

### 1. Design Principles:
- **Material Design**: Sử dụng CardView, elevation, bo góc
- **Color Scheme**: Xanh dương (#2196F3) chủ đạo, đỏ (#f44336) cho xóa
- **Typography**: Bold cho tiêu đề, regular cho nội dung
- **Spacing**: Consistent padding và margin

### 2. Responsive Layout:
- **ScrollView**: Đảm bảo form có thể cuộn trên màn hình nhỏ
- **Match parent width**: Tất cả elements chiếm full width
- **Flexible height**: Wrap content để tự động điều chỉnh

### 3. Interactive Elements:
- **Ripple effects**: Sử dụng selectableItemBackground
- **Visual feedback**: Toast messages, color changes
- **Loading states**: Disable buttons khi đang xử lý

---

## 🔧 KẾT LUẬN

Module Quản lý Hợp đồng cung cấp một giải pháp hoàn chỉnh cho việc quản lý hợp đồng lao động trong hệ thống QLNS. Với thiết kế modular, UI thân thiện và logic nghiệp vụ rõ ràng, module này đảm bảo:

- **Tính nhất quán**: Tuân thủ design pattern chung của ứng dụng
- **Hiệu suất**: Sử dụng Cursor và adapter pattern hiệu quả
- **Bảo mật**: Phân quyền rõ ràng theo vai trò người dùng
- **Khả năng mở rộng**: Dễ dàng thêm tính năng mới
- **Trải nghiệm người dùng**: Interface trực quan, dễ sử dụng

Module này tích hợp seamlessly với các module khác trong hệ thống và cung cấp foundation vững chắc cho việc quản lý hợp đồng lao động trong doanh nghiệp.
```