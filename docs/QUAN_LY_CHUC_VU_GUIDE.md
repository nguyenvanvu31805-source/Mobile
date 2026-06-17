# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ CHỨC VỤ

## 📋 TỔNG QUAN

Chức năng Quản lý Chức vụ là một module quan trọng của hệ thống QLNS, cho phép Admin/HR quản lý các chức vụ trong tổ chức, bao gồm thêm mới, chỉnh sửa, xóa chức vụ và thiết lập mức lương cơ bản cho từng vị trí.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── QuanLyChucVuActivity.java            # Activity chính - Danh sách chức vụ
├── ThemChucVuActivity.java              # Activity thêm/sửa chức vụ  
├── ChucVuAdapter.java                   # Adapter hiển thị danh sách
├── models/ChucVu.java                   # Model class chức vụ
└── database/DatabaseHelper.java         # Xử lý database

app/src/main/res/layout/
├── activity_quan_ly_chuc_vu.xml         # Layout danh sách chức vụ
├── activity_them_chuc_vu.xml            # Layout form thêm/sửa
└── item_chuc_vu.xml                     # Layout item trong ListView
```

## 📊 NGHIỆP VỤ QUẢN LÝ CHỨC VỤ

### 1. Quy trình nghiệp vụ:
- **Xem danh sách**: Hiển thị tất cả chức vụ với thông tin chi tiết
- **Tìm kiếm**: Tìm theo tên hoặc mã chức vụ
- **Thêm mới**: Tạo chức vụ với mã tự động tăng (CV001, CV002...)
- **Cập nhật**: Chỉnh sửa thông tin chức vụ, mức lương cơ bản
- **Xóa**: Soft delete chức vụ (đánh dấu ngừng hoạt động)
- **Ràng buộc**: Không cho xóa chức vụ còn nhân viên đảm nhiệm
- **Định dạng tiền tệ**: Hiển thị lương theo định dạng VNĐ

### 2. Phân quyền:
- **Admin**: Full quyền tất cả chức năng
- **HR**: Full quyền tất cả chức năng  
- **Manager**: Chỉ xem danh sách
- **Employee**: Không có quyền truy cập

---

## 📱 CHI TIẾT CÁC FILE
## 1️⃣ MODEL CLASS - ChucVu.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/models/ChucVu.java`

### Mục đích:
Model class đại diện cho đối tượng Chức vụ trong hệ thống, chứa các thuộc tính và phương thức getter/setter.

### Chi tiết code:

#### Khai báo package và class:
```java
package com.example.btl_mobile_qlns.models;                               // Khai báo package chứa các model class

public class ChucVu {                                                      // Khai báo class ChucVu public để có thể truy cập từ package khác
```

#### Khai báo thuộc tính:
```java
    private String maChucVu;                                               // Mã chức vụ (CV001, CV002...) - khóa chính
    private String tenChucVu;                                              // Tên chức vụ (VD: Giám đốc, Trưởng phòng, Nhân viên)
    private double mucLuongCoBan;                                          // Mức lương cơ bản của chức vụ (VNĐ)
    private int soNhanVien;                                                // Số lượng nhân viên đang đảm nhiệm chức vụ này (tính toán từ DB)
    private int trangThai;                                                 // Trạng thái: 1=Hoạt động, 0=Ngừng hoạt động
```

#### Constructor:
```java
    public ChucVu(String maChucVu, String tenChucVu, double mucLuongCoBan, int trangThai) {    // Constructor khởi tạo với 4 tham số chính
        this.maChucVu = maChucVu;                                          // Gán mã chức vụ
        this.tenChucVu = tenChucVu;                                        // Gán tên chức vụ
        this.mucLuongCoBan = mucLuongCoBan;                                // Gán mức lương cơ bản
        this.trangThai = trangThai;                                        // Gán trạng thái hoạt động
    }
```

#### Getter methods:
```java
    public String getMaChucVu() { return maChucVu; }                       // Getter lấy mã chức vụ
    public String getTenChucVu() { return tenChucVu; }                     // Getter lấy tên chức vụ
    public double getMucLuongCoBan() { return mucLuongCoBan; }             // Getter lấy mức lương cơ bản
    public int getSoNhanVien() { return soNhanVien; }                      // Getter lấy số nhân viên
    public int getTrangThai() { return trangThai; }                        // Getter lấy trạng thái
```

#### Setter methods:
```java
    public void setMaChucVu(String maChucVu) { this.maChucVu = maChucVu; }                     // Setter cập nhật mã chức vụ
    public void setTenChucVu(String tenChucVu) { this.tenChucVu = tenChucVu; }                 // Setter cập nhật tên chức vụ
    public void setMucLuongCoBan(double mucLuongCoBan) { this.mucLuongCoBan = mucLuongCoBan; } // Setter cập nhật mức lương cơ bản
    public void setSoNhanVien(int soNhanVien) { this.soNhanVien = soNhanVien; }                // Setter cập nhật số nhân viên
    public void setTrangThai(int trangThai) { this.trangThai = trangThai; }                    // Setter cập nhật trạng thái
}
```

---

## 2️⃣ ACTIVITY CHÍNH - QuanLyChucVuActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/QuanLyChucVuActivity.java`

### Mục đích:
Activity chính quản lý danh sách chức vụ, hiển thị thông tin, tìm kiếm và điều hướng đến các chức năng khác.

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
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.ListView;                                            // Import ListView widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
import com.example.btl_mobile_qlns.models.ChucVu;                          // Import model ChucVu

import java.util.ArrayList;                                                // Import ArrayList để lưu danh sách
import java.util.List;                                                     // Import List interface
```

#### Khai báo thuộc tính class:
```java
public class QuanLyChucVuActivity extends AppCompatActivity {              // Khai báo class kế thừa AppCompatActivity

