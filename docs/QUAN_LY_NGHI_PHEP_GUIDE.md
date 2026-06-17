# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ NGHỈ PHÉP

## 📋 TỔNG QUAN

Chức năng Quản lý Nghỉ phép cho phép nhân viên gửi đơn xin nghỉ phép và quản lý (Admin/HR/Manager) duyệt các đơn nghỉ phép. Module này hỗ trợ quy trình nghỉ phép hoàn chỉnh từ việc tạo đơn, tính toán số ngày nghỉ, đến việc duyệt và theo dõi lịch sử.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── NghiPhepActivity.java                    # Activity chính - Xử lý nghỉ phép
├── NghiPhepAdapter.java                     # Adapter hiển thị danh sách nghỉ phép
├── models/NghiPhep.java                     # Model class cho nghỉ phép
└── database/DatabaseHelper.java             # Xử lý database nghỉ phép

app/src/main/res/layout/
├── activity_nghi_phep.xml                   # Layout chính
├── item_nghi_phep.xml                       # Layout item trong ListView
└── dialog_edit_nghi_phep.xml                # Dialog chỉnh sửa đơn nghỉ phép
```

## 📊 NGHIỆP VỤ NGHỈ PHÉP

### 1. Quy trình nghiệp vụ:
- **Tạo đơn**: Nhân viên chọn ngày bắt đầu, kết thúc và nhập lý do
- **Tính toán**: Tự động tính số ngày nghỉ dựa trên khoảng thời gian
- **Gửi đơn**: Lưu đơn với trạng thái "Chờ duyệt"
- **Duyệt đơn**: Admin/HR/Manager có thể duyệt hoặc từ chối
- **Theo dõi**: Hiển thị lịch sử và trạng thái các đơn nghỉ phép

### 2. Phân quyền truy cập:
- **Employee**: Tạo đơn, xem lịch sử cá nhân, sửa/xóa đơn chờ duyệt
- **Admin/HR/Manager**: Xem tất cả đơn, duyệt/từ chối, sửa/xóa đơn
- **Role-based UI**: Giao diện thay đổi theo vai trò người dùng

### 3. Trạng thái đơn nghỉ phép:
- **Chờ duyệt**: Đơn mới tạo, có thể sửa/xóa
- **Đã duyệt**: Đơn được chấp nhận, không thể sửa
- **Từ chối**: Đơn bị từ chối, không thể sửa

---

## 📱 CHI TIẾT CÁC FILE

## 1️⃣ ACTIVITY CHÍNH - NghiPhepActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/NghiPhepActivity.java`

### Mục đích:
Activity chính xử lý tạo đơn nghỉ phép, hiển thị lịch sử và quản lý các đơn nghỉ phép.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.app.DatePickerDialog;                                       // Import DatePickerDialog để chọn ngày
import android.content.Intent;                                             // Import Intent để truyền dữ liệu
import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.ListView;                                            // Import ListView widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database

import java.text.ParseException;                                           // Import ParseException để xử lý lỗi parse
import java.text.SimpleDateFormat;                                         // Import SimpleDateFormat để format ngày
import java.util.Calendar;                                                 // Import Calendar để xử lý ngày tháng
import java.util.Date;                                                     // Import Date để xử lý ngày
import java.util.Locale;                                                   // Import Locale để định dạng theo vùng miền
import java.util.concurrent.TimeUnit;                                      // Import TimeUnit để tính toán thời gian
```

#### Khai báo thuộc tính class:
```java
public class NghiPhepActivity extends AppCompatActivity {                  // Khai báo class kế thừa AppCompatActivity
    
