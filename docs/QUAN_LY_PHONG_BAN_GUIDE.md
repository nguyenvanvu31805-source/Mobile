# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ PHÒNG BAN

## 📋 TỔNG QUAN

Chức năng Quản lý Phòng ban là một module quan trọng của hệ thống QLNS, cho phép Admin/HR quản lý các phòng ban trong tổ chức, bao gồm thêm mới, chỉnh sửa, xóa phòng ban và phân công trưởng phòng.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── QuanLyPhongBanActivity.java          # Activity chính - Danh sách phòng ban
├── ThemPhongBanActivity.java            # Activity thêm/sửa phòng ban  
├── PhongBanAdapter.java                 # Adapter hiển thị danh sách
├── models/PhongBan.java                 # Model class phòng ban
└── database/DatabaseHelper.java         # Xử lý database

app/src/main/res/layout/
├── activity_quan_ly_phong_ban.xml       # Layout danh sách phòng ban
├── activity_them_phong_ban.xml          # Layout form thêm/sửa
└── item_phong_ban.xml                   # Layout item trong ListView
```

## 📊 NGHIỆP VỤ QUẢN LÝ PHÒNG BAN

### 1. Quy trình nghiệp vụ:
- **Xem danh sách**: Hiển thị tất cả phòng ban với thông tin chi tiết
- **Tìm kiếm**: Tìm theo tên hoặc mã phòng ban
- **Thêm mới**: Tạo phòng ban với mã tự động tăng (PB001, PB002...)
- **Cập nhật**: Chỉnh sửa thông tin phòng ban, phân công trưởng phòng
- **Xóa**: Soft delete phòng ban (đánh dấu ngừng hoạt động)
- **Ràng buộc**: Không cho xóa phòng ban còn nhân viên

### 2. Phân quyền:
- **Admin**: Full quyền tất cả chức năng
- **HR**: Full quyền tất cả chức năng  
- **Manager**: Chỉ xem danh sách
- **Employee**: Không có quyền truy cập

---

## 📱 CHI TIẾT CÁC FILE
## 1️⃣ MODEL CLASS - PhongBan.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/models/PhongBan.java`

### Mục đích:
Model class đại diện cho đối tượng Phòng ban trong hệ thống, chứa các thuộc tính và phương thức getter/setter.

### Chi tiết code:

#### Khai báo package và class:
```java
package com.example.btl_mobile_qlns.models;                               // Khai báo package chứa các model class

public class PhongBan {                                                    // Khai báo class PhongBan public để có thể truy cập từ package khác
```

#### Khai báo thuộc tính:
```java
    private String maPhongBan;                                             // Mã phòng ban (PB001, PB002...) - khóa chính
    private String tenPhongBan;                                            // Tên phòng ban (VD: Phòng Nhân sự, Phòng Kế toán)
    private String truongPhong;                                            // Mã nhân viên làm trưởng phòng (foreign key)
    private String tenTruongPhong;                                         // Tên trưởng phòng (để hiển thị, không lưu DB)
    private int soNhanVien;                                                // Số lượng nhân viên trong phòng ban (tính toán từ DB)
    private int trangThai;                                                 // Trạng thái: 1=Hoạt động, 0=Ngừng hoạt động
```

#### Constructor:
```java
    public PhongBan(String maPhongBan, String tenPhongBan, String truongPhong, int trangThai) {  // Constructor khởi tạo với 4 tham số chính
        this.maPhongBan = maPhongBan;                                      // Gán mã phòng ban
        this.tenPhongBan = tenPhongBan;                                    // Gán tên phòng ban
        this.truongPhong = truongPhong;                                    // Gán mã trưởng phòng
        this.trangThai = trangThai;                                        // Gán trạng thái hoạt động
    }
```

#### Getter methods:
```java
    public String getMaPhongBan() { return maPhongBan; }                   // Getter lấy mã phòng ban
    public String getTenPhongBan() { return tenPhongBan; }                 // Getter lấy tên phòng ban
    public String getTruongPhong() { return truongPhong; }                 // Getter lấy mã trưởng phòng
    public String getTenTruongPhong() { return tenTruongPhong; }           // Getter lấy tên trưởng phòng
    public int getSoNhanVien() { return soNhanVien; }                      // Getter lấy số nhân viên
    public int getTrangThai() { return trangThai; }                        // Getter lấy trạng thái
```

#### Setter methods:
```java
    public void setMaPhongBan(String maPhongBan) { this.maPhongBan = maPhongBan; }           // Setter cập nhật mã phòng ban
    public void setTenPhongBan(String tenPhongBan) { this.tenPhongBan = tenPhongBan; }       // Setter cập nhật tên phòng ban
    public void setTruongPhong(String truongPhong) { this.truongPhong = truongPhong; }       // Setter cập nhật mã trưởng phòng
    public void setTenTruongPhong(String tenTruongPhong) { this.tenTruongPhong = tenTruongPhong; }  // Setter cập nhật tên trưởng phòng
    public void setSoNhanVien(int soNhanVien) { this.soNhanVien = soNhanVien; }              // Setter cập nhật số nhân viên
    public void setTrangThai(int trangThai) { this.trangThai = trangThai; }                  // Setter cập nhật trạng thái
}
```

---

