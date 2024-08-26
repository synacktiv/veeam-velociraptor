# Veeam Backup & Replication Velociraptor artifacts - PoC

This repository contains a collection of Velociraptor artifacts acting as a proof-of-concept for a remote Veeam Backup & Replication forensics pipeline:
- `artifacts/` contains the artifact definitions
- `tools/` contains a `qemu-img.zip` archive that can be used with `Windows.QEMU.ImgConvert`

More details can be found on the related blogpost: https://www.synacktiv.com/publications/using-veeam-metadata-for-efficient-extraction-of-backup-artefacts-23

## How to use

1. Import the [Artifact Exchange](https://docs.velociraptor.app/exchange/) into the Velociraptor server, by launching the `Server.Import.ArtifactExchange` server artifact with **default parameters**. This will make available two dependencies:
   - `Windows.Veeam.RestorePoints.MetadataFiles` and
   - `Windows.Veeam.RestorePoints.BackupFiles`.

2. Import the following pack by launching the `Server.Import.ArtifactExchange` artifact with the following value for the `ExchangeURL` parameter:
   - `https://github.com/synacktiv/veeam-velociraptor/releases/download/v0.1/veeam_velociraptor_v0.1.zip`

3. Modify `Exchange.Windows.Veeam.ProcessDiskImage` to choose which artifacts to run on each extracted disk image. The VQL queries should follow this format:
   ```
   SELECT HostName, VMName, CreationTimeUTC, BackupFilePath, 'FULL_ARTIFACT_NAME' AS ArtifactName, *
     FROM Artifact.FULL_ARTIFACT_NAME( ARTIFACT_PARAMETERS )
   ```

4. Launch either `Exchange.Windows.Veeam.RestorePoints.MetadataFiles` or `Exchange.Windows.Veeam.RestorePoints.BackupFiles` (much slower!) to list available Restore Points.

5. Using the `WHERE` clause inside a Notebook, filter on the results until you find a `WHERE` clause that returns only the Restore Points you wish to address in later steps.

6. Launch `Exchange.Windows.Veeam.ProcessBackups`, specifying the filter you want to apply (without the `WHERE` keyword) in its `Filter` parameter. Be careful about the path you choose for `TemporaryFolderPath` as this folder will contain hundreds of gigabytes (if not terabytes) of extracted backup data, during image processing. It is advised to use a dedicated drive to hold this data.

## Disclaimer

The contents of this repository are not production-ready and should only be considered as a proof-of-concept for a specific Velociraptor workflow. The following artifacts were not submitted to the Velociraptor Artifact Exchange, for this reason.

The artifacts in this repository will write gigabytes (potentially terabytes) of data (extracted from Veeam backups) to the folder specified in `TemporaryFolderPath` parameter of `Exchange.Windows.Veeam.ProcessBackups`. When processing is done, previously written files will be deleted, with minimal verification.

Given that these artifacts were tested on a very limited set of data, we advise to use this in a test environment only. Synacktiv will not be held liable for any loss using these artifacts.
