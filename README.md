<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ClientDrive CRM</title>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <style>
    body { margin: 0; padding: 0; }
  </style>
</head>
<body>
  <div id="root"></div>
  
  <script type="text/babel">
    const { useState, useEffect, useRef, useCallback } = React;
    


const STAGES = ["New Lead", "Contacted", "Proposal Sent", "Negotiating", "Closed Won", "Closed Lost"];
const STAGE_COLORS = {
  "New Lead": "#3b82f6", "Contacted": "#f59e0b", "Proposal Sent": "#8b5cf6",
  "Negotiating": "#f97316", "Closed Won": "#10b981", "Closed Lost": "#ef4444",
};
const SOURCES = ["Website", "Referral", "Cold Call", "LinkedIn", "Email Campaign", "Event", "Other"];
const COURSES = [
  "Y3 - Maths", "Y3 - English",
  "Y4 - Maths", "Y4 - English",
  "Y5 - Maths", "Y5 - English",
  "Y6 - Maths", "Y6 - English",
  "Y7 - Maths", "Y7 - English", "Y7 - Biology", "Y7 - Chemistry", "Y7 - Physics",
  "Y8 - Maths", "Y8 - English", "Y8 - Biology", "Y8 - Chemistry", "Y8 - Physics",
  "Y9 - Maths", "Y9 - English", "Y9 - Biology", "Y9 - Chemistry", "Y9 - Physics",
  "Y10 - Maths", "Y10 - English", "Y10 - Biology", "Y10 - Chemistry", "Y10 - Physics",
  "Y11 - Maths", "Y11 - English", "Y11 - Biology", "Y11 - Chemistry", "Y11 - Physics",
  "11+ Preparation",
  "Y12 - Maths", "Y12 - English", "Y12 - Biology", "Y12 - Chemistry", "Y12 - Physics", "Y12 - Psychology", "Y12 - UCAT",
  "Y13 - Maths", "Y13 - English", "Y13 - Biology", "Y13 - Chemistry", "Y13 - Physics", "Y13 - Psychology"
];
const TEAM_MEMBERS = ["Alice", "Bob", "Carlos", "Diana", "Emma", "All"];
const SALES_TEAM = ["Alice", "Bob", "Carlos", "Diana", "Emma"]; // Sales team members

const sampleData = [
  { id: "1", name: "Sarah Johnson", company: "TechCorp Ltd", email: "sarah@techcorp.com", phone: "07712 345678", course: [], value: 2400, stage: "Proposal Sent", source: "LinkedIn", assignedTo: "Alice", notes: "Very interested, needs board approval", createdAt: "2026-02-01", lastContact: "2026-02-03" },
  { id: "2", name: "Marcus Chen", company: "StartUp Hub", email: "marcus@startuphub.io", phone: "07890 123456", course: "Y11 - English", value: 1800, stage: "Negotiating", source: "Referral", assignedTo: "Bob", notes: "Price sensitive, offered 10% discount", createdAt: "2026-01-15", lastContact: "2026-02-10" },
  { id: "3", name: "Emma Williams", company: "Retail Co", email: "emma@retailco.com", phone: "07654 321098", course: "Y9 - Chemistry", value: 3200, stage: "Closed Won", source: "Website", assignedTo: "Carlos", notes: "Signed! Starting March.", createdAt: "2026-01-10", lastContact: "2026-02-10" },
  { id: "4", name: "James Patel", company: "Finance Group", email: "james@fingroup.com", phone: "07321 654987", course: "Y12 - Psychology", value: 4500, stage: "Contacted", source: "Cold Call", assignedTo: "Diana", notes: "Follow up Thursday", createdAt: "2026-02-10", lastContact: "2026-02-08" },
  { id: "5", name: "Lily Martinez", company: "Healthcare Plus", email: "lily@healthplus.com", phone: "07445 876543", course: "11+ Preparation", value: 2100, stage: "New Lead", source: "Event", assignedTo: "Emma", notes: "Met at London conference", createdAt: "2026-02-15", lastContact: "2026-02-05" },
  { id: "6", name: "Tom Brooks", company: "Media Agency", email: "tom@mediaagency.co.uk", phone: "07567 234567", course: "Y8 - Maths", value: 1600, stage: "Closed Lost", source: "Email Campaign", assignedTo: "Alice", notes: "Went with competitor", createdAt: "2026-01-20", lastContact: "2026-02-08" },
];

function generateId() { return Date.now().toString(36) + Math.random().toString(36).slice(2); }

function daysSince(dateStr) {
  const d = new Date(dateStr);
  const now = new Date();
  return Math.max(0, Math.floor((now - d) / (1000 * 60 * 60 * 24)));
}

