import sys
import json
import base64
from datetime import datetime, timedelta
from PyQt5.QtWidgets import (QApplication, QWidget, QVBoxLayout, QLabel, 
                             QLineEdit, QPushButton, QTextEdit, QMessageBox, 
                             QHBoxLayout, QComboBox, QFrame)
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QFont

# --- CONFIGURATION (MUST MATCH DOWNLOADER APP) ---
SECRET_KEY = "KIMSAN_SECURE_2026" 

class KeyAdmin(QWidget):
    def __init__(self):
        super().__init__()
        self.current_lang = "kh"
        
        # 1. Theme Definitions
        self.themes = {
            "Dark": {
                "bg": "#1a1a2e", "text": "#ffffff", "input": "#16213e", 
                "accent": "#4ff10f", "btn": "#27ae60", "border": "#0f3460"
            },
            "Light": {
                "bg": "#f8f9fa", "text": "#212529", "input": "#ffffff", 
                "accent": "#007bff", "btn": "#28a745", "border": "#dee2e6"
            },
            "Midnight": {
                "bg": "#000000", "text": "#00ff41", "input": "#0d0d0d", 
                "accent": "#00ff41", "btn": "#333333", "border": "#00ff41"
            },
            "Forest": {
                "bg": "#1b3022", "text": "#ecf0f1", "input": "#26413c", 
                "accent": "#aebfbe", "btn": "#5e8c61", "border": "#2de24e"
            },
            "Royal": {
                "bg": "#2d033b", "text": "#ffffff", "input": "#810ca8", 
                "accent": "#ffd700", "btn": "#c147e9", "border": "#e5b8f4"
            }
        }
        
        # 2. Language Dictionary
        self.lang_data = {
            "kh": {
                "title": "កម្មវិធីគ្រប់គ្រងអាជ្ញាប័ណ្ណ - ADMIN",
                "mid": "លេខសម្គាល់ម៉ាស៊ីន (Machine ID):",
                "days": "សុពលភាពប្រើប្រាស់ (ចំនួនថ្ងៃ):",
                "gen_btn": "បង្កើតកូដអាជ្ញាប័ណ្ណ",
                "copy_btn": "ចម្លងកូដ (Copy)",
                "result": "កូដដែលបានបង្កើតរួច:",
                "success": "ជោគជ័យ",
                "error": "កំហុស",
                "fill_all": "សូមបំពេញព័ត៌មានឱ្យគ្រប់គ្រាន់!",
                "num_only": "ចំនួនថ្ងៃត្រូវតែជាលេខរៀង!",
                "copied": "កូដត្រូវបានចម្លងទៅកាន់ Clipboard!",
                "theme_lbl": "រចនាប័ទ្ម:",
                "lang_lbl": "ភាសា:",
                "expiry_msg": "កូដនឹងផុតកំណត់នៅថ្ងៃទី:",
                "placeholder_mid": "ឧទាហរណ៍: 8D7F2A3B...",
                "placeholder_days": "ឧទាហរណ៍: 30, 365..."
            },
            "en": {
                "title": "LICENSE KEY GENERATOR - ADMIN",
                "mid": "User Machine ID:",
                "days": "Validity Period (Days):",
                "gen_btn": "GENERATE LICENSE KEY",
                "copy_btn": "COPY KEY TO CLIPBOARD",
                "result": "Generated License Key:",
                "success": "Success",
                "error": "Error",
                "fill_all": "Please fill in all required fields!",
                "num_only": "Days must be a numeric value!",
                "copied": "License key copied to clipboard!",
                "theme_lbl": "Theme:",
                "lang_lbl": "Language:",
                "expiry_msg": "Key will expire on:",
                "placeholder_mid": "Example: 8D7F2A3B...",
                "placeholder_days": "Example: 30, 365..."
            }
        }
        
        self.initUI()
        
    def initUI(self):
        self.setWindowTitle("KIMSAN - Admin License Manager v2.0")
        self.setFixedSize(520, 750)
        
        layout = QVBoxLayout()
        layout.setContentsMargins(30, 25, 30, 30)
        layout.setSpacing(15)
        
        # --- Toolbar ---
        toolbar = QHBoxLayout()
        self.lang_lbl_widget = QLabel()
        self.lang_combo = QComboBox()
        self.lang_combo.addItems(["Khmer", "English"])
        self.lang_combo.currentIndexChanged.connect(self.change_language)
        
        self.theme_lbl_widget = QLabel()
        self.theme_combo = QComboBox()
        self.theme_combo.addItems(list(self.themes.keys()))
        self.theme_combo.currentTextChanged.connect(self.apply_theme)
        
        toolbar.addWidget(self.lang_lbl_widget)
        toolbar.addWidget(self.lang_combo)
        toolbar.addStretch()
        toolbar.addWidget(self.theme_lbl_widget)
        toolbar.addWidget(self.theme_combo)
        layout.addLayout(toolbar)

        line = QFrame()
        line.setFrameShape(QFrame.HLine)
        line.setFrameShadow(QFrame.Sunken)
        layout.addWidget(line)

        # --- Title ---
        self.title_lbl = QLabel()
        self.title_lbl.setFont(QFont("Khmer OS Siemreap", 18, QFont.Bold))
        self.title_lbl.setAlignment(Qt.AlignCenter)
        layout.addWidget(self.title_lbl)
        
        # --- Form Inputs ---
        self.mid_lbl = QLabel()
        layout.addWidget(self.mid_lbl)
        self.mid_input = QLineEdit()
        layout.addWidget(self.mid_input)
        
        self.days_lbl = QLabel()
        layout.addWidget(self.days_lbl)
        self.days_input = QLineEdit()
        layout.addWidget(self.days_input)
        
        # --- Action Buttons ---
        self.gen_btn = QPushButton()
        self.gen_btn.setCursor(Qt.PointingHandCursor)
        self.gen_btn.setFixedHeight(55)
        self.gen_btn.clicked.connect(self.generate_key)
        layout.addWidget(self.gen_btn)
        
        self.result_lbl = QLabel()
        layout.addWidget(self.result_lbl)
        self.result_key = QTextEdit()
        self.result_key.setReadOnly(True)
        self.result_key.setFont(QFont("Consolas", 11))
        self.result_key.setMaximumHeight(120)
        layout.addWidget(self.result_key)
        
        self.copy_btn = QPushButton()
        self.copy_btn.setCursor(Qt.PointingHandCursor)
        self.copy_btn.setFixedHeight(45)
        self.copy_btn.clicked.connect(self.copy_to_clipboard)
        layout.addWidget(self.copy_btn)
        
        # --- Footer ---
        self.footer_lbl = QLabel("© 2026 DEVELOPMENT BY @kim_saa | LICENSE SYSTEM")
        self.footer_lbl.setAlignment(Qt.AlignCenter)
        self.footer_lbl.setStyleSheet("font-size: 11px; margin-top: 15px; font-weight: normal;")
        layout.addWidget(self.footer_lbl)
        
        self.setLayout(layout)
        self.apply_theme("Dark")
        self.update_translation()

    def apply_theme(self, theme_name):
        t = self.themes[theme_name]
        style = f"""
            QWidget {{ background-color: {t['bg']}; color: {t['text']}; font-family: 'Khmer OS Siemreap', 'Segoe UI'; }}
            QLineEdit {{ 
                padding: 12px; background: {t['input']}; border: 2px solid {t['border']}; 
                border-radius: 10px; color: {t['text']}; font-size: 14px;
            }}
            QTextEdit {{ 
                background: {t['input']}; color: {t['text']}; border: 2px solid {t['border']}; 
                padding: 12px; border-radius: 10px; 
            }}
            QPushButton {{ 
                background-color: {t['btn']}; color: white; font-weight: bold; 
                border-radius: 10px; border: none; font-size: 15px; 
            }}
            QPushButton:hover {{ background-color: {t['accent']}; color: black; }}
            QComboBox {{ 
                background: {t['input']}; color: {t['text']}; padding: 6px; 
                border: 1px solid {t['border']}; border-radius: 6px; 
            }}
            QLabel {{ font-weight: bold; font-size: 14px; }}
        """
        self.setStyleSheet(style)
        self.title_lbl.setStyleSheet(f"color: {t['accent']}; background: transparent;")

    def change_language(self, index):
        self.current_lang = "kh" if index == 0 else "en"
        self.update_translation()

    def update_translation(self):
        l = self.lang_data[self.current_lang]
        self.title_lbl.setText(l['title'])
        self.mid_lbl.setText(l['mid'])
        self.days_lbl.setText(l['days'])
        self.gen_btn.setText(l['gen_btn'])
        self.copy_btn.setText(l['copy_btn'])
        self.result_lbl.setText(l['result'])
        self.lang_lbl_widget.setText(l['lang_lbl'])
        self.theme_lbl_widget.setText(l['theme_lbl'])
        self.mid_input.setPlaceholderText(l['placeholder_mid'])
        self.days_input.setPlaceholderText(l['placeholder_days'])

    def generate_key(self):
        l = self.lang_data[self.current_lang]
        mid = self.mid_input.text().strip()
        days_str = self.days_input.text().strip()
        
        if not mid or not days_str:
            QMessageBox.warning(self, l['error'], l['fill_all'])
            return
            
        try:
            days = int(days_str)
            expiry_date = (datetime.now() + timedelta(days=days)).strftime("%Y-%m-%d")
            
            # 1. Create Data Object
            payload = {
                "mid": mid, 
                "expiry": expiry_date, 
                "gen_date": datetime.now().strftime("%Y-%m-%d")
            }
            
            # 2. JSON String -> XOR Encryption
            json_str = json.dumps(payload)
            encrypted = "".join(chr(ord(c) ^ ord(SECRET_KEY[i % len(SECRET_KEY)])) for i, c in enumerate(json_str))
            
            # 3. Base64 Encoding
            final_key = base64.b64encode(encrypted.encode()).decode()
            
            self.result_key.setPlainText(final_key)
            QMessageBox.information(self, l['success'], f"{l['expiry_msg']} {expiry_date}")
            
        except ValueError:
            QMessageBox.critical(self, l['error'], l['num_only'])

    def copy_to_clipboard(self):
        l = self.lang_data[self.current_lang]
        key_text = self.result_key.toPlainText().strip()
        if key_text:
            QApplication.clipboard().setText(key_text)
            QMessageBox.information(self, l['success'], l['copied'])

if __name__ == "__main__":
    app = QApplication(sys.argv)
    app.setFont(QFont("Khmer OS Siemreap", 10))
    window = KeyAdmin()
    window.show()
    sys.exit(app.exec_())
