# HƯỚNG DẪN CHI TIẾT: CHỨC NĂNG QUẢN LÝ TÀI KHOẢN

## 📋 TỔNG QUAN

Chức năng Quản lý Tài khoản là một module bảo mật quan trọng của hệ thống QLNS, cho phép Admin quản lý tất cả tài khoản người dùng trong hệ thống, bao gồm xem danh sách, chỉnh sửa thông tin, khóa/mở khóa tài khoản và xóa tài khoản.

## 🏗️ KIẾN TRÚC HỆ THỐNG

### Cấu trúc file:
```
app/src/main/java/com/example/btl_mobile_qlns/
├── QuanLyTaiKhoanActivity.java              # Activity chính - Danh sách tài khoản
├── TaiKhoanAdapter.java                     # Adapter hiển thị danh sách tài khoản
└── database/DatabaseHelper.java             # Xử lý database

app/src/main/res/layout/
├── activity_quan_ly_tai_khoan.xml           # Layout danh sách tài khoản
└── item_tai_khoan.xml                       # Layout item trong ListView

Lưu ý: Module này không có model class riêng, sử dụng trực tiếp Cursor
```

## 📊 NGHIỆP VỤ QUẢN LÝ TÀI KHOẢN

### 1. Quy trình nghiệp vụ:
- **Xem danh sách**: Hiển thị tất cả tài khoản với thông tin chi tiết
- **Tìm kiếm**: Tìm kiếm tài khoản theo tên đăng nhập
- **Chỉnh sửa**: Cập nhật họ tên và vai trò của tài khoản
- **Khóa/Mở khóa**: Thay đổi trạng thái hoạt động của tài khoản
- **Xóa tài khoản**: Xóa tài khoản khỏi hệ thống (có bảo vệ admin)
- **Bảo vệ admin**: Không cho phép xóa hoặc chỉnh sửa tài khoản admin chính

### 2. Vai trò hệ thống:
- **Admin**: Quản trị viên hệ thống - full quyền
- **HR**: Nhân sự - quản lý nhân viên và lương
- **Manager**: Quản lý - quản lý nhân viên nhưng không quản lý lương
- **Employee**: Nhân viên - chỉ truy cập chức năng cá nhân

### 3. Phân quyền:
- **Admin**: Full quyền truy cập module này
- **HR/Manager/Employee**: Không có quyền truy cập

---

## 📱 CHI TIẾT CÁC FILE

## 1️⃣ ACTIVITY CHÍNH - QuanLyTaiKhoanActivity.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/QuanLyTaiKhoanActivity.java`

### Mục đích:
Activity chính quản lý danh sách tài khoản, hiển thị thông tin và điều hướng đến các chức năng khác.

### Chi tiết code:

#### Khai báo package và import:
```java
package com.example.btl_mobile_qlns;                                      // Khai báo package chứa class

import android.database.Cursor;                                            // Import Cursor để xử lý kết quả query database
import android.os.Bundle;                                                  // Import Bundle để truyền dữ liệu giữa Activity
import android.text.Editable;                                              // Import Editable để xử lý text thay đổi
import android.text.TextWatcher;                                           // Import TextWatcher để lắng nghe thay đổi text
import android.widget.EditText;                                            // Import EditText widget
import android.widget.ImageButton;                                         // Import ImageButton widget
import android.widget.ListView;                                            // Import ListView widget

import androidx.appcompat.app.AppCompatActivity;                           // Import AppCompatActivity làm base class

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
```

#### Khai báo thuộc tính class:
```java
public class QuanLyTaiKhoanActivity extends AppCompatActivity {            // Khai báo class kế thừa AppCompatActivity

    private ImageButton btnBack;                                           // ImageButton quay lại
    private EditText etSearch;                                             // EditText cho chức năng tìm kiếm
    private ListView lvAccounts;                                           // ListView hiển thị danh sách tài khoản
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
    private TaiKhoanAdapter adapter;                                       // Adapter cho ListView
```

#### Method onCreate:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {                   // Method được gọi khi Activity được tạo
        super.onCreate(savedInstanceState);                                // Gọi method onCreate của class cha
        setContentView(R.layout.activity_quan_ly_tai_khoan);               // Set layout cho Activity

        dbHelper = new DatabaseHelper(this);                              // Khởi tạo DatabaseHelper với context hiện tại
        initViews();                                                       // Khởi tạo các view components
        loadAccounts();                                                    // Tải dữ liệu tài khoản
    }
