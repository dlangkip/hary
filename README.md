// ==UserScript==
// @name         Auto Fill Dynamic Form
// @namespace    http://tampermonkey.net/
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
