# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ CHẤM CÔNG

## 📋 TỔNG QUAN

Chức năng Quản lý Chấm công là một module quan trọng của hệ thống QLNS, cho phép nhân viên thực hiện chấm công vào/ra và quản lý theo dõi thời gian làm việc. Đồng thời cung cấp cho Admin/HR/Manager khả năng quản lý, chỉnh sửa và theo dõi dữ liệu chấm công của tất cả nhân viên.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── ChamCongActivity.java                # Activity chính - Chấm công và quản lý
├── ChamCongAdapter.java                 # Adapter hiển thị lịch sử chấm công
├── models/ChamCong.java                 # Model class chấm công
└── database/DatabaseHelper.java         # Xử lý database

app/src/main/res/layout/
├── activity_cham_cong.xml               # Layout chính chấm công
├── item_cham_cong.xml                   # Layout item lịch sử
└── dialog_edit_cham_cong.xml            # Layout dialog chỉnh sửa
```

## 📊 NGHIỆP VỤ QUẢN LÝ CHẤM CÔNG

### 1. Quy trình nghiệp vụ:
- **Chấm công cá nhân**: Tất cả nhân viên có thể chấm công vào/ra
- **Hiển thị thời gian thực**: Đồng hồ thời gian thực cập nhật mỗi giây
- **Kiểm tra trạng thái**: Hiển thị trạng thái chấm công hôm nay
- **Lịch sử chấm công**: Xem lịch sử 30 ngày gần nhất
- **Quản lý dữ liệu**: Admin/HR/Manager có thể sửa/xóa dữ liệu chấm công
- **Tính giờ tăng ca**: Tự động tính giờ tăng ca (> 8 giờ/ngày)
- **Ghi chú**: Thêm ghi chú cho từng lần chấm công

### 2. Phân quyền:
- **All roles**: Chấm công cá nhân, xem lịch sử cá nhân
- **Admin/HR/Manager**: Quản lý chấm công tất cả nhân viên, sửa/xóa dữ liệu
- **Employee**: Chỉ chấm công và xem lịch sử cá nhân

### 3. Tính năng đặc biệt:
- **Real-time clock**: Cập nhật thời gian mỗi giây
- **Overtime calculation**: Tính giờ tăng ca tự động
- **Time picker**: Chọn thời gian bằng dialog picker
- **Status management**: Quản lý trạng thái "Có mặt", "Vắng mặt"
- **Employee filtering**: Lọc theo nhân viên (Admin/HR/Manager)

---

## 📱 CHI TIẾT CÁC FILE
## 1️⃣ MODEL CLASS - ChamCong.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/models/ChamCong.java`

### Mục đích:
Model class đại diện cho đối tượng Chấm công trong hệ thống, chứa các thuộc tính và phương thức getter/setter.

### Chi tiết code:

#### Khai báo package và class:
```java
package com.example.btl_mobile_qlns.models;                               // Khai báo package chứa các model class

public class ChamCong {                                                    // Khai báo class ChamCong public để có thể truy cập từ package khác
```

#### Khai báo thuộc tính:
```java
    private String ngayChamCong;                                           // Ngày chấm công (yyyy-MM-dd)
    private String gioVao;                                                 // Giờ vào làm (HH:mm:ss)
    private String gioRa;                                                  // Giờ ra về (HH:mm:ss)
    private double soGioLam;                                               // Số giờ làm việc trong ngày
    private String trangThai;                                              // Trạng thái: "Có mặt", "Vắng mặt"
    private String maNhanVien;                                             // Mã nhân viên (để hiển thị cho Admin/Manager)
    private String hoTen;                                                  // Họ tên nhân viên (để hiển thị cho Admin/Manager)
    private String ghiChu;                                                 // Ghi chú thêm (tăng ca, nghỉ phép, v.v.)
```

#### Constructor:
```java
    public ChamCong(String ngayChamCong, String gioVao, String gioRa, double soGioLam, String trangThai) {  // Constructor khởi tạo với 5 tham số chính
        this.ngayChamCong = ngayChamCong;                                  // Gán ngày chấm công
        this.gioVao = gioVao;                                              // Gán giờ vào
        this.gioRa = gioRa;                                                // Gán giờ ra
        this.soGioLam = soGioLam;                                          // Gán số giờ làm
        this.trangThai = trangThai;                                        // Gán trạng thái
        this.ghiChu = "";                                                  // Khởi tạo ghi chú rỗng
    }
```

#### Getter methods:
```java
    public String getNgayChamCong() { return ngayChamCong; }               // Getter lấy ngày chấm công
    public String getGioVao() { return gioVao; }                          // Getter lấy giờ vào
    public String getGioRa() { return gioRa; }                            // Getter lấy giờ ra
    public double getSoGioLam() { return soGioLam; }                       // Getter lấy số giờ làm
    public String getTrangThai() { return trangThai; }                     // Getter lấy trạng thái
    public String getMaNhanVien() { return maNhanVien; }                   // Getter lấy mã nhân viên
    public String getHoTen() { return hoTen; }                             // Getter lấy họ tên
    public String getGhiChu() { return ghiChu; }                           // Getter lấy ghi chú
```

#### Setter methods:
```java
    public void setNgayChamCong(String ngayChamCong) { this.ngayChamCong = ngayChamCong; }     // Setter cập nhật ngày chấm công
    public void setGioVao(String gioVao) { this.gioVao = gioVao; }                             // Setter cập nhật giờ vào
    public void setGioRa(String gioRa) { this.gioRa = gioRa; }                                 // Setter cập nhật giờ ra
    public void setSoGioLam(double soGioLam) { this.soGioLam = soGioLam; }                     // Setter cập nhật số giờ làm
    public void setTrangThai(String trangThai) { this.trangThai = trangThai; }                 // Setter cập nhật trạng thái
    public void setMaNhanVien(String maNhanVien) { this.maNhanVien = maNhanVien; }             // Setter cập nhật mã nhân viên
    public void setHoTen(String hoTen) { this.hoTen = hoTen; }                                 // Setter cập nhật họ tên
    public void setGhiChu(String ghiChu) { this.ghiChu = ghiChu; }                             // Setter cập nhật ghi chú
}
```

---

## 2️⃣ ACTIVITY CHÍNH - ChamCongActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ChamCongActivity.java`

### Mục đích:
Activity chính quản lý chấm công, bao gồm chấm công cá nhân và quản lý dữ liệu chấm công cho Admin/Manager.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.view.View;                                                  // Import View để xử lý UI components
import android.widget.Button;                                              // Import Button widget
import android.widget.ListView;                                            // Import ListView widget
import android.widget.Spinner;                                             // Import Spinner widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo
import android.widget.ArrayAdapter;                                        // Import ArrayAdapter cho Spinner

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
import com.example.btl_mobile_qlns.models.ChamCong;                        // Import model ChamCong

import java.text.SimpleDateFormat;                                         // Import SimpleDateFormat để format ngày giờ
import java.util.ArrayList;                                                // Import ArrayList để lưu danh sách
import java.util.Date;                                                     // Import Date để xử lý thời gian
import java.util.List;                                                     // Import List interface
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
```

#### Khai báo thuộc tính class:
```java
public class ChamCongActivity extends AppCompatActivity {                  // Khai báo class kế thừa AppCompatActivity