    private TextView tvTitle, tvTongSo;                                    // TextView hiển thị tiêu đề và tổng số chức vụ
    private EditText etTimKiem;                                            // EditText cho chức năng tìm kiếm
    private Button btnThemChucVu;                                          // Button thêm chức vụ mới
    private ListView lvChucVu;                                             // ListView hiển thị danh sách chức vụ
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private ChucVuAdapter adapter;                                         // Adapter cho ListView
    private List<ChucVu> listChucVu;                                       // List chứa danh sách chức vụ
    private String currentRole;                                            // Role của user hiện tại (Admin/HR/Manager/Employee)
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_quan_ly_chuc_vu);                 // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database helper
        setupUI();                                                         // Thiết lập giao diện người dùng
        loadPositions();                                                   // Tải danh sách chức vụ từ database
        setupSearch();                                                     // Thiết lập chức năng tìm kiếm
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        tvTitle = findViewById(R.id.tv_title);                             // Ánh xạ TextView tiêu đề
        tvTongSo = findViewById(R.id.tv_tong_so);                          // Ánh xạ TextView hiển thị tổng số
        etTimKiem = findViewById(R.id.et_tim_kiem);                        // Ánh xạ EditText tìm kiếm
        btnThemChucVu = findViewById(R.id.btn_them_chuc_vu);               // Ánh xạ Button thêm chức vụ
        lvChucVu = findViewById(R.id.lv_chuc_vu);                          // Ánh xạ ListView danh sách chức vụ
    }
```

#### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập database helper
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        currentRole = getIntent().getStringExtra("role");                 // Lấy role từ Intent được truyền từ Activity trước
    }
```

#### Method setupUI:
```java
    private void setupUI() {                                               // Method thiết lập giao diện người dùng
        tvTitle.setText("QUẢN LÝ CHỨC VỤ");                                // Set text cho tiêu đề
        
        // Chỉ Admin và HR mới có thể thêm/sửa/xóa chức vụ
        if ("Admin".equals(currentRole) || "HR".equals(currentRole)) {     // Kiểm tra quyền Admin hoặc HR
            btnThemChucVu.setVisibility(View.VISIBLE);                     // Hiển thị button thêm chức vụ
            btnThemChucVu.setOnClickListener(v -> openAddPosition());      // Set listener cho button thêm
        } else {                                                           // Nếu không phải Admin/HR
            btnThemChucVu.setVisibility(View.GONE);                        // Ẩn button thêm chức vụ
        }
    }
```

#### Method setupSearch:
```java
    private void setupSearch() {                                           // Method thiết lập chức năng tìm kiếm
        etTimKiem.addTextChangedListener(new TextWatcher() {               // Thêm listener lắng nghe thay đổi text
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {}  // Method được gọi trước khi text thay đổi (không sử dụng)
            
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {      // Method được gọi khi text đang thay đổi
                searchPositions(s.toString());                             // Gọi method tìm kiếm với text hiện tại
            }
            
            @Override
            public void afterTextChanged(Editable s) {}                    // Method được gọi sau khi text thay đổi (không sử dụng)
        });
    }
```

#### Method loadPositions:
```java
    public void loadPositions() {                                          // Method tải danh sách chức vụ từ database (public để Adapter có thể gọi)
        try {                                                              // Bắt đầu try-catch để xử lý exception
            listChucVu = new ArrayList<>();                                // Khởi tạo ArrayList mới để chứa danh sách chức vụ
            Cursor cursor = dbHelper.getAllPositionsWithDetails();         // Gọi method database để lấy tất cả chức vụ với thông tin chi tiết
            
            if (cursor != null && cursor.moveToFirst()) {                  // Kiểm tra cursor không null và có dữ liệu
                do {                                                       // Vòng lặp do-while để duyệt qua tất cả record
                    String maChucVu = cursor.getString(cursor.getColumnIndexOrThrow("MaChucVu"));           // Lấy mã chức vụ từ cursor
                    String tenChucVu = cursor.getString(cursor.getColumnIndexOrThrow("TenChucVu"));         // Lấy tên chức vụ từ cursor
                    double mucLuongCoBan = cursor.getDouble(cursor.getColumnIndexOrThrow("MucLuongCoBan")); // Lấy mức lương cơ bản từ cursor
                    int trangThai = cursor.getInt(cursor.getColumnIndexOrThrow("TrangThai"));              // Lấy trạng thái từ cursor
                    int soNhanVien = cursor.getInt(cursor.getColumnIndexOrThrow("SoNhanVien"));            // Lấy số nhân viên từ cursor
                    
                    ChucVu chucVu = new ChucVu(maChucVu, tenChucVu, mucLuongCoBan, trangThai);             // Tạo object ChucVu với constructor
                    chucVu.setSoNhanVien(soNhanVien);                      // Set số nhân viên (không có trong constructor)
                    
                    listChucVu.add(chucVu);                                // Thêm object ChucVu vào list
                } while (cursor.moveToNext());                             // Tiếp tục vòng lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor để giải phóng bộ nhớ
            }
            
            updateUI();                                                    // Cập nhật giao diện với dữ liệu mới
        } catch (Exception e) {                                            // Bắt exception nếu có lỗi
            e.printStackTrace();                                           // In stack trace để debug
            Toast.makeText(this, "Lỗi khi tải danh sách chức vụ", Toast.LENGTH_SHORT).show();  // Hiển thị thông báo lỗi
        }
    }
```

#### Method searchPositions:
```java
    private void searchPositions(String keyword) {                         // Method tìm kiếm chức vụ theo từ khóa
        try {                                                              // Bắt đầu try-catch để xử lý exception
            listChucVu = new ArrayList<>();                                // Khởi tạo ArrayList mới cho kết quả tìm kiếm
            Cursor cursor;                                                 // Khai báo biến cursor
            
            if (keyword.trim().isEmpty()) {                                // Kiểm tra từ khóa có rỗng không
                cursor = dbHelper.getAllPositionsWithDetails();            // Nếu rỗng thì lấy tất cả chức vụ
            } else {                                                       // Nếu có từ khóa
                cursor = dbHelper.searchPositions(keyword);                // Gọi method tìm kiếm với từ khóa
            }
            
            if (cursor != null && cursor.moveToFirst()) {                  // Kiểm tra cursor không null và có dữ liệu
                do {                                                       // Vòng lặp do-while để duyệt qua tất cả record
                    String maChucVu = cursor.getString(cursor.getColumnIndexOrThrow("MaChucVu"));           // Lấy mã chức vụ từ cursor
                    String tenChucVu = cursor.getString(cursor.getColumnIndexOrThrow("TenChucVu"));         // Lấy tên chức vụ từ cursor
                    double mucLuongCoBan = cursor.getDouble(cursor.getColumnIndexOrThrow("MucLuongCoBan")); // Lấy mức lương cơ bản từ cursor
                    int trangThai = cursor.getInt(cursor.getColumnIndexOrThrow("TrangThai"));              // Lấy trạng thái từ cursor
                    int soNhanVien = cursor.getInt(cursor.getColumnIndexOrThrow("SoNhanVien"));            // Lấy số nhân viên từ cursor
                    
                    ChucVu chucVu = new ChucVu(maChucVu, tenChucVu, mucLuongCoBan, trangThai);             // Tạo object ChucVu với constructor
                    chucVu.setSoNhanVien(soNhanVien);                      // Set số nhân viên (không có trong constructor)
                    
                    listChucVu.add(chucVu);                                // Thêm object ChucVu vào list
                } while (cursor.moveToNext());                             // Tiếp tục vòng lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor để giải phóng bộ nhớ
            }
            
            updateUI();                                                    // Cập nhật giao diện với kết quả tìm kiếm
        } catch (Exception e) {                                            // Bắt exception nếu có lỗi
            e.printStackTrace();                                           // In stack trace để debug
        }
    }
```