    private TextView tvTitle, tvSoNgayNghi;                                // TextView cho tiêu đề và số ngày nghỉ
    private EditText etNgayBatDau, etNgayKetThuc, etLyDo;                  // EditText cho ngày bắt đầu, kết thúc và lý do
    private Button btnXinNghiPhep;                                         // Button gửi đơn xin nghỉ phép
    private ListView lvLichSuNghiPhep;                                     // ListView hiển thị lịch sử nghỉ phép
    
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private String currentUsername;                                        // Tên đăng nhập hiện tại
    private String maNhanVien;                                             // Mã nhân viên hiện tại
    private String currentRole;                                            // Vai trò hiện tại
    private NghiPhepAdapter adapter;                                       // Adapter cho ListView
```
#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_nghi_phep);                       // Set layout cho Activity
        
        initViews();                                                       // Khởi tạo các view components
        setupDatabase();                                                   // Thiết lập database và lấy thông tin user
        setupDatePickers();                                                // Thiết lập date picker cho ngày
        setupButtons();                                                    // Thiết lập các button events
        loadLeaveHistory();                                                // Load lịch sử nghỉ phép
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        tvTitle = findViewById(R.id.tv_title);                             // Ánh xạ TextView tiêu đề
        tvSoNgayNghi = findViewById(R.id.tv_so_ngay_nghi);                 // Ánh xạ TextView số ngày nghỉ
        etNgayBatDau = findViewById(R.id.et_ngay_bat_dau);                 // Ánh xạ EditText ngày bắt đầu
        etNgayKetThuc = findViewById(R.id.et_ngay_ket_thuc);               // Ánh xạ EditText ngày kết thúc
        etLyDo = findViewById(R.id.et_ly_do);                              // Ánh xạ EditText lý do
        btnXinNghiPhep = findViewById(R.id.btn_xin_nghi_phep);             // Ánh xạ Button xin nghỉ phép
        lvLichSuNghiPhep = findViewById(R.id.lv_lich_su_nghi_phep);        // Ánh xạ ListView lịch sử nghỉ phép
    }
```

#### Method setupDatabase:
```java
    private void setupDatabase() {                                         // Method thiết lập database và lấy thông tin user
        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        currentUsername = getIntent().getStringExtra("username");         // Lấy username từ Intent
        currentRole = getIntent().getStringExtra("role");                 // Lấy role từ Intent
        
        if (currentUsername != null) {                                     // Nếu username không null
            maNhanVien = dbHelper.getMaNhanVienByUsername(currentUsername); // Lấy mã nhân viên từ username
        }
        
        // Thiết lập title theo role
        if ("Employee".equals(currentRole)) {                              // Nếu là Employee
            tvTitle.setText("XIN NGHỈ PHÉP");                              // Set title cho Employee
        } else {                                                           // Nếu là Admin/HR/Manager
            tvTitle.setText("QUẢN LÝ NGHỈ PHÉP");                          // Set title cho quản lý
        }
    }
```

#### Method setupDatePickers:
```java
    private void setupDatePickers() {                                      // Method thiết lập date picker cho các EditText ngày
        etNgayBatDau.setOnClickListener(v -> showDatePicker(etNgayBatDau)); // Set listener cho EditText ngày bắt đầu
        etNgayKetThuc.setOnClickListener(v -> showDatePicker(etNgayKetThuc)); // Set listener cho EditText ngày kết thúc
        
        // Tính số ngày nghỉ khi thay đổi ngày
        etNgayBatDau.setOnFocusChangeListener((v, hasFocus) -> {           // Set listener khi focus thay đổi
            if (!hasFocus) calculateLeaveDays();                          // Tính lại số ngày khi mất focus
        });
        etNgayKetThuc.setOnFocusChangeListener((v, hasFocus) -> {          // Set listener khi focus thay đổi
            if (!hasFocus) calculateLeaveDays();                          // Tính lại số ngày khi mất focus
        });
    }
```

#### Method showDatePicker:
```java
    private void showDatePicker(EditText editText) {                       // Method hiển thị DatePickerDialog
        Calendar calendar = Calendar.getInstance();                        // Lấy Calendar instance hiện tại
        int year = calendar.get(Calendar.YEAR);                            // Lấy năm hiện tại
        int month = calendar.get(Calendar.MONTH);                          // Lấy tháng hiện tại
        int day = calendar.get(Calendar.DAY_OF_MONTH);                     // Lấy ngày hiện tại
        
        DatePickerDialog datePickerDialog = new DatePickerDialog(this,     // Tạo DatePickerDialog
            (view, year1, month1, dayOfMonth) -> {                         // Callback khi chọn ngày
                String date = String.format(Locale.getDefault(), "%d-%02d-%02d", year1, month1 + 1, dayOfMonth); // Format ngày yyyy-MM-dd
                editText.setText(date);                                    // Set ngày vào EditText
                calculateLeaveDays();                                      // Tính lại số ngày nghỉ
            }, year, month, day);                                          // Truyền ngày hiện tại làm mặc định
        
        // Không cho chọn ngày trong quá khứ
        datePickerDialog.getDatePicker().setMinDate(System.currentTimeMillis() - 1000); // Set ngày tối thiểu là hôm nay
        datePickerDialog.show();                                           // Hiển thị DatePickerDialog
    }
```

#### Method calculateLeaveDays:
```java
    private void calculateLeaveDays() {                                    // Method tính toán số ngày nghỉ
        String startDate = etNgayBatDau.getText().toString().trim();       // Lấy ngày bắt đầu từ EditText
        String endDate = etNgayKetThuc.getText().toString().trim();        // Lấy ngày kết thúc từ EditText
        
        if (!startDate.isEmpty() && !endDate.isEmpty()) {                  // Nếu cả hai ngày đều có giá trị
            try {
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()); // Tạo SimpleDateFormat
                Date start = sdf.parse(startDate);                         // Parse ngày bắt đầu
                Date end = sdf.parse(endDate);                             // Parse ngày kết thúc
                
                if (start != null && end != null) {                        // Nếu parse thành công
                    long diffInMillies = Math.abs(end.getTime() - start.getTime()); // Tính chênh lệch milliseconds
                    long diffInDays = TimeUnit.DAYS.convert(diffInMillies, TimeUnit.MILLISECONDS) + 1; // Convert sang ngày và +1
                    
                    if (end.before(start)) {                               // Nếu ngày kết thúc trước ngày bắt đầu
                        tvSoNgayNghi.setText("Ngày kết thúc phải sau ngày bắt đầu"); // Hiển thị thông báo lỗi
                        tvSoNgayNghi.setTextColor(getResources().getColor(android.R.color.holo_red_dark)); // Set màu đỏ
                    } else {                                               // Nếu ngày hợp lệ
                        tvSoNgayNghi.setText("Số ngày nghỉ: " + diffInDays + " ngày"); // Hiển thị số ngày nghỉ
                        tvSoNgayNghi.setTextColor(getResources().getColor(android.R.color.holo_blue_dark)); // Set màu xanh
                    }
                }
            } catch (ParseException e) {                                   // Catch exception khi parse ngày
                tvSoNgayNghi.setText("Định dạng ngày không hợp lệ");       // Hiển thị thông báo lỗi format
                tvSoNgayNghi.setTextColor(getResources().getColor(android.R.color.holo_red_dark)); // Set màu đỏ
            }
        } else {                                                           // Nếu chưa nhập đủ ngày
            tvSoNgayNghi.setText("Số ngày nghỉ: 0 ngày");                  // Hiển thị 0 ngày
            tvSoNgayNghi.setTextColor(getResources().getColor(android.R.color.darker_gray)); // Set màu xám
        }
    }
```

#### Method setupButtons:
```java
    private void setupButtons() {                                          // Method thiết lập các sự kiện cho button
        btnXinNghiPhep.setOnClickListener(v -> submitLeaveRequest());      // Set listener cho button xin nghỉ phép
    }
```

#### Method submitLeaveRequest:
```java
    private void submitLeaveRequest() {                                    // Method xử lý gửi đơn xin nghỉ phép
        String startDate = etNgayBatDau.getText().toString().trim();       // Lấy ngày bắt đầu từ form
        String endDate = etNgayKetThuc.getText().toString().trim();        // Lấy ngày kết thúc từ form
        String reason = etLyDo.getText().toString().trim();                // Lấy lý do từ form
        
        if (startDate.isEmpty()) {                                         // Kiểm tra ngày bắt đầu không được rỗng
            Toast.makeText(this, "Vui lòng chọn ngày bắt đầu", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        if (endDate.isEmpty()) {                                           // Kiểm tra ngày kết thúc không được rỗng
            Toast.makeText(this, "Vui lòng chọn ngày kết thúc", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        if (reason.isEmpty()) {                                            // Kiểm tra lý do không được rỗng
            Toast.makeText(this, "Vui lòng nhập lý do nghỉ phép", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        // Tính số ngày nghỉ
        int soNgayNghi = calculateDaysBetween(startDate, endDate);         // Gọi method tính số ngày nghỉ
        if (soNgayNghi <= 0) {                                             // Nếu số ngày nghỉ không hợp lệ
            Toast.makeText(this, "Ngày kết thúc phải sau ngày bắt đầu", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            return;                                                        // Thoát method
        }
        
        boolean success = dbHelper.submitLeaveRequest(maNhanVien, startDate, endDate, soNgayNghi, reason); // Gọi method database gửi đơn
        
        if (success) {                                                     // Nếu gửi đơn thành công
            Toast.makeText(this, "Gửi đơn xin nghỉ phép thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
            clearForm();                                                   // Xóa form
            loadLeaveHistory();                                            // Reload lịch sử nghỉ phép
        } else {                                                           // Nếu gửi đơn thất bại
            Toast.makeText(this, "Lỗi khi gửi đơn xin nghỉ phép", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
        }
    }
```

#### Method calculateDaysBetween:
```java
    private int calculateDaysBetween(String startDate, String endDate) {   // Method tính số ngày giữa hai ngày
        try {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()); // Tạo SimpleDateFormat
            Date start = sdf.parse(startDate);                             // Parse ngày bắt đầu
            Date end = sdf.parse(endDate);                                 // Parse ngày kết thúc
            
            if (start != null && end != null && !end.before(start)) {      // Nếu parse thành công và ngày hợp lệ
                long diffInMillies = Math.abs(end.getTime() - start.getTime()); // Tính chênh lệch milliseconds
                return (int) (TimeUnit.DAYS.convert(diffInMillies, TimeUnit.MILLISECONDS) + 1); // Convert sang ngày và +1
            }
        } catch (ParseException e) {                                       // Catch exception khi parse
            e.printStackTrace();                                           // In stack trace
        }
        return 0;                                                          // Trả về 0 nếu có lỗi
    }
```

#### Method clearForm:
```java
    private void clearForm() {                                             // Method xóa form sau khi gửi đơn thành công
        etNgayBatDau.setText("");                                          // Xóa ngày bắt đầu
        etNgayKetThuc.setText("");                                         // Xóa ngày kết thúc
        etLyDo.setText("");                                                // Xóa lý do
        tvSoNgayNghi.setText("Số ngày nghỉ: 0 ngày");                      // Reset số ngày nghỉ
        tvSoNgayNghi.setTextColor(getResources().getColor(android.R.color.darker_gray)); // Set màu xám
    }
```

#### Method loadLeaveHistory:
```java
    private void loadLeaveHistory() {                                      // Method load lịch sử nghỉ phép
        Cursor cursor;                                                     // Khai báo cursor
        if ("Employee".equals(currentRole)) {                              // Nếu là Employee
            // Employee chỉ xem lịch sử của mình
            cursor = dbHelper.getLeaveHistory(maNhanVien);                 // Lấy lịch sử nghỉ phép của nhân viên hiện tại
        } else {                                                           // Nếu là Admin/HR/Manager
            // Admin/HR/Manager xem tất cả đơn nghỉ phép
            cursor = dbHelper.getAllLeaveRequests();                       // Lấy tất cả đơn nghỉ phép
        }
        
        adapter = new NghiPhepAdapter(this, cursor, currentRole);          // Tạo adapter với cursor và role
        lvLichSuNghiPhep.setAdapter(adapter);                              // Set adapter cho ListView
    }
```

#### Method onResume và các method hỗ trợ:
```java
    @Override
    protected void onResume() {                                            // Method được gọi khi Activity resume
        super.onResume();                                                  // Gọi method onResume của class cha
        loadLeaveHistory();                                                // Reload lịch sử nghỉ phép
    }

    public void refreshHistory() {                                         // Method refresh lịch sử (được gọi từ adapter)
        loadLeaveHistory();                                                // Reload lịch sử nghỉ phép
    }

    public void showEditLeaveDialog(int maNghiPhep, String currentStart, String currentEnd, int currentDays, String currentReason) { // Method hiển thị dialog chỉnh sửa đơn nghỉ phép
        androidx.appcompat.app.AlertDialog.Builder builder = new androidx.appcompat.app.AlertDialog.Builder(this); // Tạo AlertDialog Builder
        android.view.View view = getLayoutInflater().inflate(R.layout.dialog_edit_nghi_phep, null); // Inflate layout dialog
        
        EditText etStart = view.findViewById(R.id.et_edit_start);          // Ánh xạ EditText ngày bắt đầu trong dialog
        EditText etEnd = view.findViewById(R.id.et_edit_end);              // Ánh xạ EditText ngày kết thúc trong dialog
        EditText etReason = view.findViewById(R.id.et_edit_ly_do);         // Ánh xạ EditText lý do trong dialog
        TextView tvDays = view.findViewById(R.id.tv_edit_so_ngay);         // Ánh xạ TextView số ngày trong dialog
        
        etStart.setText(currentStart);                                     // Set giá trị hiện tại cho ngày bắt đầu
        etEnd.setText(currentEnd);                                         // Set giá trị hiện tại cho ngày kết thúc
        etReason.setText(currentReason);                                   // Set giá trị hiện tại cho lý do
        tvDays.setText("Số ngày: " + currentDays);                         // Set số ngày hiện tại
        
        etStart.setOnClickListener(v -> showDatePicker(etStart));          // Set listener cho date picker ngày bắt đầu
        etEnd.setOnClickListener(v -> showDatePicker(etEnd));              // Set listener cho date picker ngày kết thúc
        
        builder.setView(view)                                              // Set view cho dialog
            .setTitle("Sửa đơn nghỉ phép")                                 // Set title cho dialog
            .setPositiveButton("Cập nhật", (dialog, which) -> {           // Set button positive
                String newStart = etStart.getText().toString();           // Lấy ngày bắt đầu mới
                String newEnd = etEnd.getText().toString();               // Lấy ngày kết thúc mới
                String newReason = etReason.getText().toString();         // Lấy lý do mới
                int newDays = calculateDaysBetween(newStart, newEnd);     // Tính số ngày mới
                
                if (newDays > 0 && !newReason.isEmpty()) {               // Nếu dữ liệu hợp lệ
                    if (dbHelper.updateLeaveRequest(maNghiPhep, newStart, newEnd, newDays, newReason)) { // Gọi method update database
                        Toast.makeText(this, "Cập nhật thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                        loadLeaveHistory();                               // Reload lịch sử
                    }
                } else {                                                  // Nếu dữ liệu không hợp lệ
                    Toast.makeText(this, "Vui lòng nhập đầy đủ thông tin", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                }
            })
            .setNegativeButton("Hủy", null)                               // Set button negative
            .show();                                                      // Hiển thị dialog
    }
}
```

---

## 2️⃣ ADAPTER CLASS - NghiPhepAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/NghiPhepAdapter.java`

### Mục đích:
Adapter class quản lý hiển thị danh sách nghỉ phép trong ListView với các chức năng sửa, xóa, duyệt theo role.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.content.Context;                                            // Import Context để truy cập resources
import android.database.Cursor;                                            // Import Cursor để xử lý dữ liệu database
import android.view.LayoutInflater;                                        // Import LayoutInflater để inflate layout
import android.view.View;                                                  // Import View để xử lý UI components
import android.view.ViewGroup;                                             // Import ViewGroup để quản lý layout
import android.widget.BaseAdapter;                                         // Import BaseAdapter làm base class
import android.widget.Button;                                              // Import Button widget
import android.widget.ImageButton;                                         // Import ImageButton widget
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo
import androidx.appcompat.app.AlertDialog;                                 // Import AlertDialog để hiển thị dialog

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
```

#### Khai báo thuộc tính class:
```java
public class NghiPhepAdapter extends BaseAdapter {                         // Khai báo class kế thừa BaseAdapter
    
