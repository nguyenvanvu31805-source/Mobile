# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG THÔNG TIN CÁ NHÂN

## 📋 TỔNG QUAN

Chức năng Thông tin cá nhân cho phép người dùng xem và chỉnh sửa thông tin cá nhân của mình, bao gồm thay đổi ảnh đại diện, cập nhật thông tin liên lạc và đổi mật khẩu.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── ThongTinCaNhanActivity.java          # Activity chính - Quản lý thông tin cá nhân
└── database/DatabaseHelper.java         # Xử lý database (methods liên quan)

app/src/main/res/layout/
├── activity_thong_tin_ca_nhan.xml       # Layout form thông tin cá nhân
└── dialog_doi_mat_khau.xml              # Layout dialog đổi mật khẩu
```

## 📊 NGHIỆP VỤ THÔNG TIN CÁ NHÂN

### 1. Quy trình nghiệp vụ:
- **Xem thông tin**: Hiển thị thông tin cá nhân của người dùng đang đăng nhập
- **Chỉnh sửa thông tin**: Cho phép cập nhật họ tên, ngày sinh, giới tính, SĐT, email
- **Thay đổi ảnh đại diện**: Upload ảnh mới từ gallery
- **Đổi mật khẩu**: Thay đổi mật khẩu đăng nhập với validation bảo mật
- **Phân quyền đặc biệt**: Admin chỉ xem, không được chỉnh sửa

### 2. Phân quyền:
- **Admin**: Chỉ xem thông tin, không thể chỉnh sửa
- **HR/Manager/Employee**: Có thể xem và chỉnh sửa thông tin cá nhân

---

## 📱 CHI TIẾT CÁC FILE
## 1️⃣ ACTIVITY CHÍNH - ThongTinCaNhanActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ThongTinCaNhanActivity.java`

### Mục đích:
Activity quản lý thông tin cá nhân của người dùng, cho phép xem, chỉnh sửa thông tin và đổi mật khẩu.

### Chi tiết code:

#### Khai báo và khởi tạo:

```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.DatePickerDialog;                                       // Import DatePickerDialog để chọn ngày
import android.content.Intent;                                             // Import Intent để chuyển Activity
import android.database.Cursor;                                            // Import Cursor để xử lý dữ liệu database
import android.net.Uri;                                                    // Import Uri để xử lý đường dẫn ảnh
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu
import android.view.LayoutInflater;                                        // Import LayoutInflater để inflate layout
import android.view.View;                                                  // Import View cơ bản
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.ImageView;                                           // Import ImageView widget
import android.widget.RadioButton;                                         // Import RadioButton widget
import android.widget.RadioGroup;                                          // Import RadioGroup widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.annotation.Nullable;                                       // Import annotation Nullable
import androidx.appcompat.app.AlertDialog;                                 // Import AlertDialog để hiển thị dialog
import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity để extend

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper

import java.util.Calendar;                                                 // Import Calendar để xử lý ngày tháng

public class ThongTinCaNhanActivity extends AppCompatActivity {            // Khai báo class extend AppCompatActivity

    private static final int PICK_IMAGE_REQUEST = 1;                       // Constant request code cho chọn ảnh
    
    // Khai báo các view components
    private TextView tvTitle, tvMaNV, tvVaiTro;                             // TextView hiển thị tiêu đề, mã NV, vai trò
    private EditText etHoTen, etNgaySinh, etSoDienThoai, etEmail;          // EditText cho các thông tin cá nhân
    private RadioGroup rgGioiTinh;                                          // RadioGroup cho giới tính
    private RadioButton rbNam, rbNu;                                        // RadioButton Nam và Nữ
    private Button btnCapNhat, btnDoiMatKhau;                               // Button cập nhật và đổi mật khẩu
    private ImageView ivAvatar;                                             // ImageView hiển thị ảnh đại diện
    
    // Khai báo các biến xử lý dữ liệu
    private DatabaseHelper dbHelper;                                        // Helper xử lý database
    private String currentUsername;                                         // Username hiện tại
    private String maNhanVien;                                              // Mã nhân viên
    private String currentRole;                                             // Vai trò hiện tại
    private String imageUri = null;                                         // URI ảnh đại diện
```

#### Phương thức onCreate():

```java
    @Override                                                              // Annotation override method từ AppCompatActivity
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_thong_tin_ca_nhan);               // Set layout XML cho Activity
        
        initViews();                                                       // Gọi method khởi tạo các view components
        setupDatabase();                                                   // Gọi method thiết lập database
        loadUserInfo();                                                    // Gọi method load thông tin người dùng
        setupButtons();                                                    // Gọi method thiết lập các button events
    }
```

#### Khởi tạo Views:

