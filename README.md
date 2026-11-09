<div align="center">

# 🖼️ Image Caption Generator 🤖

### *Upload an image. Get a meaningful caption. Powered by AI.*

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-App-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![Google GenAI](https://img.shields.io/badge/Google%20GenAI-SDK-4285F4?style=for-the-badge&logo=google&logoColor=white)
![Computer Vision](https://img.shields.io/badge/Computer%20Vision-Image%20Understanding-8A2BE2?style=for-the-badge)
![Deep Learning](https://img.shields.io/badge/Deep%20Learning-Multimodal%20AI-FF6F00?style=for-the-badge)
![License](https://img.shields.io/badge/License-TODO-lightgrey?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)

<!-- TODO: Replace with a banner or demo GIF -->
<img src="https://placehold.co/900x300/1f2937/ffffff?text=Image+Caption+Generator+Banner" alt="Project Banner" width="100%"/>

[🚀 Demo](#-screenshots--demo) · [⚙️ Installation](#️-installation--setup) · [🏗️ Architecture](#️-project-architecture) · [🔮 Future Work](#-future-improvements)

</div>

---

## 📖 Overview

> **Image Caption Generator** is an AI-powered web app that looks at an image you upload and writes a natural-language description of it.

It combines **Computer Vision** and **Natural Language Generation** by sending the uploaded image, along with a text prompt, to a pretrained multimodal AI model through the **Google GenAI SDK**. The user interface is built with **Streamlit**.

| 🎯 Goal | 💡 Approach | 🖥️ Interface |
|:---:|:---:|:---:|
| Turn images into meaningful text | Multimodal model called via API | Streamlit web app |

---

## ✨ Key Features

| | Feature | Description |
|:---:|---|---|
| 📤 | **Image Upload** | Upload an image through the Streamlit interface |
| 🧠 | **AI Caption Generation** | Image + prompt sent to a multimodal model to produce a caption |
| 🎛️ | **Controlled Creativity** | Generation uses `temperature=0.4` for focused, consistent captions |
| 🔁 | **Automatic Retry** | Retries up to **3 attempts** on temporary API errors (`429, 500, 502, 503, 504`) |
| ⏱️ | **Exponential Backoff** | Waits **2s, then 4s** between retries |
| ⚠️ | **Live User Feedback** | Shows a Streamlit warning while retrying |
| 🛡️ | **Empty-Response Handling** | Raises a clear error if the model returns no caption |
| 🧩 | **Extra features** | <!-- TODO: add anything else your app does (download caption, multiple styles, etc.) --> |

---

## 🛠️ Technologies & Tools

> Based on the code provided. Add any other libraries from your full project.

| Technology | Badge | Purpose |
|---|---|---|
| **Python** | ![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=white) | Core language |
| **Streamlit** | ![Streamlit](https://img.shields.io/badge/-Streamlit-FF4B4B?logo=streamlit&logoColor=white) | Web UI and user warnings (`st.warning`) |
| **Google GenAI SDK** | ![Google](https://img.shields.io/badge/-google--genai-4285F4?logo=google&logoColor=white) | `client.models.generate_content`, `types.GenerateContentConfig`, `errors.APIError` |
| **time** (stdlib) | ![Python](https://img.shields.io/badge/-time-3776AB?logo=python&logoColor=white) | Delay between retries |
| **Image library** | ![TODO](https://img.shields.io/badge/-TODO-lightgrey) | <!-- TODO: e.g. how the uploaded image is loaded --> |
| **Other** | ![TODO](https://img.shields.io/badge/-TODO-lightgrey) | <!-- TODO --> |

---

## 🏗️ Project Architecture

```mermaid
flowchart LR
    U["👤 User"] --> UI["🖥️ Streamlit Interface"]
    UI --> APP["🐍 main.py<br/>Application Logic"]
    APP --> RETRY["🔁 generate_caption_with_retry()"]
    RETRY --> SDK["📦 Google GenAI SDK<br/>client.models.generate_content"]
    SDK --> MODEL["🧠 Multimodal AI Model<br/>MODEL_NAME = TODO"]
    MODEL --> SDK
    SDK --> RETRY
    RETRY --> APP
    APP --> UI
    UI --> OUT["📝 Generated Caption"]

    style U fill:#e0f2fe,stroke:#0284c7
    style UI fill:#fee2e2,stroke:#ef4444
    style APP fill:#fef9c3,stroke:#ca8a04
    style RETRY fill:#ede9fe,stroke:#7c3aed
    style SDK fill:#dbeafe,stroke:#2563eb
    style MODEL fill:#dcfce7,stroke:#16a34a
    style OUT fill:#fce7f3,stroke:#db2777
```

---

## 🔄 Workflow: From Upload to Caption

```mermaid
flowchart TD
    A(["📤 User uploads image"]) --> B["🖼️ Image loaded in app<br/>(loading method: TODO)"]
    B --> C["✍️ Prompt prepared<br/>(prompt text: TODO)"]
    C --> D["🚀 generate_caption_with_retry(prompt, image)"]
    D --> E["📡 API call<br/>temperature = 0.4"]
    E --> F{"Response OK?"}

    F -- "Yes" --> G{"response.text empty?"}
    G -- "No" --> H(["✅ Caption displayed to user"])
    G -- "Yes" --> I["❌ ValueError:<br/>empty caption"]

    F -- "APIError" --> J{"Code in<br/>429, 500, 502, 503, 504?"}
    J -- "No" --> K["❌ Error raised"]
    J -- "Yes" --> L{"Last attempt?"}
    L -- "Yes" --> K
    L -- "No" --> M["⚠️ st.warning<br/>wait 2s, then 4s"]
    M --> E

    style A fill:#dbeafe,stroke:#2563eb
    style H fill:#dcfce7,stroke:#16a34a
    style I fill:#fee2e2,stroke:#ef4444
    style K fill:#fee2e2,stroke:#ef4444
    style M fill:#fef9c3,stroke:#ca8a04
```

### ⏳ Request Sequence

```mermaid
sequenceDiagram
    actor User
    participant UI as Streamlit UI
    participant Fn as generate_caption_with_retry
    participant API as Google GenAI API

    User->>UI: Upload image
    UI->>Fn: prompt + image
    loop Up to 3 attempts
        Fn->>API: generate_content(model, [prompt, image], temperature=0.4)
        alt Success with text
            API-->>Fn: caption text
            Fn-->>UI: caption
        else Temporary error (429/500/502/503/504)
            API-->>Fn: APIError
            Fn->>UI: st.warning("Retrying...")
            Note over Fn: sleep 2s, then 4s
        else Other error / final attempt
            API-->>Fn: APIError
            Fn-->>UI: raise error
        end
    end
    UI-->>User: Show caption
```

---

## 🧠 Model Architecture & How It Works

This project does **not** train its own captioning network in the code provided. It sends the image and a text prompt to a **pretrained multimodal model** hosted behind the Google GenAI API, and the model returns the caption.

```mermaid
flowchart LR
    subgraph INPUT["📥 Inputs"]
        P["✍️ Text Prompt"]
        I["🖼️ Image"]
    end

    subgraph MODEL["🧠 Pretrained Multimodal Model (hosted)"]
        direction TB
        M1["Image + text understanding"]
        M2["Text generation"]
        M1 --> M2
    end

    subgraph CONFIG["🎛️ Generation Config"]
        T["temperature = 0.4"]
    end

    P --> MODEL
    I --> MODEL
    CONFIG --> MODEL
    MODEL --> O["📝 Caption (response.text)"]

    style INPUT fill:#e0f2fe,stroke:#0284c7
    style MODEL fill:#dcfce7,stroke:#16a34a
    style CONFIG fill:#fef9c3,stroke:#ca8a04
    style O fill:#fce7f3,stroke:#db2777
```

| Component | Details |
|---|---|
| **Model** | `MODEL_NAME` → <!-- TODO: add the exact model name from your code --> |
| **Inputs** | `[prompt, image]` passed together as `contents` |
| **Temperature** | `0.4` (lower values give more focused, consistent output) |
| **Output** | Text caption read from `response.text` |
| **Prompt** | <!-- TODO: paste or describe your prompt --> |
| **Internal architecture** | Managed by the model provider (not part of this repository) |

---

## ⚙️ Installation & Setup

### 1️⃣ Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

### 2️⃣ Create a virtual environment *(recommended)*

```bash
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3️⃣ Install dependencies

```bash
pip install -r requirements.txt
```

> ⚠️ `requirements.txt` was empty in the uploaded files. Based on the code, it should include at least:
>
> ```text
> streamlit
> google-genai
> # TODO: add any other libraries your project imports (e.g. an image library)
> ```

### 4️⃣ Configure your API key

```bash
# TODO: replace with the exact method your code uses
export <API_KEY_ENV_VAR>="your_api_key_here"
```

> 🔐 Never commit your API key to GitHub. Keep it in an environment variable or a `.env` file listed in `.gitignore`.

### 5️⃣ Run the app

```bash
streamlit run main.py
```

Then open **http://localhost:8501** in your browser.

---

## 📁 Project Structure

> Only `main.py` and `requirements.txt` were provided. Update the rest to match your repo.

```text
Image-Caption-Generator/
│
├── 📄 main.py              # Streamlit app + caption generation with retry logic
├── 📄 requirements.txt     # Python dependencies
├── 📄 README.md            # Project documentation
├── 📂 assets/              # TODO: screenshots and sample images
│   ├── 🖼️ demo.png
│   └── 🖼️ sample_1.jpg
├── 📄 .env                 # TODO: API key (do NOT commit)
├── 📄 .gitignore           # TODO
└── 📄 LICENSE              # TODO
```

---

## 🖼️ Sample Input & Output

| 📥 Input Image | 📤 Generated Caption |
|:---:|:---|
| <img src="https://placehold.co/300x200/e0f2fe/0284c7?text=Sample+Image+1" width="300"/> | `TODO: paste the caption your app generated` |
| <img src="https://placehold.co/300x200/dcfce7/16a34a?text=Sample+Image+2" width="300"/> | `TODO: paste the caption your app generated` |
| <img src="https://placehold.co/300x200/fef9c3/ca8a04?text=Sample+Image+3" width="300"/> | `TODO: paste the caption your app generated` |

---

## 📸 Screenshots & Demo

<div align="center">

| 🏠 Home Page | 📤 Image Upload |
|:---:|:---:|
| <img src="https://placehold.co/450x280/1f2937/ffffff?text=Home+Page" width="450"/> | <img src="https://placehold.co/450x280/1f2937/ffffff?text=Upload+Image" width="450"/> |

| ✅ Caption Result | ⚠️ Retry Warning |
|:---:|:---:|
| <img src="https://placehold.co/450x280/1f2937/ffffff?text=Caption+Result" width="450"/> | <img src="https://placehold.co/450x280/1f2937/ffffff?text=Retry+Warning" width="450"/> |

🎥 **Live Demo:** `TODO: add link` &nbsp;|&nbsp; 🎬 **Video Walkthrough:** `TODO: add link`

</div>

---

## 🔮 Future Improvements

- [ ] 🌐 Multi-language caption support
- [ ] 🎨 Selectable caption styles (short, detailed, creative)
- [ ] 📦 Batch caption generation for multiple images
- [ ] 💾 Download or copy captions and export to CSV/JSON
- [ ] 🔊 Text-to-speech for generated captions
- [ ] 📱 Improved mobile-friendly layout
- [ ] ☁️ Cloud deployment (e.g. Streamlit Community Cloud)
- [ ] 🧪 Automated tests for the retry logic
- [ ] ➕ `TODO: add your own ideas`

---

## 👨‍💻 Author

<div align="center">

| | |
|:---:|:---|
| 👤 **Name** | `TODO: Your Name` |
| 📧 **Email** | `TODO: your.email@example.com` |
| 💼 **LinkedIn** | `TODO: link` |
| 🐙 **GitHub** | `TODO: https://github.com/<your-username>` |

</div>

---

## 🏁 Conclusion

**Image Caption Generator** shows how modern multimodal AI can turn an uploaded image into a clear text description. A simple Streamlit interface, a low-temperature generation setting, and a retry system with exponential backoff make it reliable as well as easy to use.

<div align="center">

### ⭐ If you found this project useful, please give it a star! ⭐

*Made with ❤️ and Python*

</div>