    private Context context;                                               // Context để truy cập resources và services
    private Cursor cursor;                                                 // Cursor chứa dữ liệu nghỉ phép
    private String currentRole;                                            // Vai trò hiện tại của user
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    
    public NghiPhepAdapter(Context context, Cursor cursor, String currentRole) { // Constructor nhận context, cursor và role
        this.context = context;                                            // Gán context
        this.cursor = cursor;                                              // Gán cursor
        this.currentRole = currentRole;                                    // Gán role hiện tại
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
    }
```

#### Các method bắt buộc của BaseAdapter:
```java
    @Override
    public int getCount() {                                                // Method trả về số lượng item trong adapter
        return cursor != null ? cursor.getCount() : 0;                    // Trả về số record trong cursor, 0 nếu cursor null
    }
    
    @Override
    public Object getItem(int position) {                                  // Method trả về item tại vị trí position
        if (cursor != null && cursor.moveToPosition(position)) {          // Nếu cursor không null và move thành công
            return cursor;                                                 // Trả về cursor
        }
        return null;                                                       // Trả về null nếu không thể move
    }
    
    @Override
    public long getItemId(int position) {                                  // Method trả về ID của item tại position
        return position;                                                   // Trả về position làm ID
    }
```
#### Method getView (phần 1):
```java
    @Override
    public View getView(int position, View convertView, ViewGroup parent) { // Method tạo và cấu hình view cho mỗi item
        if (convertView == null) {                                         // Nếu convertView null (tạo view mới)
            convertView = LayoutInflater.from(context).inflate(R.layout.item_nghi_phep, parent, false); // Inflate layout item
        }
        
        if (cursor != null && cursor.moveToPosition(position)) {           // Nếu cursor không null và move thành công
            // Ánh xạ các view components từ layout
            TextView tvMaNV = convertView.findViewById(R.id.tv_ma_nv);      // TextView mã nhân viên
            TextView tvHoTen = convertView.findViewById(R.id.tv_ho_ten);    // TextView họ tên
            TextView tvNgayNghi = convertView.findViewById(R.id.tv_ngay_nghi); // TextView ngày nghỉ
            TextView tvSoNgay = convertView.findViewById(R.id.tv_so_ngay);  // TextView số ngày
            TextView tvLyDo = convertView.findViewById(R.id.tv_ly_do);      // TextView lý do
            TextView tvTrangThai = convertView.findViewById(R.id.tv_trang_thai); // TextView trạng thái
            Button btnDuyet = convertView.findViewById(R.id.btn_duyet);     // Button duyệt
            Button btnTuChoi = convertView.findViewById(R.id.btn_tu_choi);  // Button từ chối
            ImageButton btnSua = convertView.findViewById(R.id.btn_sua_np); // ImageButton sửa
            ImageButton btnXoa = convertView.findViewById(R.id.btn_xoa_np); // ImageButton xóa
            
            // Lấy dữ liệu từ cursor
            int maNghiPhep = cursor.getInt(cursor.getColumnIndexOrThrow("MaNghiPhep"));         // Lấy mã nghỉ phép
            String maNhanVien = cursor.getString(cursor.getColumnIndexOrThrow("MaNhanVien"));   // Lấy mã nhân viên
            String ngayBatDau = cursor.getString(cursor.getColumnIndexOrThrow("NgayBatDau"));   // Lấy ngày bắt đầu
            String ngayKetThuc = cursor.getString(cursor.getColumnIndexOrThrow("NgayKetThuc")); // Lấy ngày kết thúc
            int soNgayNghi = cursor.getInt(cursor.getColumnIndexOrThrow("SoNgayNghi"));         // Lấy số ngày nghỉ
            String lyDo = cursor.getString(cursor.getColumnIndexOrThrow("LyDo"));               // Lấy lý do
            String trangThai = cursor.getString(cursor.getColumnIndexOrThrow("TrangThai"));     // Lấy trạng thái
```

#### Method getView (phần 2 - Hiển thị thông tin):
```java
            // Hiển thị thông tin cơ bản
            tvMaNV.setText("Mã NV: " + maNhanVien);                        // Hiển thị mã nhân viên
            tvNgayNghi.setText("Từ " + ngayBatDau + " đến " + ngayKetThuc); // Hiển thị khoảng thời gian nghỉ
            tvSoNgay.setText("Số ngày: " + soNgayNghi);                    // Hiển thị số ngày nghỉ
            tvLyDo.setText("Lý do: " + lyDo);                              // Hiển thị lý do nghỉ phép
            tvTrangThai.setText("Trạng thái: " + trangThai);               // Hiển thị trạng thái đơn
            
            // Lấy tên nhân viên nếu không phải Employee
            if (!"Employee".equals(currentRole)) {                         // Nếu không phải Employee
                String hoTen = dbHelper.getEmployeeNameByMa(maNhanVien);   // Lấy tên nhân viên từ database
                tvHoTen.setText("Họ tên: " + (hoTen != null ? hoTen : "N/A")); // Hiển thị tên nhân viên
                tvHoTen.setVisibility(View.VISIBLE);                       // Hiển thị TextView họ tên
            } else {                                                       // Nếu là Employee
                tvHoTen.setVisibility(View.GONE);                          // Ẩn TextView họ tên
            }
```

#### Method getView (phần 3 - Thiết lập màu sắc trạng thái):
```java
            // Thiết lập màu sắc cho trạng thái
            switch (trangThai) {                                           // Switch theo trạng thái
                case "Chờ duyệt":                                          // Trạng thái chờ duyệt
                    tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_orange_dark)); // Màu cam
                    break;
                case "Đã duyệt":                                           // Trạng thái đã duyệt
                    tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_green_dark)); // Màu xanh lá
                    break;
                case "Từ chối":                                            // Trạng thái từ chối
                    tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.holo_red_dark)); // Màu đỏ
                    break;
                default:                                                   // Trạng thái khác
                    tvTrangThai.setTextColor(context.getResources().getColor(android.R.color.darker_gray)); // Màu xám
                    break;
            }
