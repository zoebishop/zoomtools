<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>2107 Seattle Academy Zoom Tools</title>
    <style>
    .center {
        width: 50%;
        margin: auto;
        display: block;
        font-family: "Times New Roman", Times, serif;
        font-size: 20px;
        margin-bottom: 20px;
        background-color: rgb(33,96,120);
        color: rgb(215,236,244);
    }

    .student {
        text-align:center;
        border: solid;
        border-width: 3px;
        background-color: rgb(33,96,120);
        color: rgb(215,236,244);
        margin: auto;
        width: 40%;
    }
    
    .picked {
        background: violet;
    }
    </style>
</head>

<body>
    <select id="mySelect" onchange="classChange()">
        <option value="block1">Block 1
        <option value="block5">Block 5
    </select>
    <button id="updateButton" class='center'>
        Update Button
    </button>
    <button id="pickStudent" class='center'>
        Pick Student
    </button>
    <div id="container">
        ...
    </div>
    <script>
    console.clear();
    let course = 'block1';
    let port = 1500;
    let students = [];

    function importStudents(theFile) {
        fetch(theFile)
            .then(response => response.json())
            .then(data => {
                students = data;
                renderList();
            });

    }

    importStudents(`https://math.seattleacademy.org/${port}/${course}`);


    function pickStudent() {
        let randomStudentNum = Math.floor(Math.random() * students.length);
        let student = students[randomStudentNum];

        for (let i = 0; i < students.length; i++) {
            if (students[i].picked) {
                students[i].picked = false;
                updateStudent(students[i]);
            }
        }

        students[randomStudentNum].picked = true;
        updateStudent(students[randomStudentNum]);
        renderList();
    }

    let pickButton = document.getElementById("pickStudent");
    pickButton.addEventListener("click", pickStudent);

    function renderList() {
        students.sort(function(a, b) {
            return b.lasttime - a.lasttime;
        });

        // students.sort(function(a,b){return a.lasttime-b.lasttime})

        content = "";
        let theStyle = "";
        for (let i = 0; i < students.length; i++) {
            if (students[i].picked) {
                theStyle = 'class="student picked"';
            } else {
                theStyle = 'class="student"';
            }
            const lastTime = new Date(students[i].lasttime);

            //console.log(lastTime.toLocaleTimeString('en-US'));
            let timeString = lastTime.toLocaleTimeString('en-US')
            content += `<div ${theStyle} id="${students[i].username}">
    ${students[i].clickcount} ${students[i].first} ${students[i].last} ${timeString}
  </div>`;
        }
        let element = document.getElementById("container");
        element.innerHTML = content;
    }

    renderList();

    function listclick(e) {
        for (let i = 0; i < students.length; i++) {
            if (students[i].username == e.target.id) {
                students[i].clickcount++;
                students[i].lasttime = Date.now();
                updateStudent(students[i]);
            }
        }
        renderList();
    }

    let renderclick = document.getElementById("container");
    renderclick.addEventListener("click", listclick);

    function classChange() {
        course = document.getElementById("mySelect").value;
        console.log(course);
        importStudents(`https://math.seattleacademy.org/${port}/${course}`);
    }

    function updateStudent(student) {
        console.log(student);
        const putMethod = {
            method: 'PUT', // Method itself
            headers: {
                'Content-type': 'application/json; charset=UTF-8' // Indicates the content 
            },
            body: JSON.stringify(student) // We send data in JSON format
        }

        // make the HTTP put request using fetch api
        let url = `https://math.seattleacademy.org/${port}/${course}/`
        url += student.username;
        console.log(url);
        fetch(url, putMethod)
            .then(response => response.json())
            .then(data => console.log(data)) // Manipulate the data retrieved back, if we want to do something with it
            .catch(err => console.log(err)) // Do something with the error
    }

    function updateList() {
        console.log('updateList')
        importStudents(`https://math.seattleacademy.org/${port}/${course}`);
    }
    let updateButton = document.getElementById("updateButton");
    updateButton.addEventListener("click", updateList); 
    setInterval(updateList,1500);
    </script>
</body>

</html>