function ClientDriveCRM() {
  const [activeTab, setActiveTab] = useState("dashboard");
  const [clients, setClients] = useState([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState("");
  const [filterStage, setFilterStage] = useState("All");
  const [filterAssignee, setFilterAssignee] = useState("All");
  const [selectedClient, setSelectedClient] = useState(null);
  const [addForm, setAddForm] = useState({ name: "", company: "", email: "", phone: "", course: [], value: "", stage: "New Lead", source: "Website", assignedTo: "Alice", notes: "" });
  const [addSuccess, setAddSuccess] = useState(false);
  const [editClient, setEditClient] = useState(null);
  const [presentSlide, setPresentSlide] = useState(0);
  const [currentUser, setCurrentUser] = useState(null); // Current logged in user
  const [viewingUser, setViewingUser] = useState(null); // Who's data we're viewing (manager can switch)
  const [isManager, setIsManager] = useState(false);
  const [showLogin, setShowLogin] = useState(true);
  const [loginName, setLoginName] = useState("");

  // ── NOTIFICATION STATE ──
  const [notifTime, setNotifTime] = useState("09:00");
  const [chaseAfterDays, setChaseAfterDays] = useState(3);
  const [notifPermission, setNotifPermission] = useState("default");
  const [showNotifPanel, setShowNotifPanel] = useState(false);
  const [showNotifSettings, setShowNotifSettings] = useState(false);
  const [toasts, setToasts] = useState([]);
  const [notifFired, setNotifFired] = useState({});
  const panelRef = useRef(null);

  const addToast = useCallback((msg, type = "info") => {
    const id = generateId();
    setToasts(t => [...t, { id, msg, type }]);
    setTimeout(() => setToasts(t => t.filter(x => x.id !== id)), 5000);
  }, []);

  const chaseUpClients = clients.filter(c =>
    !["Closed Won", "Closed Lost"].includes(c.stage) &&
    daysSince(c.lastContact) >= chaseAfterDays
  ).sort((a, b) => daysSince(b.lastContact) - daysSince(a.lastContact));

  useEffect(() => {
    async function loadData() {
      try {
        const res = await window.storage.get("clientdrive_clients");
        if (res && res.value) setClients(JSON.parse(res.value));
        else { setClients(sampleData); await window.storage.set("clientdrive_clients", JSON.stringify(sampleData)); }
        const ns = await window.storage.get("clientdrive_notif_settings");
        if (ns && ns.value) {
          const s = JSON.parse(ns.value);
          if (s.notifTime) setNotifTime(s.notifTime);
          if (s.chaseAfterDays) setChaseAfterDays(s.chaseAfterDays);
        }
        const fired = await window.storage.get("clientdrive_notif_fired");
        if (fired && fired.value) setNotifFired(JSON.parse(fired.value));
      } catch { setClients(sampleData); }
      setLoading(false);
    }
    loadData();
    setNotifPermission(Notification?.permission || "default");
  }, []);

  useEffect(() => {
    if (!loading) {
      window.storage.set("clientdrive_notif_settings", JSON.stringify({ notifTime, chaseAfterDays })).catch(() => {});
    }
  }, [notifTime, chaseAfterDays, loading]);

  useEffect(() => {
    const check = () => {
      const now = new Date();
      const hh = String(now.getHours()).padStart(2, "0");
      const mm = String(now.getMinutes()).padStart(2, "0");
      const todayKey = `${now.toISOString().split("T")[0]}_${notifTime}`;
      if (`${hh}:${mm}` === notifTime && !notifFired[todayKey] && chaseUpClients.length > 0) {
        const updated = { ...notifFired, [todayKey]: true };
        setNotifFired(updated);
        window.storage.set("clientdrive_notif_fired", JSON.stringify(updated)).catch(() => {});
        if (Notification?.permission === "granted") {
          new Notification("🔔 ClientDrive CRM — Chase Up Time!", {
            body: `${chaseUpClients.length} client${chaseUpClients.length > 1 ? "s" : ""} need a follow-up today!`,
          });
        }
        addToast(`⏰ Daily reminder: ${chaseUpClients.length} client${chaseUpClients.length > 1 ? "s" : ""} need chasing up!`, "chase");
        setShowNotifPanel(true);
      }
    };
    const interval = setInterval(check, 30000);
    check();
    return () => clearInterval(interval);
  }, [notifTime, notifFired, chaseUpClients.length, addToast]);

  useEffect(() => {
    const handler = (e) => {
      if (panelRef.current && !panelRef.current.contains(e.target)) setShowNotifPanel(false);
    };
    document.addEventListener("mousedown", handler);
    return () => document.removeEventListener("mousedown", handler);
  }, []);

  async function saveClients(updated) {
    setClients(updated);
    try { await window.storage.set("clientdrive_clients", JSON.stringify(updated)); } catch {}
  }

  function handleAddClient(e) {
    e.preventDefault();
    const newClient = { ...addForm, id: generateId(), value: Number(addForm.value) || 0, createdAt: new Date().toISOString().split("T")[0], lastContact: new Date().toISOString().split("T")[0] };
    saveClients([newClient, ...clients]);
    setAddForm({ name: "", company: "", email: "", phone: "", course: [], value: "", stage: "New Lead", source: "Website", assignedTo: "Alice", notes: "" });
    setAddSuccess(true);
    setTimeout(() => setAddSuccess(false), 3000);
  }

  function handleUpdateClient(e) {
    e.preventDefault();
    const updated = clients.map(c => c.id === editClient.id ? { ...editClient, value: Number(editClient.value) } : c);
    saveClients(updated);
    setEditClient(null);
    setSelectedClient(null);
  }

  function handleDeleteClient(id) {
    saveClients(clients.filter(c => c.id !== id));
    setSelectedClient(null);
  }

  function markContacted(client) {
    const today = new Date().toISOString().split("T")[0];
    saveClients(clients.map(c => c.id === client.id ? { ...c, lastContact: today } : c));
    addToast(`✓ ${client.name} marked as contacted today`, "success");
  }

  async function requestNotifPermission() {
    if (!("Notification" in window)) { addToast("Your browser doesn't support notifications", "error"); return; }
    const perm = await Notification.requestPermission();
    setNotifPermission(perm);
    if (perm === "granted") addToast("✓ Browser notifications enabled!", "success");
    else addToast("Notifications blocked — enable in browser settings", "error");
  }

  function testNotification() {
    addToast(`⏰ Test: ${chaseUpClients.length} client${chaseUpClients.length !== 1 ? "s" : ""} need following up.`, "chase");
    setShowNotifPanel(true);
    if (Notification?.permission === "granted" && chaseUpClients.length > 0) {
      new Notification("🔔 ClientDrive CRM — Chase Up Time!", { body: `${chaseUpClients.length} client${chaseUpClients.length > 1 ? "s" : ""} waiting for follow-up!` });
    }
  }

  const filteredClients = clients.filter(c => {
    const matchSearch = !search || c.name.toLowerCase().includes(search.toLowerCase()) || c.company.toLowerCase().includes(search.toLowerCase()) || c.email.toLowerCase().includes(search.toLowerCase());
    const matchStage = filterStage === "All" || c.stage === filterStage;
    const matchAssignee = filterAssignee === "All" || c.assignedTo === filterAssignee;
    return matchSearch && matchStage && matchAssignee;
  });

  const totalPipeline = clients.filter(c => c.stage !== "Closed Lost").reduce((s, c) => s + c.value, 0);
  const closedWon = clients.filter(c => c.stage === "Closed Won").reduce((s, c) => s + c.value, 0);
  const activeLeads = clients.filter(c => !["Closed Won", "Closed Lost"].includes(c.stage)).length;
  const convRate = clients.length ? Math.round((clients.filter(c => c.stage === "Closed Won").length / clients.length) * 100) : 0;
  const stageCounts = STAGES.map(s => ({ stage: s, count: clients.filter(c => c.stage === s).length, value: clients.filter(c => c.stage === s).reduce((a, c) => a + c.value, 0) }));
  const teamStats = TEAM_MEMBERS.filter(m => m !== "All").map(m => ({
    name: m, total: clients.filter(c => c.assignedTo === m).length,
    won: clients.filter(c => c.assignedTo === m && c.stage === "Closed Won").length,
    pipeline: clients.filter(c => c.assignedTo === m && !["Closed Won", "Closed Lost"].includes(c.stage)).reduce((s, c) => s + c.value, 0),
    revenue: clients.filter(c => c.assignedTo === m && c.stage === "Closed Won").reduce((s, c) => s + c.value, 0),
  }));

  // Login screen
  if (showLogin) {
    return (
      <div style={{ background: "#0a0e1a", minHeight: "100vh", display: "flex", alignItems: "center", justifyContent: "center", fontFamily: "'Segoe UI', system-ui, sans-serif" }}>
        <div style={{ maxWidth: 500, width: "100%", padding: 40 }}>
          <div style={{ textAlign: "center", marginBottom: 32 }}>
            <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 32, fontWeight: 900, color: "#e8e0d0", marginBottom: 8 }}>
              ClientDrive CRM
            </div>
            <div style={{ fontSize: 14, color: "#6b7280" }}>Select your profile to access your clients</div>
          </div>
          <div style={{ background: "#111827", border: "1px solid #1f2937", borderRadius: 16, padding: 32 }}>
            <label className="form-label">Who are you?</label>
            <select className="form-input" style={{ marginBottom: 16 }} value={loginName} onChange={e => setLoginName(e.target.value)}>
              <option value="">Select your name...</option>
              <option value="Manager">Manager (View All)</option>
              {SALES_TEAM.map(name => <option key={name} value={name}>{name}</option>)}
            </select>
            <button 
              className="btn-gold" 
              style={{ width: "100%", padding: "12px" }}
              disabled={!loginName}
              onClick={async () => {
                if (loginName === "Manager") {
                  await window.storage.set("clientdrive_current_user", "Manager");
                  await window.storage.set("clientdrive_is_manager", true);
                  setCurrentUser("Manager");
                  setViewingUser("All");
                  setIsManager(true);
                } else {
                  await window.storage.set("clientdrive_current_user", loginName);
                  await window.storage.set("clientdrive_is_manager", false);
                  setCurrentUser(loginName);
                  setViewingUser(loginName);
                  setIsManager(false);
                }
                setShowLogin(false);
                window.location.reload();
              }}>
              Access My CRM →
            </button>
            <div style={{ marginTop: 20, padding: "12px 16px", background: "#c8a84b15", border: "1px solid #c8a84b33", borderRadius: 8, fontSize: 12, color: "#9ca3af" }}>
              <div style={{ fontWeight: 700, marginBottom: 6, color: "#c8a84b" }}>How it works:</div>
              <div style={{ marginBottom: 4 }}>• <strong>Sales Team:</strong> See only your own clients</div>
              <div>• <strong>Manager:</strong> View all team members' clients</div>
            </div>
          </div>
        </div>
      </div>
    );
  }

  if (loading) return (
    <div style={{ background: "#0a0e1a", height: "100vh", display: "flex", alignItems: "center", justifyContent: "center" }}>
      <div style={{ display: "flex", flexDirection: "column", alignItems: "center", gap: 16 }}><img src="data:image/jpeg;base64,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QFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAAAUAAAAAAAAAAAAAAAAAAAAEABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQFABAUAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEBQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAUAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEBQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAFAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAUAAAAB//xAAC/9oADAMBAAIAAwAAACHzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzjLzDDDDTzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzIEgXBPBXQ2xsba1uk7bzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoWha3Fn1OgLTQPqlfAANFrzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzgBvuFHFgbRWqq9OWZUcDe3NbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyEemuFN6x6+1u7zno19rvnvijTzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzxg0dWibzzzzzz6QNwokJvngt5bzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoviXMzzzzzzzzrr8q9mxqeUZirzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyk9wGPzzzzzzz0ERqvylyr32nGrzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyc5ybzzzzr1KSoNXt97zwjKLOrzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzUamrnrKHkh2YGzi8zzzzxcorOLzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7b6z6bccFTKlQUUtbzzywSTiQbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyyMV25Py3qF6mBWHTzzqfUePStzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoBKvJdh7yi36jShbq6bYo0VbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyxoPnYhQi7wAh4KYDzRCxRgbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyw6sA5wgqRfy76rKZHZmI7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzwzww74444456444447zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzTDDSDTzTzzzzzzzzzzy5/f3zzzznvzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzoAAAxCgCBz7zzzzzzxjzwnaz5Tzz6gX7zzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzygTzzziBgTgQjDwDBSABD6ffxvui+e64FXsaWWHXzzzzzz7zzzzzzzzzzzzzzzzzzgDzzzxTwADxATQAiBChzpDXzutYb53bvS+eZIbLzzzzzzzzzzzzzzzzzzzzzzzzygBDzSgTwQigyiDzgAyRjn9XGoS3bz7XqNnnHKp/zz77zzzzzzzzzzzzzzzzzzzzz6gigjwRAAhBCwjzxSgxzkbxjd55fyHWvmbxoZBbzzzzzzz7zzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzz77zzz7zzzzzzzzzzzzzzzzz7zzzzzzzzzzzzz7zzz77z77777777zzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz777777zzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzz/xAAC/9oADAMBAAIAAwAAABDzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzjLzDDDDTzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz4r/BJrRfjHJoYh0JAj7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoHVdTzbFlsq16i2u7JNITzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzgep1/jjESczJNF9W74UpSo57zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyIIPPdJW/ze4dVNxN67MoabKfTzzzzzzzzzzzzzzzzzyzzzzzzzzzzzzzzzzzzzzymMX38XzzzzzzmJcIcF+2Xnsd7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz1xjDQzzzzzzzz6CiLzTf0kkxwHzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzo1Q+PzzzzzzzEh0CsqXzpfyji3zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzy51Fefzzzz3sfbl3UjZ7zybzgnnzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyDZStDjqclXqp1DYEXzzzw5ec0nzzzzzzzzzzzzzzzzzyzzzzzzzzzzzzzzzzzzzz+qwxJY9k1LJ0+JqDfzzzSoJzObzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyyNlIG3b9yRTy6+dv3nmGau/hFzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzvZSzG5ZjLc6CJrCPsGN9h6jfzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzywtIgCoAxL+EjJKTfMMnFVSfzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyw7sLbbU8fXEMLM0ELBbG7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzwzww74444456448747zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzDTDSDzzzzzzzzzzzzzypDjvTzzzrbzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzpgQwzCDARTrzzzzzzwBzzq71n3zzKc77zzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzjTzzygBQTRTwCiSADABT6T7wq/CVeGxFSmjciz7zzzzzz7zzzzzzzzzzzzzzzzzzgTzzzwDgCAhggxTgBiBx6rfyuO2g0T1tH1ACgCHzzzzzzzzzzzzzzzzzzzzzzzzzhzDjhRBQSDzxACSwRgRysFT5rAvbx5VGM69fl8zzz77zzzzzzzzjzzzzzzzzzzzz6TixxhiwxggyQBShjgzgH0tGx6f8A8A+rqO8oBV/8888888+8888888888888888++888888888888888888888+888M888888888888888+88888888888888888888++888888888888888888888888888888+888888888+8888888888888888888888+888888888888888888888888888888888888888888888888888888888888888+8888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888+888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888++88888888888888888888888888888888888888888888888888888888888888++888888888888888888888++888+88888888888888888+8888888888888+888++8++++++++88888888888888888+8888888888888888888888888888888888888888++++++888888888888888888888++888888888888888888888888888888888888+88+8888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888+888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888+8888888888888888888888888888888+88888888888888888888888888888888888888888888888888888888888888888888888+8888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888++88888888888888888888888888888++888888/8QAQREAAgEDAQQFCAcHAwUAAAAAAQIDAAQRBRIhMVEQMkFhcRMUIFBSU2KRFSJAQkOBoSMwM2NykJJgorGAgtHh8P/aAAgBAgEBPwD+8sf+kJnVRlmAHM0+o2qffLeAptXj+7Ex8SBX0v8AyP8AdQ1X+T/ur6U/lfrX0n/K/WoL0yyBPJ4781PN5JNrGd9efn3Y+dC9J/D/AFoXJP3B868s3sihI/sj51t/CaDA8D6veREUs7ACp9TY5EIwPaNO7ucuxJ7+kUKHRp65d25Cr9uov50KSkpaWgtFQeIohhwOe40GBoH1XNMkKFm/Ic6uJ5Jmyx8B2D0gaFCtPTEGfaNXr5nI5DFA0ppGpTSsK8oB2E0XfkBSliN9HnStn1U7qqlmOAKuJ2mcseHYOQo9MVvNL1EJ76TSpD15APDfQ0qLtkevoqH23r6Mh9t6+jYvbao4wiKg4AU+no7sxdsk5oacnttQsUH3zQtVH3jWxGnbvrbUcBW2TTtsALmkkzTNQk2WoHI9U6jNvEQ8W6CKIq10/aAeXh2LSqFAAAAHps6oMsQBT6jbLuBLeAp9TY9SMDxo3cr8WpZKElQKT9Y/lVzGwO3SyV5SnarWXaTHKh6nJABJqVy8jOe09NhbBz5VxuHVH7q7vbdcqEWRvDIFFsnNA0Gq2RppAud3bSgAACioIINTKYpC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" alt="ClientDrive" style={{ height: 80, width: "auto" }} /><div style={{ color: "#c8a84b", fontFamily: "Georgia, serif", fontSize: 16, letterSpacing: "0.1em" }}>Loading your CRM…</div></div>
    </div>
  );

  return (
    <div style={{ fontFamily: "'Segoe UI', system-ui, sans-serif", background: "#0a0e1a", minHeight: "100vh", color: "#e8e0d0" }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=DM+Sans:wght@300;400;500;600&display=swap');
        * { box-sizing: border-box; }
        ::-webkit-scrollbar { width: 4px; } ::-webkit-scrollbar-track { background: #111827; } ::-webkit-scrollbar-thumb { background: #c8a84b44; border-radius: 2px; }
        input, select, textarea { font-family: inherit; }
        .tab-btn { background: none; border: none; padding: 10px 20px; cursor: pointer; font-size: 13px; font-weight: 600; letter-spacing: 0.08em; text-transform: uppercase; transition: all 0.2s; position: relative; }
        .tab-btn.active { color: #c8a84b; }
        .tab-btn.active::after { content: ''; position: absolute; bottom: -1px; left: 0; right: 0; height: 2px; background: #c8a84b; }
        .tab-btn:not(.active) { color: #6b7280; }
        .tab-btn:hover:not(.active) { color: #9ca3af; }
        .card { background: #111827; border: 1px solid #1f2937; border-radius: 12px; }
        .stat-card { background: linear-gradient(135deg, #111827 0%, #0d1520 100%); border: 1px solid #1f2937; border-radius: 12px; padding: 20px 24px; }
        .badge { display: inline-block; padding: 3px 10px; border-radius: 20px; font-size: 11px; font-weight: 700; letter-spacing: 0.05em; }
        .btn-gold { background: linear-gradient(135deg, #c8a84b, #e8c96a); color: #0a0e1a; border: none; padding: 10px 22px; border-radius: 8px; font-weight: 700; font-size: 13px; cursor: pointer; letter-spacing: 0.05em; transition: opacity 0.2s; }
        .btn-gold:hover { opacity: 0.9; }
        .btn-outline { background: transparent; border: 1px solid #374151; color: #9ca3af; padding: 8px 16px; border-radius: 8px; font-size: 12px; cursor: pointer; transition: all 0.2s; }
        .btn-outline:hover { border-color: #6b7280; color: #e8e0d0; }
        .btn-green { background: #10b98120; border: 1px solid #10b98144; color: #10b981; padding: 5px 12px; border-radius: 6px; font-size: 11px; font-weight: 700; cursor: pointer; white-space: nowrap; }
        .btn-green:hover { background: #10b98133; }
        .btn-red { background: transparent; border: 1px solid #ef444455; color: #ef4444; padding: 8px 16px; border-radius: 8px; font-size: 12px; cursor: pointer; }
        .btn-red:hover { background: #ef444418; }
        .form-input { background: #1f2937; border: 1px solid #374151; color: #e8e0d0; border-radius: 8px; padding: 10px 14px; font-size: 14px; width: 100%; outline: none; transition: border-color 0.2s; }
        .form-input:focus { border-color: #c8a84b; }
        .form-label { display: block; font-size: 11px; font-weight: 700; letter-spacing: 0.1em; text-transform: uppercase; color: #6b7280; margin-bottom: 6px; }
        .client-row { display: grid; grid-template-columns: 2fr 1.5fr 1.5fr 1fr 1fr 1fr; gap: 12px; padding: 14px 20px; border-bottom: 1px solid #1f2937; align-items: center; cursor: pointer; transition: background 0.15s; border-left: 3px solid transparent; }
        .client-row:hover { background: #111827; }
        .client-row-header { color: #4b5563; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.08em; border-left: 3px solid transparent !important; }
        .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 100; display: flex; align-items: center; justify-content: center; padding: 20px; }
        .modal { background: #111827; border: 1px solid #1f2937; border-radius: 16px; width: 100%; max-width: 620px; max-height: 90vh; overflow-y: auto; }
        .progress-bar { height: 6px; background: #1f2937; border-radius: 3px; overflow: hidden; }
        .progress-fill { height: 100%; border-radius: 3px; transition: width 0.5s; }
        .avatar { width: 36px; height: 36px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 800; font-size: 13px; flex-shrink: 0; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes slideDown { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes toastIn { from { opacity: 0; transform: translateX(110%); } to { opacity: 1; transform: translateX(0); } }
        @keyframes bellRing { 0%,100%{transform:rotate(0)} 20%{transform:rotate(-15deg)} 40%{transform:rotate(15deg)} 60%{transform:rotate(-10deg)} 80%{transform:rotate(10deg)} }
        .fade-in { animation: fadeIn 0.4s ease forwards; }
        .slide-down { animation: slideDown 0.2s ease forwards; }
        .bell-ring { animation: bellRing 0.6s ease; }
        .notif-panel { position: absolute; top: 68px; right: 0; width: 390px; background: #111827; border: 1px solid #1f2937; border-radius: 14px; box-shadow: 0 24px 64px rgba(0,0,0,0.7); z-index: 300; overflow: hidden; }
        .notif-bell-btn { background: none; border: none; cursor: pointer; position: relative; padding: 8px; border-radius: 8px; transition: background 0.2s; display: flex; align-items: center; justify-content: center; }
        .notif-bell-btn:hover { background: #1f2937; }
        .bell-badge { position: absolute; top: 2px; right: 2px; background: #ef4444; color: white; font-size: 9px; font-weight: 900; min-width: 16px; height: 16px; border-radius: 8px; display: flex; align-items: center; justify-content: center; border: 2px solid #0a0e1a; padding: 0 3px; }
        .toast-wrap { position: fixed; top: 80px; right: 20px; z-index: 999; display: flex; flex-direction: column; gap: 10px; pointer-events: none; max-width: 360px; }
        .toast { animation: toastIn 0.35s cubic-bezier(.22,1,.36,1) forwards; pointer-events: all; padding: 14px 18px; border-radius: 12px; font-size: 13px; font-weight: 600; line-height: 1.4; box-shadow: 0 8px 32px rgba(0,0,0,0.5); }
        .chase-row { padding: 12px 18px; border-bottom: 1px solid #1a2235; display: flex; align-items: center; gap: 12; transition: background 0.15s; }
        .chase-row:hover { background: #0d1520; }
      `}</style>

      {/* TOASTS */}
      <div className="toast-wrap">
        {toasts.map(t => (
          <div key={t.id} className="toast" style={{
            background: t.type === "chase" ? "#c8a84b15" : t.type === "success" ? "#10b98115" : t.type === "error" ? "#ef444415" : "#1f2937",
            border: `1px solid ${t.type === "chase" ? "#c8a84b55" : t.type === "success" ? "#10b98155" : t.type === "error" ? "#ef444455" : "#374151"}`,
            color: t.type === "chase" ? "#e8c86a" : t.type === "success" ? "#10b981" : t.type === "error" ? "#ef4444" : "#e8e0d0",
          }}>{t.msg}</div>
        ))}
      </div>

      {/* HEADER */}
      <div style={{ background: "#080c17", borderBottom: "1px solid #1a2235", padding: "0 32px", position: "sticky", top: 0, zIndex: 200 }}>
        <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", height: 60 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 14 }}>
            {isManager && (
              <select 
                className="form-input" 
                style={{ width: 150, fontSize: 12, padding: "6px 10px", marginRight: 8 }}
                value={viewingUser}
                onChange={async (e) => {
                  setViewingUser(e.target.value);
                  setLoading(true);
                  // Reload data for selected user
                  const storageKey = e.target.value === "All" ? "clientdrive_clients_all" : `clientdrive_clients_${e.target.value}`;
                  const res = await window.storage.get(storageKey, true);
                  if (res && res.value) {
                    setClients(JSON.parse(res.value));
                  } else {
                    setClients([]);
                  }
                  setLoading(false);
                }}>
                <option value="All">All Team</option>
                {SALES_TEAM.map(name => <option key={name} value={name}>{name}</option>)}
              </select>
            )}
            <img src="data:image/jpeg;base64,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QFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAAAUAAAAAAAAAAAAAAAAAAAAEABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQFABAUAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEBQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAUAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEBQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAFAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAUAAAAB//xAAC/9oADAMBAAIAAwAAACHzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzjLzDDDDTzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzIEgXBPBXQ2xsba1uk7bzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoWha3Fn1OgLTQPqlfAANFrzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzgBvuFHFgbRWqq9OWZUcDe3NbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyEemuFN6x6+1u7zno19rvnvijTzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzxg0dWibzzzzzz6QNwokJvngt5bzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoviXMzzzzzzzzrr8q9mxqeUZirzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyk9wGPzzzzzzz0ERqvylyr32nGrzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyc5ybzzzzr1KSoNXt97zwjKLOrzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzUamrnrKHkh2YGzi8zzzzxcorOLzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7b6z6bccFTKlQUUtbzzywSTiQbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyyMV25Py3qF6mBWHTzzqfUePStzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoBKvJdh7yi36jShbq6bYo0VbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyxoPnYhQi7wAh4KYDzRCxRgbzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyw6sA5wgqRfy76rKZHZmI7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzwzww74444456444447zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzTDDSDTzTzzzzzzzzzzy5/f3zzzznvzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzoAAAxCgCBz7zzzzzzxjzwnaz5Tzz6gX7zzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzygTzzziBgTgQjDwDBSABD6ffxvui+e64FXsaWWHXzzzzzz7zzzzzzzzzzzzzzzzzzgDzzzxTwADxATQAiBChzpDXzutYb53bvS+eZIbLzzzzzzzzzzzzzzzzzzzzzzzzygBDzSgTwQigyiDzgAyRjn9XGoS3bz7XqNnnHKp/zz77zzzzzzzzzzzzzzzzzzzzz6gigjwRAAhBCwjzxSgxzkbxjd55fyHWvmbxoZBbzzzzzzz7zzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzz77zzz7zzzzzzzzzzzzzzzzz7zzzzzzzzzzzzz7zzz77z77777777zzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz777777zzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzzzzzzzzzzzzzzzzzzzzzzzzz77zzzzzz/xAAC/9oADAMBAAIAAwAAABDzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzjLzDDDDTzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz4r/BJrRfjHJoYh0JAj7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzoHVdTzbFlsq16i2u7JNITzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzgep1/jjESczJNF9W74UpSo57zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyIIPPdJW/ze4dVNxN67MoabKfTzzzzzzzzzzzzzzzzzyzzzzzzzzzzzzzzzzzzzzymMX38XzzzzzzmJcIcF+2Xnsd7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz1xjDQzzzzzzzz6CiLzTf0kkxwHzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzo1Q+PzzzzzzzEh0CsqXzpfyji3zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzy51Fefzzzz3sfbl3UjZ7zybzgnnzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyDZStDjqclXqp1DYEXzzzw5ec0nzzzzzzzzzzzzzzzzzyzzzzzzzzzzzzzzzzzzzz+qwxJY9k1LJ0+JqDfzzzSoJzObzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyyNlIG3b9yRTy6+dv3nmGau/hFzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzvZSzG5ZjLc6CJrCPsGN9h6jfzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzywtIgCoAxL+EjJKTfMMnFVSfzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzyw7sLbbU8fXEMLM0ELBbG7zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzwzww74444456448747zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzDTDSDzzzzzzzzzzzzzypDjvTzzzrbzzzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzpgQwzCDARTrzzzzzzwBzzq71n3zzKc77zzzzzzzzzzzzzzzzzzz7zzzzzzzzzzzzjTzzygBQTRTwCiSADABT6T7wq/CVeGxFSmjciz7zzzzzz7zzzzzzzzzzzzzzzzzzgTzzzwDgCAhggxTgBiBx6rfyuO2g0T1tH1ACgCHzzzzzzzzzzzzzzzzzzzzzzzzzhzDjhRBQSDzxACSwRgRysFT5rAvbx5VGM69fl8zzz77zzzzzzzzjzzzzzzzzzzzz6TixxhiwxggyQBShjgzgH0tGx6f8A8A+rqO8oBV/8888888+8888888888888888++888888888888888888888+888M888888888888888+88888888888888888888++888888888888888888888888888888+888888888+8888888888888888888888+888888888888888888888888888888888888888888888888888888888888888+8888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888+888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888++88888888888888888888888888888888888888888888888888888888888888++888888888888888888888++888+88888888888888888+8888888888888+888++8++++++++88888888888888888+8888888888888888888888888888888888888888++++++888888888888888888888++888888888888888888888888888888888888+88+8888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888+888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888+8888888888888888888888888888888+88888888888888888888888888888888888888888888888888888888888888888888888+8888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888++88888888888888888888888888888++888888/8QAQREAAgEDAQQFCAcHAwUAAAAAAQIDAAQRBRIhMVEQMkFhcRMUIFBSU2KRFSJAQkOBoSMwM2NykJJgorGAgtHh8P/aAAgBAgEBPwD+8sf+kJnVRlmAHM0+o2qffLeAptXj+7Ex8SBX0v8AyP8AdQ1X+T/ur6U/lfrX0n/K/WoL0yyBPJ4781PN5JNrGd9efn3Y+dC9J/D/AFoXJP3B868s3sihI/sj51t/CaDA8D6veREUs7ACp9TY5EIwPaNO7ucuxJ7+kUKHRp65d25Cr9uov50KSkpaWgtFQeIohhwOe40GBoH1XNMkKFm/Ic6uJ5Jmyx8B2D0gaFCtPTEGfaNXr5nI5DFA0ppGpTSsK8oB2E0XfkBSliN9HnStn1U7qqlmOAKuJ2mcseHYOQo9MVvNL1EJ76TSpD15APDfQ0qLtkevoqH23r6Mh9t6+jYvbao4wiKg4AU+no7sxdsk5oacnttQsUH3zQtVH3jWxGnbvrbUcBW2TTtsALmkkzTNQk2WoHI9U6jNvEQ8W6CKIq10/aAeXh2LSqFAAAAHps6oMsQBT6jbLuBLeAp9TY9SMDxo3cr8WpZKElQKT9Y/lVzGwO3SyV5SnarWXaTHKh6nJABJqVy8jOe09NhbBz5VxuHVH7q7vbdcqEWRvDIFFsnNA0Gq2RppAud3bSgAACioIINTKYpC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" alt="ClientDrive" style={{ height: 44, width: "auto", objectFit: "contain" }} />
            <div style={{ width: 1, height: 28, background: "#1f2937" }} />
            <div style={{ fontSize: 11, color: "#4b5563", letterSpacing: "0.12em", textTransform: "uppercase", fontWeight: 700 }}>CRM Platform</div>
          </div>

          <div style={{ display: "flex", gap: 4, alignItems: "center" }}>
            {[
              { id: "dashboard", label: "Dashboard" },
              { id: "clients", label: "Clients" },
              { id: "add", label: "+ Add Client" },
              { id: "manager", label: "Manager" },
              { id: "present", label: "▶ Present" },
            ].map(t => (
              <button key={t.id} className={`tab-btn ${activeTab === t.id ? "active" : ""}`} onClick={() => setActiveTab(t.id)}
                style={t.id === "add" ? { color: activeTab === "add" ? "#c8a84b" : "#c8a84b88" } : {}}>
                {t.label}
              </button>
            ))}

            <div style={{ width: 1, height: 24, background: "#1f2937", margin: "0 6px" }} />

            {/* ── BELL BUTTON ── */}
            <div style={{ position: "relative" }} ref={panelRef}>
              <button className="notif-bell-btn" onClick={() => setShowNotifPanel(p => !p)} title="Chase-up reminders">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none"
                  stroke={chaseUpClients.length > 0 ? "#c8a84b" : "#6b7280"} strokeWidth="2"
                  strokeLinecap="round" strokeLinejoin="round"
                  style={{ transition: "stroke 0.3s" }}>
                  <path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/>
                  <path d="M13.73 21a2 2 0 0 1-3.46 0"/>
                </svg>
                {chaseUpClients.length > 0 && (
                  <div className="bell-badge">{chaseUpClients.length}</div>
                )}
              </button>

              {/* ── NOTIFICATION PANEL ── */}
              {showNotifPanel && (
                <div className="notif-panel slide-down">
                  {/* Panel header */}
                  <div style={{ padding: "16px 18px 14px", borderBottom: "1px solid #1f2937" }}>
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 10 }}>
                      <div>
                        <div style={{ fontWeight: 800, fontSize: 15, display: "flex", alignItems: "center", gap: 8 }}>
                          🔔 Chase-Up Reminders
                          {chaseUpClients.length > 0 && (
                            <span style={{ background: "#ef444422", color: "#ef4444", fontSize: 11, fontWeight: 900, padding: "1px 8px", borderRadius: 10 }}>
                              {chaseUpClients.length} overdue
                            </span>
                          )}
                        </div>
                        <div style={{ fontSize: 11, color: "#4b5563", marginTop: 3 }}>
                          Daily reminder fires at <strong style={{ color: "#c8a84b" }}>{notifTime}</strong> · Chase after <strong style={{ color: "#c8a84b" }}>{chaseAfterDays}</strong> days no contact
                        </div>
                      </div>
                      <button className="btn-outline" style={{ padding: "4px 10px", fontSize: 11, flexShrink: 0 }}
                        onClick={() => setShowNotifSettings(s => !s)}>
                        ⚙ {showNotifSettings ? "Hide" : "Settings"}
                      </button>
                    </div>

                    {/* Settings */}
                    {showNotifSettings && (
                      <div style={{ background: "#0d1520", borderRadius: 10, padding: "14px 16px", border: "1px solid #1f2937" }}>
                        <div style={{ fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.08em", color: "#4b5563", marginBottom: 12 }}>Reminder Settings</div>
                        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10, marginBottom: 12 }}>
                          <div>
                            <label className="form-label" style={{ fontSize: 10 }}>Daily Reminder Time</label>
                            <input type="time" className="form-input" style={{ fontSize: 13, padding: "8px 10px" }}
                              value={notifTime} onChange={e => setNotifTime(e.target.value)} />
                          </div>
                          <div>
                            <label className="form-label" style={{ fontSize: 10 }}>Chase After (days)</label>
                            <input type="number" className="form-input" style={{ fontSize: 13, padding: "8px 10px" }}
                              min={1} max={30} value={chaseAfterDays} onChange={e => setChaseAfterDays(Number(e.target.value))} />
                          </div>
                        </div>
                        <div style={{ display: "flex", gap: 8, alignItems: "center" }}>
                          {notifPermission === "granted" ? (
                            <div style={{ fontSize: 11, color: "#10b981", fontWeight: 700, flex: 1 }}>✓ Browser notifications ON</div>
                          ) : (
                            <button className="btn-gold" style={{ fontSize: 11, padding: "6px 14px", flex: 1 }} onClick={requestNotifPermission}>
                              Enable Browser Notifications
                            </button>
                          )}
                          <button className="btn-outline" style={{ fontSize: 11, padding: "6px 12px", whiteSpace: "nowrap" }} onClick={testNotification}>
                            Test Now
                          </button>
                        </div>
                        <div style={{ fontSize: 10, color: "#374151", marginTop: 8, lineHeight: 1.5 }}>
                          Browser notifications appear even when the tab is in the background. Settings are saved automatically.
                        </div>
                      </div>
                    )}
                  </div>

                  {/* Chase-up list */}
                  <div style={{ maxHeight: 350, overflowY: "auto" }}>
                    {chaseUpClients.length === 0 ? (
                      <div style={{ padding: "32px 18px", textAlign: "center" }}>
                        <div style={{ fontSize: 32, marginBottom: 8 }}>✅</div>
                        <div style={{ fontSize: 14, fontWeight: 700, color: "#10b981" }}>All caught up!</div>
                        <div style={{ fontSize: 12, color: "#374151", marginTop: 4 }}>No clients need chasing right now</div>
                      </div>
                    ) : chaseUpClients.map(c => {
                      const days = daysSince(c.lastContact);
                      const urgColor = days >= 7 ? "#ef4444" : days >= 5 ? "#f59e0b" : "#3b82f6";
                      return (
                        <div key={c.id} className="chase-row" style={{ display: "flex", alignItems: "center", gap: 10, padding: "12px 18px", borderBottom: "1px solid #1a2235" }}>
                          <div style={{ width: 3, alignSelf: "stretch", background: urgColor, borderRadius: 2, flexShrink: 0 }} />
                          <div style={{ flex: 1, minWidth: 0 }}>
                            <div style={{ fontWeight: 700, fontSize: 13, display: "flex", alignItems: "center", gap: 6, flexWrap: "wrap" }}>
                              {c.name}
                              <span style={{ fontSize: 10, fontWeight: 900, color: urgColor, background: urgColor + "18", padding: "1px 7px", borderRadius: 10, flexShrink: 0 }}>
                                {days}d ago
                              </span>
                            </div>
                            <div style={{ fontSize: 11, color: "#4b5563", marginTop: 2 }}>{c.company}</div>
                            <div style={{ fontSize: 11, marginTop: 3, display: "flex", alignItems: "center", gap: 6 }}>
                              <span className="badge" style={{ background: STAGE_COLORS[c.stage] + "22", color: STAGE_COLORS[c.stage], padding: "1px 7px", fontSize: 10 }}>{c.stage}</span>
                              <span style={{ color: "#6b7280" }}>→ {c.assignedTo}</span>
                            </div>
                          </div>
                          <button className="btn-green" onClick={() => markContacted(c)}>✓ Called</button>
                        </div>
                      );
                    })}
                  </div>

                  {chaseUpClients.length > 0 && (
                    <div style={{ padding: "10px 18px", borderTop: "1px solid #1f2937", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                      <div style={{ fontSize: 11, color: "#4b5563" }}>Click "Called" to reset their contact date to today</div>
                      <button className="btn-outline" style={{ fontSize: 11, padding: "4px 10px" }} onClick={() => setShowNotifPanel(false)}>Close</button>
                    </div>
                  )}
                </div>
              )}
            </div>
          </div>
        </div>
      </div>

      <div style={{ padding: "32px", maxWidth: 1400, margin: "0 auto" }}>

        {/* ─── DASHBOARD ─── */}
        {activeTab === "dashboard" && (
          <div className="fade-in">
            <div style={{ marginBottom: 24 }}>
              <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 28, fontWeight: 700, marginBottom: 4 }}>Sales Dashboard</div>
              <div style={{ color: "#4b5563", fontSize: 14 }}>{new Date().toLocaleDateString("en-GB", { weekday: "long", day: "numeric", month: "long", year: "numeric" })}</div>
            </div>

            {/* Chase-up banner */}
            {chaseUpClients.length > 0 && (
              <div onClick={() => setShowNotifPanel(true)} style={{ background: "linear-gradient(135deg, #c8a84b14, #f59e0b08)", border: "1px solid #c8a84b44", borderRadius: 12, padding: "16px 22px", marginBottom: 22, display: "flex", alignItems: "center", justifyContent: "space-between", gap: 16, cursor: "pointer" }}>
                <div style={{ display: "flex", alignItems: "center", gap: 14 }}>
                  <div style={{ fontSize: 26 }}>🔔</div>
                  <div>
                    <div style={{ fontWeight: 700, fontSize: 14, color: "#c8a84b" }}>
                      {chaseUpClients.length} client{chaseUpClients.length !== 1 ? "s" : ""} waiting for a follow-up
                    </div>
                    <div style={{ fontSize: 12, color: "#6b7280", marginTop: 2 }}>
                      Oldest: <strong style={{ color: "#f59e0b" }}>{chaseUpClients[0]?.name}</strong> ({daysSince(chaseUpClients[0]?.lastContact)} days) · Daily reminder at <strong style={{ color: "#c8a84b" }}>{notifTime}</strong>
                    </div>
                  </div>
                </div>
                <div style={{ color: "#c8a84b", fontWeight: 700, fontSize: 13, flexShrink: 0 }}>View Chase-Up List →</div>
              </div>
            )}

            <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 16, marginBottom: 28 }}>
              {[
                { label: "Total Pipeline", value: `£${totalPipeline.toLocaleString()}`, sub: "Active + Won", color: "#c8a84b" },
                { label: "Revenue Won", value: `£${closedWon.toLocaleString()}`, sub: "Closed deals", color: "#10b981" },
                { label: "Active Leads", value: activeLeads, sub: "In progress", color: "#3b82f6" },
                { label: "Need Chase-Up", value: chaseUpClients.length, sub: `>${chaseAfterDays} days no contact`, color: chaseUpClients.length > 0 ? "#f59e0b" : "#10b981", clickable: true },
              ].map((s, i) => (
                <div key={i} className="stat-card"
                  onClick={() => i === 3 && chaseUpClients.length > 0 && setShowNotifPanel(true)}
                  style={{ cursor: i === 3 && chaseUpClients.length > 0 ? "pointer" : "default", transition: "border-color 0.2s", borderColor: i === 3 && chaseUpClients.length > 0 ? "#f59e0b33" : "#1f2937" }}>
                  <div style={{ fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.1em", color: "#4b5563", marginBottom: 8 }}>{s.label}</div>
                  <div style={{ fontSize: 32, fontWeight: 800, color: s.color, fontFamily: "'Playfair Display', serif", lineHeight: 1 }}>{s.value}</div>
                  <div style={{ fontSize: 12, color: "#374151", marginTop: 6 }}>{s.sub}</div>
                </div>
              ))}
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 20 }}>
              <div className="card" style={{ padding: 24 }}>
                <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 20, color: "#c8a84b" }}>Pipeline by Stage</div>
                {stageCounts.map(({ stage, count, value }) => (
                  <div key={stage} style={{ marginBottom: 14 }}>
                    <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 6 }}>
                      <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                        <div style={{ width: 8, height: 8, borderRadius: "50%", background: STAGE_COLORS[stage] }} />
                        <span style={{ fontSize: 13, fontWeight: 500 }}>{stage}</span>
                        <span style={{ fontSize: 11, background: "#1f2937", padding: "1px 8px", borderRadius: 10, color: "#6b7280" }}>{count}</span>
                      </div>
                      <span style={{ fontSize: 13, fontWeight: 600, color: STAGE_COLORS[stage] }}>£{value.toLocaleString()}</span>
                    </div>
                    <div className="progress-bar">
                      <div className="progress-fill" style={{ width: clients.length ? `${(count / clients.length) * 100}%` : "0%", background: STAGE_COLORS[stage] }} />
                    </div>
                  </div>
                ))}
              </div>

              <div className="card" style={{ padding: 24 }}>
                <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 20, color: "#c8a84b" }}>Team Performance</div>
                {teamStats.map((m, i) => {
                  const colors = ["#3b82f6", "#8b5cf6", "#f59e0b", "#10b981", "#ef4444"];
                  const myChase = chaseUpClients.filter(c => c.assignedTo === m.name).length;
                  return (
                    <div key={m.name} style={{ display: "flex", alignItems: "center", gap: 12, marginBottom: 14, padding: "10px 12px", background: "#0d1520", borderRadius: 8 }}>
                      <div className="avatar" style={{ background: colors[i] + "22", color: colors[i] }}>{m.name[0]}</div>
                      <div style={{ flex: 1 }}>
                        <div style={{ fontWeight: 600, fontSize: 14, display: "flex", alignItems: "center", gap: 6 }}>
                          {m.name}
                          {myChase > 0 && <span style={{ fontSize: 10, color: "#f59e0b", background: "#f59e0b18", padding: "1px 6px", borderRadius: 8, fontWeight: 800 }}>{myChase} overdue</span>}
                        </div>
                        <div style={{ fontSize: 11, color: "#4b5563" }}>{m.total} leads · {m.won} won</div>
                      </div>
                      <div style={{ textAlign: "right" }}>
                        <div style={{ fontSize: 14, fontWeight: 700, color: "#10b981" }}>£{m.revenue.toLocaleString()}</div>
                        <div style={{ fontSize: 11, color: "#4b5563" }}>£{m.pipeline.toLocaleString()} pipeline</div>
                      </div>
                    </div>
                  );
                })}
              </div>
            </div>

            <div className="card" style={{ padding: 24, marginTop: 20 }}>
              <div style={{ fontWeight: 700, fontSize: 15, color: "#c8a84b", marginBottom: 16 }}>Recent Clients</div>
              {[...clients].sort((a, b) => b.lastContact.localeCompare(a.lastContact)).slice(0, 5).map(c => {
                const needsChase = !["Closed Won", "Closed Lost"].includes(c.stage) && daysSince(c.lastContact) >= chaseAfterDays;
                return (
                  <div key={c.id} onClick={() => { setSelectedClient(c); setActiveTab("clients"); }}
                    style={{ display: "flex", alignItems: "center", gap: 14, padding: "10px 0", borderBottom: "1px solid #1a2235", cursor: "pointer" }}>
                    <div className="avatar" style={{ background: STAGE_COLORS[c.stage] + "22", color: STAGE_COLORS[c.stage] }}>{c.name[0]}</div>
                    <div style={{ flex: 1 }}>
                      <div style={{ fontWeight: 600, fontSize: 14 }}>{c.name} <span style={{ color: "#4b5563", fontWeight: 400 }}>· {c.company}</span></div>
                      <div style={{ fontSize: 12, color: "#4b5563" }}>{Array.isArray(c.course) ? (c.course.length > 1 ? c.course.length + " subjects" : c.course[0]) : c.course} · {c.assignedTo}</div>
                    </div>
                    {needsChase && <span style={{ fontSize: 10, fontWeight: 800, color: "#f59e0b", background: "#f59e0b18", padding: "2px 8px", borderRadius: 10 }}>Chase up</span>}
                    <span className="badge" style={{ background: STAGE_COLORS[c.stage] + "22", color: STAGE_COLORS[c.stage] }}>{c.stage}</span>
                    <div style={{ color: "#c8a84b", fontWeight: 700, fontSize: 14, minWidth: 80, textAlign: "right" }}>£{c.value.toLocaleString()}</div>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {/* ─── CLIENTS ─── */}
        {activeTab === "clients" && (
          <div className="fade-in">
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 24 }}>
              <div>
                <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 28, fontWeight: 700, marginBottom: 4 }}>All Clients</div>
                <div style={{ color: "#4b5563", fontSize: 14 }}>{filteredClients.length} of {clients.length} clients</div>
              </div>
              <button className="btn-gold" onClick={() => setActiveTab("add")}>+ Quick Add Client</button>
            </div>
            <div style={{ display: "flex", gap: 12, marginBottom: 20, flexWrap: "wrap" }}>
              <input className="form-input" style={{ maxWidth: 280 }} placeholder="🔍  Search name, company, email..." value={search} onChange={e => setSearch(e.target.value)} />
              <select className="form-input" style={{ maxWidth: 160 }} value={filterStage} onChange={e => setFilterStage(e.target.value)}>
                <option value="All">All Stages</option>
                {STAGES.map(s => <option key={s}>{s}</option>)}
              </select>
              <select className="form-input" style={{ maxWidth: 140 }} value={filterAssignee} onChange={e => setFilterAssignee(e.target.value)}>
                {TEAM_MEMBERS.map(m => <option key={m}>{m}</option>)}
              </select>
              {(search || filterStage !== "All" || filterAssignee !== "All") && (
                <button className="btn-outline" onClick={() => { setSearch(""); setFilterStage("All"); setFilterAssignee("All"); }}>Clear</button>
              )}
            </div>
            <div className="card" style={{ overflow: "hidden" }}>
              <div className="client-row client-row-header" style={{ background: "#0d1520", borderBottom: "1px solid #1f2937" }}>
                <span>Client</span><span>Subject</span><span>Stage</span><span>Value</span><span>Assigned</span><span>Last Contact</span>
              </div>
              {filteredClients.length === 0 ? (
                <div style={{ padding: 40, textAlign: "center", color: "#374151" }}>No clients found</div>
              ) : filteredClients.map(c => {
                const days = daysSince(c.lastContact);
                const needsChase = !["Closed Won", "Closed Lost"].includes(c.stage) && days >= chaseAfterDays;
                return (
                  <div key={c.id} className="client-row" onClick={() => setSelectedClient(c)}
                    style={{ borderLeft: `3px solid ${needsChase ? "#f59e0b" : "transparent"}` }}>
                    <div>
                      <div style={{ fontWeight: 600, fontSize: 14, display: "flex", alignItems: "center", gap: 6 }}>
                        {c.name}
                        {needsChase && <span style={{ fontSize: 9, fontWeight: 900, color: "#f59e0b", background: "#f59e0b18", padding: "1px 6px", borderRadius: 8 }}>CHASE</span>}
                      </div>
                      <div style={{ fontSize: 12, color: "#4b5563" }}>{c.company}</div>
                    </div>
                    <div style={{ fontSize: 13, color: "#9ca3af" }}>
                      {Array.isArray(c.course) ? (
                        c.course.length > 1 ? (
                          <span>{c.course.length} subjects</span>
                        ) : c.course[0] || "—"
                      ) : c.course || "—"}
                    </div>
                    <div><span className="badge" style={{ background: STAGE_COLORS[c.stage] + "22", color: STAGE_COLORS[c.stage] }}>{c.stage}</span></div>
                    <div style={{ fontWeight: 700, color: "#c8a84b", fontSize: 14 }}>£{c.value.toLocaleString()}</div>
                    <div style={{ fontSize: 13, color: "#9ca3af" }}>{c.assignedTo}</div>
                    <div style={{ fontSize: 12, color: needsChase ? "#f59e0b" : "#4b5563" }}>
                      {c.lastContact}{needsChase ? ` (${days}d)` : ""}
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {/* ─── ADD CLIENT ─── */}
        {activeTab === "add" && (
          <div className="fade-in" style={{ maxWidth: 700, margin: "0 auto" }}>
            <div style={{ textAlign: "center", marginBottom: 32 }}>
              <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 32, fontWeight: 700, marginBottom: 6 }}>Add New Client</div>
              <div style={{ color: "#4b5563", fontSize: 14 }}>Only a name is required — fill in the rest as you go</div>
            </div>
            {addSuccess && (
              <div style={{ background: "#10b98122", border: "1px solid #10b98144", borderRadius: 10, padding: "14px 20px", marginBottom: 20, fontWeight: 600, color: "#10b981" }}>
                ✓ Client added successfully! They'll appear in your chase-up list after {chaseAfterDays} days with no contact.
              </div>
            )}
            <div className="card" style={{ padding: 32 }}>
              <form onSubmit={handleAddClient}>
                <div style={{ background: "#0d1520", borderRadius: 10, padding: 20, marginBottom: 24, border: "1px solid #c8a84b22" }}>
                  <div style={{ fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.1em", color: "#c8a84b", marginBottom: 16 }}>Essential Info</div>
                  <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 16 }}>
                    <div><label className="form-label">Full Name *</label><input required className="form-input" placeholder="e.g. John Smith" value={addForm.name} onChange={e => setAddForm({ ...addForm, name: e.target.value })} /></div>
                    <div><label className="form-label">Company</label><input className="form-input" placeholder="e.g. Acme Ltd" value={addForm.company} onChange={e => setAddForm({ ...addForm, company: e.target.value })} /></div>
                    <div><label className="form-label">Email</label><input className="form-input" type="email" placeholder="john@company.com" value={addForm.email} onChange={e => setAddForm({ ...addForm, email: e.target.value })} /></div>
                    <div><label className="form-label">Phone</label><input className="form-input" placeholder="07700 000000" value={addForm.phone} onChange={e => setAddForm({ ...addForm, phone: e.target.value })} /></div>
                  </div>
                </div>
                <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 16, marginBottom: 20 }}>
                  <div style={{ gridColumn: "1 / -1" }}>
                    <label className="form-label">Subjects & Year Groups (select multiple)</label>
                    <div style={{ background: "#0d1520", border: "1px solid #374151", borderRadius: 8, padding: "12px 14px", minHeight: 60, marginBottom: 12 }}>
                      {addForm.course && addForm.course.length === 0 ? (
                        <div style={{ color: "#4b5563", fontSize: 13 }}>Click subjects below to add...</div>
                      ) : (
                        <div style={{ display: "flex", flexWrap: "wrap", gap: 6 }}>
                          {(addForm.course || []).map(c => (
                            <span key={c} style={{ background: "#c8a84b22", border: "1px solid #c8a84b44", color: "#c8a84b", padding: "4px 10px", borderRadius: 12, fontSize: 12, fontWeight: 700, display: "flex", alignItems: "center", gap: 6 }}>
                              {c}
                              <button type="button" onClick={() => setAddForm({ ...addForm, course: (addForm.course || []).filter(x => x !== c) })} style={{ background: "none", border: "none", color: "#c8a84b", cursor: "pointer", padding: 0, fontSize: 14, lineHeight: 1 }}>×</button>
                            </span>
                          ))}
                        </div>
                      )}
                    </div>
                    <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 6, maxHeight: 200, overflowY: "auto", padding: 8, background: "#0d1520", borderRadius: 8 }}>
                      {COURSES.map(c => (
                        <button key={c} type="button" 
                          onClick={() => {
                            const current = addForm.course || [];
                            if (current.includes(c)) {
                              setAddForm({ ...addForm, course: current.filter(x => x !== c) });
                            } else {
                              setAddForm({ ...addForm, course: [...current, c] });
                            }
                          }}
                          style={{ 
                            background: (addForm.course || []).includes(c) ? "#c8a84b22" : "#1f2937",
                            border: (addForm.course || []).includes(c) ? "1px solid #c8a84b" : "1px solid #374151",
                            color: (addForm.course || []).includes(c) ? "#c8a84b" : "#9ca3af",
                            padding: "6px 8px", borderRadius: 6, fontSize: 11, fontWeight: 600, cursor: "pointer", transition: "all 0.15s", textAlign: "left"
                          }}>
                          {c}
                        </button>
                      ))}
                    </div>
                  </div>
                  <div><label className="form-label">Deal Value (£)</label><input className="form-input" type="number" placeholder="0" value={addForm.value} onChange={e => setAddForm({ ...addForm, value: e.target.value })} /></div>
                  <div><label className="form-label">Stage</label><select className="form-input" value={addForm.stage} onChange={e => setAddForm({ ...addForm, stage: e.target.value })}>{STAGES.map(s => <option key={s}>{s}</option>)}</select></div>
                  <div><label className="form-label">Source</label><select className="form-input" value={addForm.source} onChange={e => setAddForm({ ...addForm, source: e.target.value })}>{SOURCES.map(s => <option key={s}>{s}</option>)}</select></div>
                  <div><label className="form-label">Assigned To</label><select className="form-input" value={addForm.assignedTo} onChange={e => setAddForm({ ...addForm, assignedTo: e.target.value })}>{TEAM_MEMBERS.filter(m => m !== "All").map(m => <option key={m}>{m}</option>)}</select></div>
                </div>
                <div style={{ marginBottom: 24 }}>
                  <label className="form-label">Notes</label>
                  <textarea className="form-input" rows={3} placeholder="Any relevant notes..." value={addForm.notes} onChange={e => setAddForm({ ...addForm, notes: e.target.value })} style={{ resize: "vertical" }} />
                </div>
                <div style={{ display: "flex", gap: 12, justifyContent: "flex-end" }}>
                  <button type="button" className="btn-outline" onClick={() => setAddForm({ name: "", company: "", email: "", phone: "", course: [], value: "", stage: "New Lead", source: "Website", assignedTo: "Alice", notes: "" })}>Clear</button>
                  <button type="submit" className="btn-gold" style={{ padding: "12px 32px", fontSize: 15 }}>Add Client →</button>
                </div>
              </form>
            </div>
          </div>
        )}

        {/* ─── MANAGER ─── */}
        {activeTab === "manager" && (
          <div className="fade-in">
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 28 }}>
              <div>
                <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 28, fontWeight: 700, marginBottom: 4 }}>Manager View</div>
                <div style={{ color: "#4b5563", fontSize: 14 }}>Full team overview & data export</div>
              </div>
              <button className="btn-gold" onClick={() => {
                const headers = ["Name", "Company", "Email", "Phone", "Course", "Value", "Stage", "Source", "Assigned To", "Notes", "Created", "Last Contact", "Days Since Contact", "Needs Chase-Up"];
                const rows = clients.map(c => [c.name, c.company, c.email, c.phone, c.course, c.value, c.stage, c.source, c.assignedTo, `"${c.notes}"`, c.createdAt, c.lastContact, daysSince(c.lastContact), (!["Closed Won", "Closed Lost"].includes(c.stage) && daysSince(c.lastContact) >= chaseAfterDays) ? "YES" : "No"]);
                const csv = [headers, ...rows].map(r => r.join(",")).join("\n");
                const blob = new Blob([csv], { type: "text/csv" });
                const url = URL.createObjectURL(blob);
                const a = document.createElement("a"); a.href = url; a.download = `ucademy-crm-${new Date().toISOString().split("T")[0]}.csv`; a.click();
              }}>⬇ Export CSV</button>
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 16, marginBottom: 24 }}>
              {[
                { label: "Total Clients", value: clients.length, color: "#3b82f6" },
                { label: "Total Pipeline", value: `£${totalPipeline.toLocaleString()}`, color: "#c8a84b" },
                { label: "Revenue Closed", value: `£${closedWon.toLocaleString()}`, color: "#10b981" },
                { label: "Need Chase-Up", value: chaseUpClients.length, color: chaseUpClients.length > 0 ? "#f59e0b" : "#10b981" },
                { label: "Avg Deal Size", value: `£${Math.round(totalPipeline / Math.max(clients.filter(c => c.stage !== "Closed Lost").length, 1)).toLocaleString()}`, color: "#8b5cf6" },
                { label: "Lost Deals", value: clients.filter(c => c.stage === "Closed Lost").length, color: "#ef4444" },
              ].map((s, i) => (
                <div key={i} className="stat-card" style={{ padding: "16px 20px" }}>
                  <div style={{ fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.1em", color: "#4b5563", marginBottom: 6 }}>{s.label}</div>
                  <div style={{ fontSize: 28, fontWeight: 800, color: s.color, fontFamily: "'Playfair Display', serif" }}>{s.value}</div>
                </div>
              ))}
            </div>

            {/* Team table */}
            <div className="card" style={{ padding: 24, marginBottom: 20 }}>
              <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 18, color: "#c8a84b" }}>Individual Performance</div>
              <div style={{ overflowX: "auto" }}>
                <table style={{ width: "100%", borderCollapse: "collapse", fontSize: 13 }}>
                  <thead>
                    <tr style={{ color: "#4b5563", fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.08em" }}>
                      {["Salesperson", "Total", "New", "Contacted", "Proposal", "Negotiating", "Won", "Lost", "Chase-Up ⚠", "Pipeline", "Revenue", "Conv.%"].map(h => (
                        <th key={h} style={{ padding: "8px 10px", textAlign: "left", borderBottom: "1px solid #1f2937" }}>{h}</th>
                      ))}
                    </tr>
                  </thead>
                  <tbody>
                    {teamStats.map((m, i) => {
                      const colors = ["#3b82f6", "#8b5cf6", "#f59e0b", "#10b981", "#ef4444"];
                      const cr = m.total ? Math.round((m.won / m.total) * 100) : 0;
                      const myChase = chaseUpClients.filter(c => c.assignedTo === m.name).length;
                      return (
                        <tr key={m.name} style={{ borderBottom: "1px solid #0d1520" }}>
                          <td style={{ padding: "12px 10px" }}>
                            <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                              <div className="avatar" style={{ background: colors[i] + "22", color: colors[i], width: 28, height: 28, fontSize: 11 }}>{m.name[0]}</div>
                              <span style={{ fontWeight: 600 }}>{m.name}</span>
                            </div>
                          </td>
                          {[m.total,
                            clients.filter(c => c.assignedTo === m.name && c.stage === "New Lead").length,
                            clients.filter(c => c.assignedTo === m.name && c.stage === "Contacted").length,
                            clients.filter(c => c.assignedTo === m.name && c.stage === "Proposal Sent").length,
                            clients.filter(c => c.assignedTo === m.name && c.stage === "Negotiating").length,
                          ].map((v, j) => <td key={j} style={{ padding: "12px 10px", color: "#9ca3af" }}>{v}</td>)}
                          <td style={{ padding: "12px 10px", color: "#10b981", fontWeight: 700 }}>{m.won}</td>
                          <td style={{ padding: "12px 10px", color: "#ef4444" }}>{clients.filter(c => c.assignedTo === m.name && c.stage === "Closed Lost").length}</td>
                          <td style={{ padding: "12px 10px" }}>
                            <span style={{ color: myChase > 0 ? "#f59e0b" : "#10b981", fontWeight: 700 }}>{myChase > 0 ? `${myChase} ⚠` : "0 ✓"}</span>
                          </td>
                          <td style={{ padding: "12px 10px", color: "#c8a84b", fontWeight: 600 }}>£{m.pipeline.toLocaleString()}</td>
                          <td style={{ padding: "12px 10px", color: "#10b981", fontWeight: 700 }}>£{m.revenue.toLocaleString()}</td>
                          <td style={{ padding: "12px 10px" }}><span style={{ color: cr >= 50 ? "#10b981" : cr >= 25 ? "#f59e0b" : "#ef4444", fontWeight: 700 }}>{cr}%</span></td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            </div>

            {/* Chase-up table */}
            {chaseUpClients.length > 0 && (
              <div className="card" style={{ padding: 24, marginBottom: 20, border: "1px solid #f59e0b33" }}>
                <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 16, color: "#f59e0b" }}>⚠ Needs Chasing ({chaseUpClients.length})</div>
                <table style={{ width: "100%", borderCollapse: "collapse", fontSize: 13 }}>
                  <thead>
                    <tr style={{ color: "#4b5563", fontSize: 11, fontWeight: 700, textTransform: "uppercase" }}>
                      {["Client", "Company", "Stage", "Value", "Assigned To", "Last Contact", "Days Overdue", "Action"].map(h => (
                        <th key={h} style={{ padding: "8px 10px", textAlign: "left", borderBottom: "1px solid #1f2937" }}>{h}</th>
                      ))}
                    </tr>
                  </thead>
                  <tbody>
                    {chaseUpClients.map(c => {
                      const days = daysSince(c.lastContact);
                      return (
                        <tr key={c.id} style={{ borderBottom: "1px solid #0d1520" }}>
                          <td style={{ padding: "10px 10px", fontWeight: 600 }}>{c.name}</td>
                          <td style={{ padding: "10px 10px", color: "#6b7280" }}>{c.company}</td>
                          <td style={{ padding: "10px 10px" }}><span className="badge" style={{ background: STAGE_COLORS[c.stage] + "22", color: STAGE_COLORS[c.stage] }}>{c.stage}</span></td>
                          <td style={{ padding: "10px 10px", color: "#c8a84b", fontWeight: 700 }}>£{c.value.toLocaleString()}</td>
                          <td style={{ padding: "10px 10px", color: "#9ca3af" }}>{c.assignedTo}</td>
                          <td style={{ padding: "10px 10px", color: "#6b7280" }}>{c.lastContact}</td>
                          <td style={{ padding: "10px 10px" }}><span style={{ color: days >= 7 ? "#ef4444" : days >= 5 ? "#f59e0b" : "#3b82f6", fontWeight: 700 }}>{days}d</span></td>
                          <td style={{ padding: "10px 10px" }}><button className="btn-green" onClick={() => markContacted(c)}>✓ Mark Called</button></td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            )}

            <div className="card" style={{ padding: 24 }}>
              <div style={{ fontWeight: 700, fontSize: 15, marginBottom: 16, color: "#c8a84b" }}>Full Client Register</div>
              <div style={{ overflowX: "auto" }}>
                <table style={{ width: "100%", borderCollapse: "collapse", fontSize: 12 }}>
                  <thead>
                    <tr style={{ color: "#4b5563", fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.08em" }}>
                      {["Name", "Company", "Course", "Value", "Stage", "Source", "Assigned", "Last Contact", "Days"].map(h => (
                        <th key={h} style={{ padding: "8px 10px", textAlign: "left", borderBottom: "1px solid #1f2937" }}>{h}</th>
                      ))}
                    </tr>
                  </thead>
                  <tbody>
                    {clients.map(c => {
                      const days = daysSince(c.lastContact);
                      const needsChase = !["Closed Won", "Closed Lost"].includes(c.stage) && days >= chaseAfterDays;
                      return (
                        <tr key={c.id} style={{ borderBottom: "1px solid #0d1520" }}>
                          <td style={{ padding: "10px 10px", fontWeight: 600 }}>{c.name}</td>
                          <td style={{ padding: "10px 10px", color: "#6b7280" }}>{c.company}</td>
                          <td style={{ padding: "10px 10px", color: "#9ca3af" }}>{c.course}</td>
                          <td style={{ padding: "10px 10px", color: "#c8a84b", fontWeight: 700 }}>£{c.value.toLocaleString()}</td>
                          <td style={{ padding: "10px 10px" }}><span className="badge" style={{ background: STAGE_COLORS[c.stage] + "22", color: STAGE_COLORS[c.stage] }}>{c.stage}</span></td>
                          <td style={{ padding: "10px 10px", color: "#6b7280" }}>{c.source}</td>
                          <td style={{ padding: "10px 10px", color: "#9ca3af" }}>{c.assignedTo}</td>
                          <td style={{ padding: "10px 10px", color: "#4b5563" }}>{c.lastContact}</td>
                          <td style={{ padding: "10px 10px" }}><span style={{ color: needsChase ? "#f59e0b" : "#374151", fontWeight: needsChase ? 700 : 400 }}>{days}d{needsChase ? " ⚠" : ""}</span></td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {/* ─── PRESENT ─── */}
        {activeTab === "present" && (
          <div className="fade-in">
            <div style={{ textAlign: "center", marginBottom: 32 }}>
              <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 28, fontWeight: 700, marginBottom: 4 }}>Presentation Mode</div>
              <div style={{ color: "#4b5563", fontSize: 14 }}>Full-screen slides for team meetings & reviews</div>
            </div>
            <div style={{ display: "flex", gap: 16, justifyContent: "center", flexWrap: "wrap" }}>
              {[{ label: "📊 Overview", slide: 0 }, { label: "🔵 Pipeline", slide: 1 }, { label: "👥 Team", slide: 2 }, { label: "🏆 Top Deals", slide: 3 }].map(({ label, slide }) => (
                <button key={slide} className="btn-gold" style={{ padding: "14px 28px", fontSize: 15 }} onClick={() => { setPresentSlide(slide); document.getElementById("present-fs").requestFullscreen?.(); }}>
                  {label}
                </button>
              ))}
            </div>
            <div id="present-fs" style={{ marginTop: 32, background: "#050810", borderRadius: 16, overflow: "hidden", border: "1px solid #1f2937" }}>
              <PresentationView slide={presentSlide} clients={clients} totalPipeline={totalPipeline} closedWon={closedWon} activeLeads={activeLeads} convRate={convRate} stageCounts={stageCounts} teamStats={teamStats} chaseUpCount={chaseUpClients.length} onNext={() => setPresentSlide(s => Math.min(s + 1, 3))} onPrev={() => setPresentSlide(s => Math.max(s - 1, 0))} />
            </div>
          </div>
        )}
      </div>

      {/* Client Detail Modal */}
      {selectedClient && !editClient && (
        <div className="modal-overlay" onClick={() => setSelectedClient(null)}>
          <div className="modal" onClick={e => e.stopPropagation()}>
            <div style={{ padding: "28px 28px 20px", borderBottom: "1px solid #1f2937", display: "flex", justifyContent: "space-between", alignItems: "flex-start", gap: 12, flexWrap: "wrap" }}>
              <div>
                <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 22, fontWeight: 700 }}>{selectedClient.name}</div>
                <div style={{ color: "#6b7280", fontSize: 14, marginTop: 2 }}>{selectedClient.company}</div>
                {!["Closed Won", "Closed Lost"].includes(selectedClient.stage) && daysSince(selectedClient.lastContact) >= chaseAfterDays && (
                  <div style={{ marginTop: 8, fontSize: 12, color: "#f59e0b", background: "#f59e0b18", display: "inline-block", padding: "3px 10px", borderRadius: 8, fontWeight: 700 }}>
                    ⚠ Chase-up overdue — {daysSince(selectedClient.lastContact)} days since last contact
                  </div>
                )}
              </div>
              <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
                <button className="btn-green" style={{ padding: "8px 14px", fontSize: 12 }} onClick={() => { markContacted(selectedClient); setSelectedClient(null); }}>✓ Mark Called</button>
                <button className="btn-outline" onClick={() => setEditClient({ ...selectedClient })}>Edit</button>
                <button className="btn-red" onClick={() => handleDeleteClient(selectedClient.id)}>Delete</button>
                <button className="btn-outline" onClick={() => setSelectedClient(null)}>✕</button>
              </div>
            </div>
            <div style={{ padding: 28 }}>
              <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 20, marginBottom: 20 }}>
                {[
                  { l: "Email", v: selectedClient.email || "—" },
                  { l: "Phone", v: selectedClient.phone || "—" },
                  { l: "Course", v: selectedClient.course },
                  { l: "Deal Value", v: `£${selectedClient.value.toLocaleString()}` },
                  { l: "Source", v: selectedClient.source },
                  { l: "Assigned To", v: selectedClient.assignedTo },
                  { l: "Created", v: selectedClient.createdAt },
                  { l: "Last Contact", v: `${selectedClient.lastContact} (${daysSince(selectedClient.lastContact)}d ago)` },
                ].map(({ l, v }) => (
                  <div key={l}>
                    <div style={{ fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.08em", color: "#4b5563", marginBottom: 4 }}>{l}</div>
                    <div style={{ fontSize: 14, color: "#e8e0d0" }}>{v}</div>
                  </div>
                ))}
              </div>
              <div style={{ marginBottom: 20 }}>
                <div style={{ fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.08em", color: "#4b5563", marginBottom: 6 }}>Stage</div>
                <span className="badge" style={{ background: STAGE_COLORS[selectedClient.stage] + "22", color: STAGE_COLORS[selectedClient.stage], fontSize: 13, padding: "5px 14px" }}>{selectedClient.stage}</span>
              </div>
              {selectedClient.notes && (
                <div>
                  <div style={{ fontSize: 11, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.08em", color: "#4b5563", marginBottom: 6 }}>Notes</div>
                  <div style={{ background: "#0d1520", borderRadius: 8, padding: "12px 14px", fontSize: 14, color: "#9ca3af", lineHeight: 1.5 }}>{selectedClient.notes}</div>
                </div>
              )}
            </div>
          </div>
        </div>
      )}

      {/* Edit Modal */}
      {editClient && (
        <div className="modal-overlay">
          <div className="modal">
            <div style={{ padding: "24px 28px 16px", borderBottom: "1px solid #1f2937", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
              <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 20, fontWeight: 700 }}>Edit Client</div>
              <button className="btn-outline" onClick={() => setEditClient(null)}>✕</button>
            </div>
            <form onSubmit={handleUpdateClient} style={{ padding: 28 }}>
              <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14 }}>
                {[{ l: "Full Name", k: "name", t: "text" }, { l: "Company", k: "company", t: "text" }, { l: "Email", k: "email", t: "email" }, { l: "Phone", k: "phone", t: "text" }, { l: "Deal Value (£)", k: "value", t: "number" }, { l: "Last Contact", k: "lastContact", t: "date" }].map(({ l, k, t }) => (
                  <div key={k}>
                    <label className="form-label">{l}</label>
                    <input className="form-input" type={t} value={editClient[k]} onChange={e => setEditClient({ ...editClient, [k]: e.target.value })} />
                  </div>
                ))}
                <div>
                  <label className="form-label">Stage</label>
                  <select className="form-input" value={editClient.stage} onChange={e => setEditClient({ ...editClient, stage: e.target.value })}>
                    {STAGES.map(s => <option key={s}>{s}</option>)}
                  </select>
                </div>
                <div>
                  <label className="form-label">Assigned To</label>
                  <select className="form-input" value={editClient.assignedTo} onChange={e => setEditClient({ ...editClient, assignedTo: e.target.value })}>
                    {TEAM_MEMBERS.filter(m => m !== "All").map(m => <option key={m}>{m}</option>)}
                  </select>
                </div>
              </div>
              <div style={{ marginTop: 14 }}>
                <label className="form-label">Notes</label>
                <textarea className="form-input" rows={3} value={editClient.notes} onChange={e => setEditClient({ ...editClient, notes: e.target.value })} style={{ resize: "vertical" }} />
              </div>
              <div style={{ display: "flex", gap: 10, justifyContent: "flex-end", marginTop: 20 }}>
                <button type="button" className="btn-outline" onClick={() => setEditClient(null)}>Cancel</button>
                <button type="submit" className="btn-gold">Save Changes</button>
              </div>
            </form>
          </div>
        </div>
      )}
    </div>
  );
}

function PresentationView({ slide, clients, totalPipeline, closedWon, activeLeads, convRate, stageCounts, teamStats, chaseUpCount, onNext, onPrev }) {
  const slides = [
    <div key={0} style={{ textAlign: "center", width: "100%", padding: "60px 80px" }}>
      <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 16, letterSpacing: "0.3em", color: "#c8a84b", textTransform: "uppercase", marginBottom: 20 }}>ClientDrive Sales</div>
      <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 64, fontWeight: 900, color: "#ffffff", lineHeight: 1.1, marginBottom: 40 }}>Performance<br /><span style={{ color: "#c8a84b" }}>Overview</span></div>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 24, maxWidth: 900, margin: "0 auto" }}>
        {[
          { l: "Pipeline", v: `£${totalPipeline.toLocaleString()}`, c: "#c8a84b" },
          { l: "Revenue Won", v: `£${closedWon.toLocaleString()}`, c: "#10b981" },
          { l: "Active Leads", v: activeLeads, c: "#3b82f6" },
          { l: "Chase-Ups", v: chaseUpCount, c: chaseUpCount > 0 ? "#f59e0b" : "#10b981" },
        ].map(({ l, v, c }) => (
          <div key={l} style={{ background: "rgba(255,255,255,0.04)", borderRadius: 16, padding: "28px 20px", border: `1px solid ${c}33` }}>
            <div style={{ fontSize: 42, fontWeight: 900, color: c, fontFamily: "'Playfair Display', serif" }}>{v}</div>
            <div style={{ fontSize: 13, color: "#6b7280", marginTop: 8, letterSpacing: "0.05em", textTransform: "uppercase" }}>{l}</div>
          </div>
        ))}
      </div>
    </div>,

    <div key={1} style={{ width: "100%", padding: "50px 80px" }}>
      <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 44, fontWeight: 900, color: "#ffffff", marginBottom: 8 }}>Sales <span style={{ color: "#c8a84b" }}>Pipeline</span></div>
      <div style={{ color: "#4b5563", marginBottom: 40 }}>Current status across all deals</div>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 20 }}>
        {stageCounts.map(({ stage, count, value }) => (
          <div key={stage} style={{ background: "rgba(255,255,255,0.03)", border: `1px solid ${STAGE_COLORS[stage]}44`, borderRadius: 16, padding: "24px 20px" }}>
            <div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 14 }}>
              <div style={{ width: 12, height: 12, borderRadius: "50%", background: STAGE_COLORS[stage] }} />
              <span style={{ fontSize: 14, fontWeight: 700, color: STAGE_COLORS[stage] }}>{stage}</span>
            </div>
            <div style={{ fontSize: 40, fontWeight: 900, fontFamily: "'Playfair Display', serif", color: "#fff" }}>{count}</div>
            <div style={{ fontSize: 18, color: STAGE_COLORS[stage], marginTop: 4, fontWeight: 700 }}>£{value.toLocaleString()}</div>
          </div>
        ))}
      </div>
    </div>,

    <div key={2} style={{ width: "100%", padding: "50px 80px" }}>
      <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 44, fontWeight: 900, color: "#ffffff", marginBottom: 8 }}>Team <span style={{ color: "#c8a84b" }}>Performance</span></div>
      <div style={{ color: "#4b5563", marginBottom: 40 }}>Individual contributions at a glance</div>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(5, 1fr)", gap: 16 }}>
        {teamStats.map((m, i) => {
          const colors = ["#3b82f6", "#8b5cf6", "#f59e0b", "#10b981", "#ef4444"];
          const cr = m.total ? Math.round((m.won / m.total) * 100) : 0;
          return (
            <div key={m.name} style={{ background: "rgba(255,255,255,0.03)", borderRadius: 16, padding: "24px 16px", textAlign: "center", border: `1px solid ${colors[i]}33` }}>
              <div style={{ width: 52, height: 52, borderRadius: "50%", background: colors[i] + "22", color: colors[i], display: "flex", alignItems: "center", justifyContent: "center", fontSize: 22, fontWeight: 800, margin: "0 auto 14px" }}>{m.name[0]}</div>
              <div style={{ fontSize: 18, fontWeight: 700, color: "#fff", marginBottom: 12 }}>{m.name}</div>
              <div style={{ fontSize: 28, fontWeight: 900, color: "#10b981", fontFamily: "'Playfair Display', serif" }}>£{m.revenue.toLocaleString()}</div>
              <div style={{ fontSize: 12, color: "#4b5563" }}>Revenue</div>
              <div style={{ marginTop: 12, fontSize: 24, fontWeight: 800, color: colors[i] }}>{cr}%</div>
              <div style={{ fontSize: 12, color: "#4b5563" }}>Win rate</div>
            </div>
          );
        })}
      </div>
    </div>,

    <div key={3} style={{ width: "100%", padding: "50px 80px" }}>
      <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 44, fontWeight: 900, color: "#ffffff", marginBottom: 8 }}>Top <span style={{ color: "#c8a84b" }}>Opportunities</span></div>
      <div style={{ color: "#4b5563", marginBottom: 40 }}>Highest value deals in the pipeline</div>
      <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
        {[...clients].filter(c => c.stage !== "Closed Lost").sort((a, b) => b.value - a.value).slice(0, 6).map((c, i) => (
          <div key={c.id} style={{ display: "flex", alignItems: "center", gap: 20, background: "rgba(255,255,255,0.03)", borderRadius: 12, padding: "16px 24px", border: "1px solid #1f2937" }}>
            <div style={{ fontSize: 28, fontWeight: 900, fontFamily: "'Playfair Display', serif", color: i === 0 ? "#c8a84b" : "#374151", minWidth: 36 }}>#{i + 1}</div>
            <div style={{ flex: 1 }}>
              <div style={{ fontSize: 18, fontWeight: 700 }}>{c.name}</div>
              <div style={{ fontSize: 13, color: "#6b7280" }}>{c.company} · {c.course}</div>
            </div>
            <span style={{ padding: "4px 12px", borderRadius: 20, fontSize: 12, fontWeight: 700, background: STAGE_COLORS[c.stage] + "22", color: STAGE_COLORS[c.stage] }}>{c.stage}</span>
            <div style={{ textAlign: "right" }}>
              <div style={{ fontSize: 24, fontWeight: 900, color: "#c8a84b", fontFamily: "'Playfair Display', serif" }}>£{c.value.toLocaleString()}</div>
              <div style={{ fontSize: 12, color: "#4b5563" }}>{c.assignedTo}</div>
            </div>
          </div>
        ))}
      </div>
    </div>,
  ];

  return (
    <div style={{ position: "relative", background: "#050810", minHeight: 520 }}>
      <div style={{ color: "#e8e0d0" }}>{slides[slide]}</div>
      <div style={{ position: "absolute", bottom: 20, left: "50%", transform: "translateX(-50%)", display: "flex", alignItems: "center", gap: 16 }}>
        <button className="btn-outline" style={{ opacity: slide === 0 ? 0.3 : 1 }} onClick={onPrev} disabled={slide === 0}>← Prev</button>
        <div style={{ display: "flex", gap: 6 }}>
          {slides.map((_, i) => <div key={i} style={{ width: 8, height: 8, borderRadius: "50%", background: i === slide ? "#c8a84b" : "#1f2937" }} />)}
        </div>
        <button className="btn-outline" style={{ opacity: slide === slides.length - 1 ? 0.3 : 1 }} onClick={onNext} disabled={slide === slides.length - 1}>Next →</button>
      </div>
    </div>
  );
}


    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<ClientDriveCRM />);
  </script>
</body>
</html>
