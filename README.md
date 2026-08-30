# Task Manager — MERN Stack

## 1. Backend Setup

```bash
npm init -y
npm install mongodb express mongoose cors dotenv
npm install -D nodemon
```

## 2. Package.json Scripts

```json
{
  "scripts": {
    "dev": "nodemon server.js",
    "start": "node server.js"
  }
}
```

Run the backend:

```bash
npm run dev
```

## 3. Create MongoDB Atlas Database

In MongoDB Atlas, create:

* **Database:** `taskmanager`
* **Collection:** `tasks`

Final structure:

```text
Cluster0
└── taskmanager
    └── tasks
```

## 4. MongoDB Atlas Connection

## 5. .env File

```env
PORT=5000
MONGO_URI=mongodb+srv://taskmanager:tswjk7kpQuFXImLa@cluster0.x9u2gny.mongodb.net/taskmanager?appName=Cluster0
```

## 6. .gitignore

```text
node_modules
.env
```

## 7. Complete server.js

Our entire backend is inside this file:

```js
const dns = require("dns");
dns.setServers(["8.8.8.8", "8.8.4.4"]);

const express = require("express");
const { MongoClient, ObjectId } = require("mongodb");
const cors = require("cors");
require("dotenv").config();

const app = express();


// ==================== Middleware ====================

app.use(cors());
app.use(express.json());


// ==================== MongoDB Connection ====================

const client = new MongoClient(process.env.MONGO_URI);

let tasksCollection;

async function run() {
  try {
    await client.connect();
    
    const db = client.db("taskmanager");
    tasksCollection = db.collection("tasks")

    console.log("Pinged your deployment. You successfully connected to MongoDB!");
  } finally {
    // await client.close();
  }
}
run().catch(console.dir);


// ==================== Test Route ====================

app.get("/", (req, res) => {
  res.send("Task Manager API is running");
});


// ==================== GET - Get All Tasks ====================

app.get("/api/tasks", async (req, res) => {
  try {
    const tasks = await tasksCollection.find().toArray();

    res.status(200).json(tasks);
  } catch (error) {
    res.status(500).json({
      message: error.message,
    });
  }
});


// ==================== GET - Get Single Task ====================

app.get("/api/tasks/:id", async (req, res) => {
  try {
    const task = await tasksCollection.findOne({
      _id: new ObjectId(req.params.id),
    });

    if (!task) {
      return res.status(404).json({
        message: "Task not found",
      });
    }

    res.status(200).json(task);
  } catch (error) {
    res.status(500).json({
      message: error.message,
    });
  }
});


// ==================== POST - Create Task ====================

app.post("/api/tasks", async (req, res) => {
  try {
    const { title, description, completed } = req.body;

    const newTask = {
      title,
      description: description || "",
      completed: completed || false,
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    const result = await tasksCollection.insertOne(newTask);

    const task = {
      _id: result.insertedId,
      ...newTask,
    };

    res.status(201).json(task);
  } catch (error) {
    res.status(500).json({
      message: error.message,
    });
  }
});


// ==================== PUT - Update Task ====================

app.put("/api/tasks/:id", async (req, res) => {
  try {
    const { title, description, completed } = req.body;

    const updatedTask = {
      title,
      description,
      completed,
      updatedAt: new Date(),
    };

    const result = await tasksCollection.findOneAndUpdate(
      {
        _id: new ObjectId(req.params.id),
      },
      {
        $set: updatedTask,
      },
      {
        returnDocument: "after",
      }
    );

    if (!result) {
      return res.status(404).json({
        message: "Task not found",
      });
    }

    res.status(200).json(result);
  } catch (error) {
    res.status(500).json({
      message: error.message,
    });
  }
});


// ==================== DELETE - Delete Task ====================

app.delete("/api/tasks/:id", async (req, res) => {
  try {
    const task = await tasksCollection.findOneAndDelete({
      _id: new ObjectId(req.params.id),
    });

    if (!task) {
      return res.status(404).json({
        message: "Task not found",
      });
    }

    res.status(200).json({
      message: "Task deleted successfully",
      deletedTask: task,
    });
  } catch (error) {
    res.status(500).json({
      message: error.message,
    });
  }
});


// ==================== Server ====================

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

```

# React — Frontend

## 8. main.jsx

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import { RouterProvider } from "react-router/dom";
import router from "./router/Router.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>
);
```

## 9. Router.jsx

```jsx
import { createBrowserRouter } from "react-router";

import TaskManager from "../pages/TaskManager/TaskManager";

const router = createBrowserRouter([
  {
    path: "/",
    Component: TaskManager,
  },
]);

export default router;
```

## 10. hooks/useAxios.jsx
```
import axios from "axios";

const useAxios = () => {
  const axiosInstance = axios.create({
    baseURL: "http://localhost:5000/api",
  });

  return axiosInstance;
};

export default useAxios;
```

## 11. TaskManager.jsx

```jsx
import { useEffect, useState } from "react";
import useAxios from "../../hooks/useAxios";

const TaskManager = () => {
  const axios = useAxios();

  const [tasks, setTasks] = useState([]);
  const [title, setTitle] = useState("");
  const [description, setDescription] = useState("");
  const [editingId, setEditingId] = useState(null);

  // GET
  const getTasks = async () => {
    const res = await axios.get("/tasks");
    setTasks(res.data);
  };

  useEffect(() => {
    getTasks();
  }, []);

  // POST / PUT
  const handleSubmit = async (e) => {
    e.preventDefault();

    const taskData = {
      title,
      description,
    };

    if (editingId) {
      // PUT
      const res = await axios.put(
        `/tasks/${editingId}`,
        taskData
      );

      setTasks(
        tasks.map((task) =>
          task._id === editingId ? res.data : task
        )
      );

      setEditingId(null);
    } else {
      // POST
      const res = await axios.post("/tasks", taskData);

      setTasks([res.data, ...tasks]);
    }

    setTitle("");
    setDescription("");
  };

  // EDIT
  const handleEdit = (task) => {
    setEditingId(task._id);
    setTitle(task.title);
    setDescription(task.description);
  };

  // DELETE
  const handleDelete = async (id) => {
    await axios.delete(`/tasks/${id}`);

    setTasks(
      tasks.filter((task) => task._id !== id)
    );
  };

  return (
    <div className="min-h-screen bg-gray-100 p-8">
      <div className="mx-auto max-w-3xl">

        <h1 className="mb-8 text-center text-3xl font-bold">
          Task Manager
        </h1>

        {/* FORM */}
        <form
          onSubmit={handleSubmit}
          className="mb-8 rounded-lg bg-white p-6 shadow"
        >
          <input
            type="text"
            placeholder="Task title"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            className="mb-4 w-full rounded border p-3"
          />

          <textarea
            placeholder="Task description"
            value={description}
            onChange={(e) =>
              setDescription(e.target.value)
            }
            className="mb-4 w-full rounded border p-3"
          />

          <button
            type="submit"
            className="rounded bg-blue-600 px-5 py-2 text-white"
          >
            {editingId ? "Update Task" : "Add Task"}
          </button>

          {editingId && (
            <button
              type="button"
              onClick={() => {
                setEditingId(null);
                setTitle("");
                setDescription("");
              }}
              className="ml-2 rounded bg-gray-500 px-5 py-2 text-white"
            >
              Cancel
            </button>
          )}
        </form>

        {/* TASK LIST */}
        <div className="space-y-4">
          {tasks.map((task) => (
            <div
              key={task._id}
              className="flex items-center justify-between rounded-lg bg-white p-5 shadow"
            >
              <div>
                <h2 className="text-xl font-semibold">
                  {task.title}
                </h2>

                <p className="text-gray-600">
                  {task.description}
                </p>
              </div>

              <div className="flex gap-2">
                <button
                  onClick={() => handleEdit(task)}
                  className="rounded bg-yellow-500 px-4 py-2 text-white"
                >
                  Edit
                </button>

                <button
                  onClick={() =>
                    handleDelete(task._id)
                  }
                  className="rounded bg-red-600 px-4 py-2 text-white"
                >
                  Delete
                </button>
              </div>
            </div>
          ))}
        </div>

      </div>
    </div>
  );
};

export default TaskManager;
```