```java
    private void initViews() {                                             // Method private khởi tạo các view
        tvTitle = findViewById(R.id.tv_title);                             // Tìm TextView title theo ID
        tvMaNV = findViewById(R.id.tv_ma_nv);                              // Tìm TextView mã nhân viên theo ID
        tvVaiTro = findViewById(R.id.tv_vai_tro);                          // Tìm TextView vai trò theo ID
        etHoTen = findViewById(R.id.et_ho_ten);                            // Tìm EditText họ tên theo ID
        etNgaySinh = findViewById(R.id.et_ngay_sinh);                      // Tìm EditText ngày sinh theo ID
        etSoDienThoai = findViewById(R.id.et_so_dien_thoai);               // Tìm EditText số điện thoại theo ID
        etEmail = findViewById(R.id.et_email);                             // Tìm EditText email theo ID
        rgGioiTinh = findViewById(R.id.rg_gioi_tinh);                      // Tìm RadioGroup giới tính theo ID
        rbNam = findViewById(R.id.rb_nam);                                 // Tìm RadioButton Nam theo ID
        rbNu = findViewById(R.id.rb_nu);                                   // Tìm RadioButton Nữ theo ID
        btnCapNhat = findViewById(R.id.btn_cap_nhat);                      // Tìm Button cập nhật theo ID
        btnDoiMatKhau = findViewById(R.id.btn_doi_mat_khau);               // Tìm Button đổi mật khẩu theo ID
        ivAvatar = findViewById(R.id.iv_avatar);                           // Tìm ImageView avatar theo ID
    }
```

#### Thiết lập Database:

```java
    private void setupDatabase() {                                         // Method private thiết lập database
        dbHelper = new DatabaseHelper(this);                               // Khởi tạo DatabaseHelper với context hiện tại
        currentUsername = getIntent().getStringExtra("username");          // Lấy username từ Intent extras
        if (currentUsername != null) {                                     // Kiểm tra nếu username không null
            maNhanVien = dbHelper.getMaNhanVienByUsername(currentUsername); // Lấy mã nhân viên từ username
        }
    }
```

#### Load thông tin người dùng:

```java
    private void loadUserInfo() {                                          // Method private load thông tin người dùng
        if (currentUsername == null) return;                               // Return nếu username null
        
        // Lấy thông tin tài khoản
        Cursor cursorAccount = dbHelper.getUserInfo(currentUsername);       // Query thông tin tài khoản từ database
        if (cursorAccount != null && cursorAccount.moveToFirst()) {        // Kiểm tra cursor có dữ liệu
            String hoTen = cursorAccount.getString(cursorAccount.getColumnIndexOrThrow("HoTen")); // Lấy họ tên từ cursor
            currentRole = cursorAccount.getString(cursorAccount.getColumnIndexOrThrow("VaiTro")); // Lấy vai trò từ cursor
            
            tvTitle.setText("THÔNG TIN CÁ NHÂN");                          // Set text cho title
            tvVaiTro.setText("Vai trò: " + currentRole);                   // Set text vai trò với prefix
            
            cursorAccount.close();                                         // Đóng cursor để giải phóng bộ nhớ
        }
        
        // Nếu là Admin, chỉ hiển thị thông tin cơ bản
        if ("Admin".equals(currentRole)) {                                 // Kiểm tra nếu vai trò là Admin
            tvMaNV.setText("Mã: ADMIN");                                   // Set text mã Admin
            etHoTen.setText("Administrator");                              // Set text họ tên Admin
            etHoTen.setEnabled(false);                                     // Disable EditText họ tên
            etNgaySinh.setEnabled(false);                                  // Disable EditText ngày sinh
            etSoDienThoai.setEnabled(false);                               // Disable EditText số điện thoại
            etEmail.setEnabled(false);                                     // Disable EditText email
            rgGioiTinh.setEnabled(false);                                  // Disable RadioGroup giới tính
            btnCapNhat.setEnabled(false);                                  // Disable Button cập nhật
            btnCapNhat.setText("ADMIN - KHÔNG THỂ SỬA");                   // Thay đổi text button
            return;                                                        // Return để không thực hiện code phía dưới
        }
        
        // Lấy thông tin nhân viên chi tiết
        if (maNhanVien != null) {                                          // Kiểm tra nếu mã nhân viên không null
            Cursor cursorEmployee = dbHelper.getEmployeeByMa(maNhanVien);   // Query thông tin nhân viên từ database
            if (cursorEmployee != null && cursorEmployee.moveToFirst()) {   // Kiểm tra cursor có dữ liệu
                tvMaNV.setText("Mã: " + maNhanVien);                       // Set text mã nhân viên
                etHoTen.setText(cursorEmployee.getString(cursorEmployee.getColumnIndexOrThrow("HoTen"))); // Set họ tên
                etNgaySinh.setText(cursorEmployee.getString(cursorEmployee.getColumnIndexOrThrow("NgaySinh"))); // Set ngày sinh
                etSoDienThoai.setText(cursorEmployee.getString(cursorEmployee.getColumnIndexOrThrow("SoDienThoai"))); // Set SĐT
                etEmail.setText(cursorEmployee.getString(cursorEmployee.getColumnIndexOrThrow("Email"))); // Set email
                
                String gioiTinh = cursorEmployee.getString(cursorEmployee.getColumnIndexOrThrow("GioiTinh")); // Lấy giới tính
                if ("Nam".equals(gioiTinh)) {                              // Kiểm tra nếu giới tính là Nam
                    rbNam.setChecked(true);                                // Check RadioButton Nam
                } else {                                                   // Nếu không phải Nam
                    rbNu.setChecked(true);                                 // Check RadioButton Nữ
                }
                
                // Hiển thị ảnh đại diện
                imageUri = cursorEmployee.getString(cursorEmployee.getColumnIndexOrThrow("HinhAnh")); // Lấy URI ảnh
                if (imageUri != null && !imageUri.isEmpty()) {            // Kiểm tra nếu có URI ảnh
                    try {                                                  // Bắt đầu try-catch
                        ivAvatar.setImageURI(Uri.parse(imageUri));         // Parse URI và set ảnh
                    } catch (Exception e) {                                // Bắt exception nếu có lỗi
                        ivAvatar.setImageResource(R.drawable.ic_person);   // Set ảnh mặc định nếu lỗi
                    }
                }
                
                cursorEmployee.close();                                    // Đóng cursor để giải phóng bộ nhớ
            }
        }
    }
```

