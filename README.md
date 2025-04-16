# Drupal Module Compatibility Checker

Effortlessly assess your Drupal modules' compatibility for upgrades. Follow these steps:

1.  **Export Module List:** Log in to your Drupal site intended for upgrade, navigate to the modules administration page, and save the page as an HTML file.

2.  **Extract Module Information:**
    * Utilize an LLM (Language Model) to process the saved HTML file.
    * Instruct the LLM to extract the module name and its current version.
    * Organize this information in a spreadsheet (e.g., Google Sheets or Microsoft Excel) with the following columns:
        * **Column A:** Module Name
        * **Column B:** Current Version

3.  **Handle Pre-Drupal 8 Sites:** If your site is running a version older than Drupal 8, manually review and update the compatibility status of each module.

4.  **Prepare Drupal 8+ Data:** For Drupal 8 or later sites:
    * Locate and copy the contents of the `core.extension.yml` file. This file typically lists enabled modules in the format `module_name: 0`.
    * Paste this data into **Column C** of your spreadsheet.

5.  **Isolate Module Names (Column C):** Split the data in Column C to retain only the module names.

6.  **Determine Enabled Status (Column D):** Enter the following formula in the first data row of **Column D** to automatically determine if a module listed in Column A is currently enabled on your Drupal 8+ site:
    ```excel
    =IF(SUM(ArrayFormula(N(REGEXMATCH(A2, "(?m)^" & TEXTJOIN("$|^", TRUE, FILTER(C$2:C, C$2:C<>"")) & "$")))) > 0, "TRUE", "")
    ```
    Then, drag this formula down to apply it to all rows.

7.  **Set Up Google Apps Script:**
    * In your Google Sheet, navigate to **Extensions → Apps Script**.
    * Replace the default code in the script editor with the following:
    ```javascript
    function checkD11Compatibility(moduleName, version="11") {
      if (!moduleName) return "Module Name Missing";
      const url = `https://updates.drupal.org/release-history/${moduleName}/current`;
      try {
        const response = UrlFetchApp.fetch(url);
        const xml = response.getContentText();
        const document = XmlService.parse(xml);
        const project = document.getRootElement();
        const releases = project.getChild("releases");
        if (!releases) return "No";
        const releaseList = releases.getChildren("release");
        if (!releaseList || releaseList.length === 0) return "No Releases Available";
        const latestRelease = releaseList[0];
        const coreCompatibility = latestRelease.getChildText("core_compatibility");
        return coreCompatibility && coreCompatibility.includes(version) ? "Yes" : "No";
      } catch (e) {
        return "Error: " + e.message;
      }
    }
    function getD11Version(moduleName, version="11") {
      if (!moduleName) return "Module Name Missing";
      const url = `https://updates.drupal.org/release-history/${moduleName}/current`;
      try {
        const response = UrlFetchApp.fetch(url);
        const xml = response.getContentText();
        const document = XmlService.parse(xml);
        const project = document.getRootElement();
        const releases = project.getChild("releases");
        if (!releases) return "";
        const releaseList = releases.getChildren("release");
        if (!releaseList || releaseList.length === 0) return "";
        const latestRelease = releaseList[0];
        const coreCompatibility = latestRelease.getChildText("core_compatibility");
        const moduleversion = latestRelease.getChildText("version");
        return coreCompatibility && coreCompatibility.includes(version) ? moduleversion : "";
      } catch (e) {
        return "Error: " + e.message;
      }
    }
    ```
    * Save the script.

8.  **Check Drupal Compatibility (Column E onwards):** In new columns, use the following formulas to check module compatibility and retrieve the latest version for Drupal 10 and 11:
    * **Column E (Drupal 11 Compatibility):**
        * Check if a Drupal 11 version exists: `=checkD11Compatibility(A2)`
        * Get the latest Drupal 11 version: `=getD11Version(A2)`
    * **Column F (Drupal 10 Compatibility):**
        * Check if a Drupal 10 version exists: `=checkD11Compatibility(A2, 10)`
        * Get the latest Drupal 10 version: `=getD11Version(A2, 10)`

9.  **Apply to All Modules:** Drag the fill handle (the small square at the bottom-right of the selected cell) down to apply these formulas to all the module names in Column A.

This process will provide you with a clear overview of your modules' current versions, their enabled status on Drupal 8+ sites, and their compatibility with Drupal 10 and 11, along with the latest available versions for those Drupal versions.