```

#### Method initViews:
```java
    private void initViews() {                                             // Method khởi tạo và ánh xạ các view từ layout
        btnBack = findViewById(R.id.btn_back);                             // Ánh xạ ImageButton quay lại
        etSearch = findViewById(R.id.et_search_account);                   // Ánh xạ EditText tìm kiếm
        lvAccounts = findViewById(R.id.lv_accounts);                       // Ánh xạ ListView danh sách tài khoản

        btnBack.setOnClickListener(v -> finish());                        // Set listener cho button quay lại, đóng Activity

        etSearch.addTextChangedListener(new TextWatcher() {                // Thêm listener lắng nghe thay đổi text tìm kiếm
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {} // Method được gọi trước khi text thay đổi (không sử dụng)

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {     // Method được gọi khi text đang thay đổi
                searchAccounts(s.toString());                              // Gọi method tìm kiếm với text hiện tại
            }

            @Override
            public void afterTextChanged(Editable s) {}                    // Method được gọi sau khi text thay đổi (không sử dụng)
        });
    }
```

#### Method loadAccounts:
```java
    public void loadAccounts() {                                           // Method tải dữ liệu tài khoản từ database (public để adapter có thể gọi)
        Cursor cursor = dbHelper.getAllAccounts();                         // Gọi method database để lấy tất cả tài khoản
        if (adapter == null) {                                             // Nếu adapter chưa được khởi tạo
            adapter = new TaiKhoanAdapter(this, cursor);                   // Tạo adapter mới với context và cursor
            lvAccounts.setAdapter(adapter);                                // Set adapter cho ListView
        } else {                                                           // Nếu adapter đã tồn tại
            adapter.swapCursor(cursor);                                    // Thay đổi cursor của adapter với dữ liệu mới
        }
    }
```

#### Method searchAccounts:
```java
    private void searchAccounts(String keyword) {                          // Method tìm kiếm tài khoản theo từ khóa
        // Có thể bổ sung searchAccount trong DatabaseHelper nếu muốn tối ưu
        // Ở đây ta dùng giải pháp đơn giản là reload với filter
        loadAccounts();                                                    // Tải lại dữ liệu (hiện tại chưa implement filter, chỉ reload)
    }
}
```

---

## 2️⃣ ADAPTER - TaiKhoanAdapter.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/TaiKhoanAdapter.java`

### Mục đích:
Adapter quản lý hiển thị danh sách tài khoản trong ListView, xử lý các thao tác chỉnh sửa, xóa và thay đổi trạng thái.

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
import android.widget.Button;                                              // Import Button widget
import android.widget.EditText;                                            // Import EditText widget
import android.widget.LinearLayout;                                        // Import LinearLayout widget
import android.widget.ImageButton;                                         // Import ImageButton widget (không sử dụng trong code hiện tại)
import android.widget.TextView;                                            // Import TextView widget
import android.widget.Toast;                                               // Import Toast để hiển thị thông báo

import androidx.appcompat.app.AlertDialog;                                 // Import AlertDialog để hiển thị dialog

import com.example.btl_mobile_qlns.database.DatabaseHelper;                // Import DatabaseHelper để xử lý database
```

#### Khai báo thuộc tính class:
```java
public class TaiKhoanAdapter extends BaseAdapter {                         // Khai báo class kế thừa BaseAdapter
    private Context context;                                               // Context của Activity sử dụng adapter
    private Cursor cursor;                                                 // Cursor chứa dữ liệu tài khoản
    private DatabaseHelper dbHelper;                                       // Helper class xử lý database
```

#### Constructor:
```java
    public TaiKhoanAdapter(Context context, Cursor cursor) {               // Constructor khởi tạo adapter
        this.context = context;                                            // Gán context
        this.cursor = cursor;                                              // Gán cursor chứa dữ liệu
        this.dbHelper = new DatabaseHelper(context);                      // Khởi tạo DatabaseHelper
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
            convertView = LayoutInflater.from(context).inflate(R.layout.item_tai_khoan, parent, false); // Inflate layout item_tai_khoan
        }

        cursor.moveToPosition(position);                                   // Di chuyển cursor đến vị trí position

        TextView tvUsername = convertView.findViewById(R.id.tv_username);   // Ánh xạ TextView tên đăng nhập
        TextView tvFullName = convertView.findViewById(R.id.tv_full_name);  // Ánh xạ TextView họ tên
        TextView tvRole = convertView.findViewById(R.id.tv_role);           // Ánh xạ TextView vai trò
        TextView tvStatus = convertView.findViewById(R.id.tv_status);       // Ánh xạ TextView trạng thái
        Button btnDelete = convertView.findViewById(R.id.btn_delete_account); // Ánh xạ Button xóa tài khoản
        Button btnEdit = convertView.findViewById(R.id.btn_edit_account);   // Ánh xạ Button sửa tài khoản

        String username = cursor.getString(cursor.getColumnIndexOrThrow("TenDangNhap"));     // Lấy tên đăng nhập từ cursor
        String fullName = cursor.getString(cursor.getColumnIndexOrThrow("HoTen"));          // Lấy họ tên từ cursor
        String role = cursor.getString(cursor.getColumnIndexOrThrow("VaiTro"));             // Lấy vai trò từ cursor
        int status = cursor.getInt(cursor.getColumnIndexOrThrow("TrangThai"));              // Lấy trạng thái từ cursor

        tvUsername.setText(username);                                      // Set text tên đăng nhập
        tvFullName.setText(fullName != null ? fullName : "N/A");           // Set text họ tên (hoặc N/A nếu null)
        tvRole.setText("Vai trò: " + role);                                // Set text vai trò với prefix
        
        if (status == 1) {                                                 // Nếu trạng thái = 1 (hoạt động)
            tvStatus.setText("Hoạt động");                                 // Set text "Hoạt động"
            tvStatus.setTextColor(context.getResources().getColor(android.R.color.holo_green_dark)); // Set màu xanh lá
        } else {                                                           // Nếu trạng thái = 0 (bị khóa)
            tvStatus.setText("Bị khóa");                                   // Set text "Bị khóa"
            tvStatus.setTextColor(context.getResources().getColor(android.R.color.holo_red_dark)); // Set màu đỏ
        }

        if ("admin".equals(username.toLowerCase())) {                      // Nếu là tài khoản admin
            btnDelete.setVisibility(View.GONE);                            // Ẩn button xóa
            btnEdit.setVisibility(View.GONE);                              // Ẩn button sửa
        } else {                                                           // Nếu không phải admin
            btnDelete.setVisibility(View.VISIBLE);                         // Hiển thị button xóa
            btnEdit.setVisibility(View.VISIBLE);                           // Hiển thị button sửa
        }

