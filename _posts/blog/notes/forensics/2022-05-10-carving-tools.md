---
title: Carving tools
layout: post
date: 2022-05-02 1:53:14
permalink: blog/:categories/:title
categories: notes forensics
---

## Autopsy

The most common tool used in forensics to extract files from images is [**Autopsy**](https://www.autopsy.com/download/). Download it, install it and make it ingest the file to find "hidden" files. Note that Autopsy is built to support disk images and other kind of images, but not simple files.

## Binwalk 

**Binwalk** is a tool for searching binary files like images and audio files for embedded files and data.
It can be installed with `apt` or `yay` however the [source](https://github.com/ReFirmLabs/binwalk) can be found on github.
**Useful commands**:

```bash
binwalk file #Displays the embedded data 
binwalk -e file #Displays and extracts 
binwalk --dd ".*" file #Displays and extracts all files
```

## Foremost

Another common tool to find hidden files is **foremost**. You can find the configuration file of foremost in `/etc/foremost.conf`. If you just want to search for some specific files uncomment them. If you don't uncomment anything foremost will search for it's default configured file types.

```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```

## **Scalpel**

**Scalpel** is another tool that can be use to find and extract **files embedded in a file**. In this case you will need to uncomment from the configuration file (_/etc/scalpel/scalpel.conf_) the file types you want it to extract.

```bash
sudo apt-get install scalpel
scalpel file.img -o output
```

## Bulk Extractor

This tool comes inside kali but you can find it here: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

This tool can scan an image and will **extract pcaps** inside it, **network information(URLs, domains, IPs, MACs, mails)** and more **files**. You only have to do:

```
bulk_extractor memory.img -o out_folder
```

Navigate through **all the information** that the tool has gathered (passwords?), **analyse** the **packets**, search for **weird domains** (domains related to **malware** or **non-existent**).

## PhotoRec

Designed to recover lost files from various digital camera memory, hard disk...
You can find it [here](https://www.cgsecurity.org/wiki/TestDisk_Download).

## binvis

Check the [web page tool](https://binvis.io/#/).
BinVis is a great **start-point to get familiar with an unknown target** in a black-boxing scenario.

---

# Complementary tools

You can use [**viu**](https://github.com/atanunq/viu) to see images form the terminal.\
You can use the linux command line tool **pdftotext** to transform a pdf into text and read it.

---

### Sources

1. [Hacktricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools)