---
#### Thiết lập Button Events:

```java
    private void setupButtons() {                                          // Method private thiết lập sự kiện cho buttons
        // Date picker cho ngày sinh
        etNgaySinh.setOnClickListener(v -> showDatePicker());              // Set OnClickListener cho EditText ngày sinh (lambda)
        
        btnCapNhat.setOnClickListener(v -> capNhatThongTin());             // Set OnClickListener cho button cập nhật (lambda)
        
        btnDoiMatKhau.setOnClickListener(v -> showDoiMatKhauDialog());     // Set OnClickListener cho button đổi mật khẩu (lambda)

        ivAvatar.setOnClickListener(v -> openGallery());                   // Set OnClickListener cho ImageView avatar (lambda)
    }
```

#### Chọn ảnh từ Gallery:

```java
    private void openGallery() {                                           // Method private mở gallery chọn ảnh
        Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);           // Tạo Intent với action OPEN_DOCUMENT
        intent.addCategory(Intent.CATEGORY_OPENABLE);                      // Thêm category OPENABLE
        intent.setType("image/*");                                         // Set type chỉ chọn file ảnh
        startActivityForResult(intent, PICK_IMAGE_REQUEST);                // Start Activity với request code
    }

    @Override                                                              // Annotation override method
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) { // Method xử lý kết quả Activity
        super.onActivityResult(requestCode, resultCode, data);             // Gọi method của class cha
        if (requestCode == PICK_IMAGE_REQUEST && resultCode == RESULT_OK && // Kiểm tra request code và result code
            data != null && data.getData() != null) {                     // Kiểm tra data không null
            Uri uri = data.getData();                                      // Lấy URI từ data
            getContentResolver().takePersistableUriPermission(uri, Intent.FLAG_GRANT_READ_URI_PERMISSION); // Xin quyền truy cập lâu dài
            imageUri = uri.toString();                                     // Chuyển URI thành string
            ivAvatar.setImageURI(uri);                                     // Set ảnh cho ImageView
        }
    }
```

#### Dialog đổi mật khẩu:

```java
    private void showDoiMatKhauDialog() {                                  // Method private hiển thị dialog đổi mật khẩu
        // Inflate layout dialog
        View dialogView = LayoutInflater.from(this).inflate(R.layout.dialog_doi_mat_khau, null); // Inflate layout dialog từ XML
        
        EditText etMatKhauCu = dialogView.findViewById(R.id.et_mat_khau_cu);     // Tìm EditText mật khẩu cũ
        EditText etMatKhauMoi = dialogView.findViewById(R.id.et_mat_khau_moi);   // Tìm EditText mật khẩu mới
        EditText etXacNhanMatKhau = dialogView.findViewById(R.id.et_xac_nhan_mat_khau); // Tìm EditText xác nhận mật khẩu
        
        AlertDialog dialog = new AlertDialog.Builder(this)                // Tạo AlertDialog Builder
                .setView(dialogView)                                       // Set view cho dialog
                .setPositiveButton("Xác nhận", null)                      // Set nút positive (null để tự xử lý)
                .setNegativeButton("Hủy", null)                           // Set nút negative
                .create();                                                 // Tạo dialog
        
        dialog.show();                                                     // Hiển thị dialog
        
        // Override nút Xác nhận để kiểm tra validation trước khi đóng dialog
        dialog.getButton(AlertDialog.BUTTON_POSITIVE).setOnClickListener(v -> { // Set OnClickListener cho nút positive
            String matKhauCu = etMatKhauCu.getText().toString().trim();    // Lấy text mật khẩu cũ và trim
            String matKhauMoi = etMatKhauMoi.getText().toString().trim();  // Lấy text mật khẩu mới và trim
            String xacNhanMatKhau = etXacNhanMatKhau.getText().toString().trim(); // Lấy text xác nhận mật khẩu và trim
            
            // Validate mật khẩu hiện tại
            if (matKhauCu.isEmpty()) {                                     // Kiểm tra mật khẩu cũ rỗng
                etMatKhauCu.setError("Vui lòng nhập mật khẩu hiện tại");   // Set error message
                etMatKhauCu.requestFocus();                                // Focus vào EditText
                return;                                                    // Return để không thực hiện tiếp
            }
            
            // Validate mật khẩu mới
            if (matKhauMoi.isEmpty()) {                                    // Kiểm tra mật khẩu mới rỗng
                etMatKhauMoi.setError("Vui lòng nhập mật khẩu mới");       // Set error message
                etMatKhauMoi.requestFocus();                               // Focus vào EditText
                return;                                                    // Return để không thực hiện tiếp
            }
            
            if (matKhauMoi.length() < 6) {                                 // Kiểm tra độ dài mật khẩu mới
                etMatKhauMoi.setError("Mật khẩu mới phải có ít nhất 6 ký tự"); // Set error message
                etMatKhauMoi.requestFocus();                               // Focus vào EditText
                return;                                                    // Return để không thực hiện tiếp
            }
            
            // Validate xác nhận mật khẩu
            if (xacNhanMatKhau.isEmpty()) {                                // Kiểm tra xác nhận mật khẩu rỗng
                etXacNhanMatKhau.setError("Vui lòng xác nhận mật khẩu mới"); // Set error message
                etXacNhanMatKhau.requestFocus();                           // Focus vào EditText
                return;                                                    // Return để không thực hiện tiếp
            }
            
            if (!matKhauMoi.equals(xacNhanMatKhau)) {                      // Kiểm tra mật khẩu mới và xác nhận có khớp
                etXacNhanMatKhau.setError("Mật khẩu xác nhận không khớp"); // Set error message
                etXacNhanMatKhau.requestFocus();                           // Focus vào EditText
                return;                                                    // Return để không thực hiện tiếp
            }
            
            // Kiểm tra mật khẩu mới không trùng mật khẩu cũ
            if (matKhauCu.equals(matKhauMoi)) {                            // Kiểm tra mật khẩu cũ và mới có trùng
                etMatKhauMoi.setError("Mật khẩu mới không được trùng mật khẩu cũ"); // Set error message
                etMatKhauMoi.requestFocus();                               // Focus vào EditText
                return;                                                    // Return để không thực hiện tiếp
            }
            
            // Kiểm tra mật khẩu hiện tại có đúng không
            if (!dbHelper.checkCurrentPassword(currentUsername, matKhauCu)) { // Gọi method kiểm tra mật khẩu hiện tại
                etMatKhauCu.setError("Mật khẩu hiện tại không đúng");      // Set error message
                etMatKhauCu.requestFocus();                                // Focus vào EditText
                return;                                                    // Return để không thực hiện tiếp
            }
            
            // Thực hiện đổi mật khẩu
            boolean success = dbHelper.changePassword(currentUsername, matKhauMoi); // Gọi method đổi mật khẩu
            if (success) {                                                 // Kiểm tra nếu thành công
                Toast.makeText(ThongTinCaNhanActivity.this,                // Hiển thị Toast thành công
                        "Đổi mật khẩu thành công!", Toast.LENGTH_SHORT).show();
                dialog.dismiss();                                          // Đóng dialog
            } else {                                                       // Nếu thất bại
                Toast.makeText(ThongTinCaNhanActivity.this,                // Hiển thị Toast lỗi
                        "Lỗi khi đổi mật khẩu. Vui lòng thử lại!", Toast.LENGTH_SHORT).show();
            }
        });
    }
```

#### DatePicker cho ngày sinh:

```java
    private void showDatePicker() {                                        // Method private hiển thị DatePicker
        Calendar calendar = Calendar.getInstance();                        // Lấy instance Calendar hiện tại
        int year = calendar.get(Calendar.YEAR);                            // Lấy năm hiện tại
        int month = calendar.get(Calendar.MONTH);                          // Lấy tháng hiện tại
        int day = calendar.get(Calendar.DAY_OF_MONTH);                     // Lấy ngày hiện tại
        
        DatePickerDialog datePickerDialog = new DatePickerDialog(this,    // Tạo DatePickerDialog
            (view, year1, month1, dayOfMonth) -> {                         // Lambda expression xử lý kết quả
                String date = String.format("%d-%02d-%02d", year1, month1 + 1, dayOfMonth); // Format ngày theo yyyy-mm-dd
                etNgaySinh.setText(date);                                  // Set text cho EditText ngày sinh
            }, year, month, day);                                          // Truyền năm, tháng, ngày mặc định
        datePickerDialog.show();                                           // Hiển thị DatePickerDialog
    }
```

