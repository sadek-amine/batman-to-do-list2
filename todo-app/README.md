# Welcome to your Expo app 👋

This is an [Expo](https://expo.dev) project created with [`create-expo-app`](https://www.npmjs.com/package/create-expo-app).

## Get started

1. Install dependencies

   ```bash
   npm install
   ```

2. Start the app

   ```bash
   npx expo start
   ```

In the output, you'll find options to open the app in a

- [development build](https://docs.expo.dev/develop/development-builds/introduction/)
- [Android emulator](https://docs.expo.dev/workflow/android-studio-emulator/)
- [iOS simulator](https://docs.expo.dev/workflow/ios-simulator/)
- [Expo Go](https://expo.dev/go), a limited sandbox for trying out app development with Expo

You can start developing by editing the files inside the **app** directory. This project uses [file-based routing](https://docs.expo.dev/router/introduction).

## Get a fresh project

When you're ready, run:

```bash
npm run reset-project
```

This command will move the starter code to the **app-example** directory and create a blank **app** directory where you can start developing.

## Learn more

To learn more about developing your project with Expo, look at the following resources:

- [Expo documentation](https://docs.expo.dev/): Learn fundamentals, or go into advanced topics with our [guides](https://docs.expo.dev/guides).
- [Learn Expo tutorial](https://docs.expo.dev/tutorial/introduction/): Follow a step-by-step tutorial where you'll create a project that runs on Android, iOS, and the web.

## Join the community

Join our community of developers creating universal apps.

- [Expo on GitHub](https://github.com/expo/expo): View our open source platform and contribute.
- [Discord community](https://chat.expo.dev): Chat with Expo users and ask questions.













const exampleTasks: Task[] = [
  {
    id: "1",
    title: "Finish React Native To-Do App",
    completed: false,
    category: defaultCategories[7], // 🚀 Projects
    date: "2026-03-31",
  },
  {
    id: "2",
    title: "Go to the gym",
    completed: true,
    category: defaultCategories[3], // 💪 Fitness
    date: "2026-03-30",
  },
  {
    id: "3",
    title: "Study AI concepts",
    completed: false,
    category: defaultCategories[1], // 📚 Study
    date: "2026-04-01",
  },
  {
    id: "4",
    title: "Push code to GitHub",
    completed: false,
    category: defaultCategories[6], // 💻 Coding
    date: "2026-03-31",
  },
  {
    id: "5",
    title: "Buy groceries",
    completed: true,
    category: defaultCategories[4], // 🛒 Shopping
    date: "2026-03-29",
  },
];


























































import React, { useState, useEffect, useRef } from "react";
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
  Animated,
  Dimensions,
} from "react-native";
import axios from "axios";

/* ---------------- API ---------------- */
const API_URL = "http://192.168.1.11:3000";
const SCREEN_WIDTH = Dimensions.get("window").width;

/* ---------------- TYPES ---------------- */
type Category = {
  id: number;
  name: string;
};

type Task = {
  id: number;
  title: string;
  completed: boolean;
  category_name?: string;
  category_id?: number;
  date: string;
};

/* ---------------- EMOJIS ---------------- */
const emojiMap: Record<string, string> = {
  Work: "💼",
  Study: "📚",
  Health: "🏃‍♂️",
  Fitness: "💪",
  Shopping: "🛒",
  Finance: "💰",
  Coding: "💻",
  Projects: "🚀",
  Reading: "📖",
  Travel: "✈️",
  Other: "📌",
};

/* ---------------- CATEGORIES ---------------- */
const defaultCategories: Category[] = [
  { id: 1, name: "Work" },
  { id: 2, name: "Study" },
  { id: 3, name: "Health" },
  { id: 4, name: "Fitness" },
  { id: 5, name: "Shopping" },
  { id: 6, name: "Finance" },
  { id: 7, name: "Coding" },
  { id: 8, name: "Projects" },
  { id: 9, name: "Reading" },
  { id: 10, name: "Travel" },
];

/* ================= COMPONENT ================= */
export default function Index() {
  const [task, setTask] = useState("");
  const [tasks, setTasks] = useState<Task[]>([]);
  const [selectedCategory, setSelectedCategory] = useState(defaultCategories[0]);
  const [date, setDate] = useState(new Date().toISOString().split("T")[0]);

  /* 🦇 BATS STATE */
  const [bats, setBats] = useState<
    { id: number; x: number; anim: Animated.Value }[]
  >([]);

  /* ---------------- FETCH TASKS ---------------- */
  const fetchTasks = async () => {
    try {
      const res = await axios.get(`${API_URL}/tasks`);
      setTasks(res.data);
    } catch (err) {
      console.log(err);
    }
  };

  useEffect(() => {
    fetchTasks();
  }, []);

  /* ---------------- ADD TASK ---------------- */
  const addTask = async () => {
    if (!task.trim()) return;

    await axios.post(`${API_URL}/tasks`, {
      title: task,
      category_id: selectedCategory.id,
      date,
    });

    setTask("");
    fetchTasks();
  };

  /* 🦇 SPAWN BATS ANIMATION */
  const spawnBats = () => {
    const newBats = Array.from({ length: 80 }).map((_, i) => {
      const anim = new Animated.Value(0);

      const bat = {
        id: Date.now() + i,
        x: Math.random() * (SCREEN_WIDTH - 40),
        anim,
      };

      Animated.timing(anim, {
        toValue: 1,
        duration: 1000,
        useNativeDriver: true,
      }).start();

      return bat;
    });

    setBats(newBats);

    setTimeout(() => {
      setBats([]);
    }, 1200);
  };

  /* ---------------- TOGGLE ---------------- */
  const toggleTask = async (id: number) => {
    const current = tasks.find((t) => t.id === id);

    // 🦇 only when completing (false -> true)
    if (current && !current.completed) {
      spawnBats();
    }

    await axios.put(`${API_URL}/tasks/${id}`);
    fetchTasks();
  };

  /* ---------------- DELETE ---------------- */
  const deleteTask = async (id: number) => {
    await axios.delete(`${API_URL}/tasks/${id}`);
    fetchTasks();
  };

  /* ================= UI ================= */
  return (
    <View style={styles.container}>
      <Text style={styles.title}>🦇 Batman Tasks</Text>

      {/* CATEGORY */}
      <FlatList
        horizontal
        data={defaultCategories}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <TouchableOpacity
            onPress={() => setSelectedCategory(item)}
            style={[
              styles.category,
              selectedCategory.id === item.id && styles.selectedCategory,
            ]}
          >
            <Text style={{ color: "#fff" }}>
              {emojiMap[item.name]} {item.name}
            </Text>
          </TouchableOpacity>
        )}
      />

      {/* DATE */}
      <TextInput
        value={date}
        onChangeText={setDate}
        style={styles.input}
      />

      {/* INPUT */}
      <View style={styles.inputContainer}>
        <TextInput
          placeholder="Add task..."
          value={task}
          onChangeText={setTask}
          style={styles.input}
          placeholderTextColor="#94a3b8"
        />
        <TouchableOpacity onPress={addTask} style={styles.addButton}>
          <Text style={styles.addText}>+</Text>
        </TouchableOpacity>
      </View>

      {/* TASKS */}
      <FlatList
        data={tasks}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.taskItem}>
            <View>
              <Text
                style={[
                  styles.taskText,
                  item.completed && styles.completed,
                ]}
              >
                {item.title}
              </Text>

              <Text style={styles.meta}>
                {emojiMap[item.category_name ?? ""] ?? "📌"}{" "}
                {item.category_name} • {item.date}
              </Text>
            </View>

            <View style={{ flexDirection: "row", gap: 10 }}>
              <TouchableOpacity onPress={() => toggleTask(item.id)}>
                <Text style={{ fontSize: 18 }}>
                  {item.completed ? "✅" : "⭕"}
                </Text>
              </TouchableOpacity>

              <TouchableOpacity onPress={() => deleteTask(item.id)}>
                <Text>🗑️</Text>
              </TouchableOpacity>
            </View>
          </View>
        )}
      />

      {/* 🦇 BATS OVERLAY */}
      {bats.map((bat) => (
        <Animated.View
          key={bat.id}
          style={{
            position: "absolute",
            bottom: 80,
            left: bat.x,
            transform: [
              {
                translateY: bat.anim.interpolate({
                  inputRange: [0, 1],
                  outputRange: [0, -300],
                }),
              },
              {
                scale: bat.anim.interpolate({
                  inputRange: [0, 1],
                  outputRange: [0.5, 1.5],
                }),
              },
            ],
            opacity: bat.anim.interpolate({
              inputRange: [0, 1],
              outputRange: [1, 0],
            }),
          }}
        >
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
          <Text style={{ fontSize: 24 }}>🦇🦇</Text>
        </Animated.View>
      ))}
    </View>
  );
}

/* ================= STYLES ================= */
const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: "#0f172a", padding: 20 },
  title: { fontSize: 28, color: "#fff", marginBottom: 20 },

  inputContainer: { flexDirection: "row", marginVertical: 10 },

  input: {
    flex: 1,
    backgroundColor: "#1e293b",
    padding: 10,
    borderRadius: 10,
    color: "#fff",
    marginVertical: 5,
  },

  addButton: {
    backgroundColor: "#22c55e",
    padding: 10,
    borderRadius: 10,
    marginLeft: 10,
  },

  addText: { color: "#fff", fontSize: 18 },

  category: {
    padding: 10,
    marginRight: 10,
    backgroundColor: "#1e293b",
    borderRadius: 10,
  },

  selectedCategory: { backgroundColor: "#22c55e" },

  taskItem: {
    backgroundColor: "#1e293b",
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    flexDirection: "row",
    justifyContent: "space-between",
  },

  taskText: { color: "#fff" },
  completed: { textDecorationLine: "line-through", color: "#94a3b8" },

  meta: { color: "#94a3b8", fontSize: 12 },
});































sever.js



process.env.LANG = "en_US.UTF-8";
const express = require("express");
const { Pool } = require("pg");
const cors = require("cors");

const categoryMap = [
  { id: 1, name: "Work", emoji: "💼" },
  { id: 2, name: "Study", emoji: "📚" },
  { id: 3, name: "Health", emoji: "🏃‍♂️" },
  { id: 4, name: "Fitness", emoji: "💪" },
  { id: 5, name: "Shopping", emoji: "🛒" },
  { id: 6, name: "Finance", emoji: "💰" },
  { id: 7, name: "Coding", emoji: "💻" },
  { id: 8, name: "Projects", emoji: "🚀" },
  { id: 9, name: "Reading", emoji: "📖" },
  { id: 10, name: "Travel", emoji: "✈️" },
];

const app = express();
app.use(cors());
app.use(express.json());

const pool = new Pool({
  user: "postgres",
  host: "localhost",
  database: "todo_app",
  password: "sadek2025",
  port: 5432,
  client_encoding: "UTF8",
});

/* ---------------- GET TASKS ---------------- */
app.get("/tasks", async (req, res) => {
  try {
    const result = await pool.query(`
      SELECT * FROM tasks ORDER BY id DESC
    `);

    const enriched = result.rows.map(task => {
      const category = categoryMap.find(c => c.id === task.category_id);

      return {
        ...task,
        category_name: category?.name || "Unknown",
        category_emoji: category?.emoji || "📌",
      };
    });

    res.json(enriched);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Server error" });
  }
});
/* ---------------- ADD TASK ---------------- */
app.post("/tasks", async (req, res) => {
  const { title, category_id, date } = req.body;

  const result = await pool.query(
    `INSERT INTO tasks (title, category_id, date, completed)
     VALUES ($1, $2, $3, false)
     RETURNING *`,
    [title, category_id, date]
  );

  res.json(result.rows[0]);
});

/* ---------------- TOGGLE TASK ---------------- */
app.put("/tasks/:id", async (req, res) => {
  try {
    const { id } = req.params;

    const result = await pool.query(
      `UPDATE tasks 
       SET completed = NOT completed 
       WHERE id = $1 
       RETURNING *`,
      [id]
    );

    res.json(result.rows[0]);
  } catch (err) {
    console.error("PUT /tasks error:", err);
    res.status(500).json({ error: "Server error" });
  }
});

/* ---------------- DELETE TASK ---------------- */
app.delete("/tasks/:id", async (req, res) => {
  try {
    const { id } = req.params;

    await pool.query("DELETE FROM tasks WHERE id=$1", [id]);

    res.json({ message: "Deleted" });
  } catch (err) {
    console.error("DELETE /tasks error:", err);
    res.status(500).json({ error: "Server error" });
  }
});

/* ---------------- START SERVER ---------------- */
app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});




























































