# 📘 CFA Study Buddy

> AI-powered personal tutor for CFA Program candidates — built with Streamlit + Google Gemini API.

**Final Project** — *LLM-Based Tools and Gemini API Integration for Data Scientists*

---

## 🎯 Project Overview

| Field | Detail |
|---|---|
| **Project Name** | CFA Study Buddy |
| **Use Case** | Education / Exam Preparation Assistant |
| **Target Users** | CFA Program candidates (Level I, II, III) |
| **LLM Backend** | Google Gemini 2.5 Flash |
| **Framework** | Streamlit |
| **Hosting (dev)** | Google Colab + ngrok tunnel |

CFA Study Buddy adalah AI chatbot yang berperan sebagai personal tutor 24/7 untuk CFA candidates. Berbeda dengan generic chatbot, bot ini dibekali system prompt yang mempositioning dirinya sebagai CFA mentor — menggunakan terminologi CFA Institute yang konsisten, mengaitkan jawaban ke Learning Outcome Statements (LOS), dan menolak menjawab di luar scope curriculum.

### Siapa target pengguna chatbot ini?

CFA candidates yang sedang mempersiapkan ujian Level I, II, atau III — khususnya:

- **Self-study candidates** yang tidak mengambil prep course mahal dan butuh second opinion saat menemui konsep sulit
- **Working professionals** yang belajar di sela jam kerja dan butuh quick clarification tanpa membuka 8 textbook
- **Mahasiswa Indonesia** yang baru kenal CFA curriculum dan butuh penjelasan dalam Bahasa Indonesia sebelum mengerjakan soal exam English
- **Study group facilitators** yang butuh referensi cepat saat diskusi konsep dengan rekan

### Bagaimana chatbot ini membantu pengguna?

1. **Conceptual clarity on demand** — Tanya konsep apa pun di 10 topik CFA (Quant, Econ, FRA, Corporate Issuers, Equity, Fixed Income, Derivatives, Alternative Investments, Portfolio Management, Ethics), dapatkan penjelasan terstruktur yang merujuk ke LOS
2. **Formula breakdown** — Untuk pertanyaan formula, bot konsisten menampilkan formula → penjelasan tiap term → contoh numerik
3. **Ethics scenario walkthrough** — Identifikasi Standard yang relevan + resolusi sesuai CFA Institute guidance
4. **Multi-turn discussions** — Memory percakapan memungkinkan follow-up questions seperti "kasih contoh untuk kasus tadi" tanpa perlu re-explain konteks
5. **Bilingual support** — Switch ke Bahasa Indonesia untuk memahami konsep, switch ke English untuk simulasi exam language

---

## ✨ Features

### Configurable Parameters

| Parameter | Options | Purpose |
|---|---|---|
| **CFA Level Focus** | Level I / II / III / All | Menyesuaikan depth dan complexity jawaban |
| **Response Language** | English / Bahasa Indonesia | Dual-language support |
| **Tone** | Educational / Formal / Casual | Adaptasi gaya komunikasi |
| **Temperature** | 0.0 – 1.0 (slider) | Trade-off konsistensi vs. kreativitas |
| **Max Output Tokens** | 256 – 4096 (slider) | Kontrol panjang jawaban |

### Built-in Capabilities

- 💾 **Conversation memory** — Session state preserves context across questions
- 🔄 **Reset button** — Clear history untuk start fresh topic
- 📄 **Export chat history** — Download sebagai `.txt` atau `.md` untuk catatan belajar
- 🔐 **API key gating** — User memasukkan API key sendiri di UI (tidak hardcoded)
- 🎨 **Clean UI** — Sidebar untuk settings, main area untuk chat — fokus pada belajar

---

## 🏗️ Architecture

```
User Browser
    ↕  (HTTPS)
ngrok tunnel  ←→  Google Colab (port 8501)
                           ↕
                   Streamlit App
                           ↕
                   Google Gemini API
                  (gemini-2.5-flash)
```

**Tech Stack:**
- **UI Framework:** Streamlit (`st.chat_message`, `st.chat_input`, `st.session_state`)
- **LLM:** Google Gemini 2.5 Flash via `google-genai` SDK
- **Tunneling:** pyngrok (development hosting via Colab)
- **Memory:** Streamlit `session_state` + Gemini chat session

---

## 🚀 Quick Start

### Prerequisites