#### Method updateUI:
```java
    private void updateUI() {                                              // Method cập nhật giao diện người dùng
        if (adapter == null) {                                             // Kiểm tra adapter chưa được khởi tạo
            adapter = new ChucVuAdapter(this, listChucVu, currentRole);    // Tạo adapter mới với context, data và role
            lvChucVu.setAdapter(adapter);                                  // Set adapter cho ListView
        } else {                                                           // Nếu adapter đã tồn tại
            adapter.updateData(listChucVu);                                // Cập nhật data cho adapter hiện tại
        }
        
        tvTongSo.setText("Tổng số: " + listChucVu.size() + " chức vụ");    // Cập nhật text hiển thị tổng số chức vụ
    }
```

#### Method openAddPosition:
```java
    private void openAddPosition() {                                       // Method mở Activity thêm chức vụ mới
        Intent intent = new Intent(this, ThemChucVuActivity.class);        // Tạo Intent chuyển đến ThemChucVuActivity
        intent.putExtra("role", currentRole);                             // Truyền role hiện tại qua Intent
        startActivityForResult(intent, 1002);                             // Start Activity và chờ kết quả với requestCode 1002
    }
```

#### Method onActivityResult:
```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {  // Method nhận kết quả từ Activity con
        super.onActivityResult(requestCode, resultCode, data);             // Gọi method của class cha
        if (requestCode == 1002 && resultCode == RESULT_OK) {             // Kiểm tra requestCode và resultCode
            loadPositions(); // Refresh danh sách                         // Tải lại danh sách chức vụ sau khi thêm/sửa thành công
        }
    }
```

#### Method onResume:
```java
    @Override
    protected void onResume() {                                            // Method được gọi khi Activity quay lại foreground
        super.onResume();                                                  // Gọi method của class cha
        loadPositions(); // Refresh khi quay lại                          // Tải lại danh sách chức vụ để cập nhật dữ liệu mới nhất
    }
}
```
---

## 3️⃣ ACTIVITY THÊM/SỬA - ThemChucVuActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ThemChucVuActivity.java`

### Mục đích:
Activity xử lý thêm mới và chỉnh sửa thông tin chức vụ, bao gồm thiết lập mức lương cơ bản.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.Switch;                                              // Import Switch widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
```

#### Khai báo thuộc tính class:
```java
public class ThemChucVuActivity extends AppCompatActivity {                // Khai báo class kế thừa AppCompatActivity

    private TextView tvTitle;                                              // TextView hiển thị tiêu đề (Thêm/Sửa)
    private EditText etMaChucVu, etTenChucVu, etMucLuong;                  // EditText cho mã, tên chức vụ và mức lương
    private Switch swTrangThai;                                            // Switch bật/tắt trạng thái hoạt động
    private Button btnLuu, btnHuy;                                         // Button lưu và hủy
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private boolean isEditMode = false;                                    // Flag kiểm tra chế độ sửa hay thêm mới
    private String originalMaChucVu;                                       // Mã chức vụ gốc (dùng cho chế độ sửa)
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_them_chuc_vu);                    // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database helper
        checkEditMode();                                                   // Kiểm tra chế độ thêm mới hay sửa
        setupButtons();                                                    // Thiết lập các button
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        tvTitle = findViewById(R.id.tv_title);                             // Ánh xạ TextView tiêu đề
        etMaChucVu = findViewById(R.id.et_ma_chuc_vu);                     // Ánh xạ EditText mã chức vụ
        etTenChucVu = findViewById(R.id.et_ten_chuc_vu);                   // Ánh xạ EditText tên chức vụ
        etMucLuong = findViewById(R.id.et_muc_luong);                      // Ánh xạ EditText mức lương
        swTrangThai = findViewById(R.id.sw_trang_thai);                    // Ánh xạ Switch trạng thái
        btnLuu = findViewById(R.id.btn_luu);                               // Ánh xạ Button lưu
        btnHuy = findViewById(R.id.btn_huy);                               // Ánh xạ Button hủy
    }
```

#### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập database helper
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
    }
```

#### Method checkEditMode:
```java
    private void checkEditMode() {                                         // Method kiểm tra và thiết lập chế độ thêm/sửa
        isEditMode = getIntent().getBooleanExtra("edit_mode", false);      // Lấy flag edit_mode từ Intent
        
        if (isEditMode) {                                                  // Nếu là chế độ sửa
            tvTitle.setText("SỬA CHỨC VỤ");                                // Set tiêu đề là "SỬA CHỨC VỤ"
            
            originalMaChucVu = getIntent().getStringExtra("ma_chuc_vu");   // Lấy mã chức vụ gốc từ Intent
            String tenChucVu = getIntent().getStringExtra("ten_chuc_vu");  // Lấy tên chức vụ từ Intent
            double mucLuong = getIntent().getDoubleExtra("muc_luong_co_ban", 0); // Lấy mức lương từ Intent, mặc định là 0
            int trangThai = getIntent().getIntExtra("trang_thai", 1);      // Lấy trạng thái từ Intent, mặc định là 1
            
            etMaChucVu.setText(originalMaChucVu);                          // Hiển thị mã chức vụ trong EditText
            etMaChucVu.setEnabled(false); // Không cho sửa mã chức vụ      // Disable EditText mã chức vụ (không cho sửa)
            etTenChucVu.setText(tenChucVu);                                // Hiển thị tên chức vụ trong EditText
            etMucLuong.setText(String.valueOf((long)mucLuong));            // Hiển thị mức lương trong EditText (cast về long để bỏ phần thập phân)
            swTrangThai.setChecked(trangThai == 1);                        // Set trạng thái Switch (true nếu trangThai = 1)
            
            btnLuu.setText("CẬP NHẬT");                                    // Đổi text button thành "CẬP NHẬT"
        } else {                                                           // Nếu là chế độ thêm mới
            tvTitle.setText("THÊM CHỨC VỤ MỚI");                           // Set tiêu đề là "THÊM CHỨC VỤ MỚI"
            
            // Tự động tạo mã chức vụ
            String nextCode = dbHelper.getNextPositionCode();              // Gọi method database để lấy mã chức vụ tiếp theo
            etMaChucVu.setText(nextCode);                                  // Hiển thị mã tự động trong EditText
            etMaChucVu.setEnabled(false); // Không cho sửa mã tự động      // Disable EditText mã chức vụ (không cho sửa mã tự động)
            
            swTrangThai.setChecked(true); // Mặc định hoạt động            // Set Switch mặc định là true (hoạt động)
            btnLuu.setText("THÊM CHỨC VỤ");                                // Set text button là "THÊM CHỨC VỤ"
        }
    }
```

#### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập các button
        btnLuu.setOnClickListener(v -> savePosition());                   // Set listener cho button lưu, gọi method savePosition
        btnHuy.setOnClickListener(v -> finish());                         // Set listener cho button hủy, đóng Activity
    }
```

#### Method savePosition:
```java
    private void savePosition() {                                          // Method lưu thông tin chức vụ
        String maChucVu = etMaChucVu.getText().toString().trim();          // Lấy mã chức vụ từ EditText và trim khoảng trắng
        String tenChucVu = etTenChucVu.getText().toString().trim();        // Lấy tên chức vụ từ EditText và trim khoảng trắng
        String mucLuongStr = etMucLuong.getText().toString().trim();       // Lấy mức lương từ EditText và trim khoảng trắng
        int trangThai = swTrangThai.isChecked() ? 1 : 0;                   // Lấy trạng thái từ Switch: true = 1, false = 0
        
        // Validate
        if (tenChucVu.isEmpty()) {                                         // Kiểm tra tên chức vụ có rỗng không
            Toast.makeText(this, "Vui lòng nhập tên chức vụ", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            etTenChucVu.requestFocus();                                    // Focus vào EditText tên chức vụ
            return;                                                        // Thoát method
        }
        
        if (mucLuongStr.isEmpty()) {                                       // Kiểm tra mức lương có rỗng không
            Toast.makeText(this, "Vui lòng nhập mức lương cơ bản", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            etMucLuong.requestFocus();                                     // Focus vào EditText mức lương
            return;                                                        // Thoát method
        }
        
        double mucLuong;                                                   // Biến lưu mức lương sau khi parse
        try {                                                              // Bắt đầu try-catch để parse số
            mucLuong = Double.parseDouble(mucLuongStr);                    // Parse string thành double
            if (mucLuong < 0) {                                            // Kiểm tra mức lương có âm không
                Toast.makeText(this, "Mức lương phải lớn hơn 0", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                etMucLuong.requestFocus();                                 // Focus vào EditText mức lương
                return;                                                    // Thoát method
            }
        } catch (NumberFormatException e) {                                // Bắt exception nếu parse thất bại
            Toast.makeText(this, "Mức lương không hợp lệ", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            etMucLuong.requestFocus();                                     // Focus vào EditText mức lương
            return;                                                        // Thoát method
        }
        
        boolean success;                                                   // Biến lưu kết quả thành công/thất bại
        if (isEditMode) {                                                  // Nếu là chế độ sửa
            success = dbHelper.updatePosition(originalMaChucVu, tenChucVu, mucLuong, trangThai); // Gọi method update database
        } else {                                                           // Nếu là chế độ thêm mới
            success = dbHelper.addPosition(maChucVu, tenChucVu, mucLuong, trangThai); // Gọi method add database
        }
        
        if (success) {                                                     // Nếu thành công
            String message = isEditMode ? "Cập nhật chức vụ thành công" : "Thêm chức vụ thành công"; // Tạo message tương ứng
            Toast.makeText(this, message, Toast.LENGTH_SHORT).show();      // Hiển thị thông báo thành công
            setResult(RESULT_OK);                                          // Set kết quả OK để Activity cha biết
            finish();                                                      // Đóng Activity
        } else {                                                           // Nếu thất bại
            String message = isEditMode ? "Lỗi khi cập nhật chức vụ" : "Lỗi khi thêm chức vụ"; // Tạo message lỗi tương ứng
            Toast.makeText(this, message, Toast.LENGTH_SHORT).show();      // Hiển thị thông báo lỗi
        }
    }
}
```

---

## 4️⃣ ADAPTER - ChucVuAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ChucVuAdapter.java`

### Mục đích:
Adapter quản lý hiển thị danh sách chức vụ trong ListView, xử lý sự kiện sửa/xóa và định dạng tiền tệ.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.AlertDialog;                                            // Import AlertDialog để hiển thị dialog xác nhận
import android.content.Context;                                            // Import Context để truy cập resources và services
import android.content.Intent;                                             // Import Intent để chuyển Activity
import android.view.LayoutInflater;                                        // Import LayoutInflater để inflate layout
import android.view.View;                                                  // Import View để xử lý UI components
import android.view.ViewGroup;                                             // Import ViewGroup để quản lý layout container
import android.widget.BaseAdapter;                                         // Import BaseAdapter làm base class
import android.widget.Button;                                              // Import Button widget
import android.widget.LinearLayout;                                        // Import LinearLayout widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
import com.example.btl_mobile_qlns.models.ChucVu;                          // Import model ChucVu

import java.text.NumberFormat;                                             // Import NumberFormat để định dạng số
import java.util.List;                                                     // Import List interface
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
```

#### Khai báo thuộc tính class:
```java
public class ChucVuAdapter extends BaseAdapter {                           // Khai báo class kế thừa BaseAdapter
    
    private Context context;                                               // Context của Activity sử dụng adapter
    private List<ChucVu> listChucVu;                                       // List chứa danh sách chức vụ
    private String currentRole;                                            // Role của user hiện tại
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private NumberFormat currencyFormat;                                   // Formatter để định dạng tiền tệ
```

#### Constructor:
```java
    public ChucVuAdapter(Context context, List<ChucVu> listChucVu, String currentRole) { // Constructor khởi tạo adapter
        this.context = context;                                            // Gán context
        this.listChucVu = listChucVu;                                      // Gán list chức vụ
        this.currentRole = currentRole;                                    // Gán role hiện tại
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
        this.currencyFormat = NumberFormat.getCurrencyInstance(new Locale("vi", "VN")); // Khởi tạo formatter tiền tệ Việt Nam
    }
```

#### Method getCount:
```java
    @Override
    public int getCount() {                                                // Method trả về số lượng item trong list
        return listChucVu != null ? listChucVu.size() : 0;                // Trả về size của list, nếu null thì trả về 0
    }
```

#### Method getItem:
```java
    @Override
    public Object getItem(int position) {                                  // Method trả về item tại vị trí position
        return listChucVu.get(position);                                   // Trả về object ChucVu tại vị trí position
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
            convertView = LayoutInflater.from(context).inflate(R.layout.item_chuc_vu, parent, false); // Inflate layout item_chuc_vu
        }
        
        ChucVu chucVu = listChucVu.get(position);                          // Lấy object ChucVu tại vị trí position
        
        TextView tvMaChucVu = convertView.findViewById(R.id.tv_ma_chuc_vu);         // Ánh xạ TextView mã chức vụ
        TextView tvTenChucVu = convertView.findViewById(R.id.tv_ten_chuc_vu);       // Ánh xạ TextView tên chức vụ
        TextView tvMucLuong = convertView.findViewById(R.id.tv_muc_luong);          // Ánh xạ TextView mức lương
        TextView tvSoNhanVien = convertView.findViewById(R.id.tv_so_nhan_vien);     // Ánh xạ TextView số nhân viên
        TextView tvTrangThai = convertView.findViewById(R.id.tv_trang_thai);        // Ánh xạ TextView trạng thái
        LinearLayout layoutButtons = convertView.findViewById(R.id.layout_buttons); // Ánh xạ LinearLayout chứa các button
        Button btnSua = convertView.findViewById(R.id.btn_sua);                     // Ánh xạ Button sửa
        Button btnXoa = convertView.findViewById(R.id.btn_xoa);                     // Ánh xạ Button xóa
        
        // Hiển thị thông tin chức vụ
        tvMaChucVu.setText("Mã: " + chucVu.getMaChucVu());                 // Set text hiển thị mã chức vụ
        tvTenChucVu.setText(chucVu.getTenChucVu());                        // Set text hiển thị tên chức vụ
        tvMucLuong.setText("Mức lương cơ bản: " + currencyFormat.format(chucVu.getMucLuongCoBan())); // Set text hiển thị mức lương với định dạng VNĐ
        tvSoNhanVien.setText("Số nhân viên: " + chucVu.getSoNhanVien());   // Set text hiển thị số nhân viên
        
        // Hiển thị trạng thái
        String trangThaiText = chucVu.getTrangThai() == 1 ? "Hoạt động" : "Ngừng hoạt động"; // Tạo text trạng thái dựa vào giá trị int
        tvTrangThai.setText("Trạng thái: " + trangThaiText);               // Set text hiển thị trạng thái
        
        // Thiết lập màu sắc cho trạng thái
        if (chucVu.getTrangThai() == 1) {                                  // Nếu trạng thái = 1 (hoạt động)
            tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_green_dark)); // Set màu xanh lá
        } else {                                                           // Nếu trạng thái = 0 (ngừng hoạt động)
            tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_red_dark));   // Set màu đỏ
        }
        
        // Hiển thị nút sửa/xóa cho Admin và HR
        if ("Admin".equals(currentRole) || "HR".equals(currentRole)) {     // Kiểm tra quyền Admin hoặc HR
            layoutButtons.setVisibility(View.VISIBLE);                     // Hiển thị layout chứa các button
            
            btnSua.setOnClickListener(v -> editPosition(chucVu));          // Set listener cho button sửa
            btnXoa.setOnClickListener(v -> showDeleteDialog(chucVu, position)); // Set listener cho button xóa
        } else {                                                           // Nếu không phải Admin/HR
            layoutButtons.setVisibility(View.GONE);                        // Ẩn layout chứa các button
        }
        
        return convertView;                                                // Trả về View đã được setup
    }