#### Cập nhật thông tin:

```java
    private void capNhatThongTin() {                                       // Method private cập nhật thông tin
        if (maNhanVien == null || "Admin".equals(currentRole)) {          // Kiểm tra nếu mã NV null hoặc là Admin
            Toast.makeText(this, "Không thể cập nhật thông tin", Toast.LENGTH_SHORT).show(); // Hiển thị Toast lỗi
            return;                                                        // Return để không thực hiện tiếp
        }
        
        String hoTen = etHoTen.getText().toString().trim();                // Lấy họ tên và trim
        String ngaySinh = etNgaySinh.getText().toString().trim();          // Lấy ngày sinh và trim
        String soDienThoai = etSoDienThoai.getText().toString().trim();    // Lấy số điện thoại và trim
        String email = etEmail.getText().toString().trim();               // Lấy email và trim
        String gioiTinh = rbNam.isChecked() ? "Nam" : "Nữ";               // Lấy giới tính từ RadioButton (ternary operator)
        
        if (hoTen.isEmpty()) {                                             // Validate họ tên không rỗng
            etHoTen.setError("Vui lòng nhập họ tên");                      // Set error message
            etHoTen.requestFocus();                                        // Focus vào EditText
            return;                                                        // Return để không thực hiện tiếp
        }

        if (ngaySinh.isEmpty()) {                                          // Validate ngày sinh không rỗng
            etNgaySinh.setError("Vui lòng chọn ngày sinh");                // Set error message
            etNgaySinh.requestFocus();                                     // Focus vào EditText
            return;                                                        // Return để không thực hiện tiếp
        }

        if (soDienThoai.length() != 10) {                                  // Validate số điện thoại đúng 10 số
            etSoDienThoai.setError("Số điện thoại phải có đúng 10 chữ số"); // Set error message
            etSoDienThoai.requestFocus();                                  // Focus vào EditText
            return;                                                        // Return để không thực hiện tiếp
        }

        if (!android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()) { // Validate định dạng email
            etEmail.setError("Định dạng Email không hợp lệ");              // Set error message
            etEmail.requestFocus();                                        // Focus vào EditText
            return;                                                        // Return để không thực hiện tiếp
        }
        
        boolean success = dbHelper.updateEmployeePersonalInfo(maNhanVien, hoTen, ngaySinh, // Gọi method cập nhật thông tin
                                                               gioiTinh, soDienThoai, email, imageUri);
        
        if (success) {                                                     // Kiểm tra nếu cập nhật thành công
            Toast.makeText(this, "Cập nhật thông tin thành công", Toast.LENGTH_SHORT).show(); // Hiển thị Toast thành công
        } else {                                                           // Nếu cập nhật thất bại
            Toast.makeText(this, "Lỗi khi cập nhật thông tin", Toast.LENGTH_SHORT).show(); // Hiển thị Toast lỗi
        }
    }
}
```

### Đặc điểm của Activity:
- **Role-based Access**: Phân quyền rõ ràng cho Admin và các role khác
- **Comprehensive Validation**: Kiểm tra đầy đủ các trường dữ liệu
- **Security**: Validation mật khẩu với nhiều điều kiện bảo mật
- **User Experience**: DatePicker, Image picker, Error handling tốt
- **Modern Android**: Sử dụng lambda expressions, proper resource management

---
## 2️⃣ LAYOUT CHÍNH - activity_thong_tin_ca_nhan.xml

**Đường dẫn**: `app/src/main/res/layout/activity_thong_tin_ca_nhan.xml`

### Mục đích:
Layout form hiển thị và chỉnh sửa thông tin cá nhân với giao diện thân thiện người dùng.

### Chi tiết code:

```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Container chính sắp xếp theo chiều dọc -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho custom attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent -->
    android:orientation="vertical"                                         <!-- Sắp xếp các view con theo chiều dọc -->
    android:padding="16dp"                                                 <!-- Padding 16dp cho tất cả các cạnh -->
    android:background="#f5f5f5">                                          <!-- Màu nền xám nhạt -->

    <!-- Header -->
    <TextView                                                              <!-- Text hiển thị tiêu đề -->
        android:id="@+id/tv_title"                                         <!-- Tạo ID để truy cập từ Java -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ text -->
        android:text="THÔNG TIN CÁ NHÂN"                                   <!-- Nội dung text tiêu đề -->
        android:textSize="24sp"                                            <!-- Kích thước font 24sp -->
        android:textStyle="bold"                                           <!-- Kiểu chữ đậm -->
        android:textColor="#2196F3"                                        <!-- Màu chữ xanh Material -->
        android:gravity="center"                                           <!-- Căn giữa text -->
        android:layout_marginBottom="20dp" />                              <!-- Margin bottom 20dp -->

    <ScrollView                                                            <!-- Container có thể scroll -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="0dp"                                        <!-- Chiều cao = 0 (sử dụng weight) -->
        android:layout_weight="1">                                         <!-- Chiếm toàn bộ không gian còn lại -->

        <LinearLayout                                                      <!-- Container con sắp xếp theo chiều dọc -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ nội dung -->
            android:orientation="vertical"                                 <!-- Sắp xếp theo chiều dọc -->
            android:paddingBottom="20dp">                                  <!-- Padding bottom 20dp -->

            <LinearLayout                                                  <!-- Container cho phần avatar -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ nội dung -->
                android:orientation="vertical"                             <!-- Sắp xếp theo chiều dọc -->
                android:gravity="center">                                  <!-- Căn giữa nội dung -->

            <!-- Khung chọn ảnh đại diện -->
            <FrameLayout                                                   <!-- Container cho phép overlay views -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ nội dung -->
                android:layout_height="wrap_content">                      <!-- Chiều cao vừa đủ nội dung -->
                
                <ImageView                                                 <!-- View hiển thị ảnh đại diện -->
                    android:id="@+id/iv_avatar"                            <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="120dp"                           <!-- Chiều rộng 120dp -->
                    android:layout_height="120dp"                          <!-- Chiều cao 120dp -->
                    android:src="@drawable/ic_person"                      <!-- Ảnh mặc định từ drawable -->
                    android:scaleType="centerCrop"                         <!-- Cắt ảnh để vừa khung -->
                    android:background="@drawable/card_feature"            <!-- Nền từ drawable -->
                    android:padding="2dp" />                               <!-- Padding 2dp -->
                    
                <ImageView                                                 <!-- Icon overlay chỉ dẫn -->
                    android:layout_width="28dp"                            <!-- Chiều rộng 28dp -->
                    android:layout_height="28dp"                           <!-- Chiều cao 28dp -->
                    android:layout_gravity="bottom|end"                    <!-- Định vị góc dưới phải -->
                    android:src="@android:drawable/ic_menu_camera"         <!-- Icon camera từ Android system -->
                    app:tint="#2196F3" />                                  <!-- Tô màu xanh -->
            </FrameLayout>

            <TextView                                                      <!-- Text hướng dẫn -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Nhấn vào ảnh để thay đổi"                    <!-- Nội dung hướng dẫn -->
                android:textSize="12sp"                                    <!-- Kích thước font 12sp -->
                android:textColor="#888"                                   <!-- Màu chữ xám -->
                android:layout_marginTop="8dp"                             <!-- Margin top 8dp -->
                android:layout_gravity="center" />                         <!-- Căn giữa -->
        </LinearLayout>
            
            <View                                                          <!-- Đường phân cách -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="1dp"                                <!-- Chiều cao 1dp -->
                android:background="#ddd"                                  <!-- Màu nền xám nhạt -->
                android:layout_marginTop="16dp"                            <!-- Margin top 16dp -->
                android:layout_marginBottom="16dp" />                      <!-- Margin bottom 16dp -->

            <!-- Thông tin cơ bản -->
            <LinearLayout                                                  <!-- Container ngang cho thông tin cơ bản -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ nội dung -->
                android:orientation="horizontal"                           <!-- Sắp xếp theo chiều ngang -->
                android:layout_marginBottom="16dp">                        <!-- Margin bottom 16dp -->

                <TextView                                                  <!-- Text hiển thị mã nhân viên -->
                    android:id="@+id/tv_ma_nv"                             <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="0dp"                             <!-- Chiều rộng = 0 (sử dụng weight) -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ text -->
                    android:layout_weight="1"                              <!-- Chiếm 1 phần trong layout weight -->
                    android:text="Mã: "                                    <!-- Nội dung text với prefix -->
                    android:textSize="16sp"                                <!-- Kích thước font 16sp -->
                    android:textStyle="bold"                               <!-- Kiểu chữ đậm -->
                    android:textColor="#333" />                            <!-- Màu chữ xám đậm -->

                <TextView                                                  <!-- Text hiển thị vai trò -->
                    android:id="@+id/tv_vai_tro"                           <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="0dp"                             <!-- Chiều rộng = 0 (sử dụng weight) -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ text -->
                    android:layout_weight="1"                              <!-- Chiếm 1 phần trong layout weight -->
                    android:text="Vai trò: "                               <!-- Nội dung text với prefix -->
                    android:textSize="16sp"                                <!-- Kích thước font 16sp -->
                    android:textStyle="bold"                               <!-- Kiểu chữ đậm -->
                    android:textColor="#333"                               <!-- Màu chữ xám đậm -->
                    android:gravity="end" />                               <!-- Căn phải text -->

            </LinearLayout>

            <!-- Họ tên -->
            <TextView                                                      <!-- Label cho họ tên -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Họ và tên"                                   <!-- Nội dung label -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#666"                                   <!-- Màu chữ xám -->
                android:layout_marginBottom="4dp" />                       <!-- Margin bottom 4dp -->

            <EditText                                                      <!-- Input field cho họ tên -->
                android:id="@+id/et_ho_ten"                                <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Nền từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="Nhập họ và tên" />                           <!-- Placeholder text -->

            <!-- Ngày sinh -->
            <TextView                                                      <!-- Label cho ngày sinh -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Ngày sinh"                                   <!-- Nội dung label -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#666"                                   <!-- Màu chữ xám -->
                android:layout_marginBottom="4dp" />                       <!-- Margin bottom 4dp -->

            <EditText                                                      <!-- Input field cho ngày sinh -->
                android:id="@+id/et_ngay_sinh"                             <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Nền từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="YYYY-MM-DD"                                  <!-- Placeholder text với format -->
                android:focusable="false"                                  <!-- Không cho phép focus (chỉ click) -->
                android:clickable="true" />                                <!-- Cho phép click để mở DatePicker -->

            <!-- Giới tính -->
            <TextView                                                      <!-- Label cho giới tính -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Giới tính"                                   <!-- Nội dung label -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#666"                                   <!-- Màu chữ xám -->
                android:layout_marginBottom="4dp" />                       <!-- Margin bottom 4dp -->

            <RadioGroup                                                    <!-- Group cho RadioButton giới tính -->
                android:id="@+id/rg_gioi_tinh"                             <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ nội dung -->
                android:orientation="horizontal"                           <!-- Sắp xếp theo chiều ngang -->
                android:layout_marginBottom="16dp">                        <!-- Margin bottom 16dp -->

                <RadioButton                                               <!-- RadioButton cho Nam -->
                    android:id="@+id/rb_nam"                               <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="wrap_content"                    <!-- Chiều rộng vừa đủ nội dung -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:text="Nam"                                     <!-- Text hiển thị -->
                    android:layout_marginEnd="32dp" />                     <!-- Margin end 32dp -->

                <RadioButton                                               <!-- RadioButton cho Nữ -->
                    android:id="@+id/rb_nu"                                <!-- Tạo ID để truy cập từ Java -->
                    android:layout_width="wrap_content"                    <!-- Chiều rộng vừa đủ nội dung -->
                    android:layout_height="wrap_content"                   <!-- Chiều cao vừa đủ nội dung -->
                    android:text="Nữ" />                                   <!-- Text hiển thị -->

            </RadioGroup>

            <!-- Số điện thoại -->
            <TextView                                                      <!-- Label cho số điện thoại -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Số điện thoại"                               <!-- Nội dung label -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#666"                                   <!-- Màu chữ xám -->
                android:layout_marginBottom="4dp" />                       <!-- Margin bottom 4dp -->

            <EditText                                                      <!-- Input field cho số điện thoại -->
                android:id="@+id/et_so_dien_thoai"                         <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Nền từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="16dp"                         <!-- Margin bottom 16dp -->
                android:hint="Nhập số điện thoại"                          <!-- Placeholder text -->
                android:inputType="phone"                                  <!-- Kiểu input cho số điện thoại -->
                android:maxLength="10" />                                  <!-- Giới hạn tối đa 10 ký tự -->

            <!-- Email -->
            <TextView                                                      <!-- Label cho email -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ text -->
                android:text="Email"                                       <!-- Nội dung label -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#666"                                   <!-- Màu chữ xám -->
                android:layout_marginBottom="4dp" />                       <!-- Margin bottom 4dp -->

            <EditText                                                      <!-- Input field cho email -->
                android:id="@+id/et_email"                                 <!-- Tạo ID để truy cập từ Java -->
                android:layout_width="match_parent"                        <!-- Chiều rộng bằng parent -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:background="@drawable/edit_text_background"        <!-- Nền từ drawable -->
                android:padding="12dp"                                     <!-- Padding 12dp -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:layout_marginBottom="24dp"                         <!-- Margin bottom 24dp -->
                android:hint="Nhập email"                                  <!-- Placeholder text -->
                android:inputType="textEmailAddress" />                   <!-- Kiểu input cho email -->

        </LinearLayout>

    </ScrollView>

    <!-- Buttons -->
    <LinearLayout                                                          <!-- Container cho các button -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ nội dung -->
        android:orientation="vertical"                                     <!-- Sắp xếp theo chiều dọc -->
        android:layout_marginTop="16dp">                                   <!-- Margin top 16dp -->

        <Button                                                            <!-- Button cập nhật thông tin -->
            android:id="@+id/btn_cap_nhat"                                 <!-- Tạo ID để truy cập từ Java -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:text="CẬP NHẬT THÔNG TIN"                              <!-- Text trên button -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:textStyle="bold"                                       <!-- Kiểu chữ đậm -->
            android:background="@drawable/btn_primary"                     <!-- Nền từ drawable -->
            android:layout_marginBottom="12dp" />                          <!-- Margin bottom 12dp -->

        <Button                                                            <!-- Button đổi mật khẩu -->
            android:id="@+id/btn_doi_mat_khau"                             <!-- Tạo ID để truy cập từ Java -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:text="ĐỔI MẬT KHẨU"                                    <!-- Text trên button -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh -->
            android:textStyle="bold"                                       <!-- Kiểu chữ đậm -->
            android:background="@drawable/btn_secondary"                   <!-- Nền từ drawable -->
            android:layout_marginBottom="12dp" />                          <!-- Margin bottom 12dp -->

    </LinearLayout>

</LinearLayout>
```

