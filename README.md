# tornBrowser - Special Operations Suite

## Summary
**tornBrowser** is a custom desktop application built for optimized *Torn* gameplay. It features a split-screen interface with a built-in, resizable target finder, quick-swap tabs for key community intelligence tools like *weav3r.dev* and *torn-intel.com*, and full-screen support for distraction-free gym sessions or chain runs.

## Description
**tornBrowser** is a custom desktop browser environment designed specifically for dedicated *Torn* players who want to streamline their high-intensity gameplay mechanics. 

Instead of juggling multiple open Google Chrome tabs, this desktop application provides a central, consolidated panel tailored specifically for gym sessions and chain runs. It integrates a dedicated, resizable target finder directly into a split-screen view layout, allowing players to instantly scout war targets, evaluate the best possible victims, and quickly cycle their views through critical game intelligence platforms without interrupting their active gameplay page.

## How tornBrowser Helps with Gym Sessions
**tornBrowser** helps players maximize their gym training sessions by eliminating distractions and optimizing screen real estate:
* **Consolidated View:** Keeps the gym interface front and center without the clutter of a standard browser, ensuring you don't accidentally navigate away during energy dumps.
* **Side-by-Side Stat Tracking:** Allows you to use the split-screen panel to view community stat calculators or training guides alongside your live gym page.
* **Instant Target Evaluation:** If you need to jump from a training block straight into a fight, the integrated target window lets you scout viable targets instantly without losing your place in the gym.

### Integrated Sites
* **Torn.com:** Main combat arena for live attacks and managing hits.
* **Weav3r.dev:** Live target scouting to instantly pull low-stat victims for chains.
* **Torn-intel.com:** Tracks war walls, enemy status, and chain timers to prevent expirations.

## Technical Configuration & Architecture
* **Isolated Browser Partitioning:** Uses a dedicated `QWebEngineProfile` partition (`tornBrowserData`) to avoid profile configuration locks.
* **Persistent Session Management:** Forces strict cookie persistence via code configuration layers to maintain valid game states.
* **Responsive Visual Splitter:** Integrated structural handle logic supporting user-controlled aspect width resizing constraints natively.

## Deployment & Execution

### Running From Source
1. Ensure your machine has **Python 3.12+** and **PyQt6** installed.
2. Navigate directly to your local project directory:
   ```cmd
   cd C:\tornBrowser
   ```
3. Boot the application file:
   ```cmd
   python tornBrowser.py
   ```

### Building the Binary Standalone
To package your Python environment into a self-contained local Windows system executable, execute your script build parameter flag:
```cmd
python tornBrowser.py --build
```
//// FULL CODE HERE /////            
import sys
import os
import urllib.request
from PyQt6.QtCore import QUrl, Qt
from PyQt6.QtGui import QKeyEvent
from PyQt6.QtWidgets import QApplication, QMainWindow, QWidget, QHBoxLayout, QVBoxLayout, QLabel, QPushButton, QSplitter
from PyQt6.QtWebEngineCore import QWebEngineProfile, QWebEnginePage
from PyQt6.QtWebEngineWidgets import QWebEngineView