```
#### Method editPosition:
```java
    private void editPosition(ChucVu chucVu) {                             // Method xử lý sự kiện sửa chức vụ
        Intent intent = new Intent(context, ThemChucVuActivity.class);     // Tạo Intent chuyển đến ThemChucVuActivity
        intent.putExtra("role", currentRole);                             // Truyền role hiện tại qua Intent
        intent.putExtra("edit_mode", true);                               // Truyền flag edit_mode = true
        intent.putExtra("ma_chuc_vu", chucVu.getMaChucVu());              // Truyền mã chức vụ qua Intent
        intent.putExtra("ten_chuc_vu", chucVu.getTenChucVu());            // Truyền tên chức vụ qua Intent
        intent.putExtra("muc_luong_co_ban", chucVu.getMucLuongCoBan());   // Truyền mức lương cơ bản qua Intent
        intent.putExtra("trang_thai", chucVu.getTrangThai());             // Truyền trạng thái qua Intent
        
        if (context instanceof QuanLyChucVuActivity) {                     // Kiểm tra context có phải QuanLyChucVuActivity không
            ((QuanLyChucVuActivity) context).startActivityForResult(intent, 1002); // Cast context và gọi startActivityForResult
        }
    }
```

#### Method showDeleteDialog:
```java
    private void showDeleteDialog(ChucVu chucVu, int position) {           // Method hiển thị dialog xác nhận xóa
        // Kiểm tra ràng buộc: Nếu còn nhân viên thì không cho xóa
        if (chucVu.getSoNhanVien() > 0) {                                  // Kiểm tra số nhân viên > 0
            new AlertDialog.Builder(context)                               // Tạo AlertDialog builder
                .setTitle("Không thể xóa")                                 // Set tiêu đề dialog
                .setMessage("Chức vụ \"" + chucVu.getTenChucVu() + "\" hiện đang có " +  // Set message thông báo
                           chucVu.getSoNhanVien() + " nhân viên đang đảm nhiệm.\n\n" +
                           "Vui lòng thay đổi chức vụ cho các nhân viên này trước khi thực hiện xóa.")
                .setPositiveButton("Đã hiểu", null)                        // Set button "Đã hiểu" không có action
                .show();                                                   // Hiển thị dialog
            return;                                                        // Thoát method
        }

        new AlertDialog.Builder(context)                                   // Tạo AlertDialog builder cho xác nhận xóa
            .setTitle("Xóa chức vụ")                                       // Set tiêu đề dialog
            .setMessage("Bạn có chắc muốn xóa chức vụ \"" + chucVu.getTenChucVu() + "\"?\n\n" + // Set message xác nhận
                       "Lưu ý: Chức vụ sẽ được đánh dấu là ngừng hoạt động thay vì xóa hoàn toàn.")
            .setPositiveButton("Xóa", (dialog, which) -> {                 // Set button "Xóa" với action
                boolean success = dbHelper.deletePosition(chucVu.getMaChucVu()); // Gọi method xóa database
                if (success) {                                             // Nếu xóa thành công
                    Toast.makeText(context, "Đã xóa chức vụ thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                    refreshData();                                         // Refresh dữ liệu
                } else {                                                   // Nếu xóa thất bại
                    Toast.makeText(context, "Lỗi khi xóa chức vụ", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                }
            })
            .setNegativeButton("Hủy", null)                                // Set button "Hủy" không có action
            .show();                                                       // Hiển thị dialog
    }
```

#### Method refreshData:
```java
    private void refreshData() {                                           // Method refresh dữ liệu
        if (context instanceof QuanLyChucVuActivity) {                     // Kiểm tra context có phải QuanLyChucVuActivity không
            ((QuanLyChucVuActivity) context).loadPositions();              // Cast context và gọi method loadPositions
        }
    }
```

#### Method updateData:
```java
    public void updateData(List<ChucVu> newData) {                         // Method cập nhật dữ liệu mới cho adapter
        this.listChucVu = newData;                                         // Gán list mới
        notifyDataSetChanged();                                            // Thông báo adapter dữ liệu đã thay đổi để refresh UI
    }
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 5️⃣ LAYOUT DANH SÁCH - activity_quan_ly_chuc_vu.xml

**Đường dẫn**: `app/src/main/res/layout/activity_quan_ly_chuc_vu.xml`

### Mục đích:
Layout chính cho Activity quản lý danh sách chức vụ, bao gồm header, tìm kiếm và ListView.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Khai báo LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent (full height) -->
    android:orientation="vertical"                                         <!-- Orientation dọc (các child xếp từ trên xuống) -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt (#f5f5f5) -->
```

#### Header Section:
```xml
    <!-- Header -->
    <LinearLayout                                                          <!-- LinearLayout chứa phần header -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:elevation="2dp">                                           <!-- Elevation 2dp tạo shadow -->

        <TextView                                                          <!-- TextView hiển thị tiêu đề -->
            android:id="@+id/tv_title"                                     <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="QUẢN LÝ CHỨC VỤ"                                 <!-- Text mặc định hiển thị -->
            android:textSize="24sp"                                        <!-- Kích thước font 24sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương (#2196F3) -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->
```

#### Search Section:
```xml
        <!-- Tìm kiếm -->
        <EditText                                                          <!-- EditText cho chức năng tìm kiếm -->
            android:id="@+id/et_tim_kiem"                                  <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:padding="12dp"                                         <!-- Padding 12dp cho tất cả các cạnh -->
            android:hint="Tìm kiếm chức vụ..."                             <!-- Hint text hiển thị khi rỗng -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:drawableStart="@android:drawable/ic_menu_search"       <!-- Icon search ở đầu (bên trái) -->
            android:drawablePadding="8dp"                                  <!-- Padding giữa icon và text 8dp -->
            android:layout_marginBottom="12dp" />                          <!-- Margin bottom 12dp -->
```

#### Summary and Button Section:
```xml
        <!-- Thông tin tổng quan -->
        <LinearLayout                                                      <!-- LinearLayout chứa thông tin tổng quan và button -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang (các child xếp từ trái qua phải) -->
            android:gravity="center_vertical">                             <!-- Căn giữa theo chiều dọc -->

            <TextView                                                      <!-- TextView hiển thị tổng số chức vụ -->
                android:id="@+id/tv_tong_so"                               <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:layout_weight="1"                                  <!-- Weight 1 để chiếm không gian còn lại -->
                android:text="Tổng số: 0 chức vụ"                          <!-- Text mặc định hiển thị -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textColor="#666" />                                <!-- Màu chữ xám (#666) -->

            <Button                                                        <!-- Button thêm chức vụ mới -->
                android:id="@+id/btn_them_chuc_vu"                         <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="40dp"                               <!-- Chiều cao cố định 40dp -->
                android:text="THÊM CHỨC VỤ"                                <!-- Text hiển thị trên button -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:background="@drawable/btn_primary"                 <!-- Background custom từ drawable -->
                android:minWidth="140dp"                                   <!-- Chiều rộng tối thiểu 140dp -->
                android:visibility="gone" />                               <!-- Visibility gone (ẩn mặc định, hiển thị theo quyền) -->

        </LinearLayout>

    </LinearLayout>
```

#### ListView Section:
```xml
    <!-- Danh sách chức vụ -->
    <ListView                                                              <!-- ListView hiển thị danh sách chức vụ -->
        android:id="@+id/lv_chuc_vu"                                       <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="0dp"                                       <!-- Chiều cao 0dp để sử dụng layout_weight -->
        android:layout_weight="1"                                          <!-- Weight 1 để chiếm không gian còn lại -->
        android:layout_margin="16dp"                                       <!-- Margin 16dp cho tất cả các cạnh -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:divider="#E0E0E0"                                          <!-- Màu divider giữa các item -->
        android:dividerHeight="1dp"                                        <!-- Chiều cao divider 1dp -->
        android:elevation="2dp" />                                         <!-- Elevation 2dp tạo shadow -->

</LinearLayout>
```

---

## 6️⃣ LAYOUT THÊM/SỬA - activity_them_chuc_vu.xml

**Đường dẫn**: `app/src/main/res/layout/activity_them_chuc_vu.xml`

### Mục đích:
Layout cho Activity thêm mới và chỉnh sửa chức vụ, bao gồm form nhập liệu và các button.

### Chi tiết code:

#### Khai báo XML và ScrollView:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"    <!-- Khai báo ScrollView root để có thể scroll khi content dài -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent (full height) -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt (#f5f5f5) -->

    <LinearLayout                                                          <!-- LinearLayout chính chứa tất cả content -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:padding="16dp">                                            <!-- Padding 16dp cho tất cả các cạnh -->
```

#### Header Section:
```xml
        <!-- Header -->
        <LinearLayout                                                      <!-- LinearLayout chứa phần header -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="vertical"                                 <!-- Orientation dọc -->
            android:background="@android:color/white"                      <!-- Background màu trắng -->
            android:padding="16dp"                                         <!-- Padding 16dp cho tất cả các cạnh -->
            android:elevation="2dp"                                        <!-- Elevation 2dp tạo shadow -->
            android:layout_marginBottom="16dp">                            <!-- Margin bottom 16dp -->

            <TextView                                                      <!-- TextView hiển thị tiêu đề -->
                android:id="@+id/tv_title"                                 <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="THÊM CHỨC VỤ MỚI"                            <!-- Text mặc định hiển thị -->
                android:textSize="24sp"                                    <!-- Kích thước font 24sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#2196F3"                                <!-- Màu chữ xanh dương (#2196F3) -->
                android:gravity="center" />                                <!-- Căn giữa text -->

        </LinearLayout>
```

#### Form Information Section:
```xml
        <!-- Form thông tin -->
        <LinearLayout                                                      <!-- LinearLayout chứa form nhập thông tin -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="vertical"                                 <!-- Orientation dọc -->
            android:background="@android:color/white"                      <!-- Background màu trắng -->
            android:padding="16dp"                                         <!-- Padding 16dp cho tất cả các cạnh -->
            android:elevation="2dp"                                        <!-- Elevation 2dp tạo shadow -->
            android:layout_marginBottom="16dp">                            <!-- Margin bottom 16dp -->

            <!-- Mã chức vụ -->
            <TextView                                                      <!-- TextView label cho mã chức vụ -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Mã chức vụ"                                  <!-- Text label hiển thị -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ đen nhạt (#333) -->
                android:layout_marginBottom="8dp" />                       <!-- Margin bottom 8dp -->

            <EditText                                                      <!-- EditText nhập mã chức vụ -->
                android:id="@+id/et_ma_chuc_vu"                            <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp cho tất cả các cạnh -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="Mã chức vụ tự động"                          <!-- Hint text hiển thị khi rỗng -->
                android:enabled="false" />                                 <!-- Disabled (không cho chỉnh sửa) -->

            <!-- Tên chức vụ -->
            <TextView                                                      <!-- TextView label cho tên chức vụ -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Tên chức vụ *"                               <!-- Text label với dấu * bắt buộc -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ đen nhạt (#333) -->
                android:layout_marginBottom="8dp" />                       <!-- Margin bottom 8dp -->

            <EditText                                                      <!-- EditText nhập tên chức vụ -->
                android:id="@+id/et_ten_chuc_vu"                           <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp cho tất cả các cạnh -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="Nhập tên chức vụ"                            <!-- Hint text hiển thị khi rỗng -->
                android:inputType="text" />                                <!-- Input type là text -->

            <!-- Mức lương cơ bản -->
            <TextView                                                      <!-- TextView label cho mức lương -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Mức lương cơ bản (VNĐ) *"                    <!-- Text label với đơn vị tiền tệ và dấu * bắt buộc -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ đen nhạt (#333) -->
                android:layout_marginBottom="8dp" />                       <!-- Margin bottom 8dp -->

            <EditText                                                      <!-- EditText nhập mức lương -->
                android:id="@+id/et_muc_luong"                             <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp cho tất cả các cạnh -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="Nhập mức lương cơ bản"                       <!-- Hint text hiển thị khi rỗng -->
                android:inputType="number" />                              <!-- Input type là number (chỉ cho phép nhập số) -->

            <!-- Trạng thái -->
            <LinearLayout                                                  <!-- LinearLayout chứa label và Switch trạng thái -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa content -->
                android:orientation="horizontal"                           <!-- Orientation ngang -->
                android:gravity="center_vertical"                          <!-- Căn giữa theo chiều dọc -->
                android:layout_marginBottom="16dp">                        <!-- Margin bottom 16dp -->

                <TextView                                                  <!-- TextView label cho trạng thái -->
                    android:layout_width="0dp"                             <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ chứa text -->
                    android:layout_weight="1"                              <!-- Weight 1 để chiếm không gian còn lại -->
                    android:text="Trạng thái hoạt động"                    <!-- Text label hiển thị -->
                    android:textSize="16sp"                                <!-- Kích thước font 16sp -->
                    android:textStyle="bold"                               <!-- Style chữ đậm -->
                    android:textColor="#333" />                            <!-- Màu chữ đen nhạt (#333) -->

                <Switch                                                    <!-- Switch bật/tắt trạng thái -->
                    android:id="@+id/sw_trang_thai"                        <!-- ID để truy cập từ Java code -->
                    android:layout_width="wrap_content"                    <!-- Chiều rộng vừa đủ chứa Switch -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ chứa Switch -->
                    android:checked="true" />                              <!-- Mặc định checked (true) -->

            </LinearLayout>

        </LinearLayout>
```

#### Buttons Section:
```xml
        <!-- Buttons -->
        <LinearLayout                                                      <!-- LinearLayout chứa các button -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:gravity="center">                                      <!-- Căn giữa các button -->

            <Button                                                        <!-- Button hủy -->
                android:id="@+id/btn_huy"                                  <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="HỦY"                                         <!-- Text hiển thị trên button -->
                android:textColor="#2196F3"                                <!-- Màu chữ xanh dương (#2196F3) -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:background="@drawable/btn_secondary"               <!-- Background custom từ drawable -->
                android:layout_marginEnd="8dp" />                          <!-- Margin end 8dp -->

            <Button                                                        <!-- Button lưu -->
                android:id="@+id/btn_luu"                                  <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="THÊM CHỨC VỤ"                                <!-- Text mặc định hiển thị trên button -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:background="@drawable/btn_primary"                 <!-- Background custom từ drawable -->
                android:layout_marginStart="8dp" />                        <!-- Margin start 8dp -->

        </LinearLayout>

    </LinearLayout>

</ScrollView>
```
---

## 7️⃣ LAYOUT ITEM - item_chuc_vu.xml

**Đường dẫn**: `app/src/main/res/layout/item_chuc_vu.xml`

### Mục đích:
Layout cho từng item chức vụ trong ListView, hiển thị thông tin chi tiết và các button hành động.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Khai báo LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ chứa content -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:padding="16dp"                                                 <!-- Padding 16dp cho tất cả các cạnh -->
    android:background="@android:color/white">                             <!-- Background màu trắng -->
```

#### Main Information Section:
```xml
    <!-- Thông tin chính -->
    <LinearLayout                                                          <!-- LinearLayout chứa thông tin chính (mã và trạng thái) -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:layout_marginBottom="8dp">                                 <!-- Margin bottom 8dp -->

        <TextView                                                          <!-- TextView hiển thị mã chức vụ -->
            android:id="@+id/tv_ma_chuc_vu"                                <!-- ID để truy cập từ Java code -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Mã: CV001"                                       <!-- Text mặc định hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương (#2196F3) -->
            android:layout_marginEnd="16dp" />                             <!-- Margin end 16dp -->

        <TextView                                                          <!-- TextView hiển thị trạng thái -->
            android:id="@+id/tv_trang_thai"                                <!-- ID để truy cập từ Java code -->
            android:layout_width="0dp"                                     <!-- Chiều rộng 0dp để sử dụng layout_weight -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:layout_weight="1"                                      <!-- Weight 1 để chiếm không gian còn lại -->
            android:text="Trạng thái: Hoạt động"                           <!-- Text mặc định hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:gravity="end" />                                       <!-- Căn phải text -->

    </LinearLayout>

    <!-- Tên chức vụ -->
    <TextView                                                              <!-- TextView hiển thị tên chức vụ -->
        android:id="@+id/tv_ten_chuc_vu"                                   <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Giám đốc"                                            <!-- Text mặc định hiển thị -->
        android:textSize="18sp"                                            <!-- Kích thước font 18sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ đen nhạt (#333) -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->
```

#### Detail Information Section:
```xml
    <!-- Thông tin chi tiết -->
    <LinearLayout                                                          <!-- LinearLayout chứa thông tin chi tiết -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical">                                    <!-- Orientation dọc -->

        <TextView                                                          <!-- TextView hiển thị mức lương cơ bản -->
            android:id="@+id/tv_muc_luong"                                 <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Mức lương cơ bản: 50.000.000 ₫"                 <!-- Text mặc định hiển thị với ký hiệu tiền tệ VNĐ -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám (#666) -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->

        <TextView                                                          <!-- TextView hiển thị số nhân viên -->
            android:id="@+id/tv_so_nhan_vien"                              <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Số nhân viên: 5"                                 <!-- Text mặc định hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666" />                                    <!-- Màu chữ xám (#666) -->

    </LinearLayout>
```

#### Action Buttons Section:
```xml
    <!-- Buttons sửa/xóa -->
    <LinearLayout                                                          <!-- LinearLayout chứa các button hành động -->
        android:id="@+id/layout_buttons"                                   <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:gravity="end"                                              <!-- Căn phải các button -->
        android:layout_marginTop="12dp"                                    <!-- Margin top 12dp -->
        android:visibility="gone">                                         <!-- Visibility gone (ẩn mặc định, hiển thị theo quyền) -->

        <Button                                                            <!-- Button xóa -->
            android:id="@+id/btn_xoa"                                      <!-- ID để truy cập từ Java code -->
            android:layout_width="90dp"                                    <!-- Chiều rộng cố định 90dp -->
            android:layout_height="40dp"                                   <!-- Chiều cao cố định 40dp -->
            android:text="XÓA"                                             <!-- Text hiển thị trên button -->
            android:textSize="12sp"                                        <!-- Kích thước font 12sp -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:backgroundTint="#F44336"                               <!-- Background tint màu đỏ (#F44336) -->
            android:layout_marginEnd="8dp" />                              <!-- Margin end 8dp -->

        <Button                                                            <!-- Button sửa -->
            android:id="@+id/btn_sua"                                      <!-- ID để truy cập từ Java code -->
            android:layout_width="90dp"                                    <!-- Chiều rộng cố định 90dp -->
            android:layout_height="40dp"                                   <!-- Chiều cao cố định 40dp -->
            android:text="SỬA"                                             <!-- Text hiển thị trên button -->
            android:textSize="12sp"                                        <!-- Kích thước font 12sp -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:backgroundTint="#2196F3" />                            <!-- Background tint màu xanh dương (#2196F3) -->

    </LinearLayout>

</LinearLayout>
```

---

## 🔧 TÍCH HỢP VÀ LIÊN KẾT

### Database Methods (DatabaseHelper.java):
- `getAllPositionsWithDetails()`: Lấy danh sách chức vụ với thông tin chi tiết
- `searchPositions(String keyword)`: Tìm kiếm chức vụ theo từ khóa
- `getNextPositionCode()`: Tạo mã chức vụ tự động tiếp theo
- `addPosition()`: Thêm chức vụ mới
- `updatePosition()`: Cập nhật thông tin chức vụ
- `deletePosition()`: Xóa chức vụ (soft delete)

### Navigation Flow:
1. **DashboardActivity** → **QuanLyChucVuActivity** (với role)
2. **QuanLyChucVuActivity** → **ThemChucVuActivity** (thêm mới)
3. **ChucVuAdapter** → **ThemChucVuActivity** (chỉnh sửa)
4. **ThemChucVuActivity** → **QuanLyChucVuActivity** (kết quả)

### Permission System:
- **Admin/HR**: Full quyền (xem, thêm, sửa, xóa)
- **Manager**: Chỉ xem danh sách
- **Employee**: Không có quyền truy cập

### Currency Formatting:
- Sử dụng `NumberFormat.getCurrencyInstance(new Locale("vi", "VN"))` để định dạng tiền tệ VNĐ
- Hiển thị mức lương theo chuẩn Việt Nam với ký hiệu ₫

### Business Rules:
- **Auto-generated codes**: CV001, CV002, CV003...
- **Salary validation**: Mức lương phải > 0
- **Constraint checking**: Không cho xóa chức vụ còn nhân viên đảm nhiệm
- **Soft delete**: Đánh dấu trạng thái = 0 thay vì xóa hoàn toàn
- **Input validation**: Kiểm tra tên chức vụ và mức lương bắt buộc

---

## 📝 KẾT LUẬN

Module Quản lý Chức vụ được thiết kế với kiến trúc MVC rõ ràng, phân quyền chặt chẽ và giao diện thân thiện. Hệ thống đảm bảo tính toàn vẹn dữ liệu thông qua các ràng buộc nghiệp vụ, cung cấp trải nghiệm người dùng mượt mà với các chức năng tìm kiếm, thêm/sửa/xóa chức vụ và quản lý mức lương cơ bản. Định dạng tiền tệ VNĐ và validation đầu vào đảm bảo dữ liệu chính xác và phù hợp với thực tế sử dụng.