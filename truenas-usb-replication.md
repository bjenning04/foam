---
tags: truenas
---
# TrueNAS USB Replication

## 1. Create the Transport Pool

First, format the external USB drive so ZFS can write to it.

- Plug the USB drive(s).
- Go to the **Storage** tab in the TrueNAS GUI and click **Create Pool**.
- Name it something obvious, like `usb-transport`.
- Check the box next to your external drive. (If using multiple drives, select all of them and ensure the VDEV layout is set to **Stripe**).
- Click **Create**.