```

#### Method getView (phần 4 - Logic hiển thị button):
```java
            // Logic cho các nút bấm
            if ("Chờ duyệt".equals(trangThai)) {                           // Nếu đơn đang chờ duyệt
                // Sửa/Xóa đơn cho nhân viên (chính chủ) hoặc Admin
                btnSua.setVisibility(View.VISIBLE);                        // Hiển thị button sửa
                btnXoa.setVisibility(View.VISIBLE);                        // Hiển thị button xóa

                // Duyệt/Từ chối cho Quản lý
                if ("Admin".equals(currentRole) || "HR".equals(currentRole) || "Manager".equals(currentRole)) { // Nếu là quản lý
                    btnDuyet.setVisibility(View.VISIBLE);                  // Hiển thị button duyệt
                    btnTuChoi.setVisibility(View.VISIBLE);                 // Hiển thị button từ chối
                } else {                                                   // Nếu là Employee
                    btnDuyet.setVisibility(View.GONE);                     // Ẩn button duyệt
                    btnTuChoi.setVisibility(View.GONE);                    // Ẩn button từ chối
                }
            } else {                                                       // Nếu đơn đã được xử lý
                btnSua.setVisibility(View.GONE);                           // Ẩn button sửa
                btnXoa.setVisibility(View.GONE);                           // Ẩn button xóa
                btnDuyet.setVisibility(View.GONE);                         // Ẩn button duyệt
                btnTuChoi.setVisibility(View.GONE);                        // Ẩn button từ chối
            }
```

#### Method getView (phần 5 - Sự kiện button):
```java
            // Sự kiện Xóa
            btnXoa.setOnClickListener(v -> {                               // Set listener cho button xóa
                new AlertDialog.Builder(context)                           // Tạo AlertDialog xác nhận
                    .setTitle("Xác nhận hủy đơn")                          // Set title dialog
                    .setMessage("Bạn có chắc chắn muốn hủy đơn nghỉ phép này?") // Set message dialog
                    .setPositiveButton("Hủy đơn", (dialog, which) -> {    // Set button positive
                        if (dbHelper.deleteLeaveRequest(maNghiPhep)) {     // Gọi method xóa đơn nghỉ phép
                            Toast.makeText(context, "Đã hủy đơn thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                            refreshData();                                 // Refresh dữ liệu
                        }
                    })
                    .setNegativeButton("Quay lại", null)                   // Set button negative
                    .show();                                               // Hiển thị dialog
            });

            // Sự kiện Sửa (Mở Dialog nhập lại lý do)
            btnSua.setOnClickListener(v -> {                               // Set listener cho button sửa
                if (context instanceof NghiPhepActivity) {                 // Nếu context là NghiPhepActivity
                    ((NghiPhepActivity) context).showEditLeaveDialog(maNghiPhep, ngayBatDau, ngayKetThuc, soNgayNghi, lyDo); // Gọi method hiển thị dialog sửa
                }
            });

            // Sự kiện Duyệt/Từ chối
            btnDuyet.setOnClickListener(v -> {                             // Set listener cho button duyệt
                boolean success = dbHelper.approveLeaveRequest(maNghiPhep, "Đã duyệt"); // Gọi method duyệt đơn
                if (success) {                                             // Nếu duyệt thành công
                    Toast.makeText(context, "Đã duyệt đơn nghỉ phép", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                    refreshData();                                         // Refresh dữ liệu
                }
            });

            btnTuChoi.setOnClickListener(v -> {                            // Set listener cho button từ chối
                boolean success = dbHelper.approveLeaveRequest(maNghiPhep, "Từ chối"); // Gọi method từ chối đơn
                if (success) {                                             // Nếu từ chối thành công
                    Toast.makeText(context, "Đã từ chối đơn nghỉ phép", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                    refreshData();                                         // Refresh dữ liệu
                }
            });
        }
        
        return convertView;                                                // Trả về view đã cấu hình
    }
```

#### Các method hỗ trợ:
```java
    private void refreshData() {                                           // Method refresh dữ liệu
        if (context instanceof NghiPhepActivity) {                         // Nếu context là NghiPhepActivity
            ((NghiPhepActivity) context).refreshHistory();                 // Gọi method refresh history
        }
    }
    
    public void updateCursor(Cursor newCursor) {                           // Method cập nhật cursor mới
        if (cursor != null) {                                              // Nếu cursor cũ không null
            cursor.close();                                                // Đóng cursor cũ
        }
        cursor = newCursor;                                                // Gán cursor mới
        notifyDataSetChanged();                                            // Thông báo adapter dữ liệu đã thay đổi
    }
}
```

---

## 3️⃣ MODEL CLASS - NghiPhep.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/models/NghiPhep.java`

### Mục đích:
Model class đại diện cho đối tượng nghỉ phép với các thuộc tính và phương thức getter/setter.

### Chi tiết code:

#### Khai báo package và class:
```java
package com.example.btl_mobile_qlns.models;                               // Khai báo package chứa model classes

public class NghiPhep {                                                    // Khai báo class NghiPhep
    private int maNghiPhep;                                                // Mã nghỉ phép (Primary Key)
    private String maNhanVien;                                             // Mã nhân viên (Foreign Key)
    private String ngayBatDau;                                             // Ngày bắt đầu nghỉ phép
    private String ngayKetThuc;                                            // Ngày kết thúc nghỉ phép
    private int soNgayNghi;                                                // Số ngày nghỉ
    private String lyDo;                                                   // Lý do nghỉ phép
    private String trangThai;                                              // Trạng thái đơn (Chờ duyệt, Đã duyệt, Từ chối)
    private String nguoiDuyet;                                             // Người duyệt đơn
```

#### Constructor:
```java
    public NghiPhep() {}                                                   // Constructor mặc định không tham số

    public NghiPhep(int maNghiPhep, String maNhanVien, String ngayBatDau, String ngayKetThuc, // Constructor đầy đủ tham số
                   int soNgayNghi, String lyDo, String trangThai, String nguoiDuyet) {
        this.maNghiPhep = maNghiPhep;                                      // Gán mã nghỉ phép
        this.maNhanVien = maNhanVien;                                      // Gán mã nhân viên
        this.ngayBatDau = ngayBatDau;                                      // Gán ngày bắt đầu
        this.ngayKetThuc = ngayKetThuc;                                    // Gán ngày kết thúc
        this.soNgayNghi = soNgayNghi;                                      // Gán số ngày nghỉ
        this.lyDo = lyDo;                                                  // Gán lý do
        this.trangThai = trangThai;                                        // Gán trạng thái
        this.nguoiDuyet = nguoiDuyet;                                      // Gán người duyệt
    }
```

#### Getter và Setter methods:
```java
    // Getters and Setters
    public int getMaNghiPhep() { return maNghiPhep; }                      // Getter mã nghỉ phép
    public void setMaNghiPhep(int maNghiPhep) { this.maNghiPhep = maNghiPhep; } // Setter mã nghỉ phép

    public String getMaNhanVien() { return maNhanVien; }                   // Getter mã nhân viên
    public void setMaNhanVien(String maNhanVien) { this.maNhanVien = maNhanVien; } // Setter mã nhân viên

    public String getNgayBatDau() { return ngayBatDau; }                   // Getter ngày bắt đầu
    public void setNgayBatDau(String ngayBatDau) { this.ngayBatDau = ngayBatDau; } // Setter ngày bắt đầu

    public String getNgayKetThuc() { return ngayKetThuc; }                 // Getter ngày kết thúc
    public void setNgayKetThuc(String ngayKetThuc) { this.ngayKetThuc = ngayKetThuc; } // Setter ngày kết thúc

    public int getSoNgayNghi() { return soNgayNghi; }                      // Getter số ngày nghỉ
    public void setSoNgayNghi(int soNgayNghi) { this.soNgayNghi = soNgayNghi; } // Setter số ngày nghỉ

    public String getLyDo() { return lyDo; }                               // Getter lý do
    public void setLyDo(String lyDo) { this.lyDo = lyDo; }                 // Setter lý do

    public String getTrangThai() { return trangThai; }                     // Getter trạng thái
    public void setTrangThai(String trangThai) { this.trangThai = trangThai; } // Setter trạng thái

    public String getNguoiDuyet() { return nguoiDuyet; }                   // Getter người duyệt
    public void setNguoiDuyet(String nguoiDuyet) { this.nguoiDuyet = nguoiDuyet; } // Setter người duyệt
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 4️⃣ LAYOUT CHÍNH - activity_nghi_phep.xml

**Đường dẫn**: `app/src/main/res/layout/activity_nghi_phep.xml`

### Mục đích:
Layout chính cho màn hình nghỉ phép, bao gồm form tạo đơn và danh sách lịch sử.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- LinearLayout root với orientation vertical -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:background="#f5f5f5">                                          <!-- Background màu xám nhạt -->
```
#### Header Section:
```xml
    <!-- Header -->
    <LinearLayout                                                          <!-- LinearLayout chứa header -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:gravity="center_vertical"                                  <!-- Căn giữa theo chiều dọc -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:elevation="2dp">                                           <!-- Elevation 2dp tạo shadow -->

        <TextView                                                          <!-- TextView tiêu đề chính -->
            android:id="@+id/tv_title"                                     <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="XIN NGHỈ PHÉP"                                   <!-- Text tiêu đề mặc định -->
            android:textSize="24sp"                                        <!-- Kích thước font 24sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#2196F3"                                    <!-- Màu chữ xanh dương -->
            android:gravity="center" />                                    <!-- Căn giữa text -->

    </LinearLayout>
```

#### Form Section:
```xml
    <!-- Form xin nghỉ phép -->
    <LinearLayout                                                          <!-- LinearLayout chứa form xin nghỉ phép -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="vertical"                                     <!-- Orientation dọc -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:layout_margin="16dp"                                       <!-- Margin 16dp cho tất cả các cạnh -->
        android:elevation="2dp">                                           <!-- Elevation 2dp tạo shadow -->

        <TextView                                                          <!-- TextView tiêu đề form -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="ĐĂNG KÝ NGHỈ PHÉP"                               <!-- Text tiêu đề form -->
            android:textSize="18sp"                                        <!-- Kích thước font 18sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#333"                                       <!-- Màu chữ xám đậm -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->

        <!-- Ngày bắt đầu -->
        <TextView                                                          <!-- TextView label ngày bắt đầu -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Ngày bắt đầu"                                    <!-- Text label -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->

        <EditText                                                          <!-- EditText chọn ngày bắt đầu -->
            android:id="@+id/et_ngay_bat_dau"                              <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:background="@drawable/edit_text_background"            <!-- Background custom drawable -->
            android:padding="12dp"                                         <!-- Padding 12dp cho tất cả các cạnh -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:layout_marginBottom="16dp"                             <!-- Margin bottom 16dp -->
            android:hint="Chọn ngày bắt đầu"                               <!-- Hint text hướng dẫn -->
            android:focusable="false"                                      <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
            android:clickable="true" />                                    <!-- Cho phép click để mở DatePicker -->

        <!-- Ngày kết thúc -->
        <TextView                                                          <!-- TextView label ngày kết thúc -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Ngày kết thúc"                                   <!-- Text label -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->

        <EditText                                                          <!-- EditText chọn ngày kết thúc -->
            android:id="@+id/et_ngay_ket_thuc"                             <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:background="@drawable/edit_text_background"            <!-- Background custom drawable -->
            android:padding="12dp"                                         <!-- Padding 12dp cho tất cả các cạnh -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:layout_marginBottom="16dp"                             <!-- Margin bottom 16dp -->
            android:hint="Chọn ngày kết thúc"                              <!-- Hint text hướng dẫn -->
            android:focusable="false"                                      <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
            android:clickable="true" />                                    <!-- Cho phép click để mở DatePicker -->

        <!-- Số ngày nghỉ -->
        <TextView                                                          <!-- TextView hiển thị số ngày nghỉ -->
            android:id="@+id/tv_so_ngay_nghi"                              <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Số ngày nghỉ: 0 ngày"                            <!-- Text mặc định -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:layout_marginBottom="16dp" />                          <!-- Margin bottom 16dp -->

        <!-- Lý do -->
        <TextView                                                          <!-- TextView label lý do -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:text="Lý do nghỉ phép"                                 <!-- Text label -->
            android:textSize="14sp"                                        <!-- Kích thước font 14sp -->
            android:textColor="#666"                                       <!-- Màu chữ xám -->
            android:layout_marginBottom="4dp" />                           <!-- Margin bottom 4dp -->

        <EditText                                                          <!-- EditText nhập lý do -->
            android:id="@+id/et_ly_do"                                     <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="80dp"                                   <!-- Chiều cao cố định 80dp -->
            android:background="@drawable/edit_text_background"            <!-- Background custom drawable -->
            android:padding="12dp"                                         <!-- Padding 12dp cho tất cả các cạnh -->
            android:textSize="16sp"                                        <!-- Kích thước font 16sp -->
            android:layout_marginBottom="16dp"                             <!-- Margin bottom 16dp -->
            android:hint="Nhập lý do nghỉ phép"                            <!-- Hint text hướng dẫn -->
            android:gravity="top"                                          <!-- Căn text lên trên -->
            android:inputType="textMultiLine"                              <!-- Input type multi line -->
            android:maxLines="3" />                                        <!-- Tối đa 3 dòng -->

        <!-- Button gửi đơn -->
        <Button                                                            <!-- Button gửi đơn xin nghỉ phép -->
            android:id="@+id/btn_xin_nghi_phep"                            <!-- ID để truy cập từ Java code -->
            android:layout_width="match_parent"                            <!-- Chiều rộng bằng parent -->
            android:layout_height="48dp"                                   <!-- Chiều cao cố định 48dp -->
            android:text="GỬI ĐƠN XIN NGHỈ PHÉP"                          <!-- Text button -->
            android:textColor="@color/white"                               <!-- Màu chữ trắng -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:background="@drawable/btn_primary" />                  <!-- Background custom drawable -->

    </LinearLayout>
```

