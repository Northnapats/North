<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meditech - PS Dashboard</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    
    <style>
        :root {
            /* โทนสีเขียว (Emerald Green) */
            --primary: #059669; 
            --secondary: #10B981;
            --bg: #ECFDF5;
            --card: #FFFFFF;
            --text: #064E3B;
            --border: #A7F3D0;
            --success: #047857;
            --warning: #F59E0B;
            --danger: #DC2626;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg); color: var(--text);
            margin: 0; padding: 15px; box-sizing: border-box;
        }
        
        /* Alert Banner */
        .alert-banner {
            background-color: #FEF2F2; border-left: 5px solid var(--danger);
            color: #991B1B; padding: 12px 20px; border-radius: 8px; margin-bottom: 15px;
            display: none; align-items: center; justify-content: space-between;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); font-weight: 600; font-size: 0.95rem;
        }

        .dashboard {
            display: grid; grid-template-columns: 320px 1fr 400px; gap: 15px;
            max-width: 1700px; margin: 0 auto; height: 92vh;
        }
        .panel {
            background: var(--card); border-radius: 12px; padding: 20px;
            box-shadow: 0 4px 6px -1px rgba(5, 150, 105, 0.1); overflow-y: auto;
            display: flex; flex-direction: column;
        }
        
        /* Logo Section */
        .company-logo {
            text-align: center;
            margin-bottom: 15px;
            padding-bottom: 15px;
            border-bottom: 2px dashed var(--border);
        }
        .company-logo img {
            max-width: 170px; /* ปรับขนาดโลโก้ให้พอดี */
            height: auto;
        }

        h2 { margin-top: 0; color: var(--primary); font-size: 1.15rem; border-bottom: 2px solid var(--border); padding-bottom: 10px; display: flex; justify-content: space-between; }
        
        /* Form */
        .form-group { margin-bottom: 12px; }
        label { display: block; margin-bottom: 4px; font-weight: 600; font-size: 0.85rem; color: #065F46; }
        input[type="text"], input[type="date"], input[type="time"], select { 
            width: 100%; padding: 9px; border: 1px solid #6EE7B7; border-radius: 6px; box-sizing: border-box; font-family: inherit; 
        }
        input:focus, select:focus { outline: none; border-color: var(--primary); box-shadow: 0 0 0 2px #D1FAE5; }
        button.btn-add { width: 100%; padding: 11px; background-color: var(--primary); color: white; border: none; border-radius: 6px; font-weight: bold; cursor: pointer; transition: 0.2s; }
        button.btn-add:hover { background-color: #047857; }
        
        /* Calendar */
        .calendar-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
        .calendar-header button { background: white; border: 1px solid #6EE7B7; padding: 4px 8px; border-radius: 4px; cursor: pointer; color: var(--primary); font-weight: bold; }
        .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 6px; }
        .day-name { text-align: center; font-weight: bold; color: #065F46; font-size: 0.8rem; }
        .calendar-day { background: #F8FAFC; border: 1px solid #D1FAE5; border-radius: 6px; min-height: 65px; padding: 6px; cursor: pointer; position: relative; }
        .calendar-day.active { border: 2px solid var(--primary); background: #D1FAE5; }
        .calendar-day.empty { background: transparent; border: none; cursor: default; }
        .date-num { font-weight: bold; color: #064E3B; font-size: 0.85rem; }
        .task-dot { width: 7px; height: 7px; background-color: var(--primary); border-radius: 50%; display: inline-block; margin-top: 4px; }
        .task-dot.overdue { background-color: var(--danger); }

        /* Tasks & Follow-ups */
        .task-section-title { font-size: 1rem; color: var(--danger); margin-top: 0; margin-bottom: 10px; display: flex; align-items: center; gap: 5px;}
        .task-item { border: 1px solid #D1FAE5; border-radius: 8px; padding: 12px; margin-bottom: 10px; display: flex; flex-direction: column; gap: 8px; background: #FFFFFF; }
        .task-item.completed { border-left: 4px solid var(--success); opacity: 0.7; background: #F8FAFC; }
        .task-item.overdue { border-left: 4px solid var(--danger); background: #FEF2F2; }
        .task-header { display: flex; justify-content: space-between; align-items: center; }
        .task-time { font-weight: bold; color: var(--primary); background: #D1FAE5; padding: 2px 6px; border-radius: 4px; font-size: 0.8rem; }
        .task-item.overdue .task-time { color: var(--danger); background: #FEE2E2; }
        .task-detail { font-size: 0.95rem; font-weight: 600; }
        .task-location { font-size: 0.85rem; color: #065F46; display: flex; align-items: center; gap: 5px; }
        .task-actions button { padding: 4px 8px; border: none; border-radius: 4px; cursor: pointer; font-size: 0.8rem; font-weight: bold; }
        .btn-toggle { background: var(--success); color: white; margin-right: 5px;}
        .btn-delete { background: #94A3B8; color: white; }

        /* Follow-up Checklist */
        .followup-box { background: #FFFFFF; border: 2px solid var(--primary); border-radius: 8px; padding: 15px; margin-top: 20px; }
        .followup-item { display: flex; justify-content: space-between; align-items: center; padding: 8px; border-bottom: 1px solid #D1FAE5; }
        .followup-item:last-child { border-bottom: none; }
        .followup-item.done span { text-decoration: line-through; color: #9CA3AF; }

        /* Stats & Map */
        .map-tooltip { background: rgba(6, 78, 59, 0.9) !important; color: white !important; border: none !important; border-radius: 6px !important; font-size: 11px; text-align: left; line-height: 1.4; }
        .stats-table { width: 100%; border-collapse: collapse; font-size: 0.85rem; margin-top: 5px; }
        .stats-table th { background: #D1FAE5; text-align: left; padding: 8px; border-bottom: 2px solid #6EE7B7; color: var(--primary); }
        .stats-table td { padding: 8px; border-bottom: 1px solid #D1FAE5; vertical-align: top; }
        .hosp-list { font-size: 0.8rem; color: #065F46; display: block; margin-top: 4px; }

        @media (max-width: 1200px) { .dashboard { grid-template-columns: 300px 1fr; height: auto; } .map-panel { grid-column: span 2; } }
        @media (max-width: 768px) { .dashboard { grid-template-columns: 1fr; } .map-panel { grid-column: span 1; } }
    </style>
</head>
<body>

<div id="alertBanner" class="alert-banner">
    <span id="alertMessage">🔔 คุณมีงานค้างที่ยังไม่สำเร็จ!</span>
    <button onclick="document.getElementById('alertBanner').style.display='none'" style="background:none; border:none; cursor:pointer; font-weight:bold; color:#991B1B;">✕</button>
</div>

<div class="dashboard">
    <div class="panel">
        <div class="company-logo">
            <img src="logo.png" alt="Meditech Trading Co.,Ltd Logo">
        </div>

        <h2 style="font-size: 1.05rem;">📝 บันทึกตารางงาน Product Specialist</h2>
        <div class="form-group">
            <label>วันที่ปฏิบัติงาน</label>
            <input type="date" id="taskDate" required>
        </div>
        <div class="form-group">
            <label>เวลานัดหมาย</label>
            <input type="time" id="taskTime" required>
        </div>
        <div class="form-group">
            <label>รายละเอียด (เช่น สาธิต, ซ่อมบำรุง, ส่งมอบ)</label>
            <input type="text" id="taskDetail" placeholder="เช่น บำรุงรักษาและสอนใช้งานเปล Ferno" required>
        </div>
        <div class="form-group">
            <label>จังหวัด</label>
            <select id="taskProvince" required>
                <option value="">-- เลือกจังหวัด --</option>
            </select>
        </div>
        <div class="form-group">
            <label>สถานที่ / โรงพยาบาล / แผนก</label>
            <input type="text" id="taskLocation" placeholder="เช่น รพ.ศูนย์ขอนแก่น แผนก ER" required>
        </div>
        <button class="btn-add" onclick="addTask()">บันทึกลงตารางงาน</button>
    </div>

    <div class="panel" style="gap: 10px;">
        <div>
            <div class="calendar-header">
                <h2 id="monthYearText" style="border: none; margin: 0; font-size: 1.1rem;"></h2>
                <div>
                    <button onclick="changeMonth(-1)">◀</button>
                    <button onclick="changeMonth(1)">▶</button>
                </div>
            </div>
            <div class="calendar-grid" id="calendarDays"></div>
        </div>

        <div id="pendingTasksContainer" style="display: none; margin-top: 10px;">
            <h3 class="task-section-title">⚠️ งานค้างดำเนินการ (Overdue)</h3>
            <div id="pendingTaskList"></div>
        </div>

        <div style="flex: 1; margin-top: 10px;">
            <h2 id="selectedDateHeader" style="font-size: 1.1rem; color: #064E3B;">ประจำวันที่:</h2>
            <div id="taskList"></div>
        </div>

        <div class="followup-box">
            <h2 style="border: none; padding: 0; font-size: 1rem;">📌 สิ่งที่ต้องตามหลังจากพบลูกค้า</h2>
            <div style="display: flex; gap: 8px; margin-bottom: 10px;">
                <input type="text" id="followUpInput" placeholder="เช่น ส่งใบเสนอราคาให้แผนก ER รพ.ขอนแก่น" style="flex: 1;">
                <button class="btn-add" style="width: auto; padding: 8px 15px;" onclick="addFollowUp()">เพิ่ม</button>
            </div>
            <div id="followUpList" style="display: flex; flex-direction: column;">
                </div>
        </div>
    </div>

    <div class="panel map-panel">
        <h2>🗺️ ประวัติการเข้าพบลูกค้า</h2>
        <div id="map" style="height: 350px; border-radius: 8px; border: 1px solid #6EE7B7; margin-bottom: 10px; z-index: 1;"></div>
        
        <h2>🏥 รายชื่อสถานที่แยกตามจังหวัด</h2>
        <div style="overflow-y: auto; flex: 1;">
            <table class="stats-table">
                <thead>
                    <tr><th width="35%">จังหวัด</th><th>รายชื่อสถานที่ / รพ.</th></tr>
                </thead>
                <tbody id="statsTableBody">
                    <tr><td colspan="2" style="text-align:center; color:#94A3B8;">ไม่มีข้อมูลปฏิบัติงาน</td></tr>
                </tbody>
            </table>
        </div>
    </div>
</div>

<script>
    const provinceCoords = { "กรุงเทพมหานคร": [13.7563, 100.5018], "กระบี่": [8.0863, 98.9063], "กาญจนบุรี": [14.0228, 99.5328], "กาฬสินธุ์": [16.4322, 103.5065], "กำแพงเพชร": [16.4828, 99.5228], "ขอนแก่น": [16.4322, 102.8236], "จันทบุรี": [12.6112, 102.1038], "ฉะเชิงเทรา": [13.6889, 101.0753], "ชลบุรี": [13.3611, 100.9847], "ชัยนาท": [15.1856, 100.1250], "ชัยภูมิ": [15.8064, 102.0317], "ชุมพร": [10.4931, 99.1800], "เชียงราย": [19.9086, 99.8325], "เชียงใหม่": [18.7883, 98.9853], "ตรัง": [7.5564, 99.6114], "ตราด": [12.2428, 102.5175], "ตาก": [16.8839, 99.1258], "นครนายก": [14.2069, 101.2131], "นครปฐม": [13.8194, 100.0603], "นครพนม": [17.4078, 104.7811], "นครราชสีมา": [14.9799, 102.0978], "นครศรีธรรมราช": [8.4333, 99.9667], "นครสวรรค์": [15.7024, 100.1372], "นนทบุรี": [13.8621, 100.5144], "นราธิวาส": [6.4256, 101.8233], "น่าน": [18.7831, 100.7758], "บึงกาฬ": [18.3614, 103.6531], "บุรีรัมย์": [14.9951, 103.1022], "ปทุมธานี": [14.0208, 100.5250], "ประจวบคีรีขันธ์": [11.8124, 99.7972], "ปราจีนบุรี": [14.0506, 101.3725], "ปัตตานี": [6.8675, 101.2500], "พระนครศรีอยุธยา": [14.3569, 100.5775], "พะเยา": [19.1658, 99.9025], "พังงา": [8.4411, 98.5256], "พัทลุง": [7.6167, 100.0833], "พิจิตร": [16.4419, 100.3489], "พิษณุโลก": [16.8211, 100.2659], "เพชรบุรี": [13.1119, 99.9439], "เพชรบูรณ์": [16.4194, 101.1564], "แพร่": [18.1442, 100.1411], "ภูเก็ต": [7.8804, 98.3923], "มหาสารคาม": [16.1844, 103.3006], "มุกดาหาร": [16.5447, 104.7236], "แม่ฮ่องสอน": [19.3003, 97.9681], "ยโสธร": [15.7958, 104.1453], "ยะลา": [6.5397, 101.2814], "ร้อยเอ็ด": [16.0539, 103.6525], "ระนอง": [9.9658, 98.6347], "ระยอง": [12.6815, 101.2813], "ราชบุรี": [13.5283, 99.8131], "ลพบุรี": [14.7997, 100.6533], "ลำปาง": [18.2858, 99.4925], "ลำพูน": [18.5772, 99.0083], "เลย": [17.4894, 101.7225], "ศรีสะเกษ": [15.1153, 104.3294], "สกลนคร": [17.1664, 104.1486], "สงขลา": [7.1898, 100.5954], "สตูล": [6.6233, 100.0672], "สมุทรปราการ": [13.5991, 100.5968], "สมุทรสงคราม": [13.4092, 100.0025], "สมุทรสาคร": [13.5475, 100.2744], "สระแก้ว": [13.8242, 102.0647], "สระบุรี": [14.5289, 100.9108], "สิงห์บุรี": [14.8936, 100.3967], "สุโขทัย": [17.0039, 99.8264], "สุพรรณบุรี": [14.4742, 100.1175], "สุราษฎร์ธานี": [9.1342, 99.3334], "สุรินทร์": [14.8824, 103.4936], "หนองคาย": [17.8781, 102.7422], "หนองบัวลำภู": [17.2033, 102.4403], "อ่างทอง": [14.5894, 100.4550], "อำนาจเจริญ": [15.8547, 104.6258], "อุดรธานี": [17.4139, 102.7856], "อุตรดิตถ์": [17.6258, 100.0992], "อุทัยธานี": [15.3783, 100.0247], "อุบลราชธานี": [15.2287, 104.8564] };

    let tasks = JSON.parse(localStorage.getItem('psAppV3_Tasks')) || [];
    let followUps = JSON.parse(localStorage.getItem('psAppV3_FollowUps')) || [];
    let currentDate = new Date();
    
    const getTodayStr = () => {
        const tzOffset = (new Date()).getTimezoneOffset() * 60000;
        return (new Date(Date.now() - tzOffset)).toISOString().split('T')[0];
    };
    
    let selectedDateStr = getTodayStr();
    let map, markerLayer;

    document.getElementById('taskDate').value = selectedDateStr;
    const provinceSelect = document.getElementById('taskProvince');
    Object.keys(provinceCoords).sort().forEach(prov => {
        const opt = document.createElement('option'); opt.value = prov; opt.textContent = prov; provinceSelect.appendChild(opt);
    });

    function initMap() {
        map = L.map('map').setView([13.2000, 101.0000], 5);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '© OpenStreetMap' }).addTo(map);
        markerLayer = L.layerGroup().addTo(map);
    }

    function saveTasks() {
        localStorage.setItem('psAppV3_Tasks', JSON.stringify(tasks));
        checkNotifications();
        renderCalendar();
        renderTaskList();
        updateMapAndStats();
    }

    function checkNotifications() {
        const today = getTodayStr();
        const overdueTasks = tasks.filter(t => t.date < today && !t.completed);
        const banner = document.getElementById('alertBanner');
        
        if (overdueTasks.length > 0) {
            document.getElementById('alertMessage').innerHTML = `⚠️ <b>การแจ้งเตือน:</b> คุณมีคิวงานค้างดำเนินการจำนวน <b>${overdueTasks.length}</b> งาน!`;
            banner.style.display = 'flex';
        } else {
            banner.style.display = 'none';
        }
    }

    function renderCalendar() {
        const year = currentDate.getFullYear(); const month = currentDate.getMonth();
        const monthNames = ["มกราคม", "กุมภาพันธ์", "มีนาคม", "เมษายน", "พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม", "พฤศจิกายน", "ธันวาคม"];
        document.getElementById('monthYearText').textContent = `${monthNames[month]} ${year}`;

        const firstDay = new Date(year, month, 1).getDay();
        const daysInMonth = new Date(year, month + 1, 0).getDate();
        const calendarGrid = document.getElementById('calendarDays');
        const todayStr = getTodayStr();
        
        calendarGrid.innerHTML = `<div class="day-name">อา</div><div class="day-name">จ</div><div class="day-name">อ</div><div class="day-name">พ</div><div class="day-name">พฤ</div><div class="day-name">ศ</div><div class="day-name">ส</div>`;

        for (let i = 0; i < firstDay; i++) calendarGrid.innerHTML += `<div class="calendar-day empty"></div>`;

        for (let day = 1; day <= daysInMonth; day++) {
            const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
            const tasksOnThisDay = tasks.filter(t => t.date === dateStr);
            
            let dotsHtml = '';
            tasksOnThisDay.forEach(t => {
                const isOverdue = t.date < todayStr && !t.completed;
                dotsHtml += `<div class="task-dot ${isOverdue ? 'overdue' : ''}"></div> `;
            });

            const isActive = dateStr === selectedDateStr ? 'active' : '';
            calendarGrid.innerHTML += `
                <div class="calendar-day ${isActive}" onclick="selectDate('${dateStr}')">
                    <div class="date-num">${day}</div>
                    <div style="display:flex; gap:2px; flex-wrap:wrap; margin-top:2px;">${dotsHtml}</div>
                </div>`;
        }
    }

    function changeMonth(step) { currentDate.setMonth(currentDate.getMonth() + step); renderCalendar(); }
    function selectDate(dateStr) { selectedDateStr = dateStr; document.getElementById('taskDate').value = dateStr; renderCalendar(); renderTaskList(); }

    function addTask() {
        const date = document.getElementById('taskDate').value; const time = document.getElementById('taskTime').value;
        const detail = document.getElementById('taskDetail').value.trim(); const province = document.getElementById('taskProvince').value;
        const location = document.getElementById('taskLocation').value.trim();

        if (date && time && detail && province && location) {
            tasks.push({ id: Date.now(), date: date, time: time, detail: detail, province: province, location: location, completed: false });
            
            document.getElementById('taskTime').value = ''; document.getElementById('taskDetail').value = '';
            document.getElementById('taskProvince').value = ''; document.getElementById('taskLocation').value = '';
            
            selectedDateStr = date; saveTasks();
        } else { alert('กรุณากรอกข้อมูลให้ครบถ้วน'); }
    }

    function generateTaskHTML(task, isOverdue = false) {
        return `
            <div class="task-item ${task.completed ? 'completed' : ''} ${isOverdue ? 'overdue' : ''}">
                <div class="task-header">
                    <span class="task-time">${isOverdue ? '⚠️ ค้างตั้งแต่: ' + task.date : '⏰ ' + task.time + ' น.'}</span>
                    <div class="task-actions">
                        <button class="btn-toggle" onclick="toggleTask(${task.id})">${task.completed ? '❌ ยกเลิก' : '✅ สำเร็จ'}</button>
                        <button class="btn-delete" onclick="deleteTask(${task.id})">ลบ</button>
                    </div>
                </div>
                <div class="task-detail">📋 [${task.province}] ${task.detail}</div>
                <div class="task-location">📍 ${task.location}</div>
            </div>`;
    }

    function renderTaskList() {
        const todayStr = getTodayStr();
        const overdueTasks = tasks.filter(t => t.date < todayStr && !t.completed).sort((a, b) => a.date.localeCompare(b.date));
        const pendingContainer = document.getElementById('pendingTasksContainer');
        const pendingList = document.getElementById('pendingTaskList');
        
        if (overdueTasks.length > 0) {
            pendingList.innerHTML = overdueTasks.map(t => generateTaskHTML(t, true)).join('');
            pendingContainer.style.display = 'block';
        } else {
            pendingContainer.style.display = 'none';
        }

        const displayDate = new Date(selectedDateStr).toLocaleDateString('th-TH', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
        document.getElementById('selectedDateHeader').textContent = `📅 ประจำวันที่: ${displayDate}`;
        const taskList = document.getElementById('taskList');
        
        const filteredTasks = tasks.filter(t => t.date === selectedDateStr).sort((a, b) => a.time.localeCompare(b.time));
        
        if (filteredTasks.length === 0) {
            taskList.innerHTML = '<p style="color: #94A3B8; text-align: center; margin-top: 20px;">-- ไม่มีงานในวันนี้ --</p>';
        } else {
            taskList.innerHTML = filteredTasks.map(t => generateTaskHTML(t, false)).join('');
        }
    }

    function updateMapAndStats() {
        if (!markerLayer) return; markerLayer.clearLayers();

        const provData = {};
        tasks.forEach(t => {
            if (t.province) {
                if (!provData[t.province]) provData[t.province] = { count: 0, hospitals: new Set() };
                provData[t.province].count += 1;
                provData[t.province].hospitals.add(t.location);
            }
        });

        const statsTableBody = document.getElementById('statsTableBody');
        statsTableBody.innerHTML = '';
        const sortedProvinces = Object.entries(provData).sort((a, b) => b[1].count - a[1].count);

        if (sortedProvinces.length === 0) {
            statsTableBody.innerHTML = '<tr><td colspan="2" style="text-align:center; color:#94A3B8;">ยังไม่มีประวัติการเดินทาง</td></tr>';
        } else {
            sortedProvinces.forEach(([province, data]) => {
                const hospArray = Array.from(data.hospitals);
                const hospHtmlList = hospArray.map(h => `• ${h}`).join('<br>');
                
                statsTableBody.innerHTML += `
                    <tr>
                        <td><b>📍 ${province}</b><br><span style="font-size:0.75rem; color:var(--primary);">(${data.count} งาน)</span></td>
                        <td><span class="hosp-list">${hospHtmlList}</span></td>
                    </tr>`;
                
                const coords = provinceCoords[province];
                if (coords) {
                    const radius = 8 + (data.count * 2); 
                    const circle = L.circleMarker(coords, { radius: radius, fillColor: '#059669', color: '#FFFFFF', weight: 2, fillOpacity: 0.85 }).addTo(markerLayer);
                    circle.bindTooltip(`<b>${province}</b> (${data.count} งาน)<br><span style="font-size:10px; color:#A7F3D0;">${hospArray.join(', ')}</span>`, {
                        className: 'map-tooltip'
                    });
                }
            });
        }
    }

    function toggleTask(id) { tasks = tasks.map(t => t.id === id ? { ...t, completed: !t.completed } : t); saveTasks(); }
    function deleteTask(id) { if(confirm('ต้องการลบข้อมูลนี้ใช่หรือไม่?')) { tasks = tasks.filter(t => t.id !== id); saveTasks(); } }

    /* ================= Follow-up Functions ================= */
    function saveFollowUps() {
        localStorage.setItem('psAppV3_FollowUps', JSON.stringify(followUps));
        renderFollowUps();
    }

    function addFollowUp() {
        const input = document.getElementById('followUpInput');
        const text = input.value.trim();
        if (text) {
            followUps.push({ id: Date.now(), text: text, completed: false });
            input.value = '';
            saveFollowUps();
        }
    }

    document.getElementById('followUpInput').addEventListener('keypress', function (e) {
        if (e.key === 'Enter') addFollowUp();
    });

    function toggleFollowUp(id) {
        followUps = followUps.map(f => f.id === id ? { ...f, completed: !f.completed } : f);
        saveFollowUps();
    }

    function deleteFollowUp(id) {
        if(confirm('ลบรายการติดตามนี้?')) {
            followUps = followUps.filter(f => f.id !== id);
            saveFollowUps();
        }
    }

    function renderFollowUps() {
        const list = document.getElementById('followUpList');
        list.innerHTML = '';
        if (followUps.length === 0) {
            list.innerHTML = '<p style="color: #9CA3AF; font-size: 0.85rem; text-align: center; margin-top: 10px;">ยังไม่มีรายการสิ่งที่ต้องติดตาม</p>';
            return;
        }
        
        const sortedFollowUps = [...followUps].sort((a, b) => a.completed - b.completed);
        
        sortedFollowUps.forEach(f => {
            list.innerHTML += `
                <div class="followup-item ${f.completed ? 'done' : ''}">
                    <div style="display: flex; align-items: center; gap: 10px;">
                        <input type="checkbox" ${f.completed ? 'checked' : ''} onchange="toggleFollowUp(${f.id})" style="width: 16px; height: 16px; accent-color: var(--primary); cursor: pointer;">
                        <span style="font-size: 0.9rem;">${f.text}</span>
                    </div>
                    <button onclick="deleteFollowUp(${f.id})" style="background: none; border: none; color: var(--danger); cursor: pointer; font-size: 0.8rem;">ลบ</button>
                </div>
            `;
        });
    }

    initMap(); 
    saveTasks();
    renderFollowUps();
</script>

</body>
</html>
