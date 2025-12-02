Here is the complete **README.md** file containing both scripts. You can create a file named `README.md` in your GitHub repository and paste the content below exactly as it is.

-----

````markdown
# Nyota Execuget Automation Scripts

## Overview
This repository contains two userscripts designed to automate the process of filling out assignment forms.

1.  **Auto Fill Assignment Form (Nyota Execuget):** The main script for filling the County, Class, Dates, and Rates on the specific creation page.
2.  **Auto Fill Dynamic Form:** A secondary script useful for forms where dropdown options load dynamically (slower loading pages).

## How to Install

1.  **Install a Script Manager:**
    * **Chrome/Edge:** Install **[Violentmonkey](https://violentmonkey.github.io/get-it/)**.
    * **Firefox:** Install **[Violentmonkey](https://addons.mozilla.org/en-US/firefox/addon/violentmonkey/)** or Tampermonkey.

2.  **Add a Script:**
    * Click the extension icon in your browser.
    * Select **"Create a new script"** (or the `+` button).
    * Delete any existing code in the editor.
    * Copy one of the scripts below and paste it in.
    * **Save** the script.

---

## Script 1: Main Assignment Auto-Filler
**Use this for the main `admin/assignments/create` page.**

### Configuration
Before saving, look at the `CONSTANTS` section at the top of the script. Change the `START_DATE`, `END_DATE`, `COUNTY`, etc., inside the quotes to match your current assignment details.

```javascript
// ==UserScript==
// @name         Auto Fill Assignment Form (Nyota Execuget)
// @namespace    ViolentMonkey
// @version      1.0
// @description  Auto-fill the assignment page with preset values
// @match        [https://nyota.execuget.co.ke/admin/assignments/create](https://nyota.execuget.co.ke/admin/assignments/create)
// @grant        none
// ==/UserScript==

(function () {
    'use strict';

    /************************************
     * CONSTANTS
     ************************************/
    const COUNTY = "Nairobi";              // County text
    const SUB_COUNTY = "Makadara";         // Sub-county text
    const CENTER = "Makadara";             // Center text
    const CLASS_NAME = "Class 1";          // Class text
    const START_DATE = "2025-11-26";       // yyyy-mm-dd format
    const END_DATE = "2025-11-29";         // yyyy-mm-dd format
    const RATE_PER_DAY = 0;                // Number
    const AMOUNT_PAID = "";                // Empty means leave blank

    /************************************
     * UTILITIES
     ************************************/
    function setSelectByText(selectId, text) {
        const select = document.querySelector(selectId);
        if (!select) return;

        const option = Array.from(select.options)
            .find(o => o.text.trim().toLowerCase() === text.toLowerCase());

        if (option) {
            select.value = option.value;
            select.dispatchEvent(new Event("change"));
        }
    }

    function setValue(id, value) {
        const el = document.querySelector(id);
        if (!el) return;
        el.value = value;
        el.dispatchEvent(new Event("input", { bubbles: true }));
        el.dispatchEvent(new Event("change", { bubbles: true }));
    }

    /************************************
     * MAIN FUNCTION
     ************************************/
    function autoFill() {

        // --- County/Subcounty/Center/Class ---
        setSelectByText("#county_id", COUNTY);
        setSelectByText("#sub_county_id", SUB_COUNTY);
        setSelectByText("#center_id", CENTER);
        setSelectByText("#class_room_id", CLASS_NAME);

        // --- Dates ---
        setValue("#start_date", START_DATE);
        setValue("#end_date", END_DATE);

        // --- Rate/day & payment ---
        setValue("#payment_rate_per_day", RATE_PER_DAY);
        if (AMOUNT_PAID !== "") setValue("#amount_paid", AMOUNT_PAID);

        // Calculate Days Automatically
        const start = new Date(START_DATE);
        const end = new Date(END_DATE);
        if (!isNaN(start) && !isNaN(end)) {
            const diffDays = Math.floor((end - start) / (1000 * 60 * 60 * 24)) + 1;
            setValue("#number_of_days", diffDays);
        }

        // Trigger recalculations for total & amount due
        const rate = Number(RATE_PER_DAY) || 0;
        const days = Number(document.querySelector("#number_of_days")?.value || 0);
        const total = rate * days;
        const paid = Number(AMOUNT_PAID) || 0;

        setValue("#total_amount", total.toFixed(2));
        setValue("#amount_due", (total - paid).toFixed(2));
    }

    /************************************
     * WAIT FOR PAGE
     ************************************/
    const interval = setInterval(() => {
        if (document.querySelector("#county_id")) {
            clearInterval(interval);
            autoFill();
        }
    }, 300);

})();
````

-----

## Script 2: Dynamic Form Auto-Filler

**Use this version if the dropdown menus (like Sub County) take a few seconds to load.** This script waits for the dropdown options to appear before selecting them.

### Configuration

Edit the `SUB_COUNTY`, `CENTER`, and `CLASS` variables in the `CONFIGURATION` section. You may also need to update the `@match` URL to the specific page you are using it on.

```javascript
// ==UserScript==
// @name         Auto Fill Dynamic Form
// @namespace    [http://tampermonkey.net/](http://tampermonkey.net/)
// @version      1.0
// @description  Autofill Sub County, Center, Class
// @match        *://*/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // ---------------------------
    // 🔧 CONFIGURATION (EDIT THESE)
    // ---------------------------
    const SUB_COUNTY = "MAKADARA";
    const CENTER = "MAKADARA";
    const CLASS = "Class 1";   // example — change to whatever text is inside the dropdown
    // ---------------------------

    // Helper: Wait until element exists
    function waitForElement(selector, callback) {
        const observer = new MutationObserver(() => {
            const elem = document.querySelector(selector);
            if (elem) {
                observer.disconnect();
                callback(elem);
            }
        });
        observer.observe(document.body, { childList: true, subtree: true });
    }

    // Helper: Select option by its visible text
    function selectByText(selectElem, text) {
        const options = [...selectElem.options];
        const match = options.find(opt => opt.text.trim().toLowerCase() === text.trim().toLowerCase());
        if (match) selectElem.value = match.value;
        selectElem.dispatchEvent(new Event("change", { bubbles: true }));
    }

    // Autofill function
    function autoFill() {
        waitForElement("select[name='sub_county']", (elem) => {
            let interval = setInterval(() => {
                if (elem.options.length > 1) {
                    clearInterval(interval);
                    selectByText(elem, SUB_COUNTY);
                }
            }, 300);
        });

        waitForElement("select[name='center']", (elem) => {
            let interval = setInterval(() => {
                if (elem.options.length > 1) {
                    clearInterval(interval);
                    selectByText(elem, CENTER);
                }
            }, 300);
        });

        waitForElement("select[name='class']", (elem) => {
            let interval = setInterval(() => {
                if (elem.options.length > 1) {
                    clearInterval(interval);
                    selectByText(elem, CLASS);
                }
            }, 300);
        });
    }

    autoFill();
})();
```

```
```
