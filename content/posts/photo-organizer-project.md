+++
date = '2025-12-01T14:23:59-06:00'
draft = false
title = 'Photo Organizer and Deduplicator'
tags = ['synology', 'dsm', 'photo-management', 'automation', 'python', 'package']
image = '/images/icon_256.png'
+++

# DSM 7 Package
<figure style="text-align: center; margin: 1em 0;">
  <img src="/images/icon_256.png" alt="Photo Organizer Icon" style="width: 256px; height: 256px; max-width: 256px;" />
</figure>

## Table of Contents

1. [Overview](#overview)
2. [Purpose](#purpose)
3. [Key Features](#key-features)
4. [Package Details](#package-details)
5. [GitHub Repository](#github-repository)
6. [Download](#download)
7. [How It Works](#how-it-works)
8. [File Processing Logic](#file-processing-logic)
9. [Duplicate Handling](#duplicate-handling)
10. [Statistics Tracking](#statistics-tracking)
11. [Logging](#logging)
12. [Synology Integration](#synology-integration)
13. [License](#license)
14. [Support](#support)

## Overview

Photo Organizer and Deduplicator is a Synology NAS package that automatically organizes and manages your photo collection. This intelligent application continuously monitors a designated folder and automatically sorts photos by their capture date, detects duplicates, and organizes your entire photo library with minimal user intervention.

## Purpose

The Photo Organizer script is designed to solve common photo management challenges:

- **Automatic Organization**: Automatically sorts photos into year-based folders based on when they were taken
- **Duplicate Detection**: Identifies and handles both exact duplicates and visually similar images
- **Continuous Monitoring**: Watches a source folder and processes new photos as they arrive
- **Smart File Naming**: Renames files to a consistent `yyyymmdd_hhmmss.*` format for easy organization
- **EXIF Metadata Support**: Uses photo metadata when available, with intelligent fallbacks

## Key Features

### Automatic Folder Monitoring
The application continuously watches a user-defined source directory (`Photos/PhotosToProcess` by default) and automatically processes new files as they are added.

### Intelligent Date Detection
- **Primary Method**: Extracts date from EXIF metadata (`DateTimeOriginal`) when available
- **Fallback Method**: Uses file creation or modification date if EXIF data is missing
- **Smart Organization**: Files are organized into year-based subfolders (e.g., `Photos/2024/`)

### Duplicate Detection
- **Exact Duplicates**: Uses MD5 hash comparison to identify identical files
- **Similar Images**: Employs perceptual hashing to detect visually similar images
- **Quality Comparison**: When duplicates are found, compares image quality (resolution, file size, color depth) and keeps the best version
- **Smart Handling**: 
  - Exact duplicates are moved to a `Duplicates` folder with unique naming
  - Similar images are saved with `_A`, `_B`, etc. suffixes in the main Photos folder

### File Renaming
Files with date information are automatically renamed to `yyyymmdd_hhmmss.*` format, making it easy to find and organize photos chronologically.

## Package Details

### Package Information
- **Package Name**: PhotoOrganizer
- **Version**: 1.0.0-0009
- **Display Name**: Photo Organizer and Deduplicator
- **Architecture**: noarch (works on all Synology NAS models)
- **Minimum DSM Version**: 7.0-40000
- **Maintainer**: MORCE.codes
- **Type**: Third-party package

### Technical Requirements
- **Python**: 3.7 or higher
- **Dependencies**:
  - [Pillow](https://python-pillow.org/) - For reading EXIF data and image processing
  - [watchdog](https://python-watchdog.readthedocs.io/) - For folder monitoring
  - [imagehash](https://github.com/JohannesBuchner/imagehash) - For perceptual hashing (similar image detection)
  - hashlib - Built-in Python library for MD5 hash calculation

### Installation
1. Download the `.spk` file (see download link below)
2. Open Synology Package Center
3. Click "Manual Install" and select the downloaded `.spk` file
4. Follow the installation wizard
5. Configure the source and destination directories during setup

### Usage
Once installed, the service runs automatically in the background:
1. Place photos in the configured source directory (default: `Photos/PhotosToProcess`)
2. The service automatically processes existing photos
3. New photos added to the folder are automatically organized
4. Photos are sorted into year-based folders with consistent naming
5. Duplicates are detected and handled according to the configured settings

## GitHub Repository

[View on GitHub](https://github.com/52454D434F/DSM-Projects)

The project source code, build scripts, and documentation are available on GitHub. Contributions, bug reports, and feature requests are welcome!

## Download

### Latest Version: 1.0.0-0009

[Download PhotoOrganizer-1.0.0-0009.spk](/downloads/PhotoOrganizer-1.0.0-0009.spk)

## How It Works

1. **Monitoring**: The service uses a file system watcher to detect new files in the source directory
2. **Processing**: When a new photo is detected:
   - EXIF metadata is extracted to determine the capture date
   - If EXIF data is unavailable, file timestamps are used
   - The photo is checked against existing files for duplicates
   - Image quality is compared if duplicates are found
3. **Organization**: Photos are moved to appropriate year-based folders and renamed consistently
4. **Duplicate Handling**: Exact duplicates are moved to a dedicated folder, while similar images are preserved with unique suffixes

## File Processing Logic

The Photo Organizer uses a sophisticated multi-step process to handle each file that appears in the source directory.

### Processing Workflow

1. **File Detection**: The system waits 0.5 seconds after detecting a new file to ensure it's fully written (important for large files being copied)

2. **File Type Detection**: Determines if the file is an image or video based on file extension

3. **Date Extraction** (in priority order):
   - **Images**: Attempts to read EXIF `DateTimeOriginal` metadata
   - **Videos**: Attempts to read creation date from video metadata (MP4/MOV using mutagen library)
   - **Fallback**: Uses the older of file creation or modification timestamp

4. **Destination Path Generation**:
   - If date found: Creates path `YYYY/MM_Mmm/` (e.g., `2024/08_Aug/`)
   - Filename format: `yyyymmdd_hhmmss.ext` (without subseconds)
   - If no date found: Moves to `NoDateFound/` folder with original filename

5. **Duplicate Check**: Before moving, checks if a file with the same name already exists at the destination

6. **File Movement**: Moves the file to the appropriate destination folder

### Date Extraction Methods

#### EXIF Metadata (Images)
- Reads `DateTimeOriginal` tag from image EXIF data
- Supports both modern `getexif()` and legacy `_getexif()` Pillow methods
- Format: `YYYY:MM:DD HH:MM:SS` (no milliseconds in EXIF standard)
- Preserves sub-second precision from file timestamps when EXIF is unavailable

#### Video Metadata
- For MP4/M4V files: Reads creation date from MP4 metadata tags (`©day` or `\xa9day`)
- For MOV files: Reads creation date from QuickTime metadata
- Supports ISO 8601 format: `YYYY-MM-DD` or `YYYY-MM-DDTHH:MM:SS`
- Falls back to file timestamps if metadata unavailable

#### File Timestamps
- Uses the older of creation time or modification time
- Preserves sub-second precision (microseconds) when available
- Works for both images and videos when metadata is missing
- Provides reliable fallback for files without embedded metadata

## Duplicate Handling

The application implements intelligent duplicate detection and handling to prevent storage waste and maintain a clean photo library.

### Duplicate Detection Process

1. **Fast Size Check**: First compares file sizes - if different, files are not duplicates (performance optimization)

2. **MD5 Hash Comparison**: If sizes match, calculates MD5 hash of both files
   - MD5 hash provides cryptographic-level duplicate detection
   - Only performed when file sizes match (optimization)
   - Ensures 100% accuracy in identifying identical files

3. **Content Comparison**: If hashes match, files are identical duplicates

### Duplicate Handling Strategies

#### Exact Duplicates (Same MD5 Hash)

**If `DELETE_DUPLICATES = true`** (default):
- The duplicate file is deleted immediately
- Statistics track bytes deleted
- Synology indexer is updated to remove the file from index
- No storage space wasted on duplicates

**If `DELETE_DUPLICATES = false`**:
- Duplicate is moved to `Duplicates/` folder
- Filename format: `yyyymmdd_hhmmss.ssss.ext` (with subseconds if available)
- If subseconds not available: `yyyymmdd_hhmmss.0001.ext`, `yyyymmdd_hhmmss.0002.ext`, etc.
- Statistics track bytes moved
- Preserves all files for manual review

#### Different Content (Same Filename, Different Hash)

When files have the same destination filename but different content:

1. **Modification Time Comparison**: Compares file modification times
2. **Keep Older File**: The older (earlier modified) file stays in the destination folder
3. **Move Newer File**: The newer file is moved to `Duplicates/` folder with subseconds format
4. **Duplicate Check in Duplicates Folder**: Before moving, checks if an identical file already exists in Duplicates folder
   - If found: Deletes the current duplicate instead of creating another
   - If not found: Moves with unique subseconds/sequential naming

### Filename Formats

#### Destination Folder
- Format: `yyyymmdd_hhmmss.ext`
- Example: `20250214_134757.jpg`
- No subseconds included (clean, consistent naming)
- Used for existence checks and organization

#### Duplicates Folder
- **With subseconds**: `yyyymmdd_hhmmss.ssss.ext` (e.g., `20250214_134757.3519.jpg`)
  - Uses actual subseconds from file timestamp when available
  - Provides unique identification based on capture time precision
- **Without subseconds**: `yyyymmdd_hhmmss.0001.ext`, `yyyymmdd_hhmmss.0002.ext`, etc.
  - Sequential numbering ensures unique filenames
  - Starts from 0001 and increments as needed

### Duplicate Detection in Duplicates Folder

Before moving a file to the Duplicates folder, the system:
1. Calculates MD5 hash of the file to be moved
2. Scans all files in the Duplicates folder
3. Compares hashes to find exact matches
4. If duplicate found: Deletes the file instead of moving it
5. Prevents accumulation of multiple identical duplicates
6. Ensures Duplicates folder only contains unique files

## Statistics Tracking

The application maintains comprehensive statistics about file operations to help users understand the service's activity.

### Tracked Metrics

- **Bytes Moved**: Total size of files successfully organized and moved to destination folders
- **Bytes Deleted**: Total size of duplicate files removed from the system

### Statistics Logging

Statistics are automatically logged in the system log when:

1. **Service Stops**: When the service is stopped (graceful shutdown or signal)
   - Final statistics are always logged before shutdown
   - Prevents loss of statistics data

2. **Idle Timeout**: After 1 minute (60 seconds) of no file activity
   - Statistics are logged before resetting counters
   - Counters reset to zero after logging
   - Prevents statistics from growing indefinitely
   - Provides periodic activity summaries

3. **Manual Reset**: Statistics can be reset programmatically

### Statistics Format

Logged entries appear in `system_log.csv` with format:
```
Info, System, 2025/01/15 14:30:22, PhotoOrganizer, Statistics (service stopped): 2.45 GB moved, 150.30 MB deleted
```

The format includes:
- Log level: `Info`
- Log type: `System`
- Timestamp: `YYYY/MM/DD HH:MM:SS`
- User: System user running the service
- Event: Description with formatted byte counts

### Statistics Reset Behavior

- Statistics reset automatically after 1 minute of idle time
- Reset prevents counters from accumulating over long periods
- Each reset logs the accumulated statistics before clearing
- Service stop always logs final statistics (even if recently reset)
- Prevents duplicate logging on exit through flag management

### Performance Considerations

- Statistics are tracked in memory (global variables)
- Minimal performance impact on file operations
- Logging is asynchronous and doesn't block file processing
- Byte counters use efficient integer arithmetic

## Logging

Photo Organizer includes comprehensive logging capabilities to track all operations, system events, and file activities. The logging system uses multiple methods to ensure reliability and compatibility with Synology DSM.

### Log Files

The application maintains two primary CSV log files in the destination directory:

1. **`photo_organizer_log.csv`** - Detailed file operation log
   - Tracks all file operations (moves, deletes, duplicates)
   - Records file detection events
   - Includes file size, timestamps, IP addresses, and user information
   - Format: `Log, Time, IP address, User, Event, File/Folder, File size, File name, Additional Info`

2. **`system_log.csv`** - System events log
   - Records service start/stop events
   - Logs dependency checks and system status
   - Tracks statistics (bytes moved/deleted)
   - Format: `Level, Log, Time, User, Event`
   - Log levels: Info, Warning, Error

### Logging Methods

The application uses multiple logging methods for maximum reliability:

1. **System Logger** (`logger` command)
   - Integrates with Synology's system logging infrastructure
   - Messages are accessible through DSM's Log Center
   - Tag: `PhotoOrganizer`
   - Priority: `user.info`

2. **Synology Logging** (`synologset1`)
   - Synology-specific logging method
   - Provides additional integration with DSM logging system
   - Used as a fallback if standard logger is unavailable

3. **Package Log Files**
   - CSV files stored in the destination directory
   - Always accessible to the package user
   - Provides detailed operation history
   - Can be viewed with any CSV reader or text editor

### Logged Events

The following events are logged:

#### File Operations
- **File Detected**: New file detected in source directory
- **File Moved**: File successfully moved to destination
- **Duplicate Deleted**: Exact duplicate removed
- **Moved to Duplicates**: Duplicate file moved to Duplicates folder
- **Error**: Any error during file processing

#### System Events
- **Service Started**: Application initialization
- **Service Stopped**: Application shutdown
- **Dependency Check**: Verification of required Python packages
- **Statistics**: Summary of bytes moved and deleted (logged on service stop or idle timeout)

### Log File Locations

On Synology NAS, log files are stored in:
- Default location: `/volume1/photo/photo_organizer_log.csv` and `/volume1/photo/system_log.csv`
- Alternative: Configured destination directory

### Viewing Logs

#### Via File Station or SSH
```bash
# View recent file operations
tail -n 50 /volume1/photo/photo_organizer_log.csv

# View system events
tail -n 50 /volume1/photo/system_log.csv

# View all logs
cat /volume1/photo/photo_organizer_log.csv
```

#### Via DSM Log Center
System-level events logged via `logger` and `synologset1` are accessible through:
- **DSM Control Panel** → **Log Center** → **System Logs**
- Filter by tag: `PhotoOrganizer`

### Log Statistics

The application tracks and logs statistics including:
- Total bytes moved (files organized)
- Total bytes deleted (duplicates removed)
- Statistics are logged when:
  - Service stops
  - Service is idle for extended periods
  - Manual statistics reset occurs

### Log Format Details

#### File Operation Log Format
Each entry in `photo_organizer_log.csv` contains:
- **Log**: Always "SMB" (for compatibility with Synology file transfer logs)
- **Time**: Timestamp in `YYYY-MM-DD HH:MM:SS` format
- **IP address**: Local IP address of the NAS
- **User**: System user running the service
- **Event**: Type of operation (File moved, Duplicate Deleted, etc.)
- **File/Folder**: Full system path to the file
- **File size**: Human-readable file size (e.g., "2.45 MB")
- **File name**: Base filename
- **Additional Info**: Contextual information about the operation

#### System Log Format
Each entry in `system_log.csv` contains:
- **Level**: Log level (Info, Warning, Error)
- **Log**: Always "System"
- **Time**: Timestamp in `YYYY/MM/DD HH:MM:SS` format
- **User**: System user
- **Event**: Description of the system event

### Troubleshooting with Logs

Logs are essential for troubleshooting issues:
1. Check `system_log.csv` for dependency errors or service start/stop issues
2. Review `photo_organizer_log.csv` for file processing errors
3. Look for "Error" entries to identify specific problems
4. Verify file paths and permissions from logged file operations

For detailed debugging instructions, see the [DEBUGGING.md](source/PhotoOrganizer/DEBUGGING.md) file in the project repository.

## Synology Integration

The Photo Organizer is specifically designed to integrate seamlessly with Synology DiskStation Manager (DSM), providing native NAS functionality.

### Synology Detection

The application automatically detects if it's running on a Synology NAS by checking:
- Presence of `/etc/synoinfo.conf`
- DSM version file at `/etc.defaults/VERSION`
- Standard Synology volume path `/volume1`

### Photo Indexer Integration

The application automatically updates Synology's built-in photo indexer (`synoindex`) after file operations:

#### Indexer Update Process

1. **File Deletion**: Removes deleted files from the index
   - Command: `synoindex -D <file_path>`
   - Ensures deleted duplicates don't appear in Photo Station
   - Keeps index synchronized with actual file system

2. **File Movement**: Updates index when files are moved
   - Removes old path: `synoindex -D <old_path>`
   - Adds new path: `synoindex -A <new_path>`
   - Keeps Photo Station synchronized with file organization
   - Ensures photos appear in correct location in Photo Station

3. **Error Handling**: Silently handles cases where `synoindex` is unavailable
   - No errors thrown if indexer command fails
   - Service continues operating normally
   - Timeout protection (5 seconds) prevents hanging
   - Graceful degradation on non-Synology systems

### Path Conversion

The application converts user-visible paths to system paths for logging:
- User path: `/volume1/photo/Photo Organizer/file.jpg`
- System path: `/var/services/photo/Photo Organizer/file.jpg`
- Enables compatibility with Synology's Log Center
- Maintains consistency with DSM's internal path structure

### Synology-Specific Features

- **Package Installation**: Designed as a Synology `.spk` package
  - Installs through Package Center
  - Follows Synology package structure and conventions
  - Includes proper service management scripts

- **Service Management**: Integrates with DSM's package service system
  - Can be started/stopped via Package Center
  - Respects Synology's service lifecycle
  - Proper signal handling for graceful shutdown

- **Log Integration**: Logs appear in DSM Log Center
  - Uses Synology's logging infrastructure
  - Accessible through DSM web interface
  - Filterable by tag and priority

- **User Permissions**: Respects Synology user and permission system
  - Runs with appropriate package user permissions
  - Respects folder access controls
  - Integrates with DSM security model

- **Volume Support**: Works with multiple Synology volumes
  - Supports `/volume1`, `/volume2`, etc.
  - Handles volume-specific paths correctly
  - Adapts to different storage configurations

### Configuration on Synology

Configuration files are stored in Synology package directories:
- Primary: `/var/packages/PhotoOrganizer/config.ini`
- Alternative: `/var/packages/PhotoOrganizer/var/config.ini`
- Fallback: `/volume1/@appstore/PhotoOrganizer/var/config.ini`

The application automatically searches these locations in order and uses the first available configuration file.

### Benefits of Synology Integration

- **Seamless Operation**: Works like native Synology applications
  - No special configuration required
  - Follows DSM conventions and standards
  - Integrates with existing workflows

- **Photo Station Compatibility**: Organized photos appear correctly in Photo Station
  - Indexer updates ensure proper recognition
  - Maintains metadata and organization
  - Supports Photo Station's browsing features

- **System Logging**: Integrates with DSM's centralized logging
  - All events visible in Log Center
  - Consistent with other Synology services
  - Easy troubleshooting and monitoring

- **Service Management**: Can be started/stopped via Package Center
  - Standard Synology package interface
  - Status monitoring through Package Center
  - Automatic startup on boot (if configured)

- **Resource Management**: Respects Synology's resource limits and scheduling
  - Works within DSM's resource constraints
  - Compatible with other services
  - Efficient resource utilization

## License

Copyright (c) 2025 MORCE.codes

This package provides automatic photo organization functionality for Synology NAS. The Photo Organizer and Deduplicator application is provided as-is for organizing photos based on EXIF metadata.

## Support

For issues, questions, or contributions, please visit the [GitHub repository](https://github.com/52454D434F/DSM-Projects)

---

**Note**: This package is designed specifically for Synology DiskStation Manager (DSM) and requires a Synology NAS running DSM 7.0 or later.