claude   


mport React, { useState, useEffect, useCallback, useRef } from "react";
import {
  View, Text, TextInput, TouchableOpacity, FlatList,
  StyleSheet, Animated, Dimensions, Modal, ScrollView,
  KeyboardAvoidingView, Platform, StatusBar,
} from "react-native";
import axios from "axios";

const API_URL = "http://192.168.1.24:3000";
const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get("window");

/* ── TYPES ── */
type Category = { id: number; name: string };
type Task = {
  id: number; title: string; completed: boolean;
  category_name?: string; category_id?: number; date: string;
};
type Screen = "tasks" | "stats";
type FilterStatus = "all" | "done" | "pending";

/* ── THEME ── */
const C = {
  bg:       "#060608",
  surface:  "#0d0d14",
  card:     "#111118",
  border:   "#1e1e2e",
  gold:     "#f5c518",
  goldDim:  "#a8860f",
  white:    "#f0f0f0",
  muted:    "#555570",
  danger:   "#c0392b",
};

/* ── CATEGORIES ── */
const defaultCategories: Category[] = [
  { id: 1,  name: "Work"     },
  { id: 2,  name: "Study"    },
  { id: 3,  name: "Health"   },
  { id: 4,  name: "Fitness"  },
  { id: 5,  name: "Shopping" },
  { id: 6,  name: "Finance"  },
  { id: 7,  name: "Coding"   },
  { id: 8,  name: "Projects" },
  { id: 9,  name: "Reading"  },
  { id: 10, name: "Travel"   },
];