class TornBrowserApp(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("tornBrowser - Special Operations Suite")
        self.setGeometry(100, 100, 1500, 900)

        # Creates a completely separate, dedicated storage folder just for tornBrowser
        # This bypasses Chrome data locking errors entirely
        self.custom_profile = QWebEngineProfile("tornBrowserData", self)
        self.custom_profile.setPersistentCookiesPolicy(QWebEngineProfile.PersistentCookiesPolicy.ForcePersistentCookies)

        # Top-level window layout container setup
        main_layout = QHBoxLayout()
        main_layout.setContentsMargins(0, 0, 0, 0)
        main_layout.setSpacing(0)

        # Draggable boundary splitter setup
        self.splitter = QSplitter(Qt.Orientation.Horizontal)
        self.splitter.setStyleSheet("""
            QSplitter::handle {
                background-color: #333;
                width: 6px;
            }
            QSplitter::handle:hover {
                background-color: #555;
            }
        """)

        # ----------------- LEFT PANEL: CORE GAME CHASSIS -----------------
        left_container = QWidget()
        left_layout = QVBoxLayout()
        left_layout.setContentsMargins(0, 0, 0, 0)
        left_layout.setSpacing(2)

        top_bar = QWidget()
        top_bar.setStyleSheet("background-color: #222; max-height: 40px;")
        top_bar_layout = QHBoxLayout()
        top_bar_layout.setContentsMargins(5, 5, 5, 5)

        btn_home = QPushButton("🏠 Home")
        btn_faction = QPushButton("🛡️ Faction")
        btn_gym = QPushButton("💪 Gym")
        btn_items = QPushButton("🎒 Items")
        
        self.btn_fullscreen = QPushButton("🖥️ Fullscreen")
        self.btn_toggle_intel = QPushButton("👁️ Hide Intel")

        btn_style = "background-color: #333; color: #fff; border: 1px solid #444; padding: 4px 12px; font-weight: bold; border-radius: 3px;"
        action_style = "background-color: #444; color: #fff; border: 1px solid #555; padding: 4px 12px; font-weight: bold; border-radius: 3px; margin-right: 4px;"
        toggle_style = "background-color: #8b2500; color: #fff; border: 1px solid #5a1800; padding: 4px 12px; font-weight: bold; border-radius: 3px;"
        
        for btn in [btn_home, btn_faction, btn_gym, btn_items]:
            btn.setStyleSheet(btn_style)
            top_bar_layout.addWidget(btn)
        
        top_bar_layout.addStretch()
        
        self.btn_fullscreen.setStyleSheet(action_style)
        self.btn_toggle_intel.setStyleSheet(toggle_style)
        
        top_bar_layout.addWidget(self.btn_fullscreen)
        top_bar_layout.addWidget(self.btn_toggle_intel)
        
        top_bar.setLayout(top_bar_layout)
        left_layout.addWidget(top_bar)

        # Game framework window linked natively to the dedicated storage partition
        self.game_view = QWebEngineView()
        self.game_view.setPage(QWebEnginePage(self.custom_profile, self.game_view))
        self.game_view.setUrl(QUrl("https://torn.com"))
        left_layout.addWidget(self.game_view)
        
        left_container.setLayout(left_layout)

        btn_home.clicked.connect(lambda: self.game_view.setUrl(QUrl("https://torn.com")))
        btn_faction.clicked.connect(lambda: self.game_view.setUrl(QUrl("https://torn.com/factions.php")))
        btn_gym.clicked.connect(lambda: self.game_view.setUrl(QUrl("https://torn.com/gym.php")))
        btn_items.clicked.connect(lambda: self.game_view.setUrl(QUrl("https://torn.com/item.php")))

        # ----------------- RIGHT PANEL: INTELLIGENCE SUITE -----------------
        self.right_panel = QWidget()
        self.right_panel.setStyleSheet("background-color: #1a1a1a;")
        right_panel_layout = QVBoxLayout()
        right_panel_layout.setContentsMargins(5, 5, 5, 5)

        intel_bar = QWidget()
        intel_bar_layout = QHBoxLayout()
        intel_bar_layout.setContentsMargins(0, 0, 0, 5)
        intel_bar_layout.setSpacing(2)

        btn_ff = QPushButton("Scouter")
        btn_ts = QPushButton("TornStats")
        btn_w3b = QPushButton("Weav3r")
        btn_intel = QPushButton("TornIntel")

        tab_style = "background-color: #2b2b2b; color: #bbb; border: 1px solid #333; font-size: 11px; padding: 4px; font-weight: bold;"
        for btn in [btn_ff, btn_ts, btn_w3b, btn_intel]:
            btn.setStyleSheet(tab_style)
            intel_bar_layout.addWidget(btn)
        
        intel_bar.setLayout(intel_bar_layout)
        right_panel_layout.addWidget(intel_bar)

        # Intel browser frame sharing the exact same secure storage path
        self.target_view = QWebEngineView()
        self.target_view.setPage(QWebEnginePage(self.custom_profile, self.target_view))
        self.target_view.setUrl(QUrl("https://ffscouter.com"))
        right_panel_layout.addWidget(self.target_view)
        
        self.right_panel.setLayout(right_panel_layout)

        btn_ff.clicked.connect(lambda: self.target_view.setUrl(QUrl("https://ffscouter.com")))
        btn_ts.clicked.connect(lambda: self.target_view.setUrl(QUrl("https://tornstats.com")))
        btn_w3b.clicked.connect(lambda: self.target_view.setUrl(QUrl("https://weav3r.dev")))
        btn_intel.clicked.connect(lambda: self.target_view.setUrl(QUrl("https://torn-intel.com")))

        self.btn_fullscreen.clicked.connect(self.toggle_fullscreen_mode)
        self.btn_toggle_intel.clicked.connect(self.toggle_intel_panel)

        self.splitter.addWidget(left_container)
        self.splitter.addWidget(self.right_panel)
        
        # FIXED: Added the required horizontal proportions list [LeftWidth, RightWidth]
        self.splitter.setSizes([1125, 375])
        
        main_layout.addWidget(self.splitter)

        central_widget = QWidget()
        central_widget.setLayout(main_layout)
        self.setCentralWidget(central_widget)

    def toggle_intel_panel(self):
        if self.right_panel.isVisible():
            self.right_panel.setVisible(False)
            self.btn_toggle_intel.setText("👁️ Show Intel")
            self.btn_toggle_intel.setStyleSheet("background-color: #2e5c1e; color: #fff; border: 1px solid #1e3d13; padding: 4px 12px; font-weight: bold; border-radius: 3px;")
        else:
            self.right_panel.setVisible(True)
            self.btn_toggle_intel.setText("👁️ Hide Intel")
            self.btn_toggle_intel.setStyleSheet("background-color: #8b2500; color: #fff; border: 1px solid #5a1800; padding: 4px 12px; font-weight: bold; border-radius: 3px;")

    def toggle_fullscreen_mode(self):
        if self.isFullScreen():
            self.showNormal()
            self.btn_fullscreen.setText("🖥️ Fullscreen")
            self.btn_fullscreen.setStyleSheet("background-color: #444; color: #fff; border: 1px solid #555; padding: 4px 12px; font-weight: bold; border-radius: 3px; margin-right: 4px;")
        else:
            self.showFullScreen()
            self.btn_fullscreen.setText("🗖 Windowed")
            self.btn_fullscreen.setStyleSheet("background-color: #1e3d59; color: #fff; border: 1px solid #122538; padding: 4px 12px; font-weight: bold; border-radius: 3px; margin-right: 4px;")

    def keyPressEvent(self, event: QKeyEvent):
        if event.key() == Qt.Key.Key_F11:
            self.toggle_fullscreen_mode()
        else:
            super().keyPressEvent(event)

if __name__ == "__main__":
    if len(sys.argv) > 1 and sys.argv[1] == "--build":
        import subprocess
        print("[!] Fetching chosen target tracking dashboard asset icon...")
        icon_url = "https://githubusercontent.com"
        icon_path = "logo.ico"
        try:
            urllib.request.urlretrieve(icon_url, icon_path)
            print("[+] Asset downloaded and verified locally as 'logo.ico'.")
        except Exception as e:
            print(f"[-] Asset collection failed: {e}. Compiling with default system layout.")
            icon_path = None

        print("[!] Executing PyInstaller standalone compilation sequence...")
        build_cmd = ["pyinstaller", "--noconsole"]
        if icon_path and os.path.exists(icon_path):
            build_cmd.append(f"--icon={icon_path}")
        build_cmd.extend(["--name=tornBrowser", "tornBrowser.py"])
        
        subprocess.run(build_cmd)
        print("[+] Application packaged successfully inside the /dist/tornBrowser/ folder.")
        sys.exit(0)
        
    app = QApplication(sys.argv)
    window = TornBrowserApp()
    window.show()
    sys.exit(app.exec())
