![Screenshot (693)](https://github.com/user-attachments/assets/339cc7f3-5600-4f31-889f-66ceb8623b1e)
# Analisis Suara Berdasarkan Jenis Gender Manusia
Voice analysis for detecting gender involves identifying acoustic features that differ between male and female voices, such as:
1. Pitch (F0): Female voices typically have a higher pitch compared to male voices.
2. Formant Frequencies: Resonant frequencies also differ between male and female voices.
3. Timbre: The characteristics of the voice that can influence gender perception.

# Author
1. Ahmad Rafli Al Adzani (2042231001)
2. Muhammad Naufal Zuhair (2042231005)
3. Daffa Naufal Wahyuaji (2042231081)
4. Ahmad Radhy (Supervisor)
5. 
# Audio Visualizer and Recorder

This project is a Python application built using PyQt5, which provides a user-friendly interface for live audio visualization, recording, and gender detection. The application integrates real-time plotting using Matplotlib and supports saving and uploading recorded audio to Edge Impulse.

## Features

### 1. Live Audio Visualization
- **Live Signal Plot**: Displays the real-time audio waveform.
- **Impulse Signal Plot**: Visualizes the first derivative of the audio signal.
- **DFT Plot**: Shows the Discrete Fourier Transform (DFT) of the audio signal.

### 2. Audio Recording
- Records audio from the default microphone using PyAudio.
- Saves audio files in `.wav` format.

### 3. Gender Detection
- A placeholder feature that simulates gender detection (returns "Male" or "Female" randomly).
- Can be extended with a proper machine learning model for real gender detection.

### 4. Integration with Edge Impulse
- Uploads saved audio files to Edge Impulse's training API.
- Uses a secure API key for authentication.

### 5. User-Friendly UI
- Designed with PyQt5 for a modern and clean interface.
- Dark mode with consistent styling for all components.

# Complete Technical Documentation: Audio Recording Application

## Python Technical Details

### 1. Library Dependencies
```python
from PyQt5 import QtWidgets, QtCore        # GUI framework
from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg  # Plot integration
from matplotlib.figure import Figure        # Plot creation
import pyaudio                             # Audio handling
import threading                           # Concurrent execution
import numpy as np                         # Numerical operations
from scipy.io.wavfile import write         # Audio file I/O
import requests                            # API communication
import os                                  # File operations
```

### 2. Audio Processing System

#### 2.1 PyAudio Configuration
```python
self.p = pyaudio.PyAudio()
self.chunk_size = 2048      # Buffer size for audio chunks
self.rate = 44100           # Sampling rate in Hz
```

#### 2.2 Stream Setup
```python
self.stream = self.p.open(
    format=pyaudio.paInt16,  # 16-bit audio
    channels=1,              # Mono audio
    rate=self.rate,         # 44.1kHz sampling
    input=True,             # Input stream
    frames_per_buffer=self.chunk_size
)
```

#### 2.3 Recording Thread
```python
def record_audio(self):
    """
    Continuous recording function:
    1. Reads audio chunks from stream
    2. Converts to NumPy array
    3. Extends audio_data buffer
    """
    while self.running:
        data = self.stream.read(self.chunk_size, exception_on_overflow=False)
        self.audio_data.extend(np.frombuffer(data, dtype=np.int16))
```

### 3. Signal Processing

#### 3.1 Live Signal Processing
```python
# Normalization
signal = np.array(self.audio_data)
normalized_signal = signal / np.max(np.abs(signal) + 1e-6)
time = np.linspace(0, len(normalized_signal) / self.rate, len(normalized_signal))
```

#### 3.2 Impulse Response
```python
# First-order difference
impulse_signal = np.diff(normalized_signal)
```

#### 3.3 Frequency Analysis
```python
# FFT computation
hz = np.fft.rfftfreq(len(normalized_signal), 1 / self.rate)
amplitude = np.abs(np.fft.rfft(normalized_signal))
```

### 4. Data Management

#### 4.1 File Saving
```python
def save_data(self):
    """
    Saves audio data:
    1. Detects gender
    2. Generates filename
    3. Writes WAV file
    4. Uploads to Edge Impulse
    """
    write(filename, self.rate, np.array(self.audio_data).astype(np.int16))
```

#### 4.2 API Integration
```python
def upload_audio_to_edge_impulse(self, audio_filename):
    """
    Edge Impulse upload:
    1. Opens audio file
    2. Sends POST request
    3. Handles response
    """
    response = requests.post(
        self.api_url,
        headers={"x-api-key": self.api_key},
        files={"data": (os.path.basename(audio_filename), f, "audio/wav")}
    )
```

## Qt Framework Technical Details

### 1. Main Window Architecture

#### 1.1 Class Structure
```python
class MainWindow(QtWidgets.QWidget, Ui_Form):
    """
    Main application window:
    - Inherits QWidget functionality
    - Implements Ui_Form interface
    - Manages application logic
    """
```

#### 1.2 Widget Hierarchy
```python
# Plot widgets
self.plot_live_signal = LivePlotWidget(self)
self.plot_impulse_signal = LivePlotWidget(self)
self.plot_dft_signal = LivePlotWidget(self)

# Layout management
self.widget.layout = QtWidgets.QVBoxLayout(self.widget)
self.widget.layout.addWidget(self.plot_live_signal)
```

### 2. Custom Plot Integration

#### 2.1 LivePlotWidget Class
```python
class LivePlotWidget(FigureCanvas):
    """
    Custom plotting widget:
    - Inherits from FigureCanvas
    - Configures matplotlib figure
    - Manages plot appearance
    """
    def __init__(self, parent=None):
        self.figure = Figure(facecolor="black")
        self.ax = self.figure.add_subplot(111, facecolor="black")
        self.configure_plot_appearance()
        super().__init__(self.figure)
```

#### 2.2 Plot Styling
```python
def configure_plot_appearance(self):
    """
    Plot customization:
    - Sets axis colors
    - Configures labels
    - Applies dark theme
    """
    self.ax.tick_params(axis="x", colors="white")
    self.ax.tick_params(axis="y", colors="white")
    self.ax.xaxis.label.set_color("white")
    self.ax.yaxis.label.set_color("white")
    self.ax.title.set_color("white")
```

### 3. Event System

#### 3.1 Timer Implementation
```python
# Plot update timer
self.timer = QtCore.QTimer(self)
self.timer.timeout.connect(self.update_plots)
self.timer.start(100)  # 100ms refresh rate

# Time display timer
self.elapsed_time_timer = QtCore.QTimer(self)
self.elapsed_time_timer.timeout.connect(self.update_elapsed_time)
self.elapsed_time_timer.start(1000)  # 1s update interval
```

#### 3.2 Button Connections
```python
# Control button signals
self.pushButton.clicked.connect(self.start_recording)
self.pushButton_3.clicked.connect(self.stop_recording)
self.pushButton_4.clicked.connect(self.reset_data)
self.pushButton_2.clicked.connect(self.save_data)
```

### 4. UI Styling

#### 4.1 Dark Theme Implementation
```python
self.setStyleSheet("""
    QWidget {
        background-color: black;
        color: white;
    }
    QPushButton {
        background-color: grey;
        color: white;
    }
    QTextEdit {
        background-color: black;
        color: white;
        border: 1px solid grey;
    }
    QLineEdit {
        background-color: black;
        color: white;
        border: 1px solid grey;
    }
""")
```

### 5. Update Mechanisms

#### 5.1 Plot Updates
```python
def update_plots(self):
    """
    Real-time plot updates:
    1. Updates live signal
    2. Updates impulse response
    3. Updates frequency spectrum
    4. Redraws all plots
    """
    # Live signal plot
    self.plot_live_signal.ax.clear()
    self.plot_live_signal.ax.plot(time, normalized_signal, color="red")
    
    # Impulse signal plot
    self.plot_impulse_signal.ax.clear()
    self.plot_impulse_signal.ax.plot(impulse_signal, color="orange")
    
    # DFT signal plot
    self.plot_dft_signal.ax.clear()
    self.plot_dft_signal.ax.plot(hz, amplitude, color="lime")
```

#### 5.2 Time Display
```python
def update_elapsed_time(self):
    """
    Updates recording time display:
    1. Increments elapsed time
    2. Formats time string
    3. Updates display
    """
    self.elapsed_time += 1
    minutes, seconds = divmod(self.elapsed_time, 60)
    self.lineEdit.setText(f"LIFE TIME : {minutes:02}:{seconds:02}")
```

## Application Flow

### 1. Initialization
1. Create main window
2. Initialize audio system
3. Set up plot widgets
4. Configure timers
5. Connect signals

### 2. Recording Process
1. Start button pressed
2. Initialize audio stream
3. Launch recording thread
4. Begin plot updates
5. Track elapsed time

### 3. Data Processing
1. Capture audio chunks
2. Process signals
3. Update visualizations
4. Save data when requested
5. Upload to Edge Impulse

### 4. Cleanup
1. Stop recording
2. Close audio stream
3. Save final data
4. Reset interface

---

## Contact
For any inquiries or support, feel free to reach out at naufalzgt@gmail.com , nopaldapa205@gmail.com and raflidani9@gmail.com.