## 2️⃣ ACTIVITY CHÍNH - QuanLyPhongBanActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/QuanLyPhongBanActivity.java`

### Mục đích:
Activity chính quản lý danh sách phòng ban, hiển thị thông tin, tìm kiếm và điều hướng đến các chức năng khác.

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
import com.example.btl_mobile_qlns.models.PhongBan;                        // Import model PhongBan

import java.util.ArrayList;                                                // Import ArrayList để lưu danh sách
import java.util.List;                                                     // Import List interface
```

#### Khai báo thuộc tính class:
```java
public class QuanLyPhongBanActivity extends AppCompatActivity {            // Khai báo class kế thừa AppCompatActivity

    private TextView tvTitle, tvTongSo;                                    // TextView hiển thị tiêu đề và tổng số phòng ban
    private EditText etTimKiem;                                            // EditText cho chức năng tìm kiếm
    private Button btnThemPhongBan;                                        // Button thêm phòng ban mới
    private ListView lvPhongBan;                                           // ListView hiển thị danh sách phòng ban
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private PhongBanAdapter adapter;                                       // Adapter cho ListView
    private List<PhongBan> listPhongBan;                                   // List chứa danh sách phòng ban
    private String currentRole;                                            // Role của user hiện tại (Admin/HR/Manager/Employee)
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_quan_ly_phong_ban);               // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database helper
        setupUI();                                                         // Thiết lập giao diện người dùng
        loadDepartments();                                                 // Tải danh sách phòng ban từ database
        setupSearch();                                                     // Thiết lập chức năng tìm kiếm
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        tvTitle = findViewById(R.id.tv_title);                             // Ánh xạ TextView tiêu đề
        tvTongSo = findViewById(R.id.tv_tong_so);                          // Ánh xạ TextView hiển thị tổng số
        etTimKiem = findViewById(R.id.et_tim_kiem);                        // Ánh xạ EditText tìm kiếm
        btnThemPhongBan = findViewById(R.id.btn_them_phong_ban);           // Ánh xạ Button thêm phòng ban
        lvPhongBan = findViewById(R.id.lv_phong_ban);                      // Ánh xạ ListView danh sách phòng ban
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
        tvTitle.setText("QUẢN LÝ PHÒNG BAN");                              // Set text cho tiêu đề
        
        // Chỉ Admin và HR mới có thể thêm/sửa/xóa phòng ban
        if ("Admin".equals(currentRole) || "HR".equals(currentRole)) {     // Kiểm tra quyền Admin hoặc HR
            btnThemPhongBan.setVisibility(View.VISIBLE);                   // Hiển thị button thêm phòng ban
            btnThemPhongBan.setOnClickListener(v -> openAddDepartment());  // Set listener cho button thêm
        } else {                                                           // Nếu không phải Admin/HR
            btnThemPhongBan.setVisibility(View.GONE);                      // Ẩn button thêm phòng ban
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
                searchDepartments(s.toString());                           // Gọi method tìm kiếm với text hiện tại
            }
            
            @Override
            public void afterTextChanged(Editable s) {}                    // Method được gọi sau khi text thay đổi (không sử dụng)
        });
    }
```

#### Method loadDepartments:
```java
    public void loadDepartments() {                                        // Method tải danh sách phòng ban từ database (public để Adapter có thể gọi)
        try {                                                              // Bắt đầu try-catch để xử lý exception
            listPhongBan = new ArrayList<>();                              // Khởi tạo ArrayList mới để chứa danh sách phòng ban
            Cursor cursor = dbHelper.getAllDepartmentsWithDetails();       // Gọi method database để lấy tất cả phòng ban với thông tin chi tiết
            
            if (cursor != null && cursor.moveToFirst()) {                  // Kiểm tra cursor không null và có dữ liệu
                do {                                                       // Vòng lặp do-while để duyệt qua tất cả record
                    String maPhongBan = cursor.getString(cursor.getColumnIndexOrThrow("MaPhongBan"));        // Lấy mã phòng ban từ cursor
                    String tenPhongBan = cursor.getString(cursor.getColumnIndexOrThrow("TenPhongBan"));      // Lấy tên phòng ban từ cursor
                    String truongPhong = cursor.getString(cursor.getColumnIndexOrThrow("TruongPhong"));      // Lấy mã trưởng phòng từ cursor
                    int trangThai = cursor.getInt(cursor.getColumnIndexOrThrow("TrangThai"));               // Lấy trạng thái từ cursor
                    int soNhanVien = cursor.getInt(cursor.getColumnIndexOrThrow("SoNhanVien"));             // Lấy số nhân viên từ cursor
                    String tenTruongPhong = cursor.getString(cursor.getColumnIndexOrThrow("TenTruongPhong")); // Lấy tên trưởng phòng từ cursor
                    
                    PhongBan phongBan = new PhongBan(maPhongBan, tenPhongBan, truongPhong, trangThai);      // Tạo object PhongBan với constructor
                    phongBan.setSoNhanVien(soNhanVien);                    // Set số nhân viên (không có trong constructor)
                    phongBan.setTenTruongPhong(tenTruongPhong);            // Set tên trưởng phòng (không có trong constructor)
                    
                    listPhongBan.add(phongBan);                            // Thêm object PhongBan vào list
                } while (cursor.moveToNext());                             // Tiếp tục vòng lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor để giải phóng bộ nhớ
            }
            
            updateUI();                                                    // Cập nhật giao diện với dữ liệu mới
        } catch (Exception e) {                                            // Bắt exception nếu có lỗi
            e.printStackTrace();                                           // In stack trace để debug
            Toast.makeText(this, "Lỗi khi tải danh sách phòng ban", Toast.LENGTH_SHORT).show();  // Hiển thị thông báo lỗi
        }
    }
