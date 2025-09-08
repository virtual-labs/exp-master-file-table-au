### Theory

The Master File Table (MFT) is analogous to a library directory catalog. It contains an entry for every file or directory inside an NTFS volume, including the MFT itself. Every update that occurs on a file within the file system is reflected in the MFT.  
If an adversary overwrites the contents of a target file on the hard disk, an examiner can still prove the existence of the file by referring to the MFT. With the help of MFT attributes, we can also detect suspicious files using timestamp anomalies.  
Several MFT attributes (e.g., ADS, EA) have been exploited by malware (such as the ZeroAccess trojan and Valak) to hide payloads.  
This experiment aims to provide an understanding of the Master File Table, how to recover the content of deleted resident files, and how to handle Alternate Data Streams.

#### 1.1 Master File Table

All computers have a hard disk, which is divided into partitions or volumes. Each volume contains a Master File Table (MFT).  
The MFT stores metadata for every file on the volume. Each file or directory is represented by a record in the MFT, commonly referred to as an MFT record.  
Think of each record as a row in a table, where each column is an attribute storing metadata such as timestamps, file name, and data location.

#### 1.2 MFT Record Structure

The standard size for an MFT record is 1024 bytes (1 KB), a value that has been historically preserved across various OS versions and NTFS implementations with few exceptions.  
In this experiment, we assume that each MFT record is 1 KB in size.

Each record is subdivided into three sections: **Header**, **Attributes**, and **Record Slack**.  
The header contains structural metadata, attributes store detailed metadata and content, and the remaining unused space (if any) is known as record slack.

Fields inside the header and attributes are valuable for tasks such as timeline analysis, file recovery, and file system debugging.  
Timeline analysis, for example, helps identify timestamp anomalies which can expose malicious activities.

#### 1.2.1 $DATA: Resident & Non-Resident Flags

When a file is created, and its total size (metadata + content) is less than or equal to 1 KB, the file's content is stored directly within the MFT record itself—inside the **$DATA** attribute. Such files are called **Resident Files**.

If the content exceeds 1 KB, it cannot fit within the MFT record. These are called **Non-resident Files**, and the **$DATA** attribute stores pointers to external disk locations where the actual file content is saved.

#### 1.2.2 Alternate Data Streams (ADS)

Windows NTFS supports **Alternate Data Streams (ADS)**, which allow additional metadata to be attached to files.  
For instance, a file downloaded from the internet may include an ADS like *filename:Zone.Identifier*, which stores information such as the source URL and download zone.  
This helps the operating system determine whether the file is potentially unsafe.

In an MFT record, an ADS appears as an additional **$DATA** stream after the main data stream.  
While ADS is useful for storing metadata, it can be misused by malware to hide malicious payloads like shellcode.  
If the ADS content is smaller than 1 KB, it may be stored entirely within the MFT record itself, making it difficult to detect.
