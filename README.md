import customtkinter as ctk
from tkinter import messagebox
from datetime import datetime, timedelta
import csv
import os

ctk.set_appearance_mode("dark")
ctk.set_default_color_theme("blue")

# ============================
# TABLAS
# ============================

maximos_antes = [
    (0, 999, 999),
    (1000, 1999, 100),
    (2000, 2999, 75),
    (3000, 3999, 70),
    (4000, 4999, 60),
    (5000, 5999, 54),
    (6000, 6999, 48),
    (7000, 7999, 39),
    (8000, 8999, 32),
    (9000, 9999, 25),
    (10000, 10499, 22),
    (10500, 10999, 17),
    (11000, 11999, 14),
    (12000, 12999, 12),
    (13000, 13999, 6),
    (14000, 15999, 1),
]

maximos_despues = [
    (0, 999, 999),
    (1000, 1999, 115),
    (2000, 2999, 85),
    (3000, 3999, 80),
    (4000, 4999, 72),
    (5000, 5999, 67),
    (6000, 6999, 60),
    (7000, 7999, 49),
    (8000, 8999, 41),
    (9000, 9999, 36),
    (10000, 10499, 31),
    (10500, 10999, 26),
    (11000, 11999, 22),
    (12000, 12999, 19),
    (13000, 13999, 14),
    (14000, 14999, 10),
    (15000, 15999, 8),
    (16000, 17499, 6),
    (17500, 9999999, 2),
]

maximos_2025 = [
    (12000, 12999, 20),
    (13000, 13999, 18),
    (14000, 14999, 14),
    (15000, 15999, 11),
    (16000, 17499, 8),
    (17500, 9999999, 4),
]

fecha_corte_2024 = datetime(2024, 6, 1)
fecha_corte_2025 = datetime(2025, 10, 16)


# ============================
# FUNCIONES
# ============================

def obtener_maximo_nivel(nivel, fecha):
    if fecha >= fecha_corte_2025:
        tabla = maximos_2025
    elif fecha >= fecha_corte_2024:
        tabla = maximos_despues
    else:
        tabla = maximos_antes

    for minimo, maximo, max_por_dia in tabla:
        if minimo <= nivel <= maximo:
            return max_por_dia
    return 1


def calcular_proyeccion_diaria(nivel_inicial, fecha_inicio, fecha_final):
    progreso = []
    nivel = nivel_inicial
    fecha = fecha_inicio

    while fecha < fecha_final:
        max_dia = obtener_maximo_nivel(nivel, fecha)
        nivel_despues = nivel + max_dia

        progreso.append({
            "fecha": fecha.strftime("%d-%m-%Y"),
            "nivel_inicial": nivel,
            "ganados": max_dia,
            "nivel_final": nivel_despues
        })

        nivel = nivel_despues
        fecha += timedelta(days=1)

    return progreso


def calcular():
    global progreso_global

    try:
        nivel_inicial = int(entry_nivel.get())

        f_inicio = entry_inicio.get().replace(" ", "-").replace("--", "-")
        f_final = entry_final.get().replace(" ", "-").replace("--", "-")

        fecha_inicio = datetime.strptime(f_inicio, "%d-%m-%Y")
        fecha_final = datetime.strptime(f_final, "%d-%m-%Y")

        if fecha_final <= fecha_inicio:
            raise ValueError("La fecha objetivo debe ser posterior a la inicial.")

    except Exception as e:
        messagebox.showerror("Error", f"Datos inválidos:\n{e}")
        return

    progreso_global = calcular_proyeccion_diaria(nivel_inicial, fecha_inicio, fecha_final)
    nivel_final = progreso_global[-1]["nivel_final"]

    resultado_label.configure(
        text=f"🔥 Nivel proyectado: {nivel_final}\n📅 Hasta el {fecha_final.strftime('%d-%m-%Y')}",
        text_color="#00ffaa"
    )


def descargar_csv():
    global progreso_global

    if not progreso_global:
        messagebox.showwarning("Aviso", "Primero debes calcular un resultado.")
        return

    archivo = "niveles_por_dia.csv"

    with open(archivo, "w", newline="", encoding="utf-8") as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(["Fecha", "Nivel Inicial", "Ganados", "Nivel Final"])

        for dia in progreso_global:
            writer.writerow([
                dia["fecha"],
                dia["nivel_inicial"],
                dia["ganados"],
                dia["nivel_final"]
            ])

    messagebox.showinfo("Archivo generado", f"Archivo guardado como:\n{os.path.abspath(archivo)}")


# ============================
# INTERFAZ MODERNA
# ============================

root = ctk.CTk()
root.title("Calculadora de Proyección de Nivel")
root.geometry("500x560")
root.resizable(False, False)

progreso_global = []

ctk.CTkLabel(root, text="📈 Calculadora de Proyección de Nivel",
             font=("Arial", 22, "bold")).pack(pady=15)

frame = ctk.CTkFrame(root, corner_radius=15)
frame.pack(padx=20, pady=10, fill="both", expand=False)

ctk.CTkLabel(frame, text="Nivel actual:", font=("Arial", 16)).pack(pady=5)
entry_nivel = ctk.CTkEntry(frame, justify="center", height=35, font=("Arial", 15))
entry_nivel.pack(pady=5, padx=20)

ctk.CTkLabel(frame, text="Fecha inicio (D-M-AAAA):", font=("Arial", 16)).pack(pady=5)
entry_inicio = ctk.CTkEntry(frame, justify="center", height=35, font=("Arial", 15))
entry_inicio.pack(pady=5, padx=20)

ctk.CTkLabel(frame, text="Fecha objetivo (D-M-AAAA):", font=("Arial", 16)).pack(pady=5)
entry_final = ctk.CTkEntry(frame, justify="center", height=35, font=("Arial", 15))
entry_final.pack(pady=5, padx=20)

ctk.CTkButton(
    root,
    text="Calcular",
    font=("Arial", 18, "bold"),
    height=45,
    fg_color="#008cff",
    hover_color="#0068c7",
    command=calcular
).pack(pady=10)

ctk.CTkButton(
    root,
    text="📥 Descargar niveles por día",
    font=("Arial", 17, "bold"),
    height=45,
    fg_color="#00aa55",
    hover_color="#008f46",
    command=descargar_csv

).pack(pady=5)

resultado_label = ctk.CTkLabel(root, text="Esperando datos...",
                               font=("Arial", 17),
                               text_color="#bbbbbb",
                               justify="center")
resultado_label.pack(pady=15)

root.mainloop()