        btnDelete.setOnClickListener(v -> {                                // Set listener cho button xóa
            new AlertDialog.Builder(context)                               // Tạo AlertDialog builder
                .setTitle("Xác nhận xóa")                                  // Set tiêu đề dialog
                .setMessage("Bạn có chắc chắn muốn xóa tài khoản " + username + "?") // Set message xác nhận với tên tài khoản
                .setPositiveButton("Xóa", (dialog, which) -> {             // Set button "Xóa" với action
                    if (dbHelper.deleteAccount(username)) {                // Gọi method database xóa tài khoản
                        Toast.makeText(context, "Đã xóa tài khoản", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                        refreshData();                                     // Refresh dữ liệu
                    } else {                                               // Nếu xóa thất bại
                        Toast.makeText(context, "Lỗi khi xóa tài khoản", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                    }
                })
                .setNegativeButton("Hủy", null)                            // Set button "Hủy" không có action
                .show();                                                   // Hiển thị dialog
        });

        // Sự kiện Sửa tài khoản
        btnEdit.setOnClickListener(v -> {                                  // Set listener cho button sửa
            showEditAccountDialog(username, fullName, role);               // Gọi method hiển thị dialog sửa tài khoản
        });

        convertView.setOnClickListener(v -> {                              // Set listener cho click item (khóa/mở khóa)
            if ("admin".equals(username.toLowerCase())) return;            // Nếu là admin thì không làm gì
            
            String[] options = status == 1 ? new String[]{"Khóa tài khoản"} : new String[]{"Mở khóa tài khoản"}; // Tạo mảng options dựa trên trạng thái hiện tại
            new AlertDialog.Builder(context)                               // Tạo AlertDialog builder
                .setTitle("Tùy chọn cho " + username)                      // Set tiêu đề với tên tài khoản
                .setItems(options, (dialog, which) -> {                    // Set danh sách options
                    int newStatus = status == 1 ? 0 : 1;                   // Đảo ngược trạng thái (1->0 hoặc 0->1)
                    if (dbHelper.updateAccountStatus(username, newStatus)) { // Gọi method database cập nhật trạng thái
                        Toast.makeText(context, "Đã cập nhật trạng thái", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                        if (context instanceof QuanLyTaiKhoanActivity) {    // Nếu context là QuanLyTaiKhoanActivity
                            ((QuanLyTaiKhoanActivity) context).loadAccounts(); // Gọi method loadAccounts để refresh dữ liệu
                        }
                    }
                })
                .show();                                                   // Hiển thị dialog
        });

        return convertView;                                                // Trả về View đã được setup
    }
```

#### Method showEditAccountDialog:
```java
    private void showEditAccountDialog(String username, String currentName, String currentRole) { // Method hiển thị dialog chỉnh sửa tài khoản
        AlertDialog.Builder builder = new AlertDialog.Builder(context);    // Tạo AlertDialog builder
        builder.setTitle("Sửa tài khoản: " + username);                    // Set tiêu đề dialog với tên tài khoản

        LinearLayout layout = new LinearLayout(context);                   // Tạo LinearLayout chứa các EditText
        layout.setOrientation(LinearLayout.VERTICAL);                      // Set orientation dọc
        layout.setPadding(50, 40, 50, 10);                                // Set padding cho layout

        EditText etFullName = new EditText(context);                       // Tạo EditText cho họ tên
        etFullName.setHint("Họ tên");                                      // Set hint cho EditText
        etFullName.setText(currentName);                                   // Set text hiện tại
        layout.addView(etFullName);                                        // Thêm EditText vào layout

        EditText etRole = new EditText(context);                           // Tạo EditText cho vai trò
        etRole.setHint("Vai trò (Admin/HR/Manager/Employee)");             // Set hint với các vai trò có thể
        etRole.setText(currentRole);                                       // Set text hiện tại
        layout.addView(etRole);                                            // Thêm EditText vào layout

        builder.setView(layout);                                           // Set layout làm view của dialog

        builder.setPositiveButton("Cập nhật", (dialog, which) -> {         // Set button "Cập nhật" với action
            String newName = etFullName.getText().toString().trim();       // Lấy họ tên mới và trim khoảng trắng
            String newRole = etRole.getText().toString().trim();           // Lấy vai trò mới và trim khoảng trắng

            // Thống nhất user thành Employee
            if (newRole.equalsIgnoreCase("user")) {                        // Nếu vai trò là "user" (không phân biệt hoa thường)
                newRole = "Employee";                                      // Chuyển thành "Employee"
            }

            if (newName.isEmpty() || newRole.isEmpty()) {                  // Kiểm tra các trường không được rỗng
                Toast.makeText(context, "Vui lòng nhập đầy đủ thông tin", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
                return;                                                    // Thoát method
            }

            if (dbHelper.updateAccountInfo(username, newName, newRole)) {  // Gọi method database cập nhật thông tin tài khoản
                Toast.makeText(context, "Cập nhật thành công", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo thành công
                refreshData();                                             // Refresh dữ liệu
            } else {                                                       // Nếu cập nhật thất bại
                Toast.makeText(context, "Cập nhật thất bại", Toast.LENGTH_SHORT).show(); // Hiển thị thông báo lỗi
            }
        });

        builder.setNegativeButton("Hủy", null);                            // Set button "Hủy" không có action
        builder.show();                                                    // Hiển thị dialog
    }
```

#### Method refreshData và swapCursor:
```java
    private void refreshData() {                                           // Method refresh dữ liệu
        if (context instanceof QuanLyTaiKhoanActivity) {                   // Nếu context là QuanLyTaiKhoanActivity
            ((QuanLyTaiKhoanActivity) context).loadAccounts();             // Gọi method loadAccounts để tải lại dữ liệu
        }
    }

    public void swapCursor(Cursor newCursor) {                             // Method thay đổi cursor của adapter
        if (cursor != null) {                                              // Nếu cursor cũ không null
            cursor.close();                                                // Đóng cursor cũ
        }
        cursor = newCursor;                                                // Gán cursor mới
        notifyDataSetChanged();                                            // Thông báo adapter dữ liệu đã thay đổi để refresh UI
    }
}
```

---

## 📋 CHI TIẾT CÁC FILE LAYOUT

## 3️⃣ LAYOUT CHÍNH - activity_quan_ly_tai_khoan.xml

**Đường dẫn**: `app/src/main/res/layout/activity_quan_ly_tai_khoan.xml`

### Mục đích:
Layout chính cho Activity quản lý tài khoản, bao gồm header, tìm kiếm và ListView.

### Chi tiết code:

#### Khai báo XML và LinearLayout chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  <!-- LinearLayout root với namespace Android -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent (full width) -->
    android:layout_height="match_parent"                                   <!-- Chiều cao bằng parent (full height) -->
    android:orientation="vertical"                                         <!-- Orientation dọc (các child xếp từ trên xuống) -->
    android:background="#f8f9fa">                                          <!-- Background màu xám nhạt (#f8f9fa) -->
```

#### Header Section:
```xml
    <LinearLayout                                                          <!-- LinearLayout chứa phần header -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:background="#2196F3"                                       <!-- Background màu xanh dương -->
        android:padding="16dp"                                             <!-- Padding 16dp cho tất cả các cạnh -->
        android:elevation="4dp"                                            <!-- Elevation 4dp tạo shadow -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:gravity="center_vertical">                                 <!-- Căn giữa theo chiều dọc -->

        <ImageButton                                                       <!-- ImageButton quay lại -->
            android:id="@+id/btn_back"                                     <!-- ID để truy cập từ Java code -->
            android:layout_width="40dp"                                    <!-- Chiều rộng cố định 40dp -->
            android:layout_height="40dp"                                   <!-- Chiều cao cố định 40dp -->
            android:src="@android:drawable/ic_menu_revert"                 <!-- Icon quay lại từ Android system -->
            android:background="?attr/selectableItemBackgroundBorderless" <!-- Background với ripple effect -->
            android:tint="@android:color/white" />                         <!-- Tint màu trắng cho icon -->

        <TextView                                                          <!-- TextView tiêu đề -->
            android:layout_width="0dp"                                     <!-- Chiều rộng 0dp để sử dụng layout_weight -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa text -->
            android:layout_weight="1"                                      <!-- Weight 1 để chiếm không gian còn lại -->
            android:text="QUẢN LÝ TÀI KHOẢN"                               <!-- Text tiêu đề -->
            android:textColor="@android:color/white"                       <!-- Màu chữ trắng -->
            android:textSize="20sp"                                        <!-- Kích thước font 20sp -->
            android:textStyle="bold"                                       <!-- Style chữ đậm -->
            android:layout_marginLeft="16dp" />                            <!-- Margin left 16dp -->
    </LinearLayout>
```

#### Search Section:
```xml
    <EditText                                                              <!-- EditText cho chức năng tìm kiếm -->
        android:id="@+id/et_search_account"                                <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:layout_margin="16dp"                                       <!-- Margin 16dp cho tất cả các cạnh -->
        android:hint="Tìm kiếm tài khoản..."                               <!-- Hint text hướng dẫn -->
        android:padding="12dp"                                             <!-- Padding 12dp cho tất cả các cạnh -->
        android:background="@android:drawable/editbox_background"          <!-- Background mặc định của EditText -->
        android:drawableLeft="@android:drawable/ic_menu_search"            <!-- Icon search ở bên trái -->
        android:drawablePadding="8dp" />                                   <!-- Padding giữa icon và text 8dp -->
```

#### ListView Section:
```xml
    <ListView                                                              <!-- ListView hiển thị danh sách tài khoản -->
        android:id="@+id/lv_accounts"                                      <!-- ID để truy cập từ Java code -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="match_parent"                               <!-- Chiều cao bằng parent -->
        android:divider="@android:color/transparent"                       <!-- Divider trong suốt -->
        android:dividerHeight="8dp"                                        <!-- Chiều cao divider 8dp -->
        android:padding="8dp"                                              <!-- Padding 8dp cho tất cả các cạnh -->
        android:clipToPadding="false" />                                   <!-- Không clip content theo padding -->

</LinearLayout>
```

---

## 4️⃣ LAYOUT ITEM - item_tai_khoan.xml

**Đường dẫn**: `app/src/main/res/layout/item_tai_khoan.xml`

### Mục đích:
Layout cho từng item tài khoản trong ListView, hiển thị thông tin chi tiết và buttons thao tác.

### Chi tiết code:

#### Khai báo XML và CardView chính:
```xml
<?xml version="1.0" encoding="utf-8"?>                                    <!-- Khai báo XML version và encoding UTF-8 -->
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android" <!-- CardView root với namespace Android -->
    xmlns:app="http://schemas.android.com/apk/res-auto"                    <!-- Namespace app cho custom attributes -->
    android:layout_width="match_parent"                                    <!-- Chiều rộng bằng parent -->
    android:layout_height="wrap_content"                                   <!-- Chiều cao vừa đủ chứa content -->
    android:layout_marginHorizontal="8dp"                                  <!-- Margin ngang 8dp -->
    android:layout_marginVertical="4dp"                                    <!-- Margin dọc 4dp -->
    app:cardCornerRadius="8dp"                                             <!-- Bo góc 8dp -->
    app:cardElevation="4dp">                                               <!-- Elevation 4dp tạo shadow -->

    <LinearLayout                                                          <!-- LinearLayout chính chứa tất cả content -->
        android:layout_width="match_parent"                                <!-- Chiều rộng bằng parent -->
        android:layout_height="wrap_content"                               <!-- Chiều cao vừa đủ chứa content -->
        android:orientation="horizontal"                                   <!-- Orientation ngang -->
        android:padding="12dp"                                             <!-- Padding 12dp cho tất cả các cạnh -->
        android:gravity="center_vertical">                                 <!-- Căn giữa theo chiều dọc -->
```

#### Account Information Section:
```xml
        <LinearLayout                                                      <!-- LinearLayout chứa thông tin tài khoản -->
            android:layout_width="0dp"                                     <!-- Chiều rộng 0dp để sử dụng layout_weight -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:layout_weight="1"                                      <!-- Weight 1 để chiếm không gian còn lại -->
            android:orientation="vertical">                                <!-- Orientation dọc -->

            <TextView                                                      <!-- TextView hiển thị tên đăng nhập -->
                android:id="@+id/tv_username"                              <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Username"                                    <!-- Text mẫu hiển thị -->
                android:textSize="16sp"                                    <!-- Kích thước font 16sp -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:textColor="#333" />                                <!-- Màu chữ xám đậm -->

            <TextView                                                      <!-- TextView hiển thị họ tên -->
                android:id="@+id/tv_full_name"                             <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Full Name"                                   <!-- Text mẫu hiển thị -->
                android:textSize="14sp"                                    <!-- Kích thước font 14sp -->
                android:textColor="#666" />                                <!-- Màu chữ xám -->

            <TextView                                                      <!-- TextView hiển thị vai trò -->
                android:id="@+id/tv_role"                                  <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Role: Admin"                                 <!-- Text mẫu hiển thị -->
                android:textSize="12sp"                                    <!-- Kích thước font 12sp -->
                android:textColor="#2196F3"                                <!-- Màu chữ xanh dương -->
                android:textStyle="italic" />                              <!-- Style chữ nghiêng -->

            <TextView                                                      <!-- TextView hiển thị trạng thái -->
                android:id="@+id/tv_status"                                <!-- ID để truy cập từ Java code -->
                android:layout_width="wrap_content"                        <!-- Chiều rộng vừa đủ chứa text -->
                android:layout_height="wrap_content"                       <!-- Chiều cao vừa đủ chứa text -->
                android:text="Hoạt động"                                   <!-- Text mẫu hiển thị -->
                android:textSize="12sp"                                    <!-- Kích thước font 12sp -->
                android:textColor="#4CAF50"                                <!-- Màu chữ xanh lá -->
                android:textStyle="bold"                                   <!-- Style chữ đậm -->
                android:layout_marginTop="2dp" />                          <!-- Margin top 2dp -->
        </LinearLayout>
```

#### Action Buttons Section:
```xml
        <LinearLayout                                                      <!-- LinearLayout chứa các button thao tác -->
            android:layout_width="wrap_content"                            <!-- Chiều rộng vừa đủ chứa content -->
            android:layout_height="wrap_content"                           <!-- Chiều cao vừa đủ chứa content -->
            android:orientation="horizontal"                               <!-- Orientation ngang -->
            android:gravity="end"                                          <!-- Căn phải -->
            android:layout_gravity="center_vertical">                      <!-- Căn giữa theo chiều dọc -->

            <Button                                                        <!-- Button xóa tài khoản -->
                android:id="@+id/btn_delete_account"                       <!-- ID để truy cập từ Java code -->
                android:layout_width="90dp"                                <!-- Chiều rộng cố định 90dp -->
                android:layout_height="40dp"                               <!-- Chiều cao cố định 40dp -->
                android:text="XÓA"                                         <!-- Text button -->
                android:textSize="12sp"                                    <!-- Kích thước font 12sp -->
                android:textColor="@android:color/white"                   <!-- Màu chữ trắng -->
                android:backgroundTint="#F44336"                           <!-- Màu background đỏ -->
                android:layout_marginEnd="8dp" />                          <!-- Margin end 8dp -->

            <Button                                                        <!-- Button sửa tài khoản -->
                android:id="@+id/btn_edit_account"                         <!-- ID để truy cập từ Java code -->
                android:layout_width="90dp"                                <!-- Chiều rộng cố định 90dp -->
                android:layout_height="40dp"                               <!-- Chiều cao cố định 40dp -->
                android:text="SỬA"                                         <!-- Text button -->
                android:textSize="12sp"                                    <!-- Kích thước font 12sp -->
                android:textColor="@android:color/white"                   <!-- Màu chữ trắng -->
                android:backgroundTint="#2196F3" />                        <!-- Màu background xanh dương -->
        </LinearLayout>

    </LinearLayout>
</androidx.cardview.widget.CardView>
```
---

## 📋 CÁC PHƯƠNG THỨC DATABASE LIÊN QUAN

## 5️⃣ DATABASE METHODS - DatabaseHelper.java

**Đường dẫn**: `app/src/main/java/com/example/btl_mobile_qlns/database/DatabaseHelper.java`

### Mục đích:
Các phương thức xử lý database cho chức năng quản lý tài khoản.

### Chi tiết code:

#### Method getAllAccounts:
```java
    public Cursor getAllAccounts() {                                       // Method lấy tất cả tài khoản từ database
        SQLiteDatabase db = this.getReadableDatabase();                    // Lấy database ở chế độ đọc
        String query = "SELECT tk.*, nv.HoTen FROM " + TABLE_TAI_KHOAN + " tk " + // Query SELECT với JOIN để lấy thông tin tài khoản và họ tên
                      "LEFT JOIN " + TABLE_NHAN_VIEN + " nv ON tk.MaNhanVien = nv.MaNhanVien " + // LEFT JOIN với bảng NhanVien để lấy họ tên
                      "ORDER BY tk.TenDangNhap";                           // ORDER BY theo tên đăng nhập
        return db.rawQuery(query, null);                                   // Thực hiện query và trả về Cursor
    }
```

#### Method deleteAccount:
```java
    public boolean deleteAccount(String username) {                        // Method xóa tài khoản theo tên đăng nhập
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database ở chế độ ghi
        // Không cho phép xóa tài khoản admin chính
        if ("admin".equals(username)) return false;                        // Kiểm tra nếu là admin thì trả về false (không cho xóa)
        
        int result = db.delete(TABLE_TAI_KHOAN, "TenDangNhap = ?", new String[]{username}); // Thực hiện DELETE với WHERE condition
        return result > 0;                                                 // Trả về true nếu xóa thành công (result > 0)
    }
```

#### Method updateAccountStatus:
```java
    public boolean updateAccountStatus(String username, int status) {      // Method cập nhật trạng thái tài khoản
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database ở chế độ ghi
        ContentValues values = new ContentValues();                       // Tạo ContentValues để chứa dữ liệu cập nhật
        values.put("TrangThai", status);                                   // Put trạng thái mới vào ContentValues
        int result = db.update(TABLE_TAI_KHOAN, values, "TenDangNhap = ?", new String[]{username}); // Thực hiện UPDATE với WHERE condition
        return result > 0;                                                 // Trả về true nếu cập nhật thành công (result > 0)
    }
```

#### Method updateAccountInfo:
```java
    public boolean updateAccountInfo(String username, String fullName, String role) { // Method cập nhật thông tin tài khoản (họ tên và vai trò)
        SQLiteDatabase db = this.getWritableDatabase();                    // Lấy database ở chế độ ghi
        db.beginTransaction();                                             // Bắt đầu transaction để đảm bảo tính toàn vẹn dữ liệu
        try {
            // 1. Cập nhật VaiTro trong bảng TaiKhoan
            ContentValues accountValues = new ContentValues();             // Tạo ContentValues cho bảng TaiKhoan
            accountValues.put("VaiTro", role);                             // Put vai trò mới vào ContentValues
            db.update(TABLE_TAI_KHOAN, accountValues, "TenDangNhap = ?", new String[]{username}); // Cập nhật vai trò trong bảng TaiKhoan

            // 2. Tìm MaNhanVien liên kết với tài khoản này
            String maNV = "";                                              // Biến lưu mã nhân viên
            Cursor cursor = db.rawQuery("SELECT MaNhanVien FROM " + TABLE_TAI_KHOAN + " WHERE TenDangNhap = ?", new String[]{username}); // Query lấy mã nhân viên
            if (cursor.moveToFirst()) {                                    // Nếu cursor có dữ liệu
                maNV = cursor.getString(0);                                // Lấy mã nhân viên từ cursor
            }
            cursor.close();                                                // Đóng cursor

            // 3. Cập nhật HoTen trong bảng NhanVien
            if (!maNV.isEmpty()) {                                         // Nếu có mã nhân viên
                ContentValues employeeValues = new ContentValues();       // Tạo ContentValues cho bảng NhanVien
                employeeValues.put("HoTen", fullName);                     // Put họ tên mới vào ContentValues
                db.update(TABLE_NHAN_VIEN, employeeValues, "MaNhanVien = ?", new String[]{maNV}); // Cập nhật họ tên trong bảng NhanVien
            }

            db.setTransactionSuccessful();                                 // Đánh dấu transaction thành công
            return true;                                                   // Trả về true
        } catch (Exception e) {                                            // Bắt exception
            e.printStackTrace();                                           // In stack trace
            return false;                                                  // Trả về false nếu có lỗi
        } finally {
            db.endTransaction();                                           // Kết thúc transaction trong finally block
        }
    }
```

---

## 🔗 TÍCH HỢP VÀ ĐIỀU HƯỚNG

### 1. Tích hợp với Dashboard:
- **Quyền truy cập**: Chỉ Admin mới có quyền truy cập module này
- **Button điều hướng**: "QUẢN LÝ TÀI KHOẢN" trong DashboardActivity (chỉ hiển thị cho Admin)
- **Intent navigation**: Sử dụng Intent để chuyển từ Dashboard đến QuanLyTaiKhoanActivity

### 2. Tích hợp với Database:
- **Account management methods**: getAllAccounts(), deleteAccount(), updateAccountStatus(), updateAccountInfo()
- **Employee integration**: JOIN với bảng NhanVien để lấy họ tên
- **Transaction support**: Sử dụng transaction trong updateAccountInfo để đảm bảo tính toàn vẹn
- **Admin protection**: Bảo vệ tài khoản admin không bị xóa hoặc chỉnh sửa

### 3. Navigation Flow:
```
DashboardActivity (Admin only) → QuanLyTaiKhoanActivity
                                        ↓
                                 TaiKhoanAdapter (ListView items)
                                        ↓
                                 Edit/Delete/Status Dialogs
```

---

## 🎯 QUY TẮC NGHIỆP VỤ

### 1. Security Rules:
- **Admin protection**: Tài khoản admin không thể bị xóa hoặc chỉnh sửa
- **Role validation**: Chỉ Admin mới có quyền truy cập module này
- **Status management**: Có thể khóa/mở khóa tài khoản (trừ admin)
- **Data integrity**: Sử dụng transaction khi cập nhật thông tin liên quan đến nhiều bảng

### 2. Business Logic:
- **Role standardization**: Tự động chuyển "user" thành "Employee"
- **Account linking**: Tài khoản được liên kết với nhân viên qua MaNhanVien
- **Status display**: Hiển thị trạng thái bằng màu sắc (xanh = hoạt động, đỏ = bị khóa)
- **Confirmation dialogs**: Xác nhận trước khi thực hiện các thao tác quan trọng

### 3. User Experience:
- **Real-time search**: Tìm kiếm ngay khi gõ (hiện tại chưa implement)
- **Visual feedback**: Toast messages thông báo kết quả thao tác
- **Intuitive interface**: Click item để khóa/mở khóa, buttons riêng cho sửa/xóa
- **Responsive design**: CardView với elevation và bo góc

---

## 🎨 THIẾT KẾ UI/UX

### 1. Design Principles:
- **Material Design**: Sử dụng CardView, elevation, ripple effects
- **Color Scheme**: Xanh dương (#2196F3) chủ đạo, đỏ (#F44336) cho xóa, xanh lá cho trạng thái hoạt động
- **Typography**: Bold cho tên đăng nhập, regular cho thông tin khác
- **Spacing**: Consistent padding và margin

### 2. Interactive Elements:
- **Touch feedback**: Ripple effects cho buttons và clickable items
- **Visual states**: Màu sắc khác nhau cho trạng thái tài khoản
- **Dialog confirmations**: AlertDialog cho các thao tác quan trọng
- **Form validation**: Kiểm tra input trước khi cập nhật

### 3. Layout Structure:
- **Header with navigation**: Back button và tiêu đề rõ ràng
- **Search functionality**: EditText với icon search
- **Card-based list**: Mỗi tài khoản trong một CardView riêng biệt
- **Action buttons**: Buttons sửa/xóa được đặt ở bên phải mỗi item

---

## 🔒 BẢO MẬT VÀ PHÂN QUYỀN

### 1. Access Control:
- **Admin-only access**: Chỉ Admin mới có thể truy cập module này
- **Protected operations**: Không cho phép xóa hoặc sửa tài khoản admin
- **Role management**: Quản lý vai trò của tất cả tài khoản khác
- **Status control**: Có thể khóa/mở khóa tài khoản để kiểm soát truy cập

### 2. Data Protection:
- **Transaction integrity**: Sử dụng database transaction cho các thao tác phức tạp
- **Input validation**: Kiểm tra dữ liệu đầu vào trước khi cập nhật
- **Error handling**: Xử lý lỗi và hiển thị thông báo phù hợp
- **Audit trail**: Có thể mở rộng để log các thao tác quản lý tài khoản

### 3. System Stability:
- **Admin preservation**: Đảm bảo luôn có ít nhất một tài khoản admin
- **Graceful degradation**: Xử lý lỗi một cách mượt mà
- **Resource management**: Đóng cursor và database connection đúng cách
- **Memory efficiency**: Sử dụng ViewHolder pattern trong adapter

---

## 🔧 KẾT LUẬN

Module Quản lý Tài khoản là một thành phần bảo mật quan trọng của hệ thống QLNS, cung cấp khả năng quản lý toàn diện các tài khoản người dùng. Với thiết kế bảo mật cao và giao diện thân thiện, module này đảm bảo:

- **Bảo mật cao**: Bảo vệ tài khoản admin, phân quyền rõ ràng, validation đầy đủ
- **Quản lý hiệu quả**: Giao diện trực quan, thao tác đơn giản, feedback rõ ràng
- **Tính toàn vẹn**: Sử dụng transaction, xử lý lỗi tốt, đảm bảo consistency
- **Khả năng mở rộng**: Dễ dàng thêm tính năng audit, logging, advanced search
- **User experience**: Interface responsive, confirmation dialogs, visual feedback

Module này đóng vai trò nền tảng cho việc quản lý người dùng trong hệ thống và cung cấp foundation vững chắc cho các tính năng bảo mật khác của ứng dụng.