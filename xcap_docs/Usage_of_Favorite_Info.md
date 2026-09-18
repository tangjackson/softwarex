# Usage of Favorite Info

> Source: `Analyzer Help.chm / 47_FavoriteInfo.htm`
> Extracted with `extract_chmLib`, decoded as `ks_c_5601-1987` (CP949); HTML converted to Markdown. Text unmodified.

## Favorite Info

Favorite Info feature enables you to reload customized parameter window in the workspace. User-organized parameter window is to be saved as a Favorite, and appears in the parameter tree under Favorite Info category.

Favorite Info feature supports analysis types of Graph, Map, Table, CDF, and PDF.

Favorite configuration in an analysis type does not applied for the other analysis type. A Favorite configured for Graph anslysis does not applied for Map, Table, CDF, PDF analysis.

Favorite configuration is applied for the other Models.

A new favorite is added by using

- Create Favorite feature from mouse menu in analysis window

- Favorite Manager from Menu bar – Tools – Favorite Manager

Creating a new Favorite with Mouse Menu of Create Favorite

A new Favorite can be created by using mouse menu of each analysis

1. Click an analysis type tab; Graph, Map, Table, CDF, PDF.

2. Open multiple or overlaid parameters in one parameter window.

3. Customize the parameter window by using various analysis options such as filtering and etc.

4. Right-click on the customized analysis window, and select Create Favorite .

5. Favorite List Name window appears.

6. Enter a new Favorite name in the entry field. Click OK .

7. A new Favorite is listed under Favorite List in parameter tree.

8. Double-click the Favorite name or drag and drop it into the workspace to open.

Creating a new Favorite with Favorite Manager

A new Favorite can be created by using Favorite Manager from main Menu bar – Tools – Favorite Manager .

1. Open Favorite Manager from main Menu bar – Tools – Favorite Manager .

2. Click New icon for Favorite List at the upper right corner of Favorite Info window.

3. Enter a new Favorite name in the entry field.

4. Drag a parameter or parameter category from the parameter tree at the right upper corner of Favorite Info window and drop them into Parameter (Check-recursive) field.

5. Select an analysis type to apply the Favorite configuration from the drop-down button of Select an analysis type to apply Favorite List .

6. Click OK .

7. A new Favorite is listed under Favorite List in parameter tree.

8. Double-click the Favorite name or drag and drop it into the workspace to open.

Using Recursive Option

Recursive option enables you to open certain parameters (e.g. RSSI and BSIC) under different parameters categories (e.g. Scan Index#). If Recursive option is selected for certain parameters (e.g. RSSI and BSIC), all parameters under different parameter categories are configured as Favorite and able to be opened in the workspace.

1. In Favorite Manager (Favorite Info) window, select the checkbox for Recursive in Parameter (Check-recursive) list.

2. Click OK.

3. A new Favorite is listed under Favorite List in parameter tree.

4. Double-click the Favorite name or drag and drop it into the workspace to open.

5. All parameters under different parameter categories are opened in the workspace.

Exporting Favorite parameters

Export button at lower left corner of Favorite Manager helps you to export Favorite parameters. Parameter data configured as Favorite are to be exported in the designated directory.

Supporting export format are *.xls, *.csv, and *.txt.

1. Select a Favorite name from Favorite List at the upper left corner of Favorite Info window.

2. Click Expot button at the lower left corner of Favorite Info window.

3. Favorite export – Select Model window appears.

4. Configure export settings, and click OK .

5. Exported file is saved at the designated directory.

Deleting a Favorite from Favorite List tree

To delete Favorite from Favorite Info tree,

- Right-click on the Favorite name in parameter tree, and select Delete .

- Click to select a Favorite in Favorite List , and click Delete icon.

Usage of DOS Prompt File Export

Favorite List created in Favorite Manager can be exported via DOS prompt using DOS commands.

1. Save Favorite List to *.fav format in Favorite Manager .

2. Close XCAP Analyzer program. Place the *.drm and *.fav files in certain folder.

For example, place drm file in C:\Temp\log while placing *.fav file in C:\Temp\setting .

3. Execute Command prompt in XCAP program folder.

a. In window explorer, go to XCAP program folder. Right-click in the program folder while pressing Shift key on keyboard, and select Open Command Window Here .

b. Windows DOS prompt screen appears.

4. Use DOS commands.

DOS Commands for Exporting Favorite File

| No. | Command | Example | Description | Basic option |
|---|---|---|---|---|
| 1 | -EXPORT | ”XCAP-M.exe” –EXPORT “drm folder path” “csv file path to be exported” “fav file path” | Batch Export after Model Creation (Favorite) | Model Creation - Separate, output format -csv |
| 2 | -EXPORT | ”XCAP-M.exe” –EXPORT “drm folder path” “csv file path+ 파일명 .csv” “fav file path” | Name of Output file can be edited while using DOS Prompt for exporting Favorite file. When there are multiple log files to process, output files are overwritten in order. To avoid this phenomenon, output file is created after merging log files. | Model Creation - Merge |
| 3 | -MB | “XCAP-M.exe” –EXPORT “Logging File Path” “Export Path” -MB | Model creation adding –MB in the end, Message viewer (All msg) is exported. |  |
| 4 | -TXT | “XCAP-M.exe” –EXPORT “Logging File Path” “Export Path” -MB -TXT | Support Output Format - TXT |  |
| 5 | -all | ”XCAP-M.exe” –EXPORT “drm folder path” “csv file path to be exported” “fav file path” -all | Sampling interval - All, 1 sec all the times when Sampling Interval is not set. |  |
| 6 | -L3 -D -H | “XCAP-M.exe” –EXPORT “Logging File Path” “Export Path” -MB -L3 -D -H | -L3: only for L3 // New -D: Detail Message //New (For the case when ‘Include Detail Message’ is cheched.) -H: Hex Data //New (For the case when ‘Include Hexa Code’ is checked) |  |
| 7 | -BX | "XCAP-C.exe" -BX | Execute XCAP-C in XCAL |  |
| 8 | -MG | ”XCAP-M.exe” –EXPORT -mg “drm folder path” “exported csv path” “fav file path” | In case when adding –mg for fav format, export after merging log files. |  |
| 9 | -? | "OPTis Analyzer.exe" -? | Print out Help file. |  |
