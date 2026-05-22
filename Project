%%writefile app.py

import streamlit as st
import streamlit.components.v1 as components
import pdfplumber
from docx import Document
import pandas as pd
import re
import random
import sqlite3
import plotly.express as px
from datetime import date
import networkx as nx
import matplotlib.pyplot as plt

# =========================================
# PAGE CONFIG
# =========================================

st.set_page_config(
    page_title="FlashMind Ultimate",
    page_icon="🧠",
    layout="wide"
)

# =========================================
# DATABASE
# =========================================

conn = sqlite3.connect("flashmind.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS flashcards (
    question TEXT,
    answer TEXT
)
""")

conn.commit()

# =========================================
# CUSTOM CSS
# =========================================

st.markdown("""

<style>

.main{
    background:linear-gradient(to right,#eef2ff,#ffffff);
}

.title{
    text-align:center;
    font-size:65px;
    font-weight:800;
    color:#111827;
}

.subtitle{
    text-align:center;
    font-size:22px;
    color:#6b7280;
    margin-bottom:30px;
}

.metric-box{
    background:white;
    padding:25px;
    border-radius:25px;
    text-align:center;
    box-shadow:0px 8px 20px rgba(0,0,0,0.1);
}

.summary-box{
    background:white;
    padding:25px;
    border-radius:20px;
    box-shadow:0px 8px 20px rgba(0,0,0,0.08);
}

</style>

""", unsafe_allow_html=True)

# =========================================
# HEADER
# =========================================

st.markdown(
    "<div class='title'>🧠 FlashMind</div>",
    unsafe_allow_html=True
)

st.markdown(
    "<div class='subtitle'>AI Powered Study Platform</div>",
    unsafe_allow_html=True
)

# =========================================
# SIDEBAR
# =========================================

st.sidebar.title("⚙️ FlashMind Settings")

num_cards = st.sidebar.slider(
    "Number of Flashcards",
    5,
    50,
    15
)

# =========================================
# STUDY STREAK
# =========================================

if "last_visit" not in st.session_state:

    st.session_state.last_visit = str(date.today())
    st.session_state.streak = 1

if st.session_state.last_visit != str(date.today()):

    st.session_state.streak += 1
    st.session_state.last_visit = str(date.today())

st.sidebar.metric(
    "🔥 Study Streak",
    st.session_state.streak
)

# =========================================
# FILE UPLOAD
# =========================================

uploaded_file = st.file_uploader(
    "📂 Upload Study Material",
    type=["pdf", "txt", "docx", "csv"]
)

# =========================================
# EXTRACT TEXT
# =========================================

def extract_text(file):

    file_type = file.name.split(".")[-1].lower()

    text = ""

    if file_type == "pdf":

        with pdfplumber.open(file) as pdf:

            for page in pdf.pages:

                page_text = page.extract_text()

                if page_text:
                    text += page_text + " "

    elif file_type == "txt":

        text = file.read().decode("utf-8")

    elif file_type == "docx":

        doc = Document(file)

        for para in doc.paragraphs:
            text += para.text + " "

    elif file_type == "csv":

        df = pd.read_csv(file)

        text = df.astype(str).to_string()

    return text

# =========================================
# GENERATE SMART FLASHCARDS
# =========================================

def generate_flashcards(text, num_cards):

    text = re.sub(r"\s+", " ", text)

    sentences = re.split(
        r'(?<=[.!?]) +',
        text
    )

    flashcards = []

    used_questions = set()

    question_starters = [

        "What is",
        "Explain",
        "Why is",
        "How does",
        "Describe",
        "What do you understand about",
        "What are the advantages of",
        "What is the importance of"

    ]

    for sentence in sentences:

        sentence = sentence.strip()

        # Skip very short sentences

        if len(sentence.split()) < 12:
            continue

        # Clean sentence

        sentence = sentence.replace("•", "")
        sentence = sentence.replace("\n", " ")

        sentence = re.sub(
            r'[^A-Za-z0-9,.()\- ]',
            '',
            sentence
        )

        words = sentence.split()

        # Extract meaningful keywords

        keywords = []

        stopwords = [

            "which","their","there","about",
            "would","these","those","because",
            "between","through","during",
            "before","after","having",
            "being","where","while",
            "using","system","process",
            "important","different",
            "method","methods","application",
            "applications"

        ]

        for word in words:

            clean_word = word.lower()

            if (

                len(clean_word) > 5

                and clean_word not in stopwords

                and clean_word.isalpha()

            ):

                keywords.append(word)

        if len(keywords) == 0:
            continue

        topic = keywords[0]

        starter = random.choice(
            question_starters
        )

        # Generate AI-style questions

        if starter == "What is":

            question = f"What is {topic}?"

        elif starter == "Explain":

            question = f"Explain the concept of {topic}."

        elif starter == "Why is":

            question = f"Why is {topic} important?"

        elif starter == "How does":

            question = f"How does {topic} work?"

        elif starter == "Describe":

            question = f"Describe {topic}."

        elif starter == "What do you understand about":

            question = f"What do you understand about {topic}?"

        elif starter == "What are the advantages of":

            question = f"What are the advantages of {topic}?"

        else:

            question = f"What is the importance of {topic}?"

        if question in used_questions:
            continue

        used_questions.add(question)

        # Create GPT-style answer

        answer = sentence

        if len(answer) > 450:
            answer = answer[:450] + "..."

        answer = answer.strip()

        answer = answer[0].upper() + answer[1:]

        flashcards.append({

            "question": question,
            "answer": answer

        })

        if len(flashcards) >= num_cards:
            break

    return flashcards
# =========================================
# MAIN APP
# =========================================

if uploaded_file:

    with st.spinner(
        "🧠 AI is analyzing your notes..."
    ):

        text = extract_text(
            uploaded_file
        )

        flashcards = generate_flashcards(
            text,
            num_cards
        )

    # =====================================
    # METRICS
    # =====================================

    col1, col2, col3 = st.columns(3)

    with col1:

        st.markdown(f"""
        <div class='metric-box'>
        <h2>{len(flashcards)}</h2>
        <p>Flashcards</p>
        </div>
        """, unsafe_allow_html=True)

    with col2:

        st.markdown(f"""
        <div class='metric-box'>
        <h2>{len(text.split())}</h2>
        <p>Words Extracted</p>
        </div>
        """, unsafe_allow_html=True)

    with col3:

        st.markdown(f"""
        <div class='metric-box'>
        <h2>{uploaded_file.name.split('.')[-1].upper()}</h2>
        <p>File Type</p>
        </div>
        """, unsafe_allow_html=True)

    wrong_topics = []
    score = 0

    # =====================================
    # TABS
    # =====================================

    tab1, tab2, tab3, tab4, tab5, tab6 = st.tabs([

        "🧠 Flashcards",
        "🧪 Quiz",
        "📄 Summary",
        "🤖 AI Tutor",
        "🧭 Mind Map",
        "📈 Analytics"

    ])

    # =====================================
    # FLASHCARDS
    # =====================================

    with tab1:

        st.subheader("🧠 AI Flashcards")

        flashcard_html = """

        <style>

        .flashcard-container{
            display:grid;
            grid-template-columns:
            repeat(auto-fit,minmax(300px,1fr));
            gap:25px;
            margin-top:20px;
        }

        .flashcard{
            background:transparent;
            width:100%;
            height:260px;
            perspective:1000px;
        }

        .flashcard-inner{
            position:relative;
            width:100%;
            height:100%;
            transition:transform 0.8s;
            transform-style:preserve-3d;
            cursor:pointer;
        }

        .flashcard.flipped .flashcard-inner{
            transform:rotateY(180deg);
        }

        .flashcard-front,
        .flashcard-back{
            position:absolute;
            width:100%;
            height:100%;
            border-radius:25px;
            padding:25px;
            display:flex;
            flex-direction:column;
            justify-content:center;
            align-items:center;
            backface-visibility:hidden;
            box-shadow:
            0px 8px 20px rgba(0,0,0,0.15);
        }

        .flashcard-front{
            background:
            linear-gradient(135deg,#6366f1,#8b5cf6);
            color:white;
        }

        .flashcard-back{
            background:white;
            color:#111827;
            transform:rotateY(180deg);
        }

        .question{
            font-size:22px;
            font-weight:bold;
            text-align:center;
        }

        .answer{
            font-size:16px;
            line-height:1.7;
            text-align:center;
        }

        .flip-note{
            margin-top:20px;
            opacity:0.8;
            font-size:14px;
        }

        </style>

        <div class="flashcard-container">
        """

        for flashcard in flashcards:

            flashcard_html += f"""

            <div class="flashcard"
                 onclick="this.classList.toggle('flipped')">

                <div class="flashcard-inner">

                    <div class="flashcard-front">

                        <div class="question">
                            ❓ {flashcard['question']}
                        </div>

                        <div class="flip-note">
                            Click to Flip
                        </div>

                    </div>

                    <div class="flashcard-back">

                        <div class="answer">
                            ✅ {flashcard['answer']}
                        </div>

                    </div>

                </div>

            </div>
            """

        flashcard_html += "</div>"

        components.html(
            flashcard_html,
            height=1200,
            scrolling=True
        )

    # =====================================
    # QUIZ
    # =====================================

    with tab2:

        st.subheader("🧪 Quiz Mode")

        for i, flashcard in enumerate(
            flashcards
        ):

            st.write(
                f"### {flashcard['question']}"
            )

            answer = st.text_input(
                "Your Answer",
                key=f"quiz_{i}"
            )

            if answer:

                if answer.lower() in flashcard[
                    'answer'
                ].lower():

                    st.success("Correct ✅")
                    score += 1

                else:

                    st.error("Incorrect ❌")

                    wrong_topics.append(
                        flashcard['question']
                    )

        st.info(
            f"🎯 Score: {score}"
        )

    # =====================================
    # SUMMARY
    # =====================================

    with tab3:

        st.subheader("📄 Notes Summary")

        clean_summary = text[:5000]

        clean_summary = clean_summary.replace("<", "")
        clean_summary = clean_summary.replace(">", "")
        clean_summary = clean_summary.replace("{", "")
        clean_summary = clean_summary.replace("}", "")

        st.text_area(
            "Summary",
            clean_summary,
            height=500
        )

    # =====================================
    # AI TUTOR
    # =====================================

    with tab4:

        st.subheader("🤖 FlashMind AI Tutor")

        user_question = st.text_input(
            "Ask Anything From Your Notes"
        )

        if user_question:

            chunks = re.split(
                r'(?<=[.!?]) +',
                text
            )

            chunks = [

                c.strip()

                for c in chunks

                if len(c.split()) > 8

            ]

            best_chunk = ""
            best_match = 0

            for chunk in chunks:

                match_score = 0

                for word in user_question.lower().split():

                    if word in chunk.lower():

                        match_score += 1

                if match_score > best_match:

                    best_match = match_score
                    best_chunk = chunk

            if best_chunk:

                answer = best_chunk.strip()

                answer = re.sub(
                    r'\s+',
                    ' ',
                    answer
                )

                st.markdown(f"""

                <div style="
                    background:linear-gradient(135deg,#111827,#1f2937);
                    color:white;
                    padding:30px;
                    border-radius:20px;
                    margin-top:20px;
                    font-size:18px;
                    line-height:1.9;
                    box-shadow:0px 8px 20px rgba(0,0,0,0.2);
                ">

                <h2>🤖 FlashMind Answer</h2>

                <p>{answer}</p>

                </div>

                """, unsafe_allow_html=True)

            else:

                st.error(
                    "No relevant answer found in notes."
                )

    # =====================================
    # MIND MAP
    # =====================================

    with tab5:

        st.subheader("🧭 AI Mind Map")

        words = re.findall(
            r'\b[A-Za-z]{5,}\b',
            text
        )

        stopwords = [

            "which","their","there",
            "about","would","these",
            "those","because",
            "between","through",
            "during","before",
            "after","having",
            "being","where",
            "while","using",
            "system","process",
            "important","different"

        ]

        filtered_words = [

            w.lower()

            for w in words

            if w.lower() not in stopwords

        ]

        freq = {}

        for word in filtered_words:

            freq[word] = freq.get(
                word,
                0
            ) + 1

        top_topics = sorted(
            freq,
            key=freq.get,
            reverse=True
        )[:12]

        G = nx.Graph()

        center_topic = "Main Topic"

        for topic in top_topics:

            G.add_edge(
                center_topic,
                topic
            )

        fig, ax = plt.subplots(
            figsize=(10,7)
        )

        pos = nx.spring_layout(
            G,
            seed=42
        )

        nx.draw(
            G,
            pos,
            with_labels=True,
            node_size=4500,
            font_size=10,
            font_weight="bold",
            ax=ax
        )

        st.pyplot(fig)

    # =====================================
    # ANALYTICS
    # =====================================

    with tab6:

        st.subheader("📈 Study Analytics")

        analytics = pd.DataFrame({

            "Metric":[
                "Flashcards",
                "Quiz Score",
                "Weak Topics"
            ],

            "Value":[
                len(flashcards),
                score,
                len(wrong_topics)
            ]

        })

        fig = px.bar(
            analytics,
            x="Metric",
            y="Value",
            title="Study Dashboard"
        )

        st.plotly_chart(fig)

else:

    st.info(
        "📂 Upload PDF, DOCX, TXT or CSV file to begin."
    )
