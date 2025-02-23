# 🎵 DAWProject-Py  
*Python code for working with DAWProject files — enabling DAW interoperability.*

[![License](https://img.shields.io/github/license/roex-audio/dawproject-py)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/roex-audio/dawproject-py.svg)](https://github.com/roex-audio/dawproject-py/stargazers)

## 📖 About

DAWProject is an **open XML-based file format** designed for **seamless project exchange between DAWs**. It allows music producers, engineers, and developers to share **full session data** between different DAWs without losing important information.

The original **DAWProject** repository, developed by **Bitwig**, was written in **Java**. However, at **RoEx**, we primarily develop in **Python and C++**, so we converted the core classes to Python to integrate with our systems and allow more developers to build upon it.

We **love the idea of DAWProject** and want to see it in **every DAW**. The more people building on it, the better—so we're making our **Python version publicly available**. If anyone wants to **turn it into a pip package**, extend it, or modify it further, feel free!

For reference, we’ve **kept the original Java classes** and implementations for posterity.

---

## 📦 Installation

Since this is a **source-only library**, you can clone the repository and use it directly in your Python project:

```sh
git clone https://github.com/roex-audio/dawproject-py.git
cd dawproject-py
```

Then, import it into your Python project:

```python
from classes.dawProject import DawProject
```

---

## 🚀 Quick Start

### **Loading a DAWProject file (XML-based)**
```python
from classes.dawProject import DawProject

# Load a DAWProject file (XML format)
project = DawProject.load("example.dawproject")

# Print project metadata
print(f"Project Name: {project.name}")
print(f"Tracks: {len(project.structure)}")
```

### **Creating an Empty DAWProject**
```python
from classes.project import Project
from classes.application import Application

# Initialize an empty project
project = Project()
project.application = Application(name="RoEx Automix", version="1.0")

# Save the project as XML
DawProject.save_xml(project, "new_project.dawproject")
```

### **Creating a Project with Audio Tracks**
```python
from classes.utility import Utility
from classes.mixerRole import MixerRole
from classes.contentType import ContentType

# Initialize a project
project = Project()

# Create a master track
master_track = Utility.create_track(name="Master", content_types=set(), mixer_role=MixerRole.MASTER, pan=0.5, volume=1.0)
project.structure.append(master_track)

# Create an audio track
audio_track = Utility.create_track(name="Lead Synth", content_types={ContentType.AUDIO}, mixer_role=MixerRole.REGULAR, pan=0.2, volume=0.8)
audio_track.channel.destination = master_track.channel

# Add the track to the project
project.structure.append(audio_track)

# Save the project as XML
DawProject.save_xml(project, "audio_project.dawproject")
```

---

## 📜 DAWProject Format

DAWProject is an **open XML-based format** designed to enable **cross-DAW compatibility**. Instead of exporting audio stems, `.dawproject` files allow projects to be shared across DAWs while preserving:

- 🎚 **Track names, volume, and panning**  
- 🎛 **Effects and automation data**  
- 🎵 **MIDI and audio region placements**  
- 🕒 **Tempo and time signature changes**  

For the full specification, visit [DAWProject on GitHub](https://github.com/bitwig/dawproject).

---

## 🛠 Development & Contributions

We **welcome contributions!** If you’d like to extend **DAWProject-Py**, whether by improving existing functionality or turning it into a **pip package**, feel free to contribute!

### **Clone the Repository**
```sh
git clone https://github.com/roex-audio/dawproject-py.git
cd dawproject-py
```

### **Contributing**
- Fork the repository  
- Create a feature branch (`git checkout -b feature-name`)  
- Commit your changes (`git commit -m "Add feature XYZ"`)  
- Push to GitHub (`git push origin feature-name`)  
- Open a **Pull Request** 🚀  

---

## 📜 License
This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

## 🌎 Stay Connected
📢 Have ideas or feedback? Open an issue or start a discussion!

🔗 **Website:** [www.roexaudio.com](https://dawproject.com)  
💬 **Socials:** [@roexaudio](https://twitter.com/dawproject)  
📢 **GitHub Discussions:** [DAWProject-Py Discussions](https://github.com/roex-audio/dawproject-py/discussions)  
⭐ **Star this repo** if you find it useful!

---

🎶 **DAWProject-Py – Enabling DAW interoperability!** 🚀