const emojiMap: Record<string, string> = {
  Work: "💼", Study: "📚", Health: "🏃", Fitness: "💪",
  Shopping: "🛒", Finance: "💰", Coding: "💻", Projects: "🚀",
  Reading: "📖", Travel: "✈️", Other: "📌",
};

const catColors: Record<string, string> = {
  Work: "#c0392b",    Study: "#2980b9",    Health: "#27ae60",
  Fitness: "#8e44ad", Shopping: "#e67e22", Finance: "#f39c12",
  Coding: "#16a085",  Projects: "#f5c518", Reading: "#2c3e50",
  Travel: "#1abc9c",
};

/* ── BAT SYMBOL ── */
const BatSymbol = () => (
  <View style={sym.wrap}>
    <View style={sym.body} />
    <View style={[sym.wing, sym.wingL]} />
    <View style={[sym.wing, sym.wingR]} />
    <View style={sym.earL} />
    <View style={sym.earR} />
  </View>
);
const sym = StyleSheet.create({
  wrap:  { width: 52, height: 28, alignItems: "center", justifyContent: "center" },
  body:  { width: 26, height: 14, backgroundColor: C.gold, borderRadius: 7, position: "absolute" },
  wing:  { width: 20, height: 20, backgroundColor: C.gold, borderRadius: 10, position: "absolute", top: 4 },
  wingL: { left: 2,  transform: [{ scaleX: 1.3 }, { scaleY: 0.7 }] },
  wingR: { right: 2, transform: [{ scaleX: 1.3 }, { scaleY: 0.7 }] },
  earL:  { width: 6, height: 10, backgroundColor: C.gold, position: "absolute", top: -2, left: 14, transform: [{ rotate: "-20deg" }], borderRadius: 2 },
  earR:  { width: 6, height: 10, backgroundColor: C.gold, position: "absolute", top: -2, right: 14, transform: [{ rotate: "20deg"  }], borderRadius: 2 },
});