**Đặc điểm layout:**
- **ScrollView**: Cho phép scroll khi nội dung dài
- **Responsive Design**: Sử dụng weight và margin phù hợp
- **User-friendly**: Labels rõ ràng, placeholder text hướng dẫn
- **Material Design**: Màu sắc và spacing theo chuẩn Material
- **Accessibility**: InputType phù hợp cho từng trường dữ liệu

---

## 🔧 TÍCH HỢP VỚI DATABASE

### Các phương thức DatabaseHelper liên quan:

```java
// Trong DatabaseHelper.java
public Cursor getUserInfo(String username)                    // Lấy thông tin tài khoản
public String getMaNhanVienByUsername(String username)        // Lấy mã NV từ username  
public Cursor getEmployeeByMa(String maNhanVien)             // Lấy thông tin nhân viên theo mã
public boolean updateEmployeePersonalInfo(...)               // Cập nhật thông tin cá nhân
public boolean checkCurrentPassword(String username, String password) // Kiểm tra mật khẩu hiện tại
public boolean changePassword(String username, String newPassword)    // Đổi mật khẩu
```

### SQL Query quan trọng:

```sql
-- Query lấy thông tin tài khoản
SELECT tk.*, nv.HoTen 
FROM TaiKhoan tk 
LEFT JOIN NhanVien nv ON tk.MaNhanVien = nv.MaNhanVien 
WHERE tk.TenDangNhap = ?

-- Query cập nhật thông tin cá nhân
UPDATE NhanVien 
SET HoTen = ?, NgaySinh = ?, GioiTinh = ?, SoDienThoai = ?, Email = ?, HinhAnh = ?
WHERE MaNhanVien = ?

-- Query đổi mật khẩu
UPDATE TaiKhoan 
SET MatKhau = ? 
WHERE TenDangNhap = ?
```

