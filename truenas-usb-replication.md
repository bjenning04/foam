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

## 2. Take the Snapshot

- Go to the **System** tab in the TrueNAS UI and click **Shell**.
Drop into the TrueNAS Shell. Just like we did with your local apps, we need to freeze the dataset in time. Replace source-pool/data with the actual name of the dataset you are backing up.

Bash
sudo zfs snapshot -r source-pool/data@migration
3. Start the Transfer
Fire up a persistent session so you can walk away or close the browser while it copies terabytes of data to the USB drive.

Bash
tmux
Once inside tmux, run the pipe command to blast the snapshot over to your external drive:

Bash
sudo zfs send -R source-pool/data@migration | sudo zfs recv -F usb-transport/data
Note: You can safely detach by closing the browser tab. Just type tmux attach in a new shell later to check if the prompt has returned.

4. Safely Export (The Most Important Step)
Once the shell prompt returns and the transfer is complete, you must unmount the drive cleanly so you don't corrupt the ZFS headers before hitting the road.

Go back to the Storage dashboard.

Locate the usb-transport pool.

Click Export/Disconnect.

CRITICAL: Do not check the box that says "Destroy data on this pool." Just type the pool name to confirm and click Export.