```

#### Method searchDepartments:
```java
    private void searchDepartments(String keyword) {                       // Method tìm kiếm phòng ban theo từ khóa
        try {                                                              // Bắt đầu try-catch để xử lý exception
            listPhongBan = new ArrayList<>();                              // Khởi tạo ArrayList mới cho kết quả tìm kiếm
            Cursor cursor;                                                 // Khai báo biến cursor
            
            if (keyword.trim().isEmpty()) {                                // Kiểm tra từ khóa có rỗng không
                cursor = dbHelper.getAllDepartmentsWithDetails();          // Nếu rỗng thì lấy tất cả phòng ban
            } else {                                                       // Nếu có từ khóa
                cursor = dbHelper.searchDepartments(keyword);              // Gọi method tìm kiếm với từ khóa
            }
            
            if (cursor != null && cursor.moveToFirst()) {                  // Kiểm tra cursor không null và có dữ liệu
                do {                                                       // Vòng lặp do-while để duyệt qua tất cả record
                    String maPhongBan = cursor.getString(cursor.getColumnIndexOrThrow("MaPhongBan"));        // Lấy mã phòng ban từ cursor
                    String tenPhongBan = cursor.getString(cursor.getColumnIndexOrThrow("TenPhongBan"));      // Lấy tên phòng ban từ cursor
                    String truongPhong = cursor.getString(cursor.getColumnIndexOrThrow("TruongPhong"));      // Lấy mã trưởng phòng từ cursor
                    int trangThai = cursor.getInt(cursor.getColumnIndexOrThrow("TrangThai"));               // Lấy trạng thái từ cursor
                    int soNhanVien = cursor.getInt(cursor.getColumnIndexOrThrow("SoNhanVien"));             // Lấy số nhân viên từ cursor
                    String tenTruongPhong = cursor.getString(cursor.getColumnIndexOrThrow("TenTruongPhong")); // Lấy tên trưởng phòng từ cursor
                    
                    PhongBan phongBan = new PhongBan(maPhongBan, tenPhongBan, truongPhong, trangThai);      // Tạo object PhongBan với constructor
                    phongBan.setSoNhanVien(soNhanVien);                    // Set số nhân viên (không có trong constructor)
                    phongBan.setTenTruongPhong(tenTruongPhong);            // Set tên trưởng phòng (không có trong constructor)
                    
                    listPhongBan.add(phongBan);                            // Thêm object PhongBan vào list
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
            adapter = new PhongBanAdapter(this, listPhongBan, currentRole); // Tạo adapter mới với context, data và role
            lvPhongBan.setAdapter(adapter);                                // Set adapter cho ListView
        } else {                                                           // Nếu adapter đã tồn tại
            adapter.updateData(listPhongBan);                              // Cập nhật data cho adapter hiện tại
        }
        
        tvTongSo.setText("Tổng số: " + listPhongBan.size() + " phòng ban"); // Cập nhật text hiển thị tổng số phòng ban
    }
```

#### Method openAddDepartment:
```java
    private void openAddDepartment() {                                     // Method mở Activity thêm phòng ban mới
        Intent intent = new Intent(this, ThemPhongBanActivity.class);      // Tạo Intent chuyển đến ThemPhongBanActivity
        intent.putExtra("role", currentRole);                             // Truyền role hiện tại qua Intent
        startActivityForResult(intent, 1001);                             // Start Activity và chờ kết quả với requestCode 1001
    }
```

#### Method onActivityResult:
```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {  // Method nhận kết quả từ Activity con
        super.onActivityResult(requestCode, resultCode, data);             // Gọi method của class cha
        if (requestCode == 1001 && resultCode == RESULT_OK) {             // Kiểm tra requestCode và resultCode
            loadDepartments(); // Refresh danh sách                       // Tải lại danh sách phòng ban sau khi thêm/sửa thành công
        }
    }
```

#### Method onResume:
```java
    @Override
    protected void onResume() {                                            // Method được gọi khi Activity quay lại foreground
        super.onResume();                                                  // Gọi method của class cha
        loadDepartments(); // Refresh khi quay lại                        // Tải lại danh sách phòng ban để cập nhật dữ liệu mới nhất
    }
}
```

---

## 3️⃣ ACTIVITY THÊM/SỬA - ThemPhongBanActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ThemPhongBanActivity.java`

### Mục đích:
Activity xử lý thêm mới và chỉnh sửa thông tin phòng ban, bao gồm phân công trưởng phòng.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.view.View;                                                  // Import View để xử lý UI components
import android.widget.ArrayAdapter;                                        // Import ArrayAdapter cho Spinner
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.Spinner;                                             // Import Spinner widget
import android.widget.Switch;                                              // Import Switch widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database

import java.util.ArrayList;                                                // Import ArrayList để lưu danh sách
import java.util.List;                                                     // Import List interface
```

#### Khai báo thuộc tính class:
```java
public class ThemPhongBanActivity extends AppCompatActivity {              // Khai báo class kế thừa AppCompatActivity

    private TextView tvTitle;                                              // TextView hiển thị tiêu đề (Thêm/Sửa)
    private EditText etMaPhongBan, etTenPhongBan;                          // EditText cho mã và tên phòng ban
    private Spinner spTruongPhong;                                         // Spinner chọn trưởng phòng
    private Switch swTrangThai;                                            // Switch bật/tắt trạng thái hoạt động
    private Button btnLuu, btnHuy;                                         // Button lưu và hủy
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private boolean isEditMode = false;                                    // Flag kiểm tra chế độ sửa hay thêm mới
    private String originalMaPhongBan;                                     // Mã phòng ban gốc (dùng cho chế độ sửa)
    private List<String> listMaNhanVien;                                   // List chứa mã nhân viên có thể làm trưởng phòng
    private List<String> listTenNhanVien;                                  // List chứa tên nhân viên để hiển thị trong Spinner
```
#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_them_phong_ban);                  // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database helper
        checkEditMode();                                                   // Kiểm tra chế độ thêm mới hay sửa
        setupManagerSpinner();                                             // Thiết lập Spinner chọn trưởng phòng
        setupButtons();                                                    // Thiết lập các button
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        tvTitle = findViewById(R.id.tv_title);                             // Ánh xạ TextView tiêu đề
        etMaPhongBan = findViewById(R.id.et_ma_phong_ban);                 // Ánh xạ EditText mã phòng ban
        etTenPhongBan = findViewById(R.id.et_ten_phong_ban);               // Ánh xạ EditText tên phòng ban
        spTruongPhong = findViewById(R.id.sp_truong_phong);                // Ánh xạ Spinner trưởng phòng
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
            tvTitle.setText("SỬA PHÒNG BAN");                              // Set tiêu đề là "SỬA PHÒNG BAN"
            
            originalMaPhongBan = getIntent().getStringExtra("ma_phong_ban"); // Lấy mã phòng ban gốc từ Intent
            String tenPhongBan = getIntent().getStringExtra("ten_phong_ban"); // Lấy tên phòng ban từ Intent
            String truongPhong = getIntent().getStringExtra("truong_phong"); // Lấy mã trưởng phòng từ Intent
            int trangThai = getIntent().getIntExtra("trang_thai", 1);      // Lấy trạng thái từ Intent, mặc định là 1
            
            etMaPhongBan.setText(originalMaPhongBan);                      // Hiển thị mã phòng ban trong EditText
            etMaPhongBan.setEnabled(false); // Không cho sửa mã phòng ban  // Disable EditText mã phòng ban (không cho sửa)
            etTenPhongBan.setText(tenPhongBan);                            // Hiển thị tên phòng ban trong EditText
            swTrangThai.setChecked(trangThai == 1);                        // Set trạng thái Switch (true nếu trangThai = 1)
            
            btnLuu.setText("CẬP NHẬT");                                    // Đổi text button thành "CẬP NHẬT"
        } else {                                                           // Nếu là chế độ thêm mới
            tvTitle.setText("THÊM PHÒNG BAN MỚI");                         // Set tiêu đề là "THÊM PHÒNG BAN MỚI"
            
            // Tự động tạo mã phòng ban
            String nextCode = dbHelper.getNextDepartmentCode();            // Gọi method database để lấy mã phòng ban tiếp theo
            etMaPhongBan.setText(nextCode);                                // Hiển thị mã tự động trong EditText
            etMaPhongBan.setEnabled(false); // Không cho sửa mã tự động    // Disable EditText mã phòng ban (không cho sửa mã tự động)
            
            swTrangThai.setChecked(true); // Mặc định hoạt động            // Set Switch mặc định là true (hoạt động)
            btnLuu.setText("THÊM PHÒNG BAN");                              // Set text button là "THÊM PHÒNG BAN"
        }
    }
```

#### Method setupManagerSpinner:
```java
    private void setupManagerSpinner() {                                   // Method thiết lập Spinner chọn trưởng phòng
        try {                                                              // Bắt đầu try-catch để xử lý exception
            listMaNhanVien = new ArrayList<>();                            // Khởi tạo ArrayList chứa mã nhân viên
            listTenNhanVien = new ArrayList<>();                           // Khởi tạo ArrayList chứa tên nhân viên để hiển thị
            
            // Thêm option "Không có trưởng phòng"
            listMaNhanVien.add("");                                        // Thêm mã rỗng cho option "Không có trưởng phòng"
            listTenNhanVien.add("Không có trưởng phòng");                  // Thêm text hiển thị cho option đầu tiên
            
            // Lấy danh sách nhân viên có thể làm trưởng phòng (chức vụ Trưởng phòng hoặc Giám đốc)
            Cursor cursor = dbHelper.getManagerCandidates();               // Gọi method database lấy danh sách ứng viên trưởng phòng
            if (cursor != null && cursor.moveToFirst()) {                  // Kiểm tra cursor không null và có dữ liệu
                do {                                                       // Vòng lặp do-while để duyệt qua tất cả record
                    String maNV = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));     // Lấy mã nhân viên từ cursor
                    String hoTen = cursor.getString(cursor.getColumnIndexOrThrow("HoTen"));         // Lấy họ tên từ cursor
                    String tenChucVu = cursor.getString(cursor.getColumnIndexOrThrow("TenChucVu")); // Lấy tên chức vụ từ cursor
                    
                    listMaNhanVien.add(maNV);                              // Thêm mã nhân viên vào list
                    listTenNhanVien.add(maNV + " - " + hoTen + " (" + tenChucVu + ")"); // Thêm text hiển thị với format: Mã - Tên (Chức vụ)
                } while (cursor.moveToNext());                             // Tiếp tục vòng lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor để giải phóng bộ nhớ
            }
            
            ArrayAdapter<String> adapter = new ArrayAdapter<>(this,        // Tạo ArrayAdapter cho Spinner
                android.R.layout.simple_spinner_item, listTenNhanVien);    // Sử dụng layout mặc định và list tên nhân viên
            adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
            spTruongPhong.setAdapter(adapter);                             // Set adapter cho Spinner
            
            // Nếu là edit mode, chọn trưởng phòng hiện tại
            if (isEditMode) {                                              // Kiểm tra nếu là chế độ sửa
                String currentTruongPhong = getIntent().getStringExtra("truong_phong"); // Lấy mã trưởng phòng hiện tại từ Intent
                if (currentTruongPhong != null) {                          // Kiểm tra mã trưởng phòng không null
                    int position = listMaNhanVien.indexOf(currentTruongPhong); // Tìm vị trí của mã trưởng phòng trong list
                    if (position >= 0) {                                   // Nếu tìm thấy vị trí
                        spTruongPhong.setSelection(position);              // Set selection cho Spinner tại vị trí đó
                    }
                }
            }
            
        } catch (Exception e) {                                            // Bắt exception nếu có lỗi
            e.printStackTrace();                                           // In stack trace để debug
            Toast.makeText(this, "Lỗi khi tải danh sách nhân viên", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```

#### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập các button
        btnLuu.setOnClickListener(v -> saveDepartment());                  // Set listener cho button lưu, gọi method saveDepartment
        btnHuy.setOnClickListener(v -> finish());                         // Set listener cho button hủy, đóng Activity
    }
```

#### Method saveDepartment:
```java
    private void saveDepartment() {                                        // Method lưu thông tin phòng ban
        String maPhongBan = etMaPhongBan.getText().toString().trim();      // Lấy mã phòng ban từ EditText và trim khoảng trắng
        String tenPhongBan = etTenPhongBan.getText().toString().trim();    // Lấy tên phòng ban từ EditText và trim khoảng trắng
        int truongPhongPosition = spTruongPhong.getSelectedItemPosition(); // Lấy vị trí được chọn trong Spinner
        String truongPhong = truongPhongPosition > 0 ? listMaNhanVien.get(truongPhongPosition) : null; // Lấy mã trưởng phòng nếu position > 0, ngược lại null
        int trangThai = swTrangThai.isChecked() ? 1 : 0;                   // Lấy trạng thái từ Switch: true = 1, false = 0
        
        // Validate
        if (tenPhongBan.isEmpty()) {                                       // Kiểm tra tên phòng ban có rỗng không
            Toast.makeText(this, "Vui lòng nhập tên phòng ban", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            etTenPhongBan.requestFocus();                                  // Focus vào EditText tên phòng ban
            return;                                                        // Thoát method
        }
        
        boolean success;                                                   // Biến lưu kết quả thành công/thất bại
        if (isEditMode) {                                                  // Nếu là chế độ sửa
            success = dbHelper.updateDepartment(originalMaPhongBan, tenPhongBan, truongPhong, trangThai); // Gọi method update database
        } else {                                                           // Nếu là chế độ thêm mới
            success = dbHelper.addDepartment(maPhongBan, tenPhongBan, truongPhong, trangThai); // Gọi method add database
        }
        
        if (success) {                                                     // Nếu thành công
            String message = isEditMode ? "Cập nhật phòng ban thành công" : "Thêm phòng ban thành công"; // Tạo message tương ứng
            Toast.makeText(this, message, Toast.LENGTH_SHORT).show();      // Hiển thị thông báo thành công
            setResult(RESULT_OK);                                          // Set kết quả OK để Activity cha biết
            finish();                                                      // Đóng Activity
        } else {                                                           // Nếu thất bại
            String message = isEditMode ? "Lỗi khi cập nhật phòng ban" : "Lỗi khi thêm phòng ban"; // Tạo message lỗi tương ứng
            Toast.makeText(this, message, Toast.LENGTH_SHORT).show();      // Hiển thị thông báo lỗi
        }
    }
}
```

---

## 4️⃣ ADAPTER - PhongBanAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/PhongBanAdapter.java`

### Mục đích:
Adapter quản lý hiển thị danh sách phòng ban trong ListView, xử lý sự kiện sửa/xóa.

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
import com.example.btl_mobile_qlns.models.PhongBan;                        // Import model PhongBan

import java.util.List;                                                     // Import List interface
```

#### Khai báo thuộc tính class:
```java
public class PhongBanAdapter extends BaseAdapter {                         // Khai báo class kế thừa BaseAdapter
    
    private Context context;                                               // Context của Activity sử dụng adapter
    private List<PhongBan> listPhongBan;                                   // List chứa danh sách phòng ban
    private String currentRole;                                            // Role của user hiện tại
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
```

#### Constructor:
```java
    public PhongBanAdapter(Context context, List<PhongBan> listPhongBan, String currentRole) { // Constructor khởi tạo adapter
        this.context = context;                                            // Gán context
        this.listPhongBan = listPhongBan;                                  // Gán list phòng ban
        this.currentRole = currentRole;                                    // Gán role hiện tại
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
    }
```

#### Method getCount:
```java
    @Override
    public int getCount() {                                                // Method trả về số lượng item trong list
        return listPhongBan != null ? listPhongBan.size() : 0;            // Trả về size của list, nếu null thì trả về 0
    }
```

#### Method getItem:
```java
    @Override
    public Object getItem(int position) {                                  // Method trả về item tại vị trí position
        return listPhongBan.get(position);                                 // Trả về object PhongBan tại vị trí position
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
            convertView = LayoutInflater.from(context).inflate(R.layout.item_phong_ban, parent, false); // Inflate layout item_phong_ban
        }
        
        PhongBan phongBan = listPhongBan.get(position);                    // Lấy object PhongBan tại vị trí position
        
        TextView tvMaPhongBan = convertView.findViewById(R.id.tv_ma_phong_ban);     // Ánh xạ TextView mã phòng ban
        TextView tvTenPhongBan = convertView.findViewById(R.id.tv_ten_phong_ban);   // Ánh xạ TextView tên phòng ban
        TextView tvTruongPhong = convertView.findViewById(R.id.tv_truong_phong);    // Ánh xạ TextView trưởng phòng
        TextView tvSoNhanVien = convertView.findViewById(R.id.tv_so_nhan_vien);     // Ánh xạ TextView số nhân viên
        TextView tvTrangThai = convertView.findViewById(R.id.tv_trang_thai);        // Ánh xạ TextView trạng thái
        LinearLayout layoutButtons = convertView.findViewById(R.id.layout_buttons); // Ánh xạ LinearLayout chứa các button
        Button btnSua = convertView.findViewById(R.id.btn_sua);                     // Ánh xạ Button sửa
        Button btnXoa = convertView.findViewById(R.id.btn_xoa);                     // Ánh xạ Button xóa
        
        // Hiển thị thông tin phòng ban
        tvMaPhongBan.setText("Mã: " + phongBan.getMaPhongBan());           // Set text hiển thị mã phòng ban
        tvTenPhongBan.setText(phongBan.getTenPhongBan());                  // Set text hiển thị tên phòng ban
        
        String truongPhongText = "Trưởng phòng: ";                         // Khởi tạo text trưởng phòng
        if (phongBan.getTenTruongPhong() != null && !phongBan.getTenTruongPhong().isEmpty()) { // Kiểm tra có tên trưởng phòng không
            truongPhongText += phongBan.getTenTruongPhong();               // Nếu có thì thêm tên trưởng phòng
        } else {                                                           // Nếu không có
            truongPhongText += "Chưa có";                                  // Hiển thị "Chưa có"
        }
        tvTruongPhong.setText(truongPhongText);                            // Set text hiển thị trưởng phòng
        
        tvSoNhanVien.setText("Số nhân viên: " + phongBan.getSoNhanVien()); // Set text hiển thị số nhân viên
        
        // Hiển thị trạng thái
        String trangThaiText = phongBan.getTrangThai() == 1 ? "Hoạt động" : "Ngừng hoạt động"; // Tạo text trạng thái dựa vào giá trị int
        tvTrangThai.setText("Trạng thái: " + trangThaiText);               // Set text hiển thị trạng thái
        
        // Thiết lập màu sắc cho trạng thái
        if (phongBan.getTrangThai() == 1) {                                // Nếu trạng thái = 1 (hoạt động)
            tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_green_dark)); // Set màu xanh lá
        } else {                                                           // Nếu trạng thái = 0 (ngừng hoạt động)
            tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_red_dark));   // Set màu đỏ
        }
        
        // Hiển thị nút sửa/xóa cho Admin và HR
        if ("Admin".equals(currentRole) || "HR".equals(currentRole)) {     // Kiểm tra quyền Admin hoặc HR
            layoutButtons.setVisibility(View.VISIBLE);                     // Hiển thị layout chứa các button
            
            btnSua.setOnClickListener(v -> editDepartment(phongBan));      // Set listener cho button sửa
            btnXoa.setOnClickListener(v -> showDeleteDialog(phongBan, position)); // Set listener cho button xóa
        } else {                                                           // Nếu không phải Admin/HR
            layoutButtons.setVisibility(View.GONE);                        // Ẩn layout chứa các button
        }
        
        return convertView;                                                // Trả về View đã được setup
    }