1. Google account untuk mengakses Google Colab
2. Free ngrok account → [https://ngrok.com](https://ngrok.com)
3. Google AI Studio API key → [https://aistudio.google.com](https://aistudio.google.com)

### Run on Google Colab

1. **Clone atau download** notebook `CFA_Study_Buddy_Chatbot.ipynb` dari repo ini
2. **Upload ke Google Colab** (File → Upload notebook)
3. **Setup ngrok token** di Colab Secrets:
   - Klik ikon 🔑 di sidebar kiri Colab
   - Add new secret: name = `NGROK_TOKEN`, value = token dari ngrok dashboard
4. **Run cells sequentially** (Runtime → Run all)
5. **Buka URL ngrok** yang muncul di output cell `run_streamlit(...)`
6. **Paste Google AI API Key** di sidebar app
7. **Mulai belajar!**

### Run Locally (Alternative)

Bila ingin menjalankan tanpa Colab:

```bash
# Clone repo
git clone https://github.com/<username>/cfa-study-buddy.git
cd cfa-study-buddy

# Install dependencies
pip install streamlit google-genai

# Extract Python file dari notebook (atau salin dari %%writefile cell)
# File akan bernama cfa_study_buddy.py

# Run
streamlit run cfa_study_buddy.py
```

---

## 💡 Example Prompts

### Level I — Quantitative Methods
> Explain the difference between geometric mean and arithmetic mean. When should I use each one?

### Level I — Fixed Income (in Bahasa Indonesia)
> Jelaskan apa itu duration dan convexity. Kenapa convexity adalah good thing untuk bond investor?

### Level II — Equity Valuation
> Walk me through the Gordon Growth Model. When does it break down?

### Level III — Portfolio Management
> What are the key differences between strategic and tactical asset allocation in IPS construction?

### Ethics (All Levels)
> I overheard my colleague at a coffee shop discussing material non-public information about an upcoming earnings release. Which CFA Standard applies, and what should I do?

### Test Memory (Follow-up)
After asking about duration:
> Bisa kasih contoh kalkulasi modified duration untuk bond yang barusan kamu jelaskan?

---

## 📁 Repository Structure

```
cfa-study-buddy/
├── CFA_Study_Buddy_Chatbot.ipynb    # Main Colab notebook (deliverable)
├── README.md                         # This file
└── screenshots/                      # UI screenshots
    ├── 01_welcome_screen.png
    ├── 02_sidebar_settings.png
    ├── 03_chat_in_action.png
    ├── 04_bilingual_response.png
    └── 05_export_history.png
```

---

## 🔧 Design Decisions

### Dynamic system prompt composition
Daripada hard-code satu prompt panjang, system prompt dibangun dari modular blocks (`TONE_INSTRUCTIONS`, `LANGUAGE_INSTRUCTIONS`, `LEVEL_INSTRUCTIONS`) yang dikomposisi via `build_system_prompt()`. Lebih mudah maintain dan extend dengan preference baru.

### Chat session re-initialization on config change
Gemini SDK hanya menerima `config` (system_instruction, temperature, max_output_tokens) saat `chats.create()` — tidak bisa diubah mid-session. Aplikasi melacak `config_signature` dan re-create session saat ada perubahan, sehingga preferensi baru langsung berlaku. Trade-off: server-side conversation context di Gemini reset, tapi UI history (`st.session_state.messages`) tetap.

### Default temperature 0.3
CFA candidates butuh konsistensi & akurasi lebih dari kreativitas. Temperature rendah memberikan jawaban deterministik untuk formula dan definisi — mayoritas pertanyaan CFA. User bisa naikkan untuk diskusi konseptual open-ended.

### Honest disclaimers in system prompt
Bot diinstruksikan eksplisit untuk: (1) tidak fabricate facts, (2) acknowledge uncertainty, (3) redirect off-topic queries, (4) tidak menjanjikan exam outcomes. Ini responsible AI practice — bot adalah study aid, bukan pengganti official curriculum.

---

## ⚠️ Limitations & Disclaimers

- **Bukan pengganti official curriculum.** Selalu verifikasi jawaban bot dengan CFA Institute Learning Modules.
- **Knowledge cutoff.** Bot menggunakan Gemini's training data — informasi tentang perubahan curriculum terkini mungkin tidak tercermin.
- **No guarantee of exam success.** Tool ini meningkatkan pemahaman konsep, tapi tidak menjamin nilai exam.
- **API costs.** Google AI Studio menyediakan free tier yang cukup untuk study session normal. Heavy usage mungkin perlu paid tier.

---

## 📸 Screenshots

*Tambahkan screenshot UI di folder `screenshots/` setelah running app.*

Suggested screenshots:
1. Welcome screen (sebelum API key dimasukkan)
2. Sidebar dengan semua settings expanded
3. Chat conversation in progress (English)
4. Chat conversation in progress (Bahasa Indonesia)
5. Export dialog dengan downloaded file

---

## 📚 References

- [Streamlit Documentation](https://docs.streamlit.io)
- [Google Gen AI Python SDK](https://googleapis.github.io/python-genai/)
- [Google AI Studio](https://aistudio.google.com)
- [pyngrok Documentation](https://pyngrok.readthedocs.io/)
- [CFA Institute Program Curriculum](https://www.cfainstitute.org/programs/cfa)

---

## 👤 Author

**Reinaldi Santoso**

Built as part of bootcamp final project on LLM-Based Tools and Gemini API Integration.

---

## 📄 License

MIT License — feel free to fork, adapt, and learn from this project.