#### History Header Section:
```xml
    <!-- Lịch sử nghỉ phép header -->
    <LinearLayout                                                          <!-- LinearLayout chứa header lịch sử -->
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
            android:text="LỊCH SỬ NGHỈ PHÉP"                               <!-- Text tiêu đề -->
            android:textSize="18sp"                                        <!-- Kích thước font 18sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:textColor="#333" />                                    <!-- Màu chữ xám đậm -->

    </LinearLayout>
```

#### ListView Section:
```xml
    <!-- ListView có thể scroll -->
    <ListView                                                              <!-- ListView hiển thị lịch sử nghỉ phép -->
        android:id="@+id/lv_lich_su_nghi_phep"                             <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="0dp"                                        <!-- Chiều cao 0dp để sử dụng layout_weight -->
        android:layout_weight="1"                                          <!-- Weight 1 để chiếm hết không gian còn lại -->
        android:layout_marginStart="16dp"                                  <!-- Margin start 16dp -->
        android:layout_marginEnd="16dp"                                    <!-- Margin end 16dp -->
        android:layout_marginBottom="16dp"                                 <!-- Margin bottom 16dp -->
        android:background="@android:color/white"                          <!-- Background màu trắng -->
        android:divider="#E0E0E0"                                          <!-- Màu divider xám nhạt -->
        android:dividerHeight="1dp"                                        <!-- Chiều cao divider 1dp -->
        android:elevation="2dp" />                                         <!-- Elevation 2dp tạo shadow -->

</LinearLayout>
```

---

## 5️⃣ LAYOUT ITEM - item_nghi_phep.xml

**Đường dẫn**: `app/src/main/res/layout/item_nghi_phep.xml`

### Mục đích:
Layout cho mỗi item trong ListView lịch sử nghỉ phép, hiển thị thông tin đơn và các button thao tác.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- LinearLayout root với orientation vertical -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace cho app attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
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
    <TextView                                                              <!-- TextView mã nhân viên -->
        android:id="@+id/tv_ma_nv"                                         <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Mã NV: NV001"                                        <!-- Text mặc định -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:textColor="#333"                                           <!-- Màu chữ xám đậm -->
        android:layout_marginBottom="4dp" />                               <!-- Margin bottom 4dp -->

    <TextView                                                              <!-- TextView họ tên nhân viên -->
        android:id="@+id/tv_ho_ten"                                        <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Họ tên: Nguyễn Văn A"                                <!-- Text mặc định -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textColor="#666"                                           <!-- Màu chữ xám -->
        android:layout_marginBottom="8dp"                                  <!-- Margin bottom 8dp -->
        android:visibility="gone" />                                       <!-- Visibility gone (ẩn mặc định) -->
```

#### Leave Information Section:
```xml
    <!-- Thông tin nghỉ phép -->
    <TextView                                                              <!-- TextView ngày nghỉ -->
        android:id="@+id/tv_ngay_nghi"                                     <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Từ 2024-01-15 đến 2024-01-17"                       <!-- Text mặc định -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textColor="#333"                                           <!-- Màu chữ xám đậm -->
        android:layout_marginBottom="4dp" />                               <!-- Margin bottom 4dp -->

    <TextView                                                              <!-- TextView số ngày nghỉ -->
        android:id="@+id/tv_so_ngay"                                       <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Số ngày: 3"                                          <!-- Text mặc định -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textColor="#666"                                           <!-- Màu chữ xám -->
        android:layout_marginBottom="4dp" />                               <!-- Margin bottom 4dp -->

    <TextView                                                              <!-- TextView lý do nghỉ phép -->
        android:id="@+id/tv_ly_do"                                         <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Lý do: Nghỉ phép cá nhân"                            <!-- Text mặc định -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textColor="#666"                                           <!-- Màu chữ xám -->
        android:layout_marginBottom="8dp"                                  <!-- Margin bottom 8dp -->
        android:maxLines="2"                                               <!-- Tối đa 2 dòng -->
        android:ellipsize="end" />                                         <!-- Ellipsize ở cuối nếu text quá dài -->
```

#### Status Section:
```xml
    <!-- Trạng thái -->
    <TextView                                                              <!-- TextView trạng thái đơn -->
        android:id="@+id/tv_trang_thai"                                    <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Trạng thái: Chờ duyệt"                               <!-- Text mặc định -->
        android:textSize="14sp"                                            <!-- Kích thước font 14sp -->
        android:textStyle="bold"                                           <!-- Style chữ đậm -->
        android:layout_marginBottom="8dp" />                               <!-- Margin bottom 8dp -->
```

#### Action Buttons Section:
```xml
    <!-- Buttons thao tác -->
    <LinearLayout                                                          <!-- LinearLayout chứa các button thao tác -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:gravity="end">                                             <!-- Căn về phía end (phải) -->

        <ImageButton                                                       <!-- ImageButton sửa đơn -->
            android:id="@+id/btn_sua_np"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="36dp"                                    <!-- Chiều rộng cố định 36dp -->
            android:layout_height="36dp"                                   <!-- Chiều cao cố định 36dp -->
            android:src="@android:drawable/ic_menu_edit"                   <!-- Icon edit mặc định của Android -->
            android:background="?attr/selectableItemBackgroundBorderless"  <!-- Background ripple effect -->
            android:tint="@android:color/holo_blue_dark"                   <!-- Tint màu xanh dương -->
            android:layout_marginEnd="8dp"                                 <!-- Margin end 8dp -->
            android:visibility="gone" />                                   <!-- Visibility gone (ẩn mặc định) -->

        <ImageButton                                                       <!-- ImageButton xóa đơn -->
            android:id="@+id/btn_xoa_np"                                   <!-- ID để truy cập từ Java code -->
            android:layout_width="36dp"                                    <!-- Chiều rộng cố định 36dp -->
            android:layout_height="36dp"                                   <!-- Chiều cao cố định 36dp -->
            android:src="@android:drawable/ic_menu_delete"                 <!-- Icon delete mặc định của Android -->
            android:background="@drawable/btn_danger"                      <!-- Background custom drawable màu đỏ -->
            app:tint="@android:color/white"                                <!-- Tint màu trắng -->
            android:layout_marginEnd="16dp"                                <!-- Margin end 16dp -->
            android:padding="8dp"                                          <!-- Padding 8dp cho tất cả các cạnh -->
            android:visibility="gone" />                                   <!-- Visibility gone (ẩn mặc định) -->

        <Button                                                            <!-- Button từ chối đơn -->
            android:id="@+id/btn_tu_choi"                                  <!-- ID để truy cập từ Java code -->
            android:layout_width="100dp"                                   <!-- Chiều rộng cố định 100dp -->
            android:layout_height="40dp"                                   <!-- Chiều cao cố định 40dp -->
            android:text="TỪ CHỐI"                                         <!-- Text button -->
            android:textSize="12sp"                                        <!-- Kích thước font 12sp -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:backgroundTint="#F44336"                               <!-- Background tint màu đỏ -->
            android:layout_marginEnd="8dp"                                 <!-- Margin end 8dp -->
            android:visibility="gone" />                                   <!-- Visibility gone (ẩn mặc định) -->

        <Button                                                            <!-- Button duyệt đơn -->
            android:id="@+id/btn_duyet"                                    <!-- ID để truy cập từ Java code -->
            android:layout_width="100dp"                                   <!-- Chiều rộng cố định 100dp -->
            android:layout_height="40dp"                                   <!-- Chiều cao cố định 40dp -->
            android:text="DUYỆT"                                           <!-- Text button -->
            android:textSize="12sp"                                        <!-- Kích thước font 12sp -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:backgroundTint="#4CAF50"                               <!-- Background tint màu xanh lá -->
            android:visibility="gone" />                                   <!-- Visibility gone (ẩn mặc định) -->

    </LinearLayout>

