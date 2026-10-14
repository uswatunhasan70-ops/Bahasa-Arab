// Ganti dengan URL Web App Apps Script Anda dari langkah Deploy di atas
const GOOGLE_APP_SCRIPT_URL = 'URL_WEB_APP_ANDA_DI_SINI'; 

// 1. Fungsi Mengirim Skor Kuis ke Google Spreadsheet & LocalStorage
function saveStudentRecord(name, cls, score) {
    // A. Simpan di LocalStorage (pilihan lokal)
    let records = JSON.parse(localStorage.getItem('mpi_records_arab') || '[]');
    let existingIdx = records.findIndex(r => r.name.toLowerCase() === name.toLowerCase() && r.cls === cls);
    let dateStr = new Date().toLocaleDateString('id-ID');
    
    if (existingIdx >= 0) {
        if (score > records[existingIdx].score) {
            records[existingIdx].score = score;
            records[existingIdx].date = dateStr;
        }
    } else {
        records.push({ name: name, cls: cls, score: score, date: dateStr });
    }
    localStorage.setItem('mpi_records_arab', JSON.stringify(records));

    // B. Kirim langsung ke Google Spreadsheet via Fetch API
    if (GOOGLE_APP_SCRIPT_URL && GOOGLE_APP_SCRIPT_URL !== 'URL_WEB_APP_ANDA_DI_SINI') {
        fetch(GOOGLE_APP_SCRIPT_URL, {
            method: 'POST',
            mode: 'no-cors', // Menghindari kendala CORS dari GitHub Pages
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                name: name,
                cls: cls,
                score: score
            })
        })
        .then(() => {
            console.log('✅ Data berhasil dikirim ke Google Spreadsheet!');
        })
        .catch(err => {
            console.error('❌ Gagal mengirim ke Google Spreadsheet:', err);
        });
    }
}

// 2. Update Fungsi Panel Guru / Data Murid (Memuat data dari Google Spreadsheet)
async function fetchOnlineScores() {
    if (!GOOGLE_APP_SCRIPT_URL || GOOGLE_APP_SCRIPT_URL === 'URL_WEB_APP_ANDA_DI_SINI') {
        return JSON.parse(localStorage.getItem('mpi_records_arab') || '[]');
    }
    
    try {
        const response = await fetch(GOOGLE_APP_SCRIPT_URL);
        const onlineData = await response.json();
        if (Array.isArray(onlineData) && onlineData.length > 0) {
            return onlineData;
        }
    } catch (e) {
        console.warn('Gagal memuat data online, fallback ke local storage', e);
    }
    return JSON.parse(localStorage.getItem('mpi_records_arab') || '[]');
}
