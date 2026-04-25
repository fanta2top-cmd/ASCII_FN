import tkinter as tk
from tkinter import filedialog
from PIL import Image
import colorama
from colorama import Style
import os

colorama.init()

# Тут просто за дизайн Label
BG_COLOR = "#1e1e2e"
FG_COLOR = "#cdd6f4"
ACCENT_COLOR = "#fab387"
BTN_COLOR = "#89b4fa"
BTN_TEXT = "#11111b"


class ImageProcessor:
    def __init__(self):
        self.chars = "@#S%?*+;:,."

    def convert_to_ascii(self, path, width):
        try:
            img = Image.open(path)
            w_percent = (width / float(img.size[0]))
            h_size = int((float(img.size[1]) * float(w_percent)) / 1.65)
            img = img.resize((width, h_size))
            pixels = img.load()
            result = []

            for y in range(img.size[1]):
                line = ""
                for x in range(img.size[0]):
                    r, g, b = pixels[x, y][:3]
                    brightness = int(0.299 * r + 0.587 * g + 0.114 * b)
                    char = self.chars[brightness * (len(self.chars) - 1) // 255]
                    line += f"\033[38;2;{r};{g};{b}m{char}"
                result.append(line)
            return "\n".join(result)
        except Exception as e:
            return f"Error: {e}"


class App:
    def __init__(self, root):
        self.root = root
        self.root.title("ASCII Scanner Pro")
        self.root.geometry("450x450")
        self.root.configure(bg=BG_COLOR)
        self.processor = ImageProcessor()

        tk.Label(root, text="🚀 ASCII SCANNER", font=("Segoe UI", 20, "bold"),
                 bg=BG_COLOR, fg=ACCENT_COLOR, pady=20).pack()

        # Слайдер (Повзунок ширини
        tk.Label(self.root, text="Ширина зображення", bg=BG_COLOR, fg=FG_COLOR, font=("Arial", 10)).pack(pady=(10, 0))
        self.width_scale = tk.Scale(self.root, from_=50, to_=250, orient="horizontal", length=350,
                                    bg=BG_COLOR, fg=ACCENT_COLOR, highlightthickness=0,
                                    troughcolor="#313244", activebackground=ACCENT_COLOR)
        self.width_scale.set(100)
        self.width_scale.pack(pady=5)

        # Кнопка
        self.btn_run = tk.Button(root, text="ОБРАТИ ТА ЗГЕНЕРУВАТИ", command=self.start_process,
                                 bg=BTN_COLOR, fg=BTN_TEXT, font=("Segoe UI", 12, "bold"),
                                 activebackground=ACCENT_COLOR, cursor="hand2", bd=0, padx=20, pady=12)
        self.btn_run.pack(pady=30)

        self.status = tk.Label(root, text="Ready to scan...", font=("Consolas", 9), bg=BG_COLOR, fg="#6c7086")
        self.status.pack(side="bottom", pady=10)

    def start_process(self):
        path = filedialog.askopenfilename(filetypes=[("Images", "*.jpg *.png *.jpeg *.bmp")])
        if not path: return

        width = self.width_scale.get()
        art = self.processor.convert_to_ascii(path, width)

        os.system('cls' if os.name == 'nt' else 'clear')
        print(art + Style.RESET_ALL)
        self.status.config(text="✅ Static image rendered in console!")


if __name__ == "__main__":
    root = tk.Tk()
    app = App(root)
    root.mainloop()