</LinearLayout>
```

---

## 6️⃣ LAYOUT DIALOG - dialog_edit_nghi_phep.xml

**Đường dẫn**: `app/src/main/res/layout/dialog_edit_nghi_phep.xml`

### Mục đích:
Layout dialog để chỉnh sửa thông tin đơn nghỉ phép.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- LinearLayout root với orientation vertical -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ chứa content -->
    android:orientation="vertical"                                         <!-- Orientation dọc -->
    android:padding="20dp">                                                <!-- Padding 20dp cho tất cả các cạnh -->

    <TextView                                                              <!-- TextView label ngày bắt đầu -->
        android:layout_width="wrap_content"                                <!-- Chiều rộng vừa đủ chứa text -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Ngày bắt đầu"                                        <!-- Text label -->
        android:textStyle="bold" />                                        <!-- Style chữ đậm -->
    <EditText                                                              <!-- EditText chỉnh sửa ngày bắt đầu -->
        android:id="@+id/et_edit_start"                                    <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:focusable="false"                                          <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
        android:hint="Chọn ngày" />                                        <!-- Hint text hướng dẫn -->

    <TextView                                                              <!-- TextView label ngày kết thúc -->
        android:layout_width="wrap_content"                                <!-- Chiều rộng vừa đủ chứa text -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Ngày kết thúc"                                       <!-- Text label -->
        android:layout_marginTop="10dp"                                    <!-- Margin top 10dp -->
        android:textStyle="bold" />                                        <!-- Style chữ đậm -->
    <EditText                                                              <!-- EditText chỉnh sửa ngày kết thúc -->
        android:id="@+id/et_edit_end"                                      <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:focusable="false"                                          <!-- Không cho phép focus (chỉ click để mở DatePicker) -->
        android:hint="Chọn ngày" />                                        <!-- Hint text hướng dẫn -->

    <TextView                                                              <!-- TextView hiển thị số ngày -->
        android:id="@+id/tv_edit_so_ngay"                                  <!-- ID để truy cập từ Java code -->
        android:layout_width="wrap_content"                                <!-- Chiều rộng vừa đủ chứa text -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Số ngày: 0"                                          <!-- Text mặc định -->
        android:textColor="@android:color/holo_blue_dark"                  <!-- Màu chữ xanh dương -->
        android:layout_marginTop="5dp" />                                  <!-- Margin top 5dp -->

    <TextView                                                              <!-- TextView label lý do -->
        android:layout_width="wrap_content"                                <!-- Chiều rộng vừa đủ chứa text -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa text -->
        android:text="Lý do nghỉ phép"                                     <!-- Text label -->
        android:layout_marginTop="10dp"                                    <!-- Margin top 10dp -->
        android:textStyle="bold" />                                        <!-- Style chữ đậm -->
    <EditText                                                              <!-- EditText chỉnh sửa lý do -->
        android:id="@+id/et_edit_ly_do"                                    <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:hint="Nhập lý do" />                                       <!-- Hint text hướng dẫn -->

</LinearLayout>
```
---

## 🗄️ CHI TIẾT CÁC PHƯƠNG THỨC DATABASE

## 7️⃣ DATABASE METHODS - DatabaseHelper.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/database/DatabaseHelper.java`

### Mục đích:
Các phương thức database hỗ trợ chức năng nghỉ phép, bao gồm CRUD operations và workflow management.

### Chi tiết các methods:

#### Method addLeaveRequest:
```java
    public boolean addLeaveRequest(String maNhanVien, String ngayBatDau, String ngayKetThuc, // Method thêm đơn nghỉ phép mới
                                  int soNgayNghi, String lyDo) {                           // Nhận tất cả thông tin cần thiết
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu
        values.put("MaNhanVien", maNhanVien);                             // Thêm mã nhân viên
        values.put("NgayBatDau", ngayBatDau);                             // Thêm ngày bắt đầu
        values.put("NgayKetThuc", ngayKetThuc);                           // Thêm ngày kết thúc
        values.put("SoNgayNghi", soNgayNghi);                             // Thêm số ngày nghỉ
        values.put("LyDo", lyDo);                                         // Thêm lý do nghỉ phép
        values.put("TrangThai", "Chờ duyệt");                             // Set trạng thái mặc định là "Chờ duyệt"
        
        long result = db.insert(TABLE_NGHI_PHEP, null, values);           // Insert record vào bảng NghiPhep
        return result != -1;                                              // Trả về true nếu insert thành công
    }
```

#### Method submitLeaveRequest:
```java
    public boolean submitLeaveRequest(String maNhanVien, String ngayBatDau, String ngayKetThuc, // Method gửi đơn nghỉ phép (wrapper)
                                    int soNgayNghi, String lyDo) {                         // Nhận tất cả thông tin cần thiết
        return addLeaveRequest(maNhanVien, ngayBatDau, ngayKetThuc, soNgayNghi, lyDo);    // Gọi method addLeaveRequest
    }
```

#### Method getLeaveRequests:
```java
    public Cursor getLeaveRequests(String maNhanVien) {                    // Method lấy danh sách đơn nghỉ phép của một nhân viên
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT * FROM " + TABLE_NGHI_PHEP +                // Query SELECT tất cả columns
                      " WHERE MaNhanVien = ? ORDER BY NgayBatDau DESC";    // WHERE theo mã nhân viên, ORDER BY ngày bắt đầu giảm dần
        return db.rawQuery(query, new String[]{maNhanVien});              // Thực hiện query với parameter
    }
```

#### Method getLeaveHistory:
```java
    public Cursor getLeaveHistory(String maNhanVien) {                     // Method lấy lịch sử nghỉ phép (wrapper)
        return getLeaveRequests(maNhanVien);                              // Gọi method getLeaveRequests
    }
```

#### Method getAllLeaveRequests:
```java
    public Cursor getAllLeaveRequests() {                                  // Method lấy tất cả đơn nghỉ phép (cho Admin/HR/Manager)
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database instance ở chế độ đọc
        String query = "SELECT np.*, nv.HoTen FROM " + TABLE_NGHI_PHEP + " np " + // Query JOIN với bảng NhanVien để lấy tên
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON np.MaNhanVien = nv.MaNhanVien " + // LEFT JOIN để lấy thông tin nhân viên
                      "ORDER BY np.NgayBatDau DESC";                       // ORDER BY ngày bắt đầu giảm dần
        return db.rawQuery(query, null);                                  // Thực hiện query không có parameter
    }
```