    private TextView tvCurrentTime, tvCurrentDate, tvStatus, tvTitle;      // TextView hiển thị thời gian, ngày, trạng thái, tiêu đề
    private Button btnChamCongVao, btnChamCongRa;                          // Button chấm công vào và ra
    private ListView lvLichSuChamCong;                                     // ListView hiển thị lịch sử chấm công
    private Spinner spNhanVien;                                            // Spinner chọn nhân viên (cho Admin/Manager)
    private View layoutChamCongCaNhan, layoutQuanLyChamCong;               // Layout cho chấm công cá nhân và quản lý
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private String currentUsername;                                        // Username của user hiện tại
    private String maNhanVien;                                             // Mã nhân viên của user hiện tại
    private String currentRole;                                            // Role của user hiện tại
    private SimpleDateFormat timeFormat, dateFormat;                       // Format cho thời gian và ngày
    private ChamCongAdapter adapter;                                       // Adapter cho ListView
    private List<ChamCong> listChamCong;                                   // List chứa danh sách chấm công
    private List<String> listMaNhanVien;                                   // List chứa mã nhân viên cho Spinner
    private List<String> listTenNhanVien;                                  // List chứa tên nhân viên cho Spinner
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_cham_cong);                       // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database helper
        setupFormats();                                                    // Thiết lập format ngày giờ
        setupUI();                                                         // Thiết lập giao diện người dùng
        updateCurrentTime();                                               // Bắt đầu cập nhật thời gian thực
        loadTodayStatus();                                                 // Tải trạng thái chấm công hôm nay
        loadAttendanceHistory();                                           // Tải lịch sử chấm công
        setupButtons();                                                    // Thiết lập các button
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        tvTitle = findViewById(R.id.tv_title);                             // Ánh xạ TextView tiêu đề
        tvCurrentTime = findViewById(R.id.tv_current_time);                // Ánh xạ TextView thời gian hiện tại
        tvCurrentDate = findViewById(R.id.tv_current_date);                // Ánh xạ TextView ngày hiện tại
        tvStatus = findViewById(R.id.tv_status);                           // Ánh xạ TextView trạng thái chấm công
        btnChamCongVao = findViewById(R.id.btn_cham_cong_vao);             // Ánh xạ Button chấm công vào
        btnChamCongRa = findViewById(R.id.btn_cham_cong_ra);               // Ánh xạ Button chấm công ra
        lvLichSuChamCong = findViewById(R.id.lv_lich_su_cham_cong);        // Ánh xạ ListView lịch sử chấm công
        spNhanVien = findViewById(R.id.sp_nhan_vien);                      // Ánh xạ Spinner chọn nhân viên
        layoutChamCongCaNhan = findViewById(R.id.layout_cham_cong_ca_nhan); // Ánh xạ Layout chấm công cá nhân
        layoutQuanLyChamCong = findViewById(R.id.layout_quan_ly_cham_cong); // Ánh xạ Layout quản lý chấm công
    }
```

#### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập database helper
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        currentUsername = getIntent().getStringExtra("username");         // Lấy username từ Intent được truyền từ Activity trước
        currentRole = getIntent().getStringExtra("role");                 // Lấy role từ Intent được truyền từ Activity trước
        
        if (currentUsername != null) {                                     // Kiểm tra username không null
            maNhanVien = dbHelper.getMaNhanVienByUsername(currentUsername); // Lấy mã nhân viên từ username
        }
    }
```

#### Method setupFormats:
```java
    private void setupFormats() {                                          // Method thiết lập format ngày giờ
        timeFormat = new SimpleDateFormat("HH:mm:ss", Locale.getDefault()); // Format thời gian: giờ:phút:giây
        dateFormat = new SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()); // Format ngày: năm-tháng-ngày
    }
```

#### Method setupUI:
```java
    private void setupUI() {                                               // Method thiết lập giao diện người dùng
        // Tất cả role đều có thể chấm công cá nhân
        tvTitle.setText("CHẤM CÔNG");                                      // Set text cho tiêu đề
        layoutChamCongCaNhan.setVisibility(View.VISIBLE);                  // Hiển thị layout chấm công cá nhân cho tất cả role
        
        // Admin/HR/Manager có thêm chức năng quản lý
        if (!"Employee".equals(currentRole)) {                             // Kiểm tra nếu không phải Employee
            layoutQuanLyChamCong.setVisibility(View.VISIBLE);              // Hiển thị layout quản lý chấm công
            setupEmployeeSpinner();                                        // Thiết lập Spinner chọn nhân viên
        } else {                                                           // Nếu là Employee
            layoutQuanLyChamCong.setVisibility(View.GONE);                 // Ẩn layout quản lý chấm công
        }
    }
```

#### Method setupEmployeeSpinner:
```java
    private void setupEmployeeSpinner() {                                  // Method thiết lập Spinner chọn nhân viên
        try {                                                              // Bắt đầu try-catch để xử lý exception
            listMaNhanVien = new ArrayList<>();                            // Khởi tạo ArrayList chứa mã nhân viên
            listTenNhanVien = new ArrayList<>();                           // Khởi tạo ArrayList chứa tên nhân viên để hiển thị
            
            // Thêm option "Tất cả nhân viên"
            listMaNhanVien.add("ALL");                                     // Thêm mã "ALL" cho option tất cả
            listTenNhanVien.add("Tất cả nhân viên");                       // Thêm text hiển thị cho option đầu tiên
            
            Cursor cursor = dbHelper.getAllEmployees();                    // Gọi method database lấy tất cả nhân viên
            if (cursor != null && cursor.moveToFirst()) {                  // Kiểm tra cursor không null và có dữ liệu
                do {                                                       // Vòng lặp do-while để duyệt qua tất cả record
                    String maNV = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));     // Lấy mã nhân viên từ cursor
                    String hoTen = cursor.getString(cursor.getColumnIndexOrThrow("HoTen"));         // Lấy họ tên từ cursor
                    
                    if (maNV != null && hoTen != null) {                   // Kiểm tra dữ liệu không null
                        listMaNhanVien.add(maNV);                          // Thêm mã nhân viên vào list
                        listTenNhanVien.add(maNV + " - " + hoTen);         // Thêm text hiển thị với format: Mã - Tên
                    }
                } while (cursor.moveToNext());                             // Tiếp tục vòng lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor để giải phóng bộ nhớ
            }
            
            if (spNhanVien != null && listTenNhanVien != null && !listTenNhanVien.isEmpty()) { // Kiểm tra Spinner và list không null/rỗng
                ArrayAdapter<String> adapter = new ArrayAdapter<>(this,    // Tạo ArrayAdapter cho Spinner
                    android.R.layout.simple_spinner_item, listTenNhanVien); // Sử dụng layout mặc định và list tên nhân viên
                adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item); // Set layout cho dropdown
                spNhanVien.setAdapter(adapter);                            // Set adapter cho Spinner
                
                // Listener để load dữ liệu khi chọn nhân viên
                spNhanVien.setOnItemSelectedListener(new android.widget.AdapterView.OnItemSelectedListener() { // Set listener cho Spinner
                    @Override
                    public void onItemSelected(android.widget.AdapterView<?> parent, View view, int position, long id) { // Method được gọi khi chọn item
                        try {                                              // Bắt đầu try-catch để xử lý exception
                            if (listMaNhanVien != null && position >= 0 && position < listMaNhanVien.size()) { // Kiểm tra vị trí hợp lệ
                                loadAttendanceHistory();                   // Tải lại lịch sử chấm công theo nhân viên được chọn
                            }
                        } catch (Exception e) {                            // Bắt exception nếu có lỗi
                            e.printStackTrace();                           // In stack trace để debug
                            Toast.makeText(ChamCongActivity.this, "Lỗi khi tải dữ liệu chấm công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                        }
                    }
                    
                    @Override
                    public void onNothingSelected(android.widget.AdapterView<?> parent) {} // Method được gọi khi không chọn gì (không sử dụng)
                });
            }
        } catch (Exception e) {                                            // Bắt exception nếu có lỗi
            e.printStackTrace();                                           // In stack trace để debug
            Toast.makeText(this, "Lỗi khi tải danh sách nhân viên", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```