const GoldLine = () => (
  <View style={{ height: 1, backgroundColor: C.gold, opacity: 0.2, marginVertical: 10 }} />
);

/* ══════════════════════════════════════
   MAIN
══════════════════════════════════════ */
export default function Index() {
  const [screen, setScreen]                     = useState<Screen>("tasks");
  const [task, setTask]                         = useState("");
  const [tasks, setTasks]                       = useState<Task[]>([]);
  const [selectedCategory, setSelectedCategory] = useState(defaultCategories[0]);
  const [date, setDate]                         = useState(new Date().toISOString().split("T")[0]);
  const [search, setSearch]                     = useState("");
  const [filterStatus, setFilterStatus]         = useState<FilterStatus>("all");
  const [filterCat, setFilterCat]               = useState<number | null>(null);
  const [editTask, setEditTask]                 = useState<Task | null>(null);
  const [editTitle, setEditTitle]               = useState("");
  const [editDate, setEditDate]                 = useState("");
  const [editCat, setEditCat]                   = useState(defaultCategories[0]);
  const [bats, setBats]                         = useState<{ id: number; x: number; anim: Animated.Value }[]>([]);

  const pulseAnim  = useRef(new Animated.Value(1)).current;
  const headerAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.timing(headerAnim, { toValue: 1, duration: 900, useNativeDriver: true }).start();
    Animated.loop(
      Animated.sequence([
        Animated.timing(pulseAnim, { toValue: 1.12, duration: 1400, useNativeDriver: true }),
        Animated.timing(pulseAnim, { toValue: 1,    duration: 1400, useNativeDriver: true }),
      ])
    ).start();
  }, []);

  const fetchTasks = useCallback(async () => {
    try {
      const res = await axios.get(`${API_URL}/tasks`);
      setTasks(res.data);
    } catch (err) { console.log(err); }
  }, []);

  useEffect(() => { fetchTasks(); }, [fetchTasks]);

  const addTask = async () => {
    if (!task.trim()) return;
    await axios.post(`${API_URL}/tasks`, { title: task, category_id: selectedCategory.id, date });
    setTask(""); fetchTasks();
  };

  const toggleTask = useCallback(async (id: number) => {
    const current = tasks.find((t) => t.id === id);
    if (current && !current.completed) spawnBats();
    await axios.put(`${API_URL}/tasks/${id}`);
    fetchTasks();
  }, [tasks]);

  const deleteTask = useCallback(async (id: number) => {
    await axios.delete(`${API_URL}/tasks/${id}`);
    fetchTasks();
  }, []);

  const saveEdit = async () => {
    if (!editTask) return;
    await axios.patch(`${API_URL}/tasks/${editTask.id}`, {
      title: editTitle, date: editDate, category_id: editCat.id,
    });
    setEditTask(null); fetchTasks();
  };

  const spawnBats = () => {
    const newBats = Array.from({ length: 24 }).map((_, i) => {
      const anim = new Animated.Value(0);
      Animated.timing(anim, { toValue: 1, duration: 1200, useNativeDriver: true }).start();
      return { id: Date.now() + i, x: Math.random() * (SCREEN_WIDTH - 40), anim };
    });
    setBats(newBats);
    setTimeout(() => setBats([]), 1400);
  };

  const filtered = tasks.filter((t) => {
    const matchSearch = t.title.toLowerCase().includes(search.toLowerCase());
    const matchStatus = filterStatus === "all" ? true : filterStatus === "done" ? t.completed : !t.completed;
    const matchCatF   = filterCat === null || t.category_id === filterCat;
    return matchSearch && matchStatus && matchCatF;
  });

  const total      = tasks.length;
  const done       = tasks.filter((t) => t.completed).length;
  const rate       = total === 0 ? 0 : Math.round((done / total) * 100);
  const byCategory = defaultCategories.map((c) => ({
    ...c,
    count:     tasks.filter((t) => t.category_id === c.id).length,
    doneCount: tasks.filter((t) => t.category_id === c.id && t.completed).length,
  })).filter((c) => c.count > 0);

  /* ── TASK RENDER ── */
  const renderTask = useCallback(({ item }: { item: Task }) => {
    const color = catColors[item.category_name ?? ""] ?? C.gold;
    return (
      <View style={[s.taskCard, { borderLeftColor: color, borderLeftWidth: 3 }]}>
        <View style={{ flex: 1 }}>
          <Text style={[s.taskTitle, item.completed && s.strikethrough]}>
            {item.title}
          </Text>
          <View style={s.taskMeta}>
            <View style={[s.catBadge, { backgroundColor: color + "22", borderColor: color + "55" }]}>
              <Text style={[s.catBadgeText, { color }]}>
                {emojiMap[item.category_name ?? ""] ?? "📌"} {item.category_name}
              </Text>
            </View>
            <Text style={s.dateText}>📅 {item.date}</Text>
          </View>
        </View>

        <View style={s.taskActions}>
          <TouchableOpacity
            onPress={() => {
              setEditTask(item);
              setEditTitle(item.title);
              setEditDate(item.date);
              setEditCat(defaultCategories.find(c => c.id === item.category_id) ?? defaultCategories[0]);
            }}
            style={s.iconBtn}
          >
            <Text style={s.iconText}>✏️</Text>
          </TouchableOpacity>

          <TouchableOpacity onPress={() => toggleTask(item.id)} style={s.iconBtn}>
            <Text style={s.iconText}>{item.completed ? "✅" : "⬜"}</Text>
          </TouchableOpacity>

          <TouchableOpacity onPress={() => deleteTask(item.id)} style={[s.iconBtn, { backgroundColor: "#2a0808" }]}>
            <Text style={s.iconText}>🗑️</Text>
          </TouchableOpacity>
        </View>
      </View>
    );
  }, [tasks]);

  /* ══════════════
     STATS SCREEN
  ══════════════ */
  if (screen === "stats") {
    return (
      <View style={s.screen}>
        <StatusBar barStyle="light-content" />

        <View style={s.statsHeader}>
          <TouchableOpacity onPress={() => setScreen("tasks")} style={s.backBtn}>
            <Text style={s.backText}>← BACK</Text>
          </TouchableOpacity>
          <Text style={s.screenTitle}>MISSION REPORT</Text>
          <BatSymbol />
        </View>

        <GoldLine />

        <View style={s.overallCard}>
          <Text style={s.overallLabel}>OVERALL COMPLETION</Text>
          <Text style={s.overallPct}>{rate}%</Text>
          <View style={s.progressTrack}>
            <View style={[s.progressBar, { width: `${rate}%` as any }]} />
          </View>
          <Text style={s.overallSub}>{done} missions completed · {total - done} remaining</Text>
        </View>

        <ScrollView showsVerticalScrollIndicator={false}>
          {byCategory.map((c) => {
            const pct   = c.count === 0 ? 0 : Math.round((c.doneCount / c.count) * 100);
            const color = catColors[c.name] ?? C.gold;
            return (
              <View key={c.id} style={s.statRow}>
                <View style={s.statRowLeft}>
                  <Text style={s.statEmoji}>{emojiMap[c.name]}</Text>
                  <View>
                    <Text style={s.statName}>{c.name.toUpperCase()}</Text>
                    <Text style={s.statSub}>{c.doneCount}/{c.count} done</Text>
                  </View>
                </View>
                <View style={s.statBarWrap}>
                  <View style={[s.statBar, { width: `${pct}%` as any, backgroundColor: color }]} />
                </View>
                <Text style={[s.statPct, { color }]}>{pct}%</Text>
              </View>
            );
          })}
          <View style={{ height: 40 }} />
        </ScrollView>
      </View>
    );
  }

  /* ══════════════
     TASKS SCREEN
  ══════════════ */
  return (
    <KeyboardAvoidingView
      style={{ flex: 1 }}
      behavior={Platform.OS === "ios" ? "padding" : "height"}
      keyboardVerticalOffset={Platform.OS === "ios" ? 60 : 0}
    >
      <View style={s.screen}>
        <StatusBar barStyle="light-content" />

        {/* HEADER */}
        <Animated.View style={[s.header, {
          opacity: headerAnim,
          transform: [{ translateY: headerAnim.interpolate({ inputRange: [0, 1], outputRange: [-20, 0] }) }],
        }]}>
          <Animated.View style={{ transform: [{ scale: pulseAnim }] }}>
            <BatSymbol />
          </Animated.View>
          <View style={s.headerCenter}>
            <Text style={s.appTitle}>GOTHAM TASKS</Text>
            <Text style={s.appSub}>Dark Knight's Mission Board</Text>
          </View>
          <TouchableOpacity onPress={() => setScreen("stats")} style={s.statsNavBtn}>
            <Text style={s.statsNavText}>📊</Text>
          </TouchableOpacity>
        </Animated.View>

        <GoldLine />

        {/* SEARCH */}
        <View style={s.searchRow}>
          <Text style={s.searchIcon}>🔍</Text>
          <TextInput
            placeholder="Search missions..."
            value={search}
            onChangeText={setSearch}
            style={s.searchInput}
            placeholderTextColor={C.muted}
          />
          {search.length > 0 && (
            <TouchableOpacity onPress={() => setSearch("")}>
              <Text style={{ color: C.muted, fontSize: 20, paddingHorizontal: 8 }}>×</Text>
            </TouchableOpacity>
          )}
        </View>

        {/* STATUS FILTER */}
        <View style={s.filterRow}>
          {(["all", "pending", "done"] as FilterStatus[]).map((fs) => (
            <TouchableOpacity
              key={fs}
              onPress={() => setFilterStatus(fs)}
              style={[s.filterChip, filterStatus === fs && s.filterChipActive]}
            >
              <Text style={[s.filterChipText, filterStatus === fs && s.filterChipTextActive]}>
                {fs === "all" ? "ALL" : fs === "pending" ? "⏳ ACTIVE" : "✅ DONE"}
              </Text>
            </TouchableOpacity>
          ))}
          <Text style={s.taskCount}>{filtered.length} tasks</Text>
        </View>

        {/* CATEGORY STRIP */}
        <FlatList
          horizontal
          data={[{ id: 0, name: "All" } as Category, ...defaultCategories]}
          keyExtractor={(item) => item.id.toString()}
          showsHorizontalScrollIndicator={false}
          renderItem={({ item }) => {
            const isActive = item.id === 0 ? filterCat === null : filterCat === item.id;
            const color    = catColors[item.name] ?? C.gold;
            return (
              <TouchableOpacity
                onPress={() => {
                  setFilterCat(item.id === 0 ? null : item.id);
                  if (item.id !== 0) setSelectedCategory(item);
                }}
                style={[
                  s.catChip,
                  isActive && {
                    backgroundColor: (item.id === 0 ? C.gold : color) + "33",
                    borderColor: item.id === 0 ? C.gold : color,
                  },
                ]}
              >
                <Text style={[s.catChipText, isActive && { color: item.id === 0 ? C.gold : color }]}>
                  {item.id === 0 ? "🗂 ALL" : `${emojiMap[item.name]} ${item.name.toUpperCase()}`}
                </Text>
              </TouchableOpacity>
            );
          }}
          style={{ marginBottom: 12 }}
        />

        {/* ADD TASK AREA */}
        <View style={s.addArea}>
          <View style={s.addRow}>
            <TextInput
              placeholder="New mission..."
              value={task}
              onChangeText={setTask}
              style={s.addInput}
              placeholderTextColor={C.muted}
              onSubmitEditing={addTask}
              returnKeyType="done"
            />
            <TouchableOpacity onPress={addTask} style={s.addBtn}>
              <Text style={s.addBtnText}>＋</Text>
            </TouchableOpacity>
          </View>

          <View style={s.addMeta}>
            <TextInput
              value={date}
              onChangeText={setDate}
              style={s.dateInput}
              placeholderTextColor={C.muted}
              placeholder="YYYY-MM-DD"
            />
            <ScrollView horizontal showsHorizontalScrollIndicator={false}>
              {defaultCategories.map((c) => {
                const color  = catColors[c.name];
                const active = selectedCategory.id === c.id;
                return (
                  <TouchableOpacity
                    key={c.id}
                    onPress={() => { setSelectedCategory(c); setFilterCat(c.id); }}
                    style={[s.miniCat, active && { borderColor: color, backgroundColor: color + "22" }]}
                  >
                    <Text style={s.miniCatText}>{emojiMap[c.name]}</Text>
                  </TouchableOpacity>
                );
              })}
            </ScrollView>
          </View>
        </View>

        <GoldLine />

        {/* TASK LIST */}
        <FlatList
          data={filtered}
          keyExtractor={(item) => item.id.toString()}
          renderItem={renderTask}
          removeClippedSubviews
          maxToRenderPerBatch={10}
          windowSize={10}
          showsVerticalScrollIndicator={false}
          keyboardShouldPersistTaps="handled"
          ListEmptyComponent={
            <View style={s.emptyWrap}>
              <Text style={s.emptyBat}>🦇</Text>
              <Text style={s.emptyText}>NO MISSIONS FOUND</Text>
              <Text style={s.emptySubText}>Gotham is quiet... for now.</Text>
            </View>
          }
        />

        {/* EDIT MODAL */}
        <Modal visible={!!editTask} transparent animationType="fade">
          <KeyboardAvoidingView style={{ flex: 1 }} behavior={Platform.OS === "ios" ? "padding" : "height"}>
            <View style={s.modalBg}>
              <View style={s.modalCard}>
                <View style={s.modalHeader}>
                  <BatSymbol />
                  <Text style={s.modalTitle}>EDIT MISSION</Text>
                </View>
                <GoldLine />

                <Text style={s.modalLabel}>TITLE</Text>
                <TextInput
                  value={editTitle}
                  onChangeText={setEditTitle}
                  style={s.modalInput}
                  placeholderTextColor={C.muted}
                />

                <Text style={s.modalLabel}>DATE</Text>
                <TextInput
                  value={editDate}
                  onChangeText={setEditDate}
                  style={s.modalInput}
                  placeholderTextColor={C.muted}
                />

                <Text style={s.modalLabel}>CATEGORY</Text>
                <FlatList
                  horizontal
                  data={defaultCategories}
                  keyExtractor={(item) => item.id.toString()}
                  showsHorizontalScrollIndicator={false}
                  renderItem={({ item }) => {
                    const color = catColors[item.name];
                    return (
                      <TouchableOpacity
                        onPress={() => setEditCat(item)}
                        style={[
                          s.catChip,
                          editCat.id === item.id && { backgroundColor: color + "33", borderColor: color },
                        ]}
                      >
                        <Text style={[s.catChipText, editCat.id === item.id && { color }]}>
                          {emojiMap[item.name]} {item.name.toUpperCase()}
                        </Text>
                      </TouchableOpacity>
                    );
                  }}
                  style={{ marginBottom: 20 }}
                />

                <View style={{ flexDirection: "row", gap: 12 }}>
                  <TouchableOpacity onPress={saveEdit} style={[s.modalBtn, { backgroundColor: C.goldDim }]}>
                    <Text style={s.modalBtnText}>SAVE</Text>
                  </TouchableOpacity>
                  <TouchableOpacity onPress={() => setEditTask(null)} style={[s.modalBtn, { backgroundColor: "#2a0808" }]}>
                    <Text style={s.modalBtnText}>CANCEL</Text>
                  </TouchableOpacity>
                </View>
              </View>
            </View>
          </KeyboardAvoidingView>
        </Modal>

        {/* BATS */}
        {bats.map((bat) => (
          <Animated.View
            key={bat.id}
            style={{
              position: "absolute", bottom: 100, left: bat.x,
              transform: [
                { translateY: bat.anim.interpolate({ inputRange: [0, 1], outputRange: [0, -SCREEN_HEIGHT * 0.6] }) },
                { scale:      bat.anim.interpolate({ inputRange: [0, 0.5, 1], outputRange: [0.4, 1.2, 0.8] }) },
              ],
              opacity: bat.anim.interpolate({ inputRange: [0, 0.7, 1], outputRange: [1, 0.8, 0] }),
            }}
          >
            <Text style={{ fontSize: 22 }}>🦇</Text>
          </Animated.View>
        ))}

      </View>
    </KeyboardAvoidingView>
  );
}

