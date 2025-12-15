# Fixing Keychron HID Device Connection Issue on Linux

If your **Keychron keyboard** show **HID device connected** when you try to connect the keyboard with the **Keychron Launcher**, follow the steps below to fix it.

---

## Automatically try to create the exception
This handles steps 1-5 from below automatically, prompting you if multiple devices are detected.
Copy this into a new file and make that executable (`chmod +x filename`)
This will prompt you for sudo.

```bash
#!/bin/bash

# Configuration
VENDOR_ID="3434" # Keychron Vendor ID
RULE_FILE="/etc/udev/rules.d/99-keychron.rules"

# Detect the actual username (even if script is run with sudo)
TARGET_USER=${SUDO_USER:-$USER}

echo "--- Keychron HID Connection Fixer ---"
echo "Target User: $TARGET_USER"

# 1. Check dependencies
if ! command -v lsusb &> /dev/null; then
    echo "Error: 'lsusb' is not installed. Please install 'usbutils' (e.g., sudo apt install usbutils)."
    exit 1
fi

# 2. Find Keychron devices
echo "Scanning for Keychron devices (Vendor ID: $VENDOR_ID)..."

# Capture all matching lines into an array
mapfile -t DEVICE_LIST < <(lsusb | grep "ID ${VENDOR_ID}:")

DEVICE_COUNT=${#DEVICE_LIST[@]}

if [ "$DEVICE_COUNT" -eq 0 ]; then
    echo "❌ Error: No Keychron device found connected via USB."
    echo "Please connect the keyboard via cable and try again."
    exit 1
fi

SELECTED_LINE=""

# 3. Handle Single vs Multiple Devices
if [ "$DEVICE_COUNT" -eq 1 ]; then
    # Only one device found
    SELECTED_LINE="${DEVICE_LIST[0]}"
    echo "✅ Found 1 device."
else
    # Multiple devices found - Prompt user
    echo "⚠️  Found $DEVICE_COUNT Keychron devices. Please select the one to fix:"
    echo "---------------------------------------------------"
    
    # Print the list with numbers
    for i in "${!DEVICE_LIST[@]}"; do
        echo "[$((i+1))] ${DEVICE_LIST[$i]}"
    done
    
    echo "---------------------------------------------------"
    
    # Loop until valid input
    while true; do
        read -p "Enter number (1-$DEVICE_COUNT): " SELECTION
        # Check if input is a number and within range
        if [[ "$SELECTION" =~ ^[0-9]+$ ]] && [ "$SELECTION" -ge 1 ] && [ "$SELECTION" -le "$DEVICE_COUNT" ]; then
            INDEX=$((SELECTION-1))
            SELECTED_LINE="${DEVICE_LIST[$INDEX]}"
            break
        else
            echo "Invalid selection. Please try again."
        fi
    done
fi

echo "Selected Device: $SELECTED_LINE"

# 4. Extract Product ID
# Logic: Look for "ID 3434:", take everything after it, then take the first word/token before any space
PRODUCT_ID=$(echo "$SELECTED_LINE" | sed -n "s/.*ID ${VENDOR_ID}:\([[:alnum:]]*\).*/\1/p")

if [ -z "$PRODUCT_ID" ]; then
    echo "❌ Error: Could not parse Product ID from the selection."
    exit 1
fi

echo "Detected Product ID: $PRODUCT_ID"

# 5. Add User to Input Group
echo "Adding user '$TARGET_USER' to the 'input' group..."
sudo usermod -aG input "$TARGET_USER"

# 6. Create and Write udev Rule
echo "Creating udev rule at $RULE_FILE..."

# Construct the rule string
UDEV_RULE="KERNEL==\"hidraw*\", SUBSYSTEM==\"hidraw\", ATTRS{idVendor}==\"${VENDOR_ID}\", ATTRS{idProduct}==\"${PRODUCT_ID}\", MODE=\"0660\", GROUP=\"${TARGET_USER}\", TAG+=\"uaccess\", TAG+=\"udev-acl\""

# Write to file using sudo
echo "$UDEV_RULE" | sudo tee "$RULE_FILE" > /dev/null

echo "✅ Rule written:"
echo "$UDEV_RULE"

# 7. Reload udev Rules
echo "Reloading udev rules..."
sudo udevadm control --reload-rules
sudo udevadm trigger

echo "-------------------------------------"
echo "🎉 Success! The fix has been applied for Product ID: $PRODUCT_ID"
echo "You may need to unplug and replug your keyboard for the changes to take effect."
```

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
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="3434", ATTRS{idProduct}=="d030", MODE="0666", TAG+="uaccess", TAG+="udev-acl"
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