#### Method updateCurrentTime:
```java
    private void updateCurrentTime() {                                     // Method cập nhật thời gian hiện tại
        Date now = new Date();                                             // Tạo object Date với thời gian hiện tại
        tvCurrentTime.setText(timeFormat.format(now));                    // Set text hiển thị thời gian với format HH:mm:ss
        tvCurrentDate.setText(dateFormat.format(now));                    // Set text hiển thị ngày với format yyyy-MM-dd
        
        // Cập nhật thời gian mỗi giây
        tvCurrentTime.postDelayed(this::updateCurrentTime, 1000);         // Gọi lại method này sau 1000ms (1 giây) để cập nhật liên tục
    }
```

#### Method loadTodayStatus:
```java
    private void loadTodayStatus() {                                       // Method tải trạng thái chấm công hôm nay
        if (maNhanVien == null) return;                                    // Kiểm tra mã nhân viên không null, nếu null thì thoát
        
        String today = dateFormat.format(new Date());                     // Lấy ngày hôm nay với format yyyy-MM-dd
        boolean[] status = dbHelper.getTodayAttendanceStatus(maNhanVien, today); // Gọi method database lấy trạng thái chấm công hôm nay
        
        boolean hasCheckedIn = status[0];                                  // Lấy trạng thái đã chấm công vào
        boolean hasCheckedOut = status[1];                                 // Lấy trạng thái đã chấm công ra
        
        if (!hasCheckedIn) {                                               // Nếu chưa chấm công vào
            tvStatus.setText("Chưa chấm công vào");                        // Hiển thị trạng thái chưa chấm công vào
            btnChamCongVao.setEnabled(true);                               // Enable button chấm công vào
            btnChamCongRa.setEnabled(false);                               // Disable button chấm công ra
        } else if (!hasCheckedOut) {                                       // Nếu đã chấm công vào nhưng chưa chấm công ra
            tvStatus.setText("Đã chấm công vào - Chưa chấm công ra");      // Hiển thị trạng thái đã vào chưa ra
            btnChamCongVao.setEnabled(false);                              // Disable button chấm công vào
            btnChamCongRa.setEnabled(true);                                // Enable button chấm công ra
        } else {                                                           // Nếu đã chấm công đầy đủ
            tvStatus.setText("Đã hoàn thành chấm công hôm nay");           // Hiển thị trạng thái hoàn thành
            btnChamCongVao.setEnabled(false);                              // Disable button chấm công vào
            btnChamCongRa.setEnabled(false);                               // Disable button chấm công ra
        }
    }
```

#### Method loadAttendanceHistory:
```java
    public void loadAttendanceHistory() {                                  // Method tải lịch sử chấm công (public để Adapter có thể gọi)
        try {                                                              // Bắt đầu try-catch để xử lý exception
            listChamCong = new ArrayList<>();                              // Khởi tạo ArrayList mới để chứa danh sách chấm công
            Cursor cursor = null;                                          // Khai báo biến cursor
            
            if ("Employee".equals(currentRole)) {                          // Nếu là Employee
                // Employee: Chỉ xem lịch sử của mình
                if (maNhanVien == null) return;                            // Kiểm tra mã nhân viên không null
                cursor = dbHelper.getAttendanceHistory(maNhanVien, 30);    // Lấy lịch sử chấm công 30 ngày của nhân viên hiện tại
            } else {                                                       // Nếu không phải Employee (Admin/HR/Manager)
                // Admin/HR/Manager: Xem theo nhân viên được chọn
                if (spNhanVien == null || listMaNhanVien == null || listMaNhanVien.isEmpty()) { // Kiểm tra Spinner chưa sẵn sàng
                    // Fallback: load tất cả nếu spinner chưa sẵn sàng
                    cursor = dbHelper.getAllAttendanceHistory(30);         // Lấy tất cả lịch sử chấm công 30 ngày
                } else {                                                   // Nếu Spinner đã sẵn sàng
                    int selectedPosition = spNhanVien.getSelectedItemPosition(); // Lấy vị trí được chọn trong Spinner
                    if (selectedPosition == 0) {                           // Nếu chọn "Tất cả nhân viên" (vị trí 0)
                        // Tất cả nhân viên
                        cursor = dbHelper.getAllAttendanceHistory(30);     // Lấy tất cả lịch sử chấm công 30 ngày
                    } else if (selectedPosition > 0 && selectedPosition < listMaNhanVien.size()) { // Nếu chọn nhân viên cụ thể
                        String selectedMaNV = listMaNhanVien.get(selectedPosition); // Lấy mã nhân viên được chọn
                        if (selectedMaNV != null && !selectedMaNV.isEmpty()) { // Kiểm tra mã nhân viên không null/rỗng
                            cursor = dbHelper.getAttendanceHistory(selectedMaNV, 30); // Lấy lịch sử chấm công của nhân viên được chọn
                        } else {                                           // Nếu mã nhân viên null/rỗng
                            cursor = dbHelper.getAllAttendanceHistory(30); // Fallback: lấy tất cả
                        }
                    } else {                                               // Nếu vị trí không hợp lệ
                        cursor = dbHelper.getAllAttendanceHistory(30);     // Fallback: lấy tất cả
                    }
                }
            }

            if (cursor != null && cursor.moveToFirst()) {                  // Kiểm tra cursor không null và có dữ liệu
                do {                                                       // Vòng lặp do-while để duyệt qua tất cả record
                    try {                                                  // Bắt đầu try-catch cho từng record
                        String maNV = null;                                // Biến lưu mã nhân viên
                        String hoTen = null;                               // Biến lưu họ tên
                        
                        // Lấy thông tin nhân viên nếu không phải Employee
                        if (!"Employee".equals(currentRole)) {             // Nếu không phải Employee
                            int maNVIndex = cursor.getColumnIndex("MaNhanVien"); // Lấy index của cột MaNhanVien
                            if (maNVIndex >= 0) {                          // Kiểm tra index hợp lệ
                                maNV = cursor.getString(maNVIndex);         // Lấy mã nhân viên từ cursor
                                if (maNV != null && !maNV.isEmpty()) {     // Kiểm tra mã nhân viên không null/rỗng
                                    hoTen = dbHelper.getEmployeeNameByMa(maNV); // Lấy họ tên từ mã nhân viên
                                }
                            }
                        }
                        
                        int ngayIndex = cursor.getColumnIndex("NgayChamCong");     // Lấy index của cột NgayChamCong
                        int gioVaoIndex = cursor.getColumnIndex("GioVao");         // Lấy index của cột GioVao
                        int gioRaIndex = cursor.getColumnIndex("GioRa");           // Lấy index của cột GioRa
                        int soGioIndex = cursor.getColumnIndex("SoGioLam");        // Lấy index của cột SoGioLam
                        int trangThaiIndex = cursor.getColumnIndex("TrangThai");   // Lấy index của cột TrangThai
                        int ghiChuIndex = cursor.getColumnIndex("GhiChu");         // Lấy index của cột GhiChu
                        
                        if (ngayIndex >= 0 && gioVaoIndex >= 0 && gioRaIndex >= 0 && // Kiểm tra tất cả index cần thiết hợp lệ
                            soGioIndex >= 0 && trangThaiIndex >= 0) {
                            
                            String ngay = cursor.getString(ngayIndex);             // Lấy ngày chấm công từ cursor
                            String gioVao = cursor.getString(gioVaoIndex);         // Lấy giờ vào từ cursor
                            String gioRa = cursor.getString(gioRaIndex);           // Lấy giờ ra từ cursor
                            double soGio = cursor.getDouble(soGioIndex);           // Lấy số giờ làm từ cursor
                            String trangThai = cursor.getString(trangThaiIndex);   // Lấy trạng thái từ cursor
                            String ghiChu = ghiChuIndex >= 0 ? cursor.getString(ghiChuIndex) : ""; // Lấy ghi chú từ cursor (nếu có)

                            ChamCong chamCong = new ChamCong(ngay, gioVao, gioRa, soGio, trangThai); // Tạo object ChamCong với constructor
                            if (maNV != null) {                            // Nếu có mã nhân viên
                                chamCong.setMaNhanVien(maNV);              // Set mã nhân viên
                                chamCong.setHoTen(hoTen);                  // Set họ tên
                            }
                            chamCong.setGhiChu(ghiChu);                    // Set ghi chú
                            
                            listChamCong.add(chamCong);                    // Thêm object ChamCong vào list
                        }
                    } catch (Exception e) {                                // Bắt exception cho từng record
                        e.printStackTrace();                               // In stack trace để debug
                        // Tiếp tục với record tiếp theo
                    }
                } while (cursor.moveToNext());                             // Tiếp tục vòng lặp đến record tiếp theo
                cursor.close();                                            // Đóng cursor để giải phóng bộ nhớ
            }

            if (adapter == null) {                                         // Kiểm tra adapter chưa được khởi tạo
                adapter = new ChamCongAdapter(this, listChamCong, currentRole); // Tạo adapter mới với context, data và role
                lvLichSuChamCong.setAdapter(adapter);                      // Set adapter cho ListView
            } else {                                                       // Nếu adapter đã tồn tại
                adapter.updateData(listChamCong);                          // Cập nhật data cho adapter hiện tại
            }
        } catch (Exception e) {                                            // Bắt exception nếu có lỗi
            e.printStackTrace();                                           // In stack trace để debug
            Toast.makeText(this, "Lỗi khi tải lịch sử chấm công: " + e.getMessage(), Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```

#### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập các button
        btnChamCongVao.setOnClickListener(v -> chamCongVao());             // Set listener cho button chấm công vào
        btnChamCongRa.setOnClickListener(v -> chamCongRa());               // Set listener cho button chấm công ra
    }
```

#### Method chamCongVao:
```java
    private void chamCongVao() {                                           // Method xử lý chấm công vào
        if (maNhanVien == null) {                                          // Kiểm tra mã nhân viên không null
            Toast.makeText(this, "Không tìm thấy thông tin nhân viên", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        Date now = new Date();                                             // Tạo object Date với thời gian hiện tại
        String today = dateFormat.format(now);                            // Format ngày hiện tại
        String currentTime = timeFormat.format(now);                      // Format thời gian hiện tại
        
        boolean success = dbHelper.chamCongVao(maNhanVien, today, currentTime); // Gọi method database chấm công vào
        
        if (success) {                                                     // Nếu chấm công thành công
            Toast.makeText(this, "Chấm công vào thành công: " + currentTime, Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
            loadTodayStatus();                                             // Tải lại trạng thái hôm nay
            loadAttendanceHistory(); // Refresh lịch sử                    // Tải lại lịch sử chấm công
        } else {                                                           // Nếu chấm công thất bại
            Toast.makeText(this, "Lỗi khi chấm công vào", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```

#### Method chamCongRa:
```java
    private void chamCongRa() {                                            // Method xử lý chấm công ra
        if (maNhanVien == null) {                                          // Kiểm tra mã nhân viên không null
            Toast.makeText(this, "Không tìm thấy thông tin nhân viên", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        Date now = new Date();                                             // Tạo object Date với thời gian hiện tại
        String today = dateFormat.format(now);                            // Format ngày hiện tại
        String currentTime = timeFormat.format(now);                      // Format thời gian hiện tại
        
        boolean success = dbHelper.chamCongRa(maNhanVien, today, currentTime); // Gọi method database chấm công ra
        
        if (success) {                                                     // Nếu chấm công thành công
            Toast.makeText(this, "Chấm công ra thành công: " + currentTime, Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
            loadTodayStatus();                                             // Tải lại trạng thái hôm nay
            loadAttendanceHistory(); // Refresh lịch sử                    // Tải lại lịch sử chấm công
        } else {                                                           // Nếu chấm công thất bại
            Toast.makeText(this, "Lỗi khi chấm công ra", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
}
```

---

## 3️⃣ ADAPTER - ChamCongAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/ChamCongAdapter.java`

### Mục đích:
Adapter quản lý hiển thị danh sách lịch sử chấm công trong ListView, xử lý sự kiện sửa/xóa cho Admin/Manager.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.AlertDialog;                                            // Import AlertDialog để hiển thị dialog xác nhận
import android.app.TimePickerDialog;                                       // Import TimePickerDialog để chọn thời gian
import android.content.Context;                                            // Import Context để truy cập resources và services
import android.view.LayoutInflater;                                        // Import LayoutInflater để inflate layout
import android.view.View;                                                  // Import View để xử lý UI components
import android.view.ViewGroup;                                             // Import ViewGroup để quản lý layout container
import android.widget.BaseAdapter;                                         // Import BaseAdapter làm base class
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.LinearLayout;                                        // Import LinearLayout widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
import com.example.btl_mobile_qlns.models.ChamCong;                        // Import model ChamCong

import java.util.Calendar;                                                 // Import Calendar để xử lý thời gian
import java.util.List;                                                     // Import List interface
```

#### Khai báo thuộc tính class:
```java
public class ChamCongAdapter extends BaseAdapter {                         // Khai báo class kế thừa BaseAdapter
    
    private Context context;                                               // Context của Activity sử dụng adapter
    private List<ChamCong> listChamCong;                                   // List chứa danh sách chấm công
    private String currentRole;                                            // Role của user hiện tại
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
```

#### Constructor:
```java
    public ChamCongAdapter(Context context, List<ChamCong> listChamCong, String currentRole) { // Constructor khởi tạo adapter với role
        this.context = context;                                            // Gán context
        this.listChamCong = listChamCong;                                  // Gán list chấm công
        this.currentRole = currentRole;                                    // Gán role hiện tại
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
    }
    
    // Constructor cũ để tương thích
    public ChamCongAdapter(Context context, List<ChamCong> listChamCong) { // Constructor khởi tạo adapter không có role (tương thích ngược)
        this.context = context;                                            // Gán context
        this.listChamCong = listChamCong;                                  // Gán list chấm công
        this.currentRole = "Employee";                                     // Mặc định role là Employee
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
    }
```

#### Method getCount:
```java
    @Override
    public int getCount() {                                                // Method trả về số lượng item trong list
        return listChamCong != null ? listChamCong.size() : 0;            // Trả về size của list, nếu null thì trả về 0
    }
```

#### Method getItem:
```java
    @Override
    public Object getItem(int position) {                                  // Method trả về item tại vị trí position
        return listChamCong.get(position);                                 // Trả về object ChamCong tại vị trí position
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
            convertView = LayoutInflater.from(context).inflate(R.layout.item_cham_cong, parent, false); // Inflate layout item_cham_cong
        }
        
        ChamCong chamCong = listChamCong.get(position);                    // Lấy object ChamCong tại vị trí position
        
        TextView tvMaNV = convertView.findViewById(R.id.tv_ma_nv);          // Ánh xạ TextView mã nhân viên
        TextView tvHoTen = convertView.findViewById(R.id.tv_ho_ten);        // Ánh xạ TextView họ tên
        TextView tvNgay = convertView.findViewById(R.id.tv_ngay);           // Ánh xạ TextView ngày
        TextView tvGioVao = convertView.findViewById(R.id.tv_gio_vao);      // Ánh xạ TextView giờ vào
        TextView tvGioRa = convertView.findViewById(R.id.tv_gio_ra);        // Ánh xạ TextView giờ ra
        TextView tvSoGio = convertView.findViewById(R.id.tv_so_gio);        // Ánh xạ TextView số giờ
        TextView tvTrangThai = convertView.findViewById(R.id.tv_trang_thai); // Ánh xạ TextView trạng thái
        TextView tvGhiChu = convertView.findViewById(R.id.tv_ghi_chu);      // Ánh xạ TextView ghi chú
        LinearLayout layoutButtons = convertView.findViewById(R.id.layout_buttons); // Ánh xạ LinearLayout chứa các button
        Button btnSua = convertView.findViewById(R.id.btn_sua);             // Ánh xạ Button sửa
        Button btnXoa = convertView.findViewById(R.id.btn_xoa);             // Ánh xạ Button xóa
        
        // Hiển thị thông tin nhân viên nếu không phải Employee
        if (!"Employee".equals(currentRole) && chamCong.getMaNhanVien() != null) { // Kiểm tra không phải Employee và có mã nhân viên
            tvMaNV.setText("Mã NV: " + chamCong.getMaNhanVien());          // Set text hiển thị mã nhân viên
            tvHoTen.setText("Họ tên: " + (chamCong.getHoTen() != null ? chamCong.getHoTen() : "N/A")); // Set text hiển thị họ tên (hoặc N/A nếu null)
            tvMaNV.setVisibility(View.VISIBLE);                            // Hiển thị TextView mã nhân viên
            tvHoTen.setVisibility(View.VISIBLE);                           // Hiển thị TextView họ tên
        } else {                                                           // Nếu là Employee hoặc không có mã nhân viên
            tvMaNV.setVisibility(View.GONE);                               // Ẩn TextView mã nhân viên
            tvHoTen.setVisibility(View.GONE);                              // Ẩn TextView họ tên
        }
        
        tvNgay.setText("Ngày: " + chamCong.getNgayChamCong());             // Set text hiển thị ngày chấm công
        tvGioVao.setText("Giờ vào: " + (chamCong.getGioVao() != null ? chamCong.getGioVao() : "Chưa chấm")); // Set text hiển thị giờ vào (hoặc "Chưa chấm" nếu null)
        tvGioRa.setText("Giờ ra: " + (chamCong.getGioRa() != null ? chamCong.getGioRa() : "Chưa chấm")); // Set text hiển thị giờ ra (hoặc "Chưa chấm" nếu null)
        
        // Tính giờ tăng ca
        double soGioLam = chamCong.getSoGioLam();                          // Lấy số giờ làm việc
        double gioTangCa = dbHelper.tinhGioTangCa(soGioLam);               // Tính giờ tăng ca từ database helper
        
        if (gioTangCa > 0) {                                               // Nếu có giờ tăng ca
            tvSoGio.setText(String.format("Số giờ: %.1f (Tăng ca: %.1f)", soGioLam, gioTangCa)); // Hiển thị số giờ và giờ tăng ca
        } else {                                                           // Nếu không có giờ tăng ca
            tvSoGio.setText("Số giờ: " + String.format("%.1f", soGioLam)); // Chỉ hiển thị số giờ làm việc
        }
        
        tvTrangThai.setText("Trạng thái: " + chamCong.getTrangThai());     // Set text hiển thị trạng thái
        
        // Hiển thị ghi chú nếu có
        if (chamCong.getGhiChu() != null && !chamCong.getGhiChu().trim().isEmpty()) { // Kiểm tra ghi chú không null và không rỗng
            tvGhiChu.setText("Ghi chú: " + chamCong.getGhiChu());          // Set text hiển thị ghi chú
            tvGhiChu.setVisibility(View.VISIBLE);                          // Hiển thị TextView ghi chú
        } else {                                                           // Nếu không có ghi chú
            tvGhiChu.setVisibility(View.GONE);                             // Ẩn TextView ghi chú
        }
        
        // Thiết lập màu sắc cho trạng thái
        switch (chamCong.getTrangThai()) {                                 // Switch case theo trạng thái
            case "Có mặt":                                                 // Nếu trạng thái là "Có mặt"
                tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_green_dark)); // Set màu xanh lá
                break;
            case "Vắng mặt":                                               // Nếu trạng thái là "Vắng mặt"
                tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_red_dark)); // Set màu đỏ
                break;
            default:                                                       // Các trạng thái khác
                tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.darker_gray)); // Set màu xám
                break;
        }
        
        // Hiển thị nút sửa/xóa cho Admin/HR/Manager
        if (("Admin".equals(currentRole) || "HR".equals(currentRole) || "Manager".equals(currentRole))  // Kiểm tra quyền Admin/HR/Manager
            && chamCong.getMaNhanVien() != null) {                         // Và có mã nhân viên
            layoutButtons.setVisibility(View.VISIBLE);                     // Hiển thị layout chứa các button
            
            btnSua.setOnClickListener(v -> showEditDialog(chamCong, position)); // Set listener cho button sửa
            btnXoa.setOnClickListener(v -> showDeleteDialog(chamCong, position)); // Set listener cho button xóa
        } else {                                                           // Nếu không có quyền hoặc không có mã nhân viên
            layoutButtons.setVisibility(View.GONE);                        // Ẩn layout chứa các button
        }
        
        return convertView;                                                // Trả về View đã được setup
    }
```

#### Method showEditDialog:
```java
    private void showEditDialog(ChamCong chamCong, int position) {         // Method hiển thị dialog chỉnh sửa chấm công
        AlertDialog.Builder builder = new AlertDialog.Builder(context);    // Tạo AlertDialog builder
        View dialogView = LayoutInflater.from(context).inflate(R.layout.dialog_edit_cham_cong, null); // Inflate layout dialog
        
        EditText etGioVao = dialogView.findViewById(R.id.et_gio_vao);      // Ánh xạ EditText giờ vào
        EditText etGioRa = dialogView.findViewById(R.id.et_gio_ra);        // Ánh xạ EditText giờ ra
        EditText etGhiChu = dialogView.findViewById(R.id.et_ghi_chu);      // Ánh xạ EditText ghi chú
        
        etGioVao.setText(chamCong.getGioVao());                            // Set text hiện tại cho giờ vào
        etGioRa.setText(chamCong.getGioRa());                              // Set text hiện tại cho giờ ra
        etGhiChu.setText(chamCong.getGhiChu() != null ? chamCong.getGhiChu() : ""); // Set text hiện tại cho ghi chú (hoặc rỗng nếu null)
        
        // Time picker cho giờ vào
        etGioVao.setOnClickListener(v -> showTimePicker(etGioVao));        // Set listener cho EditText giờ vào để mở time picker
        etGioRa.setOnClickListener(v -> showTimePicker(etGioRa));          // Set listener cho EditText giờ ra để mở time picker
        
        builder.setView(dialogView)                                        // Set view cho dialog
               .setTitle("Sửa chấm công - " + chamCong.getNgayChamCong())  // Set tiêu đề dialog với ngày chấm công
               .setPositiveButton("Lưu", (dialog, which) -> {              // Set button "Lưu" với action
                   String gioVao = etGioVao.getText().toString().trim();   // Lấy giờ vào từ EditText
                   String gioRa = etGioRa.getText().toString().trim();     // Lấy giờ ra từ EditText
                   String ghiChu = etGhiChu.getText().toString().trim();   // Lấy ghi chú từ EditText
                   
                   boolean success = dbHelper.updateAttendance(chamCong.getMaNhanVien(),  // Gọi method database cập nhật chấm công
                                                             chamCong.getNgayChamCong(), 
                                                             gioVao, gioRa, ghiChu);
                   if (success) {                                          // Nếu cập nhật thành công
                       Toast.makeText(context, "Cập nhật thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                       refreshData();                                      // Refresh dữ liệu
                   } else {                                                // Nếu cập nhật thất bại
                       Toast.makeText(context, "Lỗi khi cập nhật", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                   }
               })
               .setNegativeButton("Hủy", null)                             // Set button "Hủy" không có action
               .show();                                                    // Hiển thị dialog
    }
```

#### Method showTimePicker:
```java
    private void showTimePicker(EditText editText) {                       // Method hiển thị time picker
        Calendar calendar = Calendar.getInstance();                        // Lấy instance Calendar hiện tại
        int hour = calendar.get(Calendar.HOUR_OF_DAY);                     // Lấy giờ hiện tại
        int minute = calendar.get(Calendar.MINUTE);                        // Lấy phút hiện tại
        
        TimePickerDialog timePickerDialog = new TimePickerDialog(context,  // Tạo TimePickerDialog
            (view, hourOfDay, minute1) -> {                                // Callback khi chọn thời gian
                String time = String.format("%02d:%02d:00", hourOfDay, minute1); // Format thời gian thành HH:mm:00
                editText.setText(time);                                    // Set text cho EditText
            }, hour, minute, true);                                        // Khởi tạo với giờ/phút hiện tại, format 24h
        timePickerDialog.show();                                           // Hiển thị dialog
    }
```

#### Method showDeleteDialog:
```java
    private void showDeleteDialog(ChamCong chamCong, int position) {       // Method hiển thị dialog xác nhận xóa
        new AlertDialog.Builder(context)                                   // Tạo AlertDialog builder
            .setTitle("Xóa dữ liệu chấm công")                             // Set tiêu đề dialog
            .setMessage("Bạn có chắc muốn xóa dữ liệu chấm công ngày " + chamCong.getNgayChamCong() + "?") // Set message xác nhận
            .setPositiveButton("Xóa", (dialog, which) -> {                 // Set button "Xóa" với action
                boolean success = dbHelper.deleteAttendance(chamCong.getMaNhanVien(),  // Gọi method database xóa chấm công
                                                          chamCong.getNgayChamCong());
                if (success) {                                             // Nếu xóa thành công
                    Toast.makeText(context, "Xóa thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                    refreshData();                                         // Refresh dữ liệu
                } else {                                                   // Nếu xóa thất bại
                    Toast.makeText(context, "Lỗi khi xóa", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                }
            })
            .setNegativeButton("Hủy", null)                                // Set button "Hủy" không có action
            .show();                                                       // Hiển thị dialog
    }
```

#### Method refreshData:
```java
    private void refreshData() {                                           // Method refresh dữ liệu
        if (context instanceof ChamCongActivity) {                         // Kiểm tra context có phải ChamCongActivity không
            ((ChamCongActivity) context).loadAttendanceHistory();          // Cast context và gọi method loadAttendanceHistory
        }
    }
```

#### Method updateData:
```java
    public void updateData(List<ChamCong> newData) {                       // Method cập nhật dữ liệu mới cho adapter
        this.listChamCong = newData;                                       // Gán list mới
        notifyDataSetChanged();                                            // Thông báo adapter dữ liệu đã thay đổi để refresh UI
    }
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 4️⃣ LAYOUT CHÍNH - activity_cham_cong.xml

**Đường dẫn**: `app/src/main/res/layout/activity_cham_cong.xml`

### Mục đích:
Layout chính cho Activity chấm công, bao gồm đồng hồ thời gian thực, chấm công cá nhân và quản lý chấm công.

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
            android:text="CHẤM CÔNG"                                       <!-- Text mặc định hiển thị -->
            android:textSize="24sp"                                        <!-- Kích thước font 24sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương (#2196F3) -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->

        <!-- Thời gian hiện tại -->
        <LinearLayout                                                      <!-- LinearLayout chứa thời gian hiện tại -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:gravity="center">                                      <!-- Căn giữa content -->

            <TextView                                                      <!-- TextView hiển thị ngày hiện tại -->
                android:id="@+id/tv_current_date"                          <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="2024-01-01"                                  <!-- Text mặc định hiển thị -->
                android:textSize="18sp"                                    <!-- Kích thước font 18sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333"                                   <!-- Màu chữ đen nhạt (#333) -->
                android:layout_marginEnd="16dp" />                         <!-- Margin end 16dp -->

            <TextView                                                      <!-- TextView hiển thị thời gian hiện tại -->
                android:id="@+id/tv_current_time"                          <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="00:00:00"                                    <!-- Text mặc định hiển thị -->
                android:textSize="24sp"                                    <!-- Kích thước font 24sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#2196F3" />                             <!-- Màu chữ xanh dương (#2196F3) -->

        </LinearLayout>

    </LinearLayout>
```

#### Personal Attendance Section:
```xml
    <!-- Layout cho Employee - Chấm công cá nhân -->
    <LinearLayout                                                          <!-- LinearLayout cho chấm công cá nhân -->
        android:id="@+id/layout_cham_cong_ca_nhan"                         <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:layout_margin="16dp"                                       <!-- Margin 16dp cho tất cả các cạnh -->
        android:elevation="2dp"                                            <!-- Elevation 2dp tạo shadow -->
        android:visibility="visible">                                      <!-- Visibility visible (hiển thị mặc định) -->

        <TextView                                                          <!-- TextView hiển thị trạng thái chấm công -->
            android:id="@+id/tv_status"                                    <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Chưa chấm công vào"                              <!-- Text mặc định hiển thị -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám (#666) -->
            android:gravity="center"                                       <!-- Căn giữa text -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->

        <LinearLayout                                                      <!-- LinearLayout chứa các button chấm công -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal">                              <!-- Orientation ngang -->

            <Button                                                        <!-- Button chấm công vào -->
                android:id="@+id/btn_cham_cong_vao"                        <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="CHẤM CÔNG VÀO"                               <!-- Text hiển thị trên button -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:background="@drawable/btn_cham_cong_vao"           <!-- Background custom từ drawable -->
                android:layout_marginEnd="8dp" />                          <!-- Margin end 8dp -->

            <Button                                                        <!-- Button chấm công ra -->
                android:id="@+id/btn_cham_cong_ra"                         <!-- ID để truy cập từ Java code -->
                android:layout_width="0dp"                                 <!-- Chiều rộng 0dp để sử dụng layout_weight -->
                android:layout_height="48dp"                               <!-- Chiều cao cố định 48dp -->
                android:layout_weight="1"                                  <!-- Weight 1 để chia đều không gian -->
                android:text="CHẤM CÔNG RA"                                <!-- Text hiển thị trên button -->
                android:textColor="@color/white"                           <!-- Màu chữ trắng -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:background="@drawable/btn_cham_cong_ra"            <!-- Background custom từ drawable -->
                android:layout_marginStart="8dp" />                        <!-- Margin start 8dp -->

        </LinearLayout>

    </LinearLayout>
```
#### Management Section:
```xml
    <!-- Layout cho Admin/HR/Manager - Quản lý chấm công -->
    <LinearLayout                                                          <!-- LinearLayout cho quản lý chấm công -->
        android:id="@+id/layout_quan_ly_cham_cong"                         <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:layout_margin="16dp"                                       <!-- Margin 16dp cho tất cả các cạnh -->
        android:elevation="2dp"                                            <!-- Elevation 2dp tạo shadow -->
        android:visibility="gone">                                         <!-- Visibility gone (ẩn mặc định, hiển thị theo quyền) -->

        <TextView                                                          <!-- TextView label cho Spinner -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Chọn nhân viên"                                  <!-- Text label hiển thị -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#333"                                       <!-- Màu chữ đen nhạt (#333) -->
            android:layout_marginBottom="8dp" />                           <!-- Margin bottom 8dp -->

        <Spinner                                                           <!-- Spinner chọn nhân viên -->
            android:id="@+id/sp_nhan_vien"                                 <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:background="@drawable/edit_text_background"            <!-- Background custom từ drawable -->
            android:padding="12dp" />                                      <!-- Padding 12dp cho tất cả các cạnh -->

    </LinearLayout>
```

#### History Section:
```xml
    <!-- Lịch sử chấm công -->
    <LinearLayout                                                          <!-- LinearLayout chứa tiêu đề lịch sử -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:layout_marginStart="16dp"                                  <!-- Margin start 16dp -->
        android:layout_marginEnd="16dp"                                    <!-- Margin end 16dp -->
        android:layout_marginBottom="8dp"                                  <!-- Margin bottom 8dp -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:elevation="2dp">                                           <!-- Elevation 2dp tạo shadow -->

        <TextView                                                          <!-- TextView tiêu đề lịch sử -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="LỊCH SỬ CHẤM CÔNG"                               <!-- Text tiêu đề hiển thị -->
            android:textSize="18sp"                                        <!-- Kích thước font 18sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#333" />                                    <!-- Màu chữ đen nhạt (#333) -->

    </LinearLayout>

    <ListView                                                              <!-- ListView hiển thị lịch sử chấm công -->
        android:id="@+id/lv_lich_su_cham_cong"                             <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="0dp"                                       <!-- Chiều cao 0dp để sử dụng layout_weight -->
        android:layout_weight="1"                                          <!-- Weight 1 để chiếm không gian còn lại -->
        android:layout_marginStart="16dp"                                  <!-- Margin start 16dp -->
        android:layout_marginEnd="16dp"                                    <!-- Margin end 16dp -->
        android:layout_marginBottom="16dp"                                 <!-- Margin bottom 16dp -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:divider="#E0E0E0"                                          <!-- Màu divider giữa các item -->
        android:dividerHeight="1dp"                                        <!-- Chiều cao divider 1dp -->
        android:elevation="2dp" />                                         <!-- Elevation 2dp tạo shadow -->

</LinearLayout>
```

---

## 5️⃣ LAYOUT ITEM - item_cham_cong.xml

**Đường dẫn**: `app/src/main/res/layout/item_cham_cong.xml`

### Mục đích:
Layout cho từng item lịch sử chấm công trong ListView, hiển thị thông tin chi tiết và các button hành động.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Khai báo LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ chứa content -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:padding="12dp"                                                 <!-- Padding 12dp cho tất cả các cạnh -->
    android:background="@android:color/white"                              <!-- Background màu trắng -->
    android:layout_marginBottom="8dp"                                      <!-- Margin bottom 8dp -->
    android:elevation="1dp">                                               <!-- Elevation 1dp tạo shadow nhẹ -->
```

#### Employee Information Section:
```xml
    <!-- Thông tin nhân viên (chỉ hiện với Admin/HR/Manager) -->
    <TextView                                                              <!-- TextView hiển thị mã nhân viên -->
        android:id="@+id/tv_ma_nv"                                         <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Mã NV: NV001"                                        <!-- Text mặc định hiển thị -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ đen nhạt (#333) -->
        android:layout_marginBottom="4dp"                                  <!-- Margin bottom 4dp -->
        android:visibility="gone" />                                       <!-- Visibility gone (ẩn mặc định, hiển thị theo quyền) -->

    <TextView                                                              <!-- TextView hiển thị họ tên nhân viên -->
        android:id="@+id/tv_ho_ten"                                        <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Họ tên: Nguyễn Văn A"                                <!-- Text mặc định hiển thị -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textColor="#666"                                           <!-- Màu chữ xám (#666) -->
        android:layout_marginBottom="8dp"                                  <!-- Margin bottom 8dp -->
        android:visibility="gone" />                                       <!-- Visibility gone (ẩn mặc định, hiển thị theo quyền) -->
```

#### Attendance Information Section:
```xml
    <!-- Thông tin chấm công -->
    <TextView                                                              <!-- TextView hiển thị ngày chấm công -->
        android:id="@+id/tv_ngay"                                          <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Ngày: 2024-01-15"                                    <!-- Text mặc định hiển thị -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ đen nhạt (#333) -->
        android:layout_marginBottom="4dp" />                               <!-- Margin bottom 4dp -->

    <LinearLayout                                                          <!-- LinearLayout chứa giờ vào và giờ ra -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal">                                  <!-- Orientation ngang -->

        <TextView                                                          <!-- TextView hiển thị giờ vào -->
            android:id="@+id/tv_gio_vao"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="0dp"                                     <!-- Chiều rộng 0dp để sử dụng layout_weight -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:layout_weight="1"                                      <!-- Weight 1 để chia đều không gian -->
            android:text="Giờ vào: 08:00"                                  <!-- Text mặc định hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666" />                                    <!-- Màu chữ xám (#666) -->

        <TextView                                                          <!-- TextView hiển thị giờ ra -->
            android:id="@+id/tv_gio_ra"                                    <!-- ID để truy cập từ Java code -->
            android:layout_width="0dp"                                     <!-- Chiều rộng 0dp để sử dụng layout_weight -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:layout_weight="1"                                      <!-- Weight 1 để chia đều không gian -->
            android:text="Giờ ra: 17:00"                                   <!-- Text mặc định hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666" />                                    <!-- Màu chữ xám (#666) -->

    </LinearLayout>

    <LinearLayout                                                          <!-- LinearLayout chứa số giờ và trạng thái -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:layout_marginTop="4dp">                                    <!-- Margin top 4dp -->

        <TextView                                                          <!-- TextView hiển thị số giờ làm -->
            android:id="@+id/tv_so_gio"                                    <!-- ID để truy cập từ Java code -->
            android:layout_width="0dp"                                     <!-- Chiều rộng 0dp để sử dụng layout_weight -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:layout_weight="1"                                      <!-- Weight 1 để chia đều không gian -->
            android:text="Số giờ: 8.0"                                     <!-- Text mặc định hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666" />                                    <!-- Màu chữ xám (#666) -->

        <TextView                                                          <!-- TextView hiển thị trạng thái -->
            android:id="@+id/tv_trang_thai"                                <!-- ID để truy cập từ Java code -->
            android:layout_width="0dp"                                     <!-- Chiều rộng 0dp để sử dụng layout_weight -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:layout_weight="1"                                      <!-- Weight 1 để chia đều không gian -->
            android:text="Trạng thái: Có mặt"                              <!-- Text mặc định hiển thị -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:gravity="end" />                                       <!-- Căn phải text -->

    </LinearLayout>
```

#### Notes Section:
```xml
    <!-- Ghi chú -->
    <TextView                                                              <!-- TextView hiển thị ghi chú -->
        android:id="@+id/tv_ghi_chu"                                       <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Ghi chú: Tăng ca"                                    <!-- Text mặc định hiển thị -->
        android:textSize="13sp"                                            <!-- Kích thước font 13sp -->
        android:textColor="#FF9800"                                        <!-- Màu chữ cam (#FF9800) -->
        android:textStyle="italic"                                         <!-- Style chữ nghiêng -->
        android:layout_marginTop="4dp"                                     <!-- Margin top 4dp -->
        android:visibility="gone" />                                       <!-- Visibility gone (ẩn mặc định, hiển thị khi có ghi chú) -->
```

#### Action Buttons Section:
```xml
    <!-- Buttons sửa/xóa (chỉ hiện với Admin/HR/Manager) -->
    <LinearLayout                                                          <!-- LinearLayout chứa các button hành động -->
        android:id="@+id/layout_buttons"                                   <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:gravity="end"                                              <!-- Căn phải các button -->
        android:layout_marginTop="8dp"                                     <!-- Margin top 8dp -->
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

## 6️⃣ LAYOUT DIALOG - dialog_edit_cham_cong.xml

**Đường dẫn**: `app/src/main/res/layout/dialog_edit_cham_cong.xml`

### Mục đích:
Layout cho dialog chỉnh sửa thông tin chấm công, bao gồm giờ vào, giờ ra và ghi chú.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- Khai báo LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ chứa content -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:padding="16dp">                                                <!-- Padding 16dp cho tất cả các cạnh -->
```

#### Time In Section:
```xml
    <!-- Giờ vào -->
    <TextView                                                              <!-- TextView label cho giờ vào -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Giờ vào"                                             <!-- Text label hiển thị -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ đen nhạt (#333) -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->

    <EditText                                                              <!-- EditText nhập giờ vào -->
        android:id="@+id/et_gio_vao"                                       <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="48dp"                                       <!-- Chiều cao cố định 48dp -->
        android:background="@drawable/edit_text_background"                <!-- Background custom từ drawable -->
        android:padding="12dp"                                             <!-- Padding 12dp cho tất cả các cạnh -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:layout_marginBottom="16dp"                                 <!-- Margin bottom 16dp -->
        android:hint="HH:mm:ss"                                            <!-- Hint text hiển thị format thời gian -->
        android:focusable="false"                                          <!-- Không cho focus (chỉ click để mở time picker) -->
        android:clickable="true" />                                        <!-- Cho phép click -->
```

#### Time Out Section:
```xml
    <!-- Giờ ra -->
    <TextView                                                              <!-- TextView label cho giờ ra -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Giờ ra"                                              <!-- Text label hiển thị -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ đen nhạt (#333) -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->

    <EditText                                                              <!-- EditText nhập giờ ra -->
        android:id="@+id/et_gio_ra"                                        <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="48dp"                                       <!-- Chiều cao cố định 48dp -->
        android:background="@drawable/edit_text_background"                <!-- Background custom từ drawable -->
        android:padding="12dp"                                             <!-- Padding 12dp cho tất cả các cạnh -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:hint="HH:mm:ss"                                            <!-- Hint text hiển thị format thời gian -->
        android:focusable="false"                                          <!-- Không cho focus (chỉ click để mở time picker) -->
        android:clickable="true"                                           <!-- Cho phép click -->
        android:layout_marginBottom="16dp" />                              <!-- Margin bottom 16dp -->
```

#### Notes Section:
```xml
    <!-- Ghi chú -->
    <TextView                                                              <!-- TextView label cho ghi chú -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Ghi chú"                                             <!-- Text label hiển thị -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ đen nhạt (#333) -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->

    <EditText                                                              <!-- EditText nhập ghi chú -->
        android:id="@+id/et_ghi_chu"                                       <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:minHeight="48dp"                                           <!-- Chiều cao tối thiểu 48dp -->
        android:background="@drawable/edit_text_background"                <!-- Background custom từ drawable -->
        android:padding="12dp"                                             <!-- Padding 12dp cho tất cả các cạnh -->
        android:textSize="16sp"                                            <!-- Kích thước font 16sp -->
        android:hint="Nhập ghi chú (nếu có)"                               <!-- Hint text hiển thị khi rỗng -->
        android:inputType="text"                                           <!-- Input type là text -->
        android:maxLines="3" />                                            <!-- Tối đa 3 dòng -->

</LinearLayout>
```

---

## 🔧 TÍCH HỢP VÀ LIÊN KẾT

### Database Methods (DatabaseHelper.java):
- `getTodayAttendanceStatus()`: Kiểm tra trạng thái chấm công hôm nay
- `chamCongVao()`: Thực hiện chấm công vào
- `chamCongRa()`: Thực hiện chấm công ra
- `getAttendanceHistory()`: Lấy lịch sử chấm công của nhân viên
- `getAllAttendanceHistory()`: Lấy lịch sử chấm công tất cả nhân viên
- `updateAttendance()`: Cập nhật thông tin chấm công
- `deleteAttendance()`: Xóa dữ liệu chấm công
- `tinhGioTangCa()`: Tính giờ tăng ca (> 8 giờ)
- `getEmployeeNameByMa()`: Lấy tên nhân viên từ mã

### Navigation Flow:
1. **DashboardActivity** → **ChamCongActivity** (với username và role)
2. **ChamCongActivity** → **Real-time updates** (thời gian, trạng thái)
3. **ChamCongAdapter** → **Edit/Delete dialogs** (Admin/Manager)

### Permission System:
- **All roles**: Chấm công cá nhân, xem lịch sử cá nhân
- **Admin/HR/Manager**: Quản lý chấm công tất cả nhân viên, sửa/xóa dữ liệu
- **Employee**: Chỉ chấm công và xem lịch sử cá nhân

### Real-time Features:
- **Clock updates**: Cập nhật thời gian mỗi giây
- **Status checking**: Kiểm tra trạng thái chấm công real-time
- **Button states**: Enable/disable button theo trạng thái
- **Auto refresh**: Tự động refresh sau khi chấm công

### Business Rules:
- **One check-in per day**: Chỉ cho phép chấm công vào 1 lần/ngày
- **Sequential flow**: Phải chấm công vào trước khi chấm công ra
- **Overtime calculation**: Tự động tính giờ tăng ca > 8 giờ
- **Time picker**: Sử dụng TimePickerDialog cho chỉnh sửa
- **Data validation**: Kiểm tra tính hợp lệ của thời gian

---

## 📝 KẾT LUẬN

Module Quản lý Chấm công được thiết kế với kiến trúc linh hoạt, hỗ trợ cả chấm công cá nhân và quản lý tập trung. Hệ thống cung cấp trải nghiệm người dùng mượt mà với đồng hồ thời gian thực, phân quyền rõ ràng và các tính năng quản lý mạnh mẽ cho Admin/Manager. Tính năng tính giờ tăng ca tự động và ghi chú linh hoạt giúp đáp ứng các nhu cầu thực tế trong quản lý nhân sự.