---

## 📋 TỔNG KẾT

### Luồng hoạt động chính:

1. **Khởi động**: ThongTinCaNhanActivity nhận username từ Intent
2. **Load dữ liệu**: Query thông tin tài khoản và nhân viên từ database
3. **Phân quyền**: Kiểm tra role và disable/enable các field tương ứng
4. **Chỉnh sửa**: Người dùng có thể thay đổi thông tin và ảnh đại diện
5. **Validation**: Kiểm tra dữ liệu trước khi cập nhật
6. **Đổi mật khẩu**: Dialog riêng với validation bảo mật cao
7. **Cập nhật**: Lưu thông tin mới vào database

### Ưu điểm của thiết kế:

- **Security First**: Validation mật khẩu nghiêm ngặt, phân quyền rõ ràng
- **User Experience**: Giao diện thân thiện, DatePicker, Image picker
- **Data Integrity**: Validation đầy đủ cho tất cả các trường
- **Role-based Access**: Admin không thể chỉnh sửa, các role khác có quyền đầy đủ
- **Modern UI**: Material Design, responsive layout
- **Error Handling**: Xử lý lỗi và hiển thị thông báo phù hợp

### Điểm cần cải thiện:

- **Image Compression**: Nén ảnh trước khi lưu để tiết kiệm dung lượng
- **Offline Support**: Cache dữ liệu để hoạt động offline
- **Biometric Authentication**: Thêm xác thực sinh trắc học cho đổi mật khẩu
- **Password Strength**: Thêm meter đánh giá độ mạnh mật khẩu
- **Audit Trail**: Log các thay đổi thông tin để theo dõi

---

*Tài liệu này cung cấp cái nhìn toàn diện về module Thông tin cá nhân trong hệ thống QLNS. Mọi thắc mắc về implementation chi tiết có thể tham khảo source code hoặc liên hệ team phát triển.*