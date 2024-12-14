# IA
Este proyecto es una aplicación web de gestión de tareas
import tkinter as tk
from tkinter import simpledialog, messagebox
import mysql.connector

# Configuración de la conexión a la base de datos
db_config = {
    'user': 'root',
    'password': '12345',
    'host': 'localhost'
}

def connect_db():
    return mysql.connector.connect(**db_config)

def create_database_and_table():
    connection = connect_db()
    cursor = connection.cursor()
    
    # Crear la base de datos si no existe
    cursor.execute("CREATE DATABASE IF NOT EXISTS task_manager")
    
    # Usar la base de datos
    cursor.execute("USE task_manager")
    
    # Crear la tabla si no existe
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS tasks (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(255) NOT NULL,
        description TEXT,
        completed BOOLEAN DEFAULT FALSE
    )
    """)
    
    cursor.close()
    connection.close()

class Task:
    def __init__(self, id, title, description, completed=False):
        self.id = id
        self.title = title
        self.description = description
        self.completed = completed

def list_tasks():
    connection = connect_db()
    connection.database = 'task_manager'  # Usar la base de datos
    cursor = connection.cursor()
    cursor.execute("SELECT id, title, description, completed FROM tasks")
    tasks = cursor.fetchall()
    cursor.close()
    connection.close()
    return [Task(id, title, description, completed) for (id, title, description, completed) in tasks]

def add_task(title, description):
    connection = connect_db()
    connection.database = 'task_manager'  # Usar la base de datos
    cursor = connection.cursor()
    cursor.execute("INSERT INTO tasks (title, description) VALUES (%s, %s)", (title, description))
    connection.commit()
    cursor.close()
    connection.close()

def mark_task_completed(task_id):
    connection = connect_db()
    connection.database = 'task_manager'  # Usar la base de datos
    cursor = connection.cursor()
    cursor.execute("UPDATE tasks SET completed = TRUE WHERE id = %s", (task_id,))
    connection.commit()
    cursor.close()
    connection.close()

def delete_completed_tasks():
    connection = connect_db()
    connection.database = 'task_manager'  # Usar la base de datos
    cursor = connection.cursor()
    cursor.execute("DELETE FROM tasks WHERE completed = TRUE")
    connection.commit()
    cursor.close()
    connection.close()

class TaskManagerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Gestor de Tareas")
        
        self.task_listbox = tk.Listbox(root, width=50, height=15)
        self.task_listbox.pack(pady=10)

        self.add_button = tk.Button(root, text="Agregar Tarea", command=self.add_task)
        self.add_button.pack(pady=5)

        self.complete_button = tk.Button(root, text="Marcar como Completada", command=self.complete_task)
        self.complete_button.pack(pady=5)

        self.delete_button = tk.Button(root, text="Eliminar Completadas", command=self.delete_completed_tasks)
        self.delete_button.pack(pady=5)

        self.load_tasks()

    def load_tasks(self):
        self.task_listbox.delete(0, tk.END)
        tasks = list_tasks()
        for task in tasks:
            status = "✔" if task.completed else "✖"
            self.task_listbox.insert(tk.END, f"{task.id}: {task.title} - {status}")

    def add_task(self):
        title = simpledialog.askstring("Título", "Ingrese el título de la tarea:")
        description = simpledialog.askstring("Descripción", "Ingrese la descripción de la tarea:")
        if title:
            try:
                add_task(title, description)
                self.load_tasks()
            except Exception as e:
                messagebox.showerror("Error", f"Error al agregar la tarea: {e}")

    def complete_task(self):
        selected_task = self.task_listbox.curselection()
        if selected_task:
            task_id = self.task_listbox.get(selected_task).split(":")[0]
            try:
                mark_task_completed(int(task_id))
                self.load_tasks()
            except Exception as e:
                messagebox.showerror("Error", f"Error al marcar la tarea como completada: {e}")
        else:
            messagebox.showwarning("Advertencia", "Seleccione una tarea para marcar como completada.")

    def delete_completed_tasks(self):
        try:
            delete_completed_tasks()  # Llamar a la función para eliminar tareas completadas
            self.load_tasks()  # Recargar la lista de tareas
            messagebox.showinfo("Éxito", "Tareas completadas eliminadas con éxito.")
        except Exception as e:
            messagebox.showerror("Error", f"Error al eliminar tareas completadas: {e}")

if __name__ == "__main__":
    create_database_and_table()  # Crear la base de datos y la tabla si no existen
    root = tk.Tk()
    app = TaskManagerApp(root)
    root.mainloop()
