---
title: "The SSD Squeeze: Tools to Beat NVMe Shortages and Price Hikes"
date: 2026-08-15T09:57:05+08:00
draft: false
tags:
  - storage
  - ssd
  - nvme
  - disk-cleanup
  - windirstat
  - czkawka
---

[Micron reported in its June 2026 SEC filing](https://www.sec.gov/Archives/edgar/data/723125/000072312526000015/mu-20260528.htm) that memory and storage demand was outpacing industry supply and that its NAND average selling prices had risen by the mid-80% range quarter over quarter. Before buying another drive, it is worth seeing how much space you can recover from the SSD you already own.

Use [WinDirStat](https://windirstat.net/) or [Czkawka](https://github.com/qarmin/czkawka) to identify and remove unnecessary files, freeing up space on your SSD.

## Find large and unnecessary files

Disk cleanup tools like WinDirStat and Czkawka can help you visualize disk usage and identify large or unnecessary files that can be deleted to free up space. This is especially useful for NVMe SSDs, which can be expensive and in short supply. By regularly cleaning up your disk, you can maintain optimal performance and avoid running out of storage. I use them both regularly when I see my SSD filling up, and they have helped me reclaim a significant amount of space.

## Reclaim space used by games

For games, consider using [Steam's storage manager](https://help.steampowered.com/en/faqs/view/4BD4-4528-6B2E-8327) to uninstall or move games and reclaim space.

[Game Compressor](https://store.steampowered.com/app/4339880/Game_Compressor/) is another tool that can help compress game files to save space on your SSD, though it is not free. There is a free tool, [CompactGUI](https://github.com/IridiumIO/CompactGUI), that uses Windows' built-in compression feature to reduce the size of files and folders, which can be useful for freeing up space on your SSD.

> I have not tested Game Compressor or CompactGUI, so please use them at your own risk. Always back up important data before using any disk cleanup or compression tools.

## My starting point

WinDirStat is my starting point. I learned of it over a decade ago because I was using KDirStat on Linux and was looking for something similar for Windows.

## Linux: trim unused blocks

On Linux, you can also run `fstrim -av` to trim the SSD. This command tells the SSD which blocks of data are no longer in use and can be wiped internally, which helps maintain performance over time.

## Space used by cache, code environments, and virtual machines

- Local caches for package managers (e.g., npm, pip, apt) can take up a lot of space. Consider clearing them periodically.
- Development environments (e.g., Docker, virtualenvs) can also consume significant disk space. Remove unused containers, images, and virtual environments to free up space.

You can find most of what you need to clean or compress using WinDirStat or KDirStat.

## Order of things

1. Start with WinDirStat or KDirStat to identify large files and directories.
2. Use Steam's storage manager to uninstall or move games. Clean up unused programs and development environments, and remove duplicate files with Czkawka.
3. Use CompactGUI or any compression tool to compress files and folders if needed.
4. Stop when the time spent cleaning up is no longer worth the space reclaimed.
5. (Optional) Use a cheap external drive to archive files that you don't need immediate access to but want to keep for future reference. I often move them to OneDrive or Google Drive for long-term storage so they are still accessible but not taking up space on my SSD.
