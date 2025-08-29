# Filter Grid Generator <!-- omit in toc -->

A module to generate sets of filters and to filter JSON arrays. This module can be used in connection with any JSON dataset. 

Four separate functions are provided:

1. Generating filters grids
2. Applying sets of filters to JSON arrays
3. Resetting / clearing filtergrids
4. Populating filtergrids from sets of saved filters

https://github.com/user-attachments/assets/53a433ec-befc-4998-8fc0-78ecb605aa72

Filters can be displayed in two different ways:

1. Form

![](images/FormView.png)

2. Chips

![](images/ChipsView.png)

This module can be used in conjunction with DataGrids and [Client-Side Repeater DataGrids](https://github.com/stadium-software/repeater-datagrid-client-side) alike. Use this script with smaller datasets instead of the similar [DataGrid Advanced Search](https://github.com/stadium-software/datagrid-advanced-search). 

If you are using this module in connection with a [Client-Side Repeater DataGrid](https://github.com/stadium-software/repeater-datagrid-client-side) set up all elements necessary for the [Client-Side Repeater DataGrid](https://github.com/stadium-software/repeater-datagrid-client-side) before adding this filter grid module. 

## Contents <!-- omit in toc -->
1. [Version](#version)
   1. [Change Log](#change-log)
2. [Setup](#setup)
   1. [Application](#application)
   2. [Global Scripts](#global-scripts)
      1. [GenerateFilters Script](#generatefilters-script)
      2. [ApplyFilters Script](#applyfilters-script)
      3. [ClearFilters Script](#clearfilters-script)
      4. [SetFilters Script](#setfilters-script)
   3. [Types](#types)
      1. [FilterConfig Type Setup (Required)](#filterconfig-type-setup-required)
         1. [Manual Type Creation](#manual-type-creation)
         2. [Type Import](#type-import)
      2. [SelectedFilters Type Setup (Optional)](#selectedfilters-type-setup-optional)
         1. [Manual Type Creation](#manual-type-creation-1)
         2. [Type Import](#type-import-1)
   4. [Page](#page)
   5. [Event Handlers](#event-handlers)
      1. [Page.Load](#pageload)
      2. [Apply Button Click](#apply-button-click)
      3. [Clear Filters Button Click](#clear-filters-button-click)
   6. [Saved Filters](#saved-filters)
      1. [Saving Filters](#saving-filters)
      2. [Applying Saved Filters](#applying-saved-filters)
3. [Display Options](#display-options)
   1. [Integrated Button Display](#integrated-button-display)
4. [CSS](#css)
   1. [Before v6.12](#before-v612)
   2. [Customising CSS](#customising-css)
5. [Upgrading Stadium Repos](#upgrading-stadium-repos)

# Version
2.1

## Change Log
2.1 Integrated CSS into script and removed the requirement to include CSS files in the embedded files

2.2 Added icon color variable to [*filter-grid-variables.css*](filter-grid-variables.css)

# Setup

## Application 
1. Check the *Enable Style Sheet* checkbox in the application properties

## Global Scripts
This module requires the creation of four separate scripts. Each of these can be called separately, providing for flexibility in implementation. 

1. [GenerateFilters](#generatefilters-script): Generates the filtergrid from the configuration
2. [ApplyFilters](#applyfilters-script): Parses the filtergrid and applies the filters to a dataset
3. [ClearFilters](#clearfilters-script): Resets the filtergrid
4. [SetFilters](#setfilters-script): Populates a filtergrid with filter values from a set of saved filters

### GenerateFilters Script
1. Create a Global Script called "GenerateFilters"
2. Add the input parameters below to the Global Script
   1. FilterConfig
   2. FilterContainerClass
   3. Display
3. Drag a *JavaScript* action into the script
4. Add the Javascript below into the JavaScript code property
```javascript
/* Stadium Script 2.2 https://github.com/stadium-software/filter-grid */
let filterClassName = "." + ~.Parameters.Input.FilterContainerClass;
let filterConfig = ~.Parameters.Input.FilterConfig;
let filtersDisplay = ~.Parameters.Input.Display || "form";
loadCSS();
let displayChips = true;
if (filtersDisplay.toLowerCase() === "form") displayChips = false;
const insert = (arr, index, newItem) => [...arr.slice(0, index), newItem, ...arr.slice(index)];
let numberSelectChange = (e) => {
    let target = e.target;
    let toEl = target.closest("div").nextElementSibling.querySelector(".filtergrid-to-number");
    let fromEl = target.closest("div").nextElementSibling.querySelector(".filtergrid-from-number");
    if (target.value != "Between" && target.value != "From-To") {
        toEl.classList.add("visually-hidden");
        toEl.value = "";
        fromEl.setAttribute("placeholder", "Value");
    } else { 
        toEl.classList.remove("visually-hidden");
        fromEl.setAttribute("placeholder", "From");
    }
};
let dateSelectChange = (e) => {
    let target = e.target;
    let toEl = target.closest("div").nextElementSibling.querySelector(".filtergrid-to-date");
    let fromEl = target.closest("div").nextElementSibling.querySelector(".filtergrid-from-date");
    let targetVal = target.value.toLowerCase();
    if (targetVal == "greater than" || targetVal == "smaller than" || targetVal == "equals") {
        toEl.classList.add("visually-hidden");
        toEl.value = "";
        fromEl.setAttribute("placeholder", "Date");
    } else { 
        toEl.classList.remove("visually-hidden");
        fromEl.setAttribute("placeholder", "From");
    }
};
let filterContainer = document.querySelectorAll(filterClassName);
if (filterContainer.length == 0) {
    console.error("The container for the filter was not found. Drag a container control into the page and assign the class '" + filterClassName + "' to it.");
    return false;
} else if (filterContainer.length > 1) {
    console.error("The class '" + filterClassName + "' is assigned to multiple controls. Assign a unique classname to the filter container");
    return false;
}
filterContainer = filterContainer[0];
filterContainer.classList.add("stadium-filter-container");
if (!displayChips) {
    filterContainer.classList.add("stadium-filter-form-display");
} else {
    filterContainer.classList.add("stadium-filter-chips-display");
}
let filters = filterContainer.querySelector(".stadium-filters");
if (filters) {
    filters.remove();
}
let filterInnerContainer = filterContainer.querySelector(".stadium-filter-inner-container");
if (!filterInnerContainer) {
    filterInnerContainer = document.createElement("div");
    filterInnerContainer.classList.add("stadium-filter-inner-container");
    filterContainer.appendChild(filterInnerContainer);
}
let stadiumFilters = document.createElement("div");
stadiumFilters.classList.add("stadium-filters");
filterInnerContainer.appendChild(stadiumFilters);
let control = filterContainer.querySelectorAll(".control-container");
if (control.length > 0 && !filterContainer.querySelector(".filter-buttons") && displayChips) {
    let filterButtons = document.createElement("div");
    filterButtons.classList.add("filter-buttons");
    for (let i = 0; i < control.length; i++) {
        if (i === 0) { 
            control[i].classList.add("apply-button");
            control[i].querySelector("button").textContent = "";
        }
        if (i === 1) {
            control[i].classList.add("clear-button");
            control[i].querySelector("button").textContent = "";
        }
        filterButtons.appendChild(control[i]);
    }
    filterInnerContainer.prepend(filterButtons);
} else if (control.length > 0) {
    let stackLayout = document.createElement("div");
    stackLayout.classList.add("stack-layout-container");
    for (let i = 0; i < control.length; i++) {
        stackLayout.appendChild(control[i]);
    }
    insertAfter(stadiumFilters, stackLayout);
}
initFilterForm();
function initFilterForm() {
    for (let i = 0; i < filterConfig.length; i++) {
        let column = filterConfig[i].column;
        let colNo;
        let type = filterConfig[i].type;
        let name = filterConfig[i].name;
        let data = filterConfig[i].data;
        let display = filterConfig[i].display;
        let format = filterConfig[i].format;
        let operators = filterConfig[i].operators || [];
        operators = operators.map(v => v.toLowerCase());

        let label = document.createElement("div");
        label.classList.add("control-container","label-container");
        let labelInner = document.createElement("span");
        labelInner.textContent = name;
        label.appendChild(labelInner);

        let operator = document.createElement("div");
        let valueField = document.createElement("div");
        let select, input;

        if (type == "text") {
            select = document.createElement("select");
            let options = [{text:"Contains",value:"Contains"}, {text:"Does not contain",value:"Does not contain"}, {text:"Equals",value:"Equals"}, {text:"Does not equal",value:"Does not equal"}];
            if (displayChips) options = options = [{text:"Con",value:"Contains"}, {text:"!Con",value:"Does not contain"}, {text:"=",value:"Equals"}, {text:"!=",value:"Does not equal"}];
            for (let s = 0; s < options.length; s++) {
                let opt = options[s];
                if (operators.includes(opt.value.toLowerCase()) || operators.length == 0) {
                    let el = document.createElement("option");
                    el.textContent = opt.text;
                    el.value = opt.value;
                    el.setAttribute("orig", opt.text);
                    select.appendChild(el);
                }
            }
            if (operators.length == 1) select.setAttribute("readonly", "readonly");
            select.classList.add("form-control", "filter-operator");
            operator.classList.add("control-container", "drop-down-container");
            input = document.createElement("input");
            input.classList.add("form-control", "text-box-input", "filtergrid-text-value");
            input.setAttribute("placeholder", "Text");
            input.addEventListener("change", setTitle);
            valueField.classList.add("control-container", "text-box-container");
        }
        if (type == "number") {
            select = document.createElement("select");
            let options = [{text:"From-To",value:"From-To"}, {text:"Between",value:"Between"}, {text:"Equals",value:"Equals"}, {text:"Greater than",value:"Greater than"}, {text:"Smaller than",value:"Smaller than"}];
            if (displayChips) options = [{text:"x-y",value:"From-To"}, {text:"x< >y",value:"Between"}, {text:"=",value:"Equals"}, {text:">",value:"Greater than"}, {text:"<",value:"Smaller than"}];
            for(let s = 0; s < options.length; s++) {
                let opt = options[s];
                if (operators.includes(opt.value.toLowerCase()) || operators.length == 0) {
                    let el = document.createElement("option");
                    el.textContent = opt.text;
                    el.value = opt.value;
                    el.setAttribute("orig", opt.text);
                    select.appendChild(el);
                }
            }
            if (operators.length == 1) select.setAttribute("readonly", "readonly");
            select.classList.add("form-control", "filter-operator");
            operator.classList.add("control-container", "drop-down-container");
            let numInput1 = document.createElement("input");
            numInput1.classList.add("form-control", "text-box-input", "filtergrid-from-number");
            numInput1.setAttribute("placeholder", "From");
            numInput1.addEventListener("change", setTitle);
            let numInput2 = document.createElement("input");
            numInput2.classList.add("form-control", "text-box-input", "filtergrid-to-number");
            numInput2.setAttribute("placeholder", "To");
            numInput2.addEventListener("change", setTitle);
            input = document.createElement("div");
            input.classList.add("number-values");
            select.addEventListener("change", numberSelectChange);
            input.appendChild(numInput1);
            input.appendChild(numInput2);
        }
        if (type == "date") {
            if (!format) format = 'YYYY/MM/DD';
            select = document.createElement("select");
            let options = [{text:"From-To",value:"From-To"}, {text:"Between",value:"Between"}, {text:"Equals",value:"Equals"}, {text:"Greater than",value:"Greater than"}, {text:"Smaller than",value:"Smaller than"}];
            if (displayChips) options = [{text:"x-y",value:"From-To"}, {text:"x< >y",value:"Between"}, {text:"=",value:"Equals"}, {text:">",value:"Greater than"}, {text:"<",value:"Smaller than"}];
            for(let s = 0; s < options.length; s++) {
                let opt = options[s];
                if (operators.includes(opt.value.toLowerCase()) || operators.length == 0) {
                    let el = document.createElement("option");
                    el.textContent = opt.text;
                    el.value = opt.value;
                    el.setAttribute("orig", opt.text);
                    select.appendChild(el);
                }
            }
            if (operators.length == 1) select.setAttribute("readonly", "readonly");
            select.classList.add("form-control", "filter-operator");
            operator.classList.add("control-container", "drop-down-container");
            let dtInput1 = document.createElement("input");
            dtInput1.classList.add("form-control", "text-box-input", "filtergrid-from-date");
            dtInput1.setAttribute("placeholder", "From");
            dtInput1.setAttribute("format", format);
            dtInput1.addEventListener("change", setTitle);
            let dtInput2 = document.createElement("input");
            dtInput2.classList.add("form-control", "text-box-input", "filtergrid-to-date");
            dtInput2.setAttribute("placeholder", "To");
            dtInput2.addEventListener("change", setTitle);
            if (display == "picker") {
                dtInput1.type = "date";
                dtInput2.type = "date";
            }
            input = document.createElement("div");
            input.classList.add("date-values");
            select.addEventListener("change", dateSelectChange);
            input.appendChild(dtInput1);
            input.appendChild(dtInput2);
        }
        if (type == "boolean") {
            if (!data) {
                data = ["Show all", "Yes", "No" ];
            }
            if (!data.includes('Show all')) data = insert(data, 0, "Show all");
            if (display == "dropdown" || (display == "radio" || !display && displayChips)) {
                select = document.createElement("select");
                for(let s = 0; s < data.length; s++) {
                    let opt = data[s];
                    let el = document.createElement("option");
                    el.textContent = opt;
                    el.value = opt;
                    el.text = opt;
                    select.appendChild(el);
                }
                display = "dropdown";
                select.classList.add("form-control");
                select.addEventListener("change", setTitle);
                operator.classList.add("control-container", "drop-down-container", "filtergrid-boolean-operator", "span-2");
            } else {
                select = document.createElement("div");
                for (let s = 0; s < data.length; s++) {
                    let cont = document.createElement("div");
                    cont.classList.add("radio");
                    let opt = data[s];
                    let el = document.createElement("input");
                    let lab = document.createElement("label");
                    el.type = "radio";
                    el.name = column;
                    el.checked = false;
                    if (opt == "Show all") el.checked = true;
                    el.value = opt;
                    el.text = opt;
                    lab.textContent = opt;
                    el.id = column + "_" + opt.replaceAll(" ", "").toLowerCase();
                    lab.setAttribute("for", el.id);
                    cont.appendChild(el);
                    cont.appendChild(lab);
                    select.appendChild(cont);
                }
                operator.classList.add("control-container", "radio-button-list-container", "filtergrid-radiobutton-list", "span-2");
            }
            input = document.createElement("div");
            valueField.classList.add("no-display");
        }
        if (type == "enum") {
            data = insert(data, 0, "Show all");
            if (display == "radio" || display == "dropdown" && !displayChips) {
                select = document.createElement("div");
                for (let s = 0; s < data.length; s++) {
                    let cont = document.createElement("div");
                    cont.classList.add("radio");
                    let opt = data[s];
                    let el = document.createElement("input");
                    let lab = document.createElement("label");
                    el.type = "radio";
                    el.name = column;
                    el.checked = false;
                    if (opt == "Show all") el.checked = true;
                    el.value = opt;
                    lab.textContent = opt;
                    let fid = column + "_" + opt;
                    el.id = fid.replaceAll(" ", "").toLowerCase();
                    lab.setAttribute("for", el.id);
                    cont.appendChild(el);
                    cont.appendChild(lab);
                    select.appendChild(cont);
                }
                operator.classList.add("control-container", "radio-button-list-container", "filtergrid-radiobutton-list", "span-2");
            } else {
                select = document.createElement("select");
                for (let s = 0; s < data.length; s++) {
                    let opt = data[s];
                    let el = document.createElement("option");
                    el.textContent = opt;
                    el.value = opt;
                    select.appendChild(el);
                }
                select.classList.add("form-control");
                select.addEventListener("change", setTitle);
                operator.classList.add("control-container", "drop-down-container", "filtergrid-enum-operator", "span-2");
            }
            input = document.createElement("div");
            valueField.classList.add("no-display");
        }
        if (type == "multiselect") {
            select = document.createElement("div");
            for (let s = 0; s < data.length; s++) {
                let cont = document.createElement("div");
                cont.classList.add("checkbox");
                let opt = data[s];
                let el = document.createElement("input");
                let lab = document.createElement("label");
                el.type = "checkbox";
                el.checked = false;
                el.value = opt;
                lab.textContent = opt;
                let fid = column + "_" + opt;
                el.id = fid.replaceAll(" ", "").toLowerCase();
                lab.setAttribute("for", el.id);
                cont.appendChild(el);
                cont.appendChild(lab);
                select.appendChild(cont);
            }
            operator.classList.add("control-container", "check-box-list-container", "filtergrid-checkbox-list", "span-2");
            input = document.createElement("div");
            valueField.classList.add("no-display");
        }
        setAttributes(operator, { "foperator": column, "ftype": type, "cno": colNo, "fdisplay": display });
        operator.appendChild(select);

        setAttributes(valueField, { "fvalue": column, "ftype": type, "cno": colNo, "fdisplay": display });
        valueField.appendChild(input);
        if (!displayChips) {
            stadiumFilters.appendChild(label);
            stadiumFilters.appendChild(operator);
            stadiumFilters.appendChild(valueField);
        } else {
            let chip = document.createElement("div");
            chip.classList.add("stadium-filter-chip");
            chip.appendChild(label);
            chip.appendChild(operator);
            chip.appendChild(valueField);
            stadiumFilters.appendChild(chip);
        }
        select.dispatchEvent(new Event('change'));
    }
    if (displayChips) {
        let allOperators = filterInnerContainer.querySelectorAll(".filter-operator");
        for (let i = 0; i < allOperators.length; i++) {
            setDropDownText(allOperators[i]);
        }
        let allMultiSelects = filterInnerContainer.querySelectorAll(".check-box-list-container");
        for (let i = 0; i < allMultiSelects.length; i++) {
            createDropDown(allMultiSelects[i]);
        }
    }
}
function setTitle(e) {
    e.target.title = e.target.value;
}
function setDropDownText(select) {
    select.addEventListener('blur', (e) => {
        resetOperatorOptions(e.target);
    });
    select.addEventListener('change', (e) => {
        resetOperatorOptions(e.target);
    });
    select.addEventListener('focus', setOptionsClick);
}
function setOptionsClick(e){
    let t = e.target;
    let o = t.options;
    for (let i = 0; i < o.length; i++) {
        if (o[i].text.length > o[i].getAttribute("orig").length) break;
        o[i].text = o[i].text + " " + o[i].value;
    }
}
function resetOperatorOptions(t) {
    let o = t.options;
    for (let i = 0; i < o.length; i++) {
        o[i].text = o[i].getAttribute("orig");
    }
}
function setAttributes(el, attrs) {
  for(var key in attrs) {
    el.setAttribute(key, attrs[key]);
  }
}
function createDropDown(clist) {
    clist.classList.add("stadium-multi-select-dropdown");
    let header = document.createElement("div");
    header.textContent = "0 selected";
    header.classList.add("stadium-multi-select-dropdown-header", "control-container");
    header.addEventListener("click", function (e) {
        let head = e.target;
        let list = head.closest(".check-box-list-container");
        list.classList.toggle("expand");
        setHeader(head, list);
    });
    let clistItems = clist.querySelectorAll("div")[0];
    clistItems.classList.add("stadium-multi-select-checkboxlist");
    clistItems.before(header);

    document.body.addEventListener("click", function (e) {
        if (!e.target.closest(".check-box-list-container")) {
            let allDD = document.querySelectorAll(".check-box-list-container:has(.stadium-multi-select-checkboxlist)");
            for (let i = 0; i < allDD.length; i++) {
                let head = allDD[i].querySelector(".stadium-multi-select-dropdown-header");
                let list = head.closest(".check-box-list-container");
                list.classList.remove("expand");
                setHeader(head, list);
            }
        }
    });
}
function setHeader(header, l) {
    let s = l.querySelectorAll('input[type="checkbox"]:checked');
    let t = [];
    for (let i = 0; i < s.length; i++) {
        t.push(s[i].value);
    }
    header.textContent = s.length + " selected";
    header.title = t.join(", ");
}
function insertAfter(referenceNode, newNode) {
    referenceNode.parentNode.insertBefore(newNode, referenceNode.nextSibling);
}
function detectFlexWrap(flexContainer) {
    const flexItems = Array.from(flexContainer.children);
    if (flexItems.length <= 1) {
        return false;
    }
    let isWrapped = false;
    for (let i = 1; i < flexItems.length; i++) {
        const prevItem = flexItems[i - 1];
        const currentItem = flexItems[i];
        if (currentItem.offsetLeft < prevItem.offsetLeft) {
            isWrapped = true;
            break;
        }
    }
    let buttonContainer = filterInnerContainer.querySelector(".filter-buttons");
    if (buttonContainer) {
        if (isWrapped) {
            buttonContainer.classList.remove("row-display");
        } else {
            buttonContainer.classList.add("row-display");
        }
    }
}
if (displayChips) {
    window.onresize = function() {
        let flex = filterInnerContainer.querySelector(".stadium-filters");
        setTimeout(detectFlexWrap(flex), 1000);
    };
}
function loadCSS() {
    let moduleID = "stadium-filter-container";
    if (!document.getElementById(moduleID)) {
        let cssMain = document.createElement("style");
        cssMain.id = moduleID;
        cssMain.type = "text/css";
        cssMain.textContent = `/* Stadium CSS */
.stadium-filter-form-display.stadium-filter-container {
    margin-top: 1rem;

    .visually-hidden {
        height: 0;
        width: 0;
        overflow: hidden;
        outline: 0;
        border: 0;
        position: absolute;
        padding: 0;
    }

    .stadium-filters {
        border: 0.1rem solid var(--repeater-integrated-filter-border-color, var(--GENERAL-BORDER-COLOR));
        background-color: var(--filter-background-color, var(--BODY-BACKGROUND-COLOR));
        display: grid;
        grid-template-columns: minmax(min-content, max-content) max-content minmax(12rem, 1fr);
        align-items: center;
        padding: 0 0.6rem 1rem 0.6rem;
    }

    .stadium-filter-inner-container {
        overflow: hidden;
    }

    input:not([type='checkbox'], [type='radio']) {
        width: 100%;
        max-width: 30rem;
    }

    select.form-control {
        min-width: 13.5rem;
        width: 100%;
    }

    select.form-control[readonly='readonly'] {
        user-select: none;
        pointer-events: none;
        background-color: var(--FORM-CONTROL-BACKGROUND-COLOR, #f9f9f9);
        background-image: none;
    }

    .filtergrid-enum-operator > select.form-control,
    .filtergrid-boolean-operator > select.form-control,
    .filtergrid-checkbox-list > div {
        width: auto;
        max-width: 30rem;
    }
    .filtergrid-enum-operator:has(~.drop-down-container) > select.form-control,
    .drop-down-container ~ .filtergrid-enum-operator > select.form-control,
    .filtergrid-boolean-operator:has(~.drop-down-container) > select.form-control,
    .drop-down-container ~ .filtergrid-boolean-operator > select.form-control,
    .filtergrid-checkbox-list:has(~.drop-down-container) > div,
    .drop-down-container ~ .filtergrid-checkbox-list > div {
        width: 100%;
        max-width: 45rem;
    }

    .filter-operator {
        max-width: 13.5rem;
    }

    .control-container:has(.filtergrid-text-value),
    .filtergrid-text-value {
        padding-right: 0;
    }

    .number-values {
        display: flex;
        gap: 1rem;
        margin-top: 1rem;

        &>.form-control {
            width: calc(50% - 0.5rem);
            max-width: 14.5rem;
        }
    }

    .date-values {
        display: flex;
        gap: 1rem;
        margin-top: 1rem;

        &>.form-control {
            width: calc(50% - 0.5rem);
            max-width: 14.5rem;
        }
    }

    .filtergrid-checkbox-list,
    .filtergrid-radiobutton-list {
        padding: 0.6rem;
        margin-right: 0;
    }

    .label-container {
       overflow-wrap: break-word;
    }

    .label-container:has(+ .filtergrid-checkbox-list) {
        align-self: self-start;
    }

    .filter-button-bar {
        grid-column: 1 / 3;
        padding-bottom: 0.6rem;
        display: flex;
        > .button-container:nth-child(1) {
            order: var(--filter-clear-button-position, 1);
        }
    }
    .filtergrid-from-date:has(+ .visually-hidden),
    .filtergrid-from-number:has(+ .visually-hidden) {
        width: 100%;
        max-width: 30rem;
    }

    .no-display {
        display: none;
    }
    .span-2 {
        grid-column: 2 / span 2;
        padding-right: 0;
    }
}

.stadium-filter-chips-display.stadium-filter-container {
    .stadium-filter-inner-container {
        display: flex;
        flex-wrap: nowrap;
    }
    .stadium-filters {
        border: 0;
        background-color: transparent;
        display: inline-flex;
        row-gap: calc(var(--filter-chips-font-size, 1.1rem) + 0.5rem);
        column-gap: 1rem;
        flex-wrap: wrap;
        padding: 0.6rem;
    }
    .stadium-filter-chip {
        display: flex;
        flex-wrap: nowrap;
        position: relative;
        height: var(--filter-chips-chip-height, 3rem);
        width: var(--filter-chips-chip-width, 13rem);

        .control-container {
            padding-right: 0;
        }
        .form-control {
            height: var(--filter-chips-chip-height, 3rem);
            padding: 0 0 0 var(--filter-chips-chip-left-padding, 0.2rem);
            color: var(--FORM-CONTROL-FONT-COLOR);
            background-color: var(--FORM-CONTROL-BACKGROUND-COLOR);
            border-top: var(--FORM-CONTROL-BORDER-TOP-WIDTH) solid var(--FORM-CONTROL-BORDER-TOP-COLOR);
            border-right: var(--FORM-CONTROL-BORDER-RIGHT-WIDTH) solid var(--FORM-CONTROL-BORDER-RIGHT-COLOR);
            border-bottom: var(--FORM-CONTROL-BORDER-BOTTOM-WIDTH) solid var(--FORM-CONTROL-BORDER-BOTTOM-COLOR);
            border-left: var(--FORM-CONTROL-BORDER-LEFT-WIDTH) solid var(--FORM-CONTROL-BORDER-LEFT-COLOR);
            border-radius: var(--FORM-CONTROL-BORDER-RADIUS);
        }
        .label-container {
            position: absolute;
            top: calc((var(--filter-chips-label-font-size, 1.1rem) + 0.1rem) * -1);
            left: 0;
            font-size: var(--filter-chips-label-font-size, 1rem);
            font-style: var(--filter-chips-label-font-style, italic);
            color: var(--filter-chips-label-font-color, var(--BODY-FONT-COLOR));
            padding: 0;
            span {
                white-space: nowrap;
            }
        }
        .filtergrid-enum-operator select,
        .filtergrid-boolean-operator select {
            font-size: var(--filter-chips-font-size, 1.1rem);
            width: var(--filter-chips-chip-width, 13rem);
            background-position: calc(100% - 1rem) 0.8rem, calc(100% - 0.5rem) 0.8rem, calc(100% - 2rem) 0.2rem;
            border-radius: var(--filter-chips-chip-border-radius, 0.5rem);
        }
        .filter-operator {
            border-top-left-radius: var(--filter-chips-chip-border-radius, 0.5rem);
            border-bottom-left-radius: var(--filter-chips-chip-border-radius, 0.5rem);
            font-size: var(--filter-chips-font-size, 1.1rem);
            width: var(--filter-chips-operators-width, 3rem);
            background-image: none;
            background-position: calc(100% - 1rem) 0.8rem, calc(100% - 0.5rem) 0.8rem, calc(100% - 2rem) 0.2rem;
            padding: 0 0 0 var(--filter-chips-chip-left-padding, 0.2rem);
            color: var(--filter-chips-operators-font-color, var(--FORM-CONTROL-FONT-COLOR));
            text-align: center;
            border-right: 0.1rem solid var(--DARKER-GREY);
        }
        .filter-operator:open {
            text-align: left;
        }
        .text-box-container input {
            font-size: var(--filter-chips-font-size, 1.1rem);
            width: calc(var(--filter-chips-chip-width, 13rem) - var(--filter-chips-operators-width, 3rem));
            padding-left: var(--filter-chips-chip-left-padding, 0.2rem);
            border-left: 0;
            border-top-right-radius: var(--filter-chips-chip-border-radius, 0.5rem);
            border-bottom-right-radius: var(--filter-chips-chip-border-radius, 0.5rem);
        }
        .date-values,
        .number-values {
            display: flex;
            flex-wrap: nowrap;
            margin-top: var(--CONTROL-CONTAINER-TOP-MARGIN);
            input {
                font-size: var(--filter-chips-font-size, 1.1rem);
                width: calc((var(--filter-chips-chip-width, 13rem) - var(--filter-chips-operators-width, 3rem)) / 2);
                border-left: 0;
                padding-left: var(--filter-chips-chip-left-padding, 0.2rem);
                border-right: 0.1rem solid var(--DARKER-GREY);
            }
            input:has(+ .visually-hidden) {
                width: calc(var(--filter-chips-chip-width, 13rem) - var(--filter-chips-operators-width, 3rem));
                border-top-right-radius: var(--filter-chips-chip-border-radius, 0.5rem);
                border-bottom-right-radius: var(--filter-chips-chip-border-radius, 0.5rem);
                border-right: var(--FORM-CONTROL-BORDER-RIGHT-WIDTH) solid var(--FORM-CONTROL-BORDER-RIGHT-COLOR);
            }
            input:last-of-type {
                border-top-right-radius: var(--filter-chips-chip-border-radius, 0.5rem);
                border-bottom-right-radius: var(--filter-chips-chip-border-radius, 0.5rem);
                border-right: var(--FORM-CONTROL-BORDER-RIGHT-WIDTH) solid var(--FORM-CONTROL-BORDER-RIGHT-COLOR);
            }
            .visually-hidden {
                height: 0;
                width: 0;
                overflow: hidden;
                outline: 0;
                border: 0;
                position: absolute;
                padding: 0;
            }
        }
        .filtergrid-checkbox-list {
            width: 100%;
            label {
                font-size: var(--filter-chips-font-size, 1.1rem);
            }
        }
    }

    .check-box-list-container:has(.stadium-multi-select-checkboxlist) {
        border: 0;
        position: relative;
        margin-top: 0;
        background: transparent;
        margin-right: 0;

        &:after {
            top: 0;
            right: 0;
        }

        .stadium-multi-select-checkboxlist {
            display: none;
            position: absolute;
            z-index: 10;
            top: 3.3rem;
            left: -0.1rem;
            border: 0.1rem solid var(--filter-chips-checkbox-list-border-color, var(--GENERAL-BORDER-COLOR));
            width: 100%;
            background-color: var(--filter-chips-checkbox-list-background-color, var(--BODY-BACKGROUND-COLOR, #f9f9f9));
            > div:not(.stadium-multi-select-dropdown-header, .checkbox) {
                box-shadow: rgba(0, 0, 0, 0.07) 0 0.1rem 0.1rem, rgba(0, 0, 0, 0.07) 0 0.2rem 0.2rem, rgba(0, 0, 0, 0.07) 0 0.4rem 0.4rem, rgba(0, 0, 0, 0.07) 0 0.8rem 0.8rem, rgba(0, 0, 0, 0.07) 0 1.6rem 1.6rem;
                background-color: var(--multi-select-background-color, var(--CHECK-BOX-LIST-CONTAINER-BACKGROUND-COLOR));
            }
        }
        .stadium-multi-select-dropdown-header {
            display: flex;
            align-items: center;
            height: var(--filter-chips-chip-height, 3rem);
            width: var(--filter-chips-chip-width, 13rem);
            cursor: pointer;
            user-select: none;
            font-size: var(--filter-chips-font-size, 1.1rem);
            border-radius: var(--filter-chips-chip-border-radius, 0.5rem);
            border-top: var(--FORM-CONTROL-BORDER-TOP-WIDTH) solid var(--FORM-CONTROL-BORDER-TOP-COLOR);
            border-right: var(--FORM-CONTROL-BORDER-RIGHT-WIDTH) solid var(--FORM-CONTROL-BORDER-RIGHT-COLOR);
            border-bottom: var(--FORM-CONTROL-BORDER-BOTTOM-WIDTH) solid var(--FORM-CONTROL-BORDER-BOTTOM-COLOR);
            border-left: var(--FORM-CONTROL-BORDER-LEFT-WIDTH) solid var(--FORM-CONTROL-BORDER-LEFT-COLOR);
            background-color: var(--FORM-CONTROL-BACKGROUND-COLOR);
            padding: 0 0 0 var(--filter-chips-chip-left-padding, 0.2rem);
            color: var(--FORM-CONTROL-FONT-COLOR);
            background-image: linear-gradient(45deg, transparent 50%, var(--DROP-DOWN-FONT-COLOR) 50%), linear-gradient(135deg, var(--DROP-DOWN-FONT-COLOR) 50%, transparent 50%);
            background-position: calc(100% - 1rem) 0.8rem, calc(100% - 0.5rem) 0.8rem, calc(100% - 3rem) 0.6rem;
            background-size: 0.5rem 0.5rem, 0.5rem 0.5rem, 0.1rem 1.8rem;
            background-repeat: no-repeat;
        }
    }
    .check-box-list-container.expand:has(.stadium-multi-select-checkboxlist) {
        .stadium-multi-select-checkboxlist {
            display: block;
            box-shadow: rgba(0, 0, 0, 0.07) 0 0.1rem 0.1rem, rgba(0, 0, 0, 0.07) 0 0.2rem 0.2rem, rgba(0, 0, 0, 0.07) 0 0.4rem 0.4rem, rgba(0, 0, 0, 0.07) 0 0.8rem 0.8rem, rgba(0, 0, 0, 0.07) 0 1.6rem 1.6rem;
            .checkbox {
                display: flex;
                align-items: center;
            }
        }
    }
}
.filter-buttons.row-display {
    flex-direction: row;
}
.filter-buttons {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    .control-container {
        margin-top: 0.1rem;
        padding-right: 0.1rem;
    }
    button:is(.btn-default),
    button:is(.btn-default):hover {
        font-size: var(--filter-chips-font-size, 1.1rem);
        padding: 0.2rem 0.6rem;
    }
    .apply-button *:is(.btn-default) {
        mask-image: var(--filter-chips-apply-button-icon, url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='1em' height='1em' viewBox='0 0 24 24'%3E%3C!-- Icon from Material Symbols by Google - https://github.com/google/material-design-icons/blob/master/LICENSE --%3E%3Cpath fill='currentColor' d='m9.55 15.15l8.475-8.475q.3-.3.7-.3t.7.3t.3.713t-.3.712l-9.175 9.2q-.3.3-.7.3t-.7-.3L4.55 13q-.3-.3-.288-.712t.313-.713t.713-.3t.712.3z'/%3E%3C/svg%3E"));
        mask-repeat: no-repeat;
        mask-position: center;
        mask-size: contain;
        background-size: var(--filter-chips-apply-clear-icon-size, 2.5rem);
        background-color: var(--filter-chips-apply-clear-icon-color, var(--BODY-FONT-COLOR));
        height: var(--filter-chips-apply-clear-icon-size, 2.5rem);
        width: var(--filter-chips-apply-clear-icon-size, 2.5rem);
        box-shadow: none;
   }
    .clear-button *:is(.btn-default) {
        mask-image: var(--filter-chips-clear-button-icon, url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='1em' height='1em' viewBox='0 0 24 24'%3E%3C!-- Icon from Material Symbols by Google - https://github.com/google/material-design-icons/blob/master/LICENSE --%3E%3Cpath fill='currentColor' d='m12 13.4l-2.917 2.925q-.277.275-.704.275t-.704-.275q-.275-.275-.275-.7t.275-.7L10.6 12L7.675 9.108Q7.4 8.831 7.4 8.404t.275-.704q.275-.275.7-.275t.7.275L12 10.625L14.892 7.7q.277-.275.704-.275t.704.275q.3.3.3.713t-.3.687L13.375 12l2.925 2.917q.275.277.275.704t-.275.704q-.3.3-.712.3t-.688-.3z'/%3E%3C/svg%3E"));
        mask-repeat: no-repeat;
        mask-position: center;
        mask-size: contain;
        background-size: var(--filter-chips-apply-clear-icon-size, 2.5rem);
        background-color: var(--filter-chips-apply-clear-icon-color, var(--BODY-FONT-COLOR));
        border-color: transparent;
        height: var(--filter-chips-apply-clear-icon-size, 2.5rem);
        width: var(--filter-chips-apply-clear-icon-size, 2.5rem);
        box-shadow: none;
    }
    .apply-button *:is(.btn-default:hover), 
    .clear-button *:is(.btn-default:hover) {
        border: 0.1rem solid var(--BUTTON--BUTTON-BORDER-COLOR);
    }
}
html {
    min-height: 100%;
    font-size: 62.5%;
}`;
        document.head.appendChild(cssMain);
    }   
}
```

### ApplyFilters Script
1. Create a Global Script called "ApplyFilters"
2. Add the input parameters below to the Global Script
   1. Data
   2. FilterContainerClass
3. Add the output parameters below to the Global Script
   1. Data
4. Drag a *JavaScript* action into the script
5. Add the Javascript below into the JavaScript code property
6. Drag a *SetValue* action below the JavaScript code action
   1. Target: ~.Parameters.Output.Data
   2. Source: ~.JavaScript
```javascript
/* Stadium Script 2.2 https://github.com/stadium-software/filter-grid */
let filterClassName = "." + ~.Parameters.Input.FilterContainerClass;
let data = ~.Parameters.Input.Data || [];
if (!Array.isArray(data)) {
    console.error("The *Data* parameter must be a List or Array.");
    return false;
}
let filterContainer = document.querySelectorAll(filterClassName);
if (filterContainer.length == 0) {
    console.error("A container with the class '" + filterClassName + "' was not found.");
    return false;
} else if (filterContainer.length > 1) {
    console.error("The class '" + filterClassName + "' is assigned to multiple controls.");
    return false;
}
filterContainer = filterContainer[0];
let res = filterRepeaterData();
return res;
/*-----------------------------*/
function filterRepeaterData(){
    let returnData = data;
    let arrReturn = [];
    let operatorEls = filterContainer.querySelectorAll(".stadium-filters [foperator]");
    for (let i = 0; i < operatorEls.length; i++) {
        let ftype = operatorEls[i].getAttribute("ftype");
        let fdisplay = operatorEls[i].getAttribute("fdisplay");
        let foperator = operatorEls[i].getAttribute("foperator");
        let fvalueEl = operatorEls[i].nextElementSibling;
        if (ftype == "text") {
            let txtoperator = operatorEls[i].querySelector("select").value;
            let txtvalue = fvalueEl.querySelector("input").value;
            if (txtvalue) {
                if (txtoperator.toLowerCase() == "contains") {
                    returnData = contains(returnData, foperator, txtvalue);
                } else if (txtoperator.toLowerCase() == "does not contain") {
                    returnData = notcontains(returnData, foperator, txtvalue);
                } else if (txtoperator.toLowerCase() == "equals") {
                    returnData = equals(returnData, foperator, txtvalue);
                } else if (txtoperator.toLowerCase() == "does not equal") {
                    returnData = notequals(returnData, foperator, txtvalue);
                }
                arrReturn.push({ "column": foperator, "selectedoperator": txtoperator, selectedvalues: [txtvalue] });
            }
        }
        if (ftype == "number") {
            let numoperator = operatorEls[i].querySelector("select").value;
            let numvaluefromEl = fvalueEl.querySelector(".filtergrid-from-number");
            let numvaluetoEl = fvalueEl.querySelector(".filtergrid-to-number");
            let numvaluefrom = numvaluefromEl.value;
            let numvalueto = numvaluetoEl.value;
            if (numvaluefrom) {
                if (!numvaluefrom) numvaluefrom = -9007199254740991;
                if (!numvalueto) numvalueto = 9007199254740991;
                numvaluefromEl.value = numvaluefrom;
                numvaluetoEl.value = numvalueto;
                if (numoperator.toLowerCase() == "between") {
                    returnData = betweenNumbers(returnData, foperator, numvaluefrom, numvalueto);
                } else if (numoperator.toLowerCase() == "from-to") {
                    returnData = fromtoNumbers(returnData, foperator, numvaluefrom, numvalueto);
                } else if (numoperator.toLowerCase() == "equals") {
                    returnData = equals(returnData, foperator, numvaluefrom);
                } else if (numoperator.toLowerCase() == "greater than") {
                    returnData = greaterNumbers(returnData, foperator, numvaluefrom);
                } else if (numoperator.toLowerCase() == "smaller than") {
                    returnData = smallerNumbers(returnData, foperator, numvaluefrom);
                }
                arrReturn.push({"column": foperator, "selectedoperator":numoperator, selectedvalues: [numvaluefrom, numvalueto]});
            }
        }
        if (ftype == "date") {
            let dtoperator = operatorEls[i].querySelector("select").value;
            let fromEl = fvalueEl.querySelector(".filtergrid-from-date");
            let toEl = fvalueEl.querySelector(".filtergrid-to-date");
            let format = fromEl.getAttribute("format");
            if (fromEl.value) {
                let dtvaluefrom = dayjs(fromEl.value).format(format);
                let dtvalueto = dayjs(toEl.value).format(format);
                if (dtvaluefrom == "Invalid Date") dtvaluefrom = dayjs('1000/01/01').format(format);
                if (dtvalueto == "Invalid Date") dtvalueto = dayjs('3000/01/01').format(format);
                if (fromEl.type == "date") {
                    fromEl.value = dayjs(dtvaluefrom).format('YYYY-MM-DD');
                    toEl.value = dayjs(dtvalueto).format('YYYY-MM-DD');
                } else {
                    fromEl.value = dtvaluefrom;
                    toEl.value = dtvalueto;
                }
                if (dtoperator.toLowerCase() == "between") {
                    returnData = betweenDates(returnData, foperator, dtvaluefrom, dtvalueto, format);
                } else if (dtoperator.toLowerCase() == "from-to") {
                    returnData = fromtoDates(returnData, foperator, dtvaluefrom, dtvalueto, format);
                } else if (dtoperator.toLowerCase() == "equals") {
                    returnData = equalsDate(returnData, foperator, dtvaluefrom, format);
                } else if (dtoperator.toLowerCase() == "greater than") {
                    returnData = greaterDates(returnData, foperator, dtvaluefrom, format);
                } else if (dtoperator.toLowerCase() == "smaller than") {
                    returnData = smallerDates(returnData, foperator, dtvaluefrom, format);
                }
                arrReturn.push({"column": foperator, "selectedoperator":dtoperator, selectedvalues: [dtvaluefrom, dtvalueto]});
            }
        }
        let selectedvals = [];
        if (ftype == "boolean" && fdisplay == "dropdown") {
            let booloperator = operatorEls[i].querySelector("select").value;
            if (booloperator != "Show all" && booloperator != "") {
                returnData = equals(returnData, foperator, /^(true|yes|1)$/i.test(booloperator.toLowerCase()));
                selectedvals.push(booloperator);
            }
        } else if (ftype == "boolean") {
            let multioperator = operatorEls[i].querySelector("input[type='radio']:checked");
            if (multioperator.value != "Show all") {
                returnData = equals(returnData, foperator, /^(true|yes|1)$/i.test(multioperator.value.toLowerCase()));
                selectedvals.push(multioperator.value);
            }
        }
        if (ftype == "enum" && fdisplay == "radio") {
            let multioperator = operatorEls[i].querySelector("input[type='radio']:checked");
            if (multioperator.value != "Show all") {
                returnData = equals(returnData, foperator, multioperator.value);
                selectedvals.push(multioperator.value);
            }
        } else if (ftype == "enum") {
            let enumoperator = operatorEls[i].querySelector("select").value;
            if (enumoperator != "Show all" && enumoperator != "") {
                returnData = equals(returnData, foperator, enumoperator);
                selectedvals.push(enumoperator);
            }
        }
        if (ftype == "multiselect") {
            let multioperator = operatorEls[i].querySelectorAll("input[type='checkbox']:checked");
            for (let s = 0; s < multioperator.length; s++) {
                selectedvals.push(multioperator[s].value);
            }
            if (multioperator.length > 0) returnData = inArray(returnData, foperator, selectedvals);
        }
        if (selectedvals.length > 0) arrReturn.push({ "column": foperator, selectedvalues: selectedvals });
    }
    filterContainer.classList.remove("expand");
    return {data: returnData, filters: arrReturn};
}
function contains(d, f, v) {
    return d.filter(el => el[f].indexOf(v) > -1 && el[f] !== undefined);
}
function notcontains(d, f, v) {
    return d.filter(el => el[f] === undefined || el[f].indexOf(v) == -1);
}
function equals(d, f, v) {
    return d.filter(el => el[f] == v && el[f] !== undefined);
}
function notequals(d, f, v) {
    return d.filter(el => el[f] === undefined || el[f] != v);
}
function betweenNumbers(d, f, v1, v2) {
    return d.filter(el => el[f] > v1 && el[f] < v2 && el[f] !== undefined);
}
function fromtoNumbers(d, f, v1, v2) {
    return d.filter(el => el[f] >= v1 && el[f] <= v2 && el[f] !== undefined);
}
function greaterNumbers(d, f, v) {
    return d.filter(el => el[f] > v && el[f] !== undefined);
}
function smallerNumbers(d, f, v) {
    return d.filter(el => el[f] < v && el[f] !== undefined);
}
function inArray(d,f,a) { 
    return d.filter(el => a.includes(el[f]) && el[f] !== undefined);
}
function betweenDates(d, o, v1, v2, f) {
    return d.filter(el => dayjs(el[o]).format(f) > dayjs(v1).format(f) && dayjs(el[o]).format(f) < dayjs(v2).format(f) && el[o] !== undefined);
}
function fromtoDates(d, o, v1, v2, f) {
    return d.filter(el => dayjs(el[o]).format(f) > dayjs(v1).add(-1, 'day').format(f) && dayjs(el[o]).format(f) < dayjs(v2).add(+1, 'day').format(f) && el[o] !== undefined);
}
function greaterDates(d, o, v, f) {
    return d.filter(el => dayjs(el[o]).format(f) > dayjs(v).format(f) && el[o] !== undefined);
}
function smallerDates(d, o, v, f) {
    return d.filter(el => dayjs(el[o]).format(f) < dayjs(v).format(f) && el[o] !== undefined);
}
function equalsDate(d, o, v, f) {
    return d.filter(el => dayjs(el[o]).format(f) == dayjs(v).format(f) && el[o] !== undefined);
}
```

### ClearFilters Script
1. Create a Global Script called "ClearFilters"
2. Add the input parameters below to the Global Script
   1. FilterContainerClass
3. Drag a *JavaScript* action into the script
4. Add the Javascript below into the JavaScript code property
```javascript
/* Stadium Script 2.2 https://github.com/stadium-software/filter-grid */
let filterClassName="."+~.Parameters.Input.FilterContainerClass;
let filterContainer=document.querySelectorAll(filterClassName);
if (filterContainer.length==0) {
    console.error("The container for the filter was not found. Drag a container control into the page and assign the class '" + filterClassName + "' to it.");
    return false;
}
else if (filterContainer.length > 1) {
    console.error("The class '" + filterClassName + "' is assigned to multiple controls. Assign a unique classname to the filter container");
    return false;
}
filterContainer = filterContainer[0];
let displayChips = false;
if (filterContainer.classList.contains("stadium-filter-chips-display")) displayChips = true;
let stadiumFilters=filterContainer.querySelector(".stadium-filters");
clearForm();
function clearForm() {
    let allCheckboxes=stadiumFilters.querySelectorAll("input[type='checkbox']");
    for (let i=0; i < allCheckboxes.length; i++) {
        allCheckboxes[i].checked=false;
    }
    let allRadios=stadiumFilters.querySelectorAll("input[type='radio']");
    for (let i=0; i < allRadios.length; i++) {
        allRadios[i].checked=false;
        if (allRadios[i].value=="Show all") allRadios[i].checked=true;
    }
    let allInputs=stadiumFilters.querySelectorAll("input:not([type='checkbox'],[type='radio'])");
    for (let i=0; i < allInputs.length; i++) {
        allInputs[i].value="";
    }
    let allSelects=stadiumFilters.querySelectorAll("select");
    for (let i=0; i < allSelects.length; i++) {
        allSelects[i].options.selectedIndex=0;
    }
    let visuallyHidden=stadiumFilters.querySelectorAll(".visually-hidden");
    for (let i=0; i < visuallyHidden.length; i++) {
        visuallyHidden[i].classList.remove("visually-hidden");
    }
    let operators=stadiumFilters.querySelectorAll(".filter-operator");
    for (let i=0; i < operators.length; i++) {
        operators[i].dispatchEvent(new Event('change'));
    }
    if (displayChips) {
        let allMultiSelects = stadiumFilters.querySelectorAll(".check-box-list-container");
        for (let i = 0; i < allMultiSelects.length; i++) {
            setHeader(allMultiSelects[i]);
        }
    }
}
function setHeader(c) {
    let s = c.querySelectorAll('input[type="checkbox"]:checked');
    let header = c.querySelector(".stadium-multi-select-dropdown-header");
    let t = [];
    for (let i = 0; i < s.length; i++) {
        t.push(s[i].value);
    }
    header.textContent = s.length + " selected";
    header.title = t.join(", ");
}
```

### SetFilters Script
1. Create a Global Script called "SetFilters"
2. Add the input parameters below to the Global Script
   1. FilterContainerClass
   2. SelectedFilters
3. Drag a *JavaScript* action into the script
4. Add the Javascript below into the JavaScript code property
```javascript
/* Stadium Script 2.2 https://github.com/stadium-software/filter-grid */
let filterClassName = "." + ~.Parameters.Input.FilterContainerClass;
let selectedFilters = ~.Parameters.Input.SelectedFilters || [];
let filterContainer = document.querySelectorAll(filterClassName);
if (filterContainer.length == 0) {
    console.error("The container for the filter was not found. Drag a container control into the page and assign the class '" + filterClassName + "' to it.");
    return false;
} else if (filterContainer.length > 1) {
    console.error("The class '" + filterClassName + "' is assigned to multiple controls. Assign a unique classname to the filter container");
    return false;
}
filterContainer = filterContainer[0];
let displayChips = false;
if (filterContainer.classList.contains("stadium-filter-chips-display")) displayChips = true;
let stadiumFilters = filterContainer.querySelector(".stadium-filters");
if (selectedFilters.length > 0) {
    setSelectedFilters();
}
function setSelectedFilters() {
    clearForm();
    for (let i=0;i < selectedFilters.length;i++) {
        let column = selectedFilters[i].column, 
            selectedoperator = selectedFilters[i].selectedoperator,
            selectedvalues = selectedFilters[i].selectedvalues,
            operator = filterContainer.querySelector("[foperator='" + column + "']"),
            value = filterContainer.querySelector("[fvalue='" + column + "']"),
            select, radioinputs, checkinputs, display, type;
        if (operator) {
            display = operator.getAttribute("fdisplay");
            type = operator.getAttribute("ftype");
            radioinputs = operator.querySelectorAll("input[type='radio']");
            checkinputs = operator.querySelectorAll("input[type='checkbox']");
            select = operator.querySelector("select");
        }
        if (type == "text") {
            if (selectedoperator && [...select.options].map(el => el.value).includes(selectedoperator.toString())) select.value = selectedoperator;
            if (selectedvalues && selectedvalues.length > 0) value.querySelector("input").value = selectedvalues[0];
        }
        if (type == "number") {
            if (selectedoperator && [...select.options].map(el => el.value).includes(selectedoperator.toString())) select.value = selectedoperator;
            if (selectedvalues && selectedvalues.length > 0) value.querySelectorAll("input")[0].value = selectedvalues[0];
            if (selectedvalues && selectedvalues.length > 0) value.querySelectorAll("input")[1].value = selectedvalues[1];
        }
        if (type == "date") {
            let fval1 = value.querySelectorAll("input")[0];
            let fval2 = value.querySelectorAll("input")[1];
            if (selectedoperator && [...select.options].map(el => el.value).includes(selectedoperator.toString())) select.value = selectedoperator;
            if (selectedvalues && selectedvalues.length > 0) {
                if (fval1.type == "date") {
                    fval1.value = dayjs(selectedvalues[0]).format('YYYY-MM-DD');
                } else { 
                    fval1.value = selectedvalues[0];
                }
            }
            if (selectedvalues && selectedvalues.length > 0) {
                if (fval2.type == "date") {
                    fval2.value = dayjs(selectedvalues[1]).format('YYYY-MM-DD');
                } else {
                    fval2.value = selectedvalues[1];
                }
            }
        }
        if (type == "boolean") {
            if (display == "dropdown") {
                if (selectedvalues && selectedvalues.length > 0 && [...select.options].map(el => el.value).includes(selectedvalues[0].toString())) select.value = selectedvalues[0];
            } else {
                for (let s = 0; s < radioinputs.length; s++) {
                    if (selectedvalues && selectedvalues.length > 0 && selectedvalues[0] == radioinputs[s].value) radioinputs[s].checked = true;
                }
            }
        }
        if (type == "enum") {
            if (display == "radio") {
                for (let s = 0; s < radioinputs.length; s++) {
                    if (selectedvalues && selectedvalues.length > 0 && selectedvalues[0] == radioinputs[s].value) radioinputs[s].checked = true;
                }
            } else {
                if (selectedvalues && selectedvalues.length > 0 && [...select.options].map(el => el.value).includes(selectedvalues[0].toString())) select.value = selectedvalues[0];
            }
        }
        if (type == "multiselect") {
            for (let s = 0; s < checkinputs.length; s++) {
                if (selectedvalues && selectedvalues.length > 0 && selectedvalues.includes(checkinputs[s].value)) checkinputs[s].checked = true;
            }
            if (displayChips && checkinputs.length > 0) setHeader(checkinputs[0].closest(".stadium-multi-select-dropdown"));
        }
        if (select) select.dispatchEvent(new Event('change'));
    }
}
function setHeader(c) {
    let s = c.querySelectorAll('input[type="checkbox"]:checked');
    let header = c.querySelector(".stadium-multi-select-dropdown-header");
    let t = [];
    for (let i = 0; i < s.length; i++) {
        t.push(s[i].value);
    }
    header.textContent = s.length + " selected";
    header.title = t.join(", ");
}
function clearForm() { 
    let allCheckboxes = stadiumFilters.querySelectorAll("input[type='checkbox']");
    for (let i = 0; i < allCheckboxes.length; i++) {
        allCheckboxes[i].checked = false;
    }
    let allRadios = stadiumFilters.querySelectorAll("input[type='radio']");
    for (let i = 0; i < allRadios.length; i++) {
        allRadios[i].checked = false;
        if (allRadios[i].value == "Show all") allRadios[i].checked = true;
    } 
    let allInputs = stadiumFilters.querySelectorAll("input:not([type='checkbox'],[type='radio'])");
    for (let i = 0; i < allInputs.length; i++) {
        allInputs[i].value = "";
    }   
    let allSelects = stadiumFilters.querySelectorAll("select");
    for (let i = 0; i < allSelects.length; i++) {
        allSelects[i].options.selectedIndex = 0;
    }
    let visuallyHidden = stadiumFilters.querySelectorAll(".visually-hidden");
    for (let i = 0; i < visuallyHidden.length; i++) {
        visuallyHidden[i].classList.remove("visually-hidden");
    }
    let operators = stadiumFilters.querySelectorAll(".filter-operator");
    for (let i = 0; i < operators.length; i++) {
        operators[i].dispatchEvent(new Event('change'));
    }
    if (displayChips) {
        let allMultiSelects = stadiumFilters.querySelectorAll(".check-box-list-container");
        for (let i = 0; i < allMultiSelects.length; i++) {
            setHeader(allMultiSelects[i]);
        }
    }
}
```

## Types
The types can create one nested type manually or use the import option to generate the type. How the type is used remains the same

### FilterConfig Type Setup (Required)
The completed type should have the structure shown below

![Filters ConfigType Setup](images/TypeSetup.png)

#### Manual Type Creation
1. Add a type called "FilterConfig" to the types collection in the Stadium Application Explorer
2. Add the following properties to the type
   1. type (Any)
   2. name (Any)
   3. column (Any)
   4. display (Any)
   5. data (List)
      1. Item (Any)
   6. format (Any)
   7. operators (List)
      1. Item (Any)

#### Type Import
1. Right-click on the `Types` node in the `Application Explorer`

![](images/TypeImport.png)

2. In the `Import Type` popup
    1. Add "FilterConfig" into the `Name` input field
    2. Copy & paste the JSON below into the main input area

```json
{
	"type": "",
	"name": "",
	"column": "",
    "format": "",
    "display": "",
    "operators": [],
    "data": []
}
```

![](images/TypeImportPopup.png)

### SelectedFilters Type Setup (Optional)
To apply filters programatically, create a second type called "SelectedFilters"

#### Manual Type Creation
1. Add a type called "SelectedFilters" to the types collection in the Stadium Application Explorer
1. Add the following properties to the type
   1. column (Any)
   2. selectedoperator (Any)
   3. selectedvalues (List)
      1. Item (Any)

![Applied Filters Type](images/AppliedFiltersType.png)

#### Type Import
1. Right-click on the `Types` node in the `Application Explorer`
2. In the `Import Type` popup
    1. Add "SelectedFilters" into the `Name` input field
    2. Copy & paste the JSON below into the main input area

```json
{
    "column":"",
    "selectedoperator":"",
    "selectedvalues":[]
}
```

## Page
1. Drag a *Container* control to the page 
2. Assign a class of your choosing to the control (e.g. filter-container)
3. Drag a *Button* control to the page 
   1. Add the text "Apply Filters"
   2. Create the button *Click Event Handler*
4. Drag a *Button* control to the page
   1. Add the text "Clear Filters"
   2. Create the button *Click Event Handler*
5. Add a DataGrid or all controls necessary to generate a [Client-Side Repeater DataGrid](https://github.com/stadium-software/repeater-datagrid-client-side) on the page
6. Add a *Label* control to the page
   1. Set the Label *Visible* property to "false"

![Page Controls](images/PageControls.png)

## Event Handlers

### Page.Load
Generating the filtergrid, populating a DataGrid or Repeater with data and storing the full dataset in a label for use later. 

1. Execute your connector to fetch the data
2. Drag a *SetValue* action to the event handler
   1. Target: The *Label.Text* property
   2. Source: The data returned by the connector
3. Assign the data returned by the connector to the DataGrid or Repeater
4. Drag a *List* action and assign the *FilterConfig* type to the *List*
5. Define the fields to be generated in the filtergrid
   1. *type*: the data type of the column you wish to enable filtering for
      1. text
      2. date
      3. number
      4. boolean (by default a radiobuttonlist in "forms" display, but can be set to dropdown, always a dropdown in "chips" display)
      5. enum (by default a dropdown, can optionally be displayed as a radiobuttonlist when not using "chips" display)
      6. multiselect (checkboxlist)
   2. *name*: the label displayed for the filter
   3. *column*: the column name to which the filter must be applied (as specified in the DataGrid "Column" property)
![](images/ColumnPropertyName.png)
   4. *display*
      1. For *boolean* types: Shown as radiobuttonlists by default, but passing the value "dropdown" in this property will cause them to be shown as a dropdown instead
      2. For *enum* types: These are shown as dropdowns by default, but passing the value "radio" in this property will cause them to be shown as a radio button list instead
      3. For *date* type (optional; default is false): Add "picker" to display a browser-provided [HTML5 Date Picker](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/date) in the date input fields. 
   5. *data*: filters of type *enum* and *multiselect* require a list of data users can select from
   6. *format* (*date* type only; default 'YYYY/MM/DD'): The format in which date filter values are passed to the DataGrid search box. This format must match the format used in the DataGrid date column. The format must also be understood as a date by Lucene or Lucene will treat it as a string search. The module uses [DayJS Formats](https://day.js.org/docs/en/display/format).
   7. *operators* (list; optional; only for types 'text', 'date' and 'number'): A list of operators to show in the operators dropdown. Use when you want to show only a subset of the options below. Allowed operators
      1. text: "Contains", "Does Not Contain", "Equals", "Does Not Equal"
      2. number: "Between", "From-To", "Equals", "Greater than", "Smaller than"
      3. date: "Between", "From-To", "Equals", "Greater than", "Smaller than"

Fields Definition Example
```json
[{
	"type": "text",
	"name": "First Name",
	"column": "FirstName",
    "operators": ["Contains", "Does Not Contain"]
},{
    "type": "text",
    "name": "Last Name",
    "column": "LastName"
},{
	"type": "date",
	"name": "Start Date",
	"column": "StartDate",
    "display": "picker",
    "operators": ["Between", "From-To", "Equals"]
},{
	"type": "date",
	"name": "End Date",
	"column": "EndDate",
    "format": "YYYY-MM-DD",
    "operators": ["Greater than", "Smaller than"]
},{
	"type": "number",
	"name": "Number Of Pets",
	"column": "NoOfPets",
    "operators": ["Between", "From-To", "Equals"]
},{
	"type": "boolean",
	"name": "Healthy",
	"column": "Healthy"
},{
	"type": "boolean",
	"name": "Happy",
	"column": "Happy",
	"display": "dropdown"
},{
	"type": "enum",
	"name": "Number of Children",
	"column": "NoOfChildren",
	"data": [0,1,2,3,4,5,6,7,8,9,10]
},{
	"type": "multiselect",
	"name": "Subscription",
	"column": "Subscription",
	"data": ["No data","Subscribed","Unsubscribed"]
}]
``` 
6. Drag the "GenerateFilters" global script to the event handler and provide parameter values
   1. FilterConfig: The *FilterConfig* List defining the filter fields
   2. FilterContainerClass: The classname assigned to the filter *Container* (e.g. filter-container)
   3. Display: Optionally set to "chips" to display the filtergrid as a chips display (default is form)

### Apply Button Click
Applying filters to the full dataset and populating a DataGrid or Repeater with a reduced dataset

1. Drag a *Variable* into the event handler
2. Assign the *Label.Text* property to the Variable *Value* property
3. Drag a *List* of type "Any" into the event handler
4. Assign the *Variable* property to the List *Value* property

![](images/FilterApplyListInput.png)

5. Drag the "ApplyFilters" global script to the event handler and complete the input parameters
   1. FilterContainerClass: The classname assigned to the filter *Container* (e.g. filter-container)
   2. Data: The *List* containing the data (don't assign the *Label.text* directly)
6. The "ApplyFilters" global script outputs an objet with two properties
   1. data: A List containing the filtered dataset
   2. filters: The List of filters that were applied (this can be savedfor later)

ApplyFilters Example Output
```json
{
   "data": [
        {"ID":3,"FirstName":"Wayne","LastName":"Andrews","NoOfChildren":5,"NoOfPets":6,"StartDate":"2022-12-18T00:00:00","EndDate":"2024-03-25T08:00:00","Healthy":true,"Happy":true,"Subscription":"Subscribed"}
   ],
   "filters":[
      {"column":"ID","selectedoperator":"From-To","selectedvalues":["1","20"]},
      {"column":"Healthy","selectedvalues":["true"]},
      {"column":"Happy","selectedvalues":["Yes"]},
      {"column":"Subscription","selectedvalues":["Subscribed"]}
   ]
}
```

7. Drag a *SetValue* action to the event handler
   1. Target: The DataGrid.Data or the Repeater.List property
   2. Source: Select the ApplyFilters.Data output from the ApplyFilters script and add .data to the output to get only the List containing the filtered dataset (~.ApplyFilters.Data.data)

![Apply Button Event Handler](images/ApplyEventHandler.png)

### Clear Filters Button Click
Clearing all user-set values from a filtergrid

1. Drag the "ClearFilters" script to the event handler and provide a parameter value
   1. FilterContainerClass: The classname assigned to the filter *Container* (e.g. filter-container)
2. Drag a *Variable* into the event handler
3. Assign the *Label.Text* property to the Variable *Value* property
4. Drag a *List* into the event handler
5. Assign the *Variable* property to the List *Value* property
6. Drag a *SetValue* action to the event handler
   1. Target: The DataGrid.Data or the Repeater.List property
   2. Source: The List containing the full dataset

![Clear Button Event Handler](images/ClearEventHandler.png)

## Saved Filters
Saving user-defined filter criteria and applying it to a filtergrid at a later time

### Saving Filters
1. The "ApplyFilters" global script returns an object containing a List of applied filters (*~.ApplyFilters.Data.filters*)
2. Store this list temporarily in a hidden label on the page or in a session variable, or more permanently in a cookie or database

ApplyFilters Example Output
```json
{
   "data": [
         {"ID":3,"FirstName":"Wayne","LastName":"Andrews","NoOfChildren":5,"NoOfPets":6,"StartDate":"2022-12-18T00:00:00","EndDate":"2024-03-25T08:00:00","Healthy":true,"Happy":true,"Subscription":"Subscribed"}
   ],
   "filters":[
      {"column":"ID","selectedoperator":"From-To","selectedvalues":["1","20"]},
      {"column":"Healthy","selectedvalues":["true"]},
      {"column":"Happy","selectedvalues":["Yes"]},
      {"column":"Subscription","selectedvalues":["Subscribed"]}
   ]
}
```

### Applying Saved Filters
1. Drag the "SetFilters" global script to an event handler or script and provide parameter values
   1. FilterContainerClass: The classname assigned to the filter *Container* (e.g. filter-container)
   2. SelectedFilters: Provide a List containing saved filters ([see saved filters](#saving-filters))
2. Drag the "ApplyFilters" global script to the event handler and complete the input parameters
   1. FilterContainerClass: The classname assigned to the filter *Container* (e.g. filter-container)
   2. Data: A *List* containing the *full* dataset
3. Drag a *SetValue* to the event handler or script
   1. Target: The DataGrid.Data or the Repeater.List property
   2. Source: Select the ApplyFilters.Data output from the ApplyFilters script and add .data to the output to get only the List containing the filtered dataset (*~.ApplyFilters.Data.data*)

![Apply Saved Filters](images/ApplySaved.png)

# Display Options
1. Allowing users to collaps and expand filtergrids can be achieved by using the [Collapse Controls](https://github.com/stadium-software/collapse-controls) module
2. The filtergrid can be displayed as a chips display by setting the *Display* property to "chips" in the *GenerateFilters* global script

![Collapsible Controls](images/CollapsibleControls.png)

## Integrated Button Display
When using "chips" display, action buttons (Apply & Clear) can be shown next to the filters as shown below. 

![](images/ChipsButtons.png)

To achieve this, place the two button controls into the FilterGrid container control. 

![](images/ButtonControls.png)

# CSS
The CSS below is required for the correct functioning of the module. Variables exposed in the [*filter-grid-variables.css*](filter-grid-variables.css) file can be [customised](#customising-css).

## Before v6.12
1. Create a folder called "CSS" inside of your Embedded Files in your application
2. Drag the two CSS files from this repo [*filter-grid-variables.css*](filter-grid-variables.css) and [*filter-grid.css*](filter-grid.css) into that folder
3. Paste the link tags below into the *head* property of your application
```html
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/filter-grid-variables.css">
``` 

## Customising CSS
1. Open the CSS file called [*filter-grid-variables.css*](filter-grid-variables.css) from this repo
2. Adjust the variables in the *:root* element as you see fit
3. Stadium 6.12+ users can comment out any variable they do **not** want to customise
4. Add the [*filter-grid-variables.css*](filter-grid-variables.css) to the "CSS" folder in the EmbeddedFiles (overwrite)
5. Paste the link tag below into the *head* property of your application (if you don't already have it there)
```html
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/filter-grid-variables.css">
``` 
6. Add the file to the "CSS" inside of your Embedded Files in your application

# Upgrading Stadium Repos
Stadium Repos are not static. They change as additional features are added and bugs are fixed. Using the right method to work with Stadium Repos allows for upgrading them in a controlled manner. 

How to use and update application repos is described here: [Working with Stadium Repos](https://github.com/stadium-software/samples-upgrading)
