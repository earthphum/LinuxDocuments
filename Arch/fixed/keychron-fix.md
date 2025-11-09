# Fixing Keychron HID Device Connection Issue on Linux

If your **Keychron keyboard** show **HID device connected** with the **Keychron Launcher**, follow the steps below to fix it.

---

## 1. Add User to Input Group

Run this command to allow your user access to input devices:
```bash
sudo usermod -aG input $USER
```

---

## 2. Create a New udev Rule

Open (or create) a new file for Keychron:
```bash
sudo vim /etc/udev/rules.d/99-keychron.rules
```

---

## 3. Add the udev Rule

Paste the following rule into the file (you will modify `idProduct` later):
```bash
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="3434", ATTRS{idProduct}=="d030", MODE="0660", GROUP="myusername", TAG+="uaccess", TAG+="udev-acl"
```

> 💡 Replace `myusername` with your actual Linux username.

---

## 4. Find Your idProduct

To identify your device’s ID, run:
```bash
lsusb | grep Keychron
```

Example output:
```
Bus 001 Device 003: ID 3434:d030 Keychron Keychron Link
```

In this example:
- `idVendor` = **3434**
- `idProduct` = **d030**

---

## 5. Reload udev Rules

After saving the file, reload and trigger the new udev rules:
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## 6. If It Still Doesn’t Work

Try relaxing the file permissions temporarily:
```bash
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="3434", ATTRS{idProduct}=="d030", MODE="0666", GROUP="users", TAG+="uaccess", TAG+="udev-acl"
```

---

## 🧠 Explanation

- **`idVendor`** and **`idProduct`** identify your specific USB device.
- **`hidraw`** refers to raw HID devices (like keyboards, mice, or dongles).
- **`GROUP`** must match your username or a group you belong to (for example, `users`).
- **`MODE="0660"`** allows read/write access to the owner and group.
- **`MODE="0666"`** allows read/write access to everyone (less secure, for testing only).
- **`udevadm control --reload-rules`** reloads all udev rules.
- **`udevadm trigger`** applies them without rebooting.

---

## 🧩 Common Mistakes

- ❌ Forgetting to reload udev rules after editing.
- ❌ Using a wrong `idProduct` (each model differs).

---

## ✅ Summary

After applying and reloading the rules, reconnect your **Keychron keyboard**.
The **Keychron Launcher** should now detect it properly, allowing full functionality.

---