/* ══════════════════════════════════════
   STYLES
══════════════════════════════════════ */
const s = StyleSheet.create({
  screen: { flex: 1, backgroundColor: "transparent", paddingHorizontal: 16, paddingTop: Platform.OS === "ios" ? 54 : 36 },

  /* HEADER */
  header:        { flexDirection: "row", alignItems: "center", marginBottom: 10 },
  headerCenter:  { flex: 1, alignItems: "center" },
  appTitle:      { color: C.gold, fontSize: 20, fontWeight: "900", letterSpacing: 4 },
  appSub:        { color: C.muted, fontSize: 10, letterSpacing: 2, marginTop: 2 },
  statsNavBtn:   { width: 40, height: 40, alignItems: "center", justifyContent: "center", backgroundColor: "rgba(17,17,24,0.8)", borderRadius: 10, borderWidth: 1, borderColor: C.border },
  statsNavText:  { fontSize: 20 },

  /* SEARCH */
  searchRow:  { flexDirection: "row", alignItems: "center", backgroundColor: "rgba(13,13,20,0.85)", borderRadius: 12, borderWidth: 1, borderColor: C.border, paddingHorizontal: 12, marginBottom: 10 },
  searchIcon: { fontSize: 16, marginRight: 6 },
  searchInput:{ flex: 1, color: C.white, fontSize: 14, paddingVertical: 12 },

  /* FILTER */
  filterRow:           { flexDirection: "row", alignItems: "center", gap: 8, marginBottom: 10 },
  filterChip:          { paddingHorizontal: 14, paddingVertical: 7, backgroundColor: "rgba(17,17,24,0.85)", borderRadius: 20, borderWidth: 1, borderColor: C.border },
  filterChipActive:    { backgroundColor: C.gold + "22", borderColor: C.gold },
  filterChipText:      { color: C.muted, fontSize: 11, fontWeight: "700", letterSpacing: 1 },
  filterChipTextActive:{ color: C.gold },
  taskCount:           { marginLeft: "auto" as any, color: C.muted, fontSize: 11 },

  /* CATEGORY CHIPS */
  catChip:     { paddingHorizontal: 14, paddingVertical: 10, marginRight: 8, backgroundColor: "rgba(17,17,24,0.85)", borderRadius: 20, borderWidth: 1, borderColor: C.border },
  catChipText: { color: C.muted, fontSize: 13, fontWeight: "700", letterSpacing: 0.8 },

  /* ADD AREA */
  addArea:    { backgroundColor: "rgba(13,13,20,0.88)", borderRadius: 16, borderWidth: 1, borderColor: C.border, padding: 12, marginBottom: 10 },
  addRow:     { flexDirection: "row", alignItems: "center", gap: 10, marginBottom: 10 },
  addInput:   { flex: 1, color: C.white, fontSize: 15, paddingVertical: 10, paddingHorizontal: 4 },
  addBtn:     { width: 44, height: 44, backgroundColor: C.gold, borderRadius: 12, alignItems: "center", justifyContent: "center" },
  addBtnText: { color: C.bg, fontSize: 26, fontWeight: "900", lineHeight: 30 },
  addMeta:    { flexDirection: "row", alignItems: "center", gap: 10 },
  dateInput:  { color: C.white, fontSize: 13, backgroundColor: "rgba(6,6,8,0.6)", paddingHorizontal: 10, paddingVertical: 8, borderRadius: 8, borderWidth: 1, borderColor: C.border, minWidth: 110 },
  miniCat:    { width: 38, height: 38, borderRadius: 10, alignItems: "center", justifyContent: "center", backgroundColor: "rgba(6,6,8,0.5)", borderWidth: 1, borderColor: C.border, marginRight: 6 },
  miniCatText:{ fontSize: 18 },

  /* TASK CARD */
  taskCard:     { backgroundColor: "rgba(17,17,24,0.88)", borderRadius: 14, padding: 14, marginBottom: 10, flexDirection: "row", alignItems: "center", borderWidth: 1, borderColor: C.border },
  taskTitle:    { color: C.white, fontSize: 15, fontWeight: "600", marginBottom: 6 },
  strikethrough:{ textDecorationLine: "line-through", color: C.muted },
  taskMeta:     { flexDirection: "row", alignItems: "center", gap: 8, flexWrap: "wrap" },
  catBadge:     { paddingHorizontal: 8, paddingVertical: 3, borderRadius: 6, borderWidth: 1 },
  catBadgeText: { fontSize: 11, fontWeight: "700" },
  dateText:     { color: C.muted, fontSize: 11 },
  taskActions:  { flexDirection: "row", gap: 6, marginLeft: 8 },
  iconBtn:      { width: 34, height: 34, backgroundColor: "rgba(6,6,8,0.7)", borderRadius: 8, alignItems: "center", justifyContent: "center", borderWidth: 1, borderColor: C.border },
  iconText:     { fontSize: 16 },

  /* EMPTY */
  emptyWrap:    { alignItems: "center", marginTop: 60 },
  emptyBat:     { fontSize: 60, marginBottom: 12, opacity: 0.4 },
  emptyText:    { color: C.muted, fontSize: 16, fontWeight: "700", letterSpacing: 2 },
  emptySubText: { color: C.muted, fontSize: 12, marginTop: 6, opacity: 0.6 },

  /* MODAL */
  modalBg:      { flex: 1, backgroundColor: "rgba(0,0,0,0.88)", justifyContent: "center", padding: 20 },
  modalCard:    { backgroundColor: "rgba(17,17,24,0.98)", borderRadius: 20, padding: 22, borderWidth: 1, borderColor: C.gold + "44" },
  modalHeader:  { flexDirection: "row", alignItems: "center", gap: 12, marginBottom: 8 },
  modalTitle:   { color: C.gold, fontSize: 18, fontWeight: "900", letterSpacing: 3 },
  modalLabel:   { color: C.muted, fontSize: 11, letterSpacing: 2, marginBottom: 6, marginTop: 12 },
  modalInput:   { backgroundColor: "rgba(6,6,8,0.8)", color: C.white, paddingHorizontal: 14, paddingVertical: 12, borderRadius: 10, fontSize: 15, borderWidth: 1, borderColor: C.border, marginBottom: 4 },
  modalBtn:     { flex: 1, paddingVertical: 13, borderRadius: 12, alignItems: "center" },
  modalBtnText: { color: C.white, fontWeight: "900", letterSpacing: 2, fontSize: 13 },

  /* STATS */
  statsHeader:  { flexDirection: "row", alignItems: "center", justifyContent: "space-between", marginBottom: 10 },
  backBtn:      { paddingHorizontal: 12, paddingVertical: 8, backgroundColor: "rgba(17,17,24,0.85)", borderRadius: 10, borderWidth: 1, borderColor: C.border },
  backText:     { color: C.gold, fontWeight: "700", fontSize: 12, letterSpacing: 1 },
  screenTitle:  { color: C.gold, fontSize: 16, fontWeight: "900", letterSpacing: 3 },
  overallCard:  { backgroundColor: "rgba(17,17,24,0.9)", borderRadius: 16, padding: 20, marginBottom: 16, borderWidth: 1, borderColor: C.gold + "33", alignItems: "center" },
  overallLabel: { color: C.muted, fontSize: 11, letterSpacing: 2, marginBottom: 6 },
  overallPct:   { color: C.gold, fontSize: 56, fontWeight: "900" },
  progressTrack:{ width: "100%", height: 8, backgroundColor: "rgba(6,6,8,0.8)", borderRadius: 4, overflow: "hidden", marginVertical: 10 },
  progressBar:  { height: 8, backgroundColor: C.gold, borderRadius: 4 },
  overallSub:   { color: C.muted, fontSize: 12 },
  statRow:      { flexDirection: "row", alignItems: "center", backgroundColor: "rgba(17,17,24,0.88)", borderRadius: 12, padding: 14, marginBottom: 8, gap: 12, borderWidth: 1, borderColor: C.border },
  statRowLeft:  { flexDirection: "row", alignItems: "center", gap: 10, width: 110 },
  statEmoji:    { fontSize: 22 },
  statName:     { color: C.white, fontSize: 12, fontWeight: "700", letterSpacing: 1 },
  statSub:      { color: C.muted, fontSize: 11 },
  statBarWrap:  { flex: 1, height: 6, backgroundColor: "rgba(6,6,8,0.8)", borderRadius: 3, overflow: "hidden" },
  statBar:      { height: 6, borderRadius: 3 },
  statPct:      { width: 38, textAlign: "right", fontSize: 12, fontWeight: "700" },
});