#### Method updateLeaveRequestStatus:
```java
    public boolean updateLeaveRequestStatus(int maNghiPhep, String trangThai, String nguoiDuyet) { // Method cập nhật trạng thái đơn nghỉ phép
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("TrangThai", trangThai);                               // Thêm trạng thái mới
        values.put("NguoiDuyet", nguoiDuyet);                             // Thêm người duyệt
        
        int result = db.update(TABLE_NGHI_PHEP, values,                   // Thực hiện UPDATE
                             "MaNghiPhep = ?", new String[]{String.valueOf(maNghiPhep)}); // WHERE theo mã nghỉ phép
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }
```

#### Method approveLeaveRequest:
```java
    public boolean approveLeaveRequest(int maNghiPhep, String trangThai) { // Method duyệt/từ chối đơn nghỉ phép
        return updateLeaveRequestStatus(maNghiPhep, trangThai, "Admin");   // Gọi method updateLeaveRequestStatus với người duyệt là "Admin"
    }
```

#### Method updateLeaveRequest:
```java
    public boolean updateLeaveRequest(int maNghiPhep, String ngayBatDau, String ngayKetThuc, int soNgayNghi, String lyDo) { // Method cập nhật thông tin đơn nghỉ phép
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("NgayBatDau", ngayBatDau);                             // Cập nhật ngày bắt đầu
        values.put("NgayKetThuc", ngayKetThuc);                           // Cập nhật ngày kết thúc
        values.put("SoNgayNghi", soNgayNghi);                             // Cập nhật số ngày nghỉ
        values.put("LyDo", lyDo);                                         // Cập nhật lý do
        int result = db.update(TABLE_NGHI_PHEP, values, "MaNghiPhep = ?", new String[]{String.valueOf(maNghiPhep)}); // Thực hiện UPDATE WHERE theo mã nghỉ phép
        return result > 0;                                                // Trả về true nếu có record được cập nhật
    }
```

#### Method deleteLeaveRequest:
```java
    public boolean deleteLeaveRequest(int maNghiPhep) {                    // Method xóa đơn nghỉ phép
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database instance ở chế độ ghi
        int result = db.delete(TABLE_NGHI_PHEP, "MaNghiPhep = ?", new String[]{String.valueOf(maNghiPhep)}); // Thực hiện DELETE WHERE theo mã nghỉ phép
        return result > 0;                                                // Trả về true nếu có record được xóa
    }
```

---

## 🔗 TÍCH HỢP VÀ NAVIGATION

### 1. Tích hợp với DashboardActivity:
- **Navigation flow**: DashboardActivity → NghiPhepActivity
- **Intent data**: Truyền username và role qua Intent
- **Role-based access**: Hiển thị chức năng khác nhau theo vai trò

### 2. Tích hợp với Database:
- **Foreign key relationship**: MaNhanVien liên kết với bảng NhanVien
- **Transaction safety**: Sử dụng ContentValues và parameterized queries
- **Data consistency**: Đảm bảo tính toàn vẹn dữ liệu

### 3. Real-time Updates:
- **Auto refresh**: Tự động refresh danh sách sau khi thực hiện thao tác
- **Cursor management**: Quản lý cursor hiệu quả, đóng cursor cũ khi cập nhật
- **UI synchronization**: Đồng bộ UI với dữ liệu database

---

## 📋 QUY TẮC NGHIỆP VỤ

### 1. Quy tắc tạo đơn:
- **Date validation**: Ngày kết thúc phải sau hoặc bằng ngày bắt đầu
- **Future dates**: Chỉ cho phép chọn ngày từ hôm nay trở đi
- **Required fields**: Tất cả trường đều bắt buộc nhập
- **Auto calculation**: Tự động tính số ngày nghỉ dựa trên khoảng thời gian

### 2. Quy tắc phân quyền:
- **Employee**: Chỉ xem đơn của mình, tạo đơn mới, sửa/xóa đơn chờ duyệt
- **Admin/HR/Manager**: Xem tất cả đơn, duyệt/từ chối, sửa/xóa mọi đơn chờ duyệt
- **Status-based actions**: Chỉ có thể sửa/xóa đơn ở trạng thái "Chờ duyệt"

### 3. Quy tắc trạng thái:
- **Initial status**: Đơn mới tạo có trạng thái "Chờ duyệt"
- **Final status**: "Đã duyệt" và "Từ chối" là trạng thái cuối, không thể thay đổi
- **Status colors**: Mỗi trạng thái có màu sắc riêng để dễ nhận biết

---

## 🔒 BẢO MẬT VÀ VALIDATION

### 1. Input validation:
- **Date format**: Kiểm tra format ngày yyyy-MM-dd
- **Date logic**: Ngày kết thúc không được trước ngày bắt đầu
- **Required fields**: Validation tất cả trường bắt buộc
- **Length limits**: Giới hạn độ dài lý do nghỉ phép

### 2. Database security:
- **Parameterized queries**: Sử dụng parameter để tránh SQL injection
- **Role-based data access**: Chỉ cho phép truy cập dữ liệu theo quyền hạn
- **Transaction integrity**: Đảm bảo tính toàn vẹn dữ liệu

### 3. UI security:
- **Dynamic button visibility**: Hiển thị button theo role và trạng thái
- **Confirmation dialogs**: Xác nhận trước khi thực hiện thao tác quan trọng
- **Error handling**: Xử lý lỗi và hiển thị thông báo phù hợp

---

## 📱 TRẢI NGHIỆM NGƯỜI DÙNG

### 1. UI/UX Design:
- **Material Design**: Sử dụng elevation, colors và typography nhất quán
- **Responsive layout**: Sử dụng weight và margin để layout responsive
- **Visual feedback**: Màu sắc trạng thái và animation button
- **Accessibility**: Hint text và label rõ ràng

### 2. Interaction Design:
- **Date pickers**: DatePickerDialog với validation ngày
- **Auto calculation**: Tự động tính số ngày khi chọn ngày
- **One-tap actions**: Button duyệt/từ chối nhanh chóng
- **Edit in place**: Dialog edit không cần chuyển màn hình

### 3. Performance:
- **Efficient queries**: JOIN query để lấy thông tin nhân viên
- **Cursor reuse**: Quản lý cursor hiệu quả
- **Lazy loading**: ListView chỉ load item khi cần thiết
- **Memory management**: Đóng cursor và database connection đúng cách

---

## 🎯 KẾT LUẬN

Module Quản lý Nghỉ phép là một phần quan trọng của hệ thống QLNS, cung cấp:

### Tính năng chính:
- **Complete workflow**: Quy trình nghỉ phép hoàn chỉnh từ tạo đến duyệt
- **Role-based access**: Phân quyền rõ ràng theo vai trò
- **Real-time calculation**: Tính toán số ngày nghỉ tự động
- **Status management**: Quản lý trạng thái đơn nghỉ phép

### Ưu điểm:
- **User-friendly**: Giao diện trực quan, dễ sử dụng
- **Flexible**: Hỗ trợ nhiều vai trò với chức năng khác nhau
- **Reliable**: Validation đầy đủ và xử lý lỗi tốt
- **Efficient**: Performance tối ưu với database queries

### Tích hợp hệ thống:
- **Seamless integration**: Tích hợp mượt mà với hệ thống phân quyền
- **Database consistency**: Đảm bảo tính nhất quán dữ liệu
- **Scalable architecture**: Có thể mở rộng thêm tính năng

Module này đóng vai trò quan trọng trong việc quản lý nhân sự, giúp tự động hóa quy trình nghỉ phép và tăng hiệu quả công việc.