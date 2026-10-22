<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ClassTrack - Theo dõi bài tập & Quản lý lớp học</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <!-- Babel for JSX -->
    <script crossorigin src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 antialiased">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo } = React;

        // Dữ liệu mẫu ban đầu
        const initialStudents = [
            { id: "HS01", name: "Nguyễn Văn An", gender: "Nam", dob: "2008-05-12", phone: "0901234567", note: "Học lực giỏi, cần phát huy môn Tự nhiên." },
            { id: "HS02", name: "Trần Thị Bình", gender: "Nữ", dob: "2008-08-20", phone: "0902345678", note: "Cần chú ý thêm môn Hóa học." },
            { id: "HS03", name: "Lê Hoàng Cường", gender: "Nam", dob: "2008-01-15", phone: "0903456789", note: "Năng nổ trong các hoạt động lớp." },
            { id: "HS04", name: "Phạm Thị Dung", gender: "Nữ", dob: "2008-11-04", phone: "0904567890", note: "Chăm chỉ, hoàn thành BTVN đầy đủ." },
            { id: "HS05", name: "Vũ Minh Đức", gender: "Nam", dob: "2008-06-30", phone: "0905678901", note: "Đôi khi nộp bài sát hạn." },
            { id: "HS06", name: "Hoàng Thị Hoa", gender: "Nữ", dob: "2008-09-12", phone: "0906789012", note: "Học sinh tiến bộ tích cực." },
            { id: "HS07", name: "Bùi Văn Hùng", gender: "Nam", dob: "2008-03-22", phone: "0907890123", note: "Cần kèm sát môn Tiếng Anh." },
            { id: "HS08", name: "Đỗ Thị Lan", gender: "Nữ", dob: "2008-12-05", phone: "0908901234", note: "Cán lớp năng nổ, trách nhiệm." },
            { id: "HS09", name: "Ngô Quang Linh", gender: "Nam", dob: "2008-07-18", phone: "0909012345", note: "Tư duy tốt, sáng tạo." },
            { id: "HS10", name: "Đinh Thị Mai", gender: "Nữ", dob: "2008-10-25", phone: "0910123456", note: "Ngoan hiền, chú ý nghe giảng." }
        ];

        const initialSubjects = ["Toán", "Ngữ Văn", "Tiếng Anh", "Hóa học"];
        const initialWeeks = ["Tuần 1", "Tuần 2", "Tuần 3", "Tuần 4"];

        const initialHomeworks = [
            { id: "HW1", student: "Nguyễn Văn An", subject: "Hóa học", content: "Giải bài tập Ancol và Phenol (Bài 1-5)", assignDate: "2026-10-01", dueDate: "2026-10-05", progress: "Đã nộp", checkStatus: "Đã kiểm tra", result: "Đạt yêu cầu", score: 8.5 },
            { id: "HW2", student: "Trần Thị Bình", subject: "Toán", content: "Ôn tập số phức và ứng dụng hình học", assignDate: "2026-10-02", dueDate: "2026-10-06", progress: "Đang làm", checkStatus: "Chưa kiểm tra", result: "Chưa đánh giá", score: null },
            { id: "HW3", student: "Lê Hoàng Cường", subject: "Tiếng Anh", content: "Làm bài tập Reading Unit 3 SGK", assignDate: "2026-10-02", dueDate: "2026-10-06", progress: "Chưa làm", checkStatus: "Chưa kiểm tra", result: "Chưa đánh giá", score: null },
            { id: "HW4", student: "Phạm Thị Dung", subject: "Ngữ Văn", content: "Soạn bài Vợ Nhặt (Kim Lân)", assignDate: "2026-10-03", dueDate: "2026-10-07", progress: "Đã hoàn thành", checkStatus: "Cần bổ sung", result: "Chưa đạt yêu cầu", score: 5.0 },
            { id: "HW5", student: "Vũ Minh Đức", subject: "Hóa học", content: "Bài tập Este - Lipit nâng cao", assignDate: "2026-10-03", dueDate: "2026-10-08", progress: "Đã nộp", checkStatus: "Đã kiểm tra", result: "Đạt yêu cầu", score: 9.0 }
        ];

        function App() {
            const [currentUser, setCurrentUser] = useState(null);

            const [students, setStudents] = useState(initialStudents);
            const [homeworks, setHomeworks] = useState(initialHomeworks);
            const [subjects] = useState(initialSubjects);
            const [weeks] = useState(initialWeeks);

            const [activeTab, setActiveTab] = useState('dashboard');
            const [toast, setToast] = useState(null);
            const showToast = (msg) => {
                setToast(msg);
                setTimeout(() => setToast(null), 3000);
            };

            // ---- TRẠNG THÁI ĐĂNG NHẬP VÀ NHẬP TÊN TÙY Ý ---
            const [loginRole, setLoginRole] = useState('gvcn');
            const [inputTeacherName, setInputTeacherName] = useState('Cô Nguyễn Thị Mai');
            const [loginSubject, setLoginSubject] = useState('Hóa học');

            const handleLogin = (e) => {
                e.preventDefault();
                const trimmedName = inputTeacherName.trim() || (loginRole === 'gvcn' ? 'Giáo viên Chủ nhiệm' : 'Giáo viên Bộ môn');
                if (loginRole === 'gvcn') {
                    setCurrentUser({ role: 'gvcn', name: `${trimmedName} (GVCN)` });
                } else {
                    setCurrentUser({ role: 'gvbm', name: `${trimmedName} (${loginSubject})`, subject: loginSubject });
                }
                setActiveTab('dashboard');
                showToast("Đăng nhập thành công!");
            };

            const handleLogout = () => {
                setCurrentUser(null);
                showToast("Đã đăng xuất khỏi hệ thống.");
            };

            if (!currentUser) {
                return (
                    <div className="min-h-screen bg-slate-900 flex items-center justify-center p-4">
                        <div className="bg-white rounded-2xl shadow-2xl max-w-md w-full p-8 border border-slate-100">
                            <div className="text-center mb-6">
                                <div className="inline-flex items-center justify-center w-16 h-16 rounded-full bg-blue-900 text-teal-400 mb-4 shadow-lg">
                                    <i data-lucide="graduation-cap" className="w-8 h-8"></i>
                                </div>
                                <h1 className="text-2xl font-bold text-slate-900">ClassTrack</h1>
                                <p className="text-slate-500 text-sm mt-1">Hệ thống Quản lý học sinh & Bài tập về nhà</p>
                            </div>

                            <form onSubmit={handleLogin} className="space-y-4">
                                <div>
                                    <label className="block text-sm font-semibold text-slate-700 mb-2">Chọn vai trò</label>
                                    <div className="grid grid-cols-2 gap-3">
                                        <button
                                            type="button"
                                            onClick={() => { setLoginRole('gvcn'); setInputTeacherName('Cô Nguyễn Thị Mai'); }}
                                            className={`p-3 rounded-xl border text-sm font-medium transition-all flex items-center justify-center gap-2 ${loginRole === 'gvcn' ? 'border-blue-900 bg-blue-50 text-blue-900 shadow-sm font-bold' : 'border-slate-200 text-slate-600 hover:bg-slate-50'}`}
                                        >
                                            <i data-lucide="user-check" className="w-4 h-4"></i>
                                            GV Chủ nhiệm
                                        </button>
                                        <button
                                            type="button"
                                            onClick={() => { setLoginRole('gvbm'); setInputTeacherName('Thầy Văn Nam'); }}
                                            className={`p-3 rounded-xl border text-sm font-medium transition-all flex items-center justify-center gap-2 ${loginRole === 'gvbm' ? 'border-blue-900 bg-blue-50 text-blue-900 shadow-sm font-bold' : 'border-slate-200 text-slate-600 hover:bg-slate-50'}`}
                                        >
                                            <i data-lucide="book-open" className="w-4 h-4"></i>
                                            GV Bộ môn
                                        </button>
                                    </div>
                                </div>

                                <div>
                                    <label className="block text-sm font-semibold text-slate-700 mb-1">Nhập họ và tên giáo viên</label>
                                    <input 
                                        type="text"
                                        value={inputTeacherName}
                                        onChange={(e) => setInputTeacherName(e.target.value)}
                                        placeholder="VD: Thầy Nguyễn Văn A / Cô Trần Thị B"
                                        className="w-full px-4 py-2.5 rounded-xl border border-slate-300 text-sm focus:ring-2 focus:ring-blue-900 focus:outline-none"
                                        required
                                    />
                                </div>

                                {loginRole === 'gvbm' && (
                                    <div className="animate-fadeIn">
                                        <label className="block text-sm font-semibold text-slate-700 mb-1">Chọn môn phụ trách</label>
                                        <select 
                                            value={loginSubject}
                                            onChange={(e) => setLoginSubject(e.target.value)}
                                            className="w-full px-4 py-2.5 rounded-xl border border-slate-300 text-sm focus:ring-2 focus:ring-blue-900 focus:outline-none"
                                        >
                                            {subjects.map(sub => (
                                                <option key={sub} value={sub}>{sub}</option>
                                            ))}
                                        </select>
                                    </div>
                                )}

                                <button
                                    type="submit"
                                    className="w-full py-3 bg-blue-900 hover:bg-blue-800 text-white font-semibold rounded-xl shadow-md transition-all flex items-center justify-center gap-2 mt-2"
                                >
                                    <span>Vào hệ thống</span>
                                    <i data-lucide="arrow-right" className="w-4 h-4"></i>
                                </button>
                            </form>
                            <div className="mt-6 text-center text-xs text-slate-400">
                                ClassTrack - Phiên bản hỗ trợ tùy chỉnh tên giáo viên
                            </div>
                        </div>
                    </div>
                );
            }

            return (
                <DashboardApp 
                    currentUser={currentUser}
                    onLogout={handleLogout}
                    students={students}
                    setStudents={setStudents}
                    homeworks={homeworks}
                    setHomeworks={setHomeworks}
                    subjects={subjects}
                    weeks={weeks}
                    activeTab={activeTab}
                    setActiveTab={setActiveTab}
                    toast={toast}
                    showToast={showToast}
                />
            );
        }

        function DashboardApp({ currentUser, onLogout, students, setStudents, homeworks, setHomeworks, subjects, weeks, activeTab, setActiveTab, toast, showToast }) {
            useEffect(() => {
                if (window.lucide) {
                    window.lucide.createIcons();
                }
            });

            const [filterWeek, setFilterWeek] = useState("Tất cả");
            const [filterSubject, setFilterSubject] = useState("Tất cả");
            const [filterStudent, setFilterStudent] = useState("Tất cả");
            const [searchTerm, setSearchTerm] = useState("");

            const filteredHomeworks = useMemo(() => {
                return homeworks.filter(hw => {
                    if (currentUser.role === 'gvbm' && hw.subject !== currentUser.subject) {
                        return false;
                    }
                    if (filterSubject !== "Tất cả" && hw.subject !== filterSubject) return false;
                    if (filterStudent !== "Tất cả" && hw.student !== filterStudent) return false;
                    if (searchTerm && !hw.student.toLowerCase().includes(searchTerm.toLowerCase()) && !hw.content.toLowerCase().includes(searchTerm.toLowerCase())) return false;
                    return true;
                });
            }, [homeworks, currentUser, filterSubject, filterStudent, searchTerm]);

            const stats = useMemo(() => {
                const list = currentUser.role === 'gvbm' ? homeworks.filter(h => h.subject === currentUser.subject) : homeworks;
                const totalStudents = students.length;
                const totalHw = list.length;
                const completed = list.filter(h => h.progress === 'Đã hoàn thành' || h.progress === 'Đã nộp').length;
                const pendingOrOverdue = list.filter(h => h.progress === 'Chưa làm' || h.progress === 'Đang làm').length;
                const unchecked = list.filter(h => h.checkStatus === 'Chưa kiểm tra').length;
                const needSupplement = list.filter(h => h.checkStatus === 'Cần bổ sung').length;

                return { totalStudents, totalHw, completed, pendingOrOverdue, unchecked, needSupplement };
            }, [homeworks, students, currentUser]);

            return (
                <div className="min-h-screen flex flex-col bg-slate-50">
                    {toast && (
                        <div className="fixed bottom-5 right-5 z-50 bg-slate-900 text-white px-5 py-3 rounded-xl shadow-2xl flex items-center gap-3 border border-slate-700 animate-bounce">
                            <i data-lucide="check-circle-2" className="w-5 h-5 text-teal-400"></i>
                            <span className="text-sm font-medium">{toast}</span>
                        </div>
                    )}

                    <header className="bg-blue-900 text-white shadow-md sticky top-0 z-30">
                        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
                            <div className="flex items-center gap-3">
                                <div className="bg-teal-500 p-2 rounded-xl text-blue-950 font-bold shadow">
                                    <i data-lucide="graduation-cap" className="w-6 h-6"></i>
                                </div>
                                <div>
                                    <span className="font-bold text-lg tracking-wide">ClassTrack</span>
                                    <span className="text-xs text-teal-300 block">Theo dõi bài tập – Đồng hành cùng học sinh (Lớp 12/1)</span>
                                </div>
                            </div>
                            
                            <div className="flex items-center gap-4">
                                <div className="text-right hidden sm:block">
                                    <span className="text-sm font-semibold block text-teal-200">{currentUser.
