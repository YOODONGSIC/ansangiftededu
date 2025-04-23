# ansangiftededu
import tkinter as tk
from tkinter import filedialog, messagebox
import rawpy
import numpy as np
from astropy import units as u
from astropy.coordinates import Distance


def calculate_apparent_magnitude(raw_file_path, exposure_time, gain=1.0, background_level=0):
    with rawpy.imread(raw_file_path) as raw:
        rgb = raw.postprocess(use_camera_wb=True, output_bps=16)
        gray = np.mean(rgb, axis=2)

    max_pixel_value = np.max(gray)
    signal = max_pixel_value - background_level
    e_signal = signal * gain
    m_app = -2.5 * np.log10(e_signal / exposure_time)
    return m_app


def calculate_absolute_magnitude(m_app, distance_parsec):
    return m_app - 5 * (np.log10(distance_parsec) - 1)


def calculate_distance(m_app, m_abs):
    return 10 ** ((m_app - m_abs + 5) / 5)


class AstroApp:
    def __init__(self, root):
        self.root = root
        self.root.title("천체 등급 분석기")

        tk.Label(root, text="RAW 파일 경로:").grid(row=0, column=0)
        self.file_entry = tk.Entry(root, width=40)
        self.file_entry.grid(row=0, column=1)
        tk.Button(root, text="찾아보기", command=self.browse_file).grid(row=0, column=2)

        tk.Label(root, text="노출 시간 (초):").grid(row=1, column=0)
        self.exposure_entry = tk.Entry(root)
        self.exposure_entry.grid(row=1, column=1)

        tk.Label(root, text="Gain:").grid(row=2, column=0)
        self.gain_entry = tk.Entry(root)
        self.gain_entry.grid(row=2, column=1)

        tk.Label(root, text="배경광 (ADU):").grid(row=3, column=0)
        self.background_entry = tk.Entry(root)
        self.background_entry.grid(row=3, column=1)

        tk.Label(root, text="절대 등급 (선택):").grid(row=4, column=0)
        self.abs_mag_entry = tk.Entry(root)
        self.abs_mag_entry.grid(row=4, column=1)

        tk.Label(root, text="거리 (파섹, 선택):").grid(row=5, column=0)
        self.distance_entry = tk.Entry(root)
        self.distance_entry.grid(row=5, column=1)

        tk.Button(root, text="계산하기", command=self.calculate).grid(row=6, column=1)

        self.result_label = tk.Label(root, text="")
        self.result_label.grid(row=7, column=0, columnspan=3)

    def browse_file(self):
        filepath = filedialog.askopenfilename(filetypes=[("RAW files", "*.NEF *.CR2 *.ARW *.RAF *.DNG")])
        if filepath:
            self.file_entry.delete(0, tk.END)
            self.file_entry.insert(0, filepath)

    def calculate(self):
        try:
            filepath = self.file_entry.get()
            exposure = float(self.exposure_entry.get())
            gain = float(self.gain_entry.get())
            background = float(self.background_entry.get())

            m_app = calculate_apparent_magnitude(filepath, exposure, gain, background)

            result = f"겉보기 등급: {m_app:.2f}\n"

            if self.abs_mag_entry.get():
                m_abs = float(self.abs_mag_entry.get())
                distance = calculate_distance(m_app, m_abs)
                result += f"거리 (파섹): {distance:.2f}\n"
            elif self.distance_entry.get():
                distance = float(self.distance_entry.get())
                m_abs = calculate_absolute_magnitude(m_app, distance)
                result += f"절대 등급: {m_abs:.2f}\n"

            self.result_label.config(text=result)

        except Exception as e:
            messagebox.showerror("오류", str(e))


if __name__ == "__main__":
    root = tk.Tk()
    app = AstroApp(root)
    root.mainloop()
