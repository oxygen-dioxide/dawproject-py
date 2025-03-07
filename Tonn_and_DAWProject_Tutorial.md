# **Automating Multitrack Mixing with Tonn API and DAWProject: A Guide for Developers**

## **Introduction**

In modern music production, automation is key. Whether you’re developing a DAW-integrated tool, running a music tech startup, or streamlining workflows for independent producers, automating the mixing process can save countless hours. This tutorial demonstrates how to use the **Tonn API** to retrieve a mix preview, download stems, apply processing, and format them for **Bitwig Studio and PreSonus Studio One**—all programmatically.

What makes this particularly powerful is how **Tonn API integrates seamlessly with** [DAWProject-py](https://github.com/bitwig/dawproject-py), an open-source project that allows for structured export to DAWs. This means that, in just a few steps, you can take your audio, apply mix settings like **EQ, Compression, Gain, and Panning**, and generate a DAW session file that loads everything into your preferred environment.

This is a **simple example**, but it highlights the potential for fully automated, API-driven mixing workflows—something that can **revolutionize** audio production tools. For a more detailed example check out the GitHub repository.

---

## **What You’ll Learn**

- How to **retrieve a mix preview** from the Tonn API
- How to **download and process multitrack audio files**
- How to **apply gain, EQ, and compression settings** programmatically
- How to **export directly to DAWs** using **DAWProject-py**
- How to **handle network errors and retries gracefully**

---

## **Prerequisites**

Before diving into the code, ensure you have:

- Python 3 installed
- The following dependencies installed via `pip`:
  ```sh
  pip install requests soundfile numpy
  ```
- An **API key** from [Tonn Portal](https://tonn-portal.roexaudio.com)
- Bitwig Studio or PreSonus Studio One (optional, but recommended for testing project files)
- **Access to a cloud storage bucket to store demo audio files** (Tonn API requires audio files to be stored in a bucket before processing)

---

## **1. Setting Up API Communication**

The Tonn API acts as the backbone of this process, allowing us to send **multitrack audio** for automated mixing and retrieve the processed output. To start, let’s define some constants for interacting with the API:

```python
import time
import requests
import json
import os
import soundfile as sf

# Base API endpoint for Tonn
BASE_URL = "https://tonn.roexaudio.com"

# Retry settings for API calls
MAX_RETRIES = 3
RETRY_DELAY = 2  # seconds

API_KEY = "go to https://tonn-portal.roexaudio.com to get one"
```

We’ll use `requests` to interact with the API, ensuring that we handle failures gracefully with **retry mechanisms**.

---

## **2. Downloading Multitrack Audio Files**

Before processing, we need to fetch the **original stems** that will be used for the mix. Our function downloads these files while handling errors and retries.

```python
def download_audio(file_name, url, output_dir="audio_in"):
    """
    Downloads an audio file from the given URL and saves it locally.
    """
    os.makedirs(output_dir, exist_ok=True)
    local_filename = os.path.join(output_dir, file_name)
    
    for attempt in range(MAX_RETRIES):
        try:
            print(f"Downloading {file_name} (Attempt {attempt + 1})...")
            response = requests.get(url, stream=True, timeout=10)
            response.raise_for_status()

            with open(local_filename, "wb") as file:
                for chunk in response.iter_content(chunk_size=8192):
                    file.write(chunk)
            
            print(f"Successfully downloaded {file_name}")
            return local_filename
        except requests.exceptions.RequestException as e:
            print(f"Error downloading {file_name}: {e}")
            if attempt < MAX_RETRIES - 1:
                time.sleep(RETRY_DELAY ** (attempt + 1))
            else:
                print(f"Failed to download {file_name} after {MAX_RETRIES} attempts.")
                return None
```
We apply exponential backoff ```(RETRY_DELAY ** (attempt + 1))``` to avoid overloading the API with repeated requests.

---

## **3. Retrieving and Polling the Mix Preview**

Once we submit our multitrack audio to the Tonn API, we must poll for the preview mix until it’s ready. The following function repeatedly checks the API and retrieves the processed mix when available:

```python
def poll_preview_mix(task_id, headers, max_attempts=30, poll_interval=5):
    """
    Polls the API until the preview mix is ready.
    """
    retrieve_url = f"{BASE_URL}/retrievepreviewmix"
    retrieve_payload = {"multitrackData": {"multitrackTaskId": task_id, "retrieveFXSettings": True}}
    
    print("Polling for the preview mix...")
    for attempt in range(max_attempts):
        response = requests.post(retrieve_url, json=retrieve_payload, headers=headers)
        
        if response.status_code == 200:
            results = response.json().get("previewMixTaskResults", {})
            if results.get("status") == "MIX_TASK_PREVIEW_COMPLETED":
                print("Preview mix is ready!")
                return results
        elif response.status_code == 202:
            print(f"Attempt {attempt + 1}: Still processing...")
        else:
            print("Unexpected error while polling.")
            return None

        time.sleep(poll_interval)
    
    print("Mix preview was not available after polling. Try again later.")
    return None
```

---

## **4. Processing Audio: Gain, EQ, and Compression**

Once we have the mix output settings, we apply **gain adjustments, equalization, and compression**.

```python
def format_audio_tracks_for_daw(mix_output_settings, downloaded_files):
    """
    Applies mix settings (gain, EQ, compression) to each track.
    """
    audio_tracks = []
    
    for track_name, settings in mix_output_settings.items():
        file_path = downloaded_files.get(track_name)
        if not file_path:
            continue

        audio_x, sr = sf.read(file_path, dtype="float32")
        audio_x *= settings["gain_settings"]["initial_gain"]
        sf.write(file_path, audio_x, sr)

        audio_tracks.append({
            "file_path": file_path,
            "gain": settings["gain_settings"]["gain_amplitude"],
            "eq_settings": settings.get("eq_settings", []),
            "compressor_settings": settings.get("drc_settings", {})
        })
    
    return audio_tracks
```

---

## **5. Exporting to Bitwig Studio and PreSonus Studio One**

We format our processed tracks and **automatically create a DAW project** using DAWProject-py.

```python
import createBitwigProject

def create_bitwig_project(audio_tracks):
    createBitwigProject.create_project_with_audio_tracks(audio_tracks)
```
This step ensures seamless DAW integration, making our automated mixing process immediately usable for further production and tweaking.

---

## **6. Additional Resources**

To help you get started, we've provided a **demo repository** containing the full source code and example payloads. You can find it in the same directory.

For more on DAW session exports, check out **[DAWProject-py](https://github.com/bitwig/dawproject-py)**.

---

## **Conclusion**

By leveraging the **Tonn API**, we’ve automated a core part of music production: mixing multitrack audio programmatically.

🚀 Ready to build the future of music production? Get your API key from [Tonn Portal](https://tonn-portal.roexaudio.com) and start automating today! Also, be sure to check out the **[Demo Repository - Link TBD]** for example implementations!