```

#### Method editDepartment:
```java
    private void editDepartment(PhongBan phongBan) {                       // Method xử lý sự kiện sửa phòng ban
        Intent intent = new Intent(context, ThemPhongBanActivity.class);   // Tạo Intent chuyển đến ThemPhongBanActivity
        intent.putExtra("role", currentRole);                             // Truyền role hiện tại qua Intent
        intent.putExtra("edit_mode", true);                               // Truyền flag edit_mode = true
        intent.putExtra("ma_phong_ban", phongBan.getMaPhongBan());        // Truyền mã phòng ban qua Intent
        intent.putExtra("ten_phong_ban", phongBan.getTenPhongBan());      // Truyền tên phòng ban qua Intent
        intent.putExtra("truong_phong", phongBan.getTruongPhong());       // Truyền mã trưởng phòng qua Intent
        intent.putExtra("trang_thai", phongBan.getTrangThai());           // Truyền trạng thái qua Intent
        
        if (context instanceof QuanLyPhongBanActivity) {                   // Kiểm tra context có phải QuanLyPhongBanActivity không
            ((QuanLyPhongBanActivity) context).startActivityForResult(intent, 1001); // Cast context và gọi startActivityForResult
        }
    }
```

#### Method showDeleteDialog:
```java
    private void showDeleteDialog(PhongBan phongBan, int position) {       // Method hiển thị dialog xác nhận xóa
        // Kiểm tra ràng buộc: Nếu còn nhân viên thì không cho xóa
        if (phongBan.getSoNhanVien() > 0) {                                // Kiểm tra số nhân viên > 0
            new AlertDialog.Builder(context)                               // Tạo AlertDialog builder
                .setTitle("Không thể xóa")                                 // Set tiêu đề dialog
                .setMessage("Phòng ban \"" + phongBan.getTenPhongBan() + "\" hiện đang có " +  // Set message thông báo
                           phongBan.getSoNhanVien() + " nhân viên đang làm việc.\n\n" +
                           "Vui lòng chuyển các nhân viên này sang phòng ban khác trước khi thực hiện xóa.")
                .setPositiveButton("Đã hiểu", null)                        // Set button "Đã hiểu" không có action
                .show();                                                   // Hiển thị dialog
            return;                                                        // Thoát method
        }

        new AlertDialog.Builder(context)                                   // Tạo AlertDialog builder cho xác nhận xóa
            .setTitle("Xóa phòng ban")                                     // Set tiêu đề dialog
            .setMessage("Bạn có chắc muốn xóa phòng ban \"" + phongBan.getTenPhongBan() + "\"?\n\n" + // Set message xác nhận
                       "Lưu ý: Phòng ban sẽ được đánh dấu là ngừng hoạt động thay vì xóa hoàn toàn.")
            .setPositiveButton("Xóa", (dialog, which) -> {                 // Set button "Xóa" với action
                boolean success = dbHelper.deleteDepartment(phongBan.getMaPhongBan()); // Gọi method xóa database
                if (success) {                                             // Nếu xóa thành công
                    Toast.makeText(context, "Đã xóa phòng ban thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                    refreshData();                                         // Refresh dữ liệu
                } else {                                                   // Nếu xóa thất bại
                    Toast.makeText(context, "Lỗi khi xóa phòng ban", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                }
            })
            .setNegativeButton("Hủy", null)                                // Set button "Hủy" không có action
            .show();                                                       // Hiển thị dialog
    }
```

#### Method refreshData:
```java
    private void refreshData() {                                           // Method refresh dữ liệu
        if (context instanceof QuanLyPhongBanActivity) {                   // Kiểm tra context có phải QuanLyPhongBanActivity không
            ((QuanLyPhongBanActivity) context).loadDepartments();          // Cast context và gọi method loadDepartments
        }
    }
```

#### Method updateData:
```java
    public void updateData(List<PhongBan> newData) {                       // Method cập nhật dữ liệu mới cho adapter
        this.listPhongBan = newData;                                       // Gán list mới
        notifyDataSetChanged();                                            // Thông báo adapter dữ liệu đã thay đổi để refresh UI
    }
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 5️⃣ LAYOUT DANH SÁCH - activity_quan_ly_phong_ban.xml

**Đường dẫn**: `app/src/main/res/layout/activity_quan_ly_phong_ban.xml`

### Mục đích:
Layout chính cho Activity quản lý danh sách phòng ban, bao gồm header, tìm kiếm và ListView.

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
            android:text="QUẢN LÝ PHÒNG BAN"                               <!-- Text mặc định hiển thị -->
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
            android:hint="Tìm kiếm phòng ban..."                           <!-- Hint text hiển thị khi rỗng -->
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

            <TextView                                                      <!-- TextView hiển thị tổng số phòng ban -->
                android:id="@+id/tv_tong_so"                               <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:layout_weight="1"                                  <!-- Weight 1 để chiếm không gian còn lại -->
                android:text="Tổng số: 0 phòng ban"                        <!-- Text mặc định hiển thị -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textColor="#666" />                                <!-- Màu chữ xám (#666) -->

            <Button                                                        <!-- Button thêm phòng ban mới -->
                android:id="@+id/btn_them_phong_ban"                       <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="40dp"                               <!-- Chiều cao cố định 40dp -->
                android:text="THÊM PHÒNG BAN"                              <!-- Text hiển thị trên button -->
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
    <!-- Danh sách phòng ban -->
    <ListView                                                              <!-- ListView hiển thị danh sách phòng ban -->
        android:id="@+id/lv_phong_ban"                                     <!-- ID để truy cập từ Java code -->
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

## 6️⃣ LAYOUT THÊM/SỬA - activity_them_phong_ban.xml

**Đường dẫn**: `app/src/main/res/layout/activity_them_phong_ban.xml`

### Mục đích:
Layout cho Activity thêm mới và chỉnh sửa phòng ban, bao gồm form nhập liệu và các button.

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
                android:text="THÊM PHÒNG BAN MỚI"                          <!-- Text mặc định hiển thị -->
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

            <!-- Mã phòng ban -->
            <TextView                                                      <!-- TextView label cho mã phòng ban -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Mã phòng ban"                                <!-- Text label hiển thị -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ đen nhạt (#333) -->
                android:layout_marginBottom="8dp" />                       <!-- Margin bottom 8dp -->

            <EditText                                                      <!-- EditText nhập mã phòng ban -->
                android:id="@+id/et_ma_phong_ban"                          <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp cho tất cả các cạnh -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="Mã phòng ban tự động"                        <!-- Hint text hiển thị khi rỗng -->
                android:enabled="false" />                                 <!-- Disabled (không cho chỉnh sửa) -->

            <!-- Tên phòng ban -->
            <TextView                                                      <!-- TextView label cho tên phòng ban -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Tên phòng ban *"                             <!-- Text label với dấu * bắt buộc -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ đen nhạt (#333) -->
                android:layout_marginBottom="8dp" />                       <!-- Margin bottom 8dp -->

            <EditText                                                      <!-- EditText nhập tên phòng ban -->
                android:id="@+id/et_ten_phong_ban"                         <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp cho tất cả các cạnh -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="Nhập tên phòng ban"                          <!-- Hint text hiển thị khi rỗng -->
                android:inputType="text" />                                <!-- Input type là text -->

            <!-- Trưởng phòng -->
            <TextView                                                      <!-- TextView label cho trưởng phòng -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Trưởng phòng"                                <!-- Text label hiển thị -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ đen nhạt (#333) -->
                android:layout_marginBottom="8dp" />                       <!-- Margin bottom 8dp -->

            <Spinner                                                       <!-- Spinner chọn trưởng phòng -->
                android:id="@+id/sp_truong_phong"                          <!-- ID để truy cập từ Java code -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Background custom từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp cho tất cả các cạnh -->
                android:layout_marginBottom="16dp" />                      <!-- Margin bottom 16dp -->

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
                android:text="THÊM PHÒNG BAN"                              <!-- Text mặc định hiển thị trên button -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:background="@drawable/btn_primary"                 <!-- Background custom từ drawable -->
                android:layout_marginStart="8dp" />                        <!-- Margin start 8dp -->

        </LinearLayout>

    </LinearLayout>

</ScrollView>
```

---

## 7️⃣ LAYOUT ITEM - item_phong_ban.xml

**Đường dẫn**: `app/src/main/res/layout/item_phong_ban.xml`

### Mục đích:
Layout cho từng item phòng ban trong ListView, hiển thị thông tin chi tiết và các button hành động.

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

        <TextView                                                          <!-- TextView hiển thị mã phòng ban -->
            android:id="@+id/tv_ma_phong_ban"                              <!-- ID để truy cập từ Java code -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa text -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Mã: PB001"                                       <!-- Text mặc định hiển thị -->
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

    <!-- Tên phòng ban -->
    <TextView                                                              <!-- TextView hiển thị tên phòng ban -->
        android:id="@+id/tv_ten_phong_ban"                                 <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Phòng Nhân sự"                                       <!-- Text mặc định hiển thị -->
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

        <TextView                                                          <!-- TextView hiển thị thông tin trưởng phòng -->
            android:id="@+id/tv_truong_phong"                              <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Trưởng phòng: Nguyễn Văn A"                      <!-- Text mặc định hiển thị -->
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
- `getAllDepartmentsWithDetails()`: Lấy danh sách phòng ban với thông tin chi tiết
- `searchDepartments(String keyword)`: Tìm kiếm phòng ban theo từ khóa
- `getNextDepartmentCode()`: Tạo mã phòng ban tự động tiếp theo
- `getManagerCandidates()`: Lấy danh sách ứng viên trưởng phòng
- `addDepartment()`: Thêm phòng ban mới
- `updateDepartment()`: Cập nhật thông tin phòng ban
- `deleteDepartment()`: Xóa phòng ban (soft delete)

### Navigation Flow:
1. **DashboardActivity** → **QuanLyPhongBanActivity** (với role)
2. **QuanLyPhongBanActivity** → **ThemPhongBanActivity** (thêm mới)
3. **PhongBanAdapter** → **ThemPhongBanActivity** (chỉnh sửa)
4. **ThemPhongBanActivity** → **QuanLyPhongBanActivity** (kết quả)

### Permission System:
- **Admin/HR**: Full quyền (xem, thêm, sửa, xóa)
- **Manager**: Chỉ xem danh sách
- **Employee**: Không có quyền truy cập

---

## 📝 KẾT LUẬN

Module Quản lý Phòng ban được thiết kế với kiến trúc MVC rõ ràng, phân quyền chặt chẽ và giao diện thân thiện. Hệ thống đảm bảo tính toàn vẹn dữ liệu thông qua các ràng buộc nghiệp vụ và cung cấp trải nghiệm người dùng mượt mà với các chức năng tìm kiếm, thêm/sửa/xóa phòng ban và quản lý trưởng phòng.