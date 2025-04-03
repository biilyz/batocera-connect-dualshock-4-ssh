
# Hướng dẫn Tạo và Chạy Script Kết Nối DualShock 4 qua Bluetooth

## Giới thiệu

Script này giúp bạn kết nối lại tay cầm **DualShock 4** với hệ thống của bạn thông qua Bluetooth. Nó sẽ tự động quét, ghép đôi và kết nối lại với tay cầm dựa trên địa chỉ MAC của thiết bị.

## Các bước thực hiện

### 1. **Tạo File Script**

Trước tiên, bạn cần tạo file **`connect.sh`** để chứa script kết nối.

- Mở terminal và tạo file mới bằng lệnh:

```bash
nano connect.sh
```

### 2. **Sao Chép và Dán Script**

Sao chép nội dung script dưới đây và dán vào file `connect.sh` bạn vừa tạo:

```bash
#!/bin/bash

MAC="1C:66:6D:D7:14:DD"  # Địa chỉ MAC của tay cầm DualShock 4

echo "🔄 Đang kết nối lại DualShock 4..."

# Xóa kết nối cũ để tránh lỗi
bluetoothctl remove $MAC
sleep 2

# Bật chế độ quét
bluetoothctl scan on &
sleep 10
kill -2 $!
echo "🕐 Đang quét... Vui lòng đợi một chút."

# Kiểm tra xem thiết bị đã được phát hiện chưa
DEVICE_FOUND=""
while [ -z "$DEVICE_FOUND" ]; do
    DEVICE_FOUND=$(bluetoothctl devices | grep -w $MAC)
    if [ -z "$DEVICE_FOUND" ]; then
        sleep 1
    fi
done

# Tắt chế độ quét sau khi tìm thấy thiết bị
bluetoothctl scan off
echo "🔵 Đã tắt chế độ quét."

# Ghép đôi và kết nối lại
# Ghép đôi với thiết bị
bluetoothctl pair $MAC

# Kiểm tra nếu lệnh ghép đôi thành công
PAIR_STATUS=$(bluetoothctl info $MAC | grep "Pairing successful")

if [ -n "$PAIR_STATUS" ]; then
    echo "✅ Đã ghép đôi thành công với thiết bị."
else
    echo "❌ Ghép đôi thất bại. Dừng script."
    exit 1  # Dừng script nếu ghép đôi thất bại
fi
bluetoothctl trust $MAC
bluetoothctl connect $MAC

# Kiểm tra kết nối sau khi ghép đôi
CONNECTED=$(bluetoothctl info $MAC | grep "Connected: yes")
if [ -n "$CONNECTED" ]; then
    echo "✅ Kết nối thành công với DualShock 4!"
else
    echo "❌ Không thể kết nối với DualShock 4."
fi

sleep 1
DISCOVERING=$(bluetoothctl show | grep "Discovering" | awk '{print $2}')

if [ "$DISCOVERING" = "no" ]; then
    echo "✅ Đã ngừng tìm kiếm Bluetooth, có thể sử dụng."
else
    echo "🔄 Vẫn đang tìm kiếm Bluetooth, vui lòng chạy lại."
fi

```

### 3. **Cấp Quyền Thực Thi cho Script**

Sau khi tạo xong file `connect.sh`, bạn cần cấp quyền thực thi cho nó để có thể chạy script.

Chạy lệnh sau trong terminal:

```bash
chmod +x connect.sh
```

Lệnh này sẽ cho phép bạn chạy file `connect.sh` như một script trên hệ thống của mình.

### 4. **Chạy Script**

Bây giờ, bạn đã sẵn sàng để chạy script kết nối.

Chạy lệnh sau để thực hiện script:

```bash
./connect.sh
```

Script sẽ thực hiện các bước sau:

- **Xóa kết nối cũ** (nếu có).
- **Bật chế độ quét Bluetooth** để tìm kiếm tay cầm DualShock 4.
- **Kiểm tra xem thiết bị có được phát hiện**.
- **Dừng quét** khi tìm thấy tay cầm.
- **Ghép đôi và kết nối lại** tay cầm với hệ thống.
- **Thông báo kết quả** kết nối thành công hay thất bại.

---

## Lưu ý

- Đảm bảo rằng Bluetooth đã được bật trên hệ thống của bạn.
- Địa chỉ MAC của tay cầm **DualShock 4** cần phải chính xác. Bạn có thể thay đổi địa chỉ MAC trong script cho phù hợp.
- Nếu gặp lỗi, hãy kiểm tra lại quyền truy cập Bluetooth trên hệ thống của bạn.

---

## Cảm ơn bạn đã sử dụng!

Nếu bạn gặp bất kỳ vấn đề nào hoặc có câu hỏi, vui lòng mở một **issue** trong repo này để chúng tôi có thể hỗ trợ.
