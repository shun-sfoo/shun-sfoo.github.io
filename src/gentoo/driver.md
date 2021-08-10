# 驱动相关

## 电源驱动

```bash
emerge --ask sys-power/acpid # Power Management
emerge sys-power/thermald # intel support acpi
rc-update add thermald default
```
