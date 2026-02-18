


Uploading Untitled 6.mp4…


## 1. Identify Semester IDs
**Run on Results Page:** `https://erp.dypakurdipune.edu.in/stu_downloadStudentExamResult.htm`

This script captures your exam IDs and saves them to browser memory

```javascript
(function findAndStoreIds() {
    const dropdown = document.getElementById('resultExamScheduleCmb');
    if (!dropdown) return console.error("Dropdown not found!");

    let idList = [];
    let idTable = [];

    Array.from(dropdown.options).forEach(option => {
        const id = option.value.split('_')[0];
        const name = option.text.trim();
        if (id && id !== "0" && id !== "") {
            idList.push(id);
            idTable.push({ "Semester Name": name, "Exam ID": id });
        }
    });

    if (idList.length === 0) return console.warn("No IDs found.");

    localStorage.setItem('lastDetectedIds', idList.join(','));

    console.clear();
    console.table(idTable);
    console.log(`✅ IDs (${idList.join(',')}) saved.`);
})();
```


<br><br>
## 2. Fetch Marks

**Run on Course Page:** `https://erp.dypakurdipune.edu.in/studentCourseFileNew.htm`

This script pre-fills the saved IDs and monitors for subject clicks. It stops searching once marks are found for a subject.

```javascript
(function setupCleanSmartFetcher() {
    console.clear();
    let input = "";
    let idList = [];

    const savedIds = localStorage.getItem('lastDetectedIds') || "";

    while (true) {
        input = prompt("Enter Exam IDs separated by commas (ex: 2,43,234):", savedIds);
        if (input === null) return; 
        
        idList = input.split(',').map(id => id.trim()).filter(id => id.length > 0);
        const allNumbers = idList.every(id => !isNaN(id));

        if (idList.length > 0 && allNumbers) {
            break; 
        } else {
            alert("Invalid Input!\nPlease enter at least one numeric ID.");
        }
    }

    console.log(`%c ACTIVE `, "background: #6f42c1; color: white; padding: 5px; font-weight: bold;");

    if (!window.originalOpen) window.originalOpen = XMLHttpRequest.prototype.open;

    XMLHttpRequest.prototype.open = function(method, url) {
        this.addEventListener('load', async function() {
            if (url.includes('stu_fetchEmployeeAllocatedSubjectsDetailsBySubjectId.json')) {
                try {
                    const data = JSON.parse(this.responseText);
                    const subjectName = data.subjectName;
                    const uniSubjectId = data.uniSubjectId;
                    let marksFound = false;

                    for (const lockedId of idList) {
                        const marksUrl = `https://erp.dypakurdipune.edu.in/stu_getExamPaperDetailsCourseWiseForProvisionalForNonBitwise.json?universitySubjectId=${uniSubjectId}&examScheduleId=${lockedId}`;
                        
                        const res = await fetch(marksUrl);
                        const mData = await res.json();
                        
                        if (mData && mData.length > 0 && mData[0].obtainedMarks !== undefined) {
                            console.log(
                                `%c ${subjectName.padEnd(35)} | %c ${mData[0].obtainedMarks} / ${mData[0].totalMarks} `,
                                "color: #fff; background: #222;",
                                "color: #00ff00; font-weight: bold; background: #222;"
                            );
                            marksFound = true;
                            break; 
                        }
                    }

                    if (!marksFound) {
                        console.log(
                            `%c ${subjectName.padEnd(35)} | %c Marks Not Available `,
                            "color: #888; background: #222;",
                            "color: #ff4444; background: #222; font-weight: bold;"
                        );
                    }
                } catch (e) {}
            }
        });
        return window.originalOpen.apply(this, arguments);
    };
})();
```
