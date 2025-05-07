---
layout: default
title: Step 1 - Downloading Heads
nav_order: 2
permalink: /Downloading
parent: Installing and configuring
---

<!-- markdownlint-disable MD033 -->
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
<!-- markdownlint-enable MD033 -->

Downloading Heads ROMs
===

What board configuration should I choose?
---
Please refer to the [devices compatibility matrix]({{ site.baseurl }}/Prerequisites#supported-devices).

Downloading ROM files for initial flashing
---
If you are performing an initial external flash, follow these steps to download the required ROM files and verify their integrity:

1. **Access CircleCI Builds**:
   - Go to the [Heads GitHub repository](https://github.com/linuxboot/heads) and click on the latest commit's green checkmark.
   ![Github-green-circle-ci-checkmark](https://github.com/user-attachments/assets/ed928ced-a723-4d03-b24a-34961e5f2d90)   
   - This will display a list of CircleCI jobs. Locate the job corresponding to your board's name and click the "Details" link.
   ![CircleCI-list-of-boards-in-pipeline](https://github.com/user-attachments/assets/d09d1e9f-fac6-40ff-b980-8a9dd4b77ec3)

2. **Navigate to Artifacts**:
   - On the CircleCI job page, click the "Artifacts" tab to view the output files of the build.
   ![CircleCI-page-showing-artifact-tabs](https://github.com/user-attachments/assets/ba27c1ba-6e7b-4411-809e-b72912cc7549)

3. **Download Required Files**:
   - Download the following files by right-clicking their links and selecting "Save Link as...":
   ![CircleCI-artifacts-tabs-showing-files-including-hashes-txt](https://github.com/user-attachments/assets/ad30a6e5-d7a2-444a-88ba-1a420d1f0f9d)
     - `*.rom` files (e.g., `top.rom` and `bottom.rom` for two SPI chip boards).
     - `hashes.txt` file containing the SHA256 hashes of the ROM files.

4. **Verify ROM File Integrity**:
   - Use `sha256sum` or an equivalent tool to compute the hash of the downloaded ROM files.
   - Compare the computed hash with the corresponding entry in the `hashes.txt` file to ensure the files are valid.

5. **Prepare for Flashing**:
   - Save the verified ROM files and `hashes.txt` to the computer or USB drive that will be used for external flashing.

For more details on external flashing, refer to the [Upgrading documentation]({{ site.baseurl }}/Updating).

Internal upgrading from ZIP files
---
If you are upgrading an existing Heads installation, you can use the ZIP file provided in the CircleCI artifacts:

1. **Download the ZIP File**:
   - From the CircleCI "Artifacts" tab, download the ZIP file corresponding to your board.

2. **Transfer to Target System**:
   - Save the ZIP file to a USB drive and transfer it to the target system.

3. **Perform the Upgrade**:
   - Use the Heads GUI to upgrade the firmware by selecting the ZIP file. This method automatically verifies the ROM integrity and flashes the firmware.

**Note**: Internal upgrading via ZIP files is supported for firmware versions built after [this November 2023 commit](https://github.com/linuxboot/heads/commit/6873df60c1c965ac812a49d9d245f338d8a3b128).

For more details on internal upgrading, refer to the [Upgrading documentation]({{ site.baseurl }}/Updating).

Caution for Rolling Releases
---
Heads is a rolling release. If you do not have an external programmer, refer to the [Upgrading documentation]({{ site.baseurl }}/Updating#global-warning-no-hardware-programmer-be-cautious) for important precautions before upgrading. This includes:
- Waiting for community testing and reviewing reported issues to avoid potential regressions.
- Understanding the implications of `UNTESTED` and `UNMAINTAINED` labels in board names. For more details, refer to the [UNTESTED and UNMAINTAINED boards documentation](https://github.com/linuxboot/heads/blob/master/unmaintained_boards/README.md).

For more details on upgrading, refer to the [Upgrading documentation]({{ site.baseurl }}/Updating).